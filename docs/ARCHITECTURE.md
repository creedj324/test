# Software Architecture

## 1. Goals and constraints

| Constraint | Consequence |
| --- | --- |
| Runs on **my phone** (iOS or Android), used by one person | No app store, no accounts, no server. |
| Used in a practice room, often with no signal | **Offline-first**: everything works after the first load. |
| Must **listen** to the guitar | Microphone access plus real-time monophonic pitch detection. |
| Must **play** backing tracks and examples | A low-latency, sample-accurate audio scheduler. |
| Built by AI coding agents working in parallel | Strict module boundaries, a pure and heavily tested theory core, data-driven content. |
| Limited dexterity is a design input | Fingering engine respects a max-stretch setting; every drill has a tempo ladder starting slow. |

## 2. Platform decision

**Progressive Web App (PWA): React + TypeScript + Vite.** Install it to the home screen from
the browser.

Why not a native app (Swift/Kotlin) or React Native / Expo:

- One codebase runs on any phone, tablet, or laptop, with no app store review or developer
  account.
- Web Audio, `getUserMedia` (microphone), AudioWorklet, IndexedDB, Wake Lock, and Service
  Workers are all supported in current iOS Safari and Android Chrome. These are all the
  platform features the app needs.
- Hosting is free and simple (GitHub Pages).
- If native ever becomes necessary (for example, lower mic latency), the `theory`, `content`,
  `practice`, and `storage` layers are framework-free TypeScript and can move to React Native
  unchanged. Only `ui`/`features` and the audio adapters would be rewritten.

Known PWA caveats, with mitigations:

- iOS needs a user gesture before audio can start. The app shell has a "Tap to start" gate
  that resumes the `AudioContext`.
- iOS may evict storage for sites that aren't installed. Ask the user to install to the home
  screen, request `navigator.storage.persist()`, and offer JSON export/import of progress.
- Mic latency varies by device. Run a latency calibration step and widen grading windows.

## 3. Tech stack

| Concern | Choice | Notes |
| --- | --- | --- |
| Language | TypeScript (`strict: true`) | |
| UI | React 18 + React Router | Function components and hooks only. |
| Build | Vite + `vite-plugin-pwa` (Workbox) | Precaches the app shell, content JSON, and audio samples. |
| Styling | Tailwind CSS | Mobile-first, dark theme by default (dim practice rooms). |
| App state | Zustand | One small store per feature, plus a global settings store. |
| Persistence | IndexedDB via Dexie | Accessed only through `src/storage` repositories. |
| Theory primitives | `tonal` (wrapped) | Note/interval/chord-symbol parsing. Wrapped behind `src/theory` so it can be replaced. |
| Audio scheduling & synthesis | Tone.js | Transport with swing, `Sampler` for guitar/piano/bass, synthesized drums. |
| Pitch detection | `pitchy` (McLeod Pitch Method) in an AudioWorklet | Monophonic only. |
| Rendering | SVG (fretboard, chord diagrams, tab) | Resolution-independent; easy hit-testing for taps. |
| Validation | `zod` | Schemas for all content JSON and for import/export. |
| Unit tests | Vitest | The theory engine must have very high coverage. |
| E2E tests | Playwright on a mobile viewport | Uses the pre-installed Chromium. |
| Lint/format | ESLint (with `import/no-restricted-paths`) + Prettier | Enforces the layer rules in section 4. |
| CI/CD | GitHub Actions → GitHub Pages | Lint, typecheck, test, build, and deploy on push to `main`. |

## 4. Layered module architecture

