# ChannelIQ — Final Design Document

---

## Executive Summary

YouTube creators spend 20+ hours a week making videos and often don't know why some succeed while others flop. Existing tools (vidIQ, TubeBuddy) help with SEO and keyword research — what's trending globally — but nobody analyzes a creator's OWN channel history to find the specific formula that makes THEIR audience click and watch.

**ChannelIQ** connects to a creator's YouTube account, pulls their full video analytics history, runs AI pattern analysis, and surfaces a plain-English "winning formula" — the exact attributes (topic, length, title structure, posting day) that define their best-performing videos — plus 10 ready-to-film video ideas that match that formula.

---

## Target User Persona

**"Mid-Tier Maria"**
- YouTube channel: 50K–500K subscribers
- Full or part-time income from YouTube
- Posts 1–2x per week
- Has been using YouTube 2–4 years
- Currently uses vidIQ or TubeBuddy for SEO/keywords
- Pain: "I know what tags to use, but I still feel like I'm guessing what content to make."
- Goal: Grow from 200K to 500K subscribers in 12 months
- Fear: Algorithm changes. Spending 15 hours on a video that gets 3K views.
- Willingness to pay: $15–30/month for something that meaningfully helps channel growth

**Secondary: "Agency Anna"**
- Manages 3–5 YouTube channels for clients
- Needs to deliver content strategy reports to clients monthly
- Currently does this manually by eyeballing YouTube Studio
- Would pay $49/month for an automated report they can export and reskin

---

## Complete Technical Specification

### Stack
- **Frontend:** React 18 + Vite 5 + TypeScript, shadcn/ui, Tailwind CSS
- **Backend:** Node.js 22 + Express 5 + TypeScript
- **Database:** PostgreSQL via Supabase (with RLS, pgvector for future use)
- **Auth:** Google OAuth 2.0 with YouTube scopes (read-only)
- **YouTube API:** Data API v3 (metadata) + Analytics API (metrics)
- **AI:** Anthropic Claude claude-sonnet-4-6 with structured JSON output (response_format)
- **Queue:** BullMQ + Redis (analysis jobs, async email)
- **Email:** Resend (transactional + weekly digest)
- **Payments:** Stripe (Subscription products: Free/Pro/Agency)
- **Hosting:** Vercel (frontend) + Railway (backend + Redis)

### Key API Endpoints
```
Auth:    GET /auth/google, GET /auth/google/callback, GET /auth/me
Channel: GET /channels, POST /channels/connect, GET /channels/:id/status
Analysis: POST /channels/:id/analyze, GET /channels/:id/poll (polling)
Results: GET /channels/:id/analysis, GET /channels/:id/analysis/videos
         GET /channels/:id/analysis/videos/:videoId, POST /channels/:id/ideas
Export:  POST /channels/:id/export (generate PDF), GET /channels/:id/share
Billing: POST /billing/checkout, POST /billing/portal, POST /billing/webhook
```

### Data Model (6 tables)
1. `users` — id, google_id, email, plan, stripe_customer_id
2. `channels` — id, user_id, youtube_channel_id, access_token (encrypted), refresh_token (encrypted)
3. `analysis_runs` — id, channel_id, status, videos_total, videos_processed
4. `videos` — id, channel_id, youtube_video_id, title, duration_seconds, published_at, tags[]
5. `video_metrics` — id, video_id, views, watch_time_minutes, avg_view_duration, ctr
6. `analysis_results` — id, analysis_run_id, patterns jsonb, ideas jsonb, summary text

### AI Prompt Architecture
**Call 1 (Pattern Analysis):**
- Input: ~50 top videos + 50 bottom videos + their metrics (compact JSON, ~20K tokens)
- System: "You are a YouTube channel analyst. Find patterns that distinguish high-performing from low-performing videos on this SPECIFIC channel."
- Output (structured JSON): optimal_length_range, best_topics, worst_topics, title_patterns, format_analysis, posting_day_analysis, thumbnail_notes, key_insight, benchmark_metrics
- Model: claude-sonnet-4-6 (quality matters here)
- Estimated cost: ~$0.15 per channel analysis

**Call 2 (Idea Generation):**
- Input: patterns JSON + channel niche
- System: "Generate 10 specific video ideas that match this channel's proven winning formula."
- Output (structured JSON): array of {title, why_it_will_work, estimated_views, format, estimated_length_minutes, hooks, thumbnail_suggestion}
- Model: claude-sonnet-4-6
- Estimated cost: ~$0.05 per generation

**Total cost per analysis run: ~$0.20 → well within Pro's $19/month margin**

---

## Wireframes

See WIREFRAMES.md for complete ASCII wireframes for all 10 screens:
1. Landing page
2. Auth / Google OAuth
3. Analysis progress screen
4. Dashboard: Your Formula tab
5. Dashboard: Video Ideas tab
6. Dashboard: Video Breakdown tab
7. Dashboard: Export tab
8. Settings / Account page
9. Error states (insufficient data, analysis failed, token expired)
10. Empty state (first-time user)

---

## Implementation Roadmap

