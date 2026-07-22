# Feature Spec — Trust & Safety, Fraud, Privacy & Compliance

> Domain 5 of the [Feature Specification](../FEATURE-SPEC.md). Companion: [PRD](../PRD.md) §7.5/§8/§9, [research dossier](../research/market-research.md) §5/§8, [female-first brief](../gtm/raw/04-female-first-acquisition-safety.md).

**Design frame — two divergent threat models the incumbent conflates:** **Threat A** — the member is a *fraud/romance-scam target* (countered by verification + duplicate/stock-photo detection + scam nudges + "verified≠vouched"); **Threat B** — the member is a *harassment/exposure target* (countered by silent block, identity-linked bans, anti-harassment filtering, escalation lane, masking, scrape defense). Because the incumbent's verification "verifies the wrong thing" (a degree, not identity/intent/marital-status), our T&S layer never treats a verified badge as vouched character. **VER** = Domain 1, **PROF** = Domain 2, **COMM** = Domain 4, **BO** = back-office/admin (Domain 6).

---

## Report user

### SAFE-01: Report user — reasons, evidence, submission
- **Description:** In-context reporting from profile, chat, or received-interest surface — the primary human sensor feeding both threat models. Closes the incumbent's absence of fraud/abuse controls beyond a one-time doc scan; answers "inline one-tap" report with "status told back."
- **Expected behavior:**
  - Entry points: profile, chat thread, received-interest card, who-viewed-me, search overflow. One tap opens the sheet.
  - **Reason taxonomy → triage lane:** harassment/abusive → Threat-B; unsolicited explicit → Threat-B + auto-pull filter samples (SAFE-05); contact-fishing/off-platform → Fraud + scam-nudge (SAFE-08); asking for money → **escalation candidate** (SAFE-07) + Fraud; fake profile/stolen photos → Fraud (FRAUD-03); married/misrepresented status → Fraud + VER re-attestation; impersonation → Fraud, high priority; threat/extortion/sextortion → **immediate escalation** (SAFE-07), highest SLA; minor/underage → immediate hard lane (SAFE-07 + legal); spam → Fraud, low; other → free-text general.
  - **Evidence:** up to N screenshots + select specific in-thread messages (server-side message IDs preferred over uploaded images — tamper-resistant) + free-text (≤2,000 chars). All copied into an immutable evidence snapshot at submission.
  - On submit: reporter sees confirmation + case reference + expected-response window (SAFE-04); offered inline block/mute independent of case outcome. Reporting available to any authenticated member incl. free tier; no payment gate on safety.
- **Edge cases & handling:**
  - Reporter reports same user twice → `dedupe into existing case; no second case; SLA clock doesn't reset`.
  - Multiple reporters on same target → `auto-cluster into one case with rising severity; velocity of independent reports escalates priority`.
  - No-evidence vague "other" → `accepted, low-confidence; triage may rely on behavioral signals`.
  - Screenshot contradicts server log (doctored) → `moderator sees both; server record authoritative; possible retaliation flag on reporter`.
  - Target already banned → `report still recorded, attached to identity cluster (SAFE-06) so evidence accrues against evasion; reporter told "account no longer active, thank you"`.
  - Reporter blocks then reports → `block persists; report proceeds; can still see case status`.
  - Threat/underage/extortion reason → `never force the reporter to re-view/re-upload the disturbing content; one tap suffices; auto-preserve`.
  - Reporter under active moderation → `report still accepted (an offender can also be a victim); status noted for context, never auto-dismiss`.
  - Report against a system/concierge/RM account → `separate internal-conduct queue`.
- **Acceptance criteria:** every reason maps to exactly one primary lane with a defined SLA; submitting creates exactly one case (or appends) + returns a durable reference within 1s p95; selected in-thread messages stored as server IDs + immutable snapshot; no safety path gated behind payment/tier; threat/underage/extortion never force re-surfacing the content.
- **Dependencies:** SAFE-02/03/04/05/06/07, FRAUD-*, COMM (chat store), BO (moderation console).

### SAFE-02: Report triage & routing engine
- **Description:** Automated first-pass scoring/prioritization/routing before human review + false/retaliatory-report detection. Implements "every report actioned within SLA" and the staffed female-informed T&S commitment.
- **Expected behavior:** each report scored on reason severity, independent-reporter count, target fraud/behavioral risk (FRAUD-01), target moderation history, reporter credibility, auto-filter hits (SAFE-05). Routes to lanes: **Escalation (P0)**, **Fraud (P1/P2)**, **Harassment (P1/P2)**, **General (P3)**. Auto-actions permitted pre-human-review: temporary reversible shadow-limit on outbound messaging for targets crossing a hard behavioral+report threshold (logged, SAFE-10) — never a permanent ban without human confirmation except identity-cluster propagation of an already-human-confirmed ban (SAFE-06). Female-informed weighting elevates baseline priority for marital-status misrepresentation, contact-fishing, unsolicited explicit. **False/retaliatory-report detection:** mass-reporting after rejection/block, coordinated rings, reporting-immediately-after-being-reported flag the reporter; substantiated-false reports degrade credibility and can become their own case.
- **Edge cases & handling:**
  - Retaliation (target reports the reporter with a mirror reason) → `both cases linked; moderator sees reciprocity/timing; neither auto-actioned; adjudicated together`.
  - Brigade (10 new/low-history accounts report one high-credibility woman) → `brigade heuristic caps aggregate severity from low-trust reporters; flagged "possible coordinated false reporting"; target protected from auto shadow-limit`.
  - High-credibility reporter vs high-value paying target → `payment/tier NEVER lowers priority or protects target`.
  - Reason ≠ evidence ("spam" but a death threat attached) → `classifier can upgrade lane; downgrades require human sign-off`.
  - Report volume spike (incident/bot wave) → `shed low-priority load, but P0 escalation never throttled; overflow paged to on-call`.
  - First-ever reporter, unknown credibility → `neutral baseline, not penalized`.
  - Target is a founding-cohort/ambassador role → `no leniency; flagged only so a senior moderator handles`.
- **Acceptance criteria:** every report lands in exactly one lane with a numeric priority within 60s; no permanent ban by the automated layer (only reversible limits or propagation of a prior human ban); coordinated-report/retaliation heuristics measurably suppress auto-actioning of brigaded targets in fixtures; payment tier has zero weight in scoring (config-audit).
- **Dependencies:** SAFE-01/04/05/06, FRAUD-01/04, BO, SAFE-10.

### SAFE-04: Report SLA, status lifecycle & status-told-back
- **Description:** Time-bound resolution + closed-loop reporter communication ("reports actioned within SLA ≥99%") — the differentiator against the incumbent's silent, unresolved reporting.
- **Expected behavior:**
  - **SLA by lane:** P0 (threats/minor/extortion) — ack ≤15 min, first human action ≤1 hr, 24/7 on-call; P1 ≤4 hr; P2 ≤24 hr; P3 ≤72 hr. Configurable per region/staffing.
  - **Case states:** `Submitted → Triaged → Under review → (Action taken | Dismissed | Escalated) → Closed`, with `Reopened` on new evidence/appeal.
  - **Status-told-back:** reporter notified at Triaged and Closed with an outcome category, **without target PII or the specific sanction**. Allowed: "We reviewed and took action," "We didn't find a violation, here's how to stay safe / add more info," "We've escalated this." Never disclose ban/suspension specifics, target PII, or internal notes. In-app case status page by reference.
  - SLA breaches auto-escalate priority + page a supervisor; breach metrics feed the §10 KPI dashboard.
