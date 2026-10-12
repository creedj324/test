# Features and Screens

The app has a bottom tab bar: **Today · Fretboard · Changes · Licks · Library · More**.
Every practice screen shares the same `<TransportBar>` (play, tempo, loop, key, mix) at the
bottom so the controls are reachable with one thumb.

Each feature below lists the goal it serves (G1–G6 from the README), what it does, and its
MVP scope.

---

## F1. Today: the practice session (all goals)

- One tap starts a planned session (default 30 min) built by `practice/planner`. It is a
  sequence of timed blocks, each of which opens the matching feature with the right item
  preloaded. For example: *"5 min: spider drill at 72 BPM → 5 min: find every B♭ → 8 min:
  guide tones over a minor ii–V in G → 7 min: lick #23 in E♭ → 5 min: name the key."*
- Shows the streak, minutes this week, and the key of the day (rotating through the cycle of
  4ths).
- **MVP**: a fixed template session, then SRS-driven selection once Phase 6 is done.

## F2. Fretboard Explorer (G1)

- Pick a root and a scale, arpeggio, or chord. The fretboard shows every note, labeled by note
  name **or** degree (toggle).
- Choose a position system: all notes, CAGED shape 1–5, 3NPS pattern 1–7, or a 4-fret
  position. The **pentatonic-box bridge** overlays the familiar pentatonic box, then adds the
  2 missing notes for the major scale, then the arpeggio: learning from what the player
  already knows.
- Tap a note to hear it. "Play" runs the scale or arpeggio up and down in the chosen position
  with a moving cursor.
- Also shows octave shapes and "the same note in 5 places".

## F3. Fret Quiz: note-finding trainer (G1, uses the mic)

- **Name it**: a dot appears and you tap the note name. Timed.
- **Find it**: "Play every G on the neck, low to high", graded by the mic. Uses any octave per
  string; the app tracks which positions you found.
- **Degree mode**: "Key of A: play the b7, the 3rd, and the 9th in position 5".
- Each fretboard cell (string × fret) is an SRS item, so slow or wrong cells come back more
  often. A heatmap of response time per cell shows weak zones (typically frets 5–10 on the
  G/B strings).

## F4. Changes Trainer (G2), the core feature

Pick a progression (from the library, a song, or one you typed in), a key, a tempo, and a
style. The backing track loops while the fretboard follows the changes.

| Mode | What you see / do | Graded by mic |
| --- | --- | --- |
| **Visualize** | Fretboard shows the current chord's tones colored by tier. On beat 4 the next chord's guide tones fade in as a ghost preview. Position lock keeps it within one hand position. | — |
| **Arpeggios through the changes** | Play the chord tones of each chord in one position, ascending or descending, without leaving position. | ✓ |
| **Guide-tone line** | Play only the 3rd/7th of each chord as a whole note, moving to the nearest one. Then half notes, then embellished. | ✓ |
| **Target-note game** | Improvise freely, but **land on a chord tone on beat 1** of every chord change. Score = % of changes hit. Starts with 1 chord per 2 bars, then 1 per bar. | ✓ |
| **Scale-of-the-moment** | Shows the chord-scale for each chord with the "why", e.g. "A7 → Dm: phrygian dominant, C# is the leading tone". | — |
| **Pentatonic switch** | For blues/rock comfort: shows which pentatonic fits each chord (e.g. over a ii–V–I in C: D minor pent → G major pent (or D♭ major pent for the altered sound) → C major pent). A bridge from what the player already knows. | ✓ |

After each loop the app shows a timeline of what you played, color-coded by tier per beat, so
you can see where you missed the changes.

## F5. Key Finder and Analyzer (G4)

- **Input** a progression by:
  - typing or pasting text (`Dm7 G7 | Cmaj7 | A7 |`), with live parse errors;
  - a fast chord picker: a root wheel plus quality chips;
  - opening a saved song.
