# Features and Screens

The app has a bottom tab bar: **Today · Fretboard · Changes · Licks · Songs · More**. The Songs
tab holds the progression and voicing library, tunes, your own songs, and the Songwriting Lab.
Every practice screen shares the same `<TransportBar>` (play, tempo, loop, key, mix) at the
bottom so the controls are reachable with one thumb.

Each feature below lists the goal it serves (G1–G7 from the README), what it does, and its
MVP scope.

## Where the mic is used

The mic is optional everywhere and follows the UX principles in ARCHITECTURE §6.3: zero setup,
nothing interrupts the music, and feedback says what to change. It is used only where it answers
a question that is hard to judge by ear while playing:

| Where | The question it answers |
| --- | --- |
| Changes Trainer: target-note game, guide-tone line, arpeggios, approach patterns | "Did I actually land on the chord tones when the chord changed, and where did I miss?" |
| Changes Trainer: rhythm first | "Am I rushing or dragging, and am I really starting phrases off the beat?" |
| Fret Quiz: find it | "Which notes on the neck am I slow or wrong on?" |
| Lick Trainer: play along and 12 keys | "Did I play it right in the new key?" |
| Technique tempo ladder (optional) | "Was that run clean enough to speed up?" (with the mic off, you self-rate) |
| Ear: sing it (optional) | "Was I singing the right note?" |
| Tuner | Tuning |

There is no mic grading of bends, vibrato or tone. Your ear judges those better, so those
trainers were dropped.

---

## F0. Learning Path (all goals)

The skeleton of the course. Without it the app would be a toolbox, not a teacher. The units are
ordered, and each has prerequisites and pass criteria (`CurriculumUnit`, ARCHITECTURE §5). The
approach is **arpeggio-first**: chord tones are the skeleton and scales are the fill (Levine,
*The Jazz Theory Book*; Bert Ligon, *Connecting Chords with Linear Harmony*).

| Unit | Content | Pass criteria (examples) |
| --- | --- | --- |
| 1 | Fretboard notes on strings 6 and 5, then all strings; octave shapes | 90% of cells correct in under 1.5 s |
| 2 | Triads on 3-string sets; 1-3-5-7 arpeggios in one position, built around the familiar pentatonic boxes | Arpeggios of I, ii, V7 clean at 80 BPM in 2 positions |
| 3 | Chord-tone soloing on the blues: arpeggios through I7 / IV7 / V7, then mixed with minor pentatonic | Target-note game ≥ 80% on a 12-bar blues |
| 4 | Guide tones and shells on ii–V–I and the jazz blues, played and comped | Guide-tone line ≥ 90% in 4 keys; shells comped through a jazz blues |
| 5 | Approach notes and enclosures onto chord tones | Approach drill ≥ 80% on a ii–V–I |
| 6 | Bebop scale with chord tones on the beat | Bebop-rule drill ≥ 80% |
| 7 | Minor ii–V; b9 sounds; altered and diminished colors | Target-note game ≥ 80% on a minor ii–V–i |
| 8 | Tunes (THEORY_SPEC §10.5) through Tune Study | One solo chorus per tune at ≥ 75% |

**Scale gating.** Until unit 7, Scale-of-the-moment shows only chord tones plus the parent-key
scale. The altered, half-whole diminished and lydian dominant alternatives unlock at unit 7.
Showing every scale early overloads a player who is still learning to hear the changes.

Units can be revisited anytime, and SRS keeps reviewing earlier material.

## F1. Today: the practice session (all goals)

- One tap starts a planned session (default 30 min) built by `practice/planner`. It is a
  sequence of timed blocks, each of which opens the matching feature with the right item
  preloaded. For example: *"2 min: gentle warm-up → 3 min: legato 3-4 at 72 BPM → 4 min: find
  every B♭ → 8 min: guide tones over a minor ii–V in G → 5 min: lick #23 in E♭ → 4 min: sing
  and name the key → 4 min: free play over Blue Bossa."*
- Every session starts with a gentle warm-up and ends with free play over the current tune.
  Block weights and rest prompts are defined in ARCHITECTURE §6.4.
- Items appear as one blocked block on first exposure and in short interleaved slices after
  that.
- Shows the streak, minutes this week, and the key of the day (rotating through the cycle of
  4ths).
- **MVP**: a fixed template session, then SRS-driven selection once Phase 6 is done.

## F2. Fretboard Explorer (G1)

