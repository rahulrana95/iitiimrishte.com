# Team Role: Two-Sided Liquidity & Gender-Balance Architect (the core solve)

_Raw agent output._

**Thesis:** iitiimshaadi failed on market design, not features. It ran an uncapped auction where supply (women's attention) was free to demand (men's interests): demand flooded, supply got buried under "thousands of interests/week," supply left. Every mechanism below makes attention a *governed, rationed, reciprocal* resource. We build a balanced, throttled two-sided market where the scarce good — a real reply from a serious verified peer — is protected by design.

## 1. Cold-start: seed the constrained side first, in a dense wedge
- Binding constraint = serious verified **women**. Seed supply first; don't open demand floodgate until supply is safe. But "women first" isn't enough — the real unit is a **dense cohort in one narrow wedge** where both sides find someone this week.
- Wedge: one metro+diaspora corridor (e.g., Bangalore/Delhi ↔ London/Bay Area, 27–34). Not "global premium" (density → zero).
- **Phase 0 (wk 0–8):** manufacture supply density — recruit first ~300–500 verified women BY HAND (concierge, alumnae networks, warm intros); free lifetime premium in exchange for activity commitments (respond to N curated intros/week). Founding cohort's job is to *reply* (a seeded profile that ghosts recreates the 20%-active graveyard).
- **Phase 1 (wk 8–16):** invite men in a metered trickle, gender-gated; men enter a **waitlist** not the product (Raya model — waitlist as asset).
- **Phase 2 (wk 16+):** ignite mutuals; the unlock metric is **time-to-first-mutual-match** (< 72 hrs). Expand only after wedge clears the health bar.
- Men-first rebuilds the incumbent; both-at-once spreads spend to low density everywhere.

## 2. Gender-balance engineering — mechanisms
**Target: active-user discovery-pool ratio 1.0–1.2 M:1 F; red line 1.5:1 halts male admission.** Govern the *active/reachable* ratio, not raw signups.
- **(a) Admission valve (master lever):** men admitted from waitlist by daily quota computed from active female supply; `men_admitted = k×net_active_female_growth − ratio_correction` (k~1.2). Women skip waitlist (instant). Male priority earned by verification depth + completeness + intent, not FCFS.
- **(b) Asymmetric waitlists:** female ~zero friction; male waitlist = pressure valve, flexes with ratio, framed as curation ("we admit in balance so you actually get replies").
- **(c) Curated intros (anti-flood):** primary surface is a daily curated few high-compatibility candidates (Hinge "Most Compatible", two-sided, reachability-aware), reciprocity-weighted (a woman shown to a man only if plausibly also interested) → caps inbound at source.
- **(d) Interest-rationing/quality-gating:** hard daily outbound interest cap (3–5/day esp. men); interests require a personalized note/prompt answer (effort tax); merit-based quota expansion (replies earn more, reports/ignores earn fewer).
- **(e) Reply-rate fairness:** reachability-aware ranking with teeth — visibility ∝ responsiveness; chronic non-repliers sink. Response-SLA nudges on supply side. Reply-parity KPI is a live system input feeding ranking + admission.
- **(f) Demand caps on high-inbound profiles:** per-profile inbound cap (~10–15 pending) → profile stops appearing in new discovery until queue worked down (the single most important anti-"thousands of interests" mechanism). Queue, don't dump (ranked, best-first, manageable handful). Interest expiry ~7 days returns capacity.

## 3. Preventing the death spiral
Break it at every stage:
| Stage | Incumbent | Our circuit-breaker |
|---|---|---|
| Men enter faster | 60/40 uncapped | Gender-gated valve, hard stop 1.5:1 |
| Cheap mass-contact | uncapped interests | Outbound cap + effort tax |
| Inboxes overflow | thousands/week | Per-profile inbound cap + queue + expiry |
| Can't triage | women dormant | Ranked queue + nudges + curation |
| Men get no replies | "null and void" | Reply-rate fairness ranking + admit reply-worthy men |
| Churn, ratio worsens | collapse | Automated rebalancing before red line |
**Inversion:** incumbent let *demand* set pace; we let *supply capacity* set pace. Woman opens app to 5 curated reply-worthy people, never 400 pending. Leading indicator: female inbound/active-woman > 5–7/week → tighten admission NOW.

## 4. Local density vs global spread
Matchmaking is a *local* liquidity business wearing an *international* brand. Unit of liquidity = the **corridor** (home-metro + diaspora cluster + relocation-willingness overlap), not country/globe. Launch and measure per corridor; relocation-willingness is a first-class matching input. Never pad discovery with unreachable-geography profiles. Rule: optimize for the smallest geography that still gives a new joiner ~5 reply-worthy candidates this week.

## 5. Daily health KPIs (per corridor, by gender)
| KPI | Target | Alert |
|---|---|---|
| Active ratio M:F | 1.0–1.2:1 | >1.4:1 tighten male admission |
| Reply-rate parity gap | within 8 pts | >12 pts intervene |
| Male first-reply rate | ≥30% | <20% supply/curation problem |
| Female inbound load/wk | 3–7 | >10 throttle demand now |
| Time-to-first-mutual | <72 hrs median | >5 days density too low |
| Active/verified ratio | ≥70% | <55% reachability failing |
| Supply-side (women) D14 retention | ≥60% | <45% overload |
| Mutual matches/active/wk | ≥1 | <0.5 not liquid |
**Wake-me-at-3am metrics:** reply-rate parity gap + female D14 retention.

## 6. Out-of-balance playbook
- **Mode A demand-heavy (common):** slow valve → tighten demand caps → protect/enrich supply → reactivate dormant women → if still red, freeze male admission & pour budget into female supply. Never fix demand-heavy by adding demand.
- **Mode B supply-heavy/thin (rare):** open valve → targeted demand acquisition into that corridor → widen via relocation overlaps → if can't reach density, merge/pause corridor.
- Standing rules: rebalance automatically (closed-loop controllers) before manually; always fix ratio by adjusting the *abundant* side's throughput, never degrade the scarce side; quarantine broken corridors.

**Five commitments that would have saved the incumbent:** (1) seed verified women by hand, wedge-dense, before opening demand; (2) meter male admission to female supply, defend 1.0–1.2:1; (3) cap demand at source + at receiver; (4) rank on responsiveness, enforce reply-parity live; (5) grow corridor-by-corridor to real local density.
