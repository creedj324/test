# Agent Guide

This repo is **Fretwise**, a phone-first PWA guitar teacher. Read `docs/ARCHITECTURE.md`
first, then the work package you were assigned in `docs/ROADMAP.md`. Musical correctness is
defined by `docs/THEORY_SPEC.md`; when code and spec disagree, the spec wins unless you update
it on purpose and say so.

## Commands (after WP0 lands)

```
npm run dev         # local dev server
npm run lint        # ESLint, including layer-boundary rules
npm run typecheck   # tsc --noEmit
npm test            # Vitest
npm run e2e         # Playwright, mobile viewport
npm run build       # production build + PWA service worker
```

Run lint, typecheck, and test before every commit.

## Rules

- **Layering is enforced.** `theory` is pure TypeScript with no DOM, audio, React, or IO.
  Features never import other features. Shared code moves down a layer.
  See ARCHITECTURE §4.
- Import theory only through `src/theory/index.ts`.
- Notes are always **spelled** (letter + accidental). Never turn a pitch class into a name
  without key or chord context.
- Audio and visual timing come from the Tone.js clock (`Tone.Draw`), never from
  `setTimeout`/`setInterval`.
- All content JSON is validated by zod schemas in `src/content/schemas.ts`. All licks are
  original; do not transcribe copyrighted solos.
- Persist only through `src/storage` repositories.
- Microphone audio never leaves the device and is never written to storage.
- Respect `Settings.maxStretchFrets` anywhere fingerings or voicings are produced.
- Mobile first: design at 390×844, touch targets ≥ 44 px, and never use color alone to carry
  meaning.
- Stay inside the paths your work package owns. If you must change a shared type, update
  ARCHITECTURE §5 in the same commit.
