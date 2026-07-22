# Stage 4 — Finding Matches
_You open the app to find a few real, reachable, worth-replying-to people — not to scroll a wall of dead profiles._

**Covers:** DISC-01, SRCH-01, MATCH-01, MATCH-02, FRESH-01, DISC-02, DISC-03

## The journey (plain language, numbered)
1. You open the app and land on **Today's picks** — a small handful (usually 5) of hand-picked people, chosen just for you and refreshed once a day.
2. Each pick shows a photo, the basics, a **fit chip** ("Strong fit"), and an honest **reachability label** ("Active this week · usually replies") so you know they'll actually respond.
3. You tap a pick to open the full card and read **"Why this match"** — 2 to 4 concrete reasons pulled from things you both actually share.
4. If a pick looks promising but you're not ready, you **shortlist** (save) it — private, they're never told.
5. Some days there's **nothing new and worth showing** — the app tells you honestly instead of recycling faces you've seen.
6. Want to go hunting yourself? You open **Search** and set filters (profession, city, age…). Results are still ranked by who actually replies, never bare filtering.
7. Coming back after a few days, **new people since your last visit** float to the top; ones you've already seen are marked "seen."
8. You check **Who viewed me** to see who's been looking (people browsing in incognito stay hidden), and dip into your **Shortlist**, where every saved person shows a live status.

## Screens

### Screen: Today's Picks (home)
```
┌──────────────────────────────────────────┐
│ iitiimrishte        🔍   🔔  👤          │
├──────────────────────────────────────────┤
│ Today's 5 · curated for you              │
│ Quality over volume. Refreshes tomorrow. │
│                                          │
│ ┌────────────────────────────────────┐   │
│ │ [ photo ]   Aarav, 31 · Bangalore  │   │
│ │             Senior PM · IIT+IIM     │   │
│ │  🟢 Strong fit                      │   │
│ │  ✓ Active this week · usually replies│  │
│ │            [ ♡ Save ]  [ View → ]   │   │
│ └────────────────────────────────────┘   │
│ ┌────────────────────────────────────┐   │
│ │ [ photo ]   Priya, 29 · London     │   │
│ │             Consultant              │   │
│ │  🟡 Good fit                        │   │
│ │  ✓ Active recently                  │   │
│ │            [ ♡ Save ]  [ View → ]   │   │
│ └────────────────────────────────────┘   │
│            · · ·  3 more  · · ·           │
│  ── That's everyone for today ──         │
│  Come back tomorrow for a fresh set.     │
└──────────────────────────────────────────┘
```
- **What happens:** Your daily curated set — small, reachable, and only people who are plausibly a good match for you too. Interest is capped here on purpose so no one drowns.
- **Edge cases:**
  - `Fewer than 5 qualify today → "Today's 2 — quality over volume." No padding with weaker or dormant profiles.`
  - `You already used today's interests → picks still show; Interest button disabled ("Used today's interests"), Save still works.`
  - `A pick goes inactive after the list was built → quietly dropped and backfilled only with an equally-qualified reachable person.`
  - `Long time away → you get one current day's set, never a pile of missed days.`

### Screen: Match Card ("Why this match")
```
┌──────────────────────────────────────────┐
│ ←                                    ♡ ⋯ │
├──────────────────────────────────────────┤
│         [    photo    ]                   │
│  Aarav, 31 · Bangalore                    │
│  Senior PM · IIT Delhi + IIM-A ✓          │
│                                           │
│  🟢 Strong fit                            │
│  ✓ Active this week · usually replies     │
│                                           │
│  ── Why this match ──                     │
│  • Both want marriage in 1–2 yrs          │
│  • Comparable career stage                │
│  • Shared value: family + ambition        │
│  • Both open to relocating                │
│                                           │
│  About                                    │
│  "Building in fintech, love trekking…"    │
│                                           │
│  [   ♡ Shortlist   ] [  Send interest  ] │
└──────────────────────────────────────────┘
```
- **What happens:** The full profile with a plain-language "why" — each reason grounded only in fields you both actually share and are allowed to see. No fabricated reasons.
- **Edge cases:**
  - `Thin profile / little info → scores on what exists + a "limited info" note; never invents a reason.`
  - `They hid a field (e.g. income) → never cited directly; may show as generic "comparable career stage."`
  - `Match is only via relocation → "why" says so plainly ("open to relocating to your city"), never shown as locally reachable.`
  - `No shared visible signal at all → chip falls back to "intent + corridor" or is hidden — never an empty chip.`

