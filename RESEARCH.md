# Research — Daily Problem Hunt (Feb 25, 2026)

## Sources Searched

- Twitter/X (bird CLI): "why is there no app", "someone should build", "wish there was a tool", "frustrated with my workflow", "startup idea SaaS problem", "API costs too high"
- Reddit: r/SomebodyMakeThis (new posts)
- Hacker News: Ask HN front page
- Web searches: GLP-1 nutrition apps, AI API cost monitoring tools, YouTube analytics tools, TubeBuddy vs vidIQ comparison

---

## Candidate Problems (15+ collected)

### C1: Freelancer Finance + Project Tracker
**Source:** r/SomebodyMakeThis — "I need an app to keep my finances and projects at the same place" (5 votes)
**Problem:** Freelancers with irregular income can't see if they'll have enough money next month. They bounce between project mgmt, invoicing, and banking apps.
**Who:** 65M+ US freelancers
**Existing:** Wave, Bonsai, FreshBooks, QuickBooks
**Gap:** All too heavy / accounting-focused, not "am I okay this month?" focused
**Urgency:** Medium — chronic pain
**Verdict:** Crowded space. Skip.

---

### C2: Inflation / Price Gouging Detector
**Source:** r/SomebodyMakeThis — "Tool that shows inflation price gouging" (2 votes)
**Problem:** Show which grocery/retail items are priced more than inflation would justify.
**Who:** General consumers, ~100M US shoppers
**Existing:** Nothing quite like this
**Gap:** Real-time price database + CPI comparison isn't done
**Urgency:** HIGH — tariffs + inflation hitting consumers hard right now
**Verdict:** Interesting but requires real-time pricing data (no public API), highly dependent on data sourcing. Fragile.

---

### C3: GLP-1 / Ozempic Nutrition Optimizer
**Source:** Women's Health Magazine (5 days ago), glp1newsroom.com (4 days ago), ACLM/ASN joint advisory
**Problem:** GLP-1 users eat only 600-900 cal/day. Without guidance, they lose muscle (up to 40% of weight lost is muscle). Nutrition planning for low-appetite states is hard.
**Who:** 9M+ US GLP-1 prescriptions in 2025, growing rapidly
**Existing:** Pep, Shotsy, GLPeak, Dose AI — all injection trackers, basic nutrition logs
**Gap:** None specifically designed for "maximize nutrition density in 800 calories"
**Urgency:** VERY HIGH — major medical advisory published 5 days ago
**Verdict:** Crowded mobile app space (Pep, Shotsy). Web app less natural fit.

---

### C4: Learning Comprehension Checker (QuizDrop)
**Source:** r/SomebodyMakeThis — "A tool that assess you knowledge based on what you learn" (5 votes, 3 comments)
**Problem:** You watch a tutorial and feel like you learned it. Next day: blank. Generic quizzes don't test the SPECIFIC content you consumed.
**Who:** Students, self-learners, developers (~200M online learners)
**Existing:** Kahoot (teacher-driven), Quizlet (flashcards), CourseHero (textbooks). Nothing for "quiz me on THIS YouTube video."
**Gap:** YouTube URL → AI transcripts → specific comprehension quiz
**Urgency:** Medium — ongoing education pain
**Verdict:** Good but Medium urgency. Bookmark-worthy. Feasible. ✓ Keep.

---

### C5: AI API Cost Monitoring (Cross-Provider)
**Source:** HN (trending "1Password pricing up 33%"), Twitter developer complaints
**Problem:** Developers running AI agents see API bills exploding with no visibility by task/agent.
**Who:** AI developers, teams running agents
**Existing:** nOps, Cloudidr, LangSmith — CROWDED
**Gap:** None — market already served
**Verdict:** Too crowded. Skip.

---

### C6: Password Manager Migration Wizard
**Source:** HN — "1Password pricing increasing up to 33% in March" (117 points, 170 comments)
**Problem:** People want to leave 1Password but migration to alternatives is painful/confusing.
**Who:** ~15M 1Password users, 40M Lastpass refugees
**Existing:** Nothing for migration specifically
**Gap:** Real but one-time use, not a recurring product
**Urgency:** HIGH right now (price increase announcement)
**Verdict:** One-time utility, not a bookmarkable recurring product. Skip.

---

### C7: Skill Tree Builder for Real-Life Skills
**Source:** r/SomebodyMakeThis — posted 32 min ago at time of research
**Problem:** No community resource for "what are all the steps to learn X skill in order?"
**Who:** Self-learners, career changers
**Existing:** roadmap.sh (developer-focused)
**Gap:** Community-created, gamified skill trees for ANY domain
**Urgency:** Low — existing workarounds (YouTube, Reddit, roadmap.sh)
**Verdict:** Long-term community building play, not a quick-win product. Skip.

---

### C8: YouTube Channel Strategy Engine (ChannelIQ) ← WINNER
**Source:** Zero results for targeted search (gap confirmed); HN discussion on creator tools; Twitter creator frustration
**Problem:** YouTube creators know vidIQ/TubeBuddy for SEO (what keywords/trends are hot GLOBALLY) but nobody analyzes YOUR OWN channel's historical data to find the patterns that made YOUR top videos succeed and tell you what to make next.
**Who:** 15M+ active YouTube creators, growing fast
**Existing:** vidIQ (keyword/SEO), TubeBuddy (A/B testing, keyword), YouTube Studio (raw data, no AI insights)
**Gap:** "Channel-specific pattern analysis that says: YOUR audience watches X type of video for Y minutes, YOUR top videos follow Z pattern, HERE are 10 ideas that match your proven formula"
**Urgency:** HIGH — YouTube algorithm is constantly changing, creators desperate for signal
**Verdict:** ✓ CHOSEN. See DECISION.md for full reasoning.