```
┌────────────────────────────────────────────────────────────────────────┐
│ app/        shell, routing, audio-unlock gate, settings, navigation    │
├────────────────────────────────────────────────────────────────────────┤
│ features/   one folder per screen; features never import each other    │
│   explorer · changes · keyfinder · voicings · licks · technique ·      │
│   fretquiz · ear · session · progress · songs · tuner                  │
├────────────────────────────────────────────────────────────────────────┤
│ ui/         reusable presentational components                         │
│   Fretboard · ChordDiagram · TabView · TransportBar · ChordChart ·     │
│   NoteBadge · TempoControl · PitchMeter                                │
├──────────────┬──────────────┬───────────────┬───────────────┬──────────┤
│ audio/       │ pitch/       │ practice/     │ storage/      │ content/ │
│ engine,      │ mic, worklet,│ SRS, tempo    │ Dexie db +    │ JSON +   │
│ instruments, │ note events, │ ladders,      │ repositories, │ loaders, │
│ backing gen  │ grading      │ planner,score │ export/import │ schemas  │
├──────────────┴──────────────┴───────────────┴───────────────┴──────────┤
│ theory/     PURE TypeScript: no DOM, no audio, no React, no IO         │
│   pitch · interval · scale · chord · voicing · fretboard · fingering · │
│   analysis (key detection, roman numerals, chord-scale) · transpose    │
└────────────────────────────────────────────────────────────────────────┘
```

**Dependency rules.** CI enforces these with ESLint `import/no-restricted-paths`.

- `theory` imports nothing from the app.
- `content` imports only `theory` (for types and validation).
- `audio`, `pitch`, `practice`, `storage` may import `theory` and `content`, but not each other.
  The one exception: `practice` may import `pitch` grading types.
- `ui` may import `theory` types. It never touches `audio`, `pitch`, or `storage` directly; it
  receives data and callbacks as props.
- `features` may import anything below them. **A feature never imports another feature.**
  Anything shared moves down into `ui`, `practice`, etc.
- `app` wires features into routes.

These rules let separate agents build separate features at the same time without merge
conflicts, and keep the theory engine fully unit-testable.

### Directory layout

```
/
├─ CLAUDE.md
├─ docs/
├─ public/
│  ├─ samples/           guitar, piano, upright-bass samples (small, compressed .ogg/.m4a)
│  └─ icons/             PWA icons
├─ src/
│  ├─ theory/
│  │  ├─ pitch.ts         PitchClass, Note, MIDI, enharmonic spelling
│  │  ├─ interval.ts
│  │  ├─ scale.ts         scale/mode catalogue + spelled scale generation
│  │  ├─ chord.ts         chord symbol parse/format, chord tones, tensions, avoid notes
│  │  ├─ fretboard.ts     tuning, (string,fret) ↔ note, position/CAGED/3NPS maps
│  │  ├─ fingering.ts     cost-based fingering optimizer (stretch-aware)
│  │  ├─ voicing.ts       voicing generators (shell, drop2, drop3, rootless), voice leading
│  │  ├─ analysis/
│  │  │  ├─ key.ts        key detection + key-region segmentation
│  │  │  ├─ roman.ts      roman-numeral & function labelling
│  │  │  └─ chordScale.ts per-chord scale suggestions with note-tier colouring
│  │  ├─ transpose.ts
│  │  └─ index.ts         public API (the ONLY import path for other layers)
│  ├─ content/
│  │  ├─ schemas.ts       zod schemas
│  │  ├─ progressions/*.json
│  │  ├─ voicings/*.json   curated voicing shapes (supplement generated ones)
│  │  ├─ licks/*.json
│  │  ├─ technique/*.json
│  │  ├─ comping/*.json    rhythm patterns
│  │  └─ index.ts          typed loaders
│  ├─ audio/
│  │  ├─ engine.ts         AudioContext/Tone lifecycle, unlock, master bus
│  │  ├─ instruments.ts    guitar/piano/bass samplers, drum kit
│  │  ├─ transport.ts      tempo, swing, count-in, loop, position events
│  │  ├─ backing/          walkingBass.ts, comping.ts, drums.ts, arrange.ts
│  │  ├─ metronome.ts
│  │  └─ playback.ts       play a NoteEvent[] (licks, scales, arpeggios, voicings)
│  ├─ pitch/
│  │  ├─ mic.ts            permission, stream, input gain, noise gate
│  │  ├─ detector.worklet.ts  AudioWorklet: MPM pitch + RMS onset
│  │  ├─ noteTracker.ts    frames → stable DetectedNote events
│  │  ├─ calibration.ts    latency + noise-floor calibration
│  │  └─ grader.ts         compare DetectedNote stream to ExpectedNote targets
│  ├─ practice/
│  │  ├─ srs.ts            spaced-repetition scheduler (SM-2 variant)
│  │  ├─ tempoLadder.ts
│  │  ├─ planner.ts        daily session generator
│  │  └─ scoring.ts
│  ├─ storage/
│  │  ├─ db.ts             Dexie schema + migrations
│  │  ├─ repos/*.ts        settings, progress, sessions, songs, userLicks
│  │  └─ backup.ts         JSON export/import (zod-validated)
│  ├─ ui/
│  ├─ features/
│  └─ app/
└─ tests/e2e/
```