- **Edge cases & handling:**
  - Reporter demands to know exactly what happened → `outcome category only + privacy rationale + "verified≠vouched" education (SAFE-09)`.
  - Case dismissed but same target re-offends against same reporter → `new report auto-links; prior dismissal surfaced as context raising scrutiny; reporter not told "already dismissed" dismissively`.
  - SLA clock during regional off-hours → `P0 always 24/7; lower lanes may use business-hours-aware SLA per region but must display the honest expected window at submission`.
  - Reporter deletes their account before closure → `case proceeds on merits; status-told-back suppressed (no recipient); evidence retained per SAFE-07/PRIVC-04`.
  - Target account deleted mid-review → `closes "account no longer active"; evidence + identity cluster retained for evasion detection (overrides soft-delete per fraud-retention exception)`.
  - Outcome "warned" → `reporter told "action taken"; repeat warns roll up`.
  - Mass incident → `status-told-back batched; each reporter still receives closure`.
- **Acceptance criteria:** each lane has a configured/displayed SLA; ≥99% close within lane SLA (dashboarded); every reporter with a live account gets Triaged + Closed with an outcome category and zero target-PII/sanction-specifics; SLA breach fires auto-escalation + supervisor page (logged); all state transitions in the immutable audit (SAFE-10).
- **Dependencies:** SAFE-01/02/07/09/10, MOD-01, Notification, PRIVC-04.

---

## Moderation console & audit

### MOD-01: Moderation console actions (warn / suspend / ban / dismiss)
- **Description:** The staffed moderator's action surface with a sanction ladder + mandatory reason codes ("moderator actions with audit trail," "role-scoped and audit-logged").
- **Expected behavior:**
  - **Actions:** Dismiss; Warn (in-app, logged, counts toward ladder); Restrict/feature-limit (messaging cap, remove from discovery, photo-post block) with duration; Suspend (time-boxed lockout, member notified with reason category + appeal); Ban (permanent, identity-linked via SAFE-06); Escalate (SAFE-07); Request more info; Force re-verification (route to VER).
  - Every action requires a **reason code**, free-text rationale, and evidence links; actions without a reason code are blocked.
  - **Ladder (configurable):** dismiss ↔ warn → restrict → suspend → ban, with severity overrides — threats/extortion/minor/verified-impersonation/confirmed romance-scam jump straight to suspend/ban.
  - Bulk action across an identity cluster (SAFE-06): banning one node offers "apply to all linked accounts."
  - Member-facing messaging: suspended/banned member sees a reason **category** (never the reporter's identity or that a report existed vs proactive detection) + appeal route (SAFE-11).
  - **Four-eyes rule:** permanent bans + actions on flagged (founding-cohort/ambassador/staff-adjacent) accounts require a second moderator/supervisor.
- **Edge cases & handling:**
  - Ban without selected evidence → `blocked; must attach ≥1 evidence item or explicitly select "proactive/behavioral basis" with the FRAUD signal snapshot`.
  - Moderator acts on own account/personal connection → `conflict-of-interest guard: can't action a case where they are reporter/target/linked contact; reassigned`.
  - Reason code contradicts severity (dismiss with "credible threat") → `hard warning + supervisor review flag`.
  - Paying/concierge target → `no leniency; billing/refund handled separately; decided on merit`.
  - Same evidence re-actioned → `detect already-adjudicated evidence; prevent stacking sanctions on identical facts absent new evidence`.
  - Warn issued but never opened → `still counts; delivery status logged; escalation not blocked by non-acknowledgment`.
  - Suspension expires → `auto-reinstatement unless a superseding ban; logged; member notified`.
  - Ban with active mutual chats → `counterparty's threads show user unavailable; banned member's reason never disclosed to them`.
  - Wrongful ban discovered → `reversal is itself an audited action (SAFE-10), never a silent DB edit; member restored; cluster flags cleared with audit`.
- **Acceptance criteria:** no sanction without reason code + rationale + evidence link (or explicit behavioral-basis attachment); permanent bans + flagged-account actions require documented second-approver; every action produces exactly one write-ahead immutable audit record before effect; conflict-of-interest guard prevents self/linked adjudication; sanctioned member sees reason category + appeal, never reporter identity/report existence.
- **Dependencies:** SAFE-06/07/10/11, VER (force re-verify), BO (RBAC), FRAUD-01.

### SAFE-10: Immutable moderation & safety audit log
- **Description:** Append-only, tamper-evident ledger of every moderation, escalation, vault-access, consent, and erasure action ("moderation actions logged immutably," "audit-logged") — underpins law-enforcement readiness and compliance defensibility.
- **Expected behavior:** every actor action (human or automated) affecting a member's safety/account/data state writes an immutable record: actor ID + role, UTC+region timestamp, case ref, action type, reason code, rationale, evidence references (by content-hash, not PII copies), before/after state, second-approver ID where applicable. Tamper-evidence: hash-chained (each record includes the prior's hash) and/or WORM storage; periodic checkpoint anchoring; gaps/mismatch raise an integrity alarm. Reads are logged. Retention under the fraud-and-legal exception (PRIVC-04) even when underlying member data is erased (records reference hashes/case IDs). Segregation of duties: moderators can't edit/delete; even admins can only append (a correction is a new record referencing the old).
- **Edge cases & handling:**
  - Attempt to delete/edit a record → `impossible by design (append-only); attempt logged + alarmed`.
  - Erasure covers a member with safety history → `member-facing PII erased/anonymized (PRIVC-04) but audit records referencing pseudonymous case/hash IDs persist; audit stores no clear-text PII beyond legal basis`.
  - Hash-chain break detected → `integrity alarm, freeze dependent automated actions, security incident opened`.
  - Automated action (SAFE-02 shadow-limit) → `logged with actor="system:<rule-id>", same schema`.
  - Cross-region case → `record written to the governing residency region's audit store; global safety index holds only pointers/hashes`.
  - LE export → `signed, scoped export (SAFE-07) + logs the export event (who/scope/legal basis)`.
- **Acceptance criteria:** 100% of MOD/SAFE/PRIVC state-changing actions produce a write-ahead immutable record before effect; records hash-chained + continuous integrity verification alarms on any break; no API/role updates or deletes an existing record (corrections append-only); audit reads are themselves audited.
- **Dependencies:** all MOD/SAFE/PRIVC; BO RBAC; PRIVC-03/05.

