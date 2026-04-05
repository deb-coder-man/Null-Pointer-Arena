# Competitive Debugging Game — Product Research

## What the Product Is

A web-based 1v1 competitive game where two players race to find and fix bugs in code. Core rules:
- Error messages are shown but NOT line numbers — players must locate bugs themselves
- A description of what the code should do is provided (helps find logical errors)
- Players fix bugs until code passes all test cases — first to pass all wins
- **Power-up system**: players can pause debugging to complete mini-challenges (quiz or easy algo question) to earn power-ups that give themselves an advantage or the opponent a disadvantage
- Strategic tension: split time between debugging vs. farming power-ups

**Why:** AI is increasing the value of debugging skills (humans validate AI-generated code). No existing platform combines real-time 1v1 competition + debugging as core mechanic + strategic power-up layer.

---

## Market Validation

**Verdict: Cautiously positive.** Real white space exists. Closest competitor is CodinGame's Clash of Code (500K+ users, $3.2M revenue) — validates format but has no debugging focus and no power-up system. LeetCode (26.3M monthly visitors) has no real-time multiplayer.

**Realistic Year 3 ARR target: $3M–$8M**

**Target audiences (priority order):**
1. Job-seeking developers — interview prep motivation, largest intent, but shrinking due to AI impact on junior dev jobs
2. Competitive programmers — recreational, natural early adopters (Clash of Code players)
3. Engineering teams — B2B, highest revenue reliability, companies use Clash of Code for team events already

**Key tailwinds:**
- Gamification in education growing at 36–38% CAGR
- AI making debugging skills more valuable, not less
- Skills-based hiring growing (81% of companies adopting it)
- 47.2M developers worldwide, up 21% since 2023

**Key risks:**
- Audience passionate but narrow — B2B needed for real business
- EdTech retention can drop to 4% — gamification must be excellent
- Junior dev job market down 60% since 2022 — weakens interview-prep angle
- EdTech funding at multi-year lows if VC is part of the plan

---

## MVP Feature List

### Must ship (load-bearing walls)

- Real-time 2-player match engine with synchronised state
- Code editor with syntax highlighting, no autocomplete
- Error output panel — shows errors WITHOUT line numbers (core mechanic)
- Problem description panel (always visible, never hidden)
- Test case runner: server-side, shows exact input / expected output / actual output per test case
- Win condition: first to pass all test cases wins
- **30 problems minimum at launch** (3 difficulty tiers: Easy 5–10min, Medium 10–20min, Hard 20–30min)
- Supported languages at launch: Python, JavaScript, Java
- Matchmaking queue with single "Find Match" button
- Bot opponent fallback if no match found in 60 seconds
- ELO rating system (see below)
- Email + Google OAuth accounts
- Post-match bug breakdown — show where every bug was, time-to-first-bug for both players

### Power-ups — MVP scope (intentionally narrow)

- **Two power-ups only at launch:**
  - Defensive: reveals which test case is failing (one-time hint)
  - Offensive: delays opponent's error output by 15 seconds
- **Quiz-only mini-challenges** — skip LeetCode-style algo questions for v1 (massive scope)
- One power-up attempt per player per match
- Feature flag on the entire power-up system — if match abandonment is higher when power-ups are used, must be able to turn off fast

### Explicitly deferred (post-MVP)

- Algorithm mini-challenges as second power-up path (v2)
- Spectator mode (v2)
- Daily solo challenge (v2, high-impact retention)
- Tournament brackets (v2)
- Private rooms for teams / B2B unlock (v3)
- Mobile — desktop first, mobile = spectator only
- User-submitted problems — moderation cost too high early
- In-match chat — moderation risk, use preset reactions only

---

## Core Game Loop

1. **Landing page** — show a looping GIF/video of a real match. Single CTA: "Play Now — No Account Required"
2. **Guest match** — no account needed for first match. Explainer overlay (3 slides, skippable) plays during matchmaking wait
3. **First match** — bot opponent if no human found in 45s. Bot tuned for ~60% win rate on first match. Easy problem: 2 bugs, findable in under 2 minutes
4. **Win/loss screen** — show bug locations, ELO change, opponent's final code, immediate "Play Again" button. Registration prompt appears HERE, not before
5. **Registration hook** — "You went 1-0. Create an account to save your rating." Show what they'd lose by closing the tab

**Target match lengths:** Easy 7–12 min, Medium 12–20 min, Hard = tournament content only

---

## UI Layout (Active Match)

Three-panel desktop layout:
- **Left:** Problem description (always fully visible, no scroll required for core spec)
- **Centre:** Code editor (syntax highlighting, language selector top-right)
- **Right top:** Error output (live, exact messages, no line numbers)
- **Right bottom:** Test case dots (pass/fail per test case, live-updating)

