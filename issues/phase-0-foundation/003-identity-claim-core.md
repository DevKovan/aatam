# 003 -- Identity & claim core

## Context
Implements ADR-002 and ADR-004 in `packages/identity`. This is the highest-risk piece of the
whole system -- get the shape right here and every later feature (invites, Drive backup, the
marketing site's claim landing page) builds on it cleanly.

## Tasks
- [ ] Player record: UUID, `claim_status`, linked identifiers (email/phone, each with a
      `verified` flag), stat pointers.
- [ ] Shadow profile creation: given an email/phone and a match reference, create or update a
      player record without requiring an account.
- [ ] Verified claim: given an authenticated identity (Gmail or OTP-confirmed phone), find and
      merge any shadow profile(s) under that identifier into the caller's account. Auto-merge, no
      review step.
- [ ] Suggested claim: given a name + team + date range, return possible matches *scoped to the
      requesting user only* -- never exposed to anyone else. Merge only on explicit confirmation.
- [ ] Invite/claim token: generate a token bound to one shadow profile; resolving a token returns
      that profile's id for the sign-in-then-merge flow.
- [ ] Auth SDK access goes through a `createXWithClient(client, config)` seam (see `CLAUDE.md`) --
      no direct/global SDK calls inside `packages/identity`.

## Passing criteria
- [ ] Unit tests cover: shadow creation, verified auto-merge, suggested-match scoping (a
      suggestion for user A must not be queryable by user B), token resolution, and the
      not-yet-claimed vs already-claimed edge case (double sign-in from two devices).
- [ ] Integration test: real local-DB round trip for shadow profile creation and merge (using the
      storage adapter from issue 005, or a stub matching its interface if 005 isn't done first).
- [ ] No test module-mocks an auth SDK; a hand-written fake client is used via the DI seam.

## Test requirements
- Unit: every branch above, plus the abuse case (suggested match attempted without confirmation
  must never merge).
- Integration: real storage round-trip for create -> shadow -> claim -> merged state.

## Out of scope
- The actual Google Sign-In / Drive backup wiring (later phase-1 issue) -- this issue builds the
  logic against the DI seam only, verified with a fake client.
- UI for any of these flows.
