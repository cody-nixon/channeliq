# Multi-Role Review — ChannelIQ Plan

## Product Perspective

**Is this actually solving the problem?**  
Yes — strongly. The value prop is clear: "Stop guessing what to post. Know what your audience has already proven they want." The core insight (inside-out vs outside-in analysis) is real differentiation. vidIQ will tell you "React tutorials are trending globally." ChannelIQ tells you "YOUR audience watches React tutorials that are under 12 minutes and follow the 'Stop doing X' pattern 3x more than any other format."

**Core value prop in one sentence:** ChannelIQ turns your past YouTube analytics into a repeatable winning formula for your next video.

**Concerns:**
1. **The cold-start problem:** Creators with < 25 videos don't have enough data for meaningful pattern analysis. We need to handle this gracefully (message: "You need at least 25 videos for analysis. You currently have 14.") Minimum 25 videos as hard requirement.
2. **"The ideas felt generic" risk:** If Claude doesn't have enough signal, generated ideas will feel like any AI idea generator. We must display the DATA behind each idea ("based on your 8 top videos that follow this pattern") so users feel the recommendations are earned, not made up.
3. **Free tier conversion path:** The jump from Free (25 videos, 5 ideas) to Pro (500 videos, 10 ideas, PDF) must feel immediately obvious — show a "Pro unlock" card in the results: "Your analysis covered 25 videos. Your full channel has 247 videos. Unlock the full picture."

**Revision:** Add channel health score (0-100) as a hero metric on the dashboard. Makes the product feel more actionable at a glance.

---

## QA / Testing Perspective

**Most likely things to break:**

1. **YouTube API quota exhaustion.** YouTube Data API v3 has a daily quota of 10,000 units per project (shared across ALL users). A single channel analysis can consume ~1,000 units for a 100-video channel (50 per page = 2 list calls, then metrics). At 10 concurrent analyses this maxes out fast.
   - **Fix:** Apply for higher quota on launch (takes ~1 week, must request it). Meanwhile, limit concurrent analyses per API key. Rate limit analysis requests (max 2 per user per day).

2. **OAuth token expiry during long analysis.** Access tokens expire in 1 hour. A large channel (500 videos) could take 10+ minutes to fully process. Need to proactively refresh tokens before each batch.

3. **Creator has private videos.** YouTube API won't return analytics for private or scheduled videos. Handle gracefully — skip them, note "X videos skipped (private)."

4. **Creator has < 25 videos.** Block analysis, show friendly message.

5. **New channel (0 views on most videos).** Patterns analysis with sparse data will be useless. Enforce minimum: at least 25 videos with > 100 views each.

6. **Stripe webhook duplicate events.** Stripe can send the same webhook twice. All webhook handlers must be idempotent (check if event already processed).

7. **Claude structured output sometimes invalid JSON.** Use `response_format: { type: "json_object" }` with Claude. Parse with try/catch. If invalid JSON, retry once, then mark analysis as failed.

**Key edge cases:**
- Channel with 1 video type only (e.g., all live streams) — AI can't find patterns
- Channel in a non-English language — ensure prompts say to match output language to channel language
- Creator deletes videos between analyses — handle graceful 404s
- Duplicate video IDs in paginated results — use UPSERT not INSERT

---

## Engineering Perspective

**Is this the simplest way to build it?**

The stack is appropriate. A few complexity concerns:

1. **BullMQ + Redis is right** — the analysis job can take 2-10 minutes, so async queue is necessary. Don't try to do this in a single HTTP request (timeout risk). Use polling endpoint `/channels/:id/poll` for frontend status updates.

2. **Token encryption** — AES-256-GCM is fine. Use `crypto` Node built-in. Key in env var (ENCRYPTION_KEY=32 bytes). Don't use bcrypt for tokens (that's for passwords).

3. **YouTube Analytics API vs. YouTube Data API** — These are TWO different APIs. YouTube Data API gives video metadata. YouTube Analytics API gives the actual performance metrics (views, watch time, CTR, etc.). Both require the same OAuth scope. Plan correctly accounts for this.

