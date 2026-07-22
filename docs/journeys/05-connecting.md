# Stage 5 — Connecting
_You found someone worth reaching. Now you want to reach them — and actually get a reply._

**Covers:** INT-01, INT-02, INT-04, INT-06, INT-08, MSG-01, MSG-02, REACH-01, REACH-02

## The journey (plain language, numbered)

**Sender's side (Aarav):**
1. On a curated pick's profile he sees honest labels first: "Active this week · high inbound ~15% reply." He knows his odds before spending anything.
2. He taps **Send Interest**. A composer opens — he MUST write a personal note (min 40 chars, no phone numbers, not a copy-paste of his last one). A counter reads **"2 interests left today."**
3. He sends. The note becomes message 1 if she ever replies. His daily count ticks down. He is never allowed to spray.
4. If her inbox is full, he's told **"Inbox full — we'll resurface them"** and it costs him nothing (no interest spent).
5. He tracks everything privately in his **Reachability dashboard** — reply rate, non-reply wins (views, shortlists), and his current cap.

**Receiver's side (Priya):**
6. She never sees a flood. Her inbox is a **small ranked queue** — best-first, capped at ~10–15. "5 people are waiting," not "400 pending."
7. She reads a note, and either replies (→ opens a mutual chat) or lets it go. **She never has to reply. No penalty, ever.**
8. When she replies, a chat opens seeded with his original note. They talk.
9. Contact details reveal only when **both tap agree** — never on payment. A two-sided handshake.
10. **Women-initiate lane:** in this lane only she opens; men appear as candidates she may reach out to. Same effort-taxed note, mirrored.

---

## Screens

### Screen: Send-Interest Composer (sender)

```
┌────────────────────────────────────────────┐
│ ←         Send an interest                  │
├────────────────────────────────────────────┤
│  ┌────┐  Priya, 29 · Consultant · London    │
│  │ 👤 │  ✅ Active this week                 │
│  └────┘  📥 High inbound · ~15% reply        │
├────────────────────────────────────────────┤
│  Write a personal note (required)           │
│  ┌──────────────────────────────────────┐   │
│  │ Your work on health-policy caught my │   │
│  │ eye — I moved into public-health PM   │   │
│  │ last year. What drew you to it?_      │   │
│  └──────────────────────────────────────┘   │
│  62 / 40 min  ✓ personal  ✓ no contact info  │
│                                              │
│         ┌────────────────────────┐           │
│         │     Send interest      │           │
│         └────────────────────────┘           │
│                                              │
│   ⏳ 2 interests left today · resets 12 AM    │
└────────────────────────────────────────────┘
```
- **What happens:** Send stays greyed out until the note passes every gate. The honest reply-odds label sits right above the box so he self-selects before spending.
- **Edge cases:**
  - `Note under 40 chars → Send stays disabled, inline "add a bit more — make it personal"`
  - `Note is a copy-paste of a recent one → rejected "make it your own"; repeats quietly lower his merit`
  - `Note contains a phone/email → stripped + warned on first try, number never delivered`
  - `Out of daily interests → composer locked: "You're out for today. Resets 12:00 AM" + earn-more path (no queueing)`

---

### Screen: Her Inbox — a small ranked queue (receiver)

```
┌────────────────────────────────────────────┐
│  Interests            5 people are waiting   │
├────────────────────────────────────────────┤
│  Best matches first · you're never obliged   │
│                                              │
│  ┌──────────────────────────────────────┐   │
│  │ 👤 Aarav, 31 · IIT+IIM · PM      ⭐95 │   │
│  │ "Your work on health-policy caught…" │   │
│  │ ✅ Verified · replies often           │   │
│  └──────────────────────────────────────┘   │
│  ┌──────────────────────────────────────┐   │
│  │ 👤 Rohan, 33 · Doctor           ⭐88  │   │
│  │ "We both grew up between Delhi and…" │   │
│  └──────────────────────────────────────┘   │
│  ┌──────────────────────────────────────┐   │
│  │ 👤 Vik, 30 · Founder            ⭐81  │   │
│  │ "Saw you climb — my last trip was…"  │   │
│  └──────────────────────────────────────┘   │
│         … 2 more ·  ✕ clear any anytime      │
└────────────────────────────────────────────┘
```
- **What happens:** Never a flood — a capped, best-first queue ordered by fit, sender reachability, and note effort. She acts at her pace or not at all.
- **Edge cases:**
  - `Queue hits its cap → she leaves new discovery; new senders see "inbox full", nothing lands in a backlog`
  - `She ignores everyone while oversubscribed → zero nudges, zero penalty, no ranking drop (INT-06)`
  - `She bulk-clears the queue → allowed; declines are silent to senders (no "rejected" shame)`
  - `An interest sits 7 days untouched → it quietly expires, frees a slot; only the sender is notified, never her`

---

### Screen: Mutual match → Chat (both sides)

