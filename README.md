# aatam

Local-first, multi-sport live scoring app for India -- cricket, football, kabaddi, badminton,
tennis, volleyball and basketball at launch. Mobile-first. See `plan/VISION.md` for the product
vision and `docs/architecture-plan.md` for the full architecture and ADRs.

## Working on this repo

This repo is built with AI coding agents as the primary implementers, following the process in
`CLAUDE.md`, `issues/README.md`, and `.claude/`. Start here:

1. Read `CLAUDE.md` -- rules every session follows.
2. Read `docs/architecture-plan.md` -- the ADRs are final; don't relitigate them.
3. Read `plan/ROADMAP.md` and `issues/PROGRESS.md` -- see what's next.
4. Use the `work-issue` skill to pick up an issue end to end.

## Repo layout

- `apps/mobile`, `apps/web`, `apps/marketing` -- the three surfaces (see architecture plan).
- `packages/core-engines`, `identity`, `i18n`, `ui`, `storage` -- shared logic.
- `plan/` -- vision, roadmap, changelog.
- `issues/` -- the backlog, phase by phase.
- `.claude/` -- agent and skill definitions driving the AI-orchestrated workflow.

## Setup

Requires Node 24 (`.nvmrc`) and pnpm via Corepack (`corepack enable`; the version is pinned in
`package.json` `packageManager`).

```sh
pnpm install
pnpm lint && pnpm typecheck && pnpm test && pnpm knip   # the four CI gates
pnpm build                                              # marketing (next build) + web/mobile (expo export)
pnpm --filter @aatam/mobile start                       # Expo dev server (Android)
pnpm --filter @aatam/web start                          # web build in the browser
pnpm --filter @aatam/marketing dev                      # Next.js public site
```

Stack: pnpm workspaces + Turborepo, Expo SDK 57 (Android-only for v1), Next.js 16, TypeScript 6,
Vitest, ESLint 10 + typescript-eslint, knip. See ADR amendment A1 in `docs/architecture-plan.md`.
