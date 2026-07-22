# Stage 7 — Subscribing & Paying
_You want to unlock real conversations, know exactly what you'll pay and how to get out, and trust that we only bill you when we deliver reachable matches._

**Covers:** SUB-01, SUB-04, SUB-05, PAY-02, PAY-04, PAY-05, PAY-06, PAY-08

## The journey (plain language, numbered)
1. You hit a limit on Free (used your 3 interests this week, or tapped "reveal contact") and see a soft paywall — no dead end, just a clear "here's what unlocking gives you."
2. The paywall leads with the promise that matters: **verified people who actually reply this week**, not a wall of features.
3. You compare four honest rungs — **Free, Premium, Elite, Concierge** — and pick one. Concierge sends you to the web checkout (never buried in the app store).
4. Checkout shows everything on **one screen**: price, tax treatment, your renewal date, the exact renewal amount, and the refund policy — before any charge.
5. Auto-renew is a checkbox you tick yourself. It's never pre-ticked. If you skip it, your plan simply runs out at period end.
6. You pay with what suits your region (UPI/card/netbanking in India, cards/wallets in the West). You get an itemized receipt.
7. Days before renewal, we remind you with the exact date and amount, plus a one-tap way to cancel, pause, or manage — even from the reminder itself.
8. If you ever want out, we offer **Pause first** (freeze billing 1–3 months) and "I found someone" before cancel. Cancel is one confirmation, not a maze.
9. If you ask for a refund, you watch it move through named states with a settlement date — no "email us and wait."
10. If we fail to surface you 8 reachable matches this cycle, your paid time **auto-extends free** with a visible counter — no claim form. This is our answer to "null and void."
11. On Concierge, you follow a live milestone tracker mirroring your relationship manager's work; success-fees only ever trigger on a verified outcome.

## Screens

### Screen: Tier & Paywall
```
┌──────────────────────────────────────────┐
│ ← Unlock conversations                    │
│                                            │
│  12 verified people replied within 48h    │
│  in your matches this week.                │
│  Pick how you want to reach them.          │
│                                            │
│ ┌────────────────────────────────────────┐│
│ │ FREE — you're here                     ││
│ │ Browse the real pool · reachability    ││
│ │ labels · 3 interests / week            ││
│ └────────────────────────────────────────┘│
│ ┌────────────────────────────────────────┐│
│ │ PREMIUM        ₹1,999/mo  ·  ★ popular  ││
│ │ Unlimited messaging · full discovery   ││
│ │ "Active this week" filter              ││
│ └────────────────────────────────────────┘│
│ ┌────────────────────────────────────────┐│
│ │ ELITE                    ₹3,499/mo     ││
│ │ Priority ranking · who-viewed-me       ││
│ │ Monthly review · REACHABILITY GUARANTEE ││
│ └────────────────────────────────────────┘│
│ ┌────────────────────────────────────────┐│
│ │ CONCIERGE            from ₹75,000      ││
│ │ Human matchmaker · milestones          ││
│ │ Web checkout ↗ (best price)            ││
│ └────────────────────────────────────────┘│
│                                            │
│  [  Choose Premium  →  ₹1,999/mo  ]       │
└──────────────────────────────────────────┘
```
- **What happens:** You see four honest rungs led by reachability value ("people replied this week"), not a feature dump. Free is a permanent home, not a trial. Tapping a paid rung goes to checkout.
- **Edge cases:**
  - `No price for your region yet → tier hidden, "Not yet available where you are" — no broken checkout`
  - `You tap Concierge in the app → "Best price on the web" ↗ opens web checkout (avoids the 20–30% store cut)`
  - `US user → sees $39/mo Premium; app-store price shown ≥20% higher than web, web flagged as cheaper`
  - `Already Premium (stale screen) → action just proceeds after a server re-check, no second charge prompt`