- Pick a root and a scale, arpeggio, or chord. The fretboard shows every note, labeled by note
  name **or** degree (toggle).
- Choose a position system: all notes, a 4-fret position (default), CAGED shape 1–5, single
  string (Goodrick-style), or 3NPS pattern 1–7. 3NPS is optional and hidden when its patterns
  exceed your max stretch. The **pentatonic-box bridge** overlays the familiar pentatonic box, then adds the
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
| **Rhythm first** | Improvise on just 1–3 notes while following rhythm prompts: start on the "and" of 1, play a 2-bar phrase then rest 2 bars, Charleston, anticipate the change. Uses the rhythm cells in THEORY_SPEC §10.3a. | ✓ (rhythm and feel) |
| **Arpeggios through the changes** | Play the chord tones of each chord in one position, ascending or descending, without leaving position. Variant: **upper-structure triads** (e.g. a Dm triad over G7 = 5-b7-9). They use fewer notes and smaller spans, and are an easy step from pentatonic phrasing. | ✓ |
| **Approach patterns** | Approach the 3rd/5th/7th of every chord: chromatic from below, scale step from above, double chromatic, and a Barry Harris-style enclosure. Bebop-rule variant: descend from a chord tone on the downbeat so chord tones fall on the beats. | ✓ |
| **Guide-tone line** | Play only the 3rd/7th of each chord as a whole note, moving to the nearest one. Then half notes, then embellished. | ✓ |
| **Target-note game** | Improvise freely, but **land on a chord tone on beat 1** of every chord change. An anticipation of up to an eighth note counts. Score = % of changes hit. Starts with 1 chord per 2 bars, then 1 per bar. | ✓ |
| **Scale-of-the-moment** | Shows the chord-scale for each chord with the "why", e.g. "A7 → Dm: phrygian dominant, C# is the leading tone". | — |
| **Pentatonic switch** | For blues/rock comfort: shows which pentatonic fits each chord (e.g. over a ii–V–I in C: D minor pent → G major pent (or D♭ major pent for the altered sound) → C major pent). A bridge from what the player already knows. | ✓ |

After each loop the app shows a timeline of what you played, color-coded by tier per beat, so
you can see where you missed the changes. Rhythm and time feel (swing ratio, ahead/behind) are
reported separately from note choice.

### F4a. Tune Study

For each tune in THEORY_SPEC §10.5, or any user song:

1. Hear the form and learn the melody (when one is available: a public-domain tune or a melody
   you entered).
2. Play the roots with the backing track.
3. Comp shells through it.
4. Play the guide-tone line.
5. Play arpeggios through the changes.
6. Target-note game.
7. Play a full solo chorus.

Each step is graded and unlocks the next.

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

- **Voicing browser**: pick a chord quality and a family (shell → drop 2 → drop 3 → rootless;
  also triads, quartal, the "So What" chord, and Barry Harris 6th-diminished).
  See chord diagrams for all inversions and string sets, with degree labels on each dot. Tap to
  strum; a "max stretch" filter hides shapes that are too wide.
- **Progression library**: the progressions in THEORY_SPEC §10.1, each with teaching notes,
  roman numerals, audio, and a "play in key ___" option.
- **Voice-leading view**: shows a progression with voicings chosen for smooth motion and lines
  connecting each voice. Toggle "show the 7th → 3rd resolutions". Cycles through 12 keys.
- **Comping practice**: mute the app's comping and play along to the drums and bass. The chart
  shows which voicing to grab next. Chords aren't graded in v1, so each comping rhythm (Freddie
  Green, Charleston, anticipation, bossa) is a self-rated SRS item.
- **6th-diminished movement** (Barry Harris): drop 2 C6 alternating with B°7 along the scale.
  It sounds full while needing little stretch.
- **Songs**: the bundled tune charts (THEORY_SPEC §10.5) plus your own charts (a form with A/B
  sections, optionally with a melody). Songs then work in every other feature, including Tune
  Study.

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
  6. **Vary it**: change the rhythm, displace it by a beat, truncate it, or start it from a
     different note. Varying a lick is how it becomes your own language.
  7. "Use it": the Changes Trainer plays a song and flashes each place where the lick's
     context occurs.
- **My Licks**: add your own lick by tapping notes on the fretboard (as tab) or by playing it
  into the mic (monophonic capture, then rhythm quantization), and tag its context.