## 5. Core data models

These live in `src/theory/types.ts` and `src/content/schemas.ts`. Agents must not change their
shapes without updating this document.

```ts
// ---------- theory ----------
type PitchClass = 0|1|2|3|4|5|6|7|8|9|10|11;          // C = 0
interface Note { pc: PitchClass; letter: 'C'|'D'|'E'|'F'|'G'|'A'|'B'; acc: -2|-1|0|1|2; octave?: number }
type Midi = number;

interface Tuning { name: string; strings: Midi[] }      // low→high, standard = [40,45,50,55,59,64]
interface FretPos { string: number; fret: number }      // string 0 = low E

type Degree = string;   // '1','b3','3','#4','5','b7','7','9','b9','#9','11','#11','13','b13'

interface ChordSymbol {
  root: Note;
  quality: ChordQuality;        // 'maj7'|'7'|'m7'|'m7b5'|'dim7'|'m6'|'6'|'mMaj7'|'7alt'|'7sus4'|...
  extensions: Degree[];         // e.g. ['9','13']
  alterations: Degree[];        // e.g. ['b9','#11']
  bass?: Note;                  // slash chords
  text: string;                 // original text, e.g. 'G7(b9)'
}

type ScaleId = 'ionian'|'dorian'|'phrygian'|'lydian'|'mixolydian'|'aeolian'|'locrian'
  |'melodicMinor'|'lydianDominant'|'altered'|'locrianNat2'|'harmonicMinor'|'phrygianDominant'
  |'halfWholeDim'|'wholeHalfDim'|'wholeTone'|'majorPent'|'minorPent'|'blues'|'majorBlues'
  |'bebopDominant'|'bebopMajor'|'dorianBebop';

type NoteTier = 'root'|'chordTone'|'guideTone'|'tension'|'avoid'|'chromatic';

interface KeyCenter { tonic: Note; mode: 'major'|'minor' }

interface AnalyzedChord {
  chord: ChordSymbol;
  key: KeyCenter;               // local key
  roman: string;                // 'ii7', 'V7/ii', 'bII7 (subV)', 'iv7 (backdoor)'
  function: 'tonic'|'subdominant'|'dominant'|'secondaryDominant'|'tritoneSub'|'passingDim'|'modalInterchange'|'other';
  scales: { scale: ScaleId; root: Note; rank: number; why: string }[];
  targetNotes: Note[];          // guide tones / best landing notes
}

// ---------- content ----------
interface Progression {
  id: string;
  name: string;                 // 'Major ii–V–I'
  tags: string[];               // ['jazz','core','cadence']
  style: 'swing'|'bossa'|'ballad'|'blues-shuffle'|'straight';
  defaultKey: KeyCenter;
  defaultTempo: number;
  bars: { chords: { roman: string; beats: number }[] }[];   // written in roman numerals, so any key works
  notes: string;                // teaching notes (markdown)
}

interface Song {                // user-entered chart (e.g. a standard from a fake book)
  id: string; title: string; key?: KeyCenter; tempo: number; style: Progression['style'];
  form: { label: string; bars: { chords: { symbol: string; beats: number }[] }[] }[];  // A, B, ...
}

interface LickNote {
  degree: Degree;               // relative to the lick's reference chord root
  octave: number;               // relative octave (0 = the octave starting at the reference root)
  start: number;                // beats from lick start
  dur: number;                  // beats
  tech?: 'h'|'p'|'slide'|'bend'|'vib'|'rake';
  bendTo?: Degree;
}
interface Lick {
  id: string; name: string;
  context: 'V7'|'ii-V'|'ii-V-I'|'minor ii-V'|'maj7'|'m7'|'blues-I7'|'blues-IV7'|'blues-turnaround'|'minor-blues'|'static-minor';
  chords: { roman: string; beats: number }[];   // the harmony the lick is written over
  concepts: string[];           // ['enclosure','guide-tone','chromatic-approach','arpeggio','bebop-scale']
  difficulty: 1|2|3|4|5;
  feel: 'swing'|'straight';
  notes: LickNote[];
  suggestedFingering?: { position: number; frets: FretPos[] };  // optional; otherwise computed
  source: 'original'|'user';
}

interface VoicingShape {
  id: string;
  quality: ChordQuality;
  family: 'shell'|'drop2'|'drop3'|'rootless'|'quartal'|'triad'|'freddieGreen'|'other';
  stringSet: number[];          // e.g. [0,2,3] for shells on 6th string
  degrees: (Degree|null)[];     // per string low→high, null = muted
  rootString: number | null;    // null for rootless
}

interface TechniqueDrill {
  id: string; name: string;
  category: 'chromatic'|'spider'|'legato'|'alternatePicking'|'economyPicking'|'stringSkipping'|'stretch'|'arpeggio'|'bending'|'vibrato';
  pattern: FretPos[] | { generator: string; params: Record<string, number> };
  subdivision: 1|2|3|4|6;       // notes per beat
  startBpm: number; goalBpm: number;
  instructions: string;
}

// ---------- practice / storage ----------
interface SrsState { itemId: string; ease: number; intervalDays: number; due: string; reps: number; lapses: number }
interface TempoRecord { itemId: string; key?: string; bestCleanBpm: number; history: { date: string; bpm: number; accuracy: number }[] }
interface PracticeSession { id: string; date: string; durationSec: number; blocks: { feature: string; itemId?: string; seconds: number; score?: number }[] }
interface Settings {
  tuning: Tuning; leftHanded: boolean; fretboardOrientation: 'horizontal'|'vertical';
  maxStretchFrets: 3|4|5|6;     // dexterity-aware fingering
  noteNames: 'sharps'|'flats'|'auto'; showDegreesInsteadOfNames: boolean;
  micLatencyMs: number; micNoiseGateDb: number; dailyGoalMin: number;
}
```