### Screen: Checkout (one-screen disclosure)
```
┌──────────────────────────────────────────┐
│ ← Review & pay                            │
│                                            │
│  PREMIUM · Quarterly                       │
│  ┌──────────────────────────────────────┐ │
│  │ Price today       ₹4,999 (GST incl.) │ │
│  │ You save 17% vs monthly              │ │
│  └──────────────────────────────────────┘ │
│                                            │
│  Renews on   22 Oct 2026                   │
│  Renews at   ₹4,999  (same price)          │
│  Currency    INR ₹                         │
│                                            │
│  Refund: full refund within 7 days.        │
│  After that, pro-rated by unused days.     │
│  Delivered concierge intros non-refundable.│
│  Read full policy ›                        │
│                                            │
│  ☐  Turn on auto-renew                     │
│     Off by default. You'll be reminded     │
│     before every renewal either way.       │
│                                            │
│  Pay with:  ● UPI  ○ Card  ○ Netbanking    │
│                                            │
│  [        Pay ₹4,999 securely        ]     │
│  Displayed price = charged price.          │
└──────────────────────────────────────────┘
```
- **What happens:** Price, tax treatment, renewal date, exact renewal amount, currency, and refund terms all sit on one screen before you pay. Auto-renew is an unticked box you opt into. Your consent event is logged.
- **Edge cases:**
  - `Payment fails / 3DS abandoned → "Payment didn't go through, no charge made," retry with a fresh quote; never a silent charge without access`
  - `You double-tap Pay → idempotency returns the one result, never two charges`
  - `Catalog price changed while you sat on the screen → price is locked to the quote shown; if it expired, we re-fetch and re-disclose before charging`
  - `UPI collect pending → "Approve in your UPI app" pending state; access granted only on confirmation`

### Screen: Manage subscription (pause before cancel)
```
┌──────────────────────────────────────────┐
│ ← Manage subscription                     │
│                                            │
│  Premium · active until 22 Oct 2026        │
│  Auto-renew: OFF                           │
│                                            │
│  Thinking of leaving? Try this first:      │
│                                            │
│  ┌──────────────────────────────────────┐ │
│  │  ⏸  Pause for 1–3 months             │ │
│  │  Freeze billing AND your paid days.  │ │
│  │  Your remaining days wait for you.   │ │
│  │                     [ Pause ]        │ │
│  └──────────────────────────────────────┘ │
│  ┌──────────────────────────────────────┐ │
│  │  💍  I found someone                 │ │
│  │  Share your story · earn a referral. │ │
│  │                    [ Offboard ]      │ │
│  └──────────────────────────────────────┘ │
│                                            │
│  Still want to cancel?                     │
│  [  Cancel auto-renew  ]  (1 tap)          │
│  Your plan runs to 22 Oct, then stops.     │
│                                            │
│  Change plan · View receipts · Get refund  │
└──────────────────────────────────────────┘
```
- **What happens:** Cancel always offers Pause and "I found someone" first, but cancel itself is a single confirmation — no retention maze. Pause halts billing and freezes your remaining paid days.
- **Edge cases:**
  - `You cancel → one confirm, done; win-back offer later, never a dark pattern; plan runs out its term`
  - `Pause while a guarantee counter is running → the counter freezes and resumes from the same number, never resets`
  - `Pause window ends with no resume → auto-resumes or drops to Free per the choice you set; we notify you first, never silently keep charging`
  - `You've paused too many times in 12 months → excess pause converts to a normal cancel with proration`

### Screen: Refund status tracker
```
┌──────────────────────────────────────────┐
│ ← Refund #RF-4821                         │
│                                            │
│  Premium · requested 22 Jul 2026           │
│  Amount up to  ₹3,400                       │
│                                            │
│  ●  Requested            22 Jul  ✓         │
│  │                                          │
│  ●  Under review         22 Jul  ✓         │
│  │                                          │
│  ●  Partially approved   23 Jul  ✓         │
│  │   ┌────────────────────────────────┐    │
│  │   │ Paid, 90 days  ........ ₹4,999 │    │
│  │   │ Used, 18 days  ........ −₹1,000│    │
│  │   │ Free days from guarantee  −₹599│    │
│  │   │ (not refundable)               │    │
│  │   │ GST reversed .......... incl.  │    │
│  │   │ Refund to you ........  ₹3,400 │    │
│  │   └────────────────────────────────┘    │
│  ●  Processing           23 Jul  ⟳         │
│  │                                          │
│  ○  Refunded to UPI      est. 27 Jul       │
│                                            │
│  Refund goes back to your original UPI.    │
│  Questions? Grievance officer ›            │
└──────────────────────────────────────────┘
```
- **What happens:** Every refund moves through named states with a settlement date. Deductions for used time and guarantee-granted free days are itemized in plain numbers. Each state change also notifies you.
- **Edge cases:**
  - `Refund after usage, within cooling-off → pro-rated by the days you used, shown line by line as above`
  - `Guarantee already extended your plan → those free days are excluded from the refundable base (shown explicitly)`
  - `You filed a bank chargeback too → refund freezes, "Marked disputed — resolving with your bank," no double payout`
  - `App-store purchase → "Refunds for this plan go through Apple/Google," we reflect their decision, can't issue cash ourselves`

