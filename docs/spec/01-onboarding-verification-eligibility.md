# Feature Spec — Onboarding, Identity/Credential Verification & Eligibility

> Domain 1 of the [Feature Specification](../FEATURE-SPEC.md). Companion: [PRD](../PRD.md) §7.1/§7.5/§9, [research dossier](../research/market-research.md) §5/§8/§9, [GTM plan](../GTM-LAUNCH-PLAN.md) §1.

**Design principles carried through every feature:** (1) verification verifies **identity + intent, not just a degree** — the incumbent's core defect; (2) **symmetric standards across genders** — closing the "harder on men / looser female-side verification" seam; (3) **source-authenticated, not bare image upload** wherever a registrar/employer/ID API exists; (4) sensitive documents live in an **isolated, access-brokered vault**, never co-located with the profile DB; (5) **gender-gating governs admission *pace*, never verification *rigor*.**

---

### ONB-1: Apply / Waitlist funnel & invite mechanics
- **Description:** The single entry point is an application ("Apply for an invitation"), never open "Sign up." Scarcity is an asset, not a blocker (Raya model), directly answering the incumbent's cold-start/thin-inventory failure. Verification happens *before* the queue so the waitlist is a pool of pre-qualified, admittable people.
- **Expected behavior:**
  - Apply: email/SSO, gender, market/corridor, preliminary eligibility signals (claimed institution/employer/title). No discoverable profile yet.
  - Order: application → identity + ≥1 eligibility verification → queue (men) or instant admit (women) → cohort admission → discoverable.
  - Invite paths: organic; referral/peer-invite (reward vests on invitee *verification*, not signup); vouch/nominated profile (carries "Vouched by a verified member" badge); hand-seeded founding anchors + concierge-recruited women; ambassador-hosted salon invites.
  - Invite tokens: single-use, gender-tagged (scarce-gender may carry 2–3× reward + faster admission), time-boxed, attributed to referrer.
  - Status shown: `Application received → Verifying → Verified, in queue (position X) → Admitted → Active`.
- **Edge cases & handling:**
  - Abandons before verification → `held in Draft N days, reminder nudges, then auto-expired (DPDP-minimized)`.
  - Token reused → `rejected as consumed; prompt organic apply`.
  - Token expired → `rejected + "request fresh invite"; referrer notified`.
  - Referred invitee fails verification → `no reward vests; referrer quota restored (not penalized)`.
  - Vouched nominee never accepts → `nomination expires; no profile created; voucher stake released`.
  - Applies to a market not yet at density → `placed on market-interest list, "we open a city only when it's full enough"; converted to live queue when corridor opens`.
  - Duplicate application (new email) → `caught at identity stage (VER-3); merged/blocked, not double-queued`.
  - Mass low-effort applications from one device/IP → `velocity throttle + fraud flag`.
- **Acceptance criteria:** no applicant discoverable before identity + ≥1 eligibility verified; every admission path attributable in the reward ledger; tokens single-use/expiring/gender-tagged; reward vests only on invitee verification.
- **Dependencies:** ONB-2, ONB-3, ONB-4, VER-1..10, ELG-1, Referral/Vouch loops, T&S fraud.

### ONB-2: Registration & Authentication (email/password + Google/Apple/LinkedIn SSO)
- **Description:** Account creation and login. LinkedIn SSO is first-class because it doubles as a professional-signal source feeding VER-5, cutting verification cost in the Bengaluru↔London corridor.
- **Expected behavior:**
  - Email/password: strong password (breach-list k-anonymity check, no forced rotation); mandatory email verification (OTP/magic link) before the application proceeds.
  - SSO: Google/Apple/LinkedIn via OIDC. Apple "Hide My Email" relay supported. LinkedIn scopes (consented) pre-fill VER-5.
  - Account linking: same verified email across methods → one account (never two profiles).
  - MFA: optional at signup; **required** before deep-verification (income) and concierge tier.
  - Session: short-lived access + rotating refresh tokens; device registry; log-out-all; step-up re-auth for sensitive actions (view own documents, change email, erasure).
  - Consent captured at registration: granular, unbundled DPDP-compliant records, per-data-use toggles.
