# Stage 6 — Staying Safe
_You want to shut down a creepy, scammy, or abusive person — and actually know something was done — without ever tipping that person off._

**Covers:** SAFE-01, SAFE-03, SAFE-04, SAFE-05, SAFE-08, SAFE-09

## The journey (plain language, numbered)
1. Something feels off in a chat — a pushy money ask, an explicit photo, or just a bad vibe. The safety tools are one tap away, everywhere the person appears (profile, chat, interest card, who-viewed-me).
2. If a message is unsafe, you never get hit with it cold — it arrives as a covered-up placeholder ("1 message hidden"), and you decide whether to peek.
3. At the risky moment — someone mentions money, or pushes to move to WhatsApp — a small nudge appears just for you, reminding you of the classic scam and that a verified badge is not a character reference.
4. You can **report** the person, pick a reason in plain words, and add proof if you want. You get a case number and an honest "here's when you'll hear back."
5. Later, the app closes the loop: "We reviewed and took action" — without naming the person or the punishment.
6. Separately, you can **silently block**. The other person is simply gone — no "you've been blocked" banner, no way for them to tell.
7. Throughout, tapping any verified badge explains exactly what it proves — and, in plain type, what it does **not**.

## Screens

### Screen: Report a user — pick a reason

```
┌──────────────────────────────────────────┐
│  ✕            Report Rohan M.            │
│──────────────────────────────────────────│
│  This is private. Rohan is never told     │
│  you reported him.                        │
│                                            │
│  Why are you reporting?                    │
│                                            │
│  ○ Asked me for money / gift cards        │
│  ○ Threat, blackmail or "I'll leak..."    │
│  ○ Sent explicit / unwanted photos        │
│  ○ Fake profile or stolen photos          │
│  ○ Married / lied about their status      │
│  ○ Harassment or abusive language         │
│  ○ Pushing me to chat off the app         │
│  ○ Something else                         │
│                                            │
│  ┌────────────────────────────────────┐   │
│  │  Add proof (optional)              │   │
│  │  📎 Screenshots  ✓ Pick messages   │   │
│  └────────────────────────────────────┘   │
│                                            │
│     [  Also block Rohan  ]                 │
│     [       Submit report        ]         │
└──────────────────────────────────────────┘
```
- **What happens:** One tap from anywhere Rohan appears opens this sheet; you pick a plain-language reason, can attach screenshots or tick specific real messages as evidence, and optionally block in the same move. Reporting is free for everyone — no paywall on safety.
- **Edge cases:**
  - `Picks "Threat / blackmail" or explicit content → the disturbing message is auto-preserved; you're NEVER asked to re-open or re-upload it — one tap is enough, fast-tracked to a specialist.`
  - `Already reported this person → folded into the existing case, no duplicate, and the promised response time doesn't reset.`
  - `Rohan's account is already gone → report still logged against his identity (so evasion is caught); you see "This account is no longer active — thank you."`
  - `Report out of spite / no real issue → still accepted politely, but repeated false reports quietly lower your own report weight; never a hostile "rejected" message.`

### Screen: We got your report — and, later, what we did

```
┌──────────────────────────────────────────┐
│              ✓  Report received            │
│──────────────────────────────────────────│
│                                            │
│   Thanks — a real person on our safety     │
│   team will review this.                   │
│                                            │
│   Case #  A-4821                           │
│   Reason  Asked me for money               │
│                                            │
│   You'll hear back within                  │
│   ┌────────────────────────────────────┐   │
│   │        about 24 hours              │   │
│   └────────────────────────────────────┘   │
│                                            │
│   You don't have to wait — you can         │
│   block Rohan now, either way.             │
│                                            │
│   ── later, case #A-4821 ──────────────    │
│   ✓  We reviewed this and took action.     │
│      Thank you for keeping the community    │
│      safe.   [ How we handle reports ]     │
│                                            │
└──────────────────────────────────────────┘
```
- **What happens:** On submit you get an instant confirmation, a durable case number, and an honest expected-response window based on severity. Later the same case updates to a closed status — an outcome category only ("took action" / "didn't find a violation, here's how to stay safe" / "we've escalated this").
- **Edge cases:**
  - `Threat / extortion / minor reason → window shows "within 1 hour, 24/7" instead of 24h; highest priority.`
  - `You tap "tell me exactly what happened to Rohan" → shown outcome category only, plus a short "why we can't share the details" note and the "verified ≠ vouched" explainer — never his punishment or personal info.`
  - `We didn't find a violation → framed as "here's how to stay safe / add more info," with a reopen path — never a curt dismissal.`
  - `Same person re-offends after a "no violation" close → new report auto-links to the old one and is treated with extra scrutiny, not brushed off as "already handled."`

