# Database Schema (Supabase / Postgres) — iitiimrishte.com

Concrete, runnable table schemas for **Supabase** (Postgres 15+). This is the physical realization of the conceptual [ERD](./ERD.md), wired to the behavior in the [Feature Spec](./FEATURE-SPEC.md).

**Version:** 0.1 (Draft) · **Date:** 2026-07-22

## How to use this

- Everything lives in the `public` schema unless noted. Auth is Supabase's `auth.users`; **`public.accounts.id` = `auth.users.id`**, so `auth.uid()` is the current account id throughout RLS.
- The SQL below is grouped in dependency order (extensions → enums → reference tables → core → RLS → storage). Paste the sections in order, or split them into `supabase/migrations/000N_*.sql`.
- **Sensitive documents (gov-ID, marksheets, income proofs, biometrics) are NOT stored in these tables.** They live in a locked Storage bucket + a broker; tables hold only a token + a derived attestation. See [§12](#12-sensitive-data-vault).
- Every table has RLS **enabled**; policies in [§10](#10-row-level-security-rls) are the important ones — cross-profile visibility, blocks, and reachability are enforced through `SECURITY DEFINER` functions / RPCs, not raw client SQL.
- `updated_at` is maintained by the `moddatetime` trigger ([§14](#14-updated_at-triggers)).

---

## 1. Extensions

```sql
create extension if not exists pgcrypto      with schema extensions; -- gen_random_uuid()
create extension if not exists citext        with schema extensions; -- case-insensitive email
create extension if not exists moddatetime   with schema extensions; -- updated_at trigger
create extension if not exists pg_trgm       with schema extensions; -- fuzzy search on names/employers
-- vector (optional, V2 compatibility embeddings): create extension if not exists vector;
```

---

## 2. Enums

```sql
create type account_status        as enum ('pending','active','suspended','banned','deactivated','erased');
create type auth_provider         as enum ('password','google','apple','linkedin');
create type gender                as enum ('woman','man','nonbinary','undisclosed');
create type application_status    as enum ('draft','verifying','queued','admitted','expired','market_interest');
create type eligibility_route     as enum ('alumni_credential','professional_achievement');

create type partner_intent        as enum ('marriage','serious_relationship');
create type intent_timeline       as enum ('within_1yr','1_2yr','2plus_yr','unsure');
create type marital_status        as enum ('never_married','divorced','separated','widowed');
create type seniority_level       as enum ('ic','senior_ic','manager','director','vp_plus','founder','partner');
create type visibility_scope      as enum ('public','verified_members','on_mutual_match','connections','hidden');
create type photo_status          as enum ('pending','approved','rejected');

create type verification_signal   as enum ('gov_id','liveness','education','employer','income','marital_status','photo');
create type verification_method   as enum ('api_source','registrar','institution_email','document_review','aggregator','self_attested');
create type assurance_level       as enum ('source_authenticated','email_domain_verified','document_reviewed','self_attested');
create type verification_status   as enum ('submitted','in_review','approved','rejected','expired');

create type interest_status       as enum ('sent','accepted','declined','expired');
create type plan_code             as enum ('free','premium','elite','concierge');
create type billing_period        as enum ('monthly','quarterly','six_month','annual','lifetime','one_time');
create type subscription_status   as enum ('active','paused','cancelled','expired','refunded');
create type payment_status        as enum ('pending','succeeded','failed','refunded','charged_back');
create type milestone_status      as enum ('pending','met','missed');

create type report_reason         as enum ('harassment','explicit','contact_fishing','money_scam','fake_profile',
                                           'married_misrep','impersonation','threat_extortion','minor','spam','other');
create type report_status         as enum ('submitted','triaged','under_review','action_taken','dismissed','escalated','closed','reopened');
create type moderation_action     as enum ('warn','restrict','suspend','ban','dismiss','escalate','force_reverify','reinstate');

create type consent_purpose       as enum ('verification','biometric','matching','marketing','analytics',
                                           'fraud_prevention','cross_border_transfer','family_sharing');
create type consent_basis         as enum ('consent','contract','legitimate_interest','legal_obligation');
create type collaborator_role     as enum ('viewer','assistant');
create type notification_category as enum ('engagement','transactional','safety');
create type notification_channel  as enum ('push','email','sms','whatsapp');
```

---

## 3. Reference tables

```sql
create table countries (
  id                    text primary key,               -- ISO-3166 alpha-2, e.g. 'IN','GB'
  name                  text not null,
  data_residency_region text not null,                  -- e.g. 'in','eu','sg','us'
  default_currency      text not null                   -- FK added after currencies
);

create table currencies (
  code        text primary key,                          -- ISO-4217, e.g. 'INR','USD'
  symbol      text not null,
  minor_unit  smallint not null default 2
);
alter table countries add constraint countries_currency_fk
  foreign key (default_currency) references currencies(code);

create table locales (
  id            uuid primary key default gen_random_uuid(),
  country_id    text not null references countries(id),
  language_code text not null,                            -- e.g. 'en','hi','ar'
  formatting    jsonb not null default '{}'::jsonb,
  unique (country_id, language_code)
);

create table institutions (
  id          uuid primary key default gen_random_uuid(),
  name        text not null,
  country_id  text references countries(id),
  type        text,                                       -- 'iit','iim','ivy','global_top','other'
  aliases     text[] not null default '{}'
);
create index institutions_name_trgm on institutions using gin (name extensions.gin_trgm_ops);

create table employers (
  id                  uuid primary key default gen_random_uuid(),
  name                text not null,
  domain              text,                               -- corporate email domain for verification
  category            text,                               -- masking category: 'big_tech','top_consulting','hospital'…
  verification_source text
);
create index employers_name_trgm on employers using gin (name extensions.gin_trgm_ops);

create table income_bands (
  id         smallint primary key,
  country_id text not null references countries(id),
  currency   text not null references currencies(code),
  label      text not null,                               -- e.g. '₹50L–1Cr'
  min_amount numeric,
  max_amount numeric
);

create table plans (
  code          plan_code primary key,
  tier_rank     smallint not null,                        -- 0 free … 3 concierge
  feature_flags jsonb not null default '{}'::jsonb,       -- entitlement keys → bool/limit
  active        boolean not null default true
);

create table plan_prices (
  id             uuid primary key default gen_random_uuid(),
  plan_code      plan_code not null references plans(code),
  country_id     text not null references countries(id),
  currency       text not null references currencies(code),
  billing_period billing_period not null,
  amount         numeric not null,                        -- market-anchored, tax treatment per §7
  tax_rule       text,
  effective_from timestamptz not null default now(),
  effective_to   timestamptz,
  unique (plan_code, country_id, billing_period, effective_from)
);

create table verification_types (
  code                 verification_signal primary key,
  default_method       verification_method not null,
  is_eligibility_signal boolean not null default false
);
```

---

## 4. Accounts, applications, profiles

```sql
-- 1:1 with auth.users. auth.uid() == accounts.id everywhere.
create table accounts (
  id                uuid primary key references auth.users(id) on delete cascade,
  public_handle     text not null unique,                 -- opaque ULID/uuid; NEVER a sequential id
  phone_enc         bytea,                                 -- app/pgsodium-encrypted; unique via hash below
  phone_hash        text unique,
  auth_provider     auth_provider not null default 'password',
  account_status    account_status not null default 'pending',
  identity_verified boolean not null default false,
  country_id        text references countries(id),
  locale_id         uuid references locales(id),
  created_at        timestamptz not null default now(),
  updated_at        timestamptz not null default now(),
  deleted_at        timestamptz                            -- soft-delete; hard-purge job clears PII
);
create index accounts_status_idx on accounts(account_status);

-- The "Getting In" funnel (ONB-1/3/4)
create table applications (
  id                 uuid primary key default gen_random_uuid(),
  account_id         uuid not null references accounts(id) on delete cascade,
  corridor           text not null,                        -- e.g. 'blr-lon'
  declared_gender    gender not null,                      -- reconciled against gov-ID at verification
  claimed_route      eligibility_route,
  status             application_status not null default 'draft',
  queue_position     integer,                              -- men only; null for instant-admit
  priority_score     numeric not null default 0,           -- verification depth + completeness + intent
  invited_by         uuid references accounts(id),          -- referral/vouch attribution
  invite_token       text unique,
  admitted_at        timestamptz,
  created_at         timestamptz not null default now(),
  updated_at         timestamptz not null default now()
);
create index applications_queue_idx on applications(corridor, status, priority_score desc);

create table profiles (
  id                 uuid primary key default gen_random_uuid(),
  account_id         uuid not null unique references accounts(id) on delete cascade,
  display_name       text not null,
  dob                date not null,                         -- PII; age derived, dob never exposed
  gender             gender not null,                       -- from verified gov-ID
  height_cm          smallint,
  religion           text,                                  -- OPTIONAL, never required (NG3)
  community          text,                                  -- OPTIONAL, never required
  mother_tongue      text,
  marital_status     marital_status,
  profession_title   text,
  seniority_level    seniority_level,
  income_band_id     smallint references income_bands(id),  -- optional
  current_employer_id uuid references employers(id),
  primary_institution_id uuid references institutions(id),
  city               text,
  country_id         text references countries(id),
  willing_to_relocate text,                                 -- 'no'|'within_country'|'international'|'regions'
  relocation_regions text[] not null default '{}',
  languages          text[] not null default '{}',
  partner_intent     partner_intent,
  intent_timeline    intent_timeline,
  bio                text,
  values_json        jsonb not null default '{}'::jsonb,
  incognito          boolean not null default false,
  hidden_profile     boolean not null default false,
  eligibility_basis  eligibility_route,
  visibility_json    jsonb not null default '{}'::jsonb,    -- per-field visibility_scope map (PRIV-02)
  employer_masked    boolean not null default true,         -- PRIV-05
  strength_score     smallint not null default 0,           -- owner-only (PROF-03)
  is_discoverable    boolean not null default false,        -- gate: verified + eligible + active
  last_active_at     timestamptz,                            -- reachability ranking (MATCH-02)
  created_at         timestamptz not null default now(),
  updated_at         timestamptz not null default now()
);
create index profiles_country_city_idx  on profiles(country_id, city);
create index profiles_last_active_idx    on profiles(last_active_at desc) where is_discoverable;
create index profiles_seniority_income   on profiles(seniority_level, income_band_id);
create index profiles_languages_gin      on profiles using gin (languages);

create table profile_photos (
  id                uuid primary key default gen_random_uuid(),
  profile_id        uuid not null references profiles(id) on delete cascade,
  storage_key       text not null,                          -- key in 'profile-photos' bucket (opaque)
  visibility        visibility_scope not null default 'on_mutual_match',
  is_primary        boolean not null default false,
  liveness_linked   boolean not null default false,          -- matched to VER-2 selfie
  moderation_status photo_status not null default 'pending',
  created_at        timestamptz not null default now()
);
create index profile_photos_profile_idx on profile_photos(profile_id);

create table educations (
  id             uuid primary key default gen_random_uuid(),
  profile_id     uuid not null references profiles(id) on delete cascade,
  institution_id uuid references institutions(id),
  degree         text,
  field          text,
  start_year     smallint,
  end_year       smallint,
  is_verified    boolean not null default false
);

create table employments (
  id          uuid primary key default gen_random_uuid(),
  profile_id  uuid not null references profiles(id) on delete cascade,
  employer_id uuid references employers(id),
  title       text,
  seniority   seniority_level,
  start_date  date,
  end_date    date,                                          -- null = current
  is_verified boolean not null default false
);

create table partner_preferences (
  profile_id       uuid primary key references profiles(id) on delete cascade,
  age_min          smallint,
  age_max          smallint,
  height_min_cm    smallint,
  income_band_min  smallint references income_bands(id),
  seniority_min    seniority_level,
  locations        text[] not null default '{}',
  education_level_min text,
  religion_pref    text[] not null default '{}',
  community_pref   text[] not null default '{}',
  relocation_ok    boolean not null default false,
  dealbreakers_json jsonb not null default '{}'::jsonb
);

create table family_collaborators (
  id                 uuid primary key default gen_random_uuid(),
  owner_account_id   uuid not null references accounts(id) on delete cascade,
  collaborator_account_id uuid references accounts(id),       -- set on accept (light-KYC)
  collaborator_email_enc bytea,                               -- encrypted invite target
  role               collaborator_role not null default 'viewer',
  scope_json         jsonb not null default '{}'::jsonb,      -- FAM-02 area toggles
  consent_granted    boolean not null default false,
  invite_token       text unique,
  revoked_at         timestamptz,
  created_at         timestamptz not null default now()
);
create index family_collab_owner_idx on family_collaborators(owner_account_id);
```

---

## 5. Verification & bans

```sql
create table verification_requests (
  id                 uuid primary key default gen_random_uuid(),
  account_id         uuid not null references accounts(id) on delete cascade,
  signal             verification_signal not null references verification_types(code),
  method             verification_method not null,
  evidence_vault_ref text,                                   -- pointer into locked vault; NOT the document
  status             verification_status not null default 'submitted',
  reviewer_account_id uuid references accounts(id),           -- null = automated
  source_check_result jsonb,
  submitted_at       timestamptz not null default now(),
  decided_at         timestamptz,
  retention_expires_at timestamptz                            -- evidence purged after window (minimization)
);
create index verif_req_account_idx on verification_requests(account_id, status);

create table verification_badges (
  id            uuid primary key default gen_random_uuid(),
  profile_id    uuid not null references profiles(id) on delete cascade,
  signal        verification_signal not null references verification_types(code),
  request_id    uuid references verification_requests(id),
  assurance     assurance_level not null,
  basis_label   text,                                         -- "Employer-verified via LinkedIn"
  verified_at   timestamptz not null default now(),
  expires_at    timestamptz,                                  -- drives re-verification cadence (VER-8)
  unique (profile_id, signal)
);
create index verif_badge_profile_idx on verification_badges(profile_id);

-- SAFE-06: identity-linked bans survive re-registration. Stores HASHES only, never raw PII.
create table ban_tokens (
  id            uuid primary key default gen_random_uuid(),
  gov_id_hash   text,                                         -- salted hash of ID number
  face_embed_ref text,                                        -- pointer to vault embedding
  device_cluster_id text,
  reason        text not null,
  created_by    uuid references accounts(id),
  created_at    timestamptz not null default now()
);
create index ban_tokens_govid_idx on ban_tokens(gov_id_hash);
```

---

## 6. Discovery, interest & messaging

```sql
create table interests (
  id                  uuid primary key default gen_random_uuid(),
  sender_profile_id   uuid not null references profiles(id) on delete cascade,
  receiver_profile_id uuid not null references profiles(id) on delete cascade,
  note                text not null,                          -- INT-02 effort tax (required)
  status              interest_status not null default 'sent',
  quality_score       numeric,                                -- anti low-effort mass-like
  expires_at          timestamptz not null,                   -- INT-05 (~7 days)
  created_at          timestamptz not null default now(),
  unique (sender_profile_id, receiver_profile_id)
);
create index interests_receiver_pending_idx
  on interests(receiver_profile_id, status, quality_score desc)
  where status = 'sent';                                      -- INT-04 ranked working queue
create index interests_sender_idx on interests(sender_profile_id, status);

create table conversations (
  id                 uuid primary key default gen_random_uuid(),
  participant_a      uuid not null references profiles(id) on delete cascade,
  participant_b      uuid not null references profiles(id) on delete cascade,
  opened_by_interest uuid references interests(id),
  contact_revealed_a boolean not null default false,          -- MSG-02 two-sided handshake
  contact_revealed_b boolean not null default false,
  created_at         timestamptz not null default now(),
  last_message_at    timestamptz,
  constraint conversations_pair_unique unique (participant_a, participant_b),
  constraint conversations_pair_order check (participant_a < participant_b)
);

create table messages (
  id                uuid primary key default gen_random_uuid(),
  conversation_id   uuid not null references conversations(id) on delete cascade,
  sender_profile_id uuid not null references profiles(id),
  body              text not null,                            -- PII; contact-info redacted pre-consent
  safety_flags      jsonb not null default '{}'::jsonb,       -- SAFE-05 filter output
  created_at        timestamptz not null default now(),
  read_at           timestamptz
);
create index messages_convo_idx on messages(conversation_id, created_at);

create table profile_views (
  id                uuid primary key default gen_random_uuid(),
  viewer_profile_id uuid not null references profiles(id) on delete cascade,
  viewed_profile_id uuid not null references profiles(id) on delete cascade,
  was_incognito     boolean not null default false,           -- DISC-02: hide from who-viewed-me
  viewed_at         timestamptz not null default now()
);
create index profile_views_viewed_idx on profile_views(viewed_profile_id, viewed_at desc) where not was_incognito;
create index profile_views_dedupe_idx on profile_views(viewer_profile_id, viewed_profile_id);

create table shortlists (
  id                uuid primary key default gen_random_uuid(),
  owner_profile_id  uuid not null references profiles(id) on delete cascade,
  saved_profile_id  uuid not null references profiles(id) on delete cascade,
  created_at        timestamptz not null default now(),
  unique (owner_profile_id, saved_profile_id)
);

create table blocks (
  id                uuid primary key default gen_random_uuid(),
  actor_account_id  uuid not null references accounts(id) on delete cascade,
  target_account_id uuid not null references accounts(id) on delete cascade,
  created_at        timestamptz not null default now(),
  unique (actor_account_id, target_account_id)
);
create index blocks_target_idx on blocks(target_account_id);
```

---

## 7. Monetization

```sql
create table subscriptions (
  id                    uuid primary key default gen_random_uuid(),
  account_id            uuid not null references accounts(id) on delete cascade,
  plan_code             plan_code not null references plans(code),
  status                subscription_status not null default 'active',
  auto_renew            boolean not null default false,        -- PAY-04 explicit opt-in
  billing_period        billing_period not null,
  outcome_guarantee_json jsonb not null default '{}'::jsonb,   -- PAY-08 reachability counter
  current_period_start  timestamptz not null default now(),
  current_period_end    timestamptz,
  paused_at             timestamptz,
  created_at            timestamptz not null default now(),
  updated_at            timestamptz not null default now()
);
create index subscriptions_account_idx on subscriptions(account_id, status);

create table payments (
  id              uuid primary key default gen_random_uuid(),
  account_id      uuid not null references accounts(id) on delete cascade,
  subscription_id uuid references subscriptions(id),
  currency        text not null references currencies(code),
  amount          numeric not null,
  tax_amount      numeric not null default 0,
  gateway         text,
  gateway_ref     text,                                        -- PII-ish; idempotency key stored here
  status          payment_status not null default 'pending',
  refund_reason   text,
  created_at      timestamptz not null default now()
);
create index payments_account_idx on payments(account_id, created_at desc);

create table concierge_engagements (
  id                     uuid primary key default gen_random_uuid(),
  account_id             uuid not null references accounts(id) on delete cascade,
  subscription_id        uuid references subscriptions(id),
  relationship_manager_id uuid references accounts(id),
  scope_json             jsonb not null default '{}'::jsonb,
  success_fee_json       jsonb not null default '{}'::jsonb,
  status                 text not null default 'active',
  started_at             timestamptz not null default now()
);

create table concierge_milestones (
  id            uuid primary key default gen_random_uuid(),
  engagement_id uuid not null references concierge_engagements(id) on delete cascade,
  label         text not null,
  target_date   date,
  refundable_amount numeric,
  status        milestone_status not null default 'pending',
  met_at        timestamptz
);

-- PAY-09 referral: reward vests on invitee VERIFIED activation, ratio-weighted
create table referrals (
  id                 uuid primary key default gen_random_uuid(),
  referrer_account_id uuid not null references accounts(id) on delete cascade,
  referee_account_id  uuid references accounts(id),
  invite_token       text unique,
  reward_multiplier  numeric not null default 1,               -- scarce-gender 2–3×
  vested             boolean not null default false,
  vested_at          timestamptz,
  created_at         timestamptz not null default now()
);

-- "Vouch for a friend" reputation (PAY-09 / verified≠vouched)
create table vouches (
  id                 uuid primary key default gen_random_uuid(),
  voucher_account_id uuid not null references accounts(id) on delete cascade,
  vouched_account_id uuid not null references accounts(id) on delete cascade,
  weight             numeric not null default 1,               -- reputation-weighted, sybil-resistant
  created_at         timestamptz not null default now(),
  unique (voucher_account_id, vouched_account_id)
);
```

---

## 8. Trust & safety

```sql
create table reports (
  id                uuid primary key default gen_random_uuid(),
  reporter_account_id uuid references accounts(id) on delete set null,
  target_account_id uuid not null references accounts(id) on delete cascade,
  reason            report_reason not null,
  evidence_json     jsonb not null default '{}'::jsonb,        -- message ids + snapshot refs
  status            report_status not null default 'submitted',
  lane              text,                                       -- P0..P3
  sla_due_at        timestamptz,
  created_at        timestamptz not null default now(),
  closed_at         timestamptz
);
create index reports_target_idx on reports(target_account_id);
create index reports_open_idx   on reports(status) where status not in ('closed','dismissed');

-- Append-only. No updates/deletes (enforced by policy + no update grant). SAFE-10.
create table moderation_actions (
  id                  uuid primary key default gen_random_uuid(),
  report_id           uuid references reports(id),
  target_account_id   uuid not null references accounts(id),
  moderator_account_id uuid not null references accounts(id),
  second_approver_id  uuid references accounts(id),             -- four-eyes for permanent bans
  action              moderation_action not null,
  reason_code         text not null,
  rationale           text,
  evidence_refs       text[],
  created_at          timestamptz not null default now()
);
create index mod_actions_target_idx on moderation_actions(target_account_id, created_at desc);

create table notifications (
  id          uuid primary key default gen_random_uuid(),
  account_id  uuid not null references accounts(id) on delete cascade,
  category    notification_category not null,
  type        text not null,
  channel     notification_channel not null,
  payload_json jsonb not null default '{}'::jsonb,             -- NOTIF-05: no identity leak
  read_at     timestamptz,
  created_at  timestamptz not null default now()
);
create index notifications_account_idx on notifications(account_id, created_at desc);

create table notification_prefs (
  account_id  uuid not null references accounts(id) on delete cascade,
  type        text not null,
  channel     notification_channel not null,
  enabled     boolean not null default true,
  primary key (account_id, type, channel)
);
```

---

## 9. Consent, audit & success stories

```sql
create table consent_records (
  id            uuid primary key default gen_random_uuid(),
  account_id    uuid not null references accounts(id) on delete cascade,
  purpose       consent_purpose not null,                     -- DPDP: unbundled, one row per purpose
  lawful_basis  consent_basis not null,
  granted       boolean not null,
  policy_version text not null,
  region        text,
  granted_at    timestamptz not null default now(),
  withdrawn_at  timestamptz
);
create index consent_account_purpose_idx on consent_records(account_id, purpose);

-- Append-only, tamper-evident (hash-chained). SAFE-10 / §7.9.
create table audit_logs (
  id              bigint generated always as identity primary key,
  actor_account_id uuid references accounts(id),               -- null = system
  action          text not null,
  entity_type     text not null,
  entity_id       uuid,
  metadata_json   jsonb not null default '{}'::jsonb,
  prev_hash       text,                                        -- hash chain
  row_hash        text,
  created_at      timestamptz not null default now()
);
create index audit_entity_idx on audit_logs(entity_type, entity_id);

create table success_stories (
  id          uuid primary key default gen_random_uuid(),
  profile_id  uuid references profiles(id) on delete set null,
  consented   boolean not null default false,
  body        text,
  published_at timestamptz
);
```

---

## 10. Row-Level Security (RLS)

Enable RLS on **every** table, then grant only what a client should touch directly. Cross-profile reads (discovery, who-viewed-me) and any write with fairness/ratio logic go through `SECURITY DEFINER` RPCs — clients never run those queries raw.

```sql
-- turn RLS on for all app tables
do $$ declare t text;
begin
  for t in select tablename from pg_tables where schemaname='public' loop
    execute format('alter table public.%I enable row level security;', t);
  end loop;
end $$;
```

**Reference tables** (countries, currencies, locales, institutions, employers, income_bands, plans, plan_prices, verification_types): readable by any authenticated user.

```sql
create policy ref_read on countries   for select using (auth.role() = 'authenticated');
-- (repeat the same read-only policy for the other reference tables)
```

**Own account / profile:**

```sql
create policy account_self on accounts
  for select using (id = auth.uid());
create policy account_self_upd on accounts
  for update using (id = auth.uid());

create policy profile_owner_all on profiles
  for all using (account_id = auth.uid()) with check (account_id = auth.uid());
```

**Viewing other people's profiles** goes through a helper so blocks + discoverability + visibility are always applied:

```sql
create or replace function can_view_profile(target uuid)
returns boolean language sql stable security definer as $$
  select
    p.is_discoverable
    and p.account_id <> auth.uid()
    and not exists (                       -- neither side has blocked the other
      select 1 from blocks b
      where (b.actor_account_id = auth.uid()  and b.target_account_id = p.account_id)
         or (b.actor_account_id = p.account_id and b.target_account_id = auth.uid())
    )
  from profiles p where p.id = target;
$$;

create policy profile_read_others on profiles
  for select using ( account_id = auth.uid() or can_view_profile(id) );
```

**Interests** — sender or receiver only; delivery caps / inbound-cap enforcement live in the `send_interest()` RPC:

```sql
create policy interest_party_read on interests
  for select using (
    sender_profile_id   in (select id from profiles where account_id = auth.uid())
    or receiver_profile_id in (select id from profiles where account_id = auth.uid())
  );
-- inserts happen only via public.send_interest(...) SECURITY DEFINER (enforces INT-01/02/04/06 + BAL).
```

**Conversations / messages** — participants only:

```sql
create policy convo_party on conversations
  for select using (
    participant_a in (select id from profiles where account_id = auth.uid())
    or participant_b in (select id from profiles where account_id = auth.uid())
  );

create policy message_party on messages
  for select using (
    conversation_id in (
      select c.id from conversations c
      where c.participant_a in (select id from profiles where account_id = auth.uid())
         or c.participant_b in (select id from profiles where account_id = auth.uid())
    )
  );
```

**Blocks / shortlists / subscriptions / payments / reports / notifications / consent** — owner-scoped `for all using (account_id = auth.uid())` (or the owning profile). **`moderation_actions`, `audit_logs`, `ban_tokens`, `verification_requests`** — **no client policy at all** (service-role only; reached exclusively via back-office RPCs), which is how "least-privilege, append-only, brokered" is enforced.

---

## 11. Storage buckets (Supabase Storage)

```sql
-- Public-safe profile media (still served via signed URLs from the app, blur handled app-side)
insert into storage.buckets (id, name, public) values ('profile-photos','profile-photos', false);

-- Sensitive verification documents — locked, service-role only (the "vault", see §12)
insert into storage.buckets (id, name, public) values ('verification-docs','verification-docs', false);
```

Photo bucket policy — a user manages only their own object prefix; other members receive **short-lived signed URLs minted by the app** (authz + block + match-state checked at mint), never a public URL:

```sql
create policy own_photos on storage.objects
  for all to authenticated
  using  (bucket_id = 'profile-photos' and (storage.foldername(name))[1] = auth.uid()::text)
  with check (bucket_id = 'profile-photos' and (storage.foldername(name))[1] = auth.uid()::text);

-- verification-docs: NO authenticated policy → only the service role (back-office broker) can touch it.
```

---

## 12. Sensitive-data vault

The most sensitive data is **never** in the tables above as plaintext:

- **Documents** (gov-ID, marksheets, income proofs, liveness selfies) → the locked `verification-docs` bucket. `verification_requests.evidence_vault_ref` stores only the object key; `retention_expires_at` drives purge (data minimization).
- **Biometric embeddings** (face) → vault, referenced by `ban_tokens.face_embed_ref` / a request row; accessed only by the matching service via broker, never by moderators.
- **Field-level encryption** for `phone_enc`, `collaborator_email_enc`, and message bodies (PII) via **Supabase Vault / pgsodium** — keys separate from ciphertext so a table dump isn't plaintext, and per-subject keys enable **crypto-shred** on erasure (PRIVC-04).
- **Ban list stores hashes only** (`gov_id_hash`), never raw ID numbers — so identity-linked bans survive erasure of the profile without retaining PII.
- **Data residency:** run a Supabase project (or schema) per `countries.data_residency_region`; the global ban index holds only hashes/pointers, no PII (PRIVC-05).

---

## 13. Key indexes & access patterns

Already created inline above; the load-bearing ones:

| Access pattern | Index |
|---|---|
| Facet search (SRCH-01) | `profiles_country_city_idx`, `profiles_seniority_income`, `profiles_languages_gin`, `institutions/employers *_trgm` |
| Reachability ranking (MATCH-02) | `profiles_last_active_idx` (partial, `where is_discoverable`) |
| Inbound working queue (INT-04) | `interests_receiver_pending_idx` (partial, `where status='sent'`, ranked by `quality_score`) |
| Seen-dedupe / freshness (FRESH-01) | `profile_views_dedupe_idx` |
| Who-viewed-me (DISC-02) | `profile_views_viewed_idx` (partial, `where not was_incognito`) |
| Block enforcement | `blocks_target_idx` + `can_view_profile()` |
| Admission queue (ONB-4) | `applications_queue_idx` (ranked by `priority_score`) |
| Open moderation cases | `reports_open_idx` (partial) |

---

## 14. `updated_at` triggers

```sql
-- attach to every table that has an updated_at column
create trigger set_updated_at before update on accounts
  for each row execute function extensions.moddatetime(updated_at);
create trigger set_updated_at before update on applications
  for each row execute function extensions.moddatetime(updated_at);
create trigger set_updated_at before update on profiles
  for each row execute function extensions.moddatetime(updated_at);
create trigger set_updated_at before update on subscriptions
  for each row execute function extensions.moddatetime(updated_at);
```

---

## 15. Enforced-in-code, not in schema

Some behavior is deliberately **not** a table constraint — it lives in `SECURITY DEFINER` RPCs / Edge Functions so the fairness and safety logic can't be bypassed by a raw insert:

- **`send_interest(receiver, note)`** — checks daily cap (INT-01), effort tax (INT-02), receiver inbound cap (INT-04), block state, and the ratio governor before inserting an `interests` row.
- **Gender-gated admission valve (BAL-01/02)** — an admission job computes `applications.queue_position` from active-female supply; never client-writable.
- **Discovery / curated picks (DISC-01, MATCH-02)** — a ranking RPC returns handles; clients never scan `profiles` directly (protects reachability + reciprocity + non-enumeration).
- **Reachability guarantee (PAY-08)** — a scheduled job updates `subscriptions.outcome_guarantee_json` and extends `current_period_end`.

_This schema is a draft; column names/enums are stable references. Load-bearing details (encryption choice, per-region project split, exact caps) are noted for the build team to finalize._