- **Edge cases & handling:**
  - Email already registered → `login + reset path; never a second account`.
  - SSO email collides with password account → `secure account-link (verify both), not duplicate`.
  - SSO returns no email (Apple relay/LinkedIn restricted) → `accept relay; require reachable contact email before admission`.
  - Breached password → `reject at set-time with reason`.
  - Credential-stuffing/brute force → `rate-limit, progressive delay, CAPTCHA, lockout with self-serve recovery`.
  - SSO token later revoked by provider → `re-auth required; badges persist (not tied to login method)`.
  - Minor (DOB < 18 or < local marriage age) → `hard block at DOB; no account; data minimized`.
  - MFA device lost → `recovery codes + identity re-verification fallback; never plain email reset for high-assurance`.
  - Network failure mid-registration → `idempotent resumable draft; no ghost accounts`.
- **Acceptance criteria:** all four methods work; one person cannot get two discoverable profiles; email ownership proven before advancing; consent granular/unbundled/withdrawable/logged; MFA enforced before deep-verification + concierge; no SSO tokens stored beyond consented scope.
- **Dependencies:** ONB-1, VER-3, VER-5, PRD §9 consent, T&S.

### ONB-3: Gender-gated admission — user-facing (women instant, men metered)
- **Description:** The master market-design lever. Verified **women skip the waitlist (instant, free)**; verified **men are admitted by a daily quota computed from active female reachable capacity**. Admission *pacing* only — verification requirements are identical across genders (ONB-6). Structural cure for the death spiral.
- **Expected behavior:**
  - Women: after identity + ≥1 eligibility verification → admitted immediately; onboarding emphasizes safety/control/respect; no queue screen.
  - Men: after identical verification → queue (ONB-4) with visible position + honest framing ("we admit balanced cohorts; a longer queue protects reply quality"). Admission fires when the female-supply quota allows.
  - Automatic throttle (invisible governor): if projected male reachability would drop or ratio nears 1.5:1, male admission slows; paid channels cannot override. Users see only their own queue movement, never the raw ratio.
  - Non-binary/undisclosed gender: policy assigns admission lane by the side each person is seeking + current balance, same rigor; never a metering loophole.
  - Messaging is screenshot-symmetric.
- **Edge cases & handling:**
  - Man asks why women are instant → `transparent "Our Standards" explainer; verification is identical, metering protects reply quality for everyone`.
  - Misdeclares gender to jump queue → `gender bound to VER-2 gov-ID; mismatch triggers re-review, not silent admission`.
  - Female supply drops → `male queue lengthens automatically; admitted men never expelled; new intake throttles`.
  - Ratio breaches 1.5:1 → `new male admission HALTS until balance restores; queued men keep position`.
  - Woman verified but inactive after admission → `counts toward supply only while reachable/active — a ghosting seed does not unlock male intake`.
  - Sanctioned-country/failed-KYC applicant → `blocked regardless of gender lane`.
- **Acceptance criteria:** verification requirements provably identical across genders (audit-testable); women reach Active without a queue step; male admission is a function of active female capacity and the throttle can't be overridden by any channel; at 1.5:1 new male admission stops with positions preserved; lane assignment uses gov-ID-verified gender.
- **Dependencies:** ONB-4, VER-2, ONB-6, ratio-governor service, fairness metrics.

### ONB-4: Queue position & queue-jump mechanics
- **Description:** The waitlist as a visible, motivating story. A lengthening male queue *increases* perceived scarcity; queue-jump rewards platform-building behaviors.
- **Expected behavior:**
  - Position display: approximate band + estimated window ("balanced cohorts weekly").
  - Legitimate jump levers: (a) referring a *verified* qualified peer (2–3× for scarce gender); (b) completing more verification badges; (c) a vouch from an existing verified member; (d) corridor-relevance boost.
  - Cohort admission: weekly ~50/50 balanced, sized to female capacity.
  - No pay-to-jump at launch (explicit anti-dark-pattern; scarcity must be real).
