# Architecture plan — aatam

## Overview

A mobile-first, local-first scoring app covering cricket, football, kabaddi, badminton, tennis,
volleyball and basketball at launch, built so a player's match history follows them by email/
phone even before they've installed the app. One shared logic core drives three surfaces: the
native mobile app (the product), a web build of the same UI (secondary), and a server-rendered
public site (the only surface that needs to rank on Google or unfurl correctly in a WhatsApp
share).

```
+-----------------------------------------------------------------------+
|  apps/mobile (Expo/RN)     apps/web (RN Web)     apps/marketing       |
|  primary product           dashboards, setup     (Next.js SSR)        |
|  offline-first scoring     desktop review         match/profile       |
|                                                    pages, claim        |
|                                                    landing pages       |
+---------------+-------------------+-------------------+---------------+
                |                   |                   |
                +---------+---------+---------+---------+
                          v                   v
              packages/core-engines   packages/identity
              (cricket / set / invasion)  (UUID, shadow profile, claim)
                          |                   |
                          v                   v
              packages/storage          cloud identity index
              (on-device, source of      (thin -- id, claim
               truth for a live match)    status, stat pointers)
                                                |
                                                v
                                    per-user Google Drive backup
                                    (only after a profile is claimed)
```

## ADRs

### ADR-001: Monorepo, three apps, shared packages
**Decision.** One monorepo (pnpm workspaces + Turborepo/Nx -- pick in issue 001). Three apps
(`mobile`, `web`, `marketing`) consume shared packages (`core-engines`, `identity`, `i18n`, `ui`,
`storage`). `mobile` and `web` share nearly all UI via React Native Web; `marketing` is a
separate Next.js app that shares only `ui`'s design tokens, not its components, because it needs
server rendering that RN Web doesn't give us reliably (see ADR-006).
**Rationale.** Avoids two parallel UI codebases for mobile and desktop, while keeping the one
surface that genuinely needs SSR (public/SEO pages) out of an RN Web app that can't do SSR well.
**Implications.** `core-engines` must stay framework-free -- it's imported by three very different
runtimes, including a Node SSR context that has no RN globals at all.

### ADR-002: Player identity is a UUID, decoupled from auth
**Decision.** Every player is a UUID with a `claim_status` (`unclaimed` | `claimed`) and zero or
more linked identifiers (email, phone). Stats and match records reference the UUID, never an
account row directly.
**Rationale.** This is the only way an unrecognised player's history can be assembled before they
ever sign in, and later claimed safely once they do. Coupling stats to "has an account" makes the
whole claim flow (ADR-004) impossible to retrofit.
**Implications.** Every query that "gets a player's stats" resolves through the UUID layer, even
for the common case of a signed-in user looking at their own data.

### ADR-003: Local-first storage, thin cloud index
**Decision.** The device is the source of truth for a match in progress (local SQLite/
WatermelonDB -- pick in issue 001). The cloud holds only an identity index: player UUID -> claim
status -> pointers to the matches/stats that reference them. It is not a general backend and the
client never blocks a scoring action on it.
**Rationale.** Scoring has to work on bad stadium wifi with the screen locking mid-match. A
"cloud is the source of truth" model fails exactly at the moment the product matters most.
**Implications.** Sync is an eventually-consistent background queue. Conflict resolution (two
devices editing the same match) is a real design problem -- flagged as a risk below, resolved in
a dedicated phase-0 issue, not improvised later.

### ADR-004: Two-path claim flow
**Decision.**
- **Verified path** -- a Gmail sign-in or an OTP-confirmed phone number auto-merges any shadow
  profile under that identifier. No review step; the auth step *is* the proof of ownership.
- **Suggested path** -- an organic install with no invite link and no prior identifier on file
  shows private "possible matches" (name + team + date range) that the person must confirm before
  anything merges. Never auto-merges on a fuzzy match.
- **Invite path** -- a share link carries a claim token bound to one specific shadow profile;
  opening it goes straight to sign-in, then an auto-merge prompt. This is the default share
  mechanism and should cover most claims in practice.
**Rationale.** Safety (no impersonation) and recall (most people do eventually get their history)
are both required; picking one extreme sacrifices the other.
**Implications.** The suggested path needs a private, per-viewer "possible matches" query that
must never be visible to anyone but the person it's suggesting to.

### ADR-005: Google Sign-In as primary auth; Drive as per-user backup
**Decision.** Google Sign-In is the primary identity provider. Once a profile is claimed, its
full match history backs up to that person's own Google Drive (`appDataFolder`), not to a
platform-wide store.
**Rationale.** Matches the "own data, own Drive" promise made to users, and reuses Google's
auth as the ownership proof for the verified claim path (ADR-004).
**Implications.** Phone/OTP sign-in (for users without Gmail) needs its own, separate backup
story -- flagged as an open question in phase 0.

### ADR-006: Public/SEO surface is a separate SSR app
**Decision.** `apps/marketing` is Next.js (SSR/SSG), not React Native Web. It owns match result
pages, player profile pages, city/tournament leaderboards, and claim-link landing pages.
**Rationale.** Plain RN Web is a client-rendered SPA -- crawlers see a near-empty shell, and
WhatsApp/X link unfurling needs real server-rendered Open Graph tags. Nothing about RN Web solves
this reliably today.
**Implications.** Any content that needs to rank or unfurl must be written to work in
`apps/marketing`, not bolted onto `apps/web` later. Revisit if Expo Router's web SSR matures
enough to collapse this into one app (tracked as a future ADR, not assumed now).

