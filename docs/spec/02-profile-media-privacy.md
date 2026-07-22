# Feature Spec — Profile, Media & Privacy Controls

> Domain 2 of the [Feature Specification](../FEATURE-SPEC.md). Companion: [PRD](../PRD.md) §7.2/§9, [research dossier](../research/market-research.md) §2/§5/§6/§8, [female-first brief](../gtm/raw/04-female-first-acquisition-safety.md).

**Framing.** Implements PRD Goal G6 (privacy & safety by design) — "her data is a vault, not a directory." Incumbent defects this domain closes: enumerable `Base64(sequential-int)` profile URLs; absent incognito/photo-blur/photo-password/block; concentrated PII with no data-safety posture; thin low-control profiles treating women as scrapeable inventory. Verification issuance is owned by Domain 1 — this domain **displays and gates on** badges. In-chat block enforcement is owned by Domain 4 — this domain owns block **at the profile level**.

---

## PROF — Profile Creation & Fields

### PROF-01: Structured profile field model
- **Description:** The canonical field set with a rigid taxonomy of *required / optional / never-required / verified-vs-self-declared* classes. Fixes thin, inconsistent incumbent profiles and enforces NG3 by making community/religion strictly optional and never a gate.
- **Expected behavior:**
  - **Identity basics (required):** display first name (or initial), age (derived from DOB; DOB stored not shown), gender, current city/country, height. Legal full name collected for KYC but never a profile field / never rendered publicly.
  - **Verified education / employment (eligibility-relevant):** institution/degree/discipline/year; employer/title/seniority/industry — rendered with verified/unverified state (PROF-08); employment supports masking (PRIV-05).
  - **Profession/seniority (required at band level):** category + seniority band (IC/Senior IC/Manager/Director/VP+/Founder/Partner).
  - **Income band (optional):** coarse localized ranges; never required; independent visibility + optional verification.
  - **Location + relocation willingness:** current location (required); willingness = {no / within-country / international / to-specific-regions[]}.
  - **Languages (optional, multi-select)** with optional proficiency.
  - **Community/religion (OPTIONAL — NEVER REQUIRED):** skippable at every step; no downstream feature may hard-require it; never blocks completion, discovery, or eligibility.
  - **Family details (optional):** collaborator-editable (FAM-01). **Values/lifestyle (optional).**
  - **Partner intent & timeline (required):** intent {marriage / long-term / exploring seriously}; timeline {<1yr / 1–2yr / 2+yr / unsure}.
  - **Bio (required, min length).** **Media** governed separately (MEDIA-*).
  - Draft autosave on every change; profile is `DRAFT` until the minimum publishable set (PROF-03) is met. Each field stores value, `visibility` (PRIV-02), `verification_state` (PROF-08), `source` (self/collaborator/verified), `updated_at`.
- **Edge cases & handling:**
  - Skips community/religion entirely → `publishes normally; one neutral optional prompt max; discovery/eligibility unaffected`.
  - Blank required field → `inline validation blocks publish; no data loss on other fields`.
  - Income band set but visibility "hidden from all" → `stored; used for own-match reciprocity only if opted in; never rendered to others`.
  - Age vs DOB mismatch / age < platform minimum → `profile rejected, flagged to T&S, cannot publish`.
  - Free-text contains contact info → `auto-redacted at render + flagged (pre-consent leakage disallowed); user notified`.
  - Free-text contains slurs/harassment/solicitation → `content-policy filter blocks publish with reason`.
  - Conflicting employer (verified ≠ self-entered) → `verified value wins for display; self-entered archived; user notified`.
  - Height units per locale → `stored canonical cm; rendered per viewer locale`.
  - Paste-bomb bio → `hard max length client + server`.
- **Acceptance criteria:** publishing with community/religion empty succeeds and is fully discoverable; no path marks a profile ineligible/non-discoverable solely for missing community/religion; every field persists independent `visibility` + `verification_state`; legal name never in any profile-read API response; embedded phone/email redacted before serving to another member.
- **Dependencies:** Domain 1 (PROF-08 states), PRIV-02, FAM-01, Domain 4 (contact-leak policy), content-policy service.