4. **Claude prompt efficiency** — Don't send all 500 video transcripts to Claude. Send: title, duration, published_at, views, avg_view_duration, ctr, tags for each video. That's ~500 tokens per 100 videos. Keep prompt under 100K tokens.

5. **Avoid N+1 on video metrics** — Fetch metrics in batches. YouTube Analytics API supports multi-video queries. Build a `fetchBatchMetrics(videoIds[])` helper.

6. **Consider caching** — analysis_results can be cached for 7 days. No need to re-analyze if nothing significant has changed (< 10 new videos). Add a "force re-analyze" option.

**Unnecessary complexity to cut:**
- Don't build a custom analytics dashboard in v1. Link to YouTube Studio for raw data.
- Don't build PDF server-side in v1. Use react-pdf or browser print API with a clean print stylesheet.
- Don't build multi-channel support until Agency plan actually sells.

---

## Design Perspective

**First-time user experience (5-second test):**

User lands on page. What do they see and do in 5 seconds?

Current plan's landing page isn't defined yet. For the 5-second test, the landing page MUST answer:
- "What does this do?" → "ChannelIQ tells you exactly what YouTube video to make next."
- "How?" → "Connect your YouTube channel. AI analyzes your top-performing videos. You get a proven content formula + 10 ready-to-film ideas."
- "Should I try it?" → Free. No credit card. 1-minute setup.

The dashboard "aha moment" is the idea cards showing up. We need to make sure first-time users reach that moment within 2 minutes of landing.

**Concerns:**
1. **The results page could feel overwhelming.** Pattern analysis + video list + ideas + export + performance score = too much at once. Solution: Design a "tabs" navigation — (1) Your Formula, (2) Video Ideas, (3) Video Breakdown, (4) Export. Users focus on one thing at a time.

2. **The "why this works" explanation under each idea is critical.** Don't just say "Build this app in React" — show the data: "This matches your top 8 videos (avg 45K views). Your audience has responded to this format 3x more than any other."

3. **Progress screen matters more than expected.** A 45-60 second wait is long. Show real progress: "Fetching video titles (1/247)..." then "Analyzing patterns..." with a meaningful animation, not a generic spinner. This is where trust is built.

**Revision:** Add a "Channel Health Score" (0-100) at the top of the dashboard as a hero number. It's a composite score based on consistency, growth trend, and performance vs. benchmark. This gives users an instant orientation point.

---

## Security Perspective

**Attack surface:**

1. **OAuth tokens** — Highest risk item. If tokens leak, attacker can read all channel analytics. Store AES-256-GCM encrypted in Postgres. Key must never be in code.

2. **SSRF risk** — We fetch YouTube API on behalf of users. No user-supplied URLs are fetched directly. YouTube API calls go to `googleapis.com` — safe.

3. **YouTube API key exposure** — API key for YouTube Data API must be in backend env vars only. Never in frontend.

4. **Stripe webhook signature** — MUST verify `stripe-signature` header on every webhook. Never trust body without signature check.

5. **Supabase RLS** — Every table must have Row Level Security enabled. Users must not be able to query other users' channels or analysis results.

6. **Rate limiting:**
   - `/channels/:id/analyze` — max 2 per user per day (quota protection)
   - `/channels/:id/ideas` — max 5 regenerations per day on free, unlimited on pro
   - `/auth/*` — standard rate limiting against brute force

7. **No PII in BullMQ jobs** — Job payloads should contain only channel_id (UUID), not tokens or email addresses.

8. **Claude API key** — Backend env var only. Never shipped to frontend.

---

## Revised Plan Summary

**Additions based on review:**
1. Minimum 25 videos (with > 100 views) hard requirement before analysis
2. "Full channel unlock" upsell card in free analysis results
3. "Channel Health Score" hero metric on dashboard
4. Tab-based navigation for results (Formula / Ideas / Videos / Export)
5. Progress screen shows real-time per-video progress, not generic spinner
6. Each idea card must show the data behind the recommendation
7. Token refresh proactively before each API batch
8. All webhook handlers must be idempotent

**Cuts:**
1. Multi-channel (Agency) support pushed to v2
2. PDF generation simplified to browser print + clean CSS
3. Comment analysis pushed to v2