### Why licks are stored as scale degrees

A lick stored as degrees relative to its chord can be:

- **transposed** to any of the 12 keys for free,
- **re-fingered** in any position by the fingering engine, honoring `maxStretchFrets`,
- **dropped onto any song** wherever its `context` matches (for example, every ii–V in a tune),
- **validated** automatically: a content test checks that every non-chromatic note belongs to
  the chord or scale its context implies.

## 6. Key subsystems

### 6.1 Theory engine (`src/theory`)

A pure, deterministic library. [`THEORY_SPEC.md`](THEORY_SPEC.md) gives the full rules. Its
public API (exported from `src/theory/index.ts`):

```ts
parseChord(text: string): ChordSymbol
chordTones(c: ChordSymbol): { note: Note; degree: Degree; tier: NoteTier }[]
scaleNotes(scale: ScaleId, root: Note): Note[]
spellInKey(pc: PitchClass, key: KeyCenter): Note
romanToChord(roman: string, key: KeyCenter): ChordSymbol
analyzeProgression(chords: ChordSymbol[] | Song): AnalyzedChord[]   // key regions + roman + scales
detectKey(chords: ChordSymbol[]): { key: KeyCenter; confidence: number }[]  // ranked
notesOnFretboard(notes: Note[] | PitchClass[], tuning: Tuning, fretRange?: [number, number]): FretPos[]
positions(scale: ScaleId, root: Note, system: 'caged'|'3nps'|'position'): FretPos[][]
generateVoicings(c: ChordSymbol, family: VoicingShape['family'], tuning: Tuning): Voicing[]
voiceLead(prog: ChordSymbol[], family, opts): Voicing[]             // smoothest path
fingerLine(notes: Midi[], opts: { tuning; maxStretch; preferredPosition? }): FretPos[]
transpose(x, semitones)
```

