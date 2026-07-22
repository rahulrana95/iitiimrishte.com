# Feature Spec — Discovery, Search, Matching & Freshness

> Domain 3 of the [Feature Specification](../FEATURE-SPEC.md). Companion: [PRD](../PRD.md) §7.3, [ERD](../ERD.md) §2/§4, [liquidity brief](../gtm/raw/02-liquidity-gender-balance.md).

**Cross-cutting invariants (apply to every feature below):**
- **Discoverability gate:** a PROFILE appears in any surface only if `identity_verified = true` AND it holds ≥1 eligibility-signal badge AND `account_status = active`. Pending/suspended/banned/deactivated/erased never appear.
- **Corridor scoping:** discovery is corridor-first (home-metro + diaspora cluster + relocation overlap); unreachable-geography profiles never pad results.
- **Reachability primacy:** `last_active_at` + verified-badge presence + response-rate jointly gate/rank every surface — closes "shortlists don't reply."
- **Block/consent hygiene:** any BLOCK pair is mutually excluded from all surfaces, view logging, and alerts.
- **Non-enumerable identity:** clients only ever see `public_handle` (ULID), never internal id.

---

### SRCH-01: Faceted candidate search
- **Description:** Structured filter search over the verified pool (institute, employer, profession, seniority, income band, location, age, height, community/religion, languages, relocation willingness). Closes the incumbent's "filter-only, low-signal, unresponsive results" gap by making facets the *entry* to discovery while layering reachability (SRCH results always reachability-ranked) and compatibility on top — never bare filtering.
- **Expected behavior:**
  - Facet inputs map to ERD columns; institute/employer canonicalized via INSTITUTION/EMPLOYER (incl. `aliases[]`); age derived from `dob` (DOB never exposed).
  - Happy path: verified searcher submits ≥0 facets → discoverability gate + corridor scope + block hygiene → tier-permitted facets applied → returns a reachability-ranked, deduped page, each card carrying an "active recently" signal + compatibility chip.
  - No facets = browse of own corridor, still reachability-ranked + freshness-ordered.
  - Multi-value facets OR-within / AND-across; range facets inclusive bounds.
  - Ad-hoc facets are hard filters for this query only; they do NOT mutate saved PARTNER_PREFERENCE. Saved `dealbreakers_json` always applied even if not in the query.
  - Sort: default reachability-then-compatibility; re-sort by recently-active / newest / best-compatibility / relocation-fit. Never exposes sub-day `last_active_at`.
  - Stable keyset pagination on composite rank + `public_handle`; seen profiles shown but badged "seen."
- **Edge cases & handling:**
  - Empty result set → `return zero + "broaden your search" naming the most-constraining facet; never silently drop a facet`.
  - Thin corridor (<~5 reachable) → `return what exists + honest "thin market" notice offering widen age/relocation, adjacent-corridor opt-in, or notify-me; NEVER inject unreachable-geography profiles`.
  - All matching already seen → `still return (search is exhaustive) but rank unseen first, badge rest "seen"; surface "you've seen everyone — try daily picks or widen"`.
  - Facet on an unverified attribute → `profile returned but value renders "unverified"; "verified attributes only" toggle excludes them`.
  - Dormant candidate matches → `excluded from default ranking or sunk to collapsed "inactive — may not reply" section`.
  - Blocked user matches → `fully excluded both directions, no count leak`.
  - Incognito candidate matches → `incognito affects who-viewed-me only, NOT searchability; returned normally`.
  - Tier-locked facet → `SRCH-02 governs: locked with upsell, query runs on permitted facets`.
  - Income filter where candidate `income_band = null` → `excluded; searcher warned income filter hides non-disclosers`.
  - Community/religion filter → `permitted but never defaulted-on, never required (anti-caste-gating)`.
  - Institution alias ambiguity → `disambiguation chips, never guess`.
  - Candidate edits out of match mid-session → `card remains until refresh, then drops; no error`.
  - Own profile → always excluded.
  - Malformed input (age_min>age_max) → `normalize or field-level reject; never unbounded scan`.
- **Acceptance criteria:**
  - Every returned profile passes the discoverability gate.
  - Blocked pairs never appear either direction.
  - Dealbreakers enforced even when not queried; ad-hoc facets don't persist.
  - Empty/thin states return guidance, never a padded page.
  - Tier-locked facet visibly locked, not silently applied.
  - p95 facet query < 300 ms using ERD composite/GIN indexes.