- **Lick-of-the-day** and SRS: every learned lick comes back on a schedule in a rotating key, so it
  stays ready to use.

## F8. Technique and Dexterity (G6)

- Drill cards (THEORY_SPEC §10.4) with a big metronome, the pattern on the fretboard or tab,
  and finger numbers.
- **Tempo ladder**: tempo goes up only after clean runs. With the mic on, the app judges
  "clean" (note accuracy and timing evenness). With the mic off, you tap clean / almost /
  sloppy. A graph shows the best clean BPM over time for each drill.
- **Weak-finger focus**: drills that favor fingers 3 and 4.
- **Gentle mode (on by default)**:
  - shorter sets, rest timers, and a 2–5 min break prompt every 20–25 min;
  - relaxation checks for thumb, shoulder and jaw;
  - a "stop if pain, numbness or loss of control" notice;
  - spider and chromatic drills capped at about 3 min per session;
  - max stretch increases at most one step per week.
- **Warm-up and minimum-pressure drills** always come first: press until the note just sounds,
  then relax.

## F9. Ear, Singing and Transcription (G2, G4)

Ear training, singing and transcription are central to how jazz is learned (Berliner, *Thinking
in Jazz*), so this is a full feature.

- **Recognition**: hear a cadence, then name the tonic; hear a chord, then name its quality
  (maj7 / m7 / 7 / m7b5); hear a chord change, then name the new chord's function.
- **Sing it**: sing the guide-tone line over the backing track (the mic grades your voice), then
  play it. Sing the root of I after a ii–V.
- **Sing-then-play call and response**: the app plays a short phrase; you sing it back, then
  play it.
- **Transcribe a phrase**: import an audio file you own and get a loop, a slowdown that keeps
  the pitch, and a pitch hint for the note under the cursor. The audio is processed on the
  device and never stored, the same rule as the mic.

## F10. Progress

- Minutes per goal area, streak, best tempos, fretboard heatmap, licks known per key, and
  Changes Trainer accuracy over time.
- Export/import a backup JSON file.

## F11. Tuner and Settings (More tab)

- Chromatic tuner (reuses the pitch pipeline).
- Settings: tuning, left-handed, fretboard orientation, note names vs degrees, sharps/flats,
  max stretch, mic calibration, daily goal, and theme.

## F12. Songwriting Lab (G7)

Write songs with the same harmony knowledge the rest of the app teaches, and learn **how to
write a new section** (verse → pre-chorus → chorus → bridge) that is different from the first
one but still belongs to the same song. The rules are in THEORY_SPEC §11.

- **Sketch a song.** Add sections (Verse, Pre-chorus, Chorus, Bridge, or A/B) and type or tap in
  chords for the first one. The app shows the key and roman numerals as you go (reusing the Key
  Finder analysis).
- **"Suggest a chorus / pre-chorus / bridge".** Get up to 5 options for the next section. Each
  shows:
  - the chords (in your key) and the techniques used, such as "starts on IV", "relative-minor
    flip", "climbing bass" or "borrowed bVII";
  - 2–4 plain-English sentences on **why it works**: what stays the same (unity), what changes
    (contrast), and how it connects to the sections around it;
  - an **adventurousness** slider: 1 = stays in the key; 2 = adds borrowed chords and secondary
    dominants; 3 = new key areas, modulation and jazz reharmonization.
- **Audition.** Tap a suggestion to hear it in context: the backing track loops verse →
  suggestion → verse in your chosen style, so you judge it with your ears.
- **Edit one chord at a time.** Tap a bar to see "chords that fit here", ranked, each with a
  one-line reason.
- **Check the joins.** "Explain this transition" describes how one section hands off to the next
  and suggests a fix if the join is weak. For example: "Add D7 at the end of the verse to lead
  into the G chorus." It also flags sections that are too similar: "your chorus uses the same
  chords as the verse starting on the same chord; try starting on IV or changing how often the
  chords change."
- **Melody hints.** For each section: put chord tones on strong beats, let the chorus sit higher
  with longer notes, and use the new section's chords to pick fresh target notes. These are the
  same skills as the Changes Trainer.
- **Lessons.** Eight short lessons (THEORY_SPEC §11.7), ending with a study of how real tunes
  from the tune list build their bridges.
- **Connected to everything.** A sketch is saved as a song, so you can solo over your own song in
  the Changes Trainer, analyze it in the Key Finder, and practice it in Tune Study.

The mic is not used here.
