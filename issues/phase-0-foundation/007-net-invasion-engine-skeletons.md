# 007 -- Set/point & invasion engine skeletons

## Context
The other two engines from ADR-007, covering badminton/tennis/volleyball (set/point) and
football/basketball/kabaddi (invasion) at launch.

## Tasks
- [ ] Set/point engine: points-to-X, sets, win-by-2 logic, configurable per sport (e.g.
      badminton to 21, volleyball to 25, tennis game/set/match structure).
- [ ] Invasion engine: score events (goal/basket/raid point), periods/halves, running timer,
      configurable per sport.
- [ ] Both zero framework or storage dependency, same pattern as issue 006.

## Passing criteria
- [ ] Unit tests: a full sample match for one sport per engine (e.g. a badminton set to 21, a
      football match with two goals) with hand-verified expected state.
- [ ] Both engines' configuration surface is proven reusable: a second sport in the same family
      (e.g. volleyball after badminton) is added as configuration only, no new engine code, and
      has its own passing test.

## Out of scope
- Tennis's game/set/match nesting can be deferred to a phase-1 cricket-adjacent follow-up if it
  proves structurally different enough from the simpler point-to-X sports -- flag explicitly if
  so, don't silently under-model it.
