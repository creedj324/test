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
| altered | 1 b9 #9 3 #11 b13 b7 (C D♭ D♯ E F♯ A♭ B♭) |
| locrianNat2 | 1 2 b3 4 b5 b6 b7 |
| harmonicMinor | 1 2 b3 4 5 b6 7 |
| phrygianDominant | 1 b2 3 4 5 b6 b7 |
| halfWholeDim | 1 b2 #2 3 #4 5 6 b7 |
| wholeHalfDim | 1 2 b3 4 b5 b6 6 7 (C D E♭ F G♭ A♭ A B) |
| wholeTone | 1 2 3 #4 #5 b7 |
| majorPent | 1 2 3 5 6 |
| minorPent | 1 b3 4 5 b7 |
| blues | 1 b3 4 b5 5 b7 |
| majorBlues | 1 2 b3 3 5 6 |
| bebopDominant | 1 2 3 4 5 6 b7 7 |
| bebopMajor | 1 2 3 4 5 #5 6 7 |
| dorianBebop | 1 2 b3 3 4 5 6 b7 |
| minor6dim | 1 2 b3 4 5 b6 6 7 (Barry Harris minor-6th diminished) |

**Spelling exceptions.** Altered, the diminished scales, whole tone, and the 8-note bebop scales
cannot use one letter per degree. They use the fixed spellings in the table above. The bebop scales
add one chromatic note, spelled as written (bebopMajor's #5 = G♯ in C).

## 3. Chords

### 3.1 Symbol parsing

`parseChord` must accept the common jazz spellings and normalize them. `C6/9` and `C69` are
6/9 chords, never a slash chord with bass "9"; a slash is read as a bass note only when a note
name follows it.

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
| `Cdim`, `C°`, `Cdim7`, `C°7`, `Co7` | dim7. On jazz lead sheets ° and dim almost always mean dim7; a setting allows the triad reading. |
| `Caug`, `C+`, `C7#5`, `C+7` | aug / 7#5 |
| `Cmaj7#11`, `Cmaj9`, `Cmaj7#5`, `Cm9`, `Cm11`, `C13sus`, `C7b9b13` | extended / altered forms of the qualities above |
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
| `avoid` | scale notes a **half step above a chord tone**, on non-dominant chords: the 4 over maj7, the b6 over m7 in aeolian, **both the b2 and the b6** over iii-7 (phrygian), and
the b2 over m7b5 in locrian (the reason locrian ♮2 exists). Over **unaltered dominants** the 4 is an avoid note (unless sus). b9/b13 are **not** avoid notes when the dominant resolves to minor or is altered. |
| `chromatic` | everything else. Fine as passing or approach notes on weak beats. |

**Dorian's 6** over a functional ii-7 is not an avoid note, but it forms a tritone with the b3 and
implies V7. Show it as a tension with a "don't sustain it here" hint. In modal Dorian it is the
characteristic color note.

**Blues context.** Over I7, IV7 and V7 in a blues, the key's b3 (heard as #9 over I7) and b5 are
**blue notes** and get the `tension` tier, not `avoid` or `chromatic`.

The fretboard colors notes by tier. ARCHITECTURE §6.3 defines how the grader scores each tier.

## 4. Chord-scale assignment

The scale assigned to a chord depends on its **function in the key**, not only on its
quality. `analysis/chordScale.ts` assigns scales in two passes. **Function rows take precedence
over quality rows** (for example, "V7 → minor" wins over "V7b9").

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
| V7 → minor | phrygianDominant | altered, halfWholeDim | The b9 resolves down a half step to the tonic's 5th; the b13 is the tonic's minor 3rd (a common tone). |
| V7alt | altered | halfWholeDim | |
| V7b9 → major target | halfWholeDim | phrygianDominant | |
| V7b9 → minor target | phrygianDominant | halfWholeDim, altered | H-W's natural 13 clashes with a minor target, so phrygian dominant comes first. |
| bII7 (tritone sub), bVII7 (backdoor), II7 (e.g. bar 3 of "Take the A Train"), non-resolving 7 | lydianDominant | mixolydian | |
| 7sus4 | mixolydian (4 is a chord tone) | dorian from the 5th | |
| ii-7b5 (minor key) | locrian | locrianNat2 | |
| #i°7, #iv°7 passing dim | wholeHalfDim | spell as a rootless 7b9 of the dominant a major 3rd below | Over C#°7 between Cmaj7 and Dm7, think A7b9. |
| I7 / IV7 / V7 in a **blues** | blues (key root) + mixolydian (chord root) | minorPent, majorPent (chord root), lydianDominant (rank 3) | Mix the minor 3rd of the key with the major 3rd of each chord. Over IV7 the blues sound uses the ♮4 (the key's b7), not #11. |

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
2. **Candidate keys.** 24 keys: 12 major (ionian collection) and 12 minor (natural ∪ harmonic ∪
   melodic minor collection, so V7, i-6 and i-mMaj7 all fit). i-7, i-6 and i-mMaj7 all count as
   tonic quality.
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
     chord-scale row from section 4.1. **The blues label overrides Viterbi segmentation.** Without
     this, Cm7–F7–B♭7 in an F blues scores as a ii–V–I in B♭ and splits the form into B♭ regions.
     Inner ii–Vs are still labeled as ii–V of IV, etc.
   - **Modal**: one chord lasting ≥ 4 bars, or a two-chord vamp → "X Dorian / Mixolydian /
     ..." instead of a key.

### 5.3 Pattern labeling (`roman.ts`)

Patterns are matched on root motion plus quality, in this priority order:

1. Backdoor: iv-7 → bVII7 → I, or bVII7 → I. This must run before the ii–V rule, or Fm7–B♭7 →
   C would be labeled a ii–V of E♭.
2. Tritone substitution: dom7 → resolution a half step down. Label `subV7/x`.
3. ii–V pair: m7 (or m7b5) → dom7 whose root is a 4th up / 5th down. Draw a bracket over the
   pair, and an arrow if it resolves. Chained, non-resolving ii–Vs (e.g. "Satin Doll") are still
   bracketed.
4. Secondary dominant: dom7 a 5th above its resolution target. Label `V7/x`. If the dominant
   doesn't resolve, label it `V7/x (unresolved)` or `lydian dominant`.
5. Passing diminished: a dim7 whose root is a half step above the previous root and a half step
   below the next chord's root or bass (I → #I°7 → ii; #IV°7 → I/5), or a half step below the
   previous root and above the next (iii → bIII°7 → ii). Also #iv°7 → I in a blues (B♭7 → B°7 →
   F7 in F).
6. Modal interchange: borrowed chords from the parallel minor.

### 5.4 Output

An `AnalyzedChord[]` (see ARCHITECTURE §5). For each chord it includes `targetNotes`: the
guide tones, sorted by **smallest motion from the previous chord's guide tones**. These feed
the "guide-tone line" exercise.

## 6. Fretboard geometry

- `noteAt(pos, tuning)`, `positionsOf(pc, tuning, range)`, `octaveShapes(pos)`.
- **Position systems** for every scale and arpeggio:
  - **CAGED**: 5 shapes per key, each anchored on its root shape.
  - **Position playing**: a 4-fret window plus one stretch either way, with one finger per fret.
    This is the **default** system and the default for the "position lock" feature.
  - **3 notes per string (3NPS)**: 7 patterns, for diatonic 7-note scales only. **Optional**: 3NPS
    patterns routinely need 5-fret spans, so they are hidden when they exceed `maxStretchFrets`.
  - **Single string** (Mick Goodrick, *The Advancing Guitarist*): a scale or arpeggio along one
    string. It teaches the neck linearly and needs no stretch.
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
| **Shell** (start here) | R–3–7 or R–7–3 (R–3–6 for 6 chords). m7b5 must include the b5 (R–b5–b7, or R–b7–b3–b5); dim7 uses R–b3–bb7 or a full 4-note voicing. | R–3–7: 6-5-4 and 5-4-3. R–7–3: 6-4-3 and 5-3-2. |
| **Drop 2** | Close-position 4-note chord with the 2nd voice from the top dropped an octave; all 4 inversions | 4-3-2-1, 5-4-3-2, 6-5-4-3 |
| **Drop 3** | Close position with the 3rd voice from the top dropped an octave; all 4 inversions | 6-4-3-2, 5-3-2-1 |
| **Rootless** | Major/minor: A = 3–5–7–9, B = 7–9–3–5. Dominant (the 5 becomes the 13): A = 3–13–b7–9, B = b7–9–3–13 (Levine). 3-note versions such as 3–7–9, 7–3–13, 3–b7–#9. | 5-4-3-2, 4-3-2-1 |
| **Triads** | Major/minor/dim/aug triads, 3 inversions | each adjacent 3-string set |
| **Quartal** | Stacked perfect 4ths, for modal tunes | 5-4-3-2, 4-3-2-1 |
| **"So What" chord** | Three perfect 4ths plus a major 3rd on top (E A D G B over Em); its own shape, not pure quartal | 6-5-4-3-2, 5-4-3-2-1 |
| **6th-diminished** (Barry Harris) | Drop 2 C6 (C E G A) alternating with B°7 (B D F A♭) on each scale step of bebopMajor / minor6dim. Low stretch, good for limited dexterity. | 4-3-2-1, 5-4-3-2 |
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
| `Dm7b5 G7b9 Cm6` | Key C minor (requires the melodic-minor collection, §5.2). Scales D locrian, G phrygianDominant (rank 1, V7b9 → minor), C melodicMinor. |
| `Cm7 F7 Bbmaj7 Ebmaj7 Am7b5 D7 Gm6` | Global key **G minor** (last chord plus the D7 → Gm cadence bonus), with B♭ major as the relative. ii–V–I in B♭; E♭maj7 = IVmaj7 of B♭ / bVImaj7 of Gm; then a minor ii–V–i in G minor. |
| Jazz blues in F: `F7 \| Bb7 \| F7 \| Cm7 F7 \| Bb7 \| Bdim7 \| F7 \| D7 \| Gm7 \| C7 \| F7 D7 \| Gm7 C7` (12 bars) | "Blues in F" over the whole form (no B♭ key region). Cm7–F7 = ii–V of B♭ (IV). Bdim7 = passing #iv°7. D7 = V7/ii. Gm7–C7 = ii–V. |
| Rhythm changes A in B♭: `Bb6 G7 Cm7 F7 Dm7 G7 Cm7 F7 …` | B♭ major. G7 = V7/ii. Dm7 = iii. |
| `Dm7` for 16 bars | Modal: D Dorian. |
| `Cmaj7 Db7 Cmaj7` | Db7 → C labeled `subV7/I`, lydianDominant. |
| `Cmaj7 Fm7 Bb7 Cmaj7` | Backdoor ii–V. Fm7 = iv-7 (borrowed), Bb7 = bVII7, lydianDominant. |
| `Cmaj7 C#dim7 Dm7 G7` | C#°7 = passing diminished, also shown as A7b9. |
| `Cmaj7 Ebdim7 Dm7 G7` | E♭°7 = descending passing diminished (bIII°7). |
| `Cmaj7 Cm6` | C minor tonic recognized; Cm6 is not penalized. |

## 10. Content to author for v1

All content is **original** (no transcriptions of copyrighted solos) and validated by tests.

### 10.1 Progressions (written in roman numerals, playable in all 12 keys)

1. Long ii–V–I (2 bars per chord). The very first, slowest item.
2. Major ii–V–I (1 bar per chord, then 2 beats per chord)
3. Minor ii–V–i
4. I–vi–ii–V turnaround, and its I–VI7–ii–V variant
5. I–#I°7–ii–V (passing diminished)
6. iii–vi–ii–V–I
7. 12-bar blues (basic I7–IV7–V7)
8. Jazz blues (section 9 example)
9. Minor blues: `i-7 | iv-7 | i-7 | i-7 | iv-7 | iv-7 | i-7 | i-7 | bVI7 | V7alt | i-7 | ii-7b5 V7alt`
10. Rhythm changes, A section and bridge (III7–VI7–II7–V7)
11. ii–V–I with a tritone sub (ii–bII7–I)
12. Backdoor ii–V (iv-7–bVII7–I)
13. ii–V–I chain moving down in whole steps (Imaj7 → bVIImaj7 …), for practicing key changes
14. Cycle of ii–Vs in major plus the relative minor (an "Autumn Leaves"-type cycle)
15. Modal vamps: i-7 for 16 bars (Dorian), and a two-chord Dorian/Mixolydian vamp
16. Bossa-style i-7–iv-7 / ii-7b5–V7alt minor progression
17. I7–IV7 two-chord "funk/blues" vamp, as a bridge from the user's blues comfort zone to dominant chord-scales

### 10.2 Voicings

Curated sets for maj7, m7, 7, m7b5, dim7, m6, 6, 7alt, 7b9, 9, 13, and 7sus4 across the
families in section 8.1. That is about 150 shapes; the rest are generated.

### 10.3 Licks (target: 60 for v1)

| Context | Count | Concepts |
| --- | --- | --- |
| Blues I7 / IV7 / turnaround | 12 | Starts from the pentatonic boxes the user knows, adds the major 3rd, the 6th, and the 9th |
| V7 → I | 8 | Guide tones, the 3rd of V to the root of I, enclosures of the target 3rd |
| ii–V–I major | 12 | Arpeggio from the 3rd ("3-to-9"), bebop scale lines, chromatic approach, guide-tone lines |
| minor ii–V–i | 8 | b9 → 5, altered-scale fragments, diminished patterns |
| Turnaround I–VI7–ii–V | 6 | Secondary-dominant b9 sounds, chromatic approach |
| Rhythm changes A | 4 | Fast-moving ii–Vs, guide tones |
| maj7 / m7 static | 6 | Pentatonic superimposition (e.g. E minor pent over Cmaj7 = 3-5-13-7-9 (E G A B D)) |
| Minor blues / static minor | 4 | Dorian 6th, minor pent + 9 |

Each lick is 1–2 bars (phrase length, which suits SRS), lists `difficulty`, `concepts`, and a
`context` the app can find in any song. Every context has at least one enclosure lick.

### 10.3a Rhythmic cells (10 items)

Rhythm comes before note choice (Hal Crook, *How to Improvise*). Author 10 one- or two-bar
rhythm cells for the "Rhythm first" mode: start on the "and" of 1, start on beat 2, Charleston,
anticipating the change on the "and" of 4, 2-bar phrase / 2-bar rest, a long note then an eighth
run, triplet pickup, quarter-note triplets, displaced 3-note motif, and a call-and-response
pair.
Content tests require that every lick's notes have tier ≠ `avoid` on strong beats in their
context.

### 10.5 Tune list (bundled `Song` charts, easiest first)

Chord progressions are not copyrightable, so the app ships chord charts and titles only. Full
melodies ship **only** for tunes in the US public domain. The user can enter melodies for the
others.

| # | Tune | What it teaches | Melody bundled? |
| --- | --- | --- | --- |
| 1 | Tenor Madness / Now's the Time | B♭ / F jazz blues | no |
| 2 | Mr. P.C. | Minor blues | no |
| 3 | Summertime | Minor, 16 bars | no |
| 4 | Fly Me to the Moon | Diatonic cycle, ii–V–I | no |
| 5 | Autumn Leaves | Relative major/minor ii–Vs | no |
| 6 | Blue Bossa | Minor ii–V, modulation to D♭ | no |
| 7 | So What / Impressions | Dorian modal, half-step key shift | no |
| 8 | Sweet Georgia Brown (1925) | Cycle of dominants VI7–II7–V7 | yes (public domain) |
| 9 | Satin Doll | Chained, non-resolving ii–Vs | no |
| 10 | Take the A Train | II7 lydian dominant | no |
| 11 | Tune Up | ii–V–I down whole steps | no |
| 12 | Lady Bird | Backdoor iv–bVII7 | no |
| 13 | There Will Never Be Another You | iv-7, secondary dominants | no |
| 14 | I Got Rhythm (1930) / Oleo | Rhythm changes | I Got Rhythm: yes (public domain since 2026) |
| 15 | All the Things You Are | ii–V–Is through several keys | no |

Extra public-domain standards (US, published 1926–1930) that can ship with melodies: Bye Bye
Blackbird, Honeysuckle Rose, Ain't Misbehavin', Georgia on My Mind, Exactly Like You. Before
bundling any melody, check the publication year again: US public domain covers works published
95+ years ago.

### 10.4 Technique drills

**Warm-up first** (always block 1, 2–3 min): slow, easy scale fragments and light finger
movement with no stretch and no spider drills. Then a **minimum-pressure drill** (press until the
note just sounds, then relax; Jamie Andreas, *Principles of Correct Practice*).

Chromatic 1-2-3-4 permutations and spider exercises are useful but unmusical and high-repetition,
so they are **capped at about 3 minutes per session**. Musical drills in the current key are
preferred: arpeggios and scales through the day's progression. Also: one-finger-per-fret with a
`maxStretch` variant, legato trills (1-2, 1-3, 2-4, 3-4: weak-finger focus), alternate-picking
scale sequences (3s, 4s, 6ths), string skipping, and arpeggio sweeps (3-string triads). Each drill has `startBpm` /
`goalBpm` and is driven by the tempo ladder.

## 11. Songwriting: writing a new section that belongs

Goal: given one section of a song (say a verse), help write the next one (a pre-chorus, chorus
or bridge) so it sounds **different but related**. Everything here is rule-based and
deterministic (`analysis/songwriting.ts`). The same rules power the explanations the user reads,
so the app teaches *why* a section works, not just *what* to play.

### 11.1 Song forms

| Form | Shape | Where the contrast lives |
| --- | --- | --- |
| Verse–chorus | V C V C | Chorus is the payoff. It starts on a chord the verse avoided: often I after a verse that withheld it, or IV/vi after a verse that sat on I |
| Verse–pre-chorus–chorus–bridge | V P C V P C B C | Pre-chorus builds tension; bridge is the biggest departure, once |
| AABA (32-bar standard) | A A B A, 8 bars each | B (the bridge) leaves the tonic: often to IV (Gm7 C7 → F in C), to a new key, or through a chain of dominants (rhythm changes). It ends on V7 to set up the last A |
| ABAC (32-bar standard) | A B A C, 8 bars each | The second half starts like the first (A returns). B ends away from home (a half cadence or another key); C replaces it with the final cadence home. Example: Sweet Georgia Brown |
| 12-bar blues | One 12-bar chorus | AAB is the lyric form. The harmony does the work at bar 5 (IV), bar 9 (V) and the bar 11–12 turnaround |

### 11.2 The three questions every new section answers

1. **Unity: does it sound like the same song?** It keeps the same tonal center (or a closely
   related one), shares some chords or a bass/rhythm motif, and stays in the same style.
2. **Contrast: does it sound like a new place?** It changes at least one of: the starting chord,
   the chord vocabulary, the harmonic rhythm (how often chords change), the key area, the
   register or energy of the melody, or the phrase length.
3. **Transitions: do the joins work?** The last chord of the previous section should lead into
   the first chord of the new one (by a cadence, a common tone, or a stepwise bass line), and
   the new section's end should lead to whatever comes next.

Rule of thumb: a **chorus** is high unity and medium contrast, with a strong arrival. A
**pre-chorus** builds tension toward the chorus. A **bridge** is the biggest contrast in the
song and must find its way back.

### 11.3 Technique catalogue

Each technique is a generator (it produces candidate chords) and an explanation template.
Examples are in C major; the app shows them in the user's key.

| Id | Technique | Example | Typical role |
| --- | --- | --- | --- |
| `tonicArrival` | The verse avoids or delays I (starts on vi or IV, ends on V); the chorus starts on I, usually with the title. This "home and away" move is the most common pop contrast | Verse `Am F G G` → chorus `C G F C` | chorus |
| `newStartingChord` | Start on a chord the previous section avoided or only passed through, often IV or vi | Verse `C Am C G` (never uses F) → chorus `F G C C` | chorus |
| `relativeFlip` | Move the center of gravity between the relative minor and major | Verse `Am Em F G` (minor feel) → chorus `C G F C` | chorus |
| `parallelModeSwitch` | Same root, switch major ↔ minor | Verse in C minor → chorus in C major | chorus, bridge |
| `harmonicRhythm` | Change how often chords change: a busy verse and a broad chorus (or the reverse) | Verse 2 chords per bar → chorus 1 chord per 2 bars | any |
| `reorderChords` | Same chords, new order or new bass, so it's familiar yet fresh | Verse `C Am F G` → chorus `F C G Am` | chorus |
| `pedalPoint` | Hold one bass note (often 5) under changing chords to build tension | `F/G Dm/G C/G G` | prechorus |
| `climbingBass` | Step the bass upward into the chorus | `Dm Em F G` | prechorus |
| `borrowedChords` | Borrow iv, bVI, bVII or bIII from the parallel minor for color. Rock idioms: the double plagal `bVII IV I` and a mixolydian verse | `F Fm C`, `Ab Bb C`, `Bb F C`, verse `C Bb F C` | bridge, chorus, verse |
| `newKeyArea` | Center the bridge on IV, vi or iii (tonicize it with its own V7), then return via V7 | Bridge on IV: `Gm7 C7 \| F \| Fm \| G7`; on vi: `Am E7 \| Am \| Dm \| G7` | bridge |
| `cycleOfDominants` | Chain dominants around the cycle of 5ths back home. Uses only secondary dominants, so it is available from adventurousness 2 | Rhythm-changes bridge `E7 A7 D7 G7` | bridge |
| `secondaryDominantLeadIn` | Put the new section's first chord's own V7 at the end of the previous section | Verse ends `E7` → chorus starts `Am` | transition |
| `cadenceSetup` | End the verse/pre-chorus on V (a half cadence) so the chorus feels like an arrival; use V → vi (deceptive) to extend; end the bridge on V or V7sus to set up the return | Pre-chorus ends `G` → chorus starts `C` | transition |
| `liftModulation` | Final chorus up a half or whole step, prepared by the new key's V7 | `G7` → chorus in C, then `A7` → last chorus in D | final chorus |
| `jazzReharm` | New chords under the same melody: insert ii–Vs, tritone subs, passing diminished chords. The app doesn't know the melody, so the explanation names the chord tones that changed (A7 adds C♯) and says "check this against your melody" | `C Am Dm G` → `C A7 Dm7 Db7` | last A, variation |
| `phraseLength` | Change section length or phrase grouping (4 vs 8 bars, an extra 2-bar tag) | 8-bar verse, 4-bar pre-chorus | any |

**Minor keys.** Every technique has a minor-key version: i, bVI, bVII and V7 (from harmonic
minor) replace I, IV, V; the relative flip goes to III; borrowed chords come from the parallel
major.

**Melody and energy hints** (text shown with every suggestion; the app never generates
melodies):

- Make the chorus's highest note higher than the verse's (a new contour peak).
- Change the rhythm: busy short notes against long held notes, or start phrases on a different
  beat.
- Change phrase lengths.
- Put chord tones on strong beats and use the section's new chords to pick fresh target notes.
  This ties directly into the Changes Trainer's target-note skills.
- Change the guitar part's energy: palm-muted or low open chords in the verse, open strumming
  and higher barre or triad shapes in the chorus (within `maxStretchFrets`).

**Scope.** The Lab stays harmonic, with text hints. Melody generation, lyric or rhyme analysis,
AI-written text, and mic use are out of scope.

### 11.4 `suggestSection` algorithm

1. **Analyze the source** with §5: key, romans, harmonic rhythm, first and last chord function,
   root-motion profile, and the chord set used.
2. **Generate candidates.** Apply each technique allowed for the requested role. Add role and
   style templates from `content/songwriting` (pop, rock, blues, jazz). Respect `bars` (default:
   same length as the source) and `adventurousness`:
   - 1 = diatonic only;
   - 2 = plus borrowed chords, secondary dominants and the cycle of dominants;
   - 3 = plus new key areas, modulation and jazz reharm.
3. **Score each candidate** (0–1 each):
   - **Unity** = 0.6 × explainedShare + 0.4 × min(sharedChords, 3) / 3, where explainedShare is
     the share of chords diatonic or explainable in the song key (§5.2 emission rules). Being
     identical to the source scores 0.
   - **Contrast**:
     - a different first chord, compared by **roman degree** (not tonic/dominant function, or
       Am → C would earn nothing and `relativeFlip` would never score). A chorus starting on I
       after a verse with no I on a strong bar also counts: +0.35;
     - a different harmonic rhythm: +0.2;
     - a chord-set difference (Jaccard distance): up to +0.25;
     - a new key area, a borrowed chord, **or a chromatic dominant** (secondary dominant,
       tritone sub): +0.2.
   - **Transition in**: cadence strength from the source's last chord to the candidate's first,
     averaged with voice-leading smoothness between the two chords.

     | Join | Strength |
     | --- | --- |
     | V(7) → I; any V7/x → x | 1.0 |
     | bII7 → x | 0.9 |
     | V → vi (deceptive) | 0.8 |
     | IV → I, iv → I, bVII → I; V → IV (rock) | 0.6 |
     | Common tone or stepwise bass | 0.5 |
     | Same chord across the boundary | 0.3 |
     | Otherwise | 0.2 |

     Voice-leading smoothness = 1 − min(total semitone motion over the nearest chord-tone
     pairing, 12) / 12.
   - **Transition out**: the same, from the candidate's last chord to the next section's first
     chord (for a bridge: back to the A/verse/chorus start). If the next section doesn't exist
     yet, assume it starts on I.
4. **Role weights** for `total`:

   | Role | Unity | Contrast | Transition in | Transition out |
   | --- | --- | --- | --- | --- |
   | chorus | 0.35 | 0.25 | 0.3 | 0.1 |
   | prechorus | 0.35 | 0.15 | 0.15 | 0.35 (must lead into the chorus) |
   | bridge | 0.2 | 0.4 | 0.1 | 0.3 (must find its way back) |
   | verse / A | 0.45 | 0.15 | 0.2 | 0.2 |

5. Return the **top 5**, deduplicated (no two candidates with the same chords), each with its
   techniques and 2–4 explanation sentences. Break ties in favor of candidates with open or easy
   voicings in the current key. For example: *"This chorus starts on F (IV), a chord your verse
   never used, so it feels like arriving somewhere new. Your verse
   ends on G (V), which pulls strongly to C; here it lands on F instead, which surprises the
   ear. The chorus ends on G so it can lead straight back into the verse."*
6. **Deterministic**: the same input and seed always give the same output, so tests can pin
   results.

### 11.5 Supporting functions

- `explainTransition(a, b, key)`: describes how one section hands off to the next (cadence type,
  common tones, bass motion) and suggests a fix if the join is weak. For example: "Add D7 at the
  end of the verse to lead into the G chorus."
- `compareSections(a, b)`: flags sections that are too similar ("same chords and same starting
  chord") and suggests a contrast technique such as `newStartingChord`, `tonicArrival` or
  `harmonicRhythm`.
- `chordsThatFit(section, bar, key)`: ranked replacement chords for one bar (diatonic choices,
  substitutes sharing 2+ notes, secondary dominants of the next chord, borrowed chords), each
  with a one-line reason.

### 11.6 Golden examples (required tests)

| Input | Expected among the top 5 |
| --- | --- |
| Verse `C G Am F` (2 bars each), role chorus, adv. 1 | A candidate starting on F or Am (`newStartingChord`); no candidate identical to the verse |
| Verse `C Am C G`, role chorus, adv. 1 | A candidate starting on F ranks in the top 3 |
| Verse `Am F G G`, role chorus | A candidate starting on C (`tonicArrival`) ranks in the top 2 |
| Verse `Am F C G`, role chorus | A `relativeFlip` candidate centered on C (C is the first chord and has the most bars) |
| Verse `Am G F E7` (A minor), role chorus | A candidate using the minor-key techniques, e.g. starting on C (III) or F (bVI) |
| Verse `C Am F G`, role prechorus | A candidate ending on G (`cadenceSetup`), e.g. `Dm Em F G` (`climbingBass`) |
| Verse `C F C G`, role bridge, adv. 3 | A `newKeyArea` (F or Am) or `cycleOfDominants` candidate whose last chord is G or G7 |
| AABA in B♭ with A = rhythm-changes A, role bridge, adv. 2, 8 bars, next section = A | Bridge `D7 G7 C7 F7` ranks first or second |
| Verse `C G Am F` → chorus `C G Am F` (user-entered) | `compareSections` flags "no contrast: same chords and same starting chord" and suggests `newStartingChord` or `harmonicRhythm` |

### 11.7 Lessons (`content/songwriting`)

Eight short lessons. Each has an explanation, audio examples, and a "try it" step that opens the
Songwriting Lab preloaded:

1. Sections and form: verse, chorus, pre-chorus, bridge, AABA.
2. Unity and contrast: the three questions. Includes one prosody tip: put the title on the
   chorus's most stable chord and strongest beat; let unresolved lyrics sit on unstable chords
   (V, IV).
3. Start somewhere new: IV and vi chorus openings.
4. Harmonic rhythm: busy versus broad.
5. Building tension: pre-chorus pedal points and climbing bass lines.
6. Cadences and transitions: half, authentic and deceptive cadences, secondary-dominant
   lead-ins.
7. Bridges: new key areas, borrowed chords, the cycle of dominants, and finding the way home.
8. Real-song study, using the bundled tunes (§10.5):
   - the cycle-of-dominants bridge of I Got Rhythm;
   - Sweet Georgia Brown, where the cycle is the A section and B contrasts in the relative minor;
   - the move to D♭ in the second half of Blue Bossa;
   - the Lady Bird backdoor (Fm7–B♭7 → Cmaj7) and Tadd Dameron turnaround.