### 6.2 Audio engine (`src/audio`)

- **Lifecycle.** One `AudioContext`, created lazily and resumed from the app's "Tap to start"
  gate. A master bus has separate gain channels: `backing`, `examples`, `metronome`.
- **Transport.** Tone.js Transport handles BPM, swing ratio (0 to 0.67), loop region, and a
  count-in. Every bar and beat it publishes `{bar, beat, chordIndex, chord}` to a lightweight
  event emitter. The UI uses this to animate the fretboard ahead of time: next-chord preview
  runs one beat early.
- **Instruments.** Sampled nylon/clean electric guitar, Rhodes or piano for comping, and
  upright bass, at about 1 sample per minor third, compressed to keep the PWA cache under
  ~15 MB. Drums are synthesized (ride, hi-hat foot, brush swirl, kick, snare) so they need no
  samples.
- **Backing-track generator (`backing/`).** Input: `AnalyzedChord[]`, style, and tempo.
  Output: scheduled events.
  - *Walking bass*: beat 1 = root (or the 3rd for variety). Beats 2 and 3 = chord or scale tones
    moving toward the next root. Beat 4 = a chromatic or scale approach note (half step
    above/below, or the dominant of the next root). Keep within E1–G3 and limit leaps. Seeded
    RNG so a given loop is reproducible.
  - *Bossa/straight bass*: root and fifth patterns.
  - *Comping*: rhythm patterns from `content/comping` (Freddie Green 4-to-the-bar, Charleston,
    bossa, ballad whole notes, anticipations) × voicings from `voiceLead()`.
  - *Drums*: per-style patterns (swing ride + 2 & 4 hi-hat, bossa, shuffle, rock).
  - Each part has a mute toggle. Muting the comping lets you practice comping yourself.
- **Playback helpers.** `playNotes(NoteEvent[])` plays licks, scales, arpeggios, and voicings in
  sync with the visual cursor on the fretboard and tab.

### 6.3 Pitch detection and grading (`src/pitch`)

```
mic stream ─► AudioWorklet (2048-sample window, hop 256 @ 48 kHz)
               ├─ RMS + spectral-flux onset detector
               └─ McLeod Pitch Method → (freqHz, clarity)
           ─► noteTracker: median-filter frames, require ≥ 3 stable frames (~16 ms) and
                           clarity ≥ 0.9, fold to MIDI, emit DetectedNote{midi, cents, tOn, tOff}
           ─► grader: match DetectedNote to ExpectedNote[] within
                      timing window (default ±120 ms, minus calibrated latency) and pitch rule
```

- **Matching rules** depend on the exercise: exact MIDI note (lick playback, fret quiz), any
  octave of a pitch class (find-the-note), any member of a set (land on a chord tone), or a
  tier score (chord tone = 1.0, tension = 0.7, avoid note = 0, chromatic passing note on a
  weak beat = neutral).
- **Calibration.** On first use, play a click through the speaker while the user strums a muted
  string on it 8 times. The median offset becomes `micLatencyMs`. A 3-second room-noise sample
  sets the noise gate.
- **Limits.** Monophonic only; chords are not graded in v1. Pitch detection runs in the
  worklet thread so the UI stays at 60 fps.
- The **tuner** feature reuses this pipeline and is a good first end-to-end test of it.

