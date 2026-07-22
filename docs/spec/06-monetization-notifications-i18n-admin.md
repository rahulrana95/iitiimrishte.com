# Feature Spec — Monetization, Notifications, Internationalization & Admin/Ops

> Domain 6 of the [Feature Specification](../FEATURE-SPEC.md). Companion: [PRD](../PRD.md) §7.6–§7.9/§9–§11, [male brief](../gtm/raw/03-male-acquisition-monetization.md), [business-model brief](../gtm/raw/08-business-model-fundraising.md).

**Incumbent gaps this domain closes:** opaque/flat over-pricing (~₹26k–33k for a thin pool, "null and void"); opaque refunds + dark-pattern renewal; no reachability/outcome guarantee; India-only pricing & UI; no honest fairness/active metrics. The money layer must be *radically transparent* because the wound is broken-promise, not just price.

---

## Subscription Tiers & Paywall (SUB-)

### SUB-01: Subscription tier model & entitlement engine
- **Description:** The canonical 4-rung revenue ladder as versioned entitlement sets: **Free — Verified Browse** (ID + 1 automated V0 signal, browse real pool w/ reachability labels, blurred media, 3 interests/week), **Premium**, **Elite/Plus** (deeper badges, priority ranking, larger curated allowance, monthly review, reachability guarantee), **Concierge/Assisted** (RM — SUB-05). Every gated capability is an *entitlement key*, never a hardcoded tier check, so tiers can be repriced/re-scoped per market + A/B-tested. Counters the incumbent's single flat over-priced SKU.
- **Expected behavior:** each account has one *active plan* + zero-or-more *add-ons*; entitlements = union of plan grants + active credits. Checks evaluated server-side on every gated action; client receives a capability manifest at session start + on plan change (push-invalidated). Tier definitions versioned; a user keeps their purchased snapshot (grandfathering) until renewal. Free tier permanent, not a trial. Deep/human verification (V1, ₹300–700) is **payment-gated** — only Premium/Elite/Concierge trigger it; Free stays on automated V0. Entitlement engine gates the *verification-depth request*, not just features.
- **Edge cases & handling:**
  - Downgrades but had grandfathered entitlement → `retains until period end, then snaps to the CURRENT catalog definition of the new tier (never a stale better one)`.
  - Entitlement key unknown to server (version skew) → `deny by default (fail-closed), return capability_stale, force manifest refresh`.
  - Free hits 3-interest weekly cap → `block 4th, upgrade CTA + reset timestamp in user's timezone; never silently drop`.
  - Concierge user's underlying self-serve plan lapses → `Concierge entitlements independent; engagement continues per contract`.
  - Add-on/credit outlives base plan → `credit remains redeemable for its stated window`.
  - A/B assigns two catalog variants → `user pinned to one variant for the subscription's life; renewal re-evaluates`.
- **Acceptance criteria:** no gated action succeeds without a server-side entitlement check; changing a tier's public scope doesn't retroactively remove entitlements paid for within the current period; Free can browse the real pool + see reachability labels but can't message beyond 3/week; deep verification never invoked for a Free-only account.
- **Dependencies:** Verification (V0/V1 depth), Discovery/reachability labels, PAY-01, ADMIN-06 RBAC, PRD §7.1/§7.3.

### SUB-02: Paywall & gating enforcement
- **Description:** Presentation + enforcement converting entitlement denials into transparent upgrade moments — never dead-ending a paying user against dormant profiles; paywall leads with *reachability value*, not feature lists.
- **Expected behavior:** soft gate (preview + CTA) for discovery/curation; hard gate (blocked) for messaging beyond Free cap + contact-reveal. Every paywall shows full price, currency, renewal date/amount, refund terms *before* any charge (one screen). Reachability context injected ("Active this week") to justify unlock, aligning price to the reply, not outreach volume.
- **Edge cases & handling:**
  - Entitlement revoked mid-session (refund/chargeback/downgrade) → `next gated action re-checks + gates; in-flight composed message preserved as draft`.
  - Market with no active price → `paywall hidden; "not yet available in your region," no broken checkout`.
  - Paywall shown to an already-entitled user (stale client) → `action proceeds after server re-check; no double-charge prompt`.
  - Deep-link into a gated profile from a notification → `respects gate + reachability label; never leaks gated content`.
- **Acceptance criteria:** price + renewal + refund terms visible on the same screen before charge for 100% of paid conversions; no paywall presents a purchase the server would reject; drafts survive an entitlement change.
- **Dependencies:** SUB-01, PAY-03, PAY-04, Discovery reachability, Notification deep-links.

### SUB-03: Upgrade / downgrade / proration
- **Description:** Plan-change engine with fair, transparent proration — fixing the incumbent's tiny 6mo↔annual gap by making change math visible.
- **Expected behavior:** **Upgrade** immediate; unused current-plan value credited pro-rata against the new plan, difference charged now. **Downgrade** scheduled for period end (no immediate charge/refund; keeps higher entitlements until then). **Billing-cycle change**: quarterly is the featured default; switching applies at renewal unless the user opts to true-up now. All proration math itemized on-screen + in the receipt before confirm.
- **Edge cases & handling:**
  - Upgrade during an active reachability-guarantee credit window → `unused guarantee credit carries to the new plan; threshold recomputes to the new tier's SLA`.
  - Downgrade would strip an entitlement mid-use (active concierge) → `block downgrade of the concierge component until a milestone boundary; allow self-serve downgrade independently`.
  - Currency changed since purchase (relocated) → `proration in the ORIGINAL purchase currency; new plan billed in the new locale currency; no hidden FX gain (PAY-07)`.
  - App-store-billed upgrade → `store governs proration; platform mirrors + doesn't double-prorate; steer future purchases to web`.
  - Upgrade then cooling-off refund → `refund reverses the incremental charge + restores prior plan snapshot`.
  - Multiple changes within one period → `each re-bases proration off current remaining value; ledger append-only`.
- **Acceptance criteria:** upgrade credits unused time, downgrade never charges immediately, both show itemized math pre-confirm; no plan change results in paying twice for overlapping coverage; downgrade can't orphan an in-progress concierge milestone.
- **Dependencies:** PAY-03/06/07, SUB-05, PAY-08.