### SAFE-11: Appeals & wrongful-action reversal
- **Description:** Due-process for members who were warned/restricted/suspended/banned + controlled reversal — required so identity-linked bans (a powerful weapon) stay fair.
- **Expected behavior:** sanctioned member sees "appeal this decision" with a bounded window (e.g., 30 days; longer for permanent bans); collects statement + optional evidence. Routes to a **different** moderator/senior reviewer than the original actor. Outcomes: Upheld / Reduced / Overturned / Overturned-with-conditions (reinstate + mandatory re-verification) — each audited. On overturn: account + cluster flags cleared; member notified; wrongly propagated cluster bans lifted for genuinely unrelated accounts. Abuse guard: frivolous appeals rate-limited; appeals on threat/extortion/minor bans require senior + safety-team review and are never auto-reduced.
- **Edge cases & handling:**
  - Banned scammer appeals with fabricated documents → `reviewer cross-checks against fraud evidence + identity cluster; forged appeal evidence strengthens the ban and is preserved`.
  - Genuine false-positive (shared family/corporate-NAT device cluster) → `inspect FRAUD-01 signal provenance; if the only linkage is shared IP/device, reinstate + add exclusion so the shared signal alone doesn't re-trigger (FRAUD-05)`.
  - Appeal window elapsed → `may still request review for hard categories (wrongful minor/threat flag), not low-severity lapsed cases (configurable)`.
  - Member appeals a silent block → `silent blocks are user-to-user, NOT platform sanctions; not appealable and never confirmed to the blocked user`.
  - Reinstated member immediately re-offends → `prior appeal history weights next case toward faster escalation`.
  - Cross-border appeal → `single global identity cluster; handled by the region owning the original action; residency respected`.
- **Acceptance criteria:** appeal reviewer always distinct from the original actor; overturns propagate cluster-flag clearing + fully audited; threat/extortion/minor bans can't be auto-reduced (senior + safety review); silent blocks never surfaced as appealable/confirmed.
- **Dependencies:** MOD-01, SAFE-03/06/10, FRAUD-01/05, PRIVC-05.

---

## Blocking & anti-harassment

### SAFE-03: Silent block (invisible to blocked user)
- **Description:** A member blocks another such that the blocked user perceives no change — the blocker simply disappears, with no "you were blocked" signal. Protects the constrained side from retaliation an overt block invites (Threat B).
- **Expected behavior:** on block, the blocker vanishes from the blocked user's search, discovery, who-viewed-me, received-interest lists; any existing chat becomes inert on the blocked user's side, indistinguishable from ordinary inactivity (messages appear "sent" but never deliver; no read receipts; no "blocked" indicator). Blocker's side: thread archived/hidden or labeled "blocked" (only to the blocker); blocked user can't initiate interest, view blocker's profile (renders generic "profile unavailable," same as deactivated), or appear in blocker's surfaces. Mutual invisibility; only the blocker knows. Independent of Report + platform sanctions — a personal control needing no justification. Block list private to the blocker.
- **Edge cases & handling:**
  - Blocked user tries to open old chat / send → `send appears to succeed locally; dropped server-side; no delivery, no error revealing block. Default discard (not queued for post-unblock delivery, to avoid a flood on unblock)`.
  - Blocked user searches criteria that would surface blocker → `blocker excluded; result count adjusts naturally (no "1 hidden" tell)`.
  - Shared surface (both liked same event) → `blocker suppressed`.
  - Blocker un-blocks → `future interactions resume; NO backfill of messages sent during block; profiles reappear normally`.
  - Blocked user already screenshotted → `out of scope for block; scrape/screenshot defenses apply independently`.
  - Timing/side-channel (visible seconds ago, now gone) → `same rendering as deactivating/going incognito; incognito + blocks share the "unavailable" rendering to prevent inference`.
  - Blocked user reports blocker → `report triaged normally; block state not disclosed to moderators in a way that outs the blocker unfairly, but available in case context`.
  - Both block each other → `idempotent; still silent to each`.
  - Scammer probing whether blocked → `no probe (message-not-read, profile-gone, disappeared-from-search) yields a definitive "blocked" answer; all mimic benign states`.
- **Acceptance criteria:** no API response/delivery receipt/result count/error/timing lets a blocked user distinguish "blocked" from "inactive/deactivated/incognito"; blocked user's messages never delivered and (default) never backfilled on unblock; block needs no reason and is never surfaced to the blocked user, incl. via appeals.
- **Dependencies:** COMM (chat), PROF (visibility/incognito rendering parity), SAFE-01 (independent), scrape/screenshot defenses (PROF).

### SAFE-05: Anti-harassment message filtering (opt-in reveal)
- **Description:** Inbound classifier quarantining unsolicited explicit content, abuse, and contact-fishing before the recipient sees it, with recipient opt-in to view ("no contact-info leakage before consent"). Protects the constrained side from the "unusable flood."
- **Expected behavior:** every inbound message + image scored across sexual-explicitness, abuse/toxicity, contact-fishing (phone/email/social/off-platform patterns incl. obfuscated — "num8er," "insta @…"). Actions by band: clean → deliver; contact-fishing (pre-consent) → strip/redact inline, deliver with a "contact sharing disabled until mutual consent" notice (SAFE-08), log for FRAUD; abusive/explicit unsolicited → quarantine ("1 message was filtered" with **opt-in reveal** / Keep hidden / Report / Block). Recipient controls per-conversation + global sensitivity, default protective for the constrained side. Sender NOT told it was filtered (prevents evasion tuning). Repeated hits feed SAFE-02 + FRAUD-01. Quarantined content retained in a moderation-accessible store for evidence even if the recipient keeps it hidden.
- **Edge cases & handling:**
  - Consensual explicit between long-matched adults → `per-conversation opt-in to lower filtering after established contact; no bypass for first-contact/unsolicited`.
  - False positive (benign flagged) → `"View anyway" reveals; feedback tunes model (FRAUD-05); sender not penalized on a single low-confidence hit`.
  - Obfuscated contact info → `fuzzy/rule+ML detection; redact + nudge; log obfuscation as higher-risk`.
  - Explicit image → `edge nudity classifier; quarantined blurred placeholder + opt-in reveal; never auto-displayed`.
  - Contact split across messages ("my number is nine"…) → `cross-message aggregation window detects assembled contact/scam pattern`.
  - Opt in then report → `filtered content already preserved; report links directly`.
  - Codeswitch (Hinglish/transliteration) → `multilingual models; unsupported-language + high-risk routes to human triage, not silent pass`.
  - Contact-fishing after mutual consent → `allowed; filtering relaxes to abuse/explicit only`.
  - Filtered term is a real name/word → `context model + whitelist reduce collateral; reveal always available`.
  - Recipient incognito / silent-blocked sender → `block supersedes filter (message dropped, SAFE-03); filter store still logs for evidence`.
- **Acceptance criteria:** unsolicited explicit/abusive never auto-rendered (recipient opts in per item); pre-consent contact info redacted in-delivery, obfuscated + split forms detected in fixtures; sender receives no filtering signal; filtered content retained for moderation regardless of recipient view choice; recipient can adjust global + per-conversation sensitivity, consensual relaxation never bypasses first-contact protection.
- **Dependencies:** COMM, SAFE-01/02/03/08, FRAUD-01/04/05, §7.6 i18n, PRIVC-03 (quarantine encryption).

---

## Fraud detection

