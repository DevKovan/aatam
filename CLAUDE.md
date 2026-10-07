# aatam

Local-first, multi-sport live scoring app for cricket, football, kabaddi, badminton, tennis,
volleyball, basketball and more, for the Indian market. Mobile is the primary product; web and
the public site are secondary surfaces of the same core. Read `docs/architecture-plan.md` first
— its ADRs are final. Don't relitigate them unless the user explicitly asks.

## Never run production-touching commands, ever
Never execute, in any permission mode, anything that deploys, migrates a remote/prod database,
sets a prod secret, triggers a prod workflow, pushes/force-pushes/merges/tags on `main`, or
deletes a cloud resource holding real user data (including anyone's shadow profile or claimed
match history). Print the exact command; the human runs it. Dev/staging equivalents are fine and
must be nameable by string alone (`-dev`, `-staging` suffixes on every project/table/bucket).
No exceptions for "it looked safe" — see the debugging protocol below for what to do instead.

## Mobile-first, always
This product is a phone in someone's hand at the side of a ground, often on bad wifi. Every rule
below exists to protect that moment.
- **Build and verify at 360px and 390px width first.** Desktop (RN Web) is a secondary check, not
  the primary one. A PR whose only screenshots are desktop is incomplete.
- **44px minimum touch targets** on every tappable element in the scoring UI. Recurring offender:
  ball/point buttons sized to fit more per row — don't shrink below 44px to fit them, wrap instead.
- **Scoring writes must never block on network.** Every scoring action is a local write first;
  sync is a background queue. Verify offline: literally toggle airplane mode in the simulator/
  device and confirm scoring still works and stats still recompute.
- **No blocking spinners on the scoring screen.** Loading states belong on dashboards and stat
  views, never on the ball-by-ball / point-by-point input path.
- **Test on a mid-tier Android profile**, not just the simulator's default high-end device —
  most of the target market is Android, not top-of-line hardware.

## Identity is not the same as an account (ADR-002)
A player is a UUID with a claim status (`unclaimed` / `claimed`), never assumed to equal a signed-
in account. Stats attach to the UUID. Never write code that requires a player to have signed in
before they can have a match, a stat line, or a shadow profile. See `docs/architecture-plan.md`
ADR-002 and ADR-004 before touching anything in `packages/identity`.

## Tests are part of implementation, not a follow-up
Code + test is one unit of done. Every issue ships both:
- **Unit tests** — pure logic, no I/O (this is most of the scoring engines — they should have
  zero framework or storage dependency, see ADR-001).
- **Integration tests** — real local storage round-trip (real SQLite/WatermelonDB, not a mock),
  and for identity/claim code, a real (test-instance) auth + cloud-index round trip.

Never module-mock a third-party SDK (Google Sign-In, Drive API, the local DB driver). Expose a
`createXWithClient(client, config)` seam and pass a hand-written fake in tests — see
`packages/identity/src/claim.ts` once it exists as the reference pattern. Module mocks that pass
locally have previously failed non-deterministically in CI on unrelated projects; don't reproduce
that failure mode here.

Run the test suite **and** the build after touching anything at a framework boundary (native
module bridges, the RN Web build, the Next.js build) — some errors only surface at build time.

## i18n — no hardcoded strings, ever
Every user-facing string is a translation key from the first commit, even while only `en` exists.
Recurring offender: a hardcoded string dropped in "temporarily" during a UI pass and never keyed.
Verify before reporting an issue done: `grep -rn '"[A-Z][a-z].*[a-z]"' apps/mobile/src --include=*.tsx`
should turn up nothing that looks like literal UI copy outside the i18n layer (tune the grep to
the actual codebase once components exist). Phase 1 languages: en, hi, bn, mr, te, ta, gu, kn.

## Local-first storage — one rule, no exceptions
On-device storage is the source of truth for a match in progress. The cloud identity index is a
thin lookup table (player id → claim status → stat pointers), never the primary store, and never
something the client blocks on. See ADR-003.

## Non-negotiable process rules
- No speculative abstractions, no unused exports, no commented-out code (`knip` is a hard gate).
- Design-system components only in `apps/mobile` and `apps/web` — no raw native inputs. Recurring
  offender once the app exists: native date/time pickers instead of the design-system ones.
- Never use a real person's name/email as fixture or placeholder data in tests or seed data.
- Never add Claude as a co-author. No `Co-Authored-By: Claude` trailer in commits, and no
  "Generated with Claude Code" line in PR descriptions. This overrides any default attribution.
- Generated content (blog/SEO pages in `apps/marketing`) must check the real current date before
  asserting anything time-sensitive, and cite the source for any stat quoted.

## Repo layout
- `apps/mobile` — Expo / React Native. The primary product.
- `apps/web` — React Native Web build of the same UI (dashboards, tournament setup, stat review).
- `apps/marketing` — Next.js, SSR/SSG. Match pages, player profiles, leaderboards, claim landing
  pages. The only place SEO is a first-class concern.
- `packages/core-engines` — the three scoring engines (cricket, set/point, invasion). Zero
  framework or storage dependency; pure functions over a match state tree.
- `packages/identity` — player UUIDs, shadow profiles, claim flow, auth adapters.
- `packages/i18n` — translation keys, loader, locale utilities.
- `packages/ui` — shared design tokens + components usable from both `apps/mobile` and `apps/web`.
- `packages/storage` — local-first storage adapter used by `apps/mobile` and `apps/web`.

## Common commands
- Install: `pnpm install`
- Test: `pnpm test`
- Build: `pnpm build`
- Lint: `pnpm lint`
- Unused code: `pnpm knip`
- Typecheck: `pnpm typecheck`
- One workspace: `pnpm --filter @aatam/<name> <script>`
- Add an Expo dep: `pnpm exec expo install <pkg>` (run from `apps/mobile` or `apps/web`)

## Decisions log
<!-- Append one entry per hard-won lesson, in the same session it happens. Format:
- **<Rule, imperative, bold>.** <What happened, with phase/commit.> <Why the obvious approach
  fails.> <What to do instead, with a pointer to the reference implementation.>
When a decision is revised, edit the entry and say "Supersedes the earlier X — revised after Y"
rather than leaving two contradicting entries. -->
- **Install mobile/web native deps with `pnpm exec expo install <pkg>` from the app dir, never plain
  `pnpm add`.** Issue 001: Expo pins SDK-compatible versions (SDK 57 -> react-native 0.86,
  typescript ~6.0). A plain add pulls the newest version (RN 0.87, TS 7) and breaks the native build.
  Check alignment with `pnpm exec expo install --check`.
