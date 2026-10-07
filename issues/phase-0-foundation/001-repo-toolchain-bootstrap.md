# 001 -- Repo & toolchain bootstrap

## Context
Nothing exists yet. This issue creates the monorepo shell described in ADR-001 and fills in the
tool choices `docs/architecture-plan.md` left open (local DB, cloud index, monorepo tool).

## Tasks
- [x] Resolve the open tool choices via `grill-me` (local DB, cloud identity index, monorepo
      tool) -- don't guess; recommendations are already in the architecture plan's "Clarifying
      questions" section.
- [x] Scaffold `apps/mobile` (Expo), `apps/web` (React Native Web), `apps/marketing` (Next.js),
      and empty `packages/core-engines`, `packages/identity`, `packages/i18n`, `packages/ui`,
      `packages/storage`.
- [x] Wire lint, typecheck, test runner, unused-code checker as workspace scripts; fill in
      `CLAUDE.md`'s and `issues/README.md`'s "Common commands" / "QA commands" sections with the
      real command names.
- [x] CI workflow running exactly those scripts against every PR.
- [x] `.claude/settings.json` allowlist matches the real command names chosen here.

## Passing criteria
- [x] `pnpm install && pnpm build` succeeds from a clean checkout.
- [x] All four gates (lint, typecheck, test, unused-code) run and pass on the empty scaffold.
- [ ] CI runs the same four commands and passes on a throwaway PR.
- [x] `docs/architecture-plan.md`'s open questions are resolved and recorded as ADR amendments
      (append, don't silently edit the original ADR text).

## Test requirements
- N/A for this issue's own code (pure scaffolding) -- but the scaffold must make later issues'
  unit + integration tests runnable, which is itself the acceptance bar.

## Out of scope
- Any actual scoring logic, identity logic, or UI.
