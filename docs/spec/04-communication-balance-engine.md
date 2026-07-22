# Feature Spec — Communication, Interest & the Two-Sided Balance Engine

> Domain 4 of the [Feature Specification](../FEATURE-SPEC.md). The market-design core that fixes the incumbent's collapse. Companion: [PRD](../PRD.md) §7.4, [liquidity brief](../gtm/raw/02-liquidity-gender-balance.md), [male brief](../gtm/raw/03-male-acquisition-monetization.md), [female brief](../gtm/raw/04-female-first-acquisition-safety.md).

**Purpose.** iitiimshaadi ran an uncapped auction where women's attention was free to men's demand: demand flooded, women drowned under "thousands of interests/week," women left, men got "null and void." Every feature converts attention into a **governed, rationed, reciprocal** resource, meters demand to supply capacity, and protects the scarce good — a real reply.

**Cross-cutting definitions:**
- **Active/reachable user:** verified, session in trailing 7 days, not at hard inbound saturation.
- **Sender / Recipient:** initiator / receiver of an interest. In the women-initiate lane the woman is always the sender.
- **Oversubscribed recipient:** at/near inbound cap (INT-04). Reply-fairness penalties (INT-06) NEVER apply to this party.
- **Corridor:** the unit of liquidity. All caps/ratios/governors computed **per corridor, per gender/market.**
- **Gender for the valve:** a market-balance attribute derived from the user's gender + stated partner-preference (BAL-05), not a raw profile field.

---

## INT — Interest & Rationing

### INT-01: Outbound daily interest cap
- **Description:** Hard per-user daily ceiling on outbound interests — counters the incumbent's uncapped mass-contact. Caps demand *at the source*. Default asymmetric/configurable: base 3/day (men free), 5/day (paid); Tier 0 = 3/week (the daily cap is finer-grained enforcement under any weekly ceiling).
- **Expected behavior:**
  - Each sent interest decrements the sender's remaining allowance for that corridor-day (server-authoritative, sender-timezone-anchored). Remaining shown before/after each send. Resets at sender's local midnight.
  - Allowance = **minimum** of (a) tier entitlement, (b) merit-adjusted quota (INT-03), (c) any ratio-governor throttle (BAL-02).
  - An undeliverable interest (recipient inbound-capped, INT-04) does NOT consume allowance.
- **Edge cases & handling:**
  - Allowance exhausted → `block send; show reset time + upsell/merit path; no queueing`.
  - Allowance 0 due to governor throttle (not tier) → `block with balance-framed copy ("keeping the corridor in balance so you get replies"), NOT an upsell; log throttle reason`.
  - Timezone-change farming → `reset bound to a rolling 24h window, max one reset per calendar day; tz changes take effect after 72h cooldown for cap purposes`.
  - Merit raises cap mid-day → `new allowance immediately available; sent count preserved`.
  - Interest to a recipient who then blocks before read → `no allowance refund (block-refund farming risk); refund only when send was rejected pre-delivery (INT-04)`.
  - At-cap sender likes a curated pick → `draws same daily pool unless config grants separate curated allowance (Tier 2)`.
- **Acceptance criteria:** free male in a balanced corridor can't send a 4th in a rolling 24h (default 3); shown remaining = server remaining (no client drift on send); governor throttle reflected in effective cap within 60s; no tz change yields >1 reset/calendar day.
- **Dependencies:** INT-03, INT-04, BAL-02, monetization tiers, user timezone.

### INT-02: Effort tax — personalized note required
- **Description:** Every outbound interest MUST carry a personalized note/prompt-answer; bare one-tap likes disallowed — raises the cost of low-effort mass-likes and gives the female side *considered* contact.
- **Expected behavior:** composer requires a note passing gates before Send enables: min length (default 40 chars), not identical to sender's previous N notes (anti-copy-paste), no contact-info patterns (blocked pre-consent, MSG-02), passes anti-abuse filter. Optionally answers a recipient prompt (higher effort-score → merit weighting). The note is delivered attached and is message 1 if it becomes mutual (seeds MSG-01).
- **Edge cases & handling:**
  - Note fails min length → `Send disabled with inline reason`.
  - Duplicate of a recent note → `reject "make it personal"; repeated attempts lower merit, soft rate-limit`.
  - Contains contact info / off-platform solicitation → `strip-and-warn first offense; block + flag on repeat`.
  - Abusive/explicit → `block send, log to moderation, count toward sanction; never delivered`.
  - Passes but later reported → `report to T&S; confirmed abuse penalizes merit, may void allowance`.
  - Women-initiate lane → `same effort-tax on her opening note (she's a sender there)`.
  - Voice-note as the note → `supported; transcribed for filter checks; WCAG 2.2 AA`.
