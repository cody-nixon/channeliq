# ChannelIQ — Technical Plan

## Product

**Name:** ChannelIQ  
**Tagline:** "Know exactly what video to make next."  
**One-liner:** AI reads your YouTube history and surfaces the patterns your best videos share — so you can make more of what works.

---

## User Stories (Only What Matters)

1. **Connect:** As a creator, I can sign in with my Google/YouTube account and grant ChannelIQ access to my channel analytics.
2. **Analyze:** As a creator, I can trigger a full channel analysis that processes my last 50–500 videos.
3. **Patterns:** As a creator, I can see which video attributes (topic cluster, length, format, title structure, day posted) correlate with high performance on MY channel specifically.
4. **Ideas:** As a creator, I can get 10 video ideas generated that follow my channel's proven winning formula.
5. **Deep Dive:** As a creator, I can click any video in my history and see exactly why it over/under-performed relative to my channel average.
6. **Export:** As a creator, I can export my channel strategy report as PDF or share a read-only link.
7. **Alerts:** As a Pro creator, I can get a weekly email with my channel's performance delta and 3 new video ideas.

---

## Tech Stack

| Layer | Choice | Why |
|-------|--------|-----|
| Frontend | React + Vite + TypeScript | Standard per LESSONS.md |
| UI | shadcn/ui + Tailwind | Beautiful defaults, per LESSONS.md |
| Backend | Node.js/Express + TypeScript | Standard stack |
| Database | PostgreSQL via Supabase | Auth + DB + Edge Functions |
| Auth | Google OAuth (YouTube scope) | YouTube requires Google OAuth; same flow handles app auth |
| YouTube API | YouTube Data API v3 + YouTube Analytics API | Official, reliable, per-user quota |
| AI | Anthropic Claude claude-sonnet-4-6 (structured JSON) | Pattern analysis + idea generation |
| Queue | BullMQ + Redis | Async video analysis (takes 30s-2min per channel) |
| Email | Resend | Weekly digest |
| Payments | Stripe | Subscription billing |
| Hosting | Railway (backend) + Vercel (frontend) | Simple, fast |

---

## Architecture Overview

```
User Browser
    │
    ▼
React SPA (Vite)
    │  REST API
    ▼
Express API (Node.js)
    ├── /auth → Google OAuth (YouTube Analytics scope)
    ├── /channels → Trigger analysis, get status
    ├── /analysis → Get results, patterns, ideas
    ├── /videos → Per-video insights
    ├── /export → PDF report generation
    └── /billing → Stripe checkout, webhooks
    │
    ├── YouTube API Client
    │       ├── List videos (metadata, thumbnails, titles)
    │       └── Analytics API (views, watch time, CTR, AVD per video)
    │
    ├── BullMQ Queue (Redis)
    │       └── analysis:channel job
    │               1. Fetch all video metadata (batch, 50 at a time)
    │               2. Fetch analytics for each video
    │               3. Call Claude: analyze patterns (structured JSON output)
    │               4. Call Claude: generate 10 video ideas
    │               5. Store results in Postgres
    │
    └── PostgreSQL (Supabase)
            ├── users (id, google_id, email, plan)
            ├── channels (id, user_id, youtube_channel_id, name, thumbnail_url)
            ├── analysis_runs (id, channel_id, status, started_at, completed_at)
            ├── videos (id, channel_id, youtube_video_id, title, published_at, duration_seconds, metadata jsonb)
            ├── video_metrics (id, video_id, views, watch_time_minutes, avg_view_duration, ctr, subscribers_gained)
            └── analysis_results (id, analysis_run_id, patterns jsonb, ideas jsonb, summary text)
```

---

## API Design

### Auth
```
GET  /auth/google          → Redirect to Google OAuth
GET  /auth/google/callback → Handle callback, create session
POST /auth/logout          → Clear session
GET  /auth/me              → { user: { id, email, plan } }
```

### Channel
```
GET  /channels             → List connected channels
POST /channels/connect     → Connect YouTube channel (exchange OAuth tokens)
GET  /channels/:id/status  → { status: 'idle' | 'analyzing' | 'ready', lastAnalyzed: Date }
POST /channels/:id/analyze → Trigger analysis run (returns jobId)
GET  /channels/:id/poll    → { jobId, status, progress: 0-100, eta: seconds }
```

### Analysis
```
GET  /channels/:id/analysis        → Full analysis results (patterns, insights, ideas)
GET  /channels/:id/analysis/videos → Paginated video list with performance scores
GET  /channels/:id/analysis/videos/:videoId → Deep-dive for one video
POST /channels/:id/analysis/export → Generate PDF, return S3/storage URL
GET  /channels/:id/ideas           → Regenerate video ideas (uses 1 credit)
```

