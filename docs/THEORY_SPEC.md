# Music Theory Engine Specification

This document is the source of truth for `src/theory` and for the musical content in
`src/content`. Every rule here must have unit tests, including the golden examples in
section 9.

## 1. Pitch, spelling, and intervals

- Pitch classes are integers 0–11 (C = 0). Each `Note` also carries a **letter and an
  accidental**, so the engine always spells notes correctly: the 3rd of E♭7 is G, the 7th is
  D♭ (not C♯), and the 3rd of B7 is D♯ (not E♭).
- **Spelling rule.** Scales and chords are spelled letter by letter from the root. A diatonic
  7-note scale uses each letter once. Symmetric scales (diminished, whole tone) use the
  conventional spellings in `scale.ts` fixtures.
- **Accidental preference** when there is no chord context (bare fretboard labels): use the
  key signature of the current key. Otherwise follow `Settings.noteNames`.
- Degrees are written relative to a root: `1 b2 2 b3 3 4 #4/b5 5 #5/b6 6 b7 7`. Tensions above
  the octave are written `b9 9 #9 11 #11 b13 13`.

## 2. Scale catalogue

Each scale is defined as a list of degrees. These are the minimum for v1:

| ScaleId | Degrees |
| --- | --- |
| ionian | 1 2 3 4 5 6 7 |
| dorian | 1 2 b3 4 5 6 b7 |
| phrygian | 1 b2 b3 4 5 b6 b7 |
| lydian | 1 2 3 #4 5 6 7 |
| mixolydian | 1 2 3 4 5 6 b7 |
| aeolian | 1 2 b3 4 5 b6 b7 |
| locrian | 1 b2 b3 4 b5 b6 b7 |
| melodicMinor | 1 2 b3 4 5 6 7 |
| lydianDominant | 1 2 3 #4 5 6 b7 |
| altered | 1 b2 #2 3 b5 #5 b7 (spelled as b9 #9 3 #11 b13 b7) |
| locrianNat2 | 1 2 b3 4 b5 b6 b7 |
| harmonicMinor | 1 2 b3 4 5 b6 7 |
| phrygianDominant | 1 b2 3 4 5 b6 b7 |
| halfWholeDim | 1 b2 #2 3 #4 5 6 b7 |
| wholeHalfDim | 1 2 b3 4 b5 #5 6 7 |
| wholeTone | 1 2 3 #4 #5 b7 |
| majorPent | 1 2 3 5 6 |
| minorPent | 1 b3 4 5 b7 |
| blues | 1 b3 4 b5 5 b7 |
| majorBlues | 1 2 b3 3 5 6 |
| bebopDominant | 1 2 3 4 5 6 b7 7 |
| bebopMajor | 1 2 3 4 5 #5 6 7 |
| dorianBebop | 1 2 b3 3 4 5 6 b7 |

## 3. Chords

### 3.1 Symbol parsing

`parseChord` must accept the common jazz spellings and normalize them:

| Input examples | Quality |
| --- | --- |
| `C`, `CM`, `Cmaj` | major triad |
| `Cmaj7`, `CM7`, `CΔ`, `CΔ7`, `Cma7` | maj7 |
| `C6`, `C69`, `C6/9` | 6 (with 9) |
| `Cm`, `Cmin`, `C-` | minor triad |
| `Cm7`, `Cmin7`, `C-7` | m7 |
| `Cm6`, `C-6` | m6 |
| `CmMaj7`, `Cm(maj7)`, `C-Δ` | mMaj7 |
| `C7`, `C9`, `C13` | dominant (with extensions) |
| `C7b9`, `C7#9`, `C7#11`, `C7b13`, `C7(b9,#11)` | dominant + alterations |
| `C7alt`, `Calt` | 7alt |
| `C7sus4`, `C7sus`, `C9sus`, `Csus` | 7sus4 / sus4 |
| `Cm7b5`, `Cø`, `Cø7`, `C-7b5` | m7b5 |
| `Cdim`, `C°`, `Cdim7`, `C°7`, `Co7` | dim / dim7 |
| `Caug`, `C+`, `C7#5`, `C+7` | aug / 7#5 |
| `C/E`, `Cm7/Bb` | slash chord (`bass`) |

Unknown symbols raise a typed `ChordParseError` that carries the position of the bad text. The
UI shows this inline.

### 3.2 Chord tones and note tiers

For a chord and its chosen scale, every pitch class falls into one tier:

| Tier | Definition |
| --- | --- |
| `root` | the chord root |
| `guideTone` | the 3rd and 7th (or the 6th in 6 chords; the 4th in sus chords) |
| `chordTone` | other chord members (5th, written extensions) |
| `tension` | scale notes that are available tensions (9, 11, 13 and their alterations as allowed below) |
| `avoid` | scale notes a **half step above a chord tone**, on non-dominant chords: the 4 over maj7, the b6 over m7 in aeolian, the b2 over m7 in phrygian. Over **unaltered dominants** the 4 is an avoid note (unless sus). b9/b13 are **not** avoid notes when the dominant resolves to minor or is altered. |
| `chromatic` | everything else. Fine as passing or approach notes on weak beats. |

