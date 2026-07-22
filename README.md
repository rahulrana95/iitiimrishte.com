# iitiimrishte.com

A **verified premium-professional matchmaking platform** — international, trust-first. Conceived as a broader, better answer to the elite-alumni matrimony niche pioneered (with well-documented flaws) by iitiimshaadi.com.

> **Positioning:** the moat is *how rigorously we verify* (identity + credential + income + intent), not which college someone attended. Eligibility = **verified premium achievement** — top-tier alumni **or** accomplished professionals — across geographies. This grows the pool *and* defuses the elitism brand risk that dogs the incumbent.

## Documents

| Doc | What's in it |
|---|---|
| [`docs/research/market-research.md`](./docs/research/market-research.md) | Link-cited teardown of iitiimshaadi.com (product, pricing, reputation, safety, company/scale, gap analysis) + international competitive scan. Every claim links to its original source. |
| [`docs/PRD.md`](./docs/PRD.md) | Product Requirements — vision, personas, positioning vs incumbent, functional + non-functional requirements, compliance, metrics, risks, phased roadmap, open questions. |
| [`docs/ERD.md`](./docs/ERD.md) | Entity-Relationship model — Mermaid diagram + per-table column specs. Structurally supports verification badges, non-enumerable profiles, multi-currency/i18n, and DPDP/GDPR consent + erasure. |
| [`docs/research/raw/`](./docs/research/raw/) | Raw per-angle research outputs (12 agents), each with its own source list. |

## How this was produced

A fleet of 12 parallel research agents each took one angle on the target site — profile/site anatomy, pricing, Reddit/Quora sentiment, complaint/review aggregators, blogs & press, app-store reviews, company scale, fraud/safety, features/verification, unmet-needs gap analysis, and international premium-professional positioning. Findings were synthesized into the dossier, which grounds the PRD and ERD.

## Evidence caveats

- The target site, app stores, Reddit, Quora, and most review aggregators were **not directly fetchable** from the research environment (HTTP 403 / login gates). Findings rest on **search-indexed snapshots** of real pages, not first-hand loads.
- Company scale and pricing figures are largely **unaudited PR/aggregator claims** — flagged in-doc and to be re-verified against live pages before build commitments.
- **Reddit** returned no data (network-blocked, not absent); TeamBlind + Quora serve as the professional-audience analog.

## Top 5 opportunities (why build this)

1. Fix the **liquidity/inventory** failure (incumbent's #1 complaint: dormant, unreachable profiles).
2. Solve the **gender/response-rate collapse** ("harder on men than women").
3. Upgrade verification from **spoofable document scan** → multi-signal identity + credential + intent.
4. Rebuild the **value equation** (outcome-linked, transparent pricing/refunds).
5. Broaden the moat to **verified premium achievement, internationally** — more liquidity, less brand risk.

_Draft v0.1 — for review._