### PROF-02: Non-enumerable opaque public handle
- **Description:** Every profile's public identifier is a non-sequential, non-guessable opaque handle — designing out the incumbent's confirmed `/Members/userProfile/{Base64(sequential-int)}` enumeration/scraping defect (`OTMxODA=`→`93180`).
- **Expected behavior:**
  - Public handle = random opaque token (UUIDv4 or **randomized-high-bits ULID** — not time-ordered, to avoid temporal enumeration). Internal numeric PKs never exposed at any surface (URL, API, HTML source, image filenames, WebSocket frames, analytics events).
  - Profile URL: `/p/{opaque_handle}`; no redirect from any numeric/base64 form. Handle→internal-ID resolution server-side only, behind authz.
  - Handle stable for profile lifetime unless the user triggers rotation (PRIV-07 remediation). Media/share/deep-links use the opaque handle or a separate scoped media token (MEDIA-06).
- **Edge cases & handling:**
  - Request `/p/{valid-format-nonexistent}` → `generic 404, identical response/timing to "exists but forbidden" (no existence oracle)`.
  - Rapid sequential requests across handles → `rate-limit + scraping heuristics (PRIV-07); escalating challenge then block`.
  - Leaked share link after blocking recipient → `resolver enforces block at resolution; blocked viewer gets generic not-viewable`.
  - Handle collision → `uniqueness constraint with retry; probability logged`.
  - Legacy/imported sequential IDs → `handles minted fresh & random; original IDs never surfaced`.
  - Handle rotation → `old handle 404s (no redirect confirming continuity); active chats key on internal ID so conversations survive; outstanding share links die by design`.
  - Enumeration via image CDN filenames → `media under random keys unrelated to handle/sequence (MEDIA-06)`.
- **Acceptance criteria:** altering a valid handle doesn't yield another viewable profile at better-than-random rate; no response/header/JS/image URL/event contains the internal numeric ID; 404 for nonexistent is byte-and-timing-indistinguishable from forbidden-but-existing; automated enumeration triggers defenses.
- **Dependencies:** PRIV-07, MEDIA-06, block resolution, API gateway rate-limiting.

### PROF-03: Profile completeness & strength score
- **Description:** A transparent score guiding a discovery-ready profile + a hard "minimum publishable" gate, without penalizing privacy choices.
- **Expected behavior:**
  - **Minimum Publishable Set (MPS) — hard gate:** identity basics + profession/seniority band + ≥1 eligibility signal verified + partner intent & timeline + bio ≥ min + at least one media item *or* explicit "face-photo-optional" acknowledgement (MEDIA-05). Below MPS → `DRAFT`, not discoverable.
  - **Strength score (0–100) — soft/advisory:** weighted from verified fields (weighted higher than self-declared), optional high-signal completeness, media richness, bio quality, activity recency.
  - **Privacy neutrality:** hiding a field (visibility) or declining an optional field must NOT reduce score like leaving it *unknown*. A hidden-but-provided field counts as provided. Declining community/religion never reduces score.
  - Score shown only to owner (+ consented collaborators, FAM-02) with an itemized "what would raise this." Never shown to other members. Verification contributions labeled so users see verification is the lever.
- **Edge cases & handling:**
  - Everything except community/religion → `can reach 100; that field carries zero weight`.
  - Employer masked (PRIV-05) → `counts as provided/verified; masking doesn't reduce strength`.
  - Verification revoked/expired → `score recomputes downward; owner notified with re-verify CTA`.
  - Media all deleted below MPS → `profile auto-reverts to LIMITED (not fully discoverable); existing chats unaffected`.
  - Collaborator fills fields → `counts but flagged provenance; owner must approve before "verified"-adjacent`.
  - Keyword-stuffed bio → `quality heuristics cap bio contribution`.
- **Acceptance criteria:** hiding N provided fields scores identically to showing them; declining community/religion can't prevent 100; below-MPS never returned by discovery; strength absent from every other-member-facing response.
- **Dependencies:** Domain 1, PRIV-02, PRIV-05, MEDIA-05, Discovery (MPS gate), FAM-02.