### Sprint 1 — Foundation (Days 1–3) — 20 hrs
- Google OAuth setup (Google Cloud project, YouTube read + analytics scopes)
- Supabase setup (tables, RLS policies)
- Express API scaffold (auth routes, session management)
- React scaffold (Vite, shadcn/ui, Tailwind, React Router)
- BullMQ + Redis setup
- YouTube API client (list videos, paginate, fetch analytics)

### Sprint 2 — Core Analysis Engine (Days 4–6) — 24 hrs
- BullMQ analysis worker
- Token encryption/decryption
- YouTube batch analytics fetcher
- Claude integration (patterns analysis + ideas generation)
- analysis_results storage + retrieval
- Progress polling endpoint

### Sprint 3 — Dashboard UI (Days 7–9) — 24 hrs
- Landing page (fully designed, conversion-optimized)
- Auth flow (OAuth connect screen, channel selector)
- Analysis progress screen
- Dashboard: Your Formula tab
- Dashboard: Video Ideas tab
- Dashboard: Video Breakdown tab (paginated, search/filter)
- Video deep-dive modal

### Sprint 4 — Billing + Polish (Days 10–12) — 16 hrs
- Stripe subscription setup (Free/Pro/Agency products)
- Checkout flow + billing portal
- Webhook handler (subscription created/cancelled/updated)
- Plan gating (free tier limits enforced)
- Weekly email digest (Resend + BullMQ cron job)
- Export: Share link (public read-only view)
- Export: Print CSS for PDF (browser print)

### Sprint 5 — Testing + Launch Prep (Days 13–14) — 12 hrs
- E2E test: full user flow (connect → analyze → ideas)
- E2E test: subscription purchase + cancellation
- Error state testing
- Performance: ensure analysis completes < 2 min for 500-video channels
- Quota management (rate limiting analysis triggers)
- Production deploy (Vercel + Railway)

**Total estimated: 96 hours for an experienced full-stack developer**

---

## Estimated Effort (Experienced Dev, Solo)

| Phase | Hours |
|-------|-------|
| Google Cloud + YouTube API setup | 4 hrs |
| Database schema + Supabase RLS | 3 hrs |
| Backend API (auth, channel, analysis jobs) | 20 hrs |
| YouTube API client (pagination, analytics, token refresh) | 8 hrs |
| BullMQ worker (full analysis job) | 8 hrs |
| Claude integration (prompts, structured output, retries) | 6 hrs |
| Frontend (all screens, shadcn/ui) | 28 hrs |
| Stripe billing (checkout, webhooks, plan gating) | 10 hrs |
| Email (Resend, weekly digest) | 5 hrs |
| Export (print CSS, share links) | 4 hrs |
| Testing + polish + deploy | 12 hrs |
| **TOTAL** | **~96 hours ≈ 12 working days** |

---

## Risks and Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| YouTube API quota exhaustion (10K units/day default) | HIGH | HIGH | Apply for quota increase before launch; rate-limit analyses (2/user/day); batch requests efficiently |
| Google rejecting app during OAuth verification | MEDIUM | HIGH | Apply early; use read-only scopes only; have clean privacy policy and terms; avoid sensitive scope labels |
| Analysis results feel "generic" | MEDIUM | HIGH | Always show data citations under each insight ("based on your top 8 videos"); test with 10+ real channels during beta |
| Creators with < 25 meaningful videos | MEDIUM | MEDIUM | Hard gate at 25 videos with friendly message; no frustration in UX |
| OpenAI/Anthropic API latency | LOW | MEDIUM | 30-second Claude timeout with retry; queue user notifications on completion |
| YouTube API changes/deprecation | LOW | HIGH | Pin to specific API version; monitor Google Developer Blog |
| Competitor (vidIQ/TubeBuddy) clones feature | MEDIUM | MEDIUM | Ship fast; build community; add deeper personalization (embeddings v2) |

---

## Success Metrics

**Activation:**
- % of new users who complete first analysis: target > 70%
- Time from sign-up to "aha moment" (seeing first ideas): target < 3 minutes

**Retention:**
- Day 7 retention: target > 50% (check results once, come back after posting)
- Day 30 retention: target > 35%
- Weekly active users (trigger new analysis or regenerate ideas): target > 40% of accounts

**Revenue:**
- Free → Pro conversion: target > 8% in first 30 days
- Monthly churn (Pro): target < 5%
- MRR at 3 months: $5K (≈ 263 Pro subscribers)
- MRR at 6 months: $20K (≈ 1,050 Pro subscribers)

**Quality:**
- Average rating in creator communities: > 4.2/5
- % of users who share read-only reports: > 15%
- "Would you recommend?" NPS: target > 40

---

## The Key Insight That Makes This Different

**What vidIQ/TubeBuddy do:** "React tutorials are trending globally. Here are 50 keywords to target."

**What ChannelIQ does:** "YOUR audience has watched 8 React tutorials on YOUR channel and averaged 52K views each when they're 8–12 minutes with a 'Stop doing X' title. Here are 10 more ideas that match exactly that pattern."

The insight is about YOUR audience's proven behavior, not global trends. One is external intelligence. The other is applied intelligence from your own data.

This is why creators who are already past the SEO phase (mid-tier, 50K+ subscribers) will find this more valuable than vidIQ — they've already mastered keywords. What they need is content strategy, not discoverability tactics.

---

## Repository

https://github.com/cody-nixon/channeliq