### SUB-04: Pause-before-cancel (pause > cancel)
- **Description:** Retention lever: offer *pause* ahead of cancel + graceful "found someone" offboarding — reduces "good churn" into CAC-deflating exits.
- **Expected behavior:** cancel flow first offers (a) **Pause** (freeze billing + entitlements 1–3 months), (b) **"I found someone"** → success-story opt-in + referral prompt, (c) proceed to cancel. Pause halts the paid clock; on resume, remaining paid days honored. Pause suppresses discovery visibility + notifications by default; user opts back in on resume.
- **Edge cases & handling:**
  - Pause with an active guarantee counter → `guarantee clock pauses; on resume continues from the same count, not reset`.
  - Pause window elapses with no resume → `auto-resume OR auto-transition to Free per the user's pre-set choice; never silently keep charging; notify before pause ends`.
  - Pause then refund → `refund on unconsumed paid time excluding paused days`.
  - "Found someone" while concierge active → `offboarding pauses RM outreach + prompts milestone reconciliation, not an instant cancel`.
  - Pause on an app-store subscription → `if store lacks native pause, emulate by cancelling store auto-renew + issuing a web-side pause credit; reconcile`.
  - Serial-pause abuse (to freeze a guarantee) → `cap total pause days per 12-month period; excess converts to cancel with normal proration`.
- **Acceptance criteria:** cancel intent always surfaces pause + "found someone" before final cancel; paused time never consumes paid days or advances the guarantee counter; no charge during pause; resume/expiry matches the user's pre-set choice.
- **Dependencies:** SUB-01, PAY-04/05/06/08/09, Discovery visibility, success-story mgmt (Domain 5/ADMIN).

### SUB-05: Concierge / assisted engagement (RM, milestones, success-fee)
- **Description:** Tier 3 human RM engagement with *published, contractual* scope + milestones — the margin engine that subsidizes verification infra. Two shapes: flat (₹1,50,000/6mo, $4,000/3mo) or **base + success-fee** (₹75,000 + ₹51,000; $2,500 + $2,500) triggering only on verified engagement/marriage.
- **Expected behavior:** on purchase a **contract object** is created (scope, milestone schedule, per-milestone refund schedule, assigned RM, currency, success-fee terms). Milestones (onboarding call → shortlist delivered → N curated intros → verified engagement) each have definition, target date, evidence requirement, refundable amount if unmet. RM works the engagement via Concierge CRM (ADMIN-03); user sees a milestone tracker mirroring CRM state. **Success-fee** invoiced only on an evidence-verified outcome, never speculatively.
- **Edge cases & handling:**
  - Milestone missed (platform fault) → `refundable amount auto-credited per schedule; user notified with reason`.
  - Milestone disputed → `ADMIN-04 case; evidence = curated-intro reachability logs + RM notes; resolution credits or upholds; SLA-bounded`.
  - Success-fee claimed but outcome unverifiable → `not charged until verification; contract defines a good-faith attestation clause (no invasive proof)`.
  - Cancels mid-engagement → `pro-rated by milestones completed, not calendar time; delivered intros non-refundable, undelivered milestones refunded`.
  - RM leaves/reassigned → `engagement transfers with full CRM history; milestone dates hold; user notified`.
  - Guarantee interacts with concierge → `concierge SLA supersedes the automated self-serve guarantee; no double-crediting`.
  - Purchased via app store → `high-ticket concierge is web-only checkout to avoid the 20–30% store cut; app never sells concierge SKUs`.
  - Success-fee currency vs base currency mismatch after relocation → `honored in the contract's original currency`.
- **Acceptance criteria:** every concierge purchase creates a contract with published scope + milestone + refund schedule visible before purchase; success-fee charged only on evidence-verified outcomes; missed platform-fault milestones auto-credit per schedule; disputes resolve within SLA with an audit trail.
- **Dependencies:** ADMIN-03/04, PAY-03/06/08, Discovery/curation, PRD §7.9, §10 concierge-milestone KPI.

---

## Payments, Pricing & Billing (PAY-)

### PAY-01: Regional multi-currency pricing engine
- **Description:** Locale-aware price catalog in ₹/$/£/AED/SGD (extensible) with market-*anchored* SKUs (not FX-converted-from-INR) — India ₹1,999/mo hits the "10k" annual anchor ~66% under incumbent; US $39 reflects West ARPU 4–6× India. Kills India-only flat pricing.
- **Expected behavior:** price = f(market, tier, billing cycle) from a versioned catalog; reference FX ($1=₹85=£0.79=AED3.67=SGD1.34) is a *seed* for new markets, not a live per-charge conversion. Market determined by priority chain: billing country (payment instrument/tax ID) > verified residence > IP/locale, user can see + (where compliant) confirm. Real 6mo↔annual gaps enforced (annual ≈37% off monthly). Tax-inclusive or -exclusive display per local norm (India GST-inclusive; US tax at checkout — PAY-07).
- **Edge cases & handling:**
  - IP says UK, card says India → `billing-country (card) wins the charge; user warned if displayed vs billed market differ; no silent arbitrage`.
  - VPN/mismatched signals → `fall back to payment-instrument country; flag market-hopping for ADMIN review`.
  - No localized price yet → `product hidden in that market (SUB-02), not shown in USD fallback`.
  - Relocates mid-subscription → `current term unchanged; renewal re-resolves market + may reprice, disclosed at renewal reminder`.
  - Catalog price change while on the paywall → `price locked to the quote token issued at render (short TTL); expired quote re-fetches + re-discloses before charge`.
  - Rounding/psychological points → `explicit rounded price points per market; never raw FX decimals`.
- **Acceptance criteria:** IN/US/UK/UAE/SG users see a market-anchored local-currency price, not a live-FX conversion; annual shows a real material discount vs monthly in every market; displayed = billed price (via quote token), inclusive of the disclosed tax treatment.
- **Dependencies:** PAY-03/07, I18N-02, ADMIN-04, PRD §7.6.

