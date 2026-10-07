# 005 -- Local-first storage core + offline conflict spike

## Context
Implements ADR-003. This issue also resolves the "offline conflict resolution" risk flagged in
`docs/architecture-plan.md` -- don't leave it as a TODO for a later feature phase.

## Tasks
- [ ] `packages/storage`: adapter over the local DB chosen in issue 001, exposing a minimal
      match/player/stat read-write interface used by `core-engines` and `identity`.
- [ ] Background sync queue: local writes never block on network; queued operations retried on
      connectivity return.
- [ ] Conflict resolution design spike: write up the chosen strategy (e.g. last-write-wins per
      field with a vector clock, or CRDTs, or operation log replay) as a short design note in
      `docs/architecture-plan.md` (new ADR-010), with the trade-off explicitly stated -- don't
      leave this implicit in code.
- [ ] Implement the chosen strategy for the simplest real case: two devices scoring the same
      match's different overs/periods (should never actually conflict) vs. the same ball/point
      (should, and must resolve deterministically).

## Passing criteria
- [ ] ADR-010 written and reviewed via grill-me if the trade-off has real product implications
      (e.g. "last write wins" could silently drop a scorer's correction).
- [ ] Integration test: simulate two local stores diverging offline, then syncing, and assert the
      documented resolution behaviour.
- [ ] A scoring write succeeds with the local DB layer put into airplane-mode-equivalent (network
      calls stubbed to fail/hang) -- proves the "never blocks on network" rule in `CLAUDE.md`.

## Out of scope
- The cloud identity index itself (issue 003 covers the identity side; this issue only needs a
  storage interface identity can call against).
