# User Journeys — iitiimrishte.com

**The simple, visual layer.** Each stage of the app told as a plain-language **user journey** + **ASCII screen mockups** + the **key edge cases** for each screen.

**Version:** 0.1 (Draft) · **Date:** 2026-07-22
**How this relates to the other docs:**
- [PRD](./PRD.md) = the *what & why*
- [Feature Spec](./FEATURE-SPEC.md) (`docs/spec/`) = the *full detail* (every feature, every edge case, acceptance criteria)
- **This** (`docs/journeys/`) = the *simple, human view* — what a user actually does and sees, screen by screen

Feature IDs (e.g. `VER-2`, `BAL-01`) tie each journey back to the full spec.

---

## The 7 stages (a user's path through the app)

```
  1. Get in  →  2. Get verified  →  3. Build profile  →  4. Find matches
       →  5. Connect  →  6. Stay safe (all along)  →  7. Subscribe (when ready)
```

| # | Stage | What the user is doing | File |
|---|---|---|---|
| 1 | **Getting In** | Apply, wait (men) or walk in (women), get admitted | [`journeys/01-getting-in.md`](./journeys/01-getting-in.md) |
| 2 | **Getting Verified** | Prove identity + credential + intent | [`journeys/02-getting-verified.md`](./journeys/02-getting-verified.md) |
| 3 | **Building Your Profile** | Fill it in, set photos + privacy, invite family | [`journeys/03-building-your-profile.md`](./journeys/03-building-your-profile.md) |
| 4 | **Finding Matches** | Daily curated picks, search, who-viewed-me, shortlist | [`journeys/04-finding-matches.md`](./journeys/04-finding-matches.md) |
| 5 | **Connecting** | Send interest, small ranked inbox, chat, contact reveal | [`journeys/05-connecting.md`](./journeys/05-connecting.md) |
| 6 | **Staying Safe** | Report, silent block, filtered messages, scam nudges | [`journeys/06-staying-safe.md`](./journeys/06-staying-safe.md) |
| 7 | **Subscribing & Paying** | Tiers, honest checkout, refunds, reachability guarantee | [`journeys/07-subscribing-and-paying.md`](./journeys/07-subscribing-and-paying.md) |

---

## The one idea in every stage

Each screen quietly fixes a specific incumbent failure:

- **Get in** — a queue that keeps the room *balanced* (so replies are real), not a paywall.
- **Get verified** — same bar for everyone; badges show *how* they were verified, not a flat checkmark.
- **Build profile** — you're invisible until *you* set your privacy; hide anything without being penalized.
- **Find matches** — a few *reachable* people a day, honest empty states — never a wall of dead profiles.
- **Connect** — a small ranked inbox, never a flood; nobody is ever punished for not replying.
- **Stay safe** — report and *hear back*; block silently; get warned at the exact risky moment.
- **Subscribe** — one-screen pricing, one-tap cancel, and billing *pauses* if we don't deliver matches.

_These are a simplified, presentation-friendly view. The authoritative behavior (all states, acceptance criteria, dependencies) lives in the [Feature Spec](./FEATURE-SPEC.md)._