The fretboard colors notes by tier. The grader scores the tiers 1.0 / 1.0 / 1.0 / 0.7 / 0 and
treats chromatic notes as neutral.

## 4. Chord-scale assignment

The scale assigned to a chord depends on its **function in the key**, not only on its
quality. `analysis/chordScale.ts` assigns scales in two passes.

### 4.1 Default table, by function

| Chord / function | Primary scale | Alternatives (ranked) | Practical tip shown in UI |
| --- | --- | --- | --- |
| Imaj7 | ionian | majorPent (root), lydian, bebopMajor | Land on the 3rd or 7th; treat the 4 as a passing note. |
| IVmaj7 | lydian | majorPent (root), ionian | The #11 is the color note. |
| bVImaj7, bIIImaj7 (borrowed) | lydian | ionian | |
| ii-7 | dorian | minorPent (root), dorianBebop | Same notes as the parent major key. |
| iii-7 | phrygian | aeolian, minorPent | Aim for the 3rd and 7th; skip the b2. |
| vi-7 | aeolian | dorian, minorPent | |
| i-7 (minor-key tonic, modal) | dorian | aeolian, minorPent, blues | |
| i-6, i-mMaj7 | melodicMinor | dorian | |
| V7 → major | mixolydian | bebopDominant, majorPent (root), lydianDominant | Target the 3rd of V → the root of I, and the 7th of V → the 3rd of I (guide tones). |
| V7 → minor | phrygianDominant | altered, halfWholeDim | b9 and b13 point to the minor tonic's 5th and 3rd. |
| V7alt | altered | halfWholeDim | |
| V7b9 (secondary or not) | halfWholeDim | phrygianDominant | |
| bII7 (tritone sub), bVII7 (backdoor), IV7 (blues), non-resolving 7 | lydianDominant | mixolydian | |
| 7sus4 | mixolydian (4 is a chord tone) | dorian from the 5th | |
| ii-7b5 (minor key) | locrian | locrianNat2 | |
| #i°7, #iv°7 passing dim | wholeHalfDim | spell as a rootless 7b9 of the dominant a major 3rd below | Over C#°7 between Cmaj7 and Dm7, think A7b9. |
| I7 / IV7 / V7 in a **blues** | blues (key root) + mixolydian (chord root) | minorPent, majorPent (chord root), dorianBebop | Mix the minor 3rd of the key with the major 3rd of each chord. |

### 4.2 Fallback rule for non-diatonic chords

Take the **current key's diatonic collection** and replace only the notes that clash with the
chord's tones. Examples in C major:

- **A7 (V7/ii)** → C D E F G A B with C→C♯ gives A B C♯ D E F G = A phrygian dominant.
- **D7 (V7/V)** → F→F♯ gives D mixolydian.
- **E7 (V7/vi)** → G→G♯ gives E phrygian dominant.

This "change as few notes as possible" rule is how a jazz player hears these chords, and it
handles odd chords no table covers. When the table and the fallback disagree, the table is
rank 1 and the fallback rank 2.

## 5. Harmonic analysis and key detection

### 5.1 Inputs

`ChordSymbol[]` with durations in beats, or a `Song`.

### 5.2 Algorithm

1. **Pitch profile per chord.** Root weight 2.0, 3rd and 7th 1.5, other tones 1.0, multiplied
   by duration.
2. **Candidate keys.** 24 keys: 12 major (ionian collection) and 12 minor (natural ∪ harmonic
   minor collection, so V7 in minor fits).
3. **Emission score** of chord *c* in key *k*:
   - +3 if *c* is diatonic to *k* with the expected quality;
   - +2 if *c* is an explainable non-diatonic chord in *k*: secondary dominant that resolves to
     a diatonic target, its related ii, tritone sub, backdoor bVII7, modal-interchange iv / bVI
     / bVII / bIII, or passing diminished;
   - otherwise −(number of chord tones outside *k*).
4. **Cadence bonuses.** +4 for a V7→I (or V7→i) resolution to *k*'s tonic, +3 for ii–V–I, +2
   if the first or last chord of the form is *k*'s tonic.
5. **Segmentation.** Run Viterbi over chords with the 24 keys as states and a **key-change
   penalty** (default 6, adjustable). A short tonicization (V/x → x for ≤ 2 bars) stays a
   secondary-dominant label inside the current key. A **modulation** happens only when the new
   key is confirmed by a cadence and lasts ≥ 2 bars.
6. **Confidence.** Softmax over total path scores. Report the top 3 global keys plus the region
   list.