### PROF-08: Verified-vs-self-declared rendering & badge provenance display
- **Description:** Consistent contract distinguishing verified from self-declared fields, rendering each badge **with provenance** (basis + verified date + method) — "show his badge provenance in her UI," the female-side trust lever. Counters spoofable, undisclosed incumbent verification and pairs every badge with a "verified ≠ vouched" affordance.
- **Expected behavior:**
  - Each verifiable field/badge renders `VERIFIED` (with provenance popover) / `PENDING` / `UNVERIFIED` / `EXPIRED-STALE` / `FAILED` (owner-only, remediation).
  - Provenance popover shows signal type, method class (source-authenticated API vs document-review), verified date, re-verification status — never the underlying document/PII.
  - Persistent tappable "Verified ≠ vouched" note by the badge cluster.
  - Five-signal summary surfaced (identity, credential, employer/professional, income optional, photo authenticity + marital-status attestation) with **marital-status attestation shown prominently for men** (the honeytrap vector women fear most).
  - Self-declared fields visually + semantically labeled unverified; never implied verified.
- **Edge cases & handling:**
  - Verified but hidden → `nothing shows; masked (PRIV-05) → badge against the masked label`.
  - Verification expires mid-session → `next load reflects STALE; no cached green badge`.
  - Employer verified but masked → `popover confirms "employer verified" without disclosing name until reveal`.
  - Self-declared value edited after verification → `badge → PENDING/UNVERIFIED; re-verification required; viewers never see verified badge on unverified new content`.
  - `FAILED` → `owner-only + reason + appeal; others see unverified/absent per visibility; failure never advertised`.
  - Marital-status attestation absent for a male profile → `absence explicit in signal summary, not silently omitted`.
  - Provenance requested by a blocked user → `denied (block precedence)`.
- **Acceptance criteria:** every VERIFIED badge exposes a provenance popover (signal/method/date, no PII); self-declared can never render verified styling; "verified ≠ vouched" present wherever badges show; editing a verified field immediately removes its badge for all viewers; male profiles surface marital-status attestation prominently.
- **Dependencies:** Domain 1 (badge source of truth), PRIV-02, PRIV-05, block.

---

## MEDIA — Photos & Media

### MEDIA-01: Blur-until-mutual photo gating
- **Description:** Photos default blurred/locked to non-mutual viewers, revealing on mutual interest — closing the incumbent gap where photo-blur/photo-password is absent and photos are simply paywalled.
- **Expected behavior:**
  - Per-photo default for non-mutual viewers = `BLURRED` (server-side blur derivative, not client CSS over a full-res asset). Reveal = mutual interest/match (Domain 4), explicit per-viewer grant, or owner-set public.
  - Full-resolution original never sent to a non-authorized viewer (no client-side unblur). On mutual match, authorized photos → `CLEAR` for both. Owner may keep specific photos gated even after match (MEDIA-02). Face-optional members (MEDIA-05) may have no face photo; gating still applies to whatever exists.
- **Edge cases & handling:**
  - Viewer inspects payload for a blurred photo → `only blurred-derivative bytes exist; original key never referenced`.
  - Match then unmatch/block → `photos revert to inaccessible immediately for removed party; no new clear fetch succeeds`.
  - Owner deletes a photo a matched partner could see → `purged from serving; in chat resolves to "photo removed" placeholder (MEDIA-07)`.
  - Photo pending moderation/NSFW → `served to no one (not even blurred) until it passes`.
  - Owner sets public while incognito → `public visibility honored; incognito still suppresses who-viewed footprints (independent)`.
  - Mutual match but photo restricted to "no one" → `stays gated despite match`.
- **Acceptance criteria:** no API returns full-res bytes of a gated photo to a non-authorized viewer; unmatch/block flips previously-clear photos back to inaccessible on next fetch; moderation-failing photo served to no other member.
- **Dependencies:** Domain 4 (mutual-match signal), MEDIA-02/06/07, moderation/NSFW scan, PRIV-01.

### MEDIA-02: Per-photo visibility controls
- **Description:** Independent visibility per media item, not a single global photo toggle.
- **Expected behavior:** per-photo visibility ∈ {public / verified-members-only / on-mutual-match / specific-grants-only / hidden}, default on-mutual-match. Ordering, primary selection, and visibility independent. `hidden` = owner + consented collaborators only. Owner can grant a specific matched viewer access to an otherwise-gated photo (one-way, revocable).
- **Edge cases & handling:**
  - All photos hidden → `MPS met only if face-optional ack set (MEDIA-05), else reverts to LIMITED`.
  - Primary hidden → `prompt to pick visible primary or fall back to verified-silhouette placeholder; discovery card shows placeholder, never a hidden image`.
  - Conflicting settings (photo public but account-level gated) → `most restrictive wins`.
  - Grant then viewer blocked → `grant auto-revoked`.
  - Grant then owner rotates handle / deletes photo → `grant moot; no orphaned access`.