- **Edge cases & handling:**
  - Fake-invitee gaming → `only verified invitees move the needle; fraud clawed back + flagged`.
  - More badges but ratio-gated → `badges improve within-cohort priority, cannot breach the female-capacity ceiling`.
  - Queued man churns then returns → `position retained within grace; expired positions need re-verification only if badges lapsed`.
  - Corridor closes/pauses → `moved to market-interest with position preserved, honestly communicated`.
  - Cohort exceeds female capacity due to a supply dip → `admit to capacity; overflow rolls to next cohort keeping order`.
  - Position scraping/enumeration → `positions per-user, non-enumerable, rate-limited`.
- **Acceptance criteria:** position visible/per-user/non-enumerable; jump only via referral(verification-vested)/verification/vouch/corridor — never direct payment at launch; scarce-gender referrals carry 2–3× weight; no lever breaches the ONB-3 ceiling or 1.5:1 red line.
- **Dependencies:** ONB-1, ONB-3, VER-5/6/7, Referral loop, ratio governor.

### ONB-5: Onboarding progress, resumability & partial-verification state
- **Description:** A resilient, resumable multi-step onboarding so a network failure or incomplete verification never strands a user or creates a half-live profile — also answering the incumbent's "extremely slow / stuck on loading" reputation.
- **Expected behavior:**
  - Server-side state machine: `Registered → IdentityVerifying → IdentityVerified → EligibilityVerifying → EligibilityVerified → (Queue|InstantAdmit) → ProfileBuild → Active`.
  - Each step independently persisted; resume from the exact step.
  - Partial verification: identity-verified-but-eligibility-pending is NOT discoverable and cannot browse the live pool; sees only own progress.
  - Clear "what's left" checklist with per-signal status + SLA.
- **Edge cases & handling:**
  - Network failure mid-flow → `last step persisted server-side; resume with no loss/duplicate (idempotency keys)`.
  - Identity ok, eligibility fails → `held in EligibilityVerifying; offered alternate routes (ELG-1); not admitted`.
  - Fields done but a badge under manual review → `profile stays Pending, invisible, until badge resolves`.
  - Document to wrong slot → `re-upload allowed; superseded doc purged from vault`.
  - Session expiry mid-verification → `re-auth returns to same step; evidence retained per consent`.
  - Abandoned beyond retention → `documents purged (minimization), account → Draft, user notified`.
- **Acceptance criteria:** any step resumable after interruption with no duplicate charges/submissions/ghosts; partially verified user never discoverable and cannot browse; every pending signal shows SLA + status.
- **Dependencies:** ONB-1/2, all VER-*, ELG-1, document vault, PRD §9 retention.

### ONB-6: Symmetric verification standards across genders
- **Description:** A cross-cutting invariant: the *set and rigor* of verification is identical regardless of gender — closing the honeytrap seam and the "biased towards girls / harder on men" brand wound. Gender affects only admission *pace* (ONB-3).
- **Expected behavior:** required badge set (identity + ≥1 eligibility) and thresholds/rigor are gender-independent via a single verification policy config; fraud scrutiny/liveness strictness/document-authenticity uniform; public "Our Standards, and Why" page states this.
- **Edge cases & handling:**
  - Any config change relaxing a badge for one gender → `blocked by policy invariant + code review + audit; requires explicit logged governance override (surfaced in transparency report)`.
  - Ops verifier tries to fast-pass a female applicant → `role controls + audit log flag the deviation`.
  - Gender-differential pass/reject rates → `monitored as a fairness signal; root-caused (e.g., OCR failing on certain name scripts), never "corrected" by lowering rigor for one side`.
- **Acceptance criteria:** a single gender-agnostic policy config governs all badges (no gender branch in required-badge/threshold logic); audit test proves identical sets/thresholds; verification pass/fail parity by gender reported and any gap root-caused.
- **Dependencies:** VER-1..8, ONB-3, PRD §7.9 audit, fairness metrics.

