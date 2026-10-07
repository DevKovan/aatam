# 008 -- Phase 0 wrap-up

## Context
Standard end-of-phase issue per the playbook: full clean-checkout verification, docs, changelog.

## Tasks
- [ ] Clean-checkout run: `rm -rf node_modules dist packages/*/dist apps/*/dist && pnpm install
      && pnpm build && <all four gates>` -- fix anything that only breaks from clean.
- [ ] `plan/CHANGELOG.md` phase 0 entry: what shipped, key decisions (link ADRs 001-010), what
      was deferred and why, anything discovered and fixed.
- [ ] `CLAUDE.md` decisions log updated with any hard-won lessons from phase 0.
- [ ] `README.md` reflects the real repo layout and setup steps (not aspirational).

## Passing criteria
- [ ] Every phase-0 issue is `DONE` in `PROGRESS.md`.
- [ ] Clean-checkout build + all gates pass with zero manual intervention.
- [ ] Changelog entry exists and cites real file paths, not descriptions.