### Screen: Silent block — done, and they'll never know

```
┌──────────────────────────────────────────┐
│              ✓  Rohan is blocked           │
│──────────────────────────────────────────│
│                                            │
│         🔇                                 │
│                                            │
│   He's gone from your search, chats,       │
│   interests and who-viewed-you.            │
│                                            │
│   ┌────────────────────────────────────┐   │
│   │  Rohan is NOT told he was blocked. │   │
│   │  To him, you simply look inactive. │   │
│   │  His messages won't reach you.     │   │
│   └────────────────────────────────────┘   │
│                                            │
│   You don't owe anyone a reason.           │
│                                            │
│   Manage in  Settings › Blocked list       │
│   (only you can see this list)             │
│                                            │
│              [   Done   ]                   │
└──────────────────────────────────────────┘
```
- **What happens:** The block is instant and one-sided-invisible. Rohan disappears from all of your surfaces; on his side you just look like an inactive account. No banner, no read receipts, no error that could reveal the block. It needs no justification and is never appealable by him.
- **Edge cases:**
  - `Blocked user tries to reopen the old chat or send → his message appears to send on his phone but is silently dropped, never delivered, and (by default) never backfilled if unblocked — no error hints at a block.`
  - `Blocked user searches filters that would surface you → you're quietly excluded; result count adjusts naturally, no "1 hidden" tell.`
  - `Blocked user "probes" (profile gone? messages unread? vanished from search?) → every signal looks identical to a deactivated or incognito account; there's no way to confirm a block.`
  - `You later unblock → normal interaction resumes going forward, but messages he sent during the block are gone, not dumped in as a flood.`

### Screen: A message is hidden (opt-in reveal)

```
┌──────────────────────────────────────────┐
│  ‹  Rohan M.                       ⋮      │
│──────────────────────────────────────────│
│                                            │
│   Rohan · 9:14 pm                          │
│   ┌────────────────────────────────────┐   │
│   │  ▨▨▨▨▨  1 message hidden           │   │
│   │                                    │   │
│   │  Possible explicit content.        │   │
│   │  We covered it so it didn't hit    │   │
│   │  you by surprise.                  │   │
│   │                                    │   │
│   │  [ View anyway ]   [ Keep hidden ] │   │
│   │  [ Report ]        [ Block ]       │   │
│   └────────────────────────────────────┘   │
│                                            │
│   ⓘ Rohan is not told this was filtered.   │
│                                            │
│  ┌──────────────────────────────┐  [Send] │
│  │ Message…                     │         │
│  └──────────────────────────────┘         │
└──────────────────────────────────────────┘
```
- **What happens:** Unsafe inbound (unwanted explicit content or abuse) is quarantined before you see it and shown as a covered placeholder. You choose whether to reveal, keep it hidden, report, or block. The sender is never told his message was filtered.
- **Edge cases:**
  - `Explicit image → shown as a blurred placeholder, never auto-displayed; "View anyway" reveals it.`
  - `Benign message wrongly flagged → "View anyway" clears it and quietly teaches the filter; the sender isn't punished for one low-confidence miss.`
  - `You tap "View anyway" then report → the message was already saved as evidence, so the report attaches to it directly.`
  - `Contact info smuggled in ("insta @…", "num8er nine…") before you've both agreed to share → stripped/redacted inline with a "contact sharing is off until you both agree" note, not fully hidden.`