### 6.4 Practice system (`src/practice`)

- **Spaced repetition (SM-2 variant)** for every learnable item: fretboard note cells, licks
  (per key), voicings, progressions, and key-finding drills. Grades come from the mic grader
  where possible and from self-rating ("again / hard / good / easy") otherwise.
- **Tempo ladder.** Each drill and lick starts at `startBpm`, or 60% of the last clean tempo.
  Two consecutive runs at ≥ 90% accuracy raise the tempo 4 BPM; a run under 70% lowers it
  4 BPM. Records are kept per item and per key. Slow and accurate always beats fast and sloppy,
  which matters for the dexterity goal.
- **Planner.** Builds a daily session from a time budget (default 30 min) and a weighting
  across the six goals:
  warm-up/technique 15% · fretboard 15% · changes 25% · licks 20% · voicings/comping 15% ·
  key-finding/ear 10%.
  It pulls the items that are due in SRS first and rotates the key of the day around the circle
  of fourths.

### 6.5 Storage (`src/storage`)

Dexie database `fretwise`, version 1:

| Table | Key | Indexes |
| --- | --- | --- |
| `settings` | `'singleton'` | — |
| `srs` | `itemId` | `due` |
| `tempo` | `[itemId+key]` | `itemId` |
| `sessions` | `id` | `date` |
| `songs` | `id` | `title` |
| `userLicks` | `id` | `context` |

All access goes through repository functions. A backup exports and imports one JSON file,
validated by zod and versioned for migrations.

### 6.6 Shared UI components (`src/ui`)

- **`<Fretboard>`** is the most important component. Props:
  `{ tuning, fretRange, orientation, leftHanded, markers: Marker[], onTapPosition, highlightWindow, cursor }`,
  where `Marker = { pos: FretPos; label: string; tier: NoteTier; emphasis?: 'pulse'|'ghost' }`.
  - Colors by tier: root, chord tone, guide tone (3rd/7th), tension, avoid note, chromatic. The
    palette is color-blind safe, and every marker also has a label so color is never the only
    signal.
  - On a phone in portrait, a **vertical** orientation (nut at the top) shows 12+ frets
    legibly. Horizontal mode scrolls sideways in landscape.
  - Supports "position lock", which dims everything outside a 4–5 fret window.
- **`<ChordDiagram>`**: vertical chord box with finger numbers and degree labels.
- **`<TabView>`**: renders `LickNote[]` plus a fingering as tab with a rhythm row and a
  playback cursor.
- **`<ChordChart>`**: bar grid of chords that highlights the current bar. Tapping a bar sets
  the loop.
- **`<TransportBar>`**: play/stop, tempo, loop, count-in, swing, mixer toggles, transpose.

## 7. Non-functional requirements

- **Performance**: first load < 3 s on 4G; later loads come from the cache. The fretboard
  re-renders in under 4 ms per beat; markers are memoized and the board is not re-rendered as
  a whole.
- **Timing**: audio events are scheduled ≥ 100 ms ahead on the Tone.js clock. Visual updates
  are driven from that clock with `Tone.Draw`, never from `setTimeout`.
- **Mobile UX**: touch targets ≥ 44 px, one-handed controls at the bottom of the screen,
  Screen Wake Lock during practice, works in portrait.
- **Accessibility**: no information carried by color alone; text ≥ 16 px; respects
  `prefers-reduced-motion`.
- **Privacy**: microphone audio never leaves the device and nothing is recorded to storage.
  There are no analytics.

## 8. Possible extensions (out of scope for v1)

- **Claude-powered coach**: ask "why does Bb7 work over this bar?" or "give me three licks
  like this one". This would need an API key and a network connection, so it stays optional and
  goes behind a settings toggle.
- **Audio key finder**: detect the key and chords from a recording or a song playing in the
  room (chroma features). Hard to do well, so v1 only analyzes chord input.
- Polyphonic chord recognition for grading comping.
- MusicXML/iReal Pro chart import.