### VER-1: Verification orchestration & badge model
- **Description:** The engine that runs each independent signal, issues per-signal badges with **provenance + verified-date**, and gates discoverability. Two-tier cost stack: cheap automated for the free pool, expensive human/deep checks payment-gated.
- **Expected behavior:**
  - Signals independently badged: identity (VER-2), education (VER-4), employer/professional (VER-5), income (VER-6, optional), marital-status (VER-7), photo authenticity (VER-9).
  - Each badge = `{signal, method, source, verified_at, expires_at, assurance_level}`.
  - Assurance levels: `source-authenticated` (registrar/employer API/gov-ID) > `email-domain-verified` > `document-reviewed` (human) > `self-attested`. Rendered so "verified" is never flat/spoofable.
  - Orchestrator routes automated-passable checks instantly; ambiguous/failed → manual review console with SLA.
  - Discoverability gate: identity + ≥1 eligibility badge at ≥ document-reviewed assurance.
- **Edge cases & handling:**
  - Automated check inconclusive → `manual review, SLA-informed, not auto-rejected`.
  - Third-party API down → `queue/retry + alternate method; never block indefinitely`.
  - Partial pass → `ONB-5 partial state; offer alternate eligibility route`.
  - Conflicting signals (ID name ≠ credential name) → `name-reconciliation, hold badge`.
  - Badge issued in error → `revocable; cascades to discoverability re-check`.
- **Acceptance criteria:** every badge records method/source/assurance/date/expiry; no profile discoverable without identity + ≥1 qualifying eligibility badge; source-authenticated used wherever an API exists; manual items have SLA + logged decision.
- **Dependencies:** VER-2..9, ELG-1, document vault, review console.

### VER-2: Government ID + liveness / selfie-match (identity)
- **Description:** Real KYC-grade identity — gov ID validated against a live selfie, meeting/exceeding the Indian govt matrimony advisory and supporting international IDs. The KYC backend the incumbent lacks. DigiLocker/Aadhaar offline-eKYC is the cheap "government-grade" first India integration (Aadhaar masked/offline only, never raw).
- **Expected behavior:**
  - Supported IDs per market (passport global; Aadhaar offline-eKYC/DigiLocker India; national ID; DL; residence permit).
  - Capture: document scan (MRZ/OCR + security-feature checks) + active liveness (anti photo/screen/deepfake) + selfie-to-ID face match above threshold.
  - Extracted & bound: legal name, DOB (→ age gate), gender (→ ONB-3 lane), nationality, doc expiry.
  - Sanctions/PEP screen. On pass: identity badge at source-authenticated; selfie anchors photo authenticity (VER-9).
- **Edge cases & handling:**
  - Expired ID → `reject; request valid document`.
  - Forged/tampered doc → `auto-reject + fraud flag + identity-linked re-registration block; escalate to review`.
  - Selfie mismatch → `reject; limited retries with fresh liveness; repeated → manual review (twins/aging/injury/lighting), not permanent auto-ban`.
  - Liveness spoof → `reject + fraud flag; hardened re-attempt`.
  - Name mismatch (ID vs credential) → `accept gov-recognized name-change evidence (marriage cert/gazette/deed poll), else hold`.
  - Minor → `hard reject; terminate; data minimized; never discoverable`.
  - Sanctioned/prohibited jurisdiction → `block; collect no further docs; log`.
  - Duplicate identity (same ID/face under different email/gender) → `block second account; link existing; investigate evasion`.
  - Aadhaar full number submitted → `never store raw; convert to masked/offline-eKYC token; purge raw`.
  - Network failure during liveness → `resumable idempotent; no partial identity badge`.
  - KYC provider outage → `queue + retry + alternate; not falsely rejected`.
- **Acceptance criteria:** identity requires valid gov ID + passed active liveness + selfie-match; expired IDs/failed liveness never pass; gender+DOB downstream come from verified ID; forged docs/spoofs produce fraud flag + identity-linked block; minors/sanctioned hard-blocked with data minimized; documents only in isolated field-encrypted vault; raw Aadhaar never persisted.
- **Dependencies:** VER-1, VER-9, ONB-3, ELG-1, document vault, sanctions/PEP, KYC/DigiLocker providers, T&S ban registry.