### Screen: In-context scam nudge

```
┌──────────────────────────────────────────┐
│  ‹  Aisha K.                        ⋮     │
│──────────────────────────────────────────│
│   Aisha · 7:02 pm                          │
│   my card got blocked at the airport,      │
│   can you send ₹40,000, I'll pay back 🙏   │
│                                            │
│   ┌─ ⚠  Just for you ─────────────────┐   │
│   │  Never send money to someone you   │   │
│   │  haven't met in person. This is    │   │
│   │  the most common scam here.        │   │
│   │                                    │   │
│   │  "Overseas emergency" stories are  │   │
│   │  a classic script. Take your time. │   │
│   │                                    │   │
│   │  [ Learn more ] [ Report ] [ Block]│   │
│   │                        [ Dismiss ] │   │
│   └────────────────────────────────────┘   │
│   ⓘ Aisha can't see this reminder.         │
│                                            │
│  ┌──────────────────────────────┐  [Send] │
│  │ Message…                     │         │
│  └──────────────────────────────┘         │
└──────────────────────────────────────────┘
```
- **What happens:** When the conversation hits a known risky moment — a money request, an "overseas emergency," or a push to move off the app — a private, non-blocking nudge appears for you only. It educates, offers Report/Block/Learn more, and can be dismissed. The other person never learns they were flagged.
- **Edge cases:**
  - `It's a genuine long-distance partner talking about money/travel → nudge is advisory only, dismissible, no penalty to either side.`
  - `You dismiss the nudge → it goes away and won't nag repeatedly in the same thread; the event is quietly kept in case it's needed for later victim support — never used to blame you.`
  - `Push to move to WhatsApp/off-app → nudge reminds that leaving removes the app's protections and that "verified ≠ vouched."`
  - `Money + a threat appear together → this stops being a gentle nudge; it's fast-tracked to the safety team with the evidence preserved.`

### Screen: "Verified ≠ vouched" badge tooltip

```
┌──────────────────────────────────────────┐
│   Rohan M., 34 · Bengaluru                 │
│   ✔ Identity   ✔ IIT Delhi   ✔ Employer   │
│──────────────────────────────────────────│
│   ┌─ About the ✔ Identity badge ────────┐ │
│   │                                      │ │
│   │  WHAT THIS PROVES                    │ │
│   │  • A real government ID matches a    │ │
│   │    live selfie of this person.       │ │
│   │  • Their degree from IIT Delhi is    │ │
│   │    confirmed with the institute.     │ │
│   │                                      │ │
│   │  WHAT THIS DOES NOT PROVE            │ │
│   │  • Their character or intentions.    │ │
│   │  • That they are safe, honest, or    │ │
│   │    single.                           │ │
│   │                                      │ │
│   │  Verified is not vouched. Trust is   │ │
│   │  earned in the conversation.         │ │
│   │                                      │ │
│   │       [ Safety guide ]   [ Close ]   │ │
│   └──────────────────────────────────────┘ │
└──────────────────────────────────────────┘
```
- **What happens:** Tapping any badge shows, in equal weight, exactly what it attests and — spelled out plainly — what it does not. It links to the safety resource center and reinforces the core message that a verified badge confirms identity and credentials, never good intent.
- **Edge cases:**
  - `You keep over-trusting badges and ignoring nudges → the "what this does NOT prove" line can't be permanently dismissed at the badge level; reminders re-appear at high-trust moments (first contact-reveal, before a first meeting).`
  - `First-time user → a short safety module explaining "verified ≠ vouched" is shown once, up front, before full access.`
  - `Reached from a scam nudge or a report screen → "Learn more" lands here, so the education is one tap from every risky moment.`
  - `Screen reader / low vision → fully text-based and WCAG 2.2 AA, never an image-only badge explainer.`