- **Acceptance criteria:** each photo honors its own visibility in all read paths; account cap vs per-photo conflict → more restrictive wins; hidden primary never on a discovery card.
- **Dependencies:** MEDIA-01/05, PRIV-02 (conflict rule), block, Discovery.

### MEDIA-04: Watermarking & screenshot deterrence
- **Description:** Deter capture/redistribution via per-viewer forensic watermarking + screenshot deterrence + capture signaling (Threat-B exposure model). No incumbent equivalent.
- **Expected behavior:** every clear photo served to another member is stamped with a **per-viewer forensic watermark** encoding an opaque viewer+session token (not PII) → trace a leaked image to the fetching account. Mobile: FLAG_SECURE-equivalent to suppress OS screenshots where allowed; iOS screenshot-event detection. On detected screenshot of another's gated media: log, optionally notify owner (configurable), count toward viewer risk. Web: disable right-click/save, tokened/segmented delivery (deterrence, not prevention).
- **Edge cases & handling:**
  - Screenshot detected → `event recorded with viewer token; repeated captures across profiles → risk escalation / T&S review (scrape correlation, PRIV-07)`.
  - OS prevents detection (Android) → `rely on FLAG_SECURE + watermark; no false "they screenshotted" fired`.
  - Photographs screen with a 2nd device → `undetectable, but forensic watermark still traces if it resurfaces on-platform`.
  - Owner disables screenshot notice → `events still logged for T&S`.
  - Watermark must not degrade viewing → `imperceptible/robust, tuned`.
  - Blocked/unmatched viewer → `receives no clear media, nothing to watermark`.
- **Acceptance criteria:** every clear-media stream to a non-owner carries a per-viewer traceable watermark; a leaked on-platform image attributable to the fetching account; screenshot events (where detectable) logged + feed risk; watermark encodes no human-readable PII.
- **Dependencies:** MEDIA-06, PRIV-07, T&S, Notification.

### MEDIA-05: Face-photo-optional (verification-carried trust)
- **Description:** Members may omit a face photo; trust carried by verification, not by exposing their face to strangers — a female-first accommodation absent from incumbent.
- **Expected behavior:** publish with no face photo (or only non-face media) given an explicit "face-photo-optional" acknowledgement. Displays a **verified-silhouette placeholder** with a prominent badge cluster emphasizing identity+liveness verified. Owner may later reveal a face photo on mutual match while hidden in discovery (MEDIA-02). Discovery treats face-optional as first-class (not down-ranked for lacking a public face photo); strength score privacy-neutral.
- **Edge cases & handling:**
  - Zero media → `MPS met via ack; placeholder shown`.
  - Adds a face photo later → `ack can clear; normal gating`.
  - Viewer filters "has photo" → `face-optional-with-gated-face appear as "photo revealed on match"; truly-no-photo appear only if filter permits (respecting owner discoverability, not forcing exposure)`.
  - Verification incomplete + no face photo → `below MPS; stays draft (identity verification still required)`.
  - Faceless-catfish concern → `mitigated: identity+liveness mandatory even when face not published (face captured for liveness, just not published)`.
- **Acceptance criteria:** verified identity + no public face photo can publish and be discovered; not systematically down-ranked; silhouette placeholder always co-displays badges.
- **Dependencies:** Domain 1 (liveness/identity mandatory), MEDIA-02, PROF-03, Discovery.

### MEDIA-06: Secure tokened media delivery & opaque asset keys
- **Description:** Delivery substrate ensuring photos can't be enumerated, hotlinked, or fetched outside authorization — the media analog of PROF-02.
- **Expected behavior:** media stored under random opaque keys unrelated to handle/internal ID/upload order/timestamp. Every fetch requires a short-lived, per-viewer, per-asset signed token minted only after authz (block, visibility, match-state, moderation-pass evaluated at mint). Tokens single-purpose, bounded TTL, session-bound, non-enumerable. Gated vs clear derivative selected server-side; blurred-entitled viewer can never mint a clear token. Verification documents live in a **separate isolated vault** — never in the media path, never reachable via any member-facing token.
- **Edge cases & handling:**
  - Token replay after expiry → denied. Token shared to another user/device → `session binding rejects`.
  - Authz changes (block/unmatch) after mint before fetch → `revalidate at fetch; deny if now unauthorized (short TTL bounds window)`.
  - Direct CDN key guess → `random keys, no listing; requests without valid token 404`.
  - Verification document requested through media API → `hard-denied; documents not in this namespace`.
