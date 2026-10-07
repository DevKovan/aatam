# Vision

aatam is a local-first, multi-sport scoring app for India: cricket, football, kabaddi,
badminton, tennis, volleyball and basketball at launch, more sports over time. It exists to give
grassroots and amateur players the same live scoring, stats and history that CricHeroes gives
cricket players -- but across every sport they actually play, in their own language, and without
losing history just because they installed the app after their teammates did.

## What "done" looks like for v1

- A team can score a full match for any of the seven launch sports, fully offline, on a mid-tier
  Android phone.
- A player who's never opened the app before can install it, sign in with Google, and see every
  match they were tagged in by teammates who already use it.
- The app speaks English, Hindi, Bengali, Marathi, Telugu, Tamil, Gujarati and Kannada from day
  one.
- A shared match is genuinely shareable -- a WhatsApp link unfurls with a real preview, and the
  page it opens looks right whether or not the recipient has the app.
- Every claimed player's data lives in their own Google Drive, not in a platform database they
  don't control.

## What v1 deliberately does not attempt

- Sports needing their own scoring engine (kho-kho, wrestling) -- phase 3.
- Casual/backyard games with deep stats (carrom, pickleball, chess) -- phase 4, scoreboard-only.
- Languages beyond the phase 1 set -- phase 2 adds Malayalam, Punjabi, Odia, Assamese.
- A public API or third-party integrations.
- Monetisation of any kind.

## Non-negotiable product properties

1. **Local-first.** Scoring never waits on network.
2. **Identity before account.** A player's stats exist before they have an account; claiming them
   later is safe and, for the common case, automatic.
3. **Own data, own Drive.** A claimed profile's backup belongs to the person, not the platform.
4. **Mobile is the product.** Web and the public site are real but secondary surfaces.