### Screen: Search & Filters
```
┌──────────────────────────────────────────┐
│ ← Search                          Reset  │
├──────────────────────────────────────────┤
│  Profession   [ Doctor        ▾]         │
│  Location     [ London +50km  ▾]         │
│  Age          [ 28 ]—[ 36 ]              │
│  Institute    [ Any           ▾]         │
│  ──────────────────────────────          │
│  Income band          🔒 Premium         │
│  Height · Community · Languages 🔒        │
│  [ ] Verified attributes only  🔒        │
│                                          │
│  Sort: (•)Most reachable  ( )Newest      │
│        ( )Best fit                       │
│                                          │
│         [   Show matches   ]             │
├──────────────────────────────────────────┤
│  128 verified reachable matches          │
│  Ranked by who actually replies.         │
└──────────────────────────────────────────┘
```
- **What happens:** You set filters and get a reachability-ranked page — filtering is the entry point, but active/responsive people always rank first. Locked filters are visibly locked, never silently applied.
- **Edge cases:**
  - `Thin corridor (<~5 reachable) → shows what exists + honest "thin market" notice: widen age/relocation, try an adjacent area, or "notify me." Never injects far-away profiles.`
  - `You've already seen everyone matching → still returns all (search is exhaustive), unseen first, rest badged "seen," with "you've seen everyone — try daily picks or widen."`
  - `Filtering on income → people who didn't disclose income are hidden; you're warned this filter excludes non-disclosers.`
  - `A dormant person matches → sunk into a collapsed "inactive — may not reply" section, not mixed into the top.`

### Screen: No New Picks Today (honest empty state)
```
┌──────────────────────────────────────────┐
│ iitiimrishte        🔍   🔔  👤          │
├──────────────────────────────────────────┤
│                                          │
│              (  ·  ·  ·  )               │
│                                          │
│     No new curated picks today           │
│                                          │
│  We only show reply-worthy matches —     │
│  so we'd rather show nothing than        │
│  recycle faces you've already seen.      │
│                                          │
│      [  🔔 Notify me when there's a match ]│
│      [  Widen my preferences  ]          │
│      [  Revisit my shortlist  ]          │
│                                          │
└──────────────────────────────────────────┘
```
- **What happens:** When nothing new is genuinely worth showing, the app says so and hands you real next steps — instead of faking a full list.
- **Edge cases:**
  - `You've seen every current match → "You're all caught up" + revisit-saved or facet-search; never re-shows seen faces.`
  - `Whole corridor is dormant → least-inactive person shown clearly labeled + notify-me + widen; never fabricates reachability.`
  - `Nothing new since last visit → "No new matches since your last visit — we'll alert you," with notify toggle.`
  - `Market temporarily frozen for balance → picks keep coming when possible; set may just be smaller, never padded.`

### Screen: Who Viewed Me
```
┌──────────────────────────────────────────┐
│ ← Who viewed me                          │
├──────────────────────────────────────────┤
│  This week · 6 viewers                   │
│                                          │
│ ┌────────────────────────────────────┐   │
│ │ [photo] Priya, 29 · London         │   │
│ │  ✓ Active recently   · viewed 2h ago│  │
│ │                       [ View → ]    │   │
│ └────────────────────────────────────┘   │
│ ┌────────────────────────────────────┐   │
│ │ [photo] Daniel, 35 · Singapore     │   │
│ │  viewed yesterday · 3 times         │   │
│ │                       [ View → ]    │   │
│ └────────────────────────────────────┘   │
│                                          │
│  🔒 2 more viewers — upgrade to see all  │
│                                          │
│  Note: people browsing in incognito      │
│  are never shown here.                   │
└──────────────────────────────────────────┘
```
- **What happens:** A reverse-chronological list of who looked at you, one entry per person with a count. Powers re-engagement without breaking the privacy promise.
- **Edge cases:**
  - `Viewer was in incognito → never appears here and never bumps the count — so it can't be reverse-engineered.`
  - `Someone viewed you many times → collapsed to one entry + view count + latest time.`
  - `A viewer later blocks you (or deletes account) → their entry disappears for both sides.`
  - `You browse in incognito yourself → you still see your own who-viewed-me; incognito hides your browsing, it doesn't blind you.`

### Screen: Shortlist (saved)
```
┌──────────────────────────────────────────┐
│ ← Shortlist                     Sort ▾   │
├──────────────────────────────────────────┤
│  Private to you. They're never told.     │
│                                          │
│ ┌────────────────────────────────────┐   │
│ │ [photo] Aarav, 31 · Bangalore      │   │
│ │  🟢 Strong fit                      │   │
│ │  ✓ Active this week                 │   │
│ │  ↪ Interest sent · awaiting reply   │   │
│ └────────────────────────────────────┘   │
│ ┌────────────────────────────────────┐   │
│ │ [photo] Rohan, 33 · Mumbai         │   │
│ │  🟡 Good fit                        │   │
│ │  ⚠ Now inactive · may not reply     │   │
│ │              [🔔 Alert me if back]  │   │
│ └────────────────────────────────────┘   │
│ ┌────────────────────────────────────┐   │
│ │ [photo] Meera, 27 · Delhi          │   │
│ │  ✕ No longer available              │   │
│ └────────────────────────────────────┘   │
└──────────────────────────────────────────┘
```
- **What happens:** Your private saved list. Each entry shows a live status, not a snapshot — so it never rots into a graveyard of non-repliers. Saving costs no interest and reachable people sort above dormant.
- **Edge cases:**
  - `Saved person goes dormant → kept, labeled "may not reply," with an optional "alert me if they're back."`
  - `Saved person deactivates or is erased → shown "no longer available" and can't be actioned; on erasure the entry is removed.`
  - `Saved person blocks you → entry disappears immediately.`
  - `Free tier hits the shortlist cap → prompts you to upgrade or remove one; never silently drops your oldest save.`