- **Acceptance criteria:** no interest without a note passing all gates (API rejects noteless payloads); two identical notes within the anti-copy window both rejected after the first; a note with a phone number never delivered with the number intact pre-consent.
- **Dependencies:** INT-01, INT-03, MSG-01, MSG-02, PRD §7.4/§7.5.

### INT-03: Merit-based quota expansion
- **Description:** Outbound allowance flexes with behavior — replies earn more, reports/ignores earn fewer. Rewards reply-worthy senders; starves spray-and-pray.
- **Expected behavior:** rolling per-corridor **merit score** from reply-rate (+), mutual-match rate (+), high effort-score notes (+), reports (−−), blocks (−), chronic ignores/expiries (−), duplicate notes (−). Maps to a quota multiplier bounded [0.5×–2× base], on top of tier but never exceeding safety ceiling or an active governor throttle. Moving average with decay (recovery possible; gradual changes except a confirmed T&S sanction).
- **Edge cases & handling:**
  - New user, no history → `neutral baseline = tier base cap; no absence penalty`.
  - **Oversubscribed recipient never replies → ignores from an inbound-capped/oversubscribed recipient are EXCLUDED from the sender's merit denominator (sender not penalized because the woman is drowning).**
  - Interest expires because recipient was flooded → `same exclusion; expiry counts against merit ONLY when the recipient was reachable and chose not to act, weighted lightly`.
  - Many reports in a short window → `hard merit floor + possible temporary send suspension pending T&S; identity-linked so re-registration doesn't reset`.
  - Merit ceiling vs governor → `governor wins; high-merit man in a red-line corridor still throttled; merit preserved for reopening`.
  - Off-platform solicitation to inflate merit → `merit weights on-platform signals only`.
  - Decays to floor then reforms → `decay + positive events allow recovery; no permanent lock except T&S ban`.
- **Acceptance criteria:** strong-reply-rate sender gets a measurably higher cap (within bounds); ignores/expiries from oversubscribed recipients don't lower merit (assert with a capped recipient); a confirmed abuse report drops the cap within one cycle and survives re-registration.
- **Dependencies:** INT-01/02/04, INT-06, BAL-02, PRD §7.5.

### INT-04: Per-profile inbound cap + ranked working queue
- **Description:** The single most important anti-flood mechanism. Each recipient has a hard cap on **pending** inbound interests (~10–15). At cap, the profile leaves new discovery until worked down. Interests are never dumped — a **ranked, best-first, manageable** working queue. Structurally prevents the "woman drowning" inbox.
- **Expected behavior:**
  - Pending = received, not acted on, not expired. Pending < cap → discoverable, new interests inserted by rank. Pending == cap (**saturated**) → **removed from new-discovery + curated picks for new senders**; no new inbound accepted.
  - Ranked queue: pending ordered best-first by compatibility, sender reachability/merit, effort-score, recency. Acting (reply→mutual / decline) or expiry decrements pending, re-opens discoverability, admits next waitlisted sender (if enabled). Cap configurable per corridor; auto-tightened when female inbound load >10/wk.
- **Edge cases & handling:**
  - At cap, new sender tries → `interest rejected pre-delivery; allowance NOT consumed; "inbox is full — we'll resurface them"; optional thin inbound waitlist (config), admitted FIFO-by-rank when a slot frees`.
  - Inbound waitlist disabled → `recipient drops out of discovery; no backlog (preferred default)`.
  - At cap AND oversubscribed woman → `no reply-nudge, no penalty, no ranking demotion for the unworked queue (INT-06)`.
  - Cap lowered by governor while above new cap → `no interests dropped; recipient stays out of discovery until natural attrition`.
  - Already-delivered interest, then cap hit from others → `remains in queue; caps gate NEW inbound only`.
  - High-merit sender vs full queue → `not force-inserted over cap; priority waitlist position but never exceeds the hard cap (safety invariant, not a paywall)`.
  - Bulk-decline to clear queue → `allowed; declines silent to senders (no "rejected" shame)`.
  - Rank tie → `higher effort-score, then earlier timestamp`.
  - Dormant recipient with full queue → `interests expire (INT-05) and free capacity; also excluded by reachability (MATCH-02)`.
- **Acceptance criteria:** at pending==cap the profile doesn't appear in new-discovery/curated-picks for a not-yet-sent sender; rejected-because-full never decrements allowance; queue returns best-first deterministically; lowering the corridor cap never deletes a pending interest.
- **Dependencies:** INT-01/05/06, PRD §7.3 ranking, Domain 3 MATCH-02, BAL-02/06, MSG-01.