### PAY-02: App-store markup steering to web
- **Description:** App-store SKUs priced +20–30% to recover the store commission + steer purchase to web where margin is full. High-ticket Elite-annual/Concierge are **web-only.**
- **Expected behavior:** two SKU sets per market — **web** (list) and **app-store** (list ×1.2–1.3, rounded; a superset-minus with concierge excluded). In-app, where the store permits external-purchase messaging, show web price; otherwise present the store price without prohibited steering language, keeping web the cheaper canonical channel. Entitlements channel-agnostic (a web plan unlocks the app identically).
- **Edge cases & handling:**
  - iOS subscribe then wants cheaper web billing → `allow web subscribe to start at store-term end; never double-bill overlapping periods; reconcile via receipt validation`.
  - Store refund out-of-band → `webhook/receipt re-validation revokes entitlement to match; ledger records store-origin refund; no platform double refund`.
  - Store subscription state drift → `periodic receipt re-validation is source of truth for store-billed plans; entitlement follows store state`.
  - App-store vs web billing conflict (active on both) → `detect duplicate active plans; auto-pause the newer/duplicate, credit overlap, keep one; notify which channel remains`.
  - Concierge attempted in-app → `not a store SKU; app deep-links to web checkout (compliant)`.
  - Family-shared store purchase → `entitlement binds to the purchasing store ID mapped to one platform account; sharing doesn't multiply entitlements`.
- **Acceptance criteria:** app-store SKU ≥20% above web for the same tier/market; no user holds two concurrently-billing active plans across channels without an overlap credit; store-originated refunds/cancellations revoke entitlements via receipt re-validation.
- **Dependencies:** PAY-01/03/06, SUB-01, store receipt-validation service.

### PAY-03: Checkout & billing
- **Description:** Region-appropriate checkout (India UPI/cards/netbanking with RBI e-mandate; West cards/wallets; UAE/SG local rails) with a single pre-charge disclosure screen.
- **Expected behavior:** states `quote → consent (renewal + refund shown) → authorize → captured → active`, plus `failed`, `pending` (async UPI/bank), `refunded`, `charged_back`. Idempotent charge creation keyed by quote token. Successful capture provisions entitlement synchronously; async methods provision on webhook confirmation with a "pending" UI. Immutable append-only billing ledger per account (charges, credits, refunds, proration, guarantee credits).
- **Edge cases & handling:**
  - Authorized but capture/provisioning fails → `auto-void or refund the auth; user sees failure, never a silent charge without access`.
  - Duplicate submit / retry → `idempotency key returns the original result; no second charge`.
  - Async method (UPI collect) times out → `quote expires, no charge, user re-initiates; pending auto-resolves`.
  - 3DS/SCA abandoned → `stays failed, no entitlement, retry with fresh quote`.
  - RBI e-mandate limit / not registered for recurring → `recurring above cap requires per-charge approval; renewal reminder prompts re-auth rather than failing silently`.
  - PSP outage → `queue + retry with backoff for renewals; new purchases show "try another method"; never grant unpaid entitlement`.
  - Partial capture / currency mismatch from PSP → `reject + refund; ledger flags for ADMIN-04`.
- **Acceptance criteria:** no successful charge without the pre-charge disclosure consent event logged; double-submit never produces two charges; entitlement granted only on confirmed capture (async pending explicit); every money movement writes an immutable ledger entry.
- **Dependencies:** PAY-01/04/07, PSPs, SUB-01, ADMIN-04.

### PAY-04: Renewal transparency & explicit consent (no forced auto-renew)
- **Description:** Answers opaque/dark-pattern renewal complaints — auto-renew is **opt-in**, disclosed, reversible.
- **Expected behavior:** auto-renew defaults per legal region but is always an explicit, separately-consented checkbox at checkout — never pre-ticked where DPDP/GDPR unbundled-consent applies. Renewal date + exact amount + currency shown at purchase + re-shown in an early reminder (PAY-05). User can toggle auto-renew off any time; plan runs to period end + stops. No silent price increases — any renewal price change requires re-consent before it takes effect.
- **Edge cases & handling:**
  - Auto-renew off, period ends → `entitlement drops to Free at period end; no charge; win-back prompt (not a dark pattern)`.
  - Renewal price higher than original → `requires affirmative re-consent; absent consent, does not renew at the new price (fails safe to non-renewal, not a surprise charge)`.
  - Currency/market changed before renewal → `new market price disclosed in reminder; user must acknowledge; else non-renewal`.
  - Mandate/card expired at renewal → `reminder prompts update; grace window then non-renewal, never indefinite retry-charging`.
  - Region bans auto-renew / requires reminder N days prior → `reminder cadence region-configured; engine enforces the strictest applicable rule`.
  - User claims they never consented → `consent event (timestamp, screen version, IP) retrievable from ledger for ADMIN-04 dispute`.
- **Acceptance criteria:** auto-renew never enabled without a discrete logged consent event; a renewal at a changed price/market never occurs without re-consent; turning off auto-renew is immediate, self-serve, lets the term run out.
- **Dependencies:** PAY-03/05/01, ADMIN-04, PRD §9 consent.

### PAY-05: Early renewal reminder & one-tap cancel
- **Description:** Proactive pre-renewal reminder + frictionless one-tap cancel.
- **Expected behavior:** reminder N days before renewal (region-configured, default ≥7d; ≥14d annual) via the chosen channel, containing date, exact amount, currency, + one-tap cancel/pause/manage. **One-tap cancel**: a single confirmed action cancels auto-renew (routes through the SUB-04 pause offer first, but cancel is one confirmation, no retention maze). Available in-app, web, and from the reminder deep-link with equal ease.
- **Edge cases & handling:**
  - All channels off → `reminder still delivered via a mandatory transactional channel (email fallback) — it's a legal/financial notice, not marketing (NOTIF-03 carve-out)`.
  - Quiet hours overlap the reminder window → `transactional billing reminders bypass quiet hours or send at the boundary so they precede renewal`.
  - One-tap cancel from a stale email link after already renewed → `shows current state, offers refund path (cooling-off) instead of a broken cancel`.
  - Cancel during PSP outage → `cancel intent recorded immediately (auto-renew flag off locally), reconciled when available; never charged for a period cancelled before`.
  - Cancel then re-subscribe within same period → `re-enabling auto-renew is a fresh consent; no gap in entitlement`.
- **Acceptance criteria:** every renewal preceded by a reminder with exact amount/date, even for users with marketing notifications off; cancel requires at most one confirmation after the pause offer (no multi-step wall); cancel intent honored even if PSP is temporarily unreachable.
- **Dependencies:** NOTIF-02/03/04, PAY-04/06, SUB-04.

