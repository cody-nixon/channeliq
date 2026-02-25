# Daily Design Report — ChannelIQ

**Date:** February 25, 2026  
**Product:** ChannelIQ  
**GitHub:** https://github.com/cody-nixon/channeliq

---

## The Problem

YouTube creators at the mid-tier level (50K–500K subscribers) spend 20+ hours a week making content but often can't explain why some videos succeed dramatically while others flop. They rely on vidIQ and TubeBuddy for keyword/SEO guidance, but those tools tell you what's trending GLOBALLY — not what has worked for YOUR specific audience.

**Source signals:**
- Zero search results for "YouTube channel analytics content strategy tool 'what videos to make' historical performance 2026" — indicating a genuine gap
- Zero results on TubeBuddy for "channel-specific pattern recommendation" feature
- vidIQ's "historical data analysis" feature exists but is a raw analytics view — no AI interpretation
- Twitter search for creator frustrations confirmed guesswork about "what to post next" is a persistent pain
- HN creator tool discussions show appetite for data-driven channel strategy

---

## Reasoning For Choosing This

**Why not other candidates:**
- **AI API cost monitoring** (strong contender) → nOps, Cloudidr, LangSmith already crowd this space
- **GLP-1 nutrition tool** → Crowded mobile app market (Pep, Shotsy, Dose AI already live)
- **Price gouging detector** → No reliable real-time pricing data API exists, fragile product
- **Learning comprehension checker** → Valid but lower willingness to pay (students vs. creators)
- **Password manager migration** → One-time utility, not recurring

**Why ChannelIQ:**
1. Existing tools (vidIQ/TubeBuddy) solve a DIFFERENT problem (SEO, not strategy)
2. High willingness to pay — creators treat channel growth as career investment
3. Strong recurring product (you check weekly as you post)
4. AI makes this possible in 2026 — 6 months ago it required a data analyst
5. YouTube Analytics API makes the data accessible without scraping
6. Zero overlap with any past build

---

## Design Summary

**Core flow:**
1. Creator connects YouTube channel via Google OAuth (read-only scopes)
2. BullMQ worker fetches all video metadata + performance metrics via YouTube APIs
3. Claude claude-sonnet-4-6 analyzes patterns: optimal length, best topics, winning title structures, best posting days, thumbnail patterns
4. Claude generates 10 video ideas that match the channel's proven formula, each with a data citation
5. Results displayed in a 4-tab dashboard: Your Formula / Video Ideas / Video Breakdown / Export

**Pricing:**
- Free: 1 analysis/month, last 25 videos, 5 ideas
- Pro ($19/month): Unlimited analyses, 500 videos, 10 ideas, PDF export, weekly email
- Agency ($49/month): 5 channels, team sharing, white-label export

**Key technical risks managed:**
- YouTube API quota exhaustion → request higher quota, rate-limit analyses
- Google OAuth verification for sensitive scopes → apply early, use minimal read-only scopes
- Analysis quality → always show data citations; test with 10+ real channels in beta

**Estimated build:** ~96 hours for experienced full-stack developer (~12 working days)

---

## Key Insight That Makes This Different

Every YouTube analytics tool available today is **outside-in**: "Here's what's working across YouTube right now. Here's what's trending globally."

ChannelIQ is **inside-out**: "Here's what's working for YOUR channel specifically, based on what your audience has already proven they love."

Mid-tier creators have moved past needing global keyword data. They need a content strategist who has read all their past videos. ChannelIQ is that strategist, automated.

---

## Files in this Repo

- `RESEARCH.md` — 15+ candidate problems with sources, scoring, cross-reference
- `DECISION.md` — Scoring matrix, deep reasoning for final choice
- `PLAN.md` — Full technical architecture, API design, data model, auth flow
- `REVIEW.md` — Multi-role review: Product / QA / Engineering / Design / Security
- `WIREFRAMES.md` — Text wireframes for all 10 screens with full copy
- `DESIGN.md` — Comprehensive design document, implementation roadmap
- `REPORT.md` — This file
