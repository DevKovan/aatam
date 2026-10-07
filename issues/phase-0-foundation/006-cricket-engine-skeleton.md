# 006 -- Cricket engine skeleton

## Context
First of the three engines from ADR-007. "Skeleton" here means the state tree, types, and the
core recompute logic -- enough for issue-level correctness, with full edge-case rules (DLS, no-
ball free-hit variants, etc.) explicitly deferred to phase 1's cricket feature issues.

## Tasks
- [ ] Types: Match -> Innings -> Over -> Ball, Team -> Player, matching the structure in
      `docs/architecture-plan.md`.
- [ ] Ball recording: runs, extras (wide/no-ball/bye/leg-bye), wicket types.
- [ ] Derived stats computed on read from the ball tree: batting (runs, balls faced, strike
      rate), bowling (overs, runs conceded, wickets, economy), match (run rate).
- [ ] Zero framework or storage dependency -- pure functions over a state object, per ADR-001.

## Passing criteria
- [ ] Unit tests cover a full sample innings (multiple overs, at least one of each extra type,
      at least one wicket type) with hand-verified expected stats.
- [ ] Integration test: engine output round-trips through `packages/storage` (issue 005) and
      reloads to an identical derived-stats result.

## Out of scope
- DLS, super overs, multi-day/Test-format rules -- flagged as deferred, not silently dropped.