- **Dependencies:** SRCH-02, MATCH-01, MATCH-02, FRESH-01, ERD PROFILE/EDUCATION/EMPLOYMENT/INSTITUTION/EMPLOYER/PARTNER_PREFERENCE/BLOCK, verification.

---

### SRCH-02: Tier-gated search filters
- **Description:** Facet availability and result depth governed by the searcher's plan (`PLAN.feature_flags`), monetizing depth without recreating the incumbent's "hard paywall behind a dead pool." Free users get real verified reachable results (never a wall of dormant profiles); premium unlocks advanced facets and volume.
- **Expected behavior:**
  - Free: core facets (age, location/corridor, profession, institute), reachability-ranked, capped depth, compatibility chip visible but "why" teased. Daily picks (DISC-01) fully available — anti-flood surface never paywalled.
  - Premium: all facets incl. income/seniority/height/community/languages/relocation, full pagination, saved searches, "verified attributes only," full "why this match."
  - Concierge: premium facets + RM-operated search on member's behalf.
  - Locked facets render visibly locked with upsell; query still executes on permitted facets. Flags read at query time so plan changes apply immediately.
- **Edge cases & handling:**
  - Downgrade mid-cycle → `saved searches preserved but locked facets stripped + upgrade banner; nothing deleted`.
  - `paused` subscription (outcome-guarantee pause) → `retain last tier's discovery entitlements during pause — never degrade discovery for a user we paused`.
  - `expired`/`cancelled` → `revert to Free at period end, not mid-period; grandfather in-flight session`.
  - Free hits depth cap → `honest "N more verified reachable matches — upgrade," real count`.
  - Concierge RM on-behalf → `role-scoped + AUDIT_LOG; RM sees discovery data only, never evidence vault`.
  - Feature-flag lookup fails → `fail closed to Free facets, never open to premium; log`.
- **Acceptance criteria:** locked facets never silently apply nor block permitted-facet results; daily picks available on Free; plan change reflects within one query cycle; paused subs retain entitlements; every on-behalf search writes AUDIT_LOG.
- **Dependencies:** SRCH-01, DISC-01, MATCH-01, ERD PLAN/PLAN_PRICE/SUBSCRIPTION/AUDIT_LOG.

---

### MATCH-01: Compatibility scoring & explainable "why this match"
- **Description:** Algorithmic compatibility layered on facets across values, intent/timeline, lifestyle, and education/career parity, with a human-readable "why this match." Closes the incumbent's "filter-only, not algorithmic" gap; explainability is a trust feature.
- **Expected behavior:**
  - Dimensions/sources: Values (`values`/`bio` + `partner_intent`; community/religion only if BOTH opted in); Intent/timeline (`partner_intent`, `intent_timeline` alignment; mismatch = strong negative not hard block unless dealbreaker); Lifestyle (structured fields); Education/career parity (seniority, INSTITUTION.type, profession, income proximity — *parity not hierarchy*).
  - Score: deterministic weighted 0–100 → coarse public bands ("Strong/Good/Fair fit") so exact numbers aren't gamed; weights per-corridor configurable.
  - Bidirectional: reflects mutual plausibility (feeds DISC-01 reciprocity).
  - "Why this match": 2–4 concrete reasons grounded in *visible* matched fields only (respects target `visibility_json` — never leaks hidden fields).
  - Hard vs soft: dealbreakers applied before scoring; soft prefs shape score. V2: optional vector embedding.
- **Edge cases & handling:**
  - Sparse target → `score on available signal + "limited info" flag; never fabricate a reason`.
  - Field hidden by visibility_json → `excluded from viewer-visible scoring inputs and "why"; may be used internally only if it doesn't leak (income → generic "comparable career stage")`.
  - Intent mismatch → `heavy penalty + optional "why not" honesty; not auto-hidden unless dealbreaker`.
  - Stale score after edit → `recompute on read / dirty-flag invalidation; returning viewer never sees a reason citing a removed attribute`.
  - Target loses/expires a badge → `parity/verification inputs recompute; "why" can't cite an expired credential as verified`.
  - Score ties → `break by reachability then freshness`.
  - Zero overlapping visible signal → `minimal "intent + corridor" reason, else suppress the chip (no empty chip)`.
  - Copied/duplicate bios (gaming) → `values weighted down for low-entropy text; flag T&S if mass-duplicated`.
  - Cross-corridor via relocation → `"why" must state the relocation basis; never present as locally reachable`.