**Always visible in HUD:**
- Player timer (large, colour-transitions green → amber → red at 25%/10% remaining)
- Opponent progress: "Opponent: X/Y test cases passing" — dots only, never their code
- Power-up inventory (slot-based, max 3 slots)
- Match timer

**Never hidden during a match:** problem description, error output, test case indicators, timers

**Mobile:** Do not build a mobile code editor. Mobile = spectator mode only.

---

## ELO System Design

### Core parameters

| Parameter | Value |
|---|---|
| Starting rating | 1200 |
| Placement matches | 5 games (rating shown with "?" suffix during placement) |
| K-factor 0–10 games | 40 (fast calibration) |
| K-factor 11–50 games | 32 |
| K-factor 51–200 games | 24 |
| K-factor 200+ games | 16 |

### Tier structure — use developer job titles

| Tier | Rating |
|---|---|
| Syntax Error | 0–999 |
| Junior Dev | 1000–1199 |
| Mid-Level | 1200–1499 |
| Senior Dev | 1500–1749 |
| Staff Engineer | 1750–1999 |
| Principal | 2000+ |

Why job titles: connects the competitive meta to the real-world context (interview prep) that makes the game appealing. "You're rated Senior Dev" lands differently than "You're Gold III" for this audience.

### Matchmaking algorithm

```
0–15s    → ±100 ELO  (ideal)
15–45s   → ±200 ELO  (acceptable)
45–90s   → ±350 ELO  (show "close match found")
90s+     → ±600 ELO  (show both ratings, let players opt in knowingly)
```

**Early days:** Do not enable ranked mode until a match can be found in 90s at 11pm on a Tuesday. Use Practice mode only until ~500 registered users. Empty queues permanently damage reputation.

### Matchmaking phases

- Phase 1 (0–500 users): Practice mode only, no ranked
- Phase 2 (500–2,000 users): Ranked enabled, single pool, 600-point cap
- Phase 3 (2,000+ users): Expanding search window as above

### Protecting casual players

- **Rating floor below 1100** — can't lose rating here, only gain (prevents discouraging spiral)
- **Demotion buffer** — one match protection at tier boundary (player cannot be demoted on their very first match after reaching a tier)
- **"Practice" queue** (not "Casual") — engineers won't self-identify as casual. Default new users here for first 3 games, then prompt ranked
- **Default new users to Practice** — opt-in to ranked, not opt-out

### Power-ups and ELO

A win is a win — don't weight ELO differently based on power-up usage. Instead track power-up usage as a parallel signal on the player profile. For forfeits: winner gains 60% of expected ELO gain (discourages rage-quit farming).

### Seasons (every 3 months)

**Soft reset, not hard reset.** Formula: `new_rating = median + (old_rating - median) * 0.7`
A 1800-rated player resets to ~1620. Hard resets break matchmaking for weeks.

One week pre-season gap — ranked queues closed, Practice open. Use for recap blog post and announcing next season's rewards.

**Seasonal milestones (participation-based, not just skill-gated):**

| Milestone | Requirement | Reward |
|---|---|---|
| Challenger | Play 1 ranked game | Season badge (permanent) |
| Competitor | Play 10 ranked games | Animated avatar border |
| Veteran | Play 25 ranked games | Unique editor colour theme |
| Elite | Reach Senior Dev tier | Gold-coloured username |
| Legend | Reach Principal tier | Animated profile card + trophy |

### No rating decay at MVP

Revisit at Season 2 only if there's empirical evidence of inflation. Decay punishes casual monthly players and adds implementation complexity. Instead: returning players after 60+ days of inactivity get a 3-game re-calibration period at K-factor 40.

---

## ELO UX Design

### Display

- **Primary:** Tier name + badge (visible everywhere)
- **Secondary:** Raw ELO number (visible in profile stats and post-match)
- Show a progress bar within the current tier (distance to next tier boundary)
- Show **expected ELO change before the match starts**: "Win: +28 / Loss: -12" — proves fairness, eliminates post-match frustration

### Opponent rating

- Show opponent's **tier badge** before the match (not exact rating)
- Reveal full rating once the match starts — seeing "340 points higher" pre-match kills motivation; mid-match it reframes as a challenge
- For large mismatches: "This match is a stretch challenge. A win here would significantly boost your rating."

### Post-match feedback

**Rating gain:** Number ticks up, green delta, brief accent pulse. Complete in under 1.5 seconds.

