# Product Requirements Document — iitiimrishte.com

**A verified premium-professional matchmaking platform. International. Trust-first.**

**Version:** 0.1 (Draft) · **Date:** 2026-07-22 · **Status:** For review
**Evidence base:** [`docs/research/market-research.md`](./research/market-research.md) (12-agent teardown of iitiimshaadi.com + international competitive scan). Market claims below are cited to that dossier's original sources.

---

## 1. Overview & Vision

**iitiimrishte.com** is a matchmaking platform for **verified premium professionals worldwide** seeking a serious relationship or marriage. It takes the one thing iitiimshaadi.com genuinely got right — **credential verification as a trust product** — and fixes almost everything else: it broadens the pool from "3–4 elite Indian institutes" to **all verified high-achievers globally** (top-tier alumni *and* accomplished professionals — founders, doctors, researchers, senior operators), it makes verification about *identity and intent*, not just a degree scan, and it engineers for **liquidity, response-rate fairness, and outcome-linked value** instead of a slow app behind a hard paywall.

**Vision statement:** *The most trusted place in the world for accomplished people to meet someone equally serious — verified, private, and international by design.*

**Why now:**
- The incumbent's own numbers show a huge gap between "5 lakh members" claimed and ~30,000 active/authenticated ([dossier §7](./research/market-research.md#7-company--scale)) — the market is under-served, not saturated.
- Verified-premium is a proven, ownable global segment: EliteSingles (~85% degree-holders), The League, Raya, Dil Mil (2M+ diaspora), EliteMatrimony all validate demand and premium pricing ([dossier §9](./research/market-research.md#9-international--premium-professional-expansion-signals)).
- Diaspora-first premium is genuine whitespace — Dil Mil proved demand but skews casual dating; EliteMatrimony proved elite intent but skews India-resident HNI.

## 2. Problem Statement

The incumbent, iitiimshaadi.com, is disliked for concrete, repeatable reasons — each a design input for us:

1. **Liquidity failure.** "The database is quite small, and many members are on free accounts, making meaningful interaction difficult" — shortlisted people never reply ([App Store reviews](https://apps.apple.com/in/app/iitiimshaadi-matchmaking-app/id1626456060)).
2. **Gender-imbalance response collapse.** ~60/40 male-skew means paying men get near-zero responses; Quartz: "harder on men than women" ([Quartz](https://qz.com/india/2148968/elitist-iitiimshaadi-com-is-harder-on-men-than-women)).
3. **Weak value.** ~₹26,000–₹33,000 for a small, unresponsive pool; the canonical review: paid 4 months, "nothing, just null and void" ([Quora](https://www.quora.com/Is-IITIIMShaadi-com-legit)).
4. **Spoofable verification.** Manual marksheet review verifies a degree, not identity/marital-status/intent; no KYC backend ([PTC News](https://www.ptcnews.tv/iit-iim-shaadi-dot-com-bizarre-matrimonial-site-screening-degrees-id-cards-and-what-not)).
5. **Stale, slow product.** "Extremely slow" app, no way to find new profiles ([chrome-stats](https://chrome-stats.com/d/com.tycho.iitiimshadi/reviews)).
6. **Brand liability.** Elitism/caste/gender backlash + a non-alumnus founder undercut the trust it sells ([Homegrown](https://homegrown.co.in/homegrown-explore/iit-iim-shaadi-is-emblematic-of-the-systemic-elitism-in-indian-matrimony); [Zee News](https://zeenews.india.com/technology/ye-sab-dogalapan-hai-twitterati-trolls-iit-iim-shaadi-founder-for-not-passing-out-from-iit-or-iim-2449824.html)).

**Core problem:** accomplished people want a *trustworthy, high-signal, responsive* pool of equally serious peers — and no incumbent delivers verification, liquidity, fairness, privacy, and international reach at once.

## 3. Goals & Non-Goals

**Goals**
- G1 — **Trust:** make multi-signal verification (identity + credential + intent) the defining, visible product experience.
- G2 — **Liquidity:** guarantee a *reachable, active* pool; never let a paying user face a wall of dormant profiles.
- G3 — **Response fairness:** engineer against the gender/interest-imbalance collapse so both sides get meaningful engagement.
- G4 — **International from day one:** multi-country, multi-currency, multi-language, diaspora + local.
- G5 — **Outcome-linked value:** pricing tied to genuine access and results, with transparent refunds.
- G6 — **Privacy & safety by design:** GDPR + India DPDP compliant, consent-first, with strong anti-fraud and moderation.

**Non-Goals (for now)**
- NG1 — Not a casual/hookup dating app; intent is serious relationship/marriage.
- NG2 — Not open-registration mass matrimony; verification gate is deliberate.
- NG3 — Not caste- or institute-supremacist positioning; "premium" = verified achievement, not pedigree hierarchy.
- NG4 — No horoscope-only matching as a core mechanic (offered as optional preference, not identity).

## 4. Target Users & Personas

**P1 — Aarav, 31, India-metro professional (Bangalore).** IIT+IIM, senior PM at a tech firm. Burned by iitiimshaadi's non-responses. Wants a *responsive* pool of verified peers and proof his matches are real and serious. Pain: paying and getting "null and void."

**P2 — Priya, 29, NRI/diaspora (London).** Consultant, Indian-origin, wants someone who shares professional ambition *and* cultural context, open to India or global. Pain: existing diaspora apps skew casual; family wants verification and privacy ([NRI expectations](https://blog.nrimb.com/tips-for-finding-nri-matches-for-arranged-marriage-through-matrimony-services-in-2026/)).

**P3 — Daniel, 35, international non-Indian premium professional (Singapore).** Physician, no alumni-network tie to India, wants a serious, vetted, global matchmaking service à la EliteSingles/The League but with real intent and better verification ([EliteSingles](https://www.top10.com/dating/reviews/elitesingles)).

**P4 — Meera, 27, elite-alumni traditionalist (Delhi + family).** Values verified education/family background and privacy; her parents are co-decision-makers. Wants incognito browsing, controlled photo visibility, and optional assisted matchmaking — features the incumbent lacks or hides.

**Secondary:** parents/family collaborators (view/assist a profile with consent), and a concierge/relationship-manager operating the assisted tier.

## 5. Positioning & Differentiation

**Positioning:** *Verified premium professionals, worldwide.* The moat is **how rigorously we verify** (identity + credential + income + intent), not which college someone attended. Achievement and verification — not caste, not a 3-institute list — define eligibility, which simultaneously **grows liquidity** and **defuses the elitism brand risk** that dogs the incumbent.

### Comparison vs iitiimshaadi.com

| Dimension | iitiimshaadi.com | iitiimrishte.com |
|---|---|---|
| Eligibility | 3–4 elite Indian institutes | Verified premium achievement (top alumni **or** accomplished professionals), global |
| Verification | Manual marksheet/ID scan (spoofable), no KYC | Multi-signal: gov-ID + liveness + credential (registrar/employer/LinkedIn) + optional income; per-badge |
| Liquidity | Thin, ~20% active, shortlists don't reply | Active-pool guarantees, reachability SLAs, honest active metrics |
| Response fairness | Men get 1–2 interests; "harder on men" | Balanced discovery, reply-nudges, quality-gated outreach, curated intros |
| Matching | Filter-only, not algorithmic | Facet search **+** compatibility scoring **+** optional curation |
| Freshness | No "new profiles" surface | New-since-last-visit, dedupe seen, daily curated picks |
| Privacy | Enumerable profile IDs, opaque data policy | Non-enumerable IDs, incognito, photo-blur/consent-gated media, published data policy |
| Pricing | ₹26k–33k flat, opaque refunds | Tiered + regional, outcome-linked options, transparent refunds |
| Compliance | No public posture | GDPR + DPDP, consent-first, data-residency |
| International | India + implicit NRI | Multi-country/currency/language, diaspora + local, day one |
| Brand | Elitism/nepotism backlash | Trust, privacy, achievement — inclusive of many paths to "premium" |

**Enumerable-ID note:** the incumbent's `Base64(sequential-int)` profile URLs ([dossier §2](./research/market-research.md#2-product--profile-anatomy)) are a scraping/privacy defect we explicitly avoid (opaque UUID/ULID public handles).

## 6. Market Opportunity & Sizing (qualitative)

- **Under-served core:** incumbent's active base is ~30k despite years of operation and celebrity marketing ([dossier §7](./research/market-research.md#7-company--scale)) — demand exists but is unmet on quality.
- **Adjacent proven demand:** EliteSingles (85% degree-holders), Raya (2.5M waitlist vs six-figure base), Dil Mil (2M+ diaspora) show a large, premium-priced, verification-seeking audience globally ([dossier §9](./research/market-research.md#9-international--premium-professional-expansion-signals)).
- **Pricing headroom:** self-serve ~$20–50/mo plus concierge/success-fee ($300+/mo or one-time) is established and tolerated in this segment.
- **Whitespace:** verified, marriage-intent, *international* premium-professional platform spanning diaspora + local — no incumbent owns it.

---

## 7. Functional Requirements

Each capability notes the incumbent gap it closes. Format: **User story → key requirements → acceptance criteria (AC).**

> **Developer-ready detail:** this section is the summary. Every feature below is specified in full — name, description, expected behavior in all states, exhaustive edge cases, acceptance criteria, and dependencies — in the [**Feature Specification**](./FEATURE-SPEC.md) (`docs/spec/`). Feature IDs (e.g. `VER-2`, `BAL-01`, `SAFE-07`) are stable references used across the spec.

### 7.1 Onboarding & Multi-Tier Identity Verification *(closes: spoofable verification, fake/dormant profiles, gender-asymmetric onboarding)*

**Story:** As a new user, I complete a graduated verification so that everyone I see is provably real, eligible, and serious.

- Verification **signals** (each independently badged): (a) **Government-ID + liveness/selfie match** (identity); (b) **Credential** — degree via registrar/institution-email/document + review, and/or **employer/LinkedIn** professional verification; (c) **Income/net-worth** (optional, document or aggregator-based); (d) **Marital-status attestation** with escalating checks; (e) **Photo authenticity** (liveness-linked, anti–stock/steal).
- **Eligibility engine:** admits on *either* verified elite-alumni credential *or* verified premium-professional achievement (title/seniority/employer/income thresholds), configurable per market. No single hard institute list.
- **Symmetric standards across genders** — identical verification requirements regardless of gender to close the documented asymmetry seam.
- KYC/identity checks meet or exceed the Indian govt advisory for matrimony sites ([India TV](https://www.indiatvnews.com/news/india-user-verification-with-valid-ids-must-on-matrimonial-sites-govt-advisory-332427)) and support international ID types.

**AC:** no profile is discoverable until ≥ identity + one eligibility signal are verified; each badge shows its basis and verified date; verification method is source-authenticated (not bare image upload) wherever a registrar/employer/ID API exists; a re-verification cadence flags stale attestations.

### 7.2 Profiles *(closes: enumerable IDs, missing privacy controls, thin fields)*

**Story:** As a member, I present a rich, verified profile while controlling exactly who sees what.

- Public handle is a **non-enumerable opaque ID** (UUID/ULID); no sequential/base64 integers.
- Fields: identity basics, verified education/employment, profession/seniority, income band (optional), location + willing-to-relocate, languages, community/religion (optional, never required), family details (optional), lifestyle/values, partner intent & timeline, media.
- **Privacy controls:** incognito/hidden browsing, photo-blur / consent-gated media (reveal on mutual interest), granular per-field visibility, block/mute, and "family/parent collaborator" access by explicit consent.
- Verified badges render on the profile with provenance.

**AC:** a member can hide photos until mutual interest; blocked users cannot view or contact; no enumeration of profiles via ID guessing; every sensitive field has an independent visibility setting.

### 7.3 Search, Filtering & Matching *(closes: filter-only matching, weak/low-signal results, no freshness)*

**Story:** As a member, I get a steady flow of relevant, *contactable* candidates — not the same faces on repeat.

- **Facet search:** institute/employer/profession/seniority/income/location/age/height/community/languages/relocation.
- **Compatibility scoring** layered on facets (values, intent/timeline, lifestyle, education/career parity) — algorithmic, not filter-only.
- **Reachability-aware ranking:** prioritize *active, responsive, verified* profiles; surface an explicit "active recently" signal.
- **Freshness:** "new since your last visit," dedupe already-seen, daily curated picks, new-match alerts.

**AC:** default discovery excludes dormant/unreachable profiles or clearly labels them; a returning user sees new candidates first; compatibility score is explainable ("why this match").

### 7.4 Communication *(closes: paywalled dead-ends, response-rate collapse)*

**Story:** As a member, I can express interest and converse safely, with a fair shot at a reply.

- Interest/like → mutual-interest chat; contact-reveal gated on mutual consent (not just payment).
- **Response-fairness mechanics:** outreach quality gating (limit low-effort mass-likes), reply nudges, "your interest expires" prompts, and balanced daily inbound caps so high-demand profiles don't drown and low-inbound profiles still get seen.
- In-chat safety: report/block inline, anti-harassment filters, no contact-info leakage before consent.

**AC:** a member cannot be contacted by unverified users; both parties must consent before contact details reveal; abusive senders are rate-limited/suspended.

### 7.5 Trust & Safety *(closes: no fraud controls beyond doc scan, honeytrap seam, false security)*

- Fraud detection: device/behavioral signals, duplicate-photo/stock-photo detection, scam-pattern models, velocity checks.
- Reporting & moderation: user reports → triage queue → moderator actions (warn/suspend/ban) with audit trail; safety-escalation path for threats/extortion.
- Anti-romance-scam education prompts; "verified ≠ vouched" messaging to counter false security ([Homegrown](https://homegrown.co.in/homegrown-explore/iit-iim-shaadi-is-emblematic-of-the-systemic-elitism-in-indian-matrimony)).

**AC:** every report is actioned within an SLA; repeat offenders are blocked across re-registration (identity-linked); moderation actions are logged immutably.

### 7.6 Internationalization & Localization *(closes: India-only + implicit NRI)*

- Multi-country onboarding (ID types, credential sources per country), multi-currency pricing, multi-language UI, timezone-aware notifications, locale-aware matching (diaspora ↔ home country).
- Data-residency routing per regulatory region.

**AC:** a user in the UK, UAE, Singapore, US, or India completes verification with locally valid documents, pays in local currency, and uses the UI in a supported language.

### 7.7 Monetization *(closes: opaque/flat pricing, refund opacity, unproven concierge)*

- **Tiers:** Free (verified browse + limited interests), **Premium** (self-serve, full messaging/discovery, ~$20–50/mo-equivalent, regional), **Concierge/Assisted** (human relationship-manager, transparent scope, outcome milestones).
- **Outcome-linked options:** e.g., pause/extend if no verified reachable matches; success-fee variant for concierge.
- **Transparent refunds** and clear (non-dark-pattern) renewal — directly answering the incumbent's opaque refund/renewal complaints ([dossier §3](./research/market-research.md#3-pricing--monetization)).

**AC:** pricing, renewal terms, and refund policy are shown before purchase; concierge scope and milestones are contractual; no forced auto-renew without explicit consent.

### 7.8 Notifications & Engagement

- New verified matches, mutual interests, messages, reply-nudges, weekly curated digest; per-channel (push/email/SMS/WhatsApp where compliant) and per-type controls; quiet hours by timezone.

**AC:** every notification type is independently toggleable; no notification leaks profile identity to non-consented parties.

### 7.9 Admin / Ops / Verification Back-Office

- Verification review console (queue, evidence viewer, source-check integrations, badge issuance), moderation console, concierge CRM, refund/billing ops, analytics dashboards, and content/success-story management.

**AC:** verifiers act only on consented evidence; all back-office access is role-scoped and audit-logged; PII access is least-privilege.

---

## 8. Non-Functional Requirements

- **Performance:** p95 primary interactions < 300 ms server; app cold-start < 2.5 s — an explicit answer to "extremely slow" ([chrome-stats](https://chrome-stats.com/d/com.tycho.iitiimshadi/reviews)). Mobile-first, offline-tolerant.
- **Scalability:** stateless services, horizontally scalable search/matching; designed for millions of profiles and facet+vector queries.
- **Availability:** 99.9% target; graceful degradation of non-critical features.
- **Security:** PII encrypted at rest (field-level for IDs/documents), TLS in transit, secrets management, least-privilege, regular pen-tests, bug-bounty. Sensitive documents in an isolated, access-brokered store — not co-located with the profile DB.
- **Accessibility:** WCAG 2.2 AA.

## 9. Privacy, Compliance & Legal

- **Dual compliance:** **India DPDP Act 2023** (express, unbundled, unconditional consent; no service-conditioned bundling) and **GDPR** (EU/UK) as baseline ([Bar & Bench](https://www.barandbench.com/columns/ghosts-in-the-machine-dpdp-act); [EY](https://www.ey.com/en_in/insights/cybersecurity/decoding-the-digital-personal-data-protection-act-2023); [CookieYes](https://www.cookieyes.com/blog/india-digital-personal-data-protection-act-dpdpa/)).
- **Consent-first:** granular consent records per data use; lawful cross-border transfer mechanisms; data-residency per region.
- **Data minimization & retention:** documents retained only as long as needed for verification; user-initiated **erasure/right-to-be-forgotten**; transparent retention schedule.
- **Age/consent** gating; content policy; grievance-redressal officer (DPDP requirement) and DPO (GDPR).
- **Positioning:** publish the data-safety posture the incumbent never did — turn compliance into a trust differentiator for a privacy-sensitive audience.

## 10. Success Metrics & KPIs

| Metric | Target (steady-state) |
|---|---|
| Verification completion (start→identity+eligibility) | ≥ 60% |
| Active/verified ratio (vs incumbent's ~20%) | ≥ 70% |
| Median first-reply rate on quality outreach | ≥ 35% (both genders within 10 pts of each other — fairness KPI) |
| Match → conversation rate | ≥ 25% |
| Free → paid conversion | 6–10% |
| M1 / M3 retention | ≥ 55% / ≥ 30% |
| Verified reachable matches per active user / month | ≥ 8 |
| Concierge outcome-milestone attainment | ≥ 70% of contracted milestones |
| Marriage/serious-relationship self-reported outcomes | tracked cohort, published honestly |
| Trust & safety: reports actioned within SLA | ≥ 99% |

## 11. Analytics & Experimentation

- Event instrumentation across the funnel (register→verify→discover→interest→chat→pay→outcome).
- A/B framework for verification flows, pricing, discovery ranking, and fairness mechanics.
- Fairness dashboards (response-rate parity by gender/geography) as first-class, monitored metrics.
- Cohort outcome tracking (opt-in) to report real success rates rather than PR-style unaudited counts ([dossier §7](./research/market-research.md#7-company--scale)).

## 12. Risks & Mitigations

| Risk | Evidence | Mitigation |
|---|---|---|
| Cold-start / thin inventory | Incumbent's #1 complaint ([App Store](https://apps.apple.com/in/app/iitiimshaadi-matchmaking-app/id1626456060)) | City/segment-by-segment launch to critical density; invite waitlist (Raya-style scarcity as asset ([Raya](https://www.vidaselect.com/raya-dating-app-review))); reachability guarantees before charging |
| Gender/response imbalance | "Harder on men" ([Quartz](https://qz.com/india/2148968/elitist-iitiimshaadi-com-is-harder-on-men-than-women)) | Balanced acquisition, fairness mechanics (§7.4), curated intros, reply-rate KPIs |
| Verification cost / unit economics | Betterhalf wind-down ([startupintros](https://startupintros.com/orgs/betterhalf)) | Automate source-checks; premium pricing funds manual review; tiered verification depth |
| International regulatory | DPDP/GDPR ([Bar & Bench](https://www.barandbench.com/columns/ghosts-in-the-machine-dpdp-act)) | Compliance-by-design, data-residency, per-region legal review |
| Elitism brand backlash | Incumbent's reputational drag ([Homegrown](https://homegrown.co.in/homegrown-explore/iit-iim-shaadi-is-emblematic-of-the-systemic-elitism-in-indian-matrimony)) | Achievement-based (not caste/institute-supremacist) positioning; inclusive premium definition |
| Fake/verified false-security | ([PTC News](https://www.ptcnews.tv/iit-iim-shaadi-dot-com-bizarre-matrimonial-site-screening-degrees-id-cards-and-what-not)) | Multi-signal verification + fraud models + "verified ≠ vouched" education |
| Sensitive-PII breach | Concentrated doc collection, no incumbent posture | Isolated document vault, field-level encryption, pen-test/bug-bounty |

## 13. Phased Roadmap

**MVP (0–6 mo) — Trust + Liquidity in 1–2 launch markets (e.g., India-metro + one diaspora hub):**
- Multi-signal verification (identity + one credential), non-enumerable profiles, facet search + basic compatibility ranking, interest/mutual-chat, core privacy (incognito, photo-gating, block), Premium tier, DPDP/GDPR consent + erasure, moderation + reporting, freshness surface. Reachability guarantee before charging.

**V1 (6–12 mo) — Fairness + International:**
- Response-fairness mechanics, income/employer verification tiers, multi-currency/multi-language, additional markets (UK/UAE/Singapore/US), concierge/assisted tier with milestones, "who viewed me," curated daily picks, outcome tracking.

**V2 (12–24 mo) — Depth + Scale:**
- Advanced compatibility (vector/ML), events/curated introductions, family-collaborator flows, referral program, success-fee concierge, richer localization, trust/safety ML at scale, outcome-published cohorts.

## 14. Open Questions

1. Exact eligibility thresholds for "premium professional" per market (title/seniority/employer/income) — needs calibration with real supply data.
2. First launch-market pair — which India-metro + which diaspora hub gives the fastest density? (User leaned "broaden internationally"; MVP still needs a beachhead.)
3. Concierge economics — success-fee vs subscription vs hybrid; break-even on manual verification cost.
4. How prominent should community/religion be — offered as optional preference without reproducing caste-gating criticism?
5. Reddit/first-party sentiment gap — the research environment couldn't reach Reddit ([dossier caveat](./research/market-research.md)); a follow-up primary-source pass (Reddit, live site crawl, refund-terms, app review scrape) should confirm load-bearing figures before build commitments.
6. Data-residency architecture — single global store with regional partitions vs fully regional stacks.