- **Acceptance criteria:** every surfaced candidate has a score + ≥1 grounded reason (or explicit suppressed state); no reason references a hidden/expired field; dealbreaker-violating profiles never scored/shown; edits invalidate dependent scores before next read; only public band exposed; deterministic/reproducible.
- **Dependencies:** SRCH-01, MATCH-02, DISC-01, ERD PROFILE/PARTNER_PREFERENCE/EDUCATION/EMPLOYMENT/INSTITUTION/VERIFICATION_BADGE.

---

### MATCH-02: Reachability-aware ranking
- **Description:** Ranking that prioritizes active, responsive, verified profiles and excludes-or-labels dormant/unreachable ones — the direct structural answer to "shortlists don't reply" and "harder on men." The domain's most load-bearing anti-incumbent feature.
- **Expected behavior:**
  - Inputs: recency (`last_active_at`), verification depth, responsiveness (historical reply/accept rate from INTEREST+CONVERSATION — chronic non-repliers sink), optional per-receiver inbound-load.
  - Applied to SRCH-01, DISC-01, FRESH-01.
  - Activity tiers: Active → top + "active recently"; Semi-active → below, no negative label; Dormant → excluded from default OR collapsed "inactive — may not reply"; Unreachable → hard-excluded.
  - Reply-parity gap is a live input; low-inbound profiles get a visibility floor (fairness).
  - Inbound-cap interaction: profile at cap (~10–15 pending) stops appearing in new discovery until queue drains / interests expire (~7 days).
- **Edge cases & handling:**
  - New profile no history → `neutral/optimistic responsiveness prior + "new here"; decays to actual behavior`.
  - Active chronic non-replier → `recency overridden downward; ranks below a responsive less-recent peer`.
  - All corridor dormant → `least-dormant labeled + notify-me + relocation-widen; never fabricate reachability`.
  - Direct handle lookup of dormant → `not discovery: return with "inactive" label, don't hide`.
  - At inbound cap → `removed from new discovery; existing convos unaffected; auto re-enters on drain/expiry`.
  - Reactivation → `lifts out of dormant on next pass; may feature in FRESH "back and active"`.
  - Sparse responsiveness for a cohort → `fall back to recency + verification; don't zero out a cohort`.
  - Suspended mid-session → `drops on refresh; card shows "no longer available"`.
  - Timezone skew → `last_active_at UTC-normalized; absolute thresholds, not local-midnight`.
  - Fake-active pings → `last_active_at reflects meaningful sessions; responsiveness dominates so fake-active non-repliers still sink`.
- **Acceptance criteria:** default discovery has no dormant/unreachable unless in a labeled inactive section; responsive semi-recent outranks recently-active non-replier; new profiles not buried; at-cap profiles disappear then return; every card exposes activity signal; reply-parity queryable as live input.
- **Dependencies:** SRCH-01, DISC-01, FRESH-01, MATCH-01, INTEREST/CONVERSATION, ERD PROFILE.last_active_at, Communication domain (inbound caps).

---

### DISC-01: Curated daily picks (reciprocity-weighted, anti-flood)
- **Description:** A small, mutually-plausible, reciprocity-weighted set delivered daily as the PRIMARY discovery surface — the deliberate anti-flood mechanism that caps inbound at the source and prevents the incumbent's uncapped-auction collapse ("5 curated reply-worthy people, never 400 pending").
- **Expected behavior:**
  - Set size: small fixed handful (3–7, corridor-configurable), refreshed daily per viewer timezone.
  - Selection: passes gate + corridor + block hygiene; reachable/active only (dormant never picked); high compatibility; **reciprocity-weighted** — shown only if the viewer is plausibly also a good match for them AND candidate has inbound capacity.
  - Dedupe: already-seen (PROFILE_VIEW) / already-actioned excluded; net-new where supply allows.
  - Available on ALL tiers incl. Free.
  - A day's unactioned picks expire at next refresh (optional grace); never accumulate a backlog.