### Billing
```
POST /billing/checkout → Create Stripe checkout session
POST /billing/portal   → Customer portal (manage subscription)
POST /billing/webhook  → Stripe webhook handler
```

---

## Data Model

### `users`
```sql
id          UUID PRIMARY KEY DEFAULT gen_random_uuid()
google_id   TEXT UNIQUE NOT NULL
email       TEXT UNIQUE NOT NULL
name        TEXT
avatar_url  TEXT
plan        TEXT DEFAULT 'free'   -- 'free' | 'pro' | 'agency'
stripe_customer_id TEXT
analyses_this_month INTEGER DEFAULT 0
created_at  TIMESTAMPTZ DEFAULT NOW()
```

### `channels`
```sql
id                  UUID PRIMARY KEY DEFAULT gen_random_uuid()
user_id             UUID REFERENCES users(id) ON DELETE CASCADE
youtube_channel_id  TEXT UNIQUE NOT NULL
name                TEXT
description         TEXT
subscriber_count    INTEGER
thumbnail_url       TEXT
access_token        TEXT   -- encrypted
refresh_token       TEXT   -- encrypted
token_expires_at    TIMESTAMPTZ
created_at          TIMESTAMPTZ DEFAULT NOW()
```

### `analysis_runs`
```sql
id          UUID PRIMARY KEY DEFAULT gen_random_uuid()
channel_id  UUID REFERENCES channels(id) ON DELETE CASCADE
status      TEXT DEFAULT 'pending'   -- 'pending' | 'fetching' | 'analyzing' | 'complete' | 'failed'
videos_total    INTEGER DEFAULT 0
videos_processed INTEGER DEFAULT 0
error_message   TEXT
started_at      TIMESTAMPTZ DEFAULT NOW()
completed_at    TIMESTAMPTZ
```

### `videos`
```sql
id                UUID PRIMARY KEY DEFAULT gen_random_uuid()
channel_id        UUID REFERENCES channels(id)
analysis_run_id   UUID REFERENCES analysis_runs(id)
youtube_video_id  TEXT NOT NULL
title             TEXT
description       TEXT
published_at      TIMESTAMPTZ
duration_seconds  INTEGER
thumbnail_url     TEXT
tags              TEXT[]
category_id       INTEGER
UNIQUE(channel_id, youtube_video_id)
```

### `video_metrics`
```sql
id                     UUID PRIMARY KEY
video_id               UUID REFERENCES videos(id)
views                  INTEGER
watch_time_minutes     NUMERIC
avg_view_duration_secs NUMERIC
click_through_rate     NUMERIC          -- percentage
subscribers_gained     INTEGER
likes                  INTEGER
comments               INTEGER
shares                 INTEGER
impressions            INTEGER
fetched_at             TIMESTAMPTZ DEFAULT NOW()
```

### `analysis_results`
```sql
id              UUID PRIMARY KEY DEFAULT gen_random_uuid()
analysis_run_id UUID REFERENCES analysis_runs(id) UNIQUE
channel_id      UUID REFERENCES channels(id)

-- Structured AI output (Claude structured JSON)
patterns        JSONB NOT NULL   -- see schema below
ideas           JSONB NOT NULL   -- array of video ideas
summary         TEXT             -- executive summary paragraph
performance_benchmark JSONB      -- channel averages (views, CTR, AVD)
top_videos      JSONB            -- top 10 video IDs with notes
bottom_videos   JSONB            -- bottom 10 video IDs with notes

created_at      TIMESTAMPTZ DEFAULT NOW()
```

#### `patterns` JSONB schema (Claude output):
```json
{
  "winning_formula": {
    "optimal_length_range": { "min_minutes": 8, "max_minutes": 14 },
    "best_performing_topics": ["React hooks", "Next.js tutorials", "TypeScript basics"],
    "underperforming_topics": ["DevOps", "Career advice"],
    "title_patterns": {
      "high_performers": ["How to X (in Y minutes)", "X things about Y", "Stop doing X, do Z instead"],
      "high_ctr_words": ["Stop", "Actually", "Nobody tells you", "I was wrong about"],
      "avoid": ["Vlog", "Update", "My thoughts on"]
    },
    "format_analysis": {
      "tutorial_performance": { "avg_views": 45000, "avg_ctr": 6.2 },
      "opinion_performance": { "avg_views": 12000, "avg_ctr": 4.1 },
      "project_build_performance": { "avg_views": 38000, "avg_ctr": 5.8 }
    },
    "posting_day_analysis": {
      "best_days": ["Tuesday", "Thursday"],
      "worst_days": ["Sunday", "Monday"]
    },
    "thumbnail_notes": "Text overlay on left, face on right, high contrast colors perform 34% better on your channel",
    "key_insight": "Your audience is intermediate React developers who respond strongly to practical, project-based tutorials under 12 minutes that solve a specific problem they'll face at work."
  }
}
```