### FRAUD-01: Device & behavioral signal engine
- **Description:** Continuous risk scoring from device fingerprints + behavioral telemetry to detect fake/scam/duplicate accounts and automation ("device/behavioral signals… velocity checks"). Counters Threat A at the source and feeds every downstream control.
- **Expected behavior:** collects (consented, minimized) device fingerprint, OS/app integrity/root-jailbreak/emulator, IP+ASN+geo, VPN/proxy/datacenter-IP detection, session cadence, typing/interaction rhythm anomalies, funnel-timing anomalies (instant profile completion, scripted messaging), location-vs-claimed-residence mismatch, impossible-travel. Produces a rolling risk score + reason factors; thresholds trigger soft friction (extra verification/challenge), shadow-limit (SAFE-02), or case creation. Links accounts sharing strong device/identity signals into identity clusters (SAFE-06). Feeds SAFE-05, FRAUD-03/04.
- **Edge cases & handling:**
  - Legit shared device/IP (family, corporate NAT, university, diaspora household) → `device+IP alone never bans; requires corroborating signals; exclusion list for known shared-egress ranges; appeals create allow-pairs`.
  - Privacy-conscious legit user on VPN → `VPN alone a weak signal, not disqualifying; combined with other scam signals it elevates`.
  - Sanctioned-region access → `routed to compliance (PRIVC-06), not a generic fraud ban`.
  - Attacker rotates devices/IPs → `behavioral + credential (identity hash, SAFE-06) linkage persists across rotation; velocity across the rotation itself is a signal`.
  - False positive throttles a real user mid-conversation → `friction reversible; escalates to human before hard action; user-visible challenge (re-verify), not silent lockout`.
  - Anti-fingerprinting browser / new legit device → `low-history + high-risk-context triggers step-up verification, not ban`.
  - Signal collection vs consent/DPDP → `declared under legitimate-interest/fraud-prevention lawful basis (PRIVC-01), documented, minimized, region-scoped (PRIVC-05); no repurposing for matching/ads`.
- **Acceptance criteria:** risk score explainable (top reason factors to moderators); no single signal (IP/device/VPN) alone causes a hard sanction (multi-signal or human confirmation required); clustering links ban-evasion fixtures across device/IP rotation via identity hash; all collected signals documented against a lawful basis and region-scoped.
- **Dependencies:** VER, SAFE-06, FRAUD-02/03/04/05, PRIVC-01/05, SAFE-11.