### ADR-007: Three scoring engines cover the launch sport set
**Decision.** Cricket gets its own engine (overs/wickets/innings). Badminton, tennis and
volleyball share a set/point engine (points-to-X, sets, win-by-2). Football, basketball and
kabaddi share an invasion engine (goals/points, periods, timer). Later sports either slot into an
existing engine (table tennis, hockey, handball) or need a new one (kho-kho, wrestling) -- see
`plan/ROADMAP.md` for the full phase mapping.
**Rationale.** Avoids seven bespoke scoring implementations; new sports in an existing family are
mostly UI work.
**Implications.** Each engine's state tree and derived-stats logic lives once in
`packages/core-engines`, covered by unit tests with no framework dependency.

### ADR-008: i18n is key-driven from commit one
**Decision.** Every UI string is a translation key against `packages/i18n`, even before a second
language exists. Phase 1 launches with en, hi, bn, mr, te, ta, gu, kn; phase 2 adds ml, pa, or,
as.
**Rationale.** Retrofitting i18n onto a string-literal codebase is expensive and error-prone;
the key layer costs almost nothing to add up front.
**Implications.** No component may render a literal string; CI should eventually lint for this
(tracked as a phase-0 issue once the codebase exists to lint).

### ADR-009: Mobile-first is a process rule, not just a design rule
**Decision.** Every UI issue's acceptance criteria default to mobile viewport verification
(360-390px) before desktop; every scoring-path issue's acceptance criteria default to an offline
test.
**Rationale.** Stated as its own ADR because it changes what "done" means for nearly every issue,
not just visual ones -- see `CLAUDE.md` for the enforced version of this rule.

## Risks & tech debt to watch

| Risk | Why it matters | Mitigation |
|---|---|---|
| Offline conflict resolution | Two devices (or a device and a stale cloud state) editing the same match | Dedicated design spike in phase 0, not improvised during a later feature |
| Claim-flow abuse | Fuzzy matching, if ever loosened, is an impersonation vector | Suggested path never auto-merges; verified path relies on real auth proof only |
| Bundle size with 3+ scoring engines | Mobile app size affects install conversion, especially on low-end Android | Code-split engines per sport; only bundle what's selected/downloaded |
| Translation quality | Machine translation for 8 languages at launch risks bad UX for native speakers | Native-speaker review pass before each language ships, not just MT |
| Play Store review for Drive/Sign-In scopes | Google's OAuth verification for sensitive scopes can take days-weeks | Start the verification process in phase 0, not at launch |
| Phone-only users have no Drive backup path | ADR-005 only solves backup for Gmail users | Open question for phase 0's grill-me pass |

## Clarifying questions for review (answer before phase 0 closes)

1. Local DB choice: WatermelonDB (sync-oriented, more setup) vs plain expo-sqlite (simpler, sync
   hand-rolled)? Recommendation: WatermelonDB, since its sync engine already solves a chunk of
   ADR-003's conflict-resolution problem.
2. Cloud identity index: Firebase/Firestore vs Supabase? Recommendation: Firestore -- simplest
   path to the thin, mostly-key-value index ADR-003 describes, and pairs naturally with Google
   Sign-In.
3. Backup story for phone-only (non-Gmail) users -- device-local export/import only for v1, or
   build a second backup provider? Recommendation: device-local export/import for v1, revisit
   once phone-auth volume is known.
4. Monorepo tool: Turborepo vs Nx? Recommendation: Turborepo -- lighter setup for a 3-app,
   5-package repo; revisit if generator/scaffolding needs grow.

## ADR amendments

### Amendment A1 (2026-10-08, issue 001): open tool choices resolved
Answers to clarifying questions 1, 2 and 4 above, confirmed with the human. The original ADR text
stays as written.
- **ADR-001 monorepo tool: Turborepo** on pnpm workspaces. Shared versions (react, typescript,
  vitest) are pinned once in the `catalog:` block of `pnpm-workspace.yaml`.
- **ADR-003 local DB: WatermelonDB.** It needs a native module, so the mobile app runs on an Expo
  development build, not Expo Go. Added to `packages/storage` in issue 005, not before.
- **ADR-003 cloud identity index: Firestore.** Projects are named `aatam-dev` / `aatam-staging` /
  `aatam-prod`. Added in issue 003.
- **v1 release target: Android only (Google Play).** `apps/mobile/app.json` sets
  `platforms: ["android"]`. iOS stays possible through Expo; adding it later needs Sign in with
  Apple (Apple requires it when Google Sign-In is offered), an App Store review pass and an iOS
  backup story for ADR-005.
- `apps/web` is a separate Expo app with only the web platform (`expo export --platform web`). It
  will consume the same screens as `apps/mobile` through `packages/ui`.

Question 3 (backup for phone-only users) is still open. It belongs to the identity/backup issues,
not to toolchain bootstrap.