### INT-05: Interest expiry (~7 days)
- **Description:** Pending interests expire after ~7 days (configurable), returning inbound capacity — a *capacity-recovery* mechanism, not a punishment of the recipient.
- **Expected behavior:** each interest `expires_at` = created_at + window. On expiry: leaves the recipient's pending queue, decrements pending (frees a slot / re-opens discovery / admits a waitlisted sender), closes for the sender. Sender notified (neutral). Recipient NOT notified/penalized.
- **Edge cases & handling:**
  - Expires while recipient mid-composing a reply → `grace: extend expiry by ~24h or freeze while an action is in progress; a reply within grace still creates the mutual match — never lose a reply to a race`.
  - Becomes mutual before expiry → `expires_at cleared; converts to a thread; no longer pending`.
  - Oversubscribed recipient lets many expire → `zero penalty to her; pure capacity return`.
  - Re-send after expiry → `allowed within allowance/merit/caps; fresh interest (new note); max re-sends per window (default 1) to prevent nagging`.
  - Expiry window changed by config while interests live → `live interests keep original expires_at; only new ones use the new window`.
  - Women-initiate lane expiry → `same window on her opening interest awaiting his response; if he's oversubscribed, no penalty either`.
- **Acceptance criteria:** untouched interest removed + frees a slot at expires_at; a reply during grace still yields a mutual match; expiry produces a sender-side notification and never a recipient-side penalty/nudge.
- **Dependencies:** INT-04, INT-06, MSG-01, INT-02, INT-08.

### INT-06: Reply-fairness — sender-only nudges, never penalize the oversubscribed
- **Description:** The explicit PRD correction: PRD §7.4's reply-nudges and "your interest expires" prompts apply to **SENDERS ONLY**; an oversubscribed woman is **never** penalized for not replying. The fairness invariant that keeps the constrained side trusting the platform. Reply-parity still drives *ranking and admission* (BAL), not punishment of a drowning recipient.
- **Expected behavior:** expiry nudges go to the **sender**. Reachability-aware ranking rewards responsiveness, but a recipient's **oversubscription state suppresses any responsiveness-based demotion** (legitimately unable to reply to all). Response-SLA nudges to the supply side are gentle, opt-in-tunable, and **suppressed entirely when oversubscribed** — never a penalty/threat/visibility cut. Reply-parity gap feeds BAL (throttle the *abundant* side's demand), never degrade the scarce side.
- **Edge cases & handling:**
  - Oversubscribed recipient, many unreplied → `no nudge, no demotion, no penalty; senders get expiry notices instead`.
  - NON-oversubscribed chronic non-replier (has capacity, ignores everyone) → `gentle SLA nudges + mild reachability-label softening ("typically replies" downgrades); still no hard penalty; reachability ranking naturally sinks them. Penalty logic keys off the oversubscription flag, not reply-count alone`.
  - Toggles oversubscribed↔not as queue drains → `nudge-suppression follows state in real time`.
  - Sender "she never replied, refund me" → `handled by reachability guarantee/first-reply floor (REACH-03), not by penalizing her; guardrailed by sender's profile-strength + effort`.
  - System tries to auto-demote an oversubscribed woman via a generic responsiveness job → `guarded: any ranking/penalty job MUST check is_oversubscribed and exclude (enforced invariant, not a toggle)`.
  - Same-gender / women-as-abundant corridor → `invariant applies to whoever is the constrained/oversubscribed side; the flag is behavioral, gender-neutral (BAL-05)`.
- **Acceptance criteria:** no nudge/notification/ranking-change/penalty ever emitted while `is_oversubscribed==true`; reply-expiry nudges exclusively to senders; a chronic non-replier *with free capacity* sinks in ranking but gets no hard penalty/visibility ban; the guard is enforced at the job/service layer (unit-tested), not just UI copy.
- **Dependencies:** INT-04 (oversubscription flag), Domain 3 MATCH-02, REACH-03, BAL-02/06, PRD §7.3/§7.4.

### INT-07: Merit surfacing & coaching hook
- **Description:** Makes INT-03 legible/actionable so the effort tax reads as fair — pairs quota with "profile-strength score + concrete fixes" and "a win that isn't a reply." Prevents "null and void" resentment.
- **Expected behavior:** sender sees current cap, merit tier, and top 1–3 concrete levers ("personalize notes → +reply rate," "your reply-rate is strong → +2 today"). Non-reply "wins" surfaced (views, shortlists, badge progress). Coaching never blames the recipient — framing is about the sender's own outreach quality.
- **Edge cases & handling:**
  - Throttled by governor, not merit → `surface balance explanation, not a "fix" (avoid implying user is at fault)`.
  - Zero replies because all targets oversubscribed → `coaching reflects honestly ("your matches are in high-inbound corridors"), no penalty; guarantee context (REACH-03) shown`.
  - Low profile-strength claims guarantee → `guardrailed by min profile-strength + effort; coaching nudges to meet the floor first`.