---

### C9: Personal Knowledge Resurfacer (Resurface)
**Source:** HN — "Ask HN: Where do you save links, notes and random useful stuff?" (17 points, 39 comments)
**Problem:** Everyone has a link graveyard (Pocket, Raindrop, bookmarks). You save things but never rediscover them when relevant.
**Who:** Knowledge workers, researchers, developers
**Existing:** Readwise (Kindle highlights), Pocket (saves but no resurfacing), Raindrop.io
**Gap:** Semantic resurfacing of YOUR saves based on current focus
**Urgency:** Medium — chronic frustration
**Verdict:** Strong candidate but less monetizable than ChannelIQ. Runner-up.

---

### C10: Contract Risk Scanner for SMBs
**Source:** Multiple sources; perennial freelancer complaint
**Problem:** Small businesses and freelancers get contracts they can't afford to have a lawyer review.
**Who:** 33M US small businesses
**Existing:** Spellbook, LegalRobot, Ironclad, ContractPodAi
**Gap:** None — crowded enterprise AI legal space
**Verdict:** Very crowded. Skip.

---

### C11: Weekly Personal Productivity Retrospective (RetroMatch)
**Source:** Twitter — "I'm frustrated with how long I take to work through my queue"
**Problem:** Knowledge workers plan but never retrospect. Patterns of what works for them never emerge.
**Who:** Professionals, developers, indie hackers
**Existing:** Notion, Obsidian, journals — all input-only, no pattern analysis
**Gap:** AI-analyzed structured retrospectives with emerging pattern insights
**Urgency:** Low — "should" not a crisis
**Verdict:** Low urgency. Runner-up.

---

### C12: ProvenBy — Proof of Human-Created Work
**Source:** Twitter/X ongoing AI content fraud discussion
**Problem:** With AI-generated content flooding jobs/schools, humans need a way to PROVE their work is genuinely theirs (timestamped creation history, version trail).
**Who:** Students, freelancers, journalists, writers
**Existing:** Turnitin (detects AI), Copyleaks (detects AI) — but neither PROVES authenticity
**Gap:** Creating "Certificate of Authentic Creation" from your working history
**Urgency:** HIGH — AI fraud is exploding
**Verdict:** Interesting but requires editor integration to be meaningful (hard as web app alone). Skip.

---

### C13: Habit Breaking with 66-Day Mountain
**Source:** r/SomebodyMakeThis — "An app like this — bad habit mountain" (17 votes, 26 comments!)
**Problem:** Habit breaking apps are about building habits, not breaking bad ones. 66-day commitment with visual mountain metaphor.
**Who:** Everyone trying to break habits
**Existing:** Streaks, Habitica, Done
**Gap:** Specifically "break bad habit" framing with relapse penalties
**Urgency:** Low-medium
**Verdict:** Crowded habit app space, low monetization. Skip.

---

### C14: "ChannelIQ for TikTok/Reels"
Same concept as C8 applied to TikTok. Less API access. Harder. YouTube is the right starting point.

---

### C15: RepoReel — GitHub to Social Content
**Source:** Twitter — "building in public" trend
**Problem:** Developers ship things but don't know how to turn GitHub milestones into compelling LinkedIn/Twitter content
**Who:** Indie hackers, developer advocates, 30M GitHub users
**Existing:** Copy.ai, Jasper (generic AI writers)
**Gap:** "Convert MY GitHub activity into developer content with my voice"
**Urgency:** Medium
**Verdict:** Interesting but a niche within a niche. ChannelIQ is bigger.

---

## Cross-Reference with PAST_BUILDS.md

None of the above concepts overlap with past builds:
- PromptShot: LLM comparison ❌ (not this)
- GitScout: GitHub issue finder ❌ (not this)  
- NameDrill: Flashcards/face learning ❌ (not this)
- FlowPrompt: Teleprompter ❌ (not this)
- QuoteCheck: Quote verification ❌ (not this)
- PriceShift: SaaS price tracking ❌ (not this)
- PolicyPulse: ToS monitoring ❌ (not this)
- CancelScore: Subscription cancellation ❌ (not this)
- VibeAudit: Security scanner ❌ (not this)
- CodeBrief: GitHub codebase summaries ❌ (not this)
- DealStack: Contractor quote comparison ❌ (not this)
- TariffScope: Import tariff calculator ❌ (not this)
- VibeMatch: Game discovery ❌ (not this)

**ChannelIQ is genuinely novel. Proceed.**

---

## Winner

**C8: ChannelIQ — YouTube Channel Strategy Engine**

The problem is real, the existing solutions (vidIQ/TubeBuddy) solve a DIFFERENT problem (SEO, keywords, competitor analysis), there's no tool doing channel-specific pattern analysis, the audience has high willingness to pay, and the product is highly recurring (you check weekly).