**Rating loss:** Show it clearly (don't minimise). Immediately follow with 2–3 specific performance points (what the player did well). Then show opponent's rating for context. Never use a negative sound effect on a rating loss.

**Post-match breakdown (biggest differentiator — nobody does this well):**
- Where every bug was and what type
- Time-to-first-bug for both players
- "Debug Accuracy" score (ratio of correct edits to total edits)
- Opponent's final code state
- Timeline of when each test case passed for each player

### Placement phase UX

- Show "Placement Match 3 of 5" progress bar, NOT a moving rating number during placement (anchoring bias)
- After each match, show a narrowing estimated range: "Calibrating... you're placing in the Senior Dev or Staff Engineer range"
- Use "calibrating" language, not "provisional" — "provisional" implies the result might not be real
- Post-placement: show percentile vs. other new players, not vs. all players

### Leaderboards

- **Default view:** Nearby ratings (5 above, 5 below current player) — most motivating for majority
- **Friends leaderboard** with head-to-head history — highest engagement for developer audience
- **Weekly ladder** (resets Sunday, minimum 5 matches) — lets casual players compete without facing all-time grinders
- **Global all-time** — exists but is not the default view (opt-in)
- Show tier distribution chart near leaderboard so players know their percentile

### Plateau / "stuck" handling

1v1 format eliminates "ELO hell" (a team-game phenomenon — no teammates to blame).

When rating hasn't moved ±50 points in 20 matches, surface "Performance Insights":
- Specific bug type weaknesses: "Your losses cluster around recursion and memory bugs"
- Speed vs. accuracy trade-off diagnosis
- Suggested practice matches targeting weak areas (unrated)

### Anxiety reduction patterns

- **Streak recovery message** after 3+ loss streak: "Win rates normalise after streaks. Your last 10 matches show X% win rate overall."
- **Optional cool-down** between ranked matches (5–10 min) — presented as a feature, prevents rage-cascade losses
- **Rating history chart:** Smoothed trend line (not every individual match point), annotate significant moments (first promotion, personal best), show tier bands as horizontal regions
- Hide declining trajectory projections — only show when trend is positive

---

## Retention Mechanics (priority order)

**Must ship at launch:**
- ELO rating visible everywhere
- Match history with W/L record
- Bot opponents to fill empty queues

**Ship within 60 days:**
- Daily solo challenge (no opponent needed — single highest-impact post-MVP retention feature)
- Win/loss streak tracking
- Weekly leaderboard with soft reset

**Ship by month 3:**
- Achievement badges (design to reveal mechanics players haven't tried, not to reward grinding)
- Seasonal ranking tiers with promotion/demotion
- Re-engagement email: "You haven't played in 3 days" (send once, direct link to match)

---

## The Three Things That Could Kill This Product

1. **Content quality collapses** — 30 problems at launch but no pipeline = content cliff at week 6. Need ~10 new problems/month. Each problem takes 4–6 hours to create well. Plan for this resource explicitly before launch.

2. **First match is too hard or confusing** — New players juggle 4 cognitive loads at once. Easy problems must be genuinely easy (bug findable in under 2 min for a junior dev). Target: 40%+ of new users complete first match AND start a second.

3. **Empty matchmaking queue at launch** — Needs a concentrated launch event (not a slow drip). Hacker News "Show HN" is the single highest-leverage action. Must have bot fallback but bots are a floor, not a ceiling.

---

## Launch Strategy (to first 1,000 users)

1. **Private beta** (6–8 weeks before): 50–100 developers from personal networks. "Founding Member" badge. No public posts.
2. **Launch day** (one concentrated day):
   - Hacker News "Show HN" post at 9am ET Tuesday or Wednesday — highest leverage single action
   - Reddit: r/webdev, r/programming, r/learnprogramming, r/cscareerquestions (different framing per community)
   - Dev.to / Hashnode technical article about how the real-time sync was built
   - Personal LinkedIn/Twitter from all team members ("built this" framing, not "check out my startup")
3. **First week:** Monitor real-time funnel, fix top drop-off point within 24h, respond personally to all public feedback
4. **Key metric to watch:** % of users who completed first match and returned within 48 hours. If below 25% — fix retention before any more acquisition. Target: 35%+.

---

## Key Open Questions to Validate Early (Before Full Build)

1. **Does the power-up system feel fun or frustrating?** Test with 8–10 developers in a Wizard-of-Oz prototype before building.
2. **Does "no line numbers" feel like a fun challenge or arbitrary cruelty?** Validate with 5-session playtests.
3. **Does opponent status indicator motivate or harm performance?** Some anxiety-prone users may do better without it — test both.
4. **Do job seekers feel improved after 5 sessions?** Core retention claim for largest audience segment.
