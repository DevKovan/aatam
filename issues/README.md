# Working an issue

1. Pick the named issue, or the first `TODO` in `PROGRESS.md` whose `Depends on` is already
   `DONE`. Never skip ahead of a dependency.
2. Mark it `IN PROGRESS` in `PROGRESS.md`.
3. Implement exactly that issue's acceptance criteria -- see `.claude/agents/implementor.md`.
4. Review -- see `.claude/agents/reviewer.md`. `CHANGES NEEDED` -> back to implementor with the
   findings verbatim -> re-review. Max 3 rounds, then stop and report to the human.
5. QA -- see `.claude/agents/qa.md`. Failure -> implementor -> re-QA. Max 3 rounds, then stop.
6. Close out: every checkbox ticked, `PROGRESS.md` updated to `DONE` with date and a one-line
   note (deviations, follow-ups, anything a future phase should know).
7. One commit, issue ID in the subject. Never push (see `.claude/skills/git-discipline`).

## QA commands
- Lint: `pnpm lint`
- Typecheck: `pnpm typecheck`
- Test: `pnpm test`
- Unused code: `pnpm knip`
- Build: `pnpm build`

## Standing conventions (apply retroactively when introduced mid-plan)
- Mobile-viewport verification (360-390px) before desktop, on every UI issue.
- Offline test, on every scoring-path issue.
- i18n keys only, no literal UI strings, on every issue that touches `apps/mobile` or `apps/web`.