```
┌────────────────────────────────────────────┐
│ ←   Priya  ✅        🎉 It's mutual!         │
├────────────────────────────────────────────┤
│                                              │
│      You both said yes. Say hello.           │
│                                              │
│  ┌──────────────────────────────────────┐   │
│  │ Aarav · your opening note            │   │
│  │ "Your work on health-policy caught   │   │
│  │  my eye — what drew you to it?"      │   │
│  └──────────────────────────────────────┘   │
│              ┌───────────────────────────┐   │
│              │ Priya: The community side  │   │
│              │ of it, honestly. You?      │   │
│              └───────────────────────────┘   │
│  ┌──────────────────────────────────────┐   │
│  │ Aarav: Same — impact over titles.    │   │
│  └──────────────────────────────────────┘   │
├────────────────────────────────────────────┤
│  [ Type a message… ]   🔒 Reveal contact  ⚑  │
└────────────────────────────────────────────┘
```
- **What happens:** Chat exists only because she accepted a delivered, effort-taxed interest. His note is message 1. Report/block sit inline; contact info stays auto-masked until the reveal handshake.
- **Edge cases:**
  - `One person blocks mid-chat → thread silently freezes; blocked party sees no "blocked" flag, messages just stop`
  - `She replied only to decline politely → closes the interest silently, no persistent chat forced open`
  - `Someone types a phone number → auto-masked in the bubble until both consent to reveal`
  - `One deletes their account → thread tombstoned with neutral copy, no leftover PII`

---

### Screen: Contact reveal — both must agree (both sides)

```
┌────────────────────────────────────────────┐
│ ←         Share contact details             │
├────────────────────────────────────────────┤
│   Contact reveals only when you BOTH agree.  │
│   Never unlocked by payment.                 │
│                                              │
│   You want to share:                         │
│     [✓] Phone     [✓] WhatsApp               │
│     [ ] Employer  (stays hidden)             │
│                                              │
│         ┌────────────────────────┐           │
│         │   Offer to share       │           │
│         └────────────────────────┘           │
│  ─────────────────────────────────────────   │
│   Priya's side:                              │
│     ⏳ Waiting for Priya to agree            │
│        You can withdraw your offer anytime.  │
│                                              │
│   When both tap agree → details exchange 🔓  │
└────────────────────────────────────────────┘
```
- **What happens:** A two-sided handshake with per-field control. One person offering is never enough — both consents are recorded before anything is exchanged.
- **Edge cases:**
  - `He agrees, she never does → nothing reveals; his number is never exposed; he can withdraw the offer`
  - `Paid user assumes money unlocks it → UI states plainly: consent-gated, not purchasable`
  - `Both agree, then one blocks → platform stops facilitating contact; abuse routes to a T&S escalation lane`
  - `Underage attempt → blocked upstream by age-gating; no reveal path exists`

---

### Screen: Personal Reachability Dashboard (sender)

```
┌────────────────────────────────────────────┐
│  Your reachability            This month     │
├────────────────────────────────────────────┤
│   Interests sent            18               │
│   Replies received           4               │
│                                              │
│   Your reply rate  ██████░░░░░░░░░  22%       │
│   Corridor norm    ████████░░░░░░░  28%       │
│                                              │
│   Wins that aren't replies:                  │
│     👁 37 profile views   ⭐ 9 shortlisted     │
│                                              │
│   Today's cap: 3  ·  Merit: strong ▲         │
│   Lever → personalise notes = +reply rate    │
│                                              │
│   Guarantee: 6 / 8 reachable matches ✓        │
│   ────────────────────────────────────────   │
│   Some targets were high-inbound corridors — │
│   that's odds, not you. Not counted against  │
│   your merit.                                │
└────────────────────────────────────────────┘
```
- **What happens:** Private, first-person honesty. Shows reply rate vs corridor norm, non-reply wins, current cap + one concrete lever, and the reachability-guarantee counter — reframing scarcity as legible odds.
- **Edge cases:**
  - `Low replies in a throttled/red-line corridor → labeled "balance/throttle", not user failure; guarantee counter accruing`
  - `Low replies because targets were oversubscribed → shown honestly, excluded from self-blame and from merit math`
  - `Throttled by governor, not merit → shows a balance explanation, not a "fix your profile" nudge`
  - `New user, sparse data → onboarding guidance instead of a misleadingly empty 0%`

---

> **Women-initiate lane note:** In this lane only she opens — men appear as candidates she may reach out to, and her opening note carries the same effort-tax (min length, no contact info) against her own separately-governed allowance. If a member sets "women-initiate only," a man's attempt to send is blocked pre-delivery ("this member chooses who reaches her") and costs him nothing. The mechanism is gender-neutral: the "who initiates" role follows the constrained side of the matching market, so same-gender and non-binary users are handled without mis-gating. If he never responds and her interest expires, she gets a neutral sender-side notice — and he is never penalized unless he has free capacity and chronically ignores everyone.