- **Acceptance criteria:** no media retrievable without a valid unexpired viewer-bound token; asset keys reveal nothing about identity/sequence/time; revoked authz invalidates in-flight access within TTL; verification documents never served through member-facing endpoints.
- **Dependencies:** PROF-02, MEDIA-01/02, block, verification vault, CDN/object store.

### MEDIA-07: Media lifecycle in active conversations
- **Description:** Correct behavior for media state changes (delete, re-gate, unmatch, block) while a conversation is active.
- **Expected behavior:** photos referenced in chat by asset ID + entitlement, not by copying bytes; entitlement re-evaluated at render. Owner deletes a photo the partner had → "photo removed by owner" placeholder; no cached clear byte re-served. Owner re-gates (match→hidden) mid-chat → partner's future loads show gated/placeholder; prior views not recoverable in-app. Chat-*sent* media vs profile media distinct (chat-sent follows message retention + block rules). Unmatch/block mid-chat → all profile media inaccessible to the other party immediately.
- **Edge cases & handling:**
  - Partner has chat open when owner deletes → `live update (or next interaction) swaps to placeholder; no stale clear image beyond session refresh`.
  - Owner deletes account → `media purged; chat references "unavailable"; watermark records retained in isolated audit only, not served`.
  - Revoked collaborator who uploaded a photo → `collaborator loses access; owner retains unless owner deletes`.
  - Deleted photo resurfaces in a report → `forensic watermark traces the leaker`.
- **Acceptance criteria:** deleting media replaces every in-chat/on-profile reference with a placeholder and serves no further clear bytes; re-gating mid-chat takes effect on the partner's next load; block/unmatch during chat immediately revokes profile-media access.
- **Dependencies:** Domain 4 (chat model/retention), MEDIA-01/06, block, FAM-03, T&S audit.

---

## PRIV — Privacy Controls

### PRIV-01: Incognito / hidden browsing (and who-viewed-me interaction)
- **Description:** Browse without a footprint + a "hidden profile" mode. Implements incognito ("default-on option, first-run") and closes the absent-incognito gap.
- **Expected behavior:**
  - **Incognito browsing:** ON → the member's views don't appear in the viewed member's who-viewed-me; no visit footprint.
  - **Reciprocity rule (explicit, default symmetric):** while incognito, the member ALSO does not see incognito viewers in their own who-viewed-me — so incognito can't be used to spy while hiding.
  - **Hidden profile mode:** excluded from discovery/search for people they haven't interacted with; reachable only via existing matches/grants.
  - First-run offers privacy setup **before the member is publicly visible.** Incognito is a per-session/toggle state, defaultable-on.
- **Edge cases & handling:**
  - Incognito viewer sends interest/like → `contacting is an intentional reveal; recipient sees the interest; passive views stay hidden`.
  - Incognito then off, revisit → `only the non-incognito visit registers`.
  - Who-viewed-me is premium + viewer was incognito → `viewer simply absent; no "hidden viewer" teaser to de-anonymize`.
  - Incognito + public photo → `photo visibility independent; being incognito doesn't hide a public photo from a profile they didn't visit`.
  - Hidden profile + active matches → `matched partners retain full access; hidden only affects new discovery`.
  - Telemetry must not leak an incognito visit via any side channel (notification, badge count, "recently active near you").
- **Acceptance criteria:** incognito passive view never appears in who-viewed-me nor triggers any side-channel; symmetric suppression; sending interest while incognito reveals identity for that action only; hidden profile absent from discovery for non-connected members but available to existing matches.
- **Dependencies:** who-viewed-me (Domain 3 DISC-02), Discovery, Domain 4 (interest flow), Notification, PRIV-02.