### PAY-06: Refund policy & trackable refund status
- **Description:** Plain-language refund policy with cooling-off + a **trackable refund status** — answering opaque-refund complaints.
- **Expected behavior:** policy stated plainly pre-purchase (cooling-off 7–14 days region-set, pro-ration rules, non-refundable items like delivered concierge intros, guarantee-credit interaction). Lifecycle states, all user-visible: `requested → under_review → approved / partially_approved / denied → processing → refunded` (with settlement date) or `failed`. Refund method mirrors payment method; PSP ETA shown; every state change notifies + writes to ledger.
- **Edge cases & handling:**
  - Refund after usage → `within cooling-off: pro-rate by consumed value per published formula; outside cooling-off: only guarantee/SLA credits, not discretionary refund`.
  - Refund after a guarantee credit extended the plan → `guarantee-extended free days excluded from the refundable base`.
  - Chargeback filed while refund in progress → `freeze the refund, mark disputed, no double-refund; resolve via PSP dispute; entitlement revoked`.
  - Chargeback on a consumed subscription (friendly fraud) → `submit usage + consent evidence; on loss, re-purchase restriction; identity-linked to prevent re-registration abuse`.
  - App-store purchase refund → `directed to the store; platform reflects the decision, can't itself issue cash; UI explains the store path`.
  - Partial refund (proration/milestone) → `partially_approved with itemized breakdown; remaining entitlement preserved`.
  - Currency fluctuation between charge + refund → `refund original currency + amount (no FX loss to user)`.
  - Tax refund component → `tax reversed per jurisdiction; GST/VAT reversal itemized`.
  - Duplicate refund request → `idempotent; attaches to existing case, no double payout`.
  - Refund denied → `plain-language reason + escalation/grievance path (DPDP grievance officer)`.
- **Acceptance criteria:** policy shown before purchase + status trackable through named states with a settlement ETA; usage-based + guarantee-credit deductions itemized; no refund+chargeback double payout (disputes freeze refunds); every state change logged + notified.
- **Dependencies:** PAY-03/07/08, SUB-05, ADMIN-04, PRD §7.5 (re-registration ban), §9 grievance.

### PAY-07: Tax & FX handling
- **Description:** Correct, transparent tax (India GST, EU/UK VAT, UAE VAT, SG GST) + FX across markets.
- **Expected behavior:** tax computed by billing jurisdiction at charge time; India GST-inclusive display, West adds tax at checkout; tax-compliant invoices (GSTIN/VAT ID capture for B2B where relevant). Platform charges in the market's set currency; FX is the PSP's problem for cross-border cards — platform doesn't skim FX spread. Reverse-charge/place-of-supply rules for digital services applied per jurisdiction.
- **Edge cases & handling:**
  - Valid VAT ID provided → `apply reverse charge / VAT exemption where applicable; validate ID (VIES-style), else charge VAT`.
  - Tax rate changes between purchase + renewal → `renewal uses the rate at renewal time, disclosed in reminder`.
  - Cross-border card (Indian card, USD price) → `billed in USD; user's bank applies FX; platform discloses "billed in USD" to avoid surprise`.
  - Refund tax reversal → `reverse exact tax collected; if rate changed, reverse at the originally-collected rate`.
  - Market with no tax config yet → `block launch of paid SKUs there until a tax rule exists (fail-closed) rather than under-collect`.
  - Rounding causing tax-inclusive display to differ by 1 unit → `canonical amount is the tax-inclusive charged total; components derived from it`.
- **Acceptance criteria:** every charge produces a jurisdiction-compliant tax invoice; displayed total = charged total in the disclosed currency (no hidden FX spread); refunds reverse tax at the originally-collected rate.
- **Dependencies:** PAY-01/03/06, tax engine, ADMIN-04, PRD §9.

### PAY-08: Reachability guarantee & outcome-linked credits
- **Description:** The value answer to "null and void": ≥8 verified reachable matches/month; miss (+ under-served) → paid time auto-extends free with a visible counter; a genuine-zero-reply cycle can be credited, guardrailed by min profile-strength + effort. (Runtime logic detailed in Domain 4 REACH-03; this is the billing/entitlement side.)
- **Expected behavior:** per paid cycle track verified-reachable-matches surfaced, quality-outreach count, genuine replies, profile-strength. **Reachability SLA breach** (surfaced <8 AND platform under-served) → auto-extend paid period free; visible counter + reason. **First-reply floor breach** (zero genuine replies, platform under-served, guardrails met) → cycle credited. Credits non-cash by default (extended access); cash refund only where policy states.
- **Edge cases & handling:**
  - User did no quality outreach → `first-reply floor not triggered; SLA of surfaced matches still applies (surfacing is platform's job, replying is not guaranteed); clear messaging distinguishes them`.
  - Low profile-strength claims credit → `below threshold → not eligible; shown concrete coaching fixes, dignity-preserving`.
  - Games effort (spammy mass-likes) → `effort must be QUALITY outreach; low-effort mass-likes don't count`.
  - Guarantee during pause → `counter freezes; no breach accrues during paused days`.
  - Platform DID surface ≥8 but user unresponsive/inactive → `no breach; SLA is about platform delivery, transparently stated`.
  - Male intake throttled so supply is thin → `governor should prevent selling into thin markets; if a breach still occurs, guarantee pays out (bounded liability by design)`.
  - Concurrent SLA + first-reply breach → `apply the more favorable single remedy, not stacked double credit`.
  - Guarantee credit then refund → `refundable base excludes guarantee-granted free days`.
  - One gender systematically under-served → `feeds ADMIN-05 fairness dashboard; systemic breach may trigger cohort-wide credits`.
- **Acceptance criteria:** distinguishes "matches surfaced" (SLA) from "replies received" (not guaranteed) in logic + UI; breaches auto-extend/credit with a visible counter + reason (no manual claim for the automatic SLA); credit enforces profile-strength + genuine-effort guardrails; interacts correctly with pause/refunds/concierge (no double-remedy).
- **Dependencies:** Domain 4 REACH-03, Discovery/reachability (PRD §7.3/§7.4), ratio governor, ADMIN-05, PAY-06, SUB-04, PRD §10.