7. **Special labels:**
   - **Blues**: an I7–IV7–I7–V7 pattern with dominant tonics → "Blues in X", with the blues
     chord-scale row from section 4.1.
   - **Modal**: one chord lasting ≥ 4 bars, or a two-chord vamp → "X Dorian / Mixolydian /
     ..." instead of a key.

### 5.3 Pattern labeling (`roman.ts`)

Patterns are matched on root motion plus quality, in this priority order:

1. ii–V pair: m7 (or m7b5) → dom7 whose root is a 4th up / 5th down. Draw a bracket over the
   pair, and an arrow if it resolves.
2. Tritone substitution: dom7 → resolution a half step down. Label `subV7/x`.
3. Secondary dominant: dom7 a 5th above its resolution target. Label `V7/x`. If the dominant
   doesn't resolve, label it `V7/x (unresolved)` or `lydian dominant`.
4. Backdoor: iv-7 → bVII7 → I.
5. Passing diminished: dim7 between two chords whose roots are a whole step apart.
6. Modal interchange: borrowed chords from the parallel minor.

### 5.4 Output

An `AnalyzedChord[]` (see ARCHITECTURE §5). For each chord it includes `targetNotes`: the
guide tones, sorted by **smallest motion from the previous chord's guide tones**. These feed
the "guide-tone line" exercise.

## 6. Fretboard geometry

- `noteAt(pos, tuning)`, `positionsOf(pc, tuning, range)`, `octaveShapes(pos)`.
- **Position systems** for every scale and arpeggio:
  - **CAGED**: 5 shapes per key, each anchored on its root shape.
  - **3 notes per string (3NPS)**: 7 patterns, for diatonic 7-note scales only.
  - **Position playing**: a 4-fret window plus one stretch either way, with one finger per fret.
    This is the default for the "position lock" feature.
- Pentatonic boxes 1–5 are a CAGED projection. Label them with the familiar box numbers, since
  the user already knows them, and use them as the starting point for learning the major-scale
  and arpeggio shapes that sit "around" each box.

## 7. Fingering optimizer (`fingering.ts`)

Given a melodic line (MIDI notes) and settings, choose a `FretPos` for every note.

- **Candidates**: every (string, fret) with fret ≤ 17 that produces the note.
- **Viterbi/DP cost per transition**:
  - shift cost = |new hand position − old hand position| × 2 (the hand position is the fret
    under the index finger);
  - **stretch cost = ∞** if the span within one hand position is greater than
    `Settings.maxStretchFrets`. This is the dexterity accommodation and matters most.
  - string skip cost = 0.5 × (strings skipped);
  - same finger on adjacent frets on different strings, consecutively = 1.5 (awkward roll);
  - open string = +0.5 (configurable; open strings are fine in blues but less so in jazz lines);
  - distance from `preferredPosition` = 0.3 per fret.
- **Output**: frets plus a suggested finger (1–4) for each note.

Golden tests check that a C major scale from C3 lands in a single position when
`maxStretch = 4`, and that no output ever exceeds `maxStretch`.

## 8. Voicings

### 8.1 Families and generation

| Family | Construction | String sets (low E = 6) |
| --- | --- | --- |
| **Shell** (start here) | R–3–7 or R–7–3 (R–3–6 for 6 chords) | root on 6: strings 6-4-3; root on 5: strings 5-4-3 or 5-3-2 |
| **Drop 2** | Close-position 4-note chord with the 2nd voice from the top dropped an octave; all 4 inversions | 4-3-2-1, 5-4-3-2, 6-5-4-3 |
| **Drop 3** | Close position with the 3rd voice from the top dropped an octave; all 4 inversions | 6-4-3-2, 5-3-2-1 |
| **Rootless** | Replace the root with the 9th (for example 3–5–7–9, 7–9–3–5); 3-note versions such as 3–7–9, 7–3–13, 3–b7–#9 | 5-4-3-2, 4-3-2-1 |
| **Triads** | Major/minor/dim/aug triads, 3 inversions | each adjacent 3-string set |
| **Quartal** | Stacked 4ths, for modal tunes ("So What" style) | 5-4-3-2, 4-3-2-1 |
| **Freddie Green** | 3-note shells on strings 6-4-3, played 4 to the bar | 6-4-3 |

Playability filter: one note per string, span ≤ `maxStretchFrets`, muted strings only between
fretted strings, no more than 2 notes on the same fret unless barrable. Prefer curated
`content/voicings` shapes when they exist; otherwise generate them.

### 8.2 Voice leading (`voiceLead`)

For a progression, pick one voicing per chord to minimize the sum of semitone motion across
corresponding voices. Penalize leaving the chosen register window and changing string set
(unless allowed). Use DP over the candidate voicings. Show the result with arrows connecting
each voice to where it moves; the 7th of ii falling to the 3rd of V is the key teaching
moment.

