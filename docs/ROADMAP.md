# Implementation Roadmap

Work is split into **work packages (WPs)** small enough for one coding agent per WP. Each WP
lists its dependencies, the paths it owns (so parallel agents don't collide), and acceptance
criteria. A WP is done only when its acceptance criteria pass in CI.

```
Phase 0  Scaffold ─────────────► WP0
Phase 1  Theory core + Fretboard ► WP1a ─┬─► WP1b ─┬─► WP1c
                                         │         └─► WP1d
                                         └─► WP2 (UI fretboard)
Phase 2  Audio                   ► WP3 ─► WP4 (backing tracks)
Phase 3  Changes + Keys          ► WP5 (analysis) ─► WP6 (Key Finder) , WP7 (Changes Trainer, visual)
Phase 4  Voicings & library      ► WP8 (voicing engine) ─► WP9 (library UI + songs)
Phase 5  Licks                   ► WP10 (fingering) ─► WP11 (lick content) ─► WP12 (lick trainer)
Phase 6  Mic                     ► WP13 (pitch pipeline + tuner) ─► WP14 (graded modes everywhere), WP15 (fret quiz)
Phase 7  Practice system         ► WP16 (storage+SRS+tempo) ─► WP17 (Today planner, Technique, Progress)
```

**Parallel tracks.** Once WP1a's public API types are merged, WP2, WP3, WP5, WP8, and WP10 can
run at the same time. WP13 (mic) depends only on WP0 and can start early as well.

**First usable milestone (M1)**: WP0–WP7. This gives a fretboard explorer, backing tracks for
jazz progressions, a key finder, and the visual Changes Trainer, which covers most of the
"what notes do I play over these changes?" goal before any mic work.

---

### WP0: Project scaffold
- **Owns**: repo root config, `src/app/`, `.github/workflows/`
- **Tasks**: Vite + React + TS strict, Tailwind, ESLint (with the layer rules from ARCHITECTURE
  §4), Prettier, Vitest, Playwright (mobile viewport, pre-installed Chromium),
  `vite-plugin-pwa` manifest and icons, React Router with placeholder tabs, a "Tap to start"
  audio-unlock gate, and a GitHub Pages deploy workflow.
- **Accept**: `npm run lint && npm run typecheck && npm test && npm run build` pass; Lighthouse
  PWA installable; the e2e smoke test opens every tab at 390×844; importing `features/a` from
  `features/b` fails lint.

### WP1a: Theory primitives
- **Owns**: `src/theory/{types,pitch,interval,scale,chord,transpose,index}.ts`
- **Tasks**: data types (ARCHITECTURE §5), spelling, intervals, the scale catalogue (SPEC §2),
  the chord parser and chord tones (SPEC §3), and transpose.
- **Accept**: ≥ 95% line coverage; every parse example in SPEC §3.1 is tested; spelling tests
  for all 12 keys for every scale; no imports from outside `src/theory`.

### WP1b: Fretboard geometry
- **Deps**: WP1a. **Owns**: `src/theory/fretboard.ts`
- **Tasks**: note↔position, CAGED, 3NPS, position windows, pentatonic box numbering, octave
  shapes (SPEC §6).
- **Accept**: snapshot tests of CAGED/3NPS for C and F♯ major; all 5 pentatonic boxes match
  the standard boxes for A minor.

### WP1c: Chord-scale and note tiers
- **Deps**: WP1a. **Owns**: `src/theory/analysis/chordScale.ts`
- **Accept**: every row in SPEC §4.1 is tested; the fallback rule (SPEC §4.2) is tested on
  V/ii, V/V, and V/vi in 3 keys; avoid-note tiers are correct.

### WP1d: Content schemas and loaders
- **Deps**: WP1a. **Owns**: `src/content/`
- **Tasks**: zod schemas, loaders, and the 15 progressions (SPEC §10.1) as JSON.
- **Accept**: every progression parses and realizes in all 12 keys; a schema test runs over
  all content.

### WP2: `<Fretboard>` and Fretboard Explorer (F2)
- **Deps**: WP1b. **Owns**: `src/ui/Fretboard/`, `src/features/explorer/`
- **Accept**: vertical and horizontal orientations, left-handed mode, tier colors plus labels,
  position lock, tap events; renders 24 frets × 6 strings with 40 markers in < 4 ms; Playwright
  test picks "A dorian, CAGED 3" and sees the correct markers.

### WP3: Audio engine core
- **Deps**: WP0, WP1a. **Owns**: `src/audio/{engine,instruments,transport,metronome,playback}.ts`, `public/samples/`
- **Accept**: metronome drift < 2 ms over 5 minutes in a headless timing test; swing ratio
  works; `playNotes` drives a visual cursor through `Tone.Draw`; total sample payload ≤ 15 MB;
  works after the iOS unlock gate.

### WP4: Backing-track generator
- **Deps**: WP3, WP1c. **Owns**: `src/audio/backing/`, `src/content/comping/`, `src/ui/TransportBar/`, `src/ui/ChordChart/`
- **Accept**: unit tests show the walking bass hits the chord root or 3rd on beat 1 and an
  approach note on beat 4 that resolves by half or whole step; it is deterministic for a given
  seed; swing, bossa, ballad, and shuffle styles; per-part mute; loop and transpose.

### WP5: Harmonic analysis
- **Deps**: WP1c. **Owns**: `src/theory/analysis/{key,roman}.ts`
- **Accept**: every golden example in SPEC §9 passes; analyzing a 64-chord song takes < 20 ms.

### WP6: Key Finder screen (F5, without the drill)
- **Deps**: WP5, WP2, WP4. **Owns**: `src/features/keyfinder/`, `src/ui/ChordInput/`
- **Accept**: text and picker input; key-region bands; roman numerals with ii–V brackets;
  per-chord scales that open the fretboard; "Practice this" deep-links to the Changes Trainer
  via the URL (not via an import).

### WP7: Changes Trainer, non-mic modes (F4 Visualize, Scale-of-the-moment, Pentatonic switch)
- **Deps**: WP2, WP4, WP5. **Owns**: `src/features/changes/`
- **Accept**: the fretboard updates on the downbeat in sync with the audio (±20 ms visual);
  the next-chord ghost preview appears on beat 4; position lock works; runs the jazz blues at
  200 BPM without dropped frames on a mid-range phone.

**→ Milestone M1.**

### WP8: Voicing engine
- **Deps**: WP1b. **Owns**: `src/theory/voicing.ts`, `src/content/voicings/`
- **Accept**: drop 2 and drop 3 inversions correct for maj7, m7, 7, and m7b5 on all listed
  string sets; the playability filter respects `maxStretch`; `voiceLead` on ii–V–I yields total
  motion ≤ 4 semitones for drop 2 on strings 4-3-2-1.

### WP9: Voicings and progressions library, songs (F6)
- **Deps**: WP8, WP4. **Owns**: `src/features/voicings/`, `src/features/songs/`, `src/ui/ChordDiagram/`
- **Accept**: browse, filter, strum; the voice-leading view draws lines between voices; the
  song editor saves to storage (a temporary localStorage adapter until WP16) and songs open in
  Changes and Key Finder.

### WP10: Fingering optimizer
- **Deps**: WP1b. **Owns**: `src/theory/fingering.ts`
- **Accept**: golden tests from SPEC §7; output never exceeds `maxStretch`; a 32-note line
  finishes in < 5 ms.

### WP11: Lick content
- **Deps**: WP1d, WP1c. **Owns**: `src/content/licks/`
- **Accept**: 60 original licks per SPEC §10.3; the content test checks that strong-beat notes
  are never `avoid`; each lick has concepts, difficulty, and context.

### WP12: Lick Library and Trainer (F7, steps 1, 4, 5 without the mic)
- **Deps**: WP10, WP11, WP3. **Owns**: `src/features/licks/`, `src/ui/TabView/`
- **Accept**: tab plus fretboard cursor in sync; 50–100% speed; A/B loop; transpose through 12
  keys with re-fingering; "My Licks" entry by tapping notes.

### WP13: Pitch pipeline and Tuner
- **Deps**: WP0. **Owns**: `src/pitch/`, `src/features/tuner/`, `src/ui/PitchMeter/`
- **Accept**: offline tests on recorded guitar WAV fixtures (single notes E2–E6) detect the
  correct MIDI note ≥ 98% of the time within 50 ms of onset; tuner accurate to ±3 cents on
  synthetic sines; calibration flow stores latency; no audio leaves the device.

### WP14: Mic-graded modes
- **Deps**: WP13, WP7, WP12. **Owns**: `src/pitch/grader.ts`, graded sub-modes inside `features/changes` and `features/licks` (coordinate via a small `useGradedExercise` hook in `src/practice/`)
- **Accept**: target-note game, guide-tone line, arpeggio mode, and lick play-along report
  per-beat results; the post-loop timeline view is shown; grader unit tests cover timing
  windows and tier scoring.

### WP15: Fret Quiz (F3)
- **Deps**: WP13, WP2. **Owns**: `src/features/fretquiz/`
- **Accept**: Name-it, Find-it, and Degree modes; response-time heatmap.

### WP16: Storage, SRS, and tempo ladders
- **Deps**: WP0. **Owns**: `src/storage/`, `src/practice/{srs,tempoLadder,scoring}.ts`
- **Accept**: Dexie schema v1 with a migration test; export → wipe → import round-trips;
  SM-2 unit tests; tempo-ladder rules from ARCHITECTURE §6.4; replaces the temporary song
  adapter from WP9.

### WP17: Today planner, Technique, Progress, Ear (F1, F8, F9, F10)
- **Deps**: WP16, WP14, WP15. **Owns**: `src/practice/planner.ts`, `src/features/{today,technique,progress,ear}/`
- **Accept**: a 30-minute plan follows the weighting and pulls due items first; technique
  drills with the tempo ladder and an evenness metric; progress charts; a basic ear drill.

---

## Definition of done (every WP)

1. Lint, typecheck, unit tests, and build pass; e2e smoke passes.
2. New public functions have unit tests, and theory changes include SPEC golden tests.
3. No layer-rule violations; no edits outside the WP's owned paths without noting them in the
   PR description.
4. Docs are updated if a data model or public API changed.
5. Manually checked on a 390×844 viewport (screenshot attached to the PR).