### VER-3: Identity-linked duplicate / ban-evasion detection
- **Description:** One real person = one account; banned/fraudulent actors cannot re-register — enabling cross-registration bans.
- **Expected behavior:** on VER-2 pass, derive privacy-preserving signals (document-number hash, face-embedding hash, device/behavioral) checked against existing accounts + ban registry. Outcomes: unique → proceed; duplicate-active → block second, offer recovery of first; duplicate-banned → block + log evasion.
- **Edge cases & handling:**
  - Legit re-registration after erasure → `allowed if not banned; erased data not resurrected`.
  - Shared device, two real people → `device signal alone never blocks; identity (ID+face) decides`.
  - Identical twins → `distinct gov IDs disambiguate; manual review if needed`.
  - False-positive duplicate → `appeal + human review; no permanent lock without confirmation`.
  - Banned user with a new ID → `face-embedding + behavioral match flags evasion for review`.
- **Acceptance criteria:** a single verified identity cannot hold two discoverable profiles; banned identities blocked across re-registration via identity-linked signals (not just email/device); decisions appealable + logged.
- **Dependencies:** VER-2, T&S ban registry, fraud models, PRD §9 erasure.

### VER-4: Education-credential verification (registrar / DigiLocker / institution-email / document review)
- **Description:** Source-authenticated degree verification — the upgrade from the incumbent's spoofable manual marksheet scan. Feeds the "verified elite-alumni credential" eligibility route. Alumni-roll cross-check is a cheap, high-trust first integration.
- **Expected behavior (ranked by assurance):** (1) registrar/DigiLocker/institution API → source-authenticated; (2) institution-email OTP → email-domain-verified; (3) alumni-roll cross-check → source-authenticated; (4) document review → document-reviewed (fallback where no API/email). Badge records institution/degree/year/method + provenance.
- **Edge cases & handling:**
  - Forged degree → `authenticity checks + cross-reference; require higher-assurance or reject; fraud flag`.
  - Institution unverifiable via any source → `document-review, badge capped; eligibility weighs assurance`.
  - Name on degree ≠ verified ID → `reconcile (maiden/transliteration/legal change) with evidence; hold badge`.
  - Institution email inactive → `use registrar/DigiLocker or document review`.
  - Diploma-mill / unaccredited → `flagged; disqualified from elite-alumni route (may still qualify via professional route)`.
  - Honorary/incomplete/dropped-out → `only completed conferred credentials earn the badge`.
  - International transcript → `certified translation + review; longer SLA communicated`.
  - Registrar API outage → `queue/retry or fall back; user informed`.
  - Multiple degrees → `each badged; highest-assurance qualifying one drives eligibility`.
- **Acceptance criteria:** source-authenticated used wherever it exists, document-only clearly lower; every badge shows institution/degree/year/method/date; name mismatches reconciled first; forged/diploma-mill never earn the elite-alumni route.
- **Dependencies:** VER-1, VER-2, ELG-1, alumni/registrar/DigiLocker integrations, review console.

### VER-5: Employer / professional verification (work-email / LinkedIn / HR attestation)
- **Description:** Verifies current employment/title/seniority — basis for the "verified premium-professional achievement" eligibility route, broadening the moat beyond "which school."
- **Expected behavior (ranked):** (1) work-email OTP → email-domain-verified; (2) LinkedIn OAuth-consented data + LinkedIn verification signals → corroborating; (3) HR attestation / employment-verification provider → source-authenticated (for founders/self-employed/senior operators); (4) document review (offer letter/incorporation) → fallback. Badge records employer/title/seniority band/method/date.
- **Edge cases & handling:**
  - Founder/self-employed → `incorporation/registry proof + HR-provider or document review; title via company registry`.
  - Between jobs → `most recent verified employment badged with date; eligibility may use prior seniority (configurable)`.
  - Consultant/contractor → `client attestation or registry + document review; assurance noted`.
  - Freemail claimed as employer → `rejected as domain-verification; require alternate`.
  - Title inflation → `badge reflects VERIFIED title, not claimed; discrepancy flagged`.
  - LinkedIn name ≠ verified ID → `reconcile`.
  - Stealth startup / confidential employer → `verified-title-with-hidden-employer (privacy) while keeping internal verification`.
  - Sanctioned employer/prohibited industry → `flagged per compliance`.
  - LinkedIn rate-limited/down → `fall back to work-email or HR attestation`.