- **Edge cases & handling:**
  - Fewer qualified than target → `return smaller honest set ("today's 2 — quality over volume"); NEVER pad with lower-compat/dormant/unreachable`.
  - Zero qualified today → `empty-state "no new curated picks — we only show reply-worthy matches" + notify-me + widen; do NOT recycle seen profiles`.
  - All seen → `"you're all caught up," offer revisit-saved or facet search; no re-showing seen faces`.
  - Reciprocity conflict (high for viewer, candidate at cap/low reciprocal) → `excluded — protects over-subscribed side`.
  - One side dormant → `not picked (must be reachable both ways)`.
  - Viewer over own outbound cap → `picks shown but interest disabled ("used today's interests"); saving still allowed`.
  - Block after generation → `removed from live set immediately`.
  - Candidate edits/goes dormant post-generation → `re-validated on render; dropped and backfilled only with equally-qualified reachable candidates`.
  - Refresh crosses timezone/DST → `anchored to viewer-local morning; no double-issue/skip`.
  - Long absence → `deliver one current day's set, not accumulated days`.
  - Corridor male-admission freeze (rebalance) → `picks continue; demand-side may shrink to protect supply; never padded`.
- **Acceptance criteria:** set never exceeds max nor pads below-threshold/dormant/unreachable; every pick reachable + bidirectionally compatible + has inbound capacity; seen/actioned never appear; available on Free; empty/thin never recycle seen faces; refreshes once per viewer-local day; absence doesn't accumulate; blocked pairs never co-appear.
- **Dependencies:** MATCH-01, MATCH-02, FRESH-01, DISC-03, Communication (caps/INTEREST), SRCH-02, ERD PROFILE_VIEW/INTEREST/BLOCK, LOCALE/timezone.

---

