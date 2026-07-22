# Stage 3 — Building Your Profile
_You want to show a rich, verified profile — while deciding exactly who sees what, before any stranger can see you._

**Covers:** PROF-01, PROF-03, PROF-08, MEDIA-01, MEDIA-05, PRIV-01, PRIV-02, PRIV-05, PRIV-06, FAM-01

## The journey (plain language, numbered)
1. **We set your privacy first.** Before your profile is visible to anyone, we ask you to set incognito, photo blur, and per-field visibility. You're invisible until you say go.
2. **You fill in your profile.** Required basics (name/initial, age, city, height, profession band, intent, bio) plus optional high-signal fields. A strength meter shows how you're doing. Community and religion are clearly optional — skip them and you can still reach 100.
3. **You add photos your way.** Every photo is blurred to non-mutual viewers by default. You can make some private, or add no face photo at all — your verification carries the trust instead of your face.
4. **You mask what you want proven-but-private.** Turn on employer masking so people see "verified at [Big Tech]" without the company name until you choose to reveal it.
5. **You choose who can see what, field by field.** Income, community, employer — each has its own visibility switch. Hidden means hidden, everywhere.
6. **You can block anyone, silently.** One tap. They're never told; they simply can't find you.
7. **(Optional) You invite family.** Send a parent a private, revocable seat — Viewer or Assistant. They never message on your behalf without your OK, and you can revoke instantly.
8. **You publish** once the minimum set is met. Everything you hid stays hidden; nothing you skipped counts against you.

## Screens

### Screen: Set Your Privacy First (first-run)
```
┌──────────────────────────────────────────┐
│  ← Step 1 of 3                            │
│                                          │
│   🔒 Set your privacy                    │
│      before anyone sees you              │
│                                          │
│   You are currently INVISIBLE.           │
│   Nothing publishes until you finish.    │
│                                          │
│  ┌────────────────────────────────────┐  │
│  │ Incognito browsing        [ ON ●] │  │
│  │ Browse without leaving a trace.    │  │
│  │ (You also won't see incognito      │  │
│  │  viewers — it's symmetric.)        │  │
│  ├────────────────────────────────────┤  │
│  │ Blur my photos           [ ON ●] │  │
│  │ Clear only on a mutual match.      │  │
│  ├────────────────────────────────────┤  │
│  │ Hidden profile mode      [ OFF ○] │  │
│  │ Only people you match with find    │  │
│  │ you. No new discovery.             │  │
│  └────────────────────────────────────┘  │
│                                          │
│  These defaults are safe. Change any     │
│  time in Settings.                       │
│                                          │
│            [  Continue  →  ]              │
└──────────────────────────────────────────┘
```
- **What happens:** Before you're ever discoverable, we default you to private and explain each control in one line. You can only become visible after this step.
- **Edge cases:**
  - `Skips reading, taps Continue → safe defaults (incognito on, photos blurred) stay applied — never accidentally exposed`
  - `Turns everything off → allowed, but a one-line "you'll be fully public" confirm appears first`
  - `Closes app mid-setup → profile stays DRAFT + invisible; resumes here next time`

### Screen: Profile Editor (with strength meter)
```
┌──────────────────────────────────────────┐
│  Your profile                    DRAFT   │
│  ┌────────────────────────────────────┐  │
│  │ Profile strength         72 / 100 │  │
│  │ ▓▓▓▓▓▓▓▓▓▓▓▓▓▓░░░░░                │  │
│  │ ↑ Verify your employer  +12        │  │
│  │ ↑ Add a second photo    +6         │  │
│  │ (only you can see this score)      │  │
│  └────────────────────────────────────┘  │
│                                          │
│  BASICS                        required  │
│  Name/initial   Aarav            ✎      │
│  Age            31   ·  City  Bangalore │
│  Height         178 cm                  │
│                                          │
│  CAREER                                  │
│  Profession     Senior PM · Manager  ✎  │
│  Employer       ✔ Verified · Big Tech   │
│                 🔒 masked  ·  edit ›     │
│  Income band    optional         Add ›  │
│                                          │
│  ABOUT YOU                               │
│  Intent + timeline  Marriage · 1–2yr    │
│  Bio            ✎  (min 1 more line)     │
│                                          │
│  OPTIONAL — never required               │
│  Community / religion       Skip · Add › │
│  Languages · Family details · Values     │
│                                          │
│         [  Preview  ]   [  Next  ]       │
└──────────────────────────────────────────┘
```
- **What happens:** You fill fields; the meter (visible only to you) shows what would raise your score. Community/religion sit under a clearly labelled "never required" group with a one-tap Skip.
- **Edge cases:**
  - `Skips community/religion → publishes normally, can still reach 100; that field carries zero weight`
  - `Hides a field you filled in → counts as provided; strength does NOT drop for hiding it`
  - `Leaves a required field blank → inline "required" flag blocks publish; every other field is kept, no data lost`
  - `Pastes contact info in bio → auto-redacted before anyone sees it; you're notified`

