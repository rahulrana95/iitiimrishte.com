# Feature Specification — iitiimrishte.com

**Developer-ready feature catalogue.** Every feature has: **name · description · expected behavior in all states · edge cases & handling · acceptance criteria · dependencies.**

**Version:** 0.1 (Draft) · **Date:** 2026-07-22 · **Status:** For review
**Companions:** [PRD](./PRD.md) (the *what/why*), [ERD](./ERD.md) (the data model), [GTM & Launch Plan](./GTM-LAUNCH-PLAN.md) (go-to-market), [market-research dossier](./research/market-research.md) (the evidence). This document is the *how-it-behaves*: it expands PRD §7 into implementable detail with exhaustive edge-case handling.

> Produced by a 6-agent spec-writer team, each owning a feature domain and writing to the same strict template. Every feature is grounded in a documented incumbent gap (cited to the dossier) and the GTM decisions.

---

## How to read this

- Features are grouped into **6 domains**, one file each under [`docs/spec/`](./spec/).
- Each feature has a stable **ID** (e.g. `VER-2`, `BAL-01`, `SAFE-07`) used for cross-references throughout.
- **Expected behavior** enumerates every meaningful state, not just the happy path.
- **Edge cases & handling** use the form `case → exact system behavior`.
- **Acceptance criteria** are testable assertions.

### Global invariants (apply platform-wide)

