# Fretwise — a personal guitar teacher app

A **web app** (an installable Progressive Web App: open it in your phone's browser and add it
to your home screen; no app store), phone-first and offline-capable. It's a practice app for
an **intermediate guitarist** who knows the pentatonic/blues scales and the major scale, and
wants to:

1. **Master the fretboard**: know every note, scale, and arpeggio in every position.
2. **Play the changes**: hit the right notes over each chord and through chord changes.
3. **Learn jazz harmony**: common jazz progressions, guitar voicings, and voice leading.
4. **Know the key**: analyze a chord progression, find the key(s), and know what to play.
5. **Build a lick vocabulary**: a library of licks you can call up on demand, in any key.
6. **Improve dexterity**: graded technique drills with tempo tracking.
7. **Write songs**: turn chord knowledge into songwriting, including how to write a chorus or
   bridge that contrasts with a verse but still belongs to the same song.

The app can listen through the phone's microphone (optional, with no setup) to tell you where
you missed the changes. It generates its own backing tracks (bass, comping, drums) for any
progression.

## Status

**Planning.** Nothing is implemented yet. The plan lives in `docs/`:

| Document | What it covers |
| --- | --- |
| [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) | Platform choice, tech stack, module layout, data models, audio and pitch-detection design |
| [`docs/THEORY_SPEC.md`](docs/THEORY_SPEC.md) | The music-theory engine: chord-scale rules, key detection, voicings, fingering, content lists |
| [`docs/FEATURES.md`](docs/FEATURES.md) | Each screen and practice mode, from the player's point of view |
| [`docs/ROADMAP.md`](docs/ROADMAP.md) | Phased work packages with dependencies and acceptance criteria, sized for agents |
| [`docs/EXPERT_REVIEW.md`](docs/EXPERT_REVIEW.md) | Expert review of the theory and teaching plan, the sources it checked, and what changed |
| [`CLAUDE.md`](CLAUDE.md) | Rules and conventions for the coding agents that implement the plan |