- **Acceptance criteria:** badge reflects verified employer/title/seniority, never claimed; ≥1 non-self-attested method required for the professional route; founders/self-employed have a source-authenticated path; freemail never satisfies work-email.
- **Dependencies:** VER-1, VER-2, ONB-2 (LinkedIn SSO), ELG-1, employment-verification provider, company registries.

### VER-6: Income / net-worth verification (optional)
- **Description:** Optional, payment-gated deep verification; a threshold input for markets where the professional route uses income. Kept optional + privacy-forward to avoid "reverse dowry" commodification — income shown as a *band*, never a raw figure, never required.
- **Expected behavior:** methods = consented read-only financial aggregator, payslip/tax-document review, or CA attestation for net-worth. Output = an income/net-worth band badge with method + date; underlying figure never displayed. Absence never blocks discoverability.
- **Edge cases & handling:**
  - Declines income verification → `no badge, no penalty; use education/professional routes`.
  - Aggregator fails/revoked → `fall back to document review; tokens purged after read`.
  - Forged payslip/tax doc → `authenticity checks + cross-reference; reject + fraud flag`.
  - Income below claimed band → `badge reflects verified band`.
  - Multi-currency/overseas income → `normalized to market band at verification-date FX; method noted`.
  - Volatile founder income / equity-heavy → `net-worth attestation path; band reflects verified assets, not projections`.
  - Over-collection risk → `store only derived band + method proof reference, not full statements beyond retention`.
- **Acceptance criteria:** optional, never gates discoverability; only a band ever surfaced; forged docs rejected + flagged; aggregator tokens read-only + purged; statements minimized.
- **Dependencies:** VER-1, MFA (ONB-2), ELG-1, aggregator provider, document vault, PRD §9 minimization.

### VER-7: Marital-status attestation with escalating checks
- **Description:** Verifies marital status/intent — the vector the degree-only incumbent check ignored and precisely where matrimony fraud operates. Escalating rigor balances cost vs risk.
- **Expected behavior (escalating):** (1) self-attestation with truthfulness affirmation, logged; (2) corroborating signals (govt-record/court/marriage-registry where lawful; death cert for widowed); (3) document review (decree/death cert); (4) ongoing behavioral/velocity signals escalate to review + re-attestation. Badge shows attestation level.
- **Edge cases & handling:**
  - Bare self-attestation → `lowest assurance badge; flagged for escalation; trust-ranked accordingly`.
  - Claims divorced, no decree → `request decree for higher assurance; else shows self-attested only`.
  - Married posing as single (report/cross-record/behavior) → `escalate; on confirmation suspend + identity-linked ban — the honeytrap seam closed`.
  - Separated not divorced → `distinct status; not shown as "divorced"`.
  - Widowed → `death-certificate path; sensitive handling`.
  - No accessible registry → `document review + attestation + behavioral monitoring; assurance capped, disclosed`.
  - Annulment/foreign divorce → `document review + translation`.
  - False attestation later found → `retroactive revocation, suspension, ban, safety escalation if harm`.
- **Acceptance criteria:** at minimum affirmed self-attestation logged with escalation paths; divorced/widowed backed by document review for higher assurance; concealment triggers suspension + identity-linked ban; escalation identical across genders.
- **Dependencies:** VER-1, VER-2, T&S, review console, registry integrations where lawful.

