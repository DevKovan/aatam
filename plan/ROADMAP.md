# Roadmap

Phases are ordered by dependency, not by calendar. A phase doesn't start until the ones it
depends on are DONE. See `issues/PROGRESS.md` for live status.

## Standing policies (every phase inherits these)

- Examples/docs ship in the same phase as the feature they document, not as a later catch-up.
- Every UI issue's acceptance criteria include mobile-viewport verification (360-390px) before
  desktop, per ADR-009.
- Every scoring-path issue's acceptance criteria include an explicit offline test.
- Issue files are permanent history -- never deleted, only superseded.
- A phase that turns out to hold two unrelated concerns gets split into two numbered phases,
  with a commit that says so.
- Deferred is not dropped: a consciously-skipped feature gets one line in the phase's issue file
  saying why, so it isn't mistaken for an oversight later.

## Phase 0 -- Foundation
Monorepo, toolchain, identity/claim core, storage core, i18n pipeline, and the three scoring
engines' skeletons (state tree + types, not full rules yet). Nothing user-facing ships. See
`issues/phase-0-foundation/`.

## Phase 1 -- Launch sports (cricket engine + set/point engine + invasion engine, full rules)
- Cricket (own engine)
- Badminton, Tennis, Volleyball (set/point engine)
- Football, Basketball, Kabaddi (invasion engine)
Plus: claim flow end-to-end, Google Sign-In, Drive backup, phase 1 languages (en, hi, bn, mr, te,
ta, gu, kn), `apps/marketing` match/profile pages.

## Phase 2 -- Fast-follow sports (same engines, near-zero new logic)
- Table Tennis (set/point engine)
- Hockey (invasion engine)
- Handball (invasion engine)
Plus: phase 2 languages (ml, pa, or, as).

## Phase 3 -- New engines required
- Kho-Kho (own engine -- raid/chase timing)
- Wrestling / Kushti (own engine -- bout/points-based)

## Phase 4 -- Casual games (scoreboard only, no deep stats)
- Carrom, Pickleball -- live scoreboard only
- Chess -- result/rating tracking only, no live scoring

## Explicitly out of scope for the roadmap above
- Any sport not listed here needs a new phase entry with its own grill-me pass before work
  starts, not an ad hoc addition to an existing phase.