### PRIV-02: Granular per-field visibility & conflict resolution
- **Description:** Every sensitive field carries independent visibility; conflicts resolve deterministically.
- **Expected behavior:** visibility scope per field ∈ {public / verified-members-only / on-mutual-match / connections-only / hidden}. Account-level presets set defaults fields can further restrict but not loosen beyond an account cap. **Canonical rule: effective visibility = most restrictive of {field-level, account-level cap, block state, verification state}.** Verified-but-hidden = hidden; unverified-and-public renders as unverified. Masking (PRIV-05) is a distinct partial-visibility mode layered on top.
- **Edge cases & handling:**
  - Field public but account cap verified-members-only → `effective = verified-members-only (cap wins)`.
  - Field verified but hidden → `nothing rendered (no badge)`.
  - Field unverified but public → `rendered with explicit unverified label`.
  - Viewer blocked → `block overrides all field visibility`.
  - Income public but unverified → `value + unverified label; owner may prefer masking`.
  - Community/religion hidden → `never shown; never usable as a discovery filter that could out a hidden value`.
  - Collaborator vs owner setting → `owner always wins; collaborator can't loosen`.
- **Acceptance criteria:** served value/badge = most-restrictive of field/account-cap/block/verification; no field shown more permissively than its account cap; hidden field leaks through no filter/sort/side-channel.
- **Dependencies:** PRIV-05, PROF-08, block, FAM-03, Discovery (filter-leak prevention).

### PRIV-05: Employer masking
- **Description:** Partial-visibility mode proving employer *caliber* via verification while withholding the specific name until reveal ("verified at [Big Tech / Top Consulting / Hospital]"). No incumbent equivalent.
- **Expected behavior:** with masking ON, non-authorized viewers see a **verified category label** + "employer verified" badge, without the name. Category derived from a curated employer taxonomy at verification. Reveal = owner-set (on mutual match / grant / manual in chat) → exact name + badge to that viewer. Same pattern optional for institution and income (band shown, source masked). Preserves category-level searchability without name exposure.
- **Edge cases & handling:**
  - Employer not in taxonomy → `generic "Verified employer" rather than mis-categorization; category review requestable`.
  - Masking ON but employer visibility hidden → `hidden wins; not even category shows`.
  - Viewer granted reveal then blocked → `reveal revoked`.
  - Verification expires while masked → `category label drops its verified badge`.
  - De-anonymization risk (unique employer in a small category) → `granularity chosen to avoid singling out; rare-employer members offered a broader category`.
  - Search filters by specific employer → `must not reveal a masked profile matches a specific employer to the searcher (only category-level match surfaces)`.
- **Acceptance criteria:** with masking ON, no non-authorized viewer sees the specific employer in any surface; category label + badge display and reveal the name only after the owner-set condition; specific-employer search can't confirm a masked profile's exact employer.
- **Dependencies:** Domain 1 (employer taxonomy + verified name), PRIV-02, Search (oracle prevention), Domain 4 (in-chat reveal), block.

### PRIV-06: Profile-level block & mute
- **Description:** Block/mute at the profile level (interoperates with Domain 4 in-chat block). Implements "silent block"; closes absent granular per-member blocking.
- **Expected behavior:**
  - **Block (silent, bidirectional invisibility):** blocked user cannot view the blocker's profile/media, find them in discovery/search, or send interest/message, and is **not told**. Blocker no longer sees the blocked user anywhere.
  - **Mute:** stops that user's notifications/feed visibility for the muter without full block.
  - Block persists across sessions and is **identity-linked** (survives re-registration where identity is known). One-tap from any profile surface; optionally paired with a report.