### DISC-02: Who-viewed-me (incognito-suppressed)
- **Description:** Shows who viewed a member's profile, powering re-engagement — with incognito viewers suppressed. Balances engagement against the privacy promise the incumbent lacks.
- **Expected behavior:**
  - Non-incognito view writes PROFILE_VIEW; incognito views not recorded to who-viewed-me (may go to identity-free internal analytics).
  - Viewed member sees reverse-chron list (respecting each viewer's visibility_json), deduped to latest per viewer with count/last-viewed.
  - Tier: full list may be premium; Free sees blurred/count-only teaser — count MUST exclude incognito so it can't be reverse-engineered.
  - Blocked pairs never appear.
- **Edge cases & handling:**
  - Toggle incognito ON after viewing → `historical non-incognito views remain (public when made); future views suppressed. View-time state authoritative`.
  - Toggle OFF → `only non-incognito views ever shown; prior incognito views hidden permanently`.
  - Viewer incognito → `invisible + count must not increment (no "someone viewed" leak)`.
  - Later block → `prior view hidden both parties`.
  - Viewer erased/deactivated → `entry disappears (erasure job); no orphan identity leak`.
  - Self-view → never recorded.
  - Repeated views → collapse to one entry + count + latest ts.
  - Via family-collaborator session → `attribute to owning account per consent; don't expose collaborator identity`.
  - Viewed member themselves incognito → `still receives their own who-viewed-me (incognito hides your browsing, not blinds you)`.
  - Timezone → rendered in viewed member's locale.
  - Free teaser vs premium reveal → `both from identical incognito-excluded set`.
- **Acceptance criteria:** no incognito viewer ever appears or increments count; view-time state authoritative; blocked/erased/deactivated excluded; repeated views dedupe; teaser and reveal from same set.
- **Dependencies:** ERD PROFILE_VIEW, PROFILE.incognito, visibility_json, BLOCK, FAMILY_COLLABORATOR, erasure job, SRCH-02, LOCALE/timezone.

---

### DISC-03: Shortlist / save
- **Description:** Private saved shortlist so promising matches aren't lost — reachability-aware so it doesn't become a graveyard of non-repliers. Never visible to the saved party.
- **Expected behavior:**
  - Save/un-save any discoverable profile from any surface; private to owner (+ consented family-collaborator). Distinct from INTEREST (an outbound signal).
  - Entries show live status: reachability label, change since save ("now inactive"/"back and active"/"no longer available"), interest/mutual status.
  - Sort/filter by saved date, compatibility, activity; reachable above dormant.
  - Saving doesn't notify target, doesn't consume outbound interest cap.
- **Edge cases & handling:**
  - Saved goes dormant → `retained + "may not reply" label + optional return-alert`.
  - Saved deactivates/erased → `"no longer available," non-actionable; on erasure degrade to tombstone/remove`.
  - Saved blocks owner → `removed/hidden immediately`.
  - Saved edits out a factor → `compatibility recomputes (stale-guard)`.
  - Duplicate save → idempotent single entry.
  - Save while target incognito → `unaffected; target never told`.
  - Tier size cap → `Free may cap with upsell; at cap prompt upgrade/remove, never silently drop oldest`.
  - Interest sent to shortlisted → `entry reflects sent/mutual; mutual links to conversation`.
  - Family-collaborator saves → `attributed to owner, AUDIT_LOG, within scope`.
- **Acceptance criteria:** never visible to saved party; no notification/cap consumption; dormant/unavailable/blocked/erased labeled or removed; labels reflect live state not snapshot; saves idempotent; tier caps prompt not drop.
- **Dependencies:** MATCH-01, MATCH-02, FRESH-01, INTEREST/CONVERSATION, BLOCK, FAMILY_COLLABORATOR, erasure job, SRCH-02.

---

### FRESH-01: Freshness — new-since-last-visit, seen-dedupe, new-match alerts
- **Description:** Shows returning members new candidates first, de-duplicates already-seen profiles, and fires new-match alerts — the answer to "no new-profiles surface" and "endless recycling of the same faces."
- **Expected behavior:**
  - Per-viewer watermark (per surface). "New since last visit" = `created_at` (new join) or `last_active_at` (reactivated) crossed the watermark AND unseen AND passes gate/corridor/reachability/compatibility.
  - Seen-dedupe: any PROFILE_VIEW by this viewer excludes from "new" and daily picks; in exhaustive search shown but badged "seen." Per-viewer, not global.
  - Ordering: unseen/new first (reachability + compatibility), then previously-seen (only where surfaces show seen).
  - New-match alerts: newly qualifying reachable high-compat candidate (or mutual interest) → NOTIFICATION honoring toggles + quiet hours; never leaks identity to non-consented parties.
  - Watermark advances on visit; "new" badge clears once seen.
- **Edge cases & handling:**
  - First-ever visit → `entire eligible reachable pool is "new"; establish watermark`.
  - Long absence → `cap badge-count ("50+ new") + paginate; batch/digest not an alert dump; daily picks still one current set`.
  - Nothing new → `honest "no new matches since last visit — we'll alert you" + notify toggle; NEVER recycle seen faces`.
  - Reactivated dormant (old created_at, new last_active_at) → `counts as "new/back" but labeled "back and active," not "new member"`.
  - Seen-then-edited → `stays deduped as seen (viewing not content sets watermark); doesn't re-surface as "new" from edits (anti-spam); significant reactivation lists under "active again"`.
  - Watermark reset / new device → `watermark server-side per account survives device change; seen-dedupe independent of watermark (via PROFILE_VIEW) so reset re-shows "new" framing but NOT already-seen faces`.
  - Seen on one surface, appears on another → `PROFILE_VIEW dedupe is cross-surface`.
  - New candidate dormant → `excluded (freshness ∩ reachability)`.
  - New candidate blocks viewer → excluded.
  - Incognito viewer → `their views still dedupe for THEM (PROFILE_VIEW written for viewer's own dedupe even while suppressed from who-viewed-me)`.
  - Alert storm / sudden influx → `rate-limit + digest; respect quiet hours + toggles; rebalance/admission events don't spam`.
  - Mutual-match alert then one side inactive before viewing → `alert valid at fire time; on open show current reachability label`.
  - Cross-corridor "new" via relocation → `only if reachable + relocation-plausible; labeled with basis, never local`.
- **Acceptance criteria:** returning viewer sees unseen/new before seen; already-seen never "new" nor recycled; seen-dedupe per-viewer, cross-surface, survives device change; reactivated labeled "back/active again"; alerts honor toggles + quiet hours + no identity leak; freshness surfaces have no dormant/unreachable/blocked; long-absence returns give current set + capped/batched list, not a dump; incognito viewers still get own seen-dedupe.
- **Dependencies:** ERD PROFILE (created_at, last_active_at), PROFILE_VIEW, NOTIFICATION, MATCH-01/02, DISC-01, DISC-02, BLOCK, LOCALE/timezone, per-viewer watermark store.