### FRAUD-02: Velocity & rate-limit checks
- **Description:** Detection of abnormal rates of registration, messaging, interest-sending, profile views, verification attempts (bots, scam rings, scrapers). Defends the enumeration/scrape weakness the incumbent had.
- **Expected behavior:** per-account and per-cluster velocity windows on signups/device/IP/hour, messages/min-hour, interests/day, profile-detail fetches/hour (scrape signal), contact-reveal attempts, verification retries, password/2FA attempts. Adaptive: tighter for new/low-trust, looser for established verified (the constrained side's inbound is capped by design separately, COMM). On breach: challenge → temporary rate-cap → shadow-limit → case. Scrape-pattern fetch velocity → session throttle + non-enumerable-ID enforcement + alert.
- **Edge cases & handling:**
  - Legit power user (high but human) → `human-plausible cadence within generous ceilings; only super-human/scripted trips; avoid penalizing genuine engagement`.
  - Scraper via authenticated session → `high read-velocity + breadth of unrelated profiles + no messaging → throttle + flag + possible session revoke; PROF non-enumerable IDs prevent guess-enumeration, velocity catches crawl-via-search`.
  - Distributed scrape across many low-activity accounts → `cluster-level aggregation catches what per-account misses`.
  - Registration burst from an event/campaign (legit) → `allowlist known vectors; step-up verification, not blanket block during spikes`.
  - Attacker paces below per-account limits → `cluster + behavioral + duration-based cumulative caps catch slow-and-wide`.
  - 2FA/login brute force → `account-protection velocity locks with user-recoverable path, logged`.
- **Acceptance criteria:** configurable velocity limits per action/trust-tier/cluster; scrape-pattern read velocity triggers throttle + alert without enumerable IDs available; legit high-activity users not hard-sanctioned by velocity alone (challenge-first).
- **Dependencies:** FRAUD-01, SAFE-06, PROF (non-enumerable IDs), COMM (inbound caps), SAFE-10.

### FRAUD-03: Duplicate & stock-photo / face-reuse detection
- **Description:** Detection of stolen/stock/AI-generated/reused profile photos and the same face across accounts ("photo authenticity (anti–stock/steal)," "duplicate-photo/stock-photo detection"). Core Threat-A defense (catfishing/romance-scam).
- **Expected behavior:** on upload + periodically, perceptual-hash + face-embedding comparison against (a) stock/reverse-image corpora, (b) prior platform photos, (c) known-scam image blocklists, (d) AI-gen/deepfake detectors. Cross-account face match links accounts (SAFE-06). Liveness linkage (VER): profile photo must correspond to the liveness-verified identity; mismatch flags. Results feed risk score + can gate discoverability until resolved.
- **Edge cases & handling:**
  - Legit self-reuse across a re-registration/second legit account → `face-match to same verified identity is expected; not fraud; resolved via identity linkage`.
  - Twins/look-alikes → `near-match but distinct identity docs → not auto-banned; human review if other signals present`.
  - Model/actor/public figure with real photos in stock corpora → `reverse-image hit on a genuinely-them image; liveness+ID resolves; not penalized`.
  - AI-generated claiming real → `deepfake classifier + liveness challenge; failure blocks discoverability, routes to VER + fraud case`.
  - Scammer uses a stolen photo of a real member → `face-match across accounts + which one passed liveness for that identity determines the impostor; impostor flagged, victim protected/notified`.
  - Heavy filters/editing degrade confidence → `low-confidence → step-up liveness, not accusation`.
  - Biometric privacy → `face embeddings in the isolated vault (PRIVC-03), field-encrypted, access-brokered, region-scoped, retained per minimization; biometric processing under explicit consent (PRIVC-01)`.
  - False stolen-photo report weaponized → `automated match authoritative over a human accusation; if images don't match/aren't stock, report dismissed + may flag reporter`.
- **Acceptance criteria:** photos checked against stock/reuse/AI-gen/known-scam corpora + platform history before discoverable; same-face-across-accounts links to identity cluster + raises risk; embeddings only in the encrypted vault (never the profile DB) with biometric consent recorded; legit self-reuse + look-alikes not auto-banned.
- **Dependencies:** VER (liveness), SAFE-06, FRAUD-01, PRIVC-01/03/04/05.

### FRAUD-04: Scam-pattern & romance-fraud models
- **Description:** ML + rules detecting romance-scam, financial-fraud, honeytrap patterns from message content, conversation trajectory, and profile signals ("scam-pattern models") — targets the honeytrap seam loosened female-side verification exploits.
- **Expected behavior:** signals — rapid intimacy escalation/"love bombing," early push off-platform, requests for money/gift-cards/crypto/bank details, "overseas emergency"/stranded/medical/customs narratives, inconsistent life details, claimed-location vs device-geo mismatch, scripted/templated reuse across recipients, contact-fishing, targeting patterns (systematically messaging recently-widowed/older/HNW). Outputs per-conversation + per-account scam-likelihood → drives nudges (SAFE-08), filtering (SAFE-05), risk score, case creation — with escalation to SAFE-07 when money+threat co-occur. Cross-recipient correlation detects the same script sent to many members even if each single conversation looks marginal.
- **Edge cases & handling:**
  - Legit long-distance couple discussing travel/money → `nudge (SAFE-08) protects, but no auto-ban on content alone; hard action needs corroborating fraud signals`.
  - Genuine early move-to-WhatsApp between two matched verified adults → `contact-reveal is consent-gated (COMM); nudge educates; suspicious only with scam co-signals`.
  - Scammer adapts language to evade keywords → `embedding/trajectory models over keyword lists; cross-recipient script similarity catches paraphrase`.
  - Honeytrap via a verified-but-hijacked/rented account → `behavioral drift from prior pattern (FRAUD-01) + scam trajectory flags takeover; step-up re-verification (liveness)`.
  - Sextortion pattern (intimacy → explicit → threat to expose) → `co-occurrence escalates straight to SAFE-07 with preservation`.
  - False accusation that a normal user is a scammer → `model score + human review; single unusual message doesn't create a label`.
  - Model bias risk (geography/name/accent) → `fairness monitoring; geography-of-scam-infrastructure signals must be device/behavioral, never ethnicity/name-based; audited`.
- **Acceptance criteria:** money-request/off-platform/overseas-emergency patterns detected → nudges + scoring; cross-recipient templated-script reuse detected at network level in fixtures; money+threat co-occurrence auto-routes to SAFE-07 with preservation; outputs fairness-monitored, no protected-attribute proxy features (assertable).
- **Dependencies:** FRAUD-01, SAFE-05/07/08, COMM, SAFE-06, §11 analytics.

### FRAUD-05: Fraud-model feedback, tuning & false-positive governance
- **Description:** The closed loop ingesting moderator decisions, appeals, and user feedback to retrain/tune all fraud + filter models and govern false-positive rates — keeps the aggressive posture accurate and fair.
- **Expected behavior:** every confirmed/dismissed case, overturned appeal (SAFE-11), false-positive "view anyway" (SAFE-05), reinstatement (MOD-01) is a labeled example feeding retraining + rule tuning. Tracks per-model precision/recall, FP-rate, demographic fairness; guardrail thresholds gate model promotion. Allow-pairs/exclusions from appeals (shared-device, look-alike, VPN-legit) persist so the same FP doesn't recur. Human-in-the-loop for any threshold change widening auto-actioning.
- **Edge cases & handling:**
  - Model drift after a new scam wave → `rapid rule hotfix path independent of full retrain; logged`.
  - Adversarial poisoning (scammers gaming feedback) → `feedback weighted by reviewer trust; anomalous label patterns quarantined`.
  - Fairness regression → `model rollback; incident; §11 dashboard alert`.
  - Over-blocking legit VPN users → `FP-rate breach on that segment triggers tuning + exclusion review`.
- **Acceptance criteria:** case/appeal/FP outcomes captured as labeled training data; FP-rate + fairness guardrails gate promotion (breaches block deploy); appeal-derived exclusions demonstrably prevent recurrence of the same false positive.
- **Dependencies:** all FRAUD-*, SAFE-02/05/11, MOD-01, §11 analytics/fairness.

---

## Identity-linked bans & ban evasion

### SAFE-06: Identity-linked bans that survive re-registration
- **Description:** Bans keyed to durable identity signals (verified gov-ID hash, biometric/liveness embedding, device+behavioral cluster) so a banned user can't return with a new email/phone/profile — the verification moat as safety weapon.
- **Expected behavior:** on ban (MOD-01), record ban tokens: salted hash of gov-ID number/document, face/liveness embedding reference (in vault), device/hardware fingerprints, behavioral cluster ID — NOT raw PII. On any new registration/verification, candidate identity signals matched against the ban token store; a match blocks completion + links the attempt to the banned cluster (evidence). Silent to the evader where feasible ("we could not verify your eligibility at this time" rather than "you are banned," to avoid coaching evasion) — while genuine false positives get a human-review path (SAFE-11). Cluster propagation: banning one confirmed node offers propagation to strongly-linked nodes.
- **Edge cases & handling:**
  - Evasion via new email/phone only → `blocked by ID-hash/biometric match`.
  - Evasion via new/forged gov document → `biometric (face embedding) match to the banned identity catches doc-swap; forged-doc detection (VER) + face-reuse (FRAUD-03) corroborate`.
  - Evasion via new device + new doc + new face (fully fresh identity) → `hardest; caught only if behavioral/network or new offenses re-surface; residual risk accepted, relies on continuous FRAUD scoring`.
  - False positive (recycled phone number, shared family device, ID data-entry error) → `verification blocked but immediate human-review/appeal; ID-hash match requires a strong signal (biometric) to hard-block, not device/IP alone`.
  - Recycled phone number reassigned → `phone is a weak token; never sole basis`.
  - Ban collides with a sanctioned/minor case → `routed to the hard legal lane`.
  - Right-to-erasure by a banned user → `profile PII erased but minimal ban tokens (hashes/embedding reference) retained under documented lawful basis to prevent re-entry; user informed of the retention basis`.
  - Cross-border ban evasion → `single global ban-token index (hashes/pointers only; PII stays region-resident) enforces worldwide`.
  - Biometric embedding format change → `versioned embeddings + re-match on upgrade; ban survives model upgrades`.
  - Twin/look-alike triggers biometric match → `biometric alone insufficient for hard identity-ban; requires ID-hash or corroboration; human review`.
- **Acceptance criteria:** re-registration with the same gov-ID hash or liveness/face identity blocked at verification + linked to the banned cluster; ban tokens store only hashes/references (never raw PII; raw biometrics stay in the encrypted vault); weak tokens (phone/IP/device) never solely hard-block, strong tokens (ID-hash/biometric) do; erasure doesn't delete the minimal ban tokens (documented exception, user informed); ban propagates across regions via a PII-free global index.
- **Dependencies:** VER, MOD-01, FRAUD-01/03, SAFE-07/11, PRIVC-01/03/04/05, SAFE-10.

---

## Safety escalation, nudges & education

### SAFE-07: Safety-escalation lane (threats/extortion, evidence preservation, LE-ready)
- **Description:** A dedicated high-urgency lane for credible threats, extortion, sextortion, imminent harm, minor-safety, with legal-grade evidence preservation and law-enforcement-ready export ("safety-escalation path for threats/extortion," "evidence preservation, law-enforcement-ready").
- **Expected behavior:** entry auto (FRAUD-04 money+threat, SAFE-05 co-signals, SAFE-01 threat/minor/extortion reasons) or manual (MOD-01/SAFE-02). **P0 SLA:** ack ≤15 min, human safety-specialist action ≤1 hr, 24/7; supervisor + safety-team mandatory. **Evidence preservation:** on lane entry, a legal-hold snapshot — full server-side message thread with timestamps + metadata, media, both parties' profile states, device/IP/behavioral (FRAUD-01), identity signals — written immutably (SAFE-10) + exempt from deletion/erasure (PRIVC-04) for the hold; chain-of-custody recorded. **LE-ready export:** signed, scoped, chain-of-custody export for lawful requests; export audited; disclosure gated by legal review + LE-request policy (validity/jurisdiction/scope minimization). **Victim support:** affected member offered guidance (report to authorities, preserve evidence, block, resources), regionally appropriate hotlines; never pressured; kept informed within privacy limits. Immediate protective action on the perpetrator (suspend/ban) without waiting for full investigation when harm is credible.
- **Edge cases & handling:**
  - Active/imminent threat to life → `fastest path; may involve proactive outreach and, per policy + legal basis, emergency disclosure to authorities where lawful; documented`.
  - Extortion with a deadline ("pay or I leak photos in 24h") → `immediate preservation + perpetrator suspension + victim guidance; do NOT delete the victim's account/photos even if they panic-request erasure — offer soft-hide instead and explain the preservation need`.
  - Sextortion with intimate images → `preserve minimally + securely in vault; restrict access to safety specialists; if a minor is depicted, mandatory legal/CSAM workflow (jurisdiction-specific), hard lane, handled per legal obligations and reported to the appropriate authority per region`.
  - Minor detected → `immediate lockdown, no adult contact, legal review; age-gate failure path; identity-linked ban if fraudulent adult misrepresentation`.
  - Perpetrator tries to delete evidence (unsend/delete account) → `server-side immutable snapshot already taken; unsend/delete doesn't remove preserved evidence; account deletion → soft-delete under legal hold`.
  - False extortion claim to weaponize the lane → `specialist review of preserved evidence; unsupported claims dismissed; possible action against false reporter; innocent target's preserved data released from hold and handled per retention`.
  - Cross-border → `residency governs storage; LE export honors requesting jurisdiction's lawful process + residency constraints; legal review mandatory`.
  - Mandatory-reporting jurisdiction → `comply; grievance officer (DPDP) / DPO (GDPR) looped where required`.
  - Threat in filtered content the victim never viewed → `preserved regardless; specialist reviews`.
- **Acceptance criteria:** P0 acknowledged ≤15 min + actioned by a human specialist ≤1 hr, 24/7 (measured); lane entry creates an immutable chain-of-custody legal-hold snapshot exempt from erasure for the hold; LE export produces a signed, scoped, audited package gated by legal review; perpetrator unsend/delete/erasure cannot remove preserved evidence; minor/CSAM cases follow the jurisdiction-specific mandatory workflow.
- **Dependencies:** SAFE-01/02/04/05/10, FRAUD-01/04, MOD-01, SAFE-06, PRIVC-03/04/05, VER (age), legal/grievance-officer + DPO (PRD §9).

### SAFE-08: In-context scam nudges
- **Description:** Just-in-time in-conversation safety prompts triggered by scam-pattern signals, warning the potential victim at the exact risky moment ("anti-romance-scam education prompts," "in-context scam nudges").
- **Expected behavior:** triggers (FRAUD-04/SAFE-05): money/gift-card/crypto/bank-detail mention → "Never send money to someone you haven't met. This is the most common scam here."; early push off-platform → "Moving off-platform removes our safety protections. Verified ≠ vouched — take your time."; overseas-emergency/stranded/medical narrative → "'Overseas emergency' stories are a classic scam script. Be cautious." Shown to the *recipient/potential victim*, non-blocking, with Report / Block / Learn more (SAFE-09) / Dismiss. Severity-gated, deduped per thread, escalating with signal strength. Does not tell the sender they were flagged. Nudge events logged (feed FRAUD-05 + case context) but educational, not an accusation.
- **Edge cases & handling:**
  - Legit money/travel discussion → `advisory, non-blocking, dismissible; no penalty`.
  - Alert fatigue in a chatty legit thread → `per-thread dedupe + cooldown; don't repeat the same nudge`.
  - Sophisticated scammer coaches victim to ignore warnings → `nudge persists; combined with FRAUD escalation + possible proactive T&S outreach on high-confidence scams`.
  - Recipient dismisses then gets scammed → `nudge event preserved; supports post-incident review + victim support; no blame`.
  - Localization → `per §7.6; culturally appropriate; tested per language`.
  - Sender is the victim's real long-distance partner → `advisory only; shown privately to recipient, framed as general safety`.
  - Move-off-platform after genuine mutual consent + contact reveal → `nudge softens but still reminds once of off-platform protection loss`.
- **Acceptance criteria:** money-mention/off-platform-push/overseas-emergency each show the correct localized nudge to the recipient; nudges non-blocking, dismissible, deduped per thread, never reveal flagging to the sender; nudge events logged for FRAUD-05 + case context.
- **Dependencies:** FRAUD-04, SAFE-05/09/01/03, §7.6 i18n, FRAUD-05.

### SAFE-09: "Verified ≠ vouched" safety education
- **Description:** Persistent, well-placed education that a verified badge attests identity/credential, NOT character/intent/safety ("'verified ≠ vouched' messaging to counter false security") — countering the "verified elite enclave breeds false security" risk.
- **Expected behavior:** badge tooltips/provenance UI state exactly what each badge attests and, explicitly, what it does NOT ("This confirms identity and a degree from X. It does not vouch for character or intent."). First-run onboarding safety module; contextual reminders at high-trust moments (first contact-reveal, first off-platform push via SAFE-08, before a first meeting). Safety resource center: red-flag literacy, scam scripts, meeting safely, how verification works + its limits, how to report/block, our two-threat-model protections. Women-led editorial hook for red-flag/scam literacy.
- **Edge cases & handling:**
  - User over-trusts a badge, ignores nudges → `repeated escalating in-context reinforcement; can't be permanently dismissed at the badge-provenance level`.
  - Marketing tension (education vs "we're safe") → `resolved by honesty-as-differentiator posture; education is a trust asset`.
  - Localization → `all education localized (§7.6)`.
  - Accessibility → `WCAG 2.2 AA; not image-only`.
- **Acceptance criteria:** every verified badge exposes what it attests AND what it does not; first-run safety module presented before full platform access (completion logged); safety resource center reachable from report, nudge, and profile surfaces.
- **Dependencies:** VER (badge provenance), PROF, SAFE-08, §7.6 i18n, §8 accessibility.

---

## Privacy & compliance

### PRIVC-01: Consent records (DPDP unbundled/unconditional, GDPR lawful basis, withdrawal)
- **Description:** Granular, versioned consent + lawful-basis ledger meeting DPDP (express, unbundled, unconditional, not service-conditioned) and GDPR (lawful basis per purpose, easy withdrawal) — compliance-as-differentiator.
- **Expected behavior:** each distinct processing purpose (identity verification, credential verification, biometric/liveness, income verification, matching/personalization, device/behavioral fraud-prevention, marketing/notifications per channel, family-collaborator sharing, photo-media processing, analytics/experimentation, cross-border transfer) has its own consent or documented lawful basis. **DPDP:** consents unbundled (separately toggleable, no one master accept), unconditional (core service not withheld for declining non-essential processing), express affirmative with plain-language notice in supported Indian languages; withdrawal as easy as granting. **GDPR:** lawful basis per purpose (consent/contract/legitimate-interest with LIA/legal-obligation); withdrawal for consent purposes; DPIA references for high-risk (biometrics, profiling). **Consent ledger:** immutable, versioned (purpose, basis, policy version, timestamp, method, region, every change/withdrawal). Re-consent on material policy changes; granular preference center.
- **Edge cases & handling:**
  - Declines device/behavioral fraud processing → `fraud-prevention may rely on legitimate-interest/legal-obligation basis where lawful + documented; core safety not fully waivable but disclosed + minimized; the consent-vs-legitimate-interest distinction recorded`.
  - Declines biometric/liveness consent → `can't complete verification requiring it; offered alternatives where available; NOT bounced from unrelated processing (unbundled)`.
  - Withdrawn mid-use → `PRIVC-02 handling`.
  - Service-conditioned bundling attempted by a product change → `blocked by policy gate; DPDP unconditional rule enforced (non-essential consents can't gate core service)`.
  - Minor → `no valid consent; blocked (SAFE-07 minor path); minors excluded from a marriage platform`.
  - Cross-border transfer consent/mechanism → `recorded with the mechanism relied on (adequacy/SCCs/DPDP-permitted-country); tied to residency (PRIVC-05)`.
  - Family-collaborator access → `explicit, scoped, revocable consent per collaborator; withdrawal cuts access immediately`.
  - Proof-of-consent dispute/regulator request → `ledger produces the versioned record with the policy text version shown at capture`.
- **Acceptance criteria:** every purpose has an independent consent/lawful basis (no master bundled accept for DPDP-scope consents); declining a non-essential purpose never blocks core service (unconditional test); withdrawal available per-purpose + as easy as granting (each grant/withdrawal an immutable event); consent records capture policy version/method/timestamp/region and are producible for audit.
- **Dependencies:** VER, FRAUD-01 (legitimate-interest), PRIVC-02/04/05, Notification (channel consents), PROF (family-collaborator), SAFE-10, PRD §9 grievance officer/DPO.

### PRIVC-02: Consent withdrawal & mid-use handling
- **Description:** Runtime behavior when a user withdraws a consent while actively using the service — lawful, immediate, non-destructive cessation per purpose.
- **Expected behavior:** withdrawal takes effect prospectively + promptly per purpose; preference center reflects state; a ledger event is written. Per-purpose consequences explained before confirmation ("withdrawing verification consent will hide your profile / remove a badge"; "withdrawing marketing stops digests only"). Withdrawing a purpose doesn't cascade to unrelated purposes (unbundled). Data processed lawfully before withdrawal remains lawful; retention then per that purpose's schedule (PRIVC-04).
- **Edge cases & handling:**
  - Withdraws verification consent mid-conversation → `badge removed / discoverability suspended going forward; existing matches informed only that the badge is gone (no PII leak); not retroactively "un-verified" in audit`.
  - Withdraws biometric consent after ban tokens derived → `future biometric processing stops; existing ban-token reference retained under fraud/legal exception, disclosed`.
  - Withdraws fraud-processing "consent" but basis was legitimate-interest → `informed it continues under legitimate-interest for safety, with objection-handling per GDPR Art. 21 where applicable; documented`.
  - Withdraws family-collaborator consent → `collaborator access revoked immediately; sessions invalidated`.
  - Withdraws all consents but keeps account → `account may become non-functional for core purposes; offered erasure/soft-delete as the coherent path; not silently deleted`.
  - Withdrawal during an active safety case → `does not impede evidence preservation under legal basis (SAFE-07/PRIVC-04); user informed of the retention basis`.
  - Race mid-transaction (mid-verification job) → `job completes or aborts cleanly; final state reflects withdrawal; no partial limbo`.
- **Acceptance criteria:** withdrawal stops the specific purpose's processing promptly + writes a ledger event; consequences disclosed pre-confirmation; no unrelated purpose affected; legal/fraud-retention exceptions survive withdrawal + disclosed.
- **Dependencies:** PRIVC-01/04/05, VER, PROF, SAFE-06/07, Notification, SAFE-10.

### PRIVC-04: Right-to-erasure / soft-delete + hard-purge (with fraud/legal retention)
- **Description:** User-initiated erasure with a two-stage soft-delete → hard-purge lifecycle, reconciled against mandatory fraud-record + legal-hold retention ("user-initiated erasure; transparent retention schedule").
- **Expected behavior:** **Soft-delete (immediate):** profile removed from discovery/search/matches; media hidden; contactability off; account frozen; recoverable within a grace window (14–30 days, region-configurable). **Hard-purge (after grace):** irreversible deletion/anonymization from primary stores, vault (crypto-shred), backups (on rotation cycle, documented max lag), indices, caches, derived data (embeddings, model features) — EXCEPT retention exceptions. **Retention exceptions (documented, minimized):** identity-linked ban tokens (SAFE-06), safety/fraud evidence + legal holds (SAFE-07), immutable audit (SAFE-10, pseudonymous), legally-mandated financial records — each minimized, referenced by hash/pseudonym. Erasure audited; user gets confirmation + a statement of what (if anything) is retained + why. Published per-category retention schedule.
- **Edge cases & handling:**
  - Erasure by a banned user → `PII erased/anonymized; ban tokens retained under fraud-prevention basis; user informed the ban persists + why`.
  - Erasure by a user with an active safety case/legal hold → `hard-purge deferred for held data until hold lifts; non-held PII still erased; user informed of the specific hold`.
  - Cross-border erasure → `honored across all residency regions; each region purges; global index pointers removed; residency governs execution`.
  - Backups → `hard-purge propagates on the next rotation within a documented max window; restores re-run a purge-replay`.
  - Counterparties' chat history → `the erasing user's messages anonymized/tombstoned in the other party's thread ("[deleted]") except where preserved under a safety hold; the other user's own data untouched`.
  - Vault biometric/document data → `field-level keys crypto-shredded, guaranteeing irrecoverability even in latent copies`.
  - Erasure to escape active extortion → `protect the victim: offer soft-hide/lockdown instead of destroying their own evidence; explain; full erasure only after the case resolves or per the victim's informed choice within legal limits`.
  - Minor data → `expedited deletion except where law requires reporting/retention (CSAM per SAFE-07)`.
  - Sanctioned-person data → `retained/handled per sanctions obligations (PRIVC-06), which can override erasure`.
  - Re-registration after erasure → `allowed for a non-banned user; SAFE-06 tokens still block a banned person despite prior erasure`.
- **Acceptance criteria:** soft-delete immediate + recoverable within grace; hard-purge after the window irreversible across primary/vault/index/cache/derived/backups (within documented lag); retention exceptions enumerated, minimized to hashes/pseudonyms, documented against a legal basis, disclosed; erasure executes across all residency regions + fully audited; crypto-shredding renders vault data irrecoverable; a published per-category retention schedule exists.
- **Dependencies:** PRIVC-01/03/05/06, SAFE-06/07/10, COMM (counterparty threads), VER, backups/infra, PRD §9.

### PRIVC-03: Isolated PII/document vault (field-level encryption, access-brokered)
- **Description:** A physically/logically isolated store for the most sensitive data (gov IDs, documents, marksheets, income proofs, liveness/biometric embeddings, quarantined content), separate from the profile DB, with field-level encryption + brokered access ("sensitive documents in an isolated, access-brokered store — not co-located with the profile DB") — answers the "concentrated sensitive-PII collection with no security posture" risk.
- **Expected behavior:** vault holds sensitive artifacts keyed by opaque token; the profile DB stores only tokens + minimal derived attestations ("ID verified: true, method, date"), never the raw document. **Field-level encryption:** per-field/per-subject data keys (envelope encryption via KMS/HSM); keys separated from ciphertext; per-subject keys enable crypto-shredding. **Access broker:** no direct vault reads; every access via a broker enforcing RBAC, purpose-binding (cite a case/verification task), JIT time-boxed grants, and full audit (SAFE-10) of who accessed which field for what reason. Bulk export disabled by default; break-glass requires approval + heightened audit. Region-scoped (PRIVC-05). Data minimization: documents retained only as long as needed for verification, then purged/attestation-only.
- **Edge cases & handling:**
  - Vault breach containment → `blast radius limited: field-level per-subject encryption means a ciphertext exfil without KMS keys is not plaintext; keys in a separate KMS/HSM; playbook: revoke/rotate keys, invalidate broker grants, forensics via access audit, regulator/user breach notification per DPDP/GDPR timelines`.
  - Insider/rogue moderator tries to browse IDs → `broker denies non-purpose-bound access; every attempt logged; anomalous patterns alarmed; no bulk read`.
  - Verifier needs to view a document → `JIT time-boxed grant scoped to that case; auto-expires; watermarked view where possible; download restricted`.
  - Vault backup → `encrypted with the same field/subject key regime; crypto-shred via key destruction applies to backups`.
  - Biometric embeddings for FRAUD-03/SAFE-06 → `accessed by the matching service through the broker with a service purpose, never exposed to human moderators as raw biometrics`.
  - Legal-hold data (SAFE-07) → `flagged non-purgeable for the hold; access still brokered + audited`.
  - Key-management failure/lost key → `per-subject key loss = that subject's data unrecoverable (acceptable failure favoring confidentiality); HA/replication of KMS; documented DR`.
  - Cross-region access for a global safety case → `brokered under the legal transfer mechanism; logged; minimized`.
- **Acceptance criteria:** no service/human reads sensitive artifacts except through the broker with RBAC + purpose-binding + JIT + audit; profile DB contains no raw IDs/documents/biometrics (only tokens + attestations); field-level per-subject encryption with keys in a separate KMS/HSM; crypto-shred proven to render data irrecoverable; every vault access audited; bulk export disabled by default; breach playbook limits plaintext exposure without KMS keys + triggers notification workflows.
- **Dependencies:** VER, FRAUD-03, SAFE-05/06/07, PRIVC-04/05, BO RBAC, KMS/HSM, SAFE-10, PRD §8.

### PRIVC-05: Data-residency routing per region
- **Description:** Region-aware routing/storage so each user's data resides + is processed in the correct regulatory region, with lawful cross-border transfer only where permitted ("data-residency routing per regulatory region," "lawful cross-border transfer mechanisms").
- **Expected behavior:** on registration/verification, the governing region is determined (residence/citizenship/regulatory nexus); PII, documents (vault), consent records, audit routed to that region's stores. Cross-border access/transfer (global safety case, diaspora matching) uses documented lawful mechanisms (GDPR adequacy/SCCs; DPDP transfer rules) + minimized + audited. Global indices hold only non-PII pointers/hashes (ban-token index SAFE-06, case pointers) — never clear-text PII — so worldwide enforcement works without unlawful movement. Cross-region matching exchanges the minimum necessary via brokered, consented transfer.
- **Edge cases & handling:**
  - Diaspora user with multiple nexuses (Indian citizen resident in UK) → `governing region by policy (residence + strictest-applicable-law); documented; user informed; stricter regime's protections applied where they overlap`.
  - User relocates region → `migration workflow moves/repoints data under a lawful basis; old region purged post-migration; consent re-confirmed if basis changes`.
  - Cross-region match interest → `minimal data surfaced cross-border under consent; full contact reveal still mutual-consent gated (COMM)`.
  - Cross-border erasure → `executes in every region holding data`.
  - Cross-border safety escalation → `evidence stays region-resident; LE export honors requesting jurisdiction + residency + legal review`.
  - Data-localization-mandate region → `hard-pinned storage; no transfer out except by that region's permitted mechanism; enforced at routing layer`.
  - Sanctioned/embargoed region → `access/processing gated by PRIVC-06 before any storage`.
  - Latency/availability tradeoff → `regional stacks/partitions must not fall back to copying PII to a non-compliant region under load; degrade gracefully`.
  - Global ban vs residency → `PII-free global token index (SAFE-06) reconciles worldwide enforcement with residency`.
- **Acceptance criteria:** a user's PII/documents/consent/audit stored in the governing region (verifiable per-record region tagging); cross-border transfers only via a documented lawful mechanism + minimized + audited; global indices contain no clear-text PII; relocation + cross-border erasure/escalation respect residency + audited; localization-mandate regions hard-pinned with no non-compliant fallback.
- **Dependencies:** PRIVC-01/03/04/06, SAFE-06/07/10, COMM, VER, §7.6 i18n, infra/routing, PRD §14.

### PRIVC-06: Minor & sanctioned-data handling
- **Description:** Hard controls for two override categories: minors (never permitted; strict exclusion) and sanctioned/embargoed/PEP-flagged persons (screening + legal handling) — from "age/consent gating" + fraud posture.
- **Expected behavior:** **Age-gating:** verification establishes age; under-minimum refused; suspected minors already in-platform locked down (SAFE-07 minor path) + data handled per law (expedited deletion except mandated retention/reporting). **Sanctions/PEP screening:** identity verification screens against applicable sanctions/embargo + (where required) PEP lists; matches route to compliance review, not generic fraud ban. Neither handled by ordinary moderation alone; both loop compliance/legal (grievance officer/DPO).
- **Edge cases & handling:**
  - Minor using a fake adult ID → `doc-forgery + liveness/age-estimation mismatch flags; hard lockdown; identity-linked block; if an adult facilitated, escalate (SAFE-07)`.
  - CSAM discovered → `mandatory jurisdiction-specific reporting; images handled per legal obligation, not retained beyond law; hardest lane`.
  - Sanctioned individual attempts registration → `blocked/held per obligation; data handling may be legally required to be retained/reported, overriding erasure`.
  - False sanctions match (common-name collision) → `human compliance review before action; not auto-banned; resolved with disambiguating data`.
  - Sanctions status changes over time → `periodic re-screening per policy`.
  - Region without PEP requirement → `screening scope configured per residency; no over-collection where not required`.
- **Acceptance criteria:** minors refused/locked-down + never discoverable/contactable; CSAM follows the mandatory legal workflow; sanctions/PEP screening runs at verification per region + matches route to compliance review (not auto-ban); legal-mandated retention/reporting overrides erasure where applicable + documented.
- **Dependencies:** VER, SAFE-06/07, PRIVC-01/04/05, legal/compliance (grievance officer/DPO), PRD §9.