- **Edge cases & handling:**
  - Blocked user opens a bookmarked URL / share link → `generic not-viewable/404 (no "you are blocked" oracle)`.
  - Blocked user searches by name/employer → `blocker never appears`.
  - Shared active conversation → `conversation closes/archives; blocked party sees neutral "conversation unavailable"; profile-media access revoked`.
  - Blocked user new account → `identity-linked block re-applies once verified; pre-verification they can't contact anyway`.
  - Incognito viewer blocked → `block enforced at resolution`.
  - Unblock → `visibility restored going forward; no retroactive block footprint shown`.
  - Who-viewed-me + block → `blocked user's prior view suppressed from the blocker's list`.
  - Blocked user's family collaborator used to circumvent → `cannot circumvent on the blocked user's behalf`.
- **Acceptance criteria:** blocked user receives no explicit block signal anywhere; can't view/find/contact via profile/search/share-link/media token; blocks survive re-registration via identity; block overrides all field/photo visibility and incognito.
- **Dependencies:** Domain 4 (in-chat block interop), Domain 1 (identity linkage), PROF-02 (resolver), MEDIA-06/07, Discovery, who-viewed-me.

### PRIV-07: Scrape / enumeration defense & anomalous-access alerts
- **Description:** Active defenses against bulk harvesting + a member-facing anomalous-access alert. Generalizes the anti-enumeration posture into detection.
- **Expected behavior:** behavioral detection of enumeration/scraping (high-velocity distinct-profile views, sequential-ish patterns, headless/automation fingerprints, abnormal media-token minting, screenshot clustering). Graduated response: soft rate-limit → challenge → temporary throttle → suspension + T&S review. **Anomalous-access alert:** anomalous pattern on a member's profile (single actor viewing repeatedly, repeated screenshots) → notify (configurable) + offer remediations (tighten visibility, handle rotation, block). All profile/media access logged with viewer token for forensic correlation (privacy-scoped).
- **Edge cases & handling:**
  - Legit power-user high volume → `thresholds tuned + human-pattern signals; challenge before block`.
  - Distributed scraping across many accounts → `cross-account correlation + real-ID verification cost; identity-linked suspension`.
  - Anomalous alert could de-anonymize an incognito viewer → `alerts describe pattern ("viewed unusually often") without naming an incognito viewer`.
  - Screenshot clustering where OS blocks detection → `rely on view-velocity + watermark; no fabricated alerts`.
  - Alert fatigue → `rate-limited, severity-tiered`.
- **Acceptance criteria:** bulk enumeration triggers graduated defenses before large-scale harvest; members can be alerted + offered handle rotation/tightened visibility/block; anomalous-access alerts never de-anonymize an incognito viewer.
- **Dependencies:** PROF-02, MEDIA-04/06, API gateway, T&S, Notification, Domain 1 (identity-linked suspension).

---

## FAM — Family / Parent Collaborator Access

### FAM-01: Consented family collaborator invitation & roles
- **Description:** A member invites a parent/family member as a **consented, scoped, revocable** collaborator (persona P4/Meera) — giving family "a safe seat" the incumbent lacks. Must satisfy DPDP unbundled consent.
- **Expected behavior:** member (always the principal) invites via a private, revocable, single-use invite, assigning a role: **Viewer** (owner-permitted subset, no actions) or **Assistant** (viewer + draft/suggest edits + shortlist, changes require owner approval, FAM-03). No role can message on the owner's behalf without per-conversation owner consent (FAM-04). Collaborator must create/verify their own (light-KYC) account. Consent explicit, unbundled, separately recorded. Owner may have multiple collaborators with different scopes.
- **Edge cases & handling:**
  - Invite intercepted/forwarded → `single-use + identity binding on acceptance; mismatched identity voids it`.
  - Collaborator is also a member → `strict separation: no discovery advantage, no access to owner's blocked/hidden data beyond granted scope`.
  - Collaborators must be adults; owner must be platform age.
  - Owner under coercion → `owner retains unilateral revoke (FAM-05) + private "solo mode" hiding sensitive activity`.
  - Collaborator never accepts → `no access; invite expires`.
- **Acceptance criteria:** zero access until accept + identity verify + owner scoped consent recorded; consent records per-collaborator, unbundled, auditable; owner is sole principal; no collaborator role overrides owner settings.
- **Dependencies:** Domain 1 (light-KYC), consent/DPDP, FAM-02/03/04/05, Notification.

### FAM-02: Scoped collaborator visibility
- **Description:** Precisely what a collaborator can/can't see, independently configurable — family involvement never becomes total surveillance.
- **Expected behavior:** owner toggles access per area (basic profile, media, shortlist, incoming interests, active conversations metadata-vs-content, strength score, match suggestions). **Default excludes conversation content** (collaborators see existence/match status only unless owner shares). Collaborators never see: owner's blocked-list details outing third parties, owner's incognito history, other members' gated media the owner isn't entitled to share, or other members' PII the owner couldn't share. Owner can run **solo mode** on specific candidates/conversations, hiding them from all collaborators (coercion safety).
- **Edge cases & handling:**
  - Collaborator granted media view → `sees only what owner is entitled to and chose to share; can't mint clear tokens for third-party gated media`.
  - Solo-marked conversation → `disappears from every collaborator surface immediately, no trace`.
  - Candidate's masked employer → `stays masked to collaborator unless owner has an unlocked reveal to share; collaborators inherit owner entitlements, never more`.
  - Owner reduces scope mid-use → `access recalculates immediately; open views revoked on next load`.