- **Acceptance criteria:** every throttled/near-cap sender sees an accurate reason (merit vs governor) + ≥1 actionable lever or honest context; no coaching attributes non-replies to a recipient's character.
- **Dependencies:** INT-03, BAL-02, REACH-02, REACH-03, PRD §7.3.

### INT-08: Women-initiate discovery lane
- **Description:** A Bumble-style, marriage-intent lane where **only she opens.** Contact mutual-interest-gated by default; "curated for you," not "browse the catalog." Inverts the flood at its origin.
- **Expected behavior:** in the lane, the woman is the sole sender; men appear as candidates she may open. Her opening interest carries the effort-taxed note and counts against her (separately governed) allowance. His acceptance → mutual chat. His inbound queue (INT-04) applies. May coexist with a general lane per config; a user may set "women-initiate only" as a first-run privacy/control preference.
- **Edge cases & handling:**
  - She sets women-initiate-only, a man tries to send → `blocked pre-delivery; his allowance not consumed; "this member chooses who reaches her"`.
  - Man in the lane is himself oversubscribed (supply-heavy/inverted corridor) → `INT-04 caps his inbound; INT-06 protects him; mechanism is gender-neutral (BAL-05)`.
  - Same-gender preference user → `"initiate" role assigned by constrained-side/preference logic, not literal gender; symmetric-preference → both may open`.
  - She opens, he never responds, interest expires → `sender-side (her) expiry notice; no penalty to him unless he has free capacity and chronically ignores`.
  - She opens then blocks before he responds → `interest withdrawn from his queue; frees his slot`.
  - Non-binary / preference-any → `lane role via BAL-05 mapping; if ambiguous, default mutual-open`.
- **Acceptance criteria:** with women-initiate-only set, no inbound interest from a man is ever delivered to her; her opening interest enters his queue and on acceptance opens a mutual chat; the initiator role derives from preference/constrained-side mapping, correctly handling same-gender/non-binary.
- **Dependencies:** INT-02/04/06, MSG-01/02, BAL-05, PRD §7.2.

---

## MSG — Chat & Contact Reveal

### MSG-01: Mutual-interest chat
- **Description:** Chat opens only on **mutual interest** (recipient accepts/replies to a delivered, effort-taxed interest). Prevents unsolicited messaging; every conversation began with reciprocal consent.
- **Expected behavior:** on acceptance/reply, a thread is created seeded with the sender's original note as message 1; the interest leaves the pending queue (frees an INT-04 slot). In-chat safety inline (report/block, anti-harassment filters, no contact-info leakage before consent, MSG-02). Both parties send freely; typing/read states per config.
- **Edge cases & handling:**
  - One party blocks mid-chat → `thread frozen; silent block — blocked party sees no explicit "blocked" state (messages stop delivering/thread archives); no further sends; no un-revealed contact data remains accessible`.
  - One party deletes account mid-chat → `thread closed with neutral copy; no orphaned PII; already-revealed real-world contact can't be recalled but the platform thread is tombstoned; erasure honored`.
  - Both block each other → `thread hard-closed; identity-linked so neither reappears on re-registration`.
  - Abuse reported in chat → `flagged to T&S; sender may be muted in-thread pending review; confirmed abuse penalizes merit, may suspend (identity-linked)`.
  - Interest was expiring but replied within grace → `chat still opens`.
  - Recipient replies to decline politely → `decline does NOT open a persistent chat; closes the interest silently unless recipient explicitly opens a thread (avoids forcing unwanted conversation)`.
  - Contact-info shared before consent-reveal → `auto-filtered/masked (MSG-02)`.
- **Acceptance criteria:** no thread without a prior mutual (delivered interest + acceptance); sender's note is message 1; blocking is silent and immediately stops delivery; account deletion tombstones the thread without leaking PII.
- **Dependencies:** INT-02/04/05, MSG-02, PRD §7.4/§7.5/§9.

### MSG-02: Contact reveal gated on mutual consent (not payment)
- **Description:** Real contact details reveal ONLY on **mutual consent of both parties**, never on payment — the direct rejection of the incumbent's contact-unlock-volume monetization ("20/50/150/500"). We monetize reachability, not contact dumps.
- **Expected behavior:** two-sided handshake — A requests/offers reveal → B must explicitly consent → contact fields exchange. Either alone insufficient. Before reveal, chat auto-masks contact-info patterns. Reveal independent of tier (a paid user can't unlock contact without the other's consent). Per-field granularity possible (reveal phone but not employer), tied to PRIV-02; employer masked until reveal.
- **Edge cases & handling:**
  - A consents, B never does → `no reveal; A's contact not unilaterally exposed; A can withdraw the offer anytime`.
  - Both consent then one blocks → `future messaging stops (silent block); already-revealed real-world contact can't be recalled but the platform stops facilitating; if abuse, T&S + identity-linked ban`.
  - Contact-reveal then off-platform harassment/extortion → `in-app escalation lane for threats/extortion; report actioned within SLA + status told back; identity-linked ban`.
  - Leak contact pre-consent via note/chat/photo caption/voice note → `auto-filter across all channels incl. transcribed voice and image OCR; repeat → sanction`.
  - Paid user assumes payment unlocks contact → `UI explicitly states reveal is consent-gated, not purchasable`.
  - Minor/underage attempt → `blocked upstream by age-gating; no reveal path`.
  - Contact exchange across regions → `respects data-residency routing + lawful cross-border mechanisms`.
