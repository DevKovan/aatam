# 002 -- Name & registry check

## Context
Playbook lesson: a project name chosen without checking registry/store availability caused a
large forced rename late in a prior project. Do this in phase 0, not at launch.

## Tasks
- [ ] Check `aatam` (and 2-3 backup names) against: npm/pnpm package scope availability,
      Play Store app name/package-id collisions, App Store name collisions, a plain web search
      for existing products with the same name.
- [ ] Check domain availability for the public site (`apps/marketing`).
- [ ] Record the decision and the evidence (what was checked, when) in `CLAUDE.md`'s decisions
      log.

## Passing criteria
- [ ] Final name confirmed with the human (grill-me), with evidence cited for why it's clear.
- [ ] Any package/app-id renames from a placeholder name are applied everywhere before more code
      is written on top of it.

## Out of scope
- Actually registering the domain or reserving the store listing (human action).