- **Acceptance criteria:** conversation content invisible to collaborators by default; a collaborator's view of any third party capped at the owner's own entitlement; solo-mode items invisible to all collaborators.
- **Dependencies:** FAM-01, PRIV-02, MEDIA-06 (entitlement inheritance), Domain 4.

### FAM-03: Collaborator edit rights with owner approval
- **Description:** How Assistant-role collaborators contribute edits without silently altering a verified, privacy-sensitive profile.
- **Expected behavior:** Assistant edits are **proposals** in a pending queue; owner approves/rejects; nothing publishes without approval. Proposed values tagged `source=collaborator`; edits to a verifiable field drop it to `PENDING`/`UNVERIFIED` until owner + verification re-confirm. Collaborators cannot change privacy/visibility, loosen masking, alter block lists, or change verification data.
- **Edge cases & handling:**
  - Proposes edit to a verified field → `on approval, field re-enters verification; badge removed until re-verified`.
  - Proposes community/religion on a profile that omits it → `still optional; owner may reject; never becomes required`.
  - Owner offline long → `proposals stay pending; profile unchanged`.
  - Two collaborators conflicting edits → `both queued; owner resolves; last-approved wins`.
  - Attempts to change visibility/mask via API → `hard-denied by role check`.
- **Acceptance criteria:** no collaborator edit reaches the live profile without explicit owner approval; collaborators can't modify visibility/masking/blocks/verification; edit to a verified field removes the badge pending re-verification.
- **Dependencies:** FAM-01/02, PROF-08, PRIV-02/05, Domain 1.

### FAM-04: Collaborator communication boundaries
- **Description:** Rules preventing a collaborator from impersonating or acting as the owner in conversations — protecting the *other* member's consent (they matched the owner, not the family).
- **Expected behavior:** by default collaborators **cannot** send messages as the owner. Any assisted messaging requires explicit per-conversation owner consent and is **attributed** ("assisted by family collaborator") to the other party. Contact-reveal, mutual-match acceptance, block/report remain owner-only. Collaborators cannot initiate interests on the owner's behalf unless the owner enabled that scope with per-item confirmation.
- **Edge cases & handling:**
  - Assisted-messaging granted then revoked mid-conversation → `collaborator immediately loses send ability; drafts remain owner-only`.
  - Other member objects to family involvement → `attribution lets them decide; they can block (applies to the owner's profile)`.
  - Collaborator tries to reveal contact → `denied; owner-only`.
- **Acceptance criteria:** a collaborator can't send a message appearing solely from the owner without the attribution label; contact reveal, match acceptance, block/report never executable by a collaborator.
- **Dependencies:** FAM-01/02, Domain 4 (attribution, contact reveal), block/report.

### FAM-05: Revoking collaborator access (incl. mid-conversation) & audit
- **Description:** Immediate, complete, revocable teardown of collaborator access at any time — including while a collaborator is actively viewing or assisting a conversation.
- **Expected behavior:** owner revokes any collaborator instantly. On revoke: all that collaborator's sessions/tokens against this owner invalidated; profile/media/conversation access ends on next request; in-flight media tokens rejected at fetch (short TTL bound). Silent to the other member; does not disrupt the owner's own conversation. Consent record closed with timestamp; collaborator notified neutrally. Collaborator-held drafts/notes revert to owner-only. Full append-only audit (who accessed what, when, proposed/did), available to owner and T&S.
- **Edge cases & handling:**
  - Revoke while collaborator has a conversation open → `their view → "access ended" on next interaction/refresh; owner's chat continues uninterrupted`.
  - Revoke mid-draft → `draft discarded on collaborator side; not sent; owner may retain a copy if it was shared`.
  - Collaborator screenshotted before revoke → `outside app control; watermark traces; owner advised`.
  - Re-invite after revoke → `fresh consent + verification; prior access doesn't silently resume`.
  - Owner deletes account with active collaborators → `all access terminates; audit retained per retention in isolated store`.
  - Coercion → `revoke always available; cannot be blocked/reversed by a collaborator`.
- **Acceptance criteria:** revocation invalidates all of that collaborator's access within the media-token TTL bound, incl. mid-conversation; owner's active conversation not disrupted; append-only audit records every access + proposed action; re-granting requires fresh consent + verification.
- **Dependencies:** FAM-01/02/03/04, MEDIA-04/06 (token invalidation, watermark), consent/DPDP, T&S audit, Notification.