### Screen: Photo Manager (blur / private / face-optional)
```
┌──────────────────────────────────────────┐
│  ← Photos                                │
│                                          │
│  Default: your photos are BLURRED to     │
│  people you haven't matched with.        │
│                                          │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐     │
│  │ [photo] │ │ ▒▒▒▒▒▒▒ │ │    +    │     │
│  │ PRIMARY │ │ ▒blur▒▒ │ │  Add    │     │
│  │ On match│ │ Private │ │         │     │
│  └─────────┘ └─────────┘ └─────────┘     │
│                                          │
│  Tap a photo to set who sees it:         │
│   ○ Public                               │
│   ● Clear on mutual match  (default)     │
│   ○ Specific people I grant              │
│   ○ Private (only me)                    │
│                                          │
│  ┌────────────────────────────────────┐  │
│  │ Face-photo optional      [ ON ●] │  │
│  │ Publish with no public face photo. │  │
│  │ Your verified badges vouch for you │  │
│  │ instead. You can reveal a face     │  │
│  │ later, on a match.                 │  │
│  └────────────────────────────────────┘  │
│                                          │
│  Preview as others see you  →  🙍 (blur) │
└──────────────────────────────────────────┘
```
- **What happens:** Each photo has its own visibility. Turn on face-optional to publish with no public face photo — you'll show a verified silhouette with your badges, and nobody ever receives a clear image without authorization.
- **Edge cases:**
  - `Hides ALL photos → allowed only if face-optional is on; otherwise profile drops to LIMITED and prompts you to fix it`
  - `Face-optional ON, zero face photos → verified-silhouette placeholder + badge cluster shown; fully discoverable, not down-ranked`
  - `Sets your primary photo to Private → we ask you to pick a visible primary, or a silhouette shows on your discovery card — never the hidden image`
  - `Match then block/unmatch → clear photos revert to inaccessible immediately for that person`

### Screen: Privacy & Visibility Settings
```
┌──────────────────────────────────────────┐
│  ← Privacy & visibility                  │
│                                          │
│  BROWSING                                │
│  Incognito browsing          [ ON ●]   │
│  Hidden profile mode         [ OFF ○]   │
│                                          │
│  EMPLOYER MASKING                        │
│  ┌────────────────────────────────────┐  │
│  │ Mask my employer name    [ ON ●] │  │
│  │ Others see:                        │  │
│  │   ✔ Verified at [Big Tech]         │  │
│  │ Real name revealed only on a       │  │
│  │ match or when you choose.          │  │
│  └────────────────────────────────────┘  │
│                                          │
│  WHO SEES EACH FIELD                     │
│  Income band       Hidden          ›     │
│  Community/relig.  Hidden          ›     │
│  Location          Verified members ›    │
│  Languages         Public           ›    │
│    each: Public · Verified · On match ·  │
│          Connections · Hidden            │
│                                          │
│  BLOCKED PEOPLE                          │
│  Manage blocks (silent)            ›     │
│                                          │
│  Most-restrictive setting always wins.   │
└──────────────────────────────────────────┘
```
- **What happens:** One place for incognito, employer masking, and per-field visibility. Masking proves your employer's caliber ("verified at [Big Tech]") without the name. Every field carries its own switch; the strictest of your settings always applies.
- **Edge cases:**
  - `Masking ON but employer visibility set to Hidden → hidden wins; not even the category shows`
  - `Sets community/religion to Hidden → never shown, and never usable as a filter that could out the value`
  - `Field set Public but account cap is Verified-members-only → cap wins; shows only to verified members`
  - `Employer verification expires → the "verified at [Big Tech]" badge drops automatically`

### Screen: Block Someone (silent)
```
┌──────────────────────────────────────────┐
│  Block this person?                      │
│                                          │
│  They will NOT be told.                  │
│  • Can't view your profile or photos     │
│  • Can't find you in search/discovery    │
│  • Can't send you interest or messages   │
│  • You won't see them anywhere either    │
│                                          │
│  This stays in effect even if they       │
│  make a new account.                     │
│                                          │
│  ☐ Also report this person               │
│                                          │
│      [ Cancel ]      [ Block ]           │
└──────────────────────────────────────────┘
```
- **What happens:** One tap from any profile. The block is silent and mutual — they get no signal at all, and it follows their identity across re-registration.
- **Edge cases:**
  - `Blocked person opens an old share link → generic "not viewable" / 404, never "you're blocked"`
  - `Blocked person searches your name or employer → you simply never appear`
  - `You unblock later → visibility resumes going forward; no record of the block is shown to them`
  - `Blocked person tries via a family collaborator → cannot circumvent on their behalf`

### Screen: Invite a Family Collaborator
```
┌──────────────────────────────────────────┐
│  ← Invite family                         │
│                                          │
│  Give a parent or sibling a safe,        │
│  private seat. You stay in control.      │
│                                          │
│  Their role:                             │
│   ● Viewer                               │
│     Sees what you allow. Can't act.      │
│   ○ Assistant                            │
│     Can suggest edits & shortlist —       │
│     every change needs YOUR approval.    │
│                                          │
│  They can NEVER:                         │
│   • message or send interest as you      │
│   • see your chats (unless you share)    │
│   • change your privacy or blocks        │
│                                          │
│  Send private invite to:                 │
│  ┌────────────────────────────────────┐  │
│  │ mom@email.com                      │  │
│  └────────────────────────────────────┘  │
│  Single-use · expires in 7 days ·        │
│  they verify their own identity.         │
│                                          │
│      [  Send invite  ]                   │
│                                          │
│  Manage collaborators · Revoke   ›       │
└──────────────────────────────────────────┘
```
- **What happens:** You send a private, single-use, revocable invite and pick a scope. They get zero access until they accept, verify their own identity, and your consent is recorded. You're always the sole decision-maker.
- **Edge cases:**
  - `Revoke access → all their sessions and in-flight photo access end within seconds; your own conversations are never disrupted`
  - `Invite forwarded to the wrong person → single-use + identity binding voids it if it doesn't match`
  - `Assistant proposes an edit to a verified field → on your approval that field re-enters verification; badge removed until re-verified`
  - `Assistant proposes adding community/religion you skipped → stays optional; you can reject; never becomes required`
  - `You're under pressure → revoke is always one tap and can't be blocked; solo mode hides sensitive candidates from all collaborators`