#### `ideas` JSONB schema:
```json
[
  {
    "rank": 1,
    "title": "Stop Using useState for This — useReducer Explained in 10 Minutes",
    "why_it_will_work": "Follows your best-performing 'Stop doing X' title pattern. Practical React topic. Under 12 min. Your tutorials on state management average 52K views.",
    "estimated_views": "40K-60K",
    "format": "tutorial",
    "estimated_length_minutes": 10,
    "hooks": ["The mistake everyone makes", "Live code demo"],
    "thumbnail_suggestion": "Split screen: messy useState code vs clean useReducer. Text overlay on left."
  }
]
```

---

## Auth Flow

1. User clicks "Connect YouTube Channel"
2. Redirect to Google OAuth with scopes:
   - `https://www.googleapis.com/auth/youtube.readonly`
   - `https://www.googleapis.com/auth/yt-analytics.readonly`
3. User grants permission
4. Callback: exchange code for access + refresh tokens
5. Tokens encrypted and stored in `channels` table
6. User is now signed in and channel is connected (same OAuth flow handles both)

Token refresh: before any API call, check `token_expires_at`. If expired, use refresh_token to get new access_token.

---

## Analysis Job (BullMQ)

```
analyzeChannel(channelId) {
  1. UPDATE analysis_run status='fetching'
  2. Fetch channel video list via YouTube API (50 per page, paginate all)
  3. For each video, upsert into videos table
  4. For each video, fetch analytics (views, watch_time, avg_view_duration, ctr)
     → batch in groups of 10 to avoid quota exhaustion
  5. UPDATE analysis_run status='analyzing', videos_processed=N
  6. Build analysis payload: top/bottom 25 videos by views, full metrics
  7. Call Claude claude-sonnet-4-6:
     SYSTEM: "You are a YouTube channel strategy analyst..."
     INPUT: JSON with all video titles, lengths, publish dates, topics, metrics
     OUTPUT: structured patterns JSON (response_format JSON mode)
  8. Call Claude again:
     INPUT: patterns + channel niche + "generate 10 video ideas"
     OUTPUT: structured ideas JSON array
  9. Insert analysis_results
  10. UPDATE analysis_run status='complete'
  11. Send email notification via Resend
}
```

---

## Pricing

| Plan | Price | Features |
|------|-------|----------|
| Free | $0 | 1 analysis/month, last 25 videos, 5 video ideas, no export |
| Pro | $19/month | Unlimited analyses, last 500 videos, 10 video ideas, PDF export, weekly email digest |
| Agency | $49/month | Up to 5 channels, team sharing, white-label export |

Free tier is generous enough to get hooked. Pro is the clear upgrade for serious creators.

---

## What We Are NOT Building

- **Video SEO / keyword research** — that's vidIQ's job
- **Competitor analysis** — not in v1
- **Thumbnail A/B testing** — TubeBuddy does this
- **Shorts/Reels analytics** — YouTube Shorts have different mechanics; v2
- **Multi-platform** (TikTok, Instagram) — v3
- **Direct posting/scheduling** — out of scope
- **Comment analysis** — v2
- **Real-time alerts** — v2

---

## UI Flow

### New User
1. Landing page → "Connect Your Channel" CTA
2. Google OAuth → grant YouTube read permissions
3. Channel dashboard loads (skeleton state)
4. Auto-trigger first analysis
5. Progress screen: "Analyzing 247 videos... this takes about 45 seconds"
6. Dashboard populates with patterns + ideas

### Returning User
1. Login → Dashboard
2. See "Last analyzed: 3 days ago" with "Re-analyze" button
3. Browse patterns, video list, ideas
4. Click "Regenerate Ideas" to get fresh ideas (uses 1 credit)
5. Export to PDF for team/planning

---

## Security Notes

- YouTube OAuth tokens stored encrypted (AES-256) in Postgres
- Never log tokens
- Rate limiting on all API endpoints (express-rate-limit)
- Stripe webhook signature verification
- CORS restricted to app domain
- No storing video content — only metadata + metrics
- Supabase Row Level Security: users can only query their own channels/data