### PAY-09: Referral & Vouch program (billing side)
- **Description:** CAC-deflating referral + trust-building **Vouch**. Rewards vest on **verified activation** (not signup), **ratio-weighted** (scaled to protect balance), and Vouches feed a **vouch reputation score** complementing verification. (Product mechanics in Domain 4/Growth; this is the reward-ledger + anti-abuse side.)
- **Expected behavior:** **Referral** reward (credit/free days/discount) vests only when the referee reaches verified-activation (identity + eligibility verified AND first genuine reply / defined activation), not at signup. **Ratio-weighted:** reward magnitude scales by how much the referee helps liquidity balance (referring an under-supplied side earns more; over-supplied earns less/capped) — enforcing male-intake-as-valve. **Vouch:** a verified member vouches for another's authenticity/seriousness → a vouch reputation score shown as social proof, weighted by the voucher's own standing (sybil-resistant). Vesting on a schedule (partial at activation, remainder after a retention/payment milestone) to deter churn-and-burn.
- **Edge cases & handling:**
  - Referral farming (fake/dormant signups) → `no vesting without verified-activation; unverified referees yield zero reward`.
  - Self/circular referral rings → `device/identity/graph analysis blocks; identity-linked prevents multi-account farming`.
  - Referee refunds/chargebacks after reward vested → `claw back the vested reward; if spent, offset future or flag account`.
  - Referral into an over-supplied side → `reward down-weighted/capped; UI explains "we most need [under-supplied side] right now"`.
  - Vouch abuse (rings, paid vouches) → `reputation-weighting + graph anomaly detection discount ring vouches; low-reputation vouchers negligible; reciprocal clusters flagged`.
  - Vouched user later banned → `retroactively decrement vouchers' reputation weight + strip the vouch score`.
  - Reward currency across markets → `issued in the referrer's market currency; cross-market uses the referrer's locale`.
  - Referrer cancels before vesting completes → `unvested portion forfeits per schedule; vested honored`.
  - Notification → `referral/vouch notices must not leak the referrer's/referee's identity to a non-consented party; vouch visibility respects the vouched user's privacy (NOTIF-05)`.
  - Reward exceeds anti-abuse velocity cap → `per-period reward cap; excess referrals build vouch/social value but stop paying`.
- **Acceptance criteria:** no referral reward vests without verified-activation; reward magnitude demonstrably ratio-weighted; refund/chargeback/ban claws back rewards/reputation; vouch score reputation-weighted + sybil-resistant; vouch/referral notices never leak non-consented identities.
- **Dependencies:** Verification, activation metric (T14 first-reply), ratio governor, fraud/graph detection (Domain 5), PAY-06 clawback, NOTIF-05, ADMIN-05, PRD §13.

---

## Notifications (NOTIF-)

### NOTIF-01: Notification types & event taxonomy
- **Description:** Canonical set spanning engagement + transactional + safety: new verified match, mutual interest, new message, reply-nudge/"interest expiring," weekly curated digest, who-viewed-me (Elite), verification status change, billing/renewal/refund (transactional), guarantee-credit granted, concierge milestone update, referral/vouch events, T&S/moderation notices.
- **Expected behavior:** each type has category (engagement/transactional/safety), default channels, default on/off, quiet-hours eligibility, privacy sensitivity. Transactional (billing/refund/renewal/security) + safety notices are **essential** — reduced-suppressibility. Engagement types fully user-controllable (NOTIF-03).
- **Edge cases & handling:**
  - New type added → `defaults OFF for engagement (opt-in), ON for essential; never silently opts users into new marketing pushes`.
  - Duplicate events (message + mutual interest same instant) → `coalesce into a single digest-style notification`.
  - High-frequency events (many interests on a popular profile) → `rate-limit/batch (aligns with inbound caps) so high-demand profiles aren't drowned`.
- **Acceptance criteria:** every type classified with explicit defaults; new engagement types opt-in by default; burst events coalesced/rate-limited.
- **Dependencies:** NOTIF-02/03/04/05, Discovery, Messaging, Billing, Concierge, PRD §7.8.

### NOTIF-02: Multi-channel delivery (push / email / SMS / WhatsApp)
- **Description:** Channel-abstracted delivery "where compliant," with per-region legality gating.
- **Expected behavior:** a router picks eligible channels given user prefs (NOTIF-03), regional legality (WhatsApp template rules, India SMS DLT, US TCPA, EU GDPR/PECR), quiet hours (NOTIF-04), content sensitivity (NOTIF-05). Fallback chain (push → email) except where the user disabled the fallback. WhatsApp/SMS require verified opt-in + approved templates; marketing content honors separate consent from transactional.
- **Edge cases & handling:**
  - WhatsApp not opted-in / unavailable in region → `channel skipped; no attempt; fall back`.
  - India SMS without a DLT-registered template → `blocked; must use registered template or drop`.
  - Email hard-bounce / invalid push token → `mark channel unhealthy, fall back, prompt user to update contact`.
  - Opted into WhatsApp for transactional only → `engagement notices never via WhatsApp`.
  - Provider outage → `retry with backoff for essential; engagement dropped after TTL rather than delivered late/irrelevant`.
  - Same notice deliverable on multiple channels → `respect the user's per-type channel choice; avoid sending on all channels unless opted-in`.
- **Acceptance criteria:** no channel send violates regional consent/template law (DLT/TCPA/PECR/WhatsApp); failed primary channel falls back only to user-enabled channels; marketing vs transactional consent scopes enforced per channel.
- **Dependencies:** NOTIF-03/04/05, I18N-01, PRD §9, channel providers.

### NOTIF-03: Per-type toggles & preference center
- **Expected behavior:** a matrix of notification type × channel, each independently on/off, plus a master frequency control (real-time / daily digest / weekly). Essential transactional/safety notices show as "always on" for the mandatory channel (email), other channels optional. Changes take effect immediately + versioned as consent records.
- **Edge cases & handling:**
  - Turns off all channels for an essential type → `engagement portion honored; the mandatory transactional carve-out still delivers billing/security via email, clearly labeled non-marketing`.
  - Conflicting settings (type on, channel off) → `intersection wins: delivered only where both enabled`.
  - Digest mode → `real-time engagement batches into the digest; essential notices still send immediately`.
  - Preference change mid-flight of a queued notification → `re-evaluate against current prefs at send time, not enqueue time`.
- **Acceptance criteria:** every engagement type independently toggleable per channel + honored at send time; essential notices remain deliverable via the mandatory channel regardless of engagement settings; every preference change logged as a consent record.
- **Dependencies:** NOTIF-01/02, PAY-05, PRD §9.