- **Acceptance criteria:** contact fields never transmit without both explicit consents recorded (auditable); no tier/payment triggers reveal absent mutual consent; pre-consent contact info masked in any channel (text/voice/image); post-reveal block halts platform-facilitated contact + enables escalation.
- **Dependencies:** MSG-01, PRIV-02, PRD §7.5/§9, monetization (tiers don't gate consent).

---

## REACH — Reachability Transparency & Guarantee

### REACH-01: Reachability labels
- **Description:** Per-profile honest signals — "Active this week," "typically replies," "high inbound ~15% reply." Answers the "null and void" broken-promise wound; sets expectations *before* a user spends an interest.
- **Expected behavior:** each discoverable profile shows labels from last-active recency, historical reply-rate, current inbound load/oversubscription. Labels: activity, responsiveness, and honest load disclosure for high-inbound profiles so senders self-select. Discovery ranking prioritizes active/responsive/verified. Descriptive, never shaming — an oversubscribed woman's "high inbound" label frames *her desirability + your realistic odds*, not her failure.
- **Edge cases & handling:**
  - Oversubscribed woman → `"high inbound / lower reply odds," and removed from new discovery at cap (INT-04); no penalty framing`.
  - New user, no reply history → `neutral "New — no reply history yet"`.
  - Chronic non-replier with free capacity → `"typically slow to reply" + reachability rank sinks; still no hard penalty`.
  - Label would reveal exact inbound counts → `bucketed/rounded ("~15%," "high") — kills the incumbent's public "thousands of interests" signal`.
  - Stale activity data → `recency server-side, refreshed on session`.
  - Incognito/hidden browsing → `browsing may not bump public "active" label if privacy suppresses activity broadcast (config)`.
- **Acceptance criteria:** every discoverable profile renders ≥ activity + responsiveness signal; inbound-load figures bucketed, never exact; oversubscribed profiles labeled non-punitively and absent from new-discovery at cap; dormant profiles excluded or explicitly labeled.
- **Dependencies:** INT-04/06, PRD §7.3, PRD §7.2 incognito.

### REACH-02: Personal reachability dashboard
- **Description:** Private first-person reachability data. For men, reframes scarcity as legible odds + momentum ("a win that isn't a reply"); the honesty instrument answering the broken-promise problem before churn.
- **Expected behavior:** shows outbound sent, replies received, reply-rate vs corridor norm, mutual matches, profile views/shortlists (non-reply wins), profile-strength + fixes (INT-07), current cap + merit levers (INT-03), reachability-guarantee counter (REACH-03), corridor-health context. Distinguishes throttle reasons (merit vs governor). Honest expectation-setting language.
- **Edge cases & handling:**
  - Red-line-throttled corridor, low replies → `attribute to balance/throttle + guarantee status, not user failure; guarantee counter accruing`.
  - Low replies because targets were oversubscribed → `reflects honestly; excluded from self-blame framing + from merit`.
  - Could expose another user's data → `strictly first-person; others appear only as aggregate/bucketed norms`.
  - New user, sparse data → `onboarding-state guidance, not misleadingly empty metrics`.
- **Acceptance criteria:** shows own reply-rate, non-reply wins, cap/merit levers, guarantee counter; no other individual's private metrics exposed (norms aggregate only); throttle reason accurately labeled.
- **Dependencies:** INT-03/06/07, REACH-01/03, BAL-02, PRD §7.3.

### REACH-03: Reachability guarantee (billing pause / credit)
- **Description:** Outcome-linked commitment: ≥8 verified reachable matches/month; miss → paid time auto-extends free with a visible counter. Plus a **first-reply floor**: zero genuine replies to quality outreach in a cycle (where *we* under-served) → cycle credited, guardrailed by min profile-strength + effort. Converts the biggest risk into a bounded liability controlled by throttling intake.
- **Expected behavior:** track per paid cycle: verified-reachable-matches surfaced, quality-outreach count, genuine replies, profile-strength. Reachable-matches < threshold (default 8/mo) AND platform under-served → auto-extend paid time free with a visible counter + reason. Zero genuine replies AND platform under-served AND guardrails met → credit the cycle. Credits non-cash by default (extended access); cash refund only where policy states.
- **Edge cases & handling:**
  - Guarantee triggered because corridor is demand-heavy/throttled → `honored (this is the bounded liability the throttle controls; BAL-06 slows intake so few users trigger it)`.
  - Zero replies but user didn't meet effort/profile-strength floor → `not credited; dashboard shows unmet guardrail + coaching (transparent, not silent denial)`.
  - Zero replies caused by targeting only oversubscribed women → `counts as "we under-served" (platform allowed unreachable targets) → eligible; INT-06 already bars penalizing her`.
  - Games with minimum-viable outreach → `effort/quality gates + merit + min profile-strength block low-effort farming; repeated gaming flagged`.
  - Cancels mid-cycle → `pro-rated per transparent refund policy; cooling-off honored`.
  - "Reachable-match" definition dispute → `= mutual match with a verified, currently-reachable peer (not a dormant profile); counter uses this; auditable`.
  - Platform over-delivers → `no clawback; guarantee one-directional (user-favorable)`.
  - Corridor breaches red line and can't deliver → `guarantee auto-triggers AND BAL freezes male intake + pours budget into supply; guarantee is the user-facing net while BAL fixes the market`.
- **Acceptance criteria:** distinguishes "matches surfaced" (platform SLA) from "replies received" (not guaranteed) in logic + UI; breaches auto-extend/credit with a visible counter + reason (no manual claim for the automatic SLA); credit enforces profile-strength + genuine-effort guardrails; interacts correctly with pause/refunds/concierge (no double-remedy).
- **Dependencies:** REACH-01/02, INT-02/03 guardrails, BAL-02/06, PRD §7.7, §9, success metric (≥8 reachable/mo).

---

## BAL — The Two-Sided Balance Engine

### BAL-01: Gender-gated admission valve
- **Description:** The master lever. Men admitted from the waitlist by a **daily quota computed from active female supply**; women skip the waitlist (instant, ~zero friction). Inverts the incumbent, where demand set the pace — here supply capacity sets the pace. `men_admitted = k × net_active_female_growth − ratio_correction` (k≈1.2). Male priority earned by verification depth + completeness + intent, not FCFS.
- **Expected behavior:** verified women → admitted instantly, no waitlist. Verified men → waitlist (framed as curation: "we admit in balance so you actually get replies"). Daily male admission per corridor per the formula, floored at 0, capped by the ratio governor (BAL-02). Waitlist ordering = male priority score. Admitted men transition to full product.
- **Edge cases & handling:**
  - `net_active_female_growth ≤ 0` → `male admission = 0 (or only churn-replacement per config); pour into supply (BAL-06 Mode A)`.
  - Waitlist grows large, men frustrated → `transparency: position context + honest expectation + reachability messaging; pressure valve is intentional, framed as curation`.
  - High-priority man vs full valve → `even top-priority men wait if the day's quota is 0; priority orders within the admitted set, never overrides the ratio ceiling`.
  - Woman signs up then goes dormant (seeded-ghost risk) → `net_active_female_growth counts ACTIVE/reachable growth, not raw signups; a dormant new woman doesn't open male admission`.
  - Corridor mismatch → `admission computed per the man's corridor; relocation-overlap (BAL-04) may place him in an adjacent pool`.
  - Same-gender / non-binary → `"gender for the valve" is the market-balance attribute (BAL-05); meters the abundant side of the relevant matching market`.
  - Male-signup surge from marketing → `hits the waitlist, not the product; marketing can't override the valve`.
- **Acceptance criteria:** verified women never on the admission waitlist; daily male admissions per corridor never exceed the computed quota; male waitlist ordering reflects priority not signup time; `net_active_female_growth` uses active/reachable counts.
- **Dependencies:** PRD §7.1, BAL-02/04/05, REACH (waitlist transparency), PRD §13.

### BAL-02: Ratio governor — auto-throttle, red line 1.5:1
- **Description:** Closed-loop controller governing the **active-user discovery-pool ratio** (target 1.0–1.2 M:1 F; alert >1.4:1; **hard red line 1.5:1 halts male admission**). Throttles male intake before the red line, and throttles demand caps when female inbound load runs hot (>10/wk). Rebalances automatically before manual intervention.
- **Expected behavior:** continuously compute per-corridor active M:F ratio + female inbound load/wk. 1.0–1.2 → normal admission. >1.2→1.4 → tighten (reduce valve quota, apply `ratio_correction`, optionally tighten INT-01). ≥1.4 → alert + aggressive tightening. **≥1.5 (red line)** → halt male admission for the corridor; trigger Mode A (BAL-06); reachability guarantee becomes the backstop. Female inbound load >10/wk → throttle demand now (lower INT-01/tighten INT-04) regardless of headline ratio. Output feeds INT-01, BAL-01, surfaced honestly (REACH-02).
- **Edge cases & handling:**
  - Red line breached → `zero new male admissions; budget → female supply (BAL-06 Mode A); no degradation of the scarce side`.
  - Ratio healthy but inbound load hot → `throttle demand via caps even though ratio looks fine (leading indicator: inbound/active-woman >5–7/wk → tighten NOW)`.
  - Supply-heavy/thin (ratio <1.0, Mode B) → `OPEN valve, targeted demand acquisition, widen via relocation overlaps; never fix by degrading supply`.
  - Oscillation → `hysteresis bands + rate-limited, damped adjustments (no whipsaw)`.
  - Marketing/ops override attempt → `governor authoritative; throttle can't be manually overridden to breach the red line`.
  - Ratio on stale/dormant users → `active/reachable definitions only; dormant excluded from numerator + denominator`.
  - Corridor too small for stable ratio → `below min-population threshold, conservative defaults + human review`.
  - Same-gender segment ratio → `computed within the matching market via BAL-05, not naive headcount`.
- **Acceptance criteria:** at ratio ≥1.5 in a corridor, male admissions = 0 until recovery; governor tightens before the red line (1.2/1.4) without manual action; hot female inbound load triggers demand-cap tightening even within band; no manual override admits men past the red line.
- **Dependencies:** BAL-01, INT-01/04, BAL-04/05/06, REACH-02/03, PRD §10/§11.

### BAL-03: Reciprocity-weighted curated intros (anti-flood surface)
- **Description:** The primary discovery surface is a **daily curated few** high-compatibility candidates, two-sided + reachability-aware, **reciprocity-weighted** — a woman shown to a man only if she's plausibly also interested — capping inbound at the source. "5 curated reply-worthy people, never 400 pending."
- **Expected behavior:** small daily set (default 5–7, tier-configurable) of reachability-scored, compatibility-ranked candidates. Reciprocity weighting: surfaced to a sender only if two-sided plausibility (mutual compatibility + recipient not oversubscribed + recipient plausibly interested) clears a threshold. Respects INT-04 (never surface a saturated recipient to a new sender) + REACH-01 labels. Larger allowance for higher tiers.
- **Edge cases & handling:**
  - Recipient oversubscribed → `excluded from others' curated picks (the primary lever keeping inbound finite)`.
  - Thin corridor, not enough reciprocal candidates → `show fewer honest picks; never pad with unreachable/low-compat; may widen via relocation overlap`.
  - Demand-heavy corridor → `men's curated set skews toward genuinely reachable women only; volume may shrink (protect supply)`.
  - Exhausts daily picks → `upsell to a higher curated allowance or wait; no infinite scroll catalog`.
  - Reciprocity cold-start (new user) → `fall back to compatibility + reachability until reciprocal signal accrues; never surface saturated recipients`.
- **Acceptance criteria:** every user's primary surface is a bounded daily curated set, not an unbounded catalog; oversubscribed + unreachable-geography profiles never appear in another user's picks; thin corridors shrink the set rather than pad.
- **Dependencies:** INT-04, REACH-01, BAL-02/04, PRD §7.3 (Domain 3 DISC-01).

### BAL-04: Corridor liquidity unit & relocation-overlap matching
- **Description:** Matchmaking is a *local* liquidity business wearing an *international* brand. The unit of liquidity is the **corridor** (home-metro + diaspora cluster + relocation-willingness overlap), not country/globe. All caps/ratios/valves/health metrics computed per corridor; relocation-willingness is a first-class matching input. Rule: optimize for the smallest geography that still gives a new joiner ~5 reply-worthy candidates this week.
- **Expected behavior:** users assigned to one or more corridors from home-metro + diaspora + relocation overlaps. BAL-01/02/05/06 operate per corridor. Discovery/curation draws only from within-corridor reachable candidates; relocation overlap bridges two corridors when both parties are relocation-willing. Never pad discovery with unreachable-geography profiles.
- **Edge cases & handling:**
  - User in multiple corridors → `counted for health metrics in each with de-dup to avoid double-inflating supply`.
  - Corridor too thin → `merge with adjacent via relocation overlap, or pause/quarantine (BAL-06 Mode B); don't backfill unreachable`.
  - Relocation-willing man, home-resident woman → `matched via overlap only if both express compatible willingness; one-sided willingness doesn't force a cross-corridor surface`.
  - Corridor boundaries shift as density grows → `"smallest geography with ~5 reply-worthy/wk" is dynamic; definitions recomputed periodically; users notified transparently`.
  - Global-only preference (no metro tie) → `widest-overlap corridor but still bounded by reachability; never "global premium" (density → zero)`.
- **Acceptance criteria:** ratio/valve/caps/health KPIs computed + enforced per corridor, not globally; cross-corridor surfacing only under mutual relocation-willingness overlap; discovery never includes unreachable-geography profiles.
- **Dependencies:** BAL-01/02/03/05/06, PRD §7.3 (relocation input), §7.6 (i18n).

### BAL-05: Orientation-aware valve mapping (non-binary & same-gender)
- **Description:** The valve/governor are described in male/female terms for the launch wedge, but the *mechanism* meters the **abundant side of a matching market against the constrained side** — must handle non-binary and same-gender-preference users without mis-gating.
- **Expected behavior:** each user's **valve-role** in a market = f(gender identity, partner preference). The system partitions each corridor into matching markets (men-seeking-women, women-seeking-men, same-gender, any-gender) and computes supply/demand + ratio **within each market**. The "constrained side" is determined empirically by active/reachable supply + inbound load, NOT assumed from gender labels. Admission metering (BAL-01) and throttling (BAL-02) apply to whichever side is abundant in that specific market.
- **Edge cases & handling:**
  - Same-gender-preference user (e.g., man-seeking-man) → `placed in that matching market; constrained side determined there; valve meters the abundant side; waitlist/instant-admit follows constrained-side status, not "is female"`.
  - Non-binary user → `assigned market(s) by their preference + who's seeking them; may belong to multiple; metrics de-duped`.
  - Preference = any/all → `participates in multiple markets; admission/caps evaluated against the most-constrained relevant market to avoid over-admitting into a thin side`.
  - Market too thin to compute a stable constrained side → `conservative defaults + human review; never mis-apply the heterosexual assumption`.
  - INT-06 fairness invariant → `keys off the behavioral is_oversubscribed flag (gender-neutral), so protection applies to any oversubscribed user in any market`.
  - Reporting/KPIs → `computed per matching market, rolled up per corridor for ops`.
- **Acceptance criteria:** admission/throttling for a same-gender-preference user governed by that user's matching-market supply/demand, not a literal field; the oversubscription-protection invariant applies to any oversubscribed user regardless of gender/orientation; no user placed on/skipped from the waitlist based on gender label alone when their market's constrained side differs.
- **Dependencies:** BAL-01/02/04, INT-04/06, PRD NG3, §7.1/§7.3.

### BAL-06: Out-of-balance playbook & closed-loop rebalancing
- **Description:** The automated policy layer that reacts to corridor imbalance before humans do. Encodes **Mode A (demand-heavy, common)** and **Mode B (supply-heavy/thin, rare)** and the standing rules: rebalance automatically before manually; always fix the ratio by adjusting the *abundant* side's throughput, never degrade the scarce side; quarantine broken corridors.
- **Expected behavior:** continuously evaluate corridor health (active ratio, reply-parity gap, male first-reply rate, female inbound load, time-to-first-mutual, active/verified ratio, female D14 retention, mutual matches/active/wk). **Mode A:** slow valve → tighten demand caps → protect/enrich supply → reactivate dormant women → if still red, freeze male admission + pour budget into female supply. *Never add demand.* **Mode B:** open valve → targeted demand acquisition → widen via relocation overlaps → if density unreachable, merge/pause corridor. Escalate to humans only when automated levers are exhausted or a corridor is quarantined. "Wake-me-at-3am": reply-parity gap + female D14 retention breach page a human.
- **Edge cases & handling:**
  - Demand-heavy, tempted to acquire more men → `forbidden; Mode A never adds demand; marketing throttle enforced`.
  - Supply-heavy → `Mode B opens valve + demand acquisition; if still can't reach ~5 reply-worthy/joiner/wk, merge or pause`.
  - Reply-parity gap >12 pts → `intervene: reachability-ranking + admit reply-worthy men + tighten demand; NEVER penalize the scarce side`.
  - Female D14 retention <45% → `aggressive demand throttle + supply protection; page humans`.
  - Time-to-first-mutual >5 days → `Mode B / merge corridor`.
  - Broken corridor (incoherent metrics / possible fraud cluster) → `quarantine: pause admissions, T&S review, isolate from healthy corridors`.
  - Controller oscillation across modes → `hysteresis + minimum dwell time per mode; damped adjustments`.
  - Guarantee liabilities spike → `signals demand-heavy under-service; Mode A throttle reduces future liability; guarantee honored`.
  - Automated levers exhausted, still red → `escalate to human ops with full context; freeze intake as the safe default`.
- **Acceptance criteria:** a demand-heavy corridor never rebalanced by acquiring more demand; ratio corrections always act on the abundant side's throughput, scarce side never degraded/penalized; parity-gap + female-retention breaches trigger automated intervention + a human page; corridors failing density merged/paused, not padded.
- **Dependencies:** BAL-01/02/03/04/05, INT-01/04/06, REACH-03, PRD §10/§11/§13.
