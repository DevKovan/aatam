# 004 -- i18n pipeline

## Context
Implements ADR-008. Must exist before any UI issue, per the roadmap's standing policy of no
literal strings.

## Tasks
- [ ] `packages/i18n`: key lookup, locale loader, fallback-to-English behaviour for missing keys
      in partially-translated locales.
- [ ] `en.json` seeded with a handful of representative keys (nav labels, a sample scoring
      action) as the reference format for later phases.
- [ ] A lint rule or grep-based CI check that flags literal string-looking JSX text outside the
      i18n layer (best-effort at this stage; tighten once real components exist).
- [ ] Locale detection: device locale on mobile, `Accept-Language`/URL locale on `apps/marketing`.

## Passing criteria
- [ ] Missing-key fallback verified by a test (request a key only present in `en`, from a
      non-`en` locale, get the English string back, not a crash or a raw key).
- [ ] The literal-string CI check runs and produces zero findings against the current (empty)
      component tree, and is proven to catch a deliberately introduced literal string in a test
      fixture.

## Out of scope
- Actual translations beyond `en` -- that's a phase-1 content task, not an engineering issue.