### VER-9: Photo authenticity (liveness-linked, anti stock/steal)
- **Description:** Ensures profile photos are genuinely of the verified person — anti stock/stolen/deepfake — anchored to the VER-2 liveness selfie.
- **Expected behavior:** each photo checked for (a) face-match to VER-2 selfie anchor, (b) reverse-image/stock detection, (c) cross-platform duplicate detection, (d) deepfake/synthetic detection. Passing photos earn a "photo verified" indicator; primary must be verified. Consent-gating (blur/reveal) is orthogonal — a blurred photo is still authenticity-checked underneath.
- **Edge cases & handling:**
  - Photo ≠ verified selfie → `not verified; can't be primary/authenticated; prompt genuine photo`.
  - Stock/celebrity/scraped → `rejected + flagged`.
  - Photo already used by another account → `flagged for impersonation; earlier-verified owner protected`.
  - Deepfake/AI face → `rejected + fraud flag; may trigger identity re-verification`.
  - Heavy filters/old photo/appearance change → `soft-fail + re-capture prompt; manual review, not hard ban`.
  - Group photo → `secondary only if identifiable; never sole primary`.
  - Wants no public photo → `allowed; still uploads a private verified reference; discovery respects photo-visibility`.
- **Acceptance criteria:** primary photo face-matched to VER-2 selfie; stock/scraped/duplicate/deepfake detected + rejected/flagged; authenticity runs even when displayed blurred; appearance-change soft-fails route to human review.
- **Dependencies:** VER-2, VER-3, T&S fraud models, PRD §7.2 privacy/media.

### VER-8: Re-verification cadence, expiry & verification downgrade
- **Description:** Verification isn't permanent — a cadence flags stale attestations (employment/ID/marital/photo). Prevents the stale-pool problem.
- **Expected behavior:** per-badge `expires_at`/cadence; approaching expiry → nudge + grace. Downgrade path: on expiry without re-verification, badge drops assurance or is removed; if this leaves the profile below the discoverability gate, profile → `Limited/Hidden` until re-verified (never deleted). Lightweight delta re-verification where a prior source record exists.
- **Edge cases & handling:**
  - ID expired since onboarding → `identity badge downgraded; prompt current ID; discoverability paused if identity lapses`.
  - Changed jobs (detected) → `employment badge flagged stale; prompt re-verify; old badge marked "as of <date>"`.
  - Marital status changed → `re-attestation; concealment detection applies`.
  - Ignores re-verification nudges → `graduated: reminder → downgrade → hidden → inactive`.
  - Re-verification fails → `badge removed; eligibility re-evaluated; may drop below gate`.
  - Source no longer available (defunct institution/employer) → `retain prior badge "verified as of <date>, source since unavailable"; don't falsely re-issue`.
  - Long-inactive returner → `full re-verification of expired badges before re-entering discovery`.
- **Acceptance criteria:** every badge has expiry/cadence + re-verification path; expiry downgrades and hides (never silently keeps discoverable) if the gate is unmet; job/marital changes trigger prompts; re-verification reuses prior source records via delta.
- **Dependencies:** all VER-*, ELG-1, discoverability gate, notifications, fairness metrics.

### VER-10: Verification appeal & manual re-review
- **Description:** A fair human appeal path for any automated rejection/revocation — biometric/document checks have false positives (twins, name scripts, appearance change) and a trust brand can't afford wrongful exclusion.
- **Expected behavior:** any rejection/downgrade/duplicate-block offers an appeal with reason codes + evidence upload; routed to human reviewers (higher authority than the automated decision) with SLA; decision logged with rationale. Outcomes: overturn / uphold (with explanation) / request-more-evidence.
- **Edge cases & handling:**
  - False-positive forged-doc rejection → `reviewer overturns on genuine evidence; fraud flag cleared; block lifted`.
  - False-positive duplicate (twin/shared device) → `disambiguate via distinct IDs; second account allowed`.
  - Selfie-match false negative (appearance change) → `supervised re-capture; overturn`.
  - Repeated frivolous appeals → `rate-limited; abuse flagged`.
  - Genuinely fraudulent case → `upheld; evidence retained for ban registry`.
  - Reviewer conflict/sensitive PII → `role-scoped, least-privilege, consented-evidence-only`.