## 9. Golden examples (required tests)

| Input | Expected |
| --- | --- |
| `Dm7 G7 Cmaj7` | Key C major. ii–V–I bracket. Scales D dorian, G mixolydian, C ionian. Guide-tone line C → B → B (or F → F → E). |
| `Dm7b5 G7b9 Cm6` | Key C minor. Scales D locrian, G phrygianDominant (or halfWholeDim), C melodicMinor. |
| `Cm7 F7 Bbmaj7 Ebmaj7 Am7b5 D7 Gm6` | One key region, G minor / B♭ major (relative pair): ii–V–I in B♭, then a minor ii–V–i in G minor. |
| Jazz blues in F: `F7 Bb7 F7 Cm7 F7 Bb7 Bdim7 F7 D7 Gm7 C7 F7 D7 Gm7 C7` (12 bars) | "Blues in F". Cm7–F7 = ii–V of B♭ (IV). Bdim7 = passing #iv°. D7 = V7/ii. Gm7–C7 = ii–V. |
| Rhythm changes A in B♭: `Bb6 G7 Cm7 F7 Dm7 G7 Cm7 F7 …` | B♭ major. G7 = V7/ii. Dm7 = iii. |
| `Dm7` for 16 bars | Modal: D Dorian. |
| `Cmaj7 Db7 Cmaj7` | Db7 → C labeled `subV7/I`, lydianDominant. |
| `Cmaj7 Fm7 Bb7 Cmaj7` | Backdoor ii–V. Fm7 = iv-7 (borrowed), Bb7 = bVII7, lydianDominant. |
| `Cmaj7 C#dim7 Dm7 G7` | C#°7 = passing diminished, also shown as A7b9. |

## 10. Content to author for v1

All content is **original** (no transcriptions of copyrighted solos) and validated by tests.

### 10.1 Progressions (written in roman numerals, playable in all 12 keys)

1. Major ii–V–I (2 bars and 4 bars)
2. Minor ii–V–i
3. I–vi–ii–V turnaround
4. iii–vi–ii–V–I
5. 12-bar blues (basic I7–IV7–V7)
6. Jazz blues (section 9 example)
7. Minor blues (i-7 iv-7 bVI7 V7alt)
8. Rhythm changes, A section and bridge (III7–VI7–II7–V7)
9. ii–V–I with a tritone sub (ii–bII7–I)
10. Backdoor ii–V (iv-7–bVII7–I)
11. ii–V–I chain moving down in whole steps (Imaj7 → bVIImaj7 …), for practicing key changes
12. Cycle of ii–Vs in major plus the relative minor (an "Autumn Leaves"-type cycle)
13. Modal vamps: i-7 for 16 bars (Dorian), and a two-chord Dorian/Mixolydian vamp
14. Bossa-style i-7–iv-7 / ii-7b5–V7alt minor progression
15. I–IV7 two-chord "funk/blues" vamp, as a bridge from the user's blues comfort zone to dominant chord-scales

### 10.2 Voicings

Curated sets for maj7, m7, 7, m7b5, dim7, m6, 6, 7alt, 7b9, 9, 13, and 7sus4 across the
families in section 8.1. That is about 150 shapes; the rest are generated.

### 10.3 Licks (target: 60 for v1)

| Context | Count | Concepts |
| --- | --- | --- |
| Blues I7 / IV7 / turnaround | 12 | Starts from the pentatonic boxes the user knows, adds the major 3rd, the 6th, and the 9th |
| V7 → I | 10 | Guide tones, the 3rd of V to the root of I, enclosures of the target 3rd |
| ii–V–I major | 14 | Arpeggio from the 3rd ("3-to-9"), bebop scale lines, chromatic approach, guide-tone lines |
| minor ii–V–i | 10 | b9 → 5, altered-scale fragments, diminished patterns |
| maj7 / m7 static | 8 | Pentatonic superimposition (e.g. E minor pent over Cmaj7 = 3-5-13-7-9 (E G A B D)) |
| Minor blues / static minor | 6 | Dorian 6th, minor pent + 9 |

Each lick lists `difficulty`, `concepts`, and a `context` the app can find in any song.
Content tests require that every lick's notes have tier ≠ `avoid` on strong beats in their
context.

### 10.4 Technique drills

Chromatic 1-2-3-4 permutations (all 24), spider exercises, one-finger-per-fret with a
`maxStretch` variant, legato trills (1-2, 1-3, 2-4, 3-4: weak-finger focus), alternate-picking
scale sequences (3s, 4s, 6ths), string skipping, arpeggio sweeps (3-string triads), bending to
pitch (graded by the tuner pipeline), and vibrato width/speed. Each drill has `startBpm` /
`goalBpm` and is driven by the tempo ladder.