### NOTIF-04: Timezone quiet hours & DST handling
- **Expected behavior:** user sets quiet-hours window in their local timezone; engagement notifications suppress during it + release at the boundary (or roll into the next digest). Timezone derives from user setting > device > locale; DST via IANA tz database, not fixed offsets. Essential/safety notices may bypass quiet hours where time-critical (security alert, pre-renewal reminder that would otherwise miss the deadline).
- **Edge cases & handling:**
  - DST spring-forward (a local time doesn't exist) → `shift to the next valid instant; never drop/double-send`.
  - DST fall-back (a local time occurs twice) → `send once, first occurrence`.
  - User travels across timezones → `quiet hours follow the SET timezone unless auto-timezone enabled; changing timezone doesn't retroactively re-fire suppressed notices`.
  - Cross-timezone pair (sender IST, recipient PST) → `evaluated on the RECIPIENT's timezone`.
  - Notification queued before quiet hours, deliverable during → `held until window ends (engagement) or sent if essential`.
  - Ambiguous/unset timezone → `safe default (no sends 22:00–08:00 in the best-known locale) rather than 24h sending`.
  - Digest scheduled inside quiet hours → `shifted to the window boundary`.
- **Acceptance criteria:** quiet hours evaluated in the recipient's timezone via IANA tz data; DST spring-forward/fall-back never cause dropped/duplicate sends; only time-critical essential notices bypass quiet hours.
- **Dependencies:** I18N-02, NOTIF-01/02/03, PAY-05.

### NOTIF-05: Identity-leak prevention & consent-aware delivery
- **Description:** No notification may leak a profile's identity to a non-consented party ("no notification leaks profile identity to non-consented parties"). Critical for incognito, photo-gating, blocked-user semantics.
- **Expected behavior:** notification content redacted to the recipient's entitlement + the subject's privacy settings at send time (incognito viewers don't trigger who-viewed-me; photo-gated media not embedded before mutual consent; contact info never appears pre-consent). Honors block/mute. Push/lock-screen previews strip sensitive content (name/photo) unless the recipient enabled rich previews + is entitled.
- **Edge cases & handling:**
  - Notification to a blocked party → `not sent; no "you were blocked" signal (silent)`.
  - Incognito user viewed a profile → `no who-viewed-me notification (incognito is the point)`.
  - Recipient revoked consent between event + send → `re-check at send; suppress if withdrawn`.
  - Family/parent collaborator access → `notifications respect the member's explicit consent scope; no leaking beyond granted fields`.
  - Lock-screen exposure on a shared device → `default redacted preview ("You have a new match") without identity; opt-in for rich previews`.
  - Email to a shared/family inbox → `content minimized; no photos/contact in the body pre-consent; deep-link into the authenticated app`.
  - WhatsApp/SMS (less private) → `stricter redaction; identity/photo never sent over SMS/WhatsApp; only "open the app" prompts`.
  - Referral/vouch notice → `never reveals a non-consented party's identity`.
  - Message notification after recipient blocks mid-thread → `pending notifications for the now-blocked sender cancelled`.
- **Acceptance criteria:** no notification reveals identity/photo/contact to a party lacking consent/entitlement on any channel; incognito, block, mute states fully suppress the corresponding notifications; default previews on low-privacy channels/lock screens identity-redacted.
- **Dependencies:** Domain 2 (privacy/visibility, block/mute, incognito), SUB-01 entitlements, NOTIF-02, PAY-09, PRD §7.4 no-contact-leak.

---

## Internationalization (I18N-)

### I18N-01: Multi-language UI & content localization
- **Expected behavior:** all user-facing strings externalized to locale bundles (no hardcoded copy); ICU message format for plurals/gender/number. Locale resolves from user setting > device > Accept-Language > market default; user can override. Currency/number/date/time formatting locale-aware. Legal-sensitive copy (refund, consent, tax) professionally localized + versioned per jurisdiction — not machine-translated for compliance text.
- **Edge cases & handling:**
  - Missing translation key → `fall back to English/market default, never show the raw key; log for backlog`.
  - RTL language (Arabic for UAE) → `full RTL mirroring; bidi handling for mixed LTR names/numbers`.
  - Pluralization/gendered grammar → `ICU rules per language (Arabic dual/plural); avoid gender assumptions in matching copy`.
  - User-generated content in another language → `displayed as-authored; optional translation offered, never auto-overwriting`.
  - Locale change mid-session → `re-render without data loss; in-flight notifications use the locale at send time`.
  - Legal copy version mismatch across locales → `consent captured against the specific localized version shown; if a locale's legal copy is stale, block that flow in that locale until updated (fail-closed)`.
- **Acceptance criteria:** no hardcoded strings (missing keys fall back gracefully, never raw keys); RTL + pluralization/gender render correctly per language; compliance copy human-localized + consent binds to the exact localized version.
- **Dependencies:** NOTIF-02, PAY-01/04/06, PRD §9, §7.6.

### I18N-02: Timezone & locale handling
- **Expected behavior:** store all timestamps UTC; render in the viewer's timezone via IANA identifiers, not fixed offsets. Timezone resolves from user setting > device > verified residence; explicit setting + optional auto-update on travel. Weekly caps, interest resets, quiet hours, digests all compute against the user's timezone.
- **Edge cases & handling:**
  - DST transitions → `handled per IANA (see NOTIF-04); scheduled jobs recompute local times`.
  - Half-hour/45-min offset zones (IST, Nepal) → `supported natively via tz db`.
  - User in one country, verified residence in another → `display timezone follows the current setting; matching/business rules can use residence where relevant (I18N-03)`.
  - Ambiguous device timezone → `prompt to confirm; default to market timezone meanwhile`.
- **Acceptance criteria:** all timestamps stored UTC, rendered in the user's IANA timezone; caps/resets/digests honor the user's timezone incl. non-hour offsets + DST.
- **Dependencies:** NOTIF-04, SUB-01 caps/resets, tz database, verification residence.

### I18N-03: Locale-aware matching (diaspora ↔ home)
- **Expected behavior:** matching considers residence country, willing-to-relocate, cultural/linguistic context, diaspora↔home preferences (London NRI open to India or global). Cross-market discovery where both users' preferences + legal transfer rules allow; distance/timezone-difference surfaced honestly. Doesn't hard-segregate markets; a user can explicitly opt into home-country or global pools.
- **Edge cases & handling:**
  - Cross-region match implies cross-border data transfer → `only surface if a lawful transfer mechanism exists for both regions (I18N-04); else limit to metadata without transferring restricted PII`.
  - Timezone gap too large for practical contact → `surfaced as context, not a hard block; reachability labels account for response latency`.
  - Language mismatch → `flagged; shared-language filter available`.
  - Relocation willingness asymmetry → `both parties' relocation prefs must be compatible before prioritizing a cross-country match`.
  - Sanctioned/embargoed jurisdiction → `exclude from cross-border matching`.
- **Acceptance criteria:** cross-border matches only surface where lawful data transfer is possible for both; users can opt into home-country and/or global pools; matching respects relocation + language compatibility.
- **Dependencies:** I18N-04, Matching (Domain 3), reachability, PRD §9.

### I18N-04: Multi-country data-residency routing
- **Description:** Region-aware routing/storage (DPDP India + GDPR EU/UK baseline). (Core mechanics in Domain 5 PRIVC-05; this is the i18n/product view.)
- **Expected behavior:** user data routed/stored per residency (regional partitions or regional stacks); sensitive documents in the isolated brokered vault. Cross-border access uses lawful-transfer mechanisms + least-privilege brokering, logged. Payment/tax data residency aligned to billing jurisdiction.
- **Edge cases & handling:**
  - User relocates permanently → `residency re-evaluated; migration/replication per policy with consent; original-region obligations (retention/erasure) still honored`.
  - Erasure request → `propagates across all regional stores/backups per schedule; confirm completion across regions`.
  - Cross-border admin/concierge access → `brokered, least-privilege, audit-logged; no bulk PII egress`.
  - Region with stricter localization (data must not leave) → `global matching restricted to non-restricted metadata; full profile stays in-region`.
  - Backup/DR crossing regions → `DR replicas honor residency; encrypted, region-scoped keys`.
- **Acceptance criteria:** data stored per residency policy with sensitive docs isolated in the brokered vault; erasure + retention propagate across all regional stores + backups; every cross-border PII access lawful, least-privilege, audit-logged.
- **Dependencies:** Domain 5 PRIVC-03/04/05, ADMIN-06 audit, PAY-07, I18N-03, PRD §8/§9.

---

## Admin / Ops Consoles (ADMIN-)

### ADMIN-01: Verification review console
- **Description:** Queue-driven human review for payment-gated deep verification (V1) + escalated V0 cases — evidence viewer, source-check integrations, badge issuance; verifiers act only on consented evidence.
- **Expected behavior:** queue with SLA timers; evidence viewer for ID/liveness, credential (registrar/employer/LinkedIn), optional income; source-check results shown. Reviewer actions: approve badge, request more info, reject (reason), escalate (fraud/marital-status). Each writes an immutable audit + issues/updates the per-signal badge with provenance + date. Deep verification only surfaces for payment-intent accounts (SUB-01); Free stays automated.
- **Edge cases & handling:**
  - Evidence consent withdrawn before review → `item removed from queue; no action on non-consented evidence; user notified`.
  - Source API down → `park with retry; SLA paused; manual fallback documented`.
  - Suspected forged document → `escalate to fraud queue (ADMIN-02); identity-linked flag to block re-registration`.
  - Marital-status attestation conflict → `escalating checks; badge withheld; sensitive handling`.
  - Reviewer conflict of interest → `recuse; reassign; logged`.
  - Stale attestation (re-verification cadence) → `flagged for re-review; badge marked stale`.
  - Duplicate identity across accounts → `merge/flag; prevent multi-account`.
- **Acceptance criteria:** no verification action on non-consented evidence; every action role-scoped, reasoned, immutably logged; badges carry provenance + date; deep-verification queue contains only payment-intent accounts.
- **Dependencies:** Verification (Domain 1), ADMIN-06 RBAC/audit, ADMIN-02, isolated doc vault, SUB-01.

### ADMIN-02: Moderation console
- **Description:** Report triage + enforcement (Domain 5 MOD-01/SAFE-*) — actions with immutable audit + safety-escalation. (Detailed in Domain 5; this is the console surface within Admin/Ops.)
- **Expected behavior:** severity-ranked triaged queue; moderator reviews context (chat excerpts within consent/policy, fraud signals) + acts within SLA (≥99%). Actions: warn/rate-limit/suspend/ban (identity-linked), escalate to safety/legal. Anti-romance-scam signals + duplicate/stock-photo detection feed the queue.
- **Edge cases & handling:**
  - Repeat offender re-registers → `identity-linked ban blocks re-entry`.
  - False/weaponized reporting → `reporter-reputation weighting; retaliatory reports flagged; no auto-action without review for low-severity`.
  - Threat/extortion → `immediate safety-escalation; evidence preserved; possible LE referral`.
  - Moderator accesses chat content → `least-privilege, consent/policy-scoped, audit-logged; no bulk reading`.
  - Banned user has active paid plan → `plan handling per policy (refund unused vs forfeiture for TOS breach); PAY-06 interaction; chargeback risk flagged`.
  - Vouched/referred user banned → `clawback + reputation decrement (PAY-09)`.
  - Cross-region report → `applicable regional rules; PII access residency-aware`.
- **Acceptance criteria:** every report actioned within SLA + immutably logged; bans identity-linked + survive re-registration; moderator PII/chat access least-privilege + audited.
- **Dependencies:** Domain 5 fraud/safety, ADMIN-06, PAY-06/09, I18N-04.

### ADMIN-03: Concierge CRM
- **Description:** Operational CRM for RMs running Concierge engagements — contract, milestone tracking, curated-intro pipeline, client comms.
- **Expected behavior:** per-engagement record (contract scope/milestones/refund schedule/success-fee, assigned RM, client profile, curated-intro pipeline with reachability status, milestone states synced to the client tracker). RM logs intros delivered, notes, milestone completion (with evidence); milestone completion may trigger billing/refund events (ADMIN-04). Handoff/reassignment carries full history.
- **Edge cases & handling:**
  - Milestone marked complete without evidence → `blocked; evidence required before it counts toward billing/success-fee`.
  - Client disputes an intro's quality → `opens a dispute case (ADMIN-04) with intro reachability logs as evidence`.
  - RM reassignment → `full CRM history transfers; milestone dates preserved; client notified`.
  - Curated intro to a user who blocks/withdraws → `intro voided; doesn't count as delivered`.
  - Success-fee trigger → `outcome evidence verified before invoicing; CRM cannot self-invoice without ADMIN-04 confirmation`.
  - Concierge client also on self-serve guarantee → `concierge SLA supersedes; single remedy`.
  - RM accesses client PII → `least-privilege, consent-scoped, audit-logged`.
- **Acceptance criteria:** milestone completion requires logged evidence + syncs to the client tracker; disputes + success-fee triggers route to billing ops with an audit trail (RM can't unilaterally bill); all RM PII access role-scoped + logged.
- **Dependencies:** SUB-05, ADMIN-04, PAY-08, Discovery/curation, Notification, ADMIN-06.

### ADMIN-04: Refund / billing ops console
- **Description:** Back-office for refund review, dispute/chargeback handling, proration/credit adjustments, catalog/price management, tax-config ops.
- **Expected behavior:** refund queue with the PAY-06 state machine; agents approve/partial/deny with reasoned, logged decisions; view the immutable ledger, consent events (PAY-04), usage data (for usage-based proration), guarantee-credit history. Chargeback/dispute workspace: assemble evidence, respond to PSP, record outcome, trigger clawbacks (PAY-09) + re-purchase restrictions on friendly fraud. Catalog management (prices, SKUs incl. app-store markup, tax rules) role-scoped, versioned, effective-dated (grandfathering).
- **Edge cases & handling:**
  - Refund + concurrent chargeback → `freeze refund, mark disputed, prevent double payout`.
  - Cross-currency refund → `refund original amount/currency; tax reversed at original rate`.
  - Catalog price edit while users mid-purchase → `effective-dated; quote tokens honor the price at render; no retroactive change to active periods`.
  - Manual credit/adjustment → `requires reason + elevated role; logged; reflected in ledger + entitlements`.
  - App-store-originated refund → `ledger records store origin; entitlement revoked via receipt re-validation; no platform cash refund`.
  - Milestone/dispute-driven refund from concierge → `pulls evidence from ADMIN-03; itemized partial refund`.
  - Tax config missing for a market being launched → `catalog can't publish paid SKUs there (fail-closed)`.
  - Bulk/cohort credit (systemic fairness breach) → `issue guarantee-style credits across the affected cohort (PAY-08/ADMIN-05), logged + reason-coded`.
- **Acceptance criteria:** every refund/adjustment reasoned, role-scoped, immutably logged (no refund+chargeback double payout); catalog/price/tax edits versioned + effective-dated (active periods never retroactively repriced); store-originated refunds reconcile via receipt validation without platform double-refund.
- **Dependencies:** PAY-01/03/06/07/08/09, PAY-02 receipt validation, ADMIN-03/05/06, PRD §7.5.

### ADMIN-05: Analytics & fairness dashboards
- **Description:** First-class fairness + liquidity dashboards ("fairness dashboards as first-class, monitored metrics") — reply-rate parity by gender/geography, active/verified ratio, reachability, and the ratio governor's inputs.
- **Expected behavior:** KPIs — verification completion, **active/verified ratio (≥70%)**, **median first-reply rate + gender parity (within 10 pts)**, T14 reachability (primary churn predictor), match→conversation, free→paid, retention, verified-reachable-matches/user/month (≥8), concierge-milestone attainment, guarantee-breach rate. **Liquidity dashboard** drives the ratio governor: projected 14-day male reachability vs target; auto-throttle male intake when below (marketing can't override). Cohort outcome tracking (opt-in) for honest, audited success reporting.
- **Edge cases & handling:**
  - Parity breach (one gender/geo systematically under-served) → `alert; may trigger cohort credits (PAY-08) + intake throttle`.
  - T14 falling → `earliest churn alarm; surfaces to acquisition pacing + monetization`.
  - Small-cohort statistical noise → `minimum-n thresholds before firing fairness alerts; confidence intervals shown`.
  - Data-residency on analytics → `aggregate/pseudonymized cross-region; no raw restricted-PII egress`.
  - Vanity-metric risk → `"active" defined honestly (authenticated + engaged), not registrations, countering the 5-lakh-vs-30k gap`.
  - Guarantee-breach spike → `flags thin-supply markets; feed back to intake throttle before selling into them`.
- **Acceptance criteria:** reply-rate parity + active/verified ratio monitored as first-class metrics with alerting; the ratio governor auto-throttles male intake on projected reachability shortfall (marketing can't override); "active" + outcome metrics use honest published definitions; analytics respects data residency.
- **Dependencies:** event instrumentation (PRD §11), Discovery/reachability, ratio governor (Domain 4), PAY-08, ADMIN-04, I18N-04, PRD §10.

### ADMIN-06: Role-based access control & audit logging (cross-cutting)
- **Description:** Least-privilege, role-scoped, fully audit-logged access underpinning all consoles ("all back-office access role-scoped + audit-logged; PII access least-privilege").
- **Expected behavior:** roles (verifier, moderator, RM, billing-ops, admin, analyst) with granular least-privilege permissions; PII/document access brokered + time-boxed. Every back-office read/write of sensitive data immutably logged (who/what/when/why/case-ref); logs tamper-evident + retained. Sensitive-document access goes through the isolated brokered vault (separate from the profile DB), requiring an open case + justification.
- **Edge cases & handling:**
  - Privilege escalation attempt → `denied + alerted; no standing super-admin PII access without break-glass + audit`.
  - Break-glass emergency access → `time-boxed, heavily logged, post-hoc reviewed`.
  - Access to data in another residency region → `lawful, least-privilege, logged`.
  - Departed staff → `immediate deprovision; sessions revoked; access-review cadence`.
  - Analyst querying PII → `restricted to pseudonymized/aggregate unless a justified logged exception`.
  - Audit-log access → `itself restricted + logged (no silent tampering); immutable store`.
- **Acceptance criteria:** no back-office data access without a role grant; all sensitive access immutably logged with justification; document-vault access requires an open case + justification + brokered separately from the profile DB; break-glass + cross-region access time-boxed + reviewed.
- **Dependencies:** all ADMIN consoles, isolated doc vault (Domain 5 PRIVC-03), I18N-04, PRD §9/§7.9.