1. **Discoverability gate** — a profile appears in any surface only if identity-verified + ≥1 eligibility badge + `account_status = active`.
2. **Non-enumerable identity** — clients only ever see the opaque `public_handle`; internal IDs never leave the server (designs out the incumbent's `Base64(sequential-int)` scrape defect).
3. **Verification ≠ vouched** — a verified badge attests identity/credential, never character/intent.
4. **Gender-gating governs admission *pace*, never verification *rigor*** — verification requirements are provably identical across genders.
5. **Never penalize the oversubscribed** — reply-nudges/expiry/penalties apply to *senders*, never to a recipient at inbound cap.
6. **Corridor is the unit of liquidity** — all caps/ratios/governors computed per corridor, never globally; discovery never padded with unreachable-geography profiles.
7. **Consent-first & least-privilege** — DPDP unbundled/unconditional consent + GDPR lawful basis; sensitive documents only in the isolated, access-brokered vault; every back-office access audited.

---

## Domain files

| # | Domain | File | ID prefixes |
|---|---|---|---|
| 1 | Onboarding, Verification & Eligibility | [`spec/01-onboarding-verification-eligibility.md`](./spec/01-onboarding-verification-eligibility.md) | ONB-, AUTH-, VER-, ELG- |
| 2 | Profile, Media & Privacy | [`spec/02-profile-media-privacy.md`](./spec/02-profile-media-privacy.md) | PROF-, MEDIA-, PRIV-, FAM- |
| 3 | Discovery, Search & Matching | [`spec/03-discovery-search-matching.md`](./spec/03-discovery-search-matching.md) | SRCH-, MATCH-, DISC-, FRESH- |
| 4 | Communication & the Two-Sided Balance Engine | [`spec/04-communication-balance-engine.md`](./spec/04-communication-balance-engine.md) | INT-, MSG-, REACH-, BAL- |
| 5 | Trust & Safety, Fraud, Privacy & Compliance | [`spec/05-trust-safety-fraud-compliance.md`](./spec/05-trust-safety-fraud-compliance.md) | SAFE-, MOD-, FRAUD-, PRIVC- |
| 6 | Monetization, Notifications, i18n & Admin/Ops | [`spec/06-monetization-notifications-i18n-admin.md`](./spec/06-monetization-notifications-i18n-admin.md) | SUB-, PAY-, NOTIF-, I18N-, ADMIN- |

---

## Feature index (traceability)

### Domain 1 — Onboarding, Verification & Eligibility
- **ONB-1** Apply / Waitlist funnel & invite mechanics
- **ONB-2** Registration & Authentication (email/password + Google/Apple/LinkedIn SSO)
- **ONB-3** Gender-gated admission — user-facing (women instant, men metered)
- **ONB-4** Queue position & queue-jump mechanics
- **ONB-5** Onboarding progress, resumability & partial-verification state
- **ONB-6** Symmetric verification standards across genders
- **VER-1** Verification orchestration & badge model
- **VER-2** Government ID + liveness / selfie-match (identity)
- **VER-3** Identity-linked duplicate / ban-evasion detection
- **VER-4** Education-credential verification (registrar / DigiLocker / institution-email / document review)
- **VER-5** Employer / professional verification (work-email / LinkedIn / HR attestation)
- **VER-6** Income / net-worth verification (optional)
- **VER-7** Marital-status attestation with escalating checks
- **VER-8** Re-verification cadence, expiry & verification downgrade
- **VER-9** Photo authenticity (liveness-linked, anti stock/steal)
- **VER-10** Verification appeal & manual re-review
- **ELG-1** Eligibility engine (dual-route, per-market thresholds)
- **ELG-2** "Our Standards" transparency & eligibility explainability

### Domain 2 — Profile, Media & Privacy
- **PROF-01** Structured profile field model
- **PROF-02** Non-enumerable opaque public handle
- **PROF-03** Profile completeness & strength score
- **PROF-08** Verified-vs-self-declared rendering & badge provenance display
- **MEDIA-01** Blur-until-mutual photo gating
- **MEDIA-02** Per-photo visibility controls
- **MEDIA-04** Watermarking & screenshot deterrence
- **MEDIA-05** Face-photo-optional (verification-carried trust)
- **MEDIA-06** Secure tokened media delivery & opaque asset keys
- **MEDIA-07** Media lifecycle in active conversations
- **PRIV-01** Incognito / hidden browsing (and who-viewed-me interaction)
- **PRIV-02** Granular per-field visibility & conflict resolution
- **PRIV-05** Employer masking ("verified at [Big Tech / Top Consulting / Hospital]")
- **PRIV-06** Profile-level block & mute
- **PRIV-07** Scrape / enumeration defense & anomalous-access alerts
- **FAM-01** Consented family collaborator invitation & roles
- **FAM-02** Scoped collaborator visibility
- **FAM-03** Collaborator edit rights with owner approval
- **FAM-04** Collaborator communication boundaries
- **FAM-05** Revoking collaborator access (incl. mid-conversation) & audit

### Domain 3 — Discovery, Search & Matching
- **SRCH-01** Faceted candidate search
- **SRCH-02** Tier-gated search filters
- **MATCH-01** Compatibility scoring & explainable "why this match"
- **MATCH-02** Reachability-aware ranking
- **DISC-01** Curated daily picks (reciprocity-weighted, anti-flood)
- **DISC-02** Who-viewed-me (incognito-suppressed)
- **DISC-03** Shortlist / save
- **FRESH-01** Freshness — new-since-last-visit, seen-dedupe, new-match alerts

### Domain 4 — Communication & the Two-Sided Balance Engine
- **INT-01** Outbound daily interest cap
- **INT-02** Effort tax — personalized note required
- **INT-03** Merit-based quota expansion
- **INT-04** Per-profile inbound cap + ranked working queue *(the single most important anti-flood mechanism)*
- **INT-05** Interest expiry (~7 days)
- **INT-06** Reply-fairness — sender-only nudges, never penalize the oversubscribed
- **INT-07** Merit surfacing & coaching hook
- **INT-08** Women-initiate discovery lane
- **MSG-01** Mutual-interest chat
- **MSG-02** Contact reveal gated on mutual consent (not payment)
- **REACH-01** Reachability labels
- **REACH-02** Personal reachability dashboard
- **REACH-03** Reachability guarantee (billing pause / credit)
- **BAL-01** Gender-gated admission valve
- **BAL-02** Ratio governor — auto-throttle male intake, red line 1.5:1
- **BAL-03** Reciprocity-weighted curated intros (anti-flood surface)
- **BAL-04** Corridor liquidity unit & relocation-overlap matching
- **BAL-05** Orientation-aware valve mapping (non-binary & same-gender)
- **BAL-06** Out-of-balance playbook & closed-loop rebalancing

### Domain 5 — Trust & Safety, Fraud, Privacy & Compliance
- **SAFE-01** Report user — reasons, evidence, submission
- **SAFE-02** Report triage & routing engine
- **SAFE-03** Silent block (invisible to blocked user)
- **SAFE-04** Report SLA, status lifecycle & status-told-back
- **SAFE-05** Anti-harassment message filtering (opt-in reveal)
- **SAFE-06** Identity-linked bans that survive re-registration
- **SAFE-07** Safety-escalation lane (threats/extortion, evidence preservation, LE-ready)
- **SAFE-08** In-context scam nudges (money / off-platform / overseas-emergency)
- **SAFE-09** "Verified ≠ vouched" safety education
- **SAFE-10** Immutable moderation & safety audit log
- **SAFE-11** Appeals & wrongful-action reversal
- **MOD-01** Moderation console actions (warn / suspend / ban / dismiss)
- **FRAUD-01** Device & behavioral signal engine
- **FRAUD-02** Velocity & rate-limit checks
- **FRAUD-03** Duplicate & stock-photo / face-reuse detection
- **FRAUD-04** Scam-pattern & romance-fraud models
- **FRAUD-05** Fraud-model feedback, tuning & false-positive governance
- **PRIVC-01** Consent records (DPDP unbundled/unconditional, GDPR lawful basis, withdrawal)
- **PRIVC-02** Consent withdrawal & mid-use handling
- **PRIVC-03** Isolated PII/document vault (field-level encryption, access-brokered)
- **PRIVC-04** Right-to-erasure / soft-delete + hard-purge (with fraud/legal retention)
- **PRIVC-05** Data-residency routing per region
- **PRIVC-06** Minor & sanctioned-data handling (age-gating, sanctions/PEP)

### Domain 6 — Monetization, Notifications, i18n & Admin/Ops
- **SUB-01** Subscription tier model & entitlement engine
- **SUB-02** Paywall & gating enforcement
- **SUB-03** Upgrade / downgrade / proration
- **SUB-04** Pause-before-cancel (pause > cancel)
- **SUB-05** Concierge / assisted engagement (RM, milestones, success-fee)
- **PAY-01** Regional multi-currency pricing engine
- **PAY-02** App-store markup steering to web
- **PAY-03** Checkout & billing
- **PAY-04** Renewal transparency & explicit consent (no forced auto-renew)
- **PAY-05** Early renewal reminder & one-tap cancel
- **PAY-06** Refund policy & trackable refund status
- **PAY-07** Tax & FX handling
- **PAY-08** Reachability guarantee & outcome-linked credits (billing side)
- **PAY-09** Referral & Vouch program (billing side)
- **NOTIF-01** Notification types & event taxonomy
- **NOTIF-02** Multi-channel delivery (push / email / SMS / WhatsApp)
- **NOTIF-03** Per-type toggles & preference center
- **NOTIF-04** Timezone quiet hours & DST handling
- **NOTIF-05** Identity-leak prevention & consent-aware delivery
- **I18N-01** Multi-language UI & content localization
- **I18N-02** Timezone & locale handling
- **I18N-03** Locale-aware matching (diaspora ↔ home)
- **I18N-04** Multi-country data-residency routing
- **ADMIN-01** Verification review console
- **ADMIN-02** Moderation console
- **ADMIN-03** Concierge CRM
- **ADMIN-04** Refund / billing ops console
- **ADMIN-05** Analytics & fairness dashboards
- **ADMIN-06** Role-based access control & audit logging

---

## MVP scope pointer

Per [PRD §13](./PRD.md#13-phased-roadmap), the **MVP** subset (one launch corridor) is roughly: ONB-1..6, AUTH, VER-1/2/4/5/9 + ELG-1, PROF-01/02/03/08, MEDIA-01/02/05/06, PRIV-01/02/06, SRCH-01, MATCH-01/02, DISC-01/03, FRESH-01, INT-01..06/08, MSG-01/02, REACH-01/03, BAL-01/02/03/04, SAFE-01/02/03/04/05/06/07/10, FRAUD-01/03, PRIVC-01/03/04, SUB-01/02, PAY-01/03/04/05/06/08, NOTIF-01..05, I18N-01/02/04, ADMIN-01/02/06. Fairness mechanics, income/employer-tier depth, concierge, full i18n, and the remaining admin consoles layer in at V1/V2.

_This spec is decision-grade but a draft; feature IDs are stable references for issue tracking. Load-bearing numbers (caps, thresholds, prices) are starting defaults to be tuned against a pilot cohort._