- **Output**:
  - the likely key(s) with confidence;
  - **key regions** drawn as colored bands over the chart, with modulations marked;
  - a roman numeral under every chord, with ii–V brackets and arrows for resolutions,
    secondary dominants, tritone subs, and backdoor;
  - for each chord, ranked scales with a one-line "why" and a tap-through to the fretboard;
  - **"one scale for many bars"** suggestions: the longest stretches where one scale works
    (e.g. bars 1–4: B♭ major).
- **"Practice this"** opens the Changes Trainer with this progression.
- **Key-finding drill** (SRS): the app shows (or plays) a 4-chord progression and you answer
  the key and roman numerals. The difficulty rises from diatonic progressions to secondary
  dominants and then modulations.

## F6. Voicings and Progressions Library (G3)

- **Voicing browser**: pick a chord quality and a family (shell → drop 2 → drop 3 → rootless).
  See chord diagrams for all inversions and string sets, with degree labels on each dot. Tap to
  strum; a "max stretch" filter hides shapes that are too wide.
- **Progression library**: the progressions in THEORY_SPEC §10.1, each with teaching notes,
  roman numerals, audio, and a "play in key ___" option.
- **Voice-leading view**: shows a progression with voicings chosen for smooth motion and lines
  connecting each voice. Toggle "show the 7th → 3rd resolutions". Cycles through 12 keys.
- **Comping practice**: mute the app's comping and play along to the drums and bass. The chart
  shows which voicing to grab next (no grading of chords in v1).
- **Songs**: enter your own chord charts (a form with A/B sections). Songs then work in every
  other feature.

## F7. Lick Library and Lick Trainer (G5)

- Browse by context (blues, V7→I, ii–V–I, minor ii–V, static), by concept (enclosure,
  guide-tone, bebop, arpeggio, pentatonic superimposition), and by difficulty.
- **Lick player**: tab plus fretboard with a playback cursor; loop, slow down to 50%, and A/B
  loop a segment. Fingering comes from the optimizer and respects your max stretch. Choose a
  position to see it re-fingered.
- **Learn flow** for a lick:
  1. Listen
  2. Play along slowly (mic-graded note by note)
  3. Play it alone in the backing track's gap (call and response)
  4. Same lick in 3 new keys
  5. All 12 keys around the cycle of 4ths
  6. "Use it": the Changes Trainer plays a song and flashes each place where the lick's
     context occurs.
- **My Licks**: add your own lick by tapping notes on the fretboard (as tab) or by playing it
  into the mic (monophonic capture, then rhythm quantization), and tag its context.
- **Lick-of-the-day** and SRS: every learned lick (per key) comes back on a schedule so it stays
  ready to use.

## F8. Technique and Dexterity (G6)

- Drill cards (THEORY_SPEC §10.4) with a big metronome, the pattern on the fretboard or tab,
  and finger numbers.
- **Tempo ladder**: the mic checks note accuracy and evenness (timing deviation in ms). Tempo
  goes up only after clean runs. A graph shows the best clean BPM over time for each drill.
- **Weak-finger focus**: drills that favor fingers 3 and 4.
- **Gentle mode**: shorter sets with rest timers. Warns if a session is long without a break
  (to avoid strain).
- **Bend and vibrato trainer**: a live pitch meter shows the bend target and the vibrato
  oscillation.

## F9. Ear and Key Recognition (G4 support, light in v1)

- Hear a cadence, then name the tonic; hear a chord, then name its quality (maj7 / m7 / 7 /
  m7b5); hear a ii–V–I and sing or play the root of I (mic-graded).
- Kept small in v1; it supports key-finding by ear while soloing.

## F10. Progress

- Minutes per goal area, streak, best tempos, fretboard heatmap, licks known per key, and
  Changes Trainer accuracy over time.
- Export/import a backup JSON file.

## F11. Tuner and Settings (More tab)

- Chromatic tuner (reuses the pitch pipeline).
- Settings: tuning, left-handed, fretboard orientation, note names vs degrees, sharps/flats,
  max stretch, mic calibration, daily goal, and theme.