- **Acceptance criteria:** every automated rejection/downgrade/duplicate-ban appealable; human review within SLA with logged rationale; overturns clear associated flags/blocks; upholds retain evidence; reviewer access role-scoped + consent-bounded.
- **Dependencies:** VER-1..9, VER-3, PRD §7.9 back-office, PRD §9 retention/audit.

### ELG-1: Eligibility engine (dual-route, per-market thresholds)
- **Description:** Admits on **either** a verified elite-alumni credential **or** verified premium-professional achievement (title/seniority/employer/income) — **no single hard institute list.** The strategic pivot that grows liquidity AND defuses the elitism backlash. Thresholds configurable per market.
- **Expected behavior:**
  - Route A — Elite alumni: a VER-4 credential from a qualifying institution (a broad, curated, non-supremacist accredited set — "top alumni," not a 3-institute hierarchy) at sufficient assurance.
  - Route B — Premium professional: a VER-5 employment badge meeting the market's title/seniority/employer criteria and/or a VER-6 income band (founders, doctors, researchers, senior operators…).
  - Discoverability gate: identity (VER-2) + ≥1 satisfied route at ≥ document-reviewed assurance.
  - Per-market config: thresholds, qualifying institution set, income bands, title/seniority ladders — no global hardcoded list. Assurance-weighted. Never ranks institutes/people publicly.
- **Edge cases & handling:**
  - Qualifies on neither → `not admitted; transparent non-hierarchical criteria + alternate routes; no "not elite enough" language`.
  - Borderline professional (just below title threshold) → `configurable review lane; may admit via combined signals (mid-title + income); human review`.
  - Elite degree but junior/unemployed → `qualifies via Route A (credential is durable)`.
  - Non-elite degree but exceptional professional → `qualifies via Route B (the point of broadening)`.
  - Diploma-mill degree → `Route A denied; may qualify via Route B`.
  - Cross-market applicant → `re-evaluated against target corridor thresholds; badges port, thresholds local`.
  - Threshold change over time → `grandfather already-admitted; apply to new admissions only; logged`.
  - Title-inflation gaming → `uses verified values only`.
  - Sanctioned employer/jurisdiction → `ineligible regardless of achievement`.
  - Ambiguous/unlisted institution → `eligibility review; assurance-capped, logged`.
- **Acceptance criteria:** admission via Route A OR B (neither mandatory alone); no single hardcoded institute list (market-configurable); uses only verified values weighted by assurance; ineligibility messaging non-hierarchical with alternate routes; threshold changes logged + grandfathered; criteria/rigor identical across genders.
- **Dependencies:** VER-2/4/5/6, VER-1, ONB-3/5, per-market config service, PRD §14 calibration, compliance overrides.

### ELG-2: "Our Standards" transparency & eligibility explainability
- **Description:** A public at-launch "Our Standards, and Why" surface + per-applicant eligibility explainability — the proactive inoculation against elitism/opacity backlash and the antidote to the incumbent's opaque posture.
- **Expected behavior:** public page states symmetric verification, dual routes, broad non-hierarchical accomplishment, zero caste ever, "verified ≠ vouched." Each applicant sees which route(s) they qualified/didn't qualify for + how to add signals — never a ranked/derogatory verdict. Recurring honest verification/fairness metrics report published.
- **Edge cases & handling:**
  - Applicant disputes outcome → `route to ELG review / VER-10 appeal with human explanation`.
  - Criteria perceived as elitist → `messaging uses behavior/standard words, never identity/hierarchy; "screenshot side-by-side" test; hypocritical-across-genders lines cut`.
  - Caste/protected attribute requested anywhere → `blocked by policy; community/religion optional + buried, never eligibility`.
  - Metrics unflattering → `still published honestly (the differentiator)`.
- **Acceptance criteria:** public standards page exists at launch; every eligibility decision explainable with add-signals path + no hierarchical language; no eligibility path consumes caste/protected attribute; recurring honest metrics report published.
- **Dependencies:** ELG-1, ONB-6, VER-10, fairness metrics, GTM §5.