### Screen: Reachability guarantee banner
```
┌──────────────────────────────────────────┐
│  Discover                          ⚙      │
│                                            │
│ ┌────────────────────────────────────────┐│
│ │ 🛡  Reachability guarantee              ││
│ │                                        ││
│ │ We promised 8 reachable matches this   ││
│ │ cycle. So far we've surfaced 5.        ││
│ │                                        ││
│ │  ▓▓▓▓▓▓▓▓▓▓░░░░░░  5 / 8               ││
│ │                                        ││
│ │ We owe you 3 more reachable matches.   ││
│ │ Your billing is PAUSED and your paid   ││
│ │ time is extending free until we do.    ││
│ │                                        ││
│ │ Renewal moved: 22 Oct → 05 Nov         ││
│ │ You did your part — this is on us.     ││
│ │                          Learn how ›   ││
│ └────────────────────────────────────────┘│
│                                            │
│  Today's verified, active matches ↓        │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐   │
│  │  match   │ │  match   │ │  match   │   │
│  └──────────┘ └──────────┘ └──────────┘   │
└──────────────────────────────────────────┘
```
- **What happens:** When we under-deliver reachable matches, your paid time auto-extends free with a visible counter and reason — no claim form. The banner distinguishes "matches we surface" (our job) from "replies you get" (never guaranteed).
- **Edge cases:**
  - `Guarantee triggers automatically → billing pauses, renewal date pushes out, counter + reason shown; no manual claim needed`
  - `We surfaced 8+ but you were inactive → no breach; banner explains "we delivered; replying is up to both people"`
  - `Your profile strength is below the bar → not eligible for the reply-floor credit; shown concrete, dignity-preserving coaching fixes`
  - `Guarantee credit then you refund → the free days it granted are excluded from the refundable base (no double remedy)`

### Screen: Concierge milestone tracker
```
┌──────────────────────────────────────────┐
│ ← Your Concierge engagement               │
│                                            │
│  RM: Anjali M.   ·   Contract #CG-207      │
│  Base ₹75,000 + ₹51,000 success-fee        │
│  Success-fee bills ONLY on a verified      │
│  engagement — never before.                │
│                                            │
│  ●  Onboarding call      done · 02 Jul     │
│  │                                          │
│  ●  Shortlist delivered  done · 09 Jul     │
│  │                                          │
│  ◉  Curated intros  (3 of 5)               │
│  │   ▓▓▓▓▓▓░░░░  in progress               │
│  │   Target 30 Jul · refundable ₹20,000    │
│  │   if we miss it (platform fault)        │
│  │                                          │
│  ○  Verified engagement   not started      │
│      Success-fee milestone                 │
│                                            │
│  A missed platform-fault milestone         │
│  auto-credits you per this schedule.       │
│                                            │
│  [ Message Anjali ]   [ Raise a concern ]  │
└──────────────────────────────────────────┘
```
- **What happens:** You watch a live milestone tracker mirroring your relationship manager's CRM. Each milestone shows its target date and the exact amount refunded if we miss it. Success-fees invoice only on an evidence-verified outcome.
- **Edge cases:**
  - `Platform-fault milestone missed → the refundable amount auto-credits per schedule, you're notified with the reason`
  - `You dispute an intro's quality → opens a case; reachability logs + RM notes are the evidence; resolved within SLA`
  - `You cancel mid-engagement → pro-rated by milestones completed, not the calendar; delivered intros non-refundable, undelivered ones refunded`
  - `Your RM is reassigned → full history transfers, milestone dates hold, you're notified — engagement doesn't restart`
