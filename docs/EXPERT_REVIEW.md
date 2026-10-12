# Expert Review of the Course Material

A reviewer agent checked the theory and teaching plan against published sources. It played
three roles: a jazz theorist, a jazz/blues guitar teacher, and a music-learning scientist.
This file records what it found and what changed. The fixes are already applied to the other
docs; this is the audit trail.

## Sources consulted

- Mark Levine, *The Jazz Theory Book* and *The Jazz Piano Book*: chord-scale theory, rootless
  voicings, diminished spellings, repertoire.
- Berklee chord-scale and avoid-note conventions: [Wikipedia: Avoid note](https://en.wikipedia.org/wiki/Avoid_note),
  [HubGuitar: chord tensions](https://www.hubguitar.com/music-theory/chord-tensions).
- Lydian dominant usage: [jazzguitar.be forum](https://www.jazzguitar.be/forum/theory/17450-lydian-dominant-scale.html).
- Barry Harris 6th-diminished: [Wikipedia: Bebop scale](https://en.wikipedia.org/wiki/Bebop_scale),
  [Barry Harris notes](https://irfu.cea.fr/Pisp/frederic.galliano/Zique/barry_harris.html).
- Bert Ligon, *Connecting Chords with Linear Harmony*: arpeggio-first improvisation.
- Hal Crook, *How to Improvise*: rhythm-first improvisation.
- Mick Goodrick, *The Advancing Guitarist*: single-string study.
- Paul Berliner, *Thinking in Jazz*: ear, singing and transcription in jazz learning.
- Jamie Andreas, *Principles of Correct Practice*: minimum-pressure technique.
- Carter & Grahn (2016), [interleaved practice in musicians](https://pmc.ncbi.nlm.nih.gov/articles/PMC4989027/);
  Shea & Morgan (1979), contextual interference.
- Guadagnoli & Lee (2004): challenge-point framework.
- Frucht, [musician's dystonia review](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5502053/);
  [2023 pianist study](https://link.springer.com/article/10.1007/s00702-023-02689-4).
- Public domain status of *I Got Rhythm* (1930):
  [NPR](https://www.npr.org/transcripts/nx-s1-5609295), [Wikipedia](https://en.wikipedia.org/wiki/I_Got_Rhythm).

## A. Factual errors fixed (THEORY_SPEC)

| # | Problem | Fix |
| --- | --- | --- |
| 1 | The minor-key collection lacked melodic minor, so the m6 tonics in two golden tests (`Cm6`, `Gm6`) would be penalized in their own key | §5.2: minor = natural ∪ harmonic ∪ melodic; i-6 and i-mMaj7 count as tonic |
| 2 | The ii–V rule ran before the backdoor rule, so Fm7–B♭7–C would be labeled "ii–V of E♭" | §5.3: backdoor and tritone sub now come first |
| 3 | The passing-diminished rule missed the jazz blues B°7 (B♭7 → B°7 → F7) | §5.3: rule redefined around half-step root motion, plus #iv°7 → I |
| 4 | V7b9 → minor contradicted itself (half-whole diminished vs phrygian dominant) | §4.1: split by target; function rows take precedence over quality rows |
| 5 | IV7 in a blues was assigned lydian dominant (#11), but blues language uses the ♮4 | §4.1: removed from the lydian dominant row |
| 6 | The altered scale was spelled two conflicting ways | §2: one spelling, `1 b9 #9 3 #11 b13 b7`; spelling exceptions documented |
| 7 | Whole-half diminished was spelled with #5 | §2: spelled with b6, matching the dim7 chord tones |
| 8 | The avoid-note list was incomplete (iii-7's b6, m7b5's b2), and the Dorian 6th was not addressed | §3.2 updated; blue notes get the tension tier in a blues |
| 9 | `C°` was parsed as a triad, and several common symbols were missing | §3.1: ° defaults to dim7; added maj7#11, m9, 13sus, etc.; `6/9` is never read as a slash chord |
| 10 | m7b5 shells without the b5 sound the same as m7; a shell string set was missing | §8.1 fixed |
| 11 | Dominant rootless voicings were under-specified | §8.1: A = 3–13–b7–9, B = b7–9–3–13 (Levine) |
| 12 | The "So What" chord was described as stacked 4ths; it is three 4ths plus a major 3rd | §8.1: separate shape |
| 13 | The phrygian dominant tip was imprecise | §4.1 reworded |
| 14 | Golden examples were ambiguous (no bar lines, an undecided expected key) | §9: bar lines, a single expected key, and new tests for the blues override and for passing diminished chords |

The reviewer also confirmed these as correct: the other scale formulas, the guide-tone answer,
the jazz-blues functions, C♯°7 = A7b9, the drop 2 / drop 3 definitions, the pentatonic-switch
examples, and E minor pentatonic over Cmaj7.

## B. Teaching changes

| Priority | Change | Where |
| --- | --- | --- |
| High | **Learning Path**: 8 ordered units with mastery gates, arpeggio-first; advanced scales hidden until unit 7 | FEATURES F0, ROADMAP WP18 |
| High | **Rhythm and phrasing**: a "Rhythm first" mode, 10 rhythm cells, and time-feel reporting | FEATURES F4, SPEC §10.3a |
| High | **Tunes**: a 15-tune list plus a Tune Study flow | SPEC §10.5, FEATURES F4a |
| High | **Ear, singing and transcription** promoted to a full feature | FEATURES F9, ROADMAP WP19 |
| High | **The grader accepts anticipations** of up to an eighth note | ARCHITECTURE §6.3 |
| Medium | Approach-note, enclosure and bebop-rule drills | FEATURES F4 |
| Medium | Barry Harris 6th-diminished scale and voicings | SPEC §2, §8.1; FEATURES F6 |
| Medium | Position playing is the default system; 3NPS is optional and filtered by max stretch; single-string study added | SPEC §6 |
| Medium | Upper-structure triads over changes | FEATURES F4 |
| Medium | Progression list revised (long ii–V first, passing diminished, I–VI7–ii–V, minor blues spelled out) | SPEC §10.1 |
| Medium | Licks are 1–2 bars, with new turnaround and rhythm-changes contexts, an enclosure lick in every context, and a "Vary it" step | SPEC §10.3, FEATURES F7 |
| Low | Spider and chromatic drills capped at about 3 min | SPEC §10.4 |
| Low | Comping rhythms become self-rated SRS items | FEATURES F6 |

## C. Practice-science changes (ARCHITECTURE §6.4)

- **Blocked, then interleaved**: an item is practiced in one block when it is new, then in
  short mixed slices once it reaches 3 reps.
- **One SRS item per lick**, with a rotating key, not one item per key. Fretboard cells are
  graded on response time.
- **Tempo ladder**: 3 clean runs (≥ 95% correct notes, timing SD under 25 ms) before going
  faster; restart at 80% of the last clean tempo; some runs vary the tempo; about 8 attempts
  per drill per day.
- **Session shape**: a gentle warm-up first and free play over a tune last; ear training and
  singing get their own time.
- **Health guardrails on by default**: break prompts, relaxation checks, a pain notice, max
  stretch raised at most one step a week, and weekly practice growth capped at about 15%
  (focal-dystonia risk factors).

## D. Mic grading changes (ARCHITECTURE §6.3)

- Tensions score 0.9 in free improvisation and 0.7 in the target-note game.
- An avoid note scores 0 only when it is held. One that resolves by step is a suspension and
  counts as neutral.
- Blues blue notes score as tensions.
- Bends are graded on the pitch they settle at.
- Rhythm and time feel are reported separately from note choice.
