# Suno Field Guide

A community-compiled reference for getting consistent, professional results out of Suno AI. Covers everything from how Suno actually processes prompts under the hood, to song structure and meta tag syntax, to post-processing and mastering finished tracks. Synthesized from field testing, community guides, official Suno documentation, and Grenar's Dirty Tricks series.

This is not a beginner tutorial — it assumes you have used Suno at least a little and want to go deeper. Start with Section 1 if you want the mental model, or jump to whatever section is relevant to the problem you're trying to solve.

---

## Table of Contents

1. [Mental Model — What Suno Actually Is](#1-mental-model)
2. [Model Selection](#2-model-selection)
3. [Prompt Engineering](#3-prompt-engineering)
4. [Song Structure & Meta Tags](#4-song-structure--meta-tags)
5. [Style Field — Genre Clouds & Tag Strategy](#5-style-field)
6. [Lyric Writing for Suno](#6-lyric-writing-for-suno)
7. [Realism & Production Quality](#7-realism--production-quality)
8. [Advanced Parameters](#8-advanced-parameters)
9. [Advanced Techniques](#9-advanced-techniques)
10. [Grenar's Dirty Tricks — Lyric-Level Control](#10-grenars-dirty-tricks)
11. [Album Art & Thumbnails](#11-album-art--thumbnails)
12. [Quick Reference](#12-quick-reference)
13. [AI Lyric Anti-Patterns](#13-ai-lyric-anti-patterns)
14. [Sources](#14-sources)
15. [Contributing](#15-contributing)

---

## 1. Mental Model

**Suno is not reading your prompt like a human.** It is mapping your text into a probabilistic style-mesh, blending co-occurring musical concepts from training data, and defaulting toward statistically dominant "gravity wells" unless you actively constrain it.

**Suno is not:**
- Obeying instructions in hierarchical order
- Generating "pure" genres in isolation
- Interpreting language literally

**Suno is:**
- Mixing styles based on co-occurrence patterns in training data
- Pulling toward defaults (especially pop) unless you push back
- Performing soft classification between *conditioning text* (your prompt) and *performable text* (lyrics it will sing)

### The Pop Gravity Well

Nearly every genre gravitates toward "pop" mixing structure by default:
- Rock ↔ Pop: 315 billion statistical links
- Funk ↔ Pop: 116 billion links
- Emo ↔ Pop: 12.2 billion links

Unless you explicitly exclude pop or use strategic countermeasures, your track will likely incorporate pop hooks regardless of stated genre.

### Genre Clouds

Suno's musical universe clusters into tight groups that are nearly inseparable:

| Cloud | Tangled Tags |
|-------|-------------|
| Rap Cloud | rap, trap, bass, hip-hop, beat |
| Orchestral Cloud | orchestral, epic, cinematic, dramatic, piano |
| Indie Cloud | indie, pop, acoustic, dreamy, psychedelic |
| Dark Electronic | dark, synth, electro, synthwave, futuristic |

Asking for one element in a cloud tends to pull in the others. Use this strategically.

### Escaping Genre Gravity

1. **Explicit exclusions** — tell Suno what you don't want
2. **Force weird combinations** — "emo industrial," "orchestral phonk," "math rock gospel" push the model into territory where defaults don't apply
3. **Strategic contrast** — emphasize elements that naturally repel what you're avoiding

### Weak vs. Strong Tags

**Strong tags** (will overpower other instructions): `pop`, `rock`, `electronic`

**Weak tags** (need reinforcement): `grunge`, `math rock`, `swing`

When combining weak + strong, the strong tag wins unless you actively counterbalance it. Example: "emo metal" defaults toward emo pop (12.2B pop connections vs. 0 direct metal links) unless you reinforce the metal side heavily and exclude pop.

### Prompt Adherence Decays Within a Generation

Community observation (see budmaniak workflow, Section 9): within a single generation, Suno's first 30–60 seconds adhere most tightly to your prompt. After roughly the first minute the model tends to:
- Repeat motifs already established
- Simplify structure
- Fall back to generic patterns
- Loop chord progressions
- Lose lyrical coherence

This is normal model behavior, not a bug. Two implications:
- **For one-shot tracks**, front-load your most distinctive prompt elements — anything you need *guaranteed* should be present in the opening.
- **For longer tracks**, plan to either build section-by-section (Extend method) or assemble from multiple candidates (Assemble From Parts) rather than chasing a perfect end-to-end pass. Both workflows are in Section 9.

---

## 2. Model Selection

| Model | Best For | Weakness |
|-------|----------|----------|
| v4 | Legacy/chaos/happy accidents | Outdated, unpredictable |
| v4.5 | Fast iteration, consistent results | Mangles lyrics, needs batch generation |
| v4.5+ | Controlled creativity, surprises | Unstable, throws in random elements |
| v5 | Professional release, vocals, acoustic | Insists on weird intro vocals, less adventurous, needs more iteration |
| **v5.5** | **Most expressive vocals, personalization (Voices / Custom Models / My Taste), current default** | More sensitive to prompt quality — generic prompts give more obviously generic results |

**Decision guide:**
- Making something for release → **V5.5**, with a precise prompt
- You want your own voice on the track → **V5.5** + Voices (paid tiers)
- You have a back catalog and want Suno to sound like *you* → **V5.5** + Custom Model (Pro/Premier)
- Experimenting, want surprises → **V4.5+**
- Testing ideas quickly → **V4.5**

**V5/V5.5 note on saturation:** Both V5 and V5.5 use *less* aggressive saturation than earlier models. This is a feature for professional contexts — over-saturation is a dead giveaway when sending tracks to mix engineers and is harder to clean up in post.

**V5.5 prompt-sensitivity note:** Suno describes v5.5 as "more expressive" and more responsive to prompt quality. In practice: a vague prompt that limped along on v5 tends to sound *more* generic on v5.5, not less. Invest in the prompt before blaming the model.

**V5.5 known issues (community-reported):**
- **Mid-track degradation:** songs start clean, then hiss / get robotic / lose coherence partway through. Common complaint on v5.5; less common on v5 and v4.5.
- **Pop reversion:** v5.5 frequently "forgets" the prompted style mid-track and drifts back toward generic pop.
- **Extend is unreliable:** v5.5 Extend sometimes produces an entirely different song from the source rather than continuing it.
- **Common workaround:** generate on **v4.5 or v5**, then run **Cover on v5.5** to keep v5.5's improved vocals without inheriting its mid-track problems. See [Hybrid Model Workflow](#hybrid-model-workflow-generate-on-v45v5-cover-on-v55) in Section 9.

---

## 3. Prompt Engineering

### Stop Using Comma Lists

Suno was trained on professional music metadata with categorical structure. Structured prompts speak its native language. You can verify this on desktop: hover over your prompt and each category gets underlined as a parsed unit.

### The Core Prompt Format

```
genre: "indie folk rock, 2020s bedroom pop, Phoebe Bridgers x Big Thief vibe"

vocal: "soft female alto, intimate whisper-to-belt, gentle vibrato, slight nasal quality"

instrumentation: "fingerpicked acoustic guitar, warm upright bass, sparse piano, light ambient pads"

production: "lo-fi intimacy, tape warmth, close-miked vocals, narrow stereo image, natural room reverb"

mood: "melancholic, nostalgic, late-night introspection"
```

### The 7-Element Prompt Formula

From community analysis of high-performing tracks:

```
[GENRE] + [TEMPO] + [MOOD] + [INSTRUMENT] + [VOCAL STYLE] + [ERA] + [REFERENCE]
```

Example:
```
Indie folk, 92 BPM, melancholic, fingerstyle acoustic guitar, whispered vocals, 2010s indie style
```

### Formatting Rules That Matter

**Use periods, not commas for critical requirements.** Commas signal optional elements. Periods mark conceptual boundaries.

❌ `acoustic guitar, male vocals, emotional, reverb`

✅ `acoustic guitar with male vocals and emotional delivery and reverb-heavy production.`

**Use conjunctions for essential elements.** "And" and "with" treat items as required rather than optional.

**Verify parsing on desktop.** Mouse over the style panel — each section should be underlined as a complete unit. If not, you're missing periods.

### MAX Mode Template

For acoustic, folk, country, orchestral, and singer-songwriter material, MAX Mode delivers substantial improvement. For electronic, trap, and synthwave, it does nothing useful.

```
[Is_MAX_MODE: MAX](MAX)
[QUALITY: MAX](MAX)
[REALISM: MAX](MAX)
[REAL_INSTRUMENTS: MAX](MAX)
[START_ON: TRUE]
[START_ON: "write out the first few words of lyrics here"]

genre: "your genre"
instruments: "your instrument descriptions"
style tags: "your style descriptions"
```

### The Lyric Bleed Problem

Suno will sing anything that *looks singable*. Common triggers:
- Short poetic lines in your style prompt
- ALL CAPS phrases
- Quoted phrases that could be lyrics
- Empty lyrics box
- Prose with natural rhythmic cadence

**Mitigation:**
- Keep style prompts dense and technical — hard to sing
- Always put something in the lyrics box
- Add `///*****///` at the top of the lyrics box as a divider
- Avoid quotes outside structured fields

### Priority Order in Style Field

1. Genre
2. Dominant mood
3. Lead instrument(s)
4. Vocal style / gender
5. Production / atmosphere

### Instrument Specificity

Naming specific gear produces noticeably different tonal results:

```
Epiphone Les Paul Standard, G major blues chord progression
Fender American Tele, G major blues chord progression
Gretsch G5422TG, G major blues chord progression
```

Go further: amp type, effects chain, playing technique. The more specific the instrument context, the more distinctive the output.

### Prompt Word Field Test Results

From 200+ track mass-generation:

| Works | Unreliable / Ignored |
|-------|---------------------|
| `anthemic chorus` | `ethereal` |
| `building intensity` | `crescendo` (use "building intensity" instead) |
| `gritty vocals` (~50%) | Mixing technique descriptors |
| Emotion descriptors (`desperate`) | Technical sound descriptors (`minor key with reverb`) |

**Emotion beats technique.** `desperate` → better melodic and harmonic choices than `minor key with reverb`.

### Word Morphing Trick

Add `-er` or `-ing` to descriptors that don't normally take them:

```
etherealer / etherealing
```

Community reports unexpected tonal results. Low cost to test.

---

## 4. Song Structure & Meta Tags

### Base Tags (daveshap Syntax)

These are the validated base tags. Each can be modified with adjectives:

```
[Intro]         [Hook]          [Pre-Chorus]    [Chorus]
[Verse]         [Interlude]     [Break]         [Movement]
[Instrumental]  [Solo]          [Build]         [Bridge]
[Outro]         [End]
```

### Tag Behavior Reference

| Tag | Behavior | Best Modifiers |
|-----|----------|----------------|
| `[Intro]` | Strictly instrumental, opening only | Emotion + pace: `[Long Mellow Intro]`, `[Short Exciting Intro]` |
| `[Pre-Chorus]` | Strictly vocal, use once or before chorus only | Style + delivery: `[Haunting Whispered Pre-Chorus]`, `[Primal Scream Pre-Chorus]` |
| `[Chorus]` | Prime workhorse; system often overrides modifiers | Lyric content and vibe have more impact than tags here |
| `[Verse]` | Primary workhorse; system context-sensitive | `[Angry Verse]`, `[Whispered Verse]`, `[Spoken Verse]` |
| `[Interlude]` | Instrumental workhorse | `[Melodic Interlude]` most reliable; avoid genre-specific modifiers |
| `[Break]` | Strictly instrumental, ~1 measure; lead instrument works best | `[Violin Break]`, `[Drum Break]`, `[Lead Guitar Break]` |
| `[Solo]` | Best with instrument + pace + energy | `[Soaring Lead Guitar Solo]`, `[Fast and Intense Drum Solo]` |
| `[Build]` | Often treated like a break; use sparingly, once per song | Best sandwiched between verse and chorus |
| `[Bridge]` | System often treats like verse/chorus; use once | `[Instrumental Bridge]` most effective |
| `[Outro]` | Signals wind-down; use exactly once | Emotion + pace: `[Long Fading Outro]`, `[Mournful Outro]` |
| `[End]` | Terminal tag | `[Fade to End]`, `[Lingering End]` |
| `[Movement]` | Experimental; system may ignore | `[Begin Psychedelic Movement]`, `[Transition to Faster Harder Movement]` |

### Meta Tag Stacking

Pipe-separate multiple instructions for precise section control:

```
[Chorus | anthemic chorus | stacked harmonies | modern pop polish]
[guitar solo | 80s glam metal lead guitar | heavy distortion | wide stereo | whammy bar bends]
[Verse | Baritone Vocal | Minimal Arrangement]
```

Stacked tags override global style prompt for that specific section.

### Vocal Tags

```
[Spoken Word Narration]         [Telephone Call]
[Female Opera Singer]           [Swanky Crooning Male]
[Ethereal Female Whisper]       [Operatic Soprano Female Vocal]
[Baritone Vocal]                [Massive Choir]
[War Chant]                     [Backing Choir]
[Call and Response]             [Soft Vocal]
[raspy lead vocal]              [autotuned delivery]
[stacked harmonies]             [anthemic chorus]
[spoken word verse]             [emotional build-up]
[crowd-style vocals]
```

Vocal tags can stand in for `[Verse]` or `[Chorus]` entirely:

```
[Spoken Word Narration]
*static* ...final log... coordinates unknown...
...oxygen critical... systems failing...
...tell earth we made it... we saw such beautiful things...
...orion spur expedition... signing off... *static*
```

### Dynamic & Arrangement Tags

```
[Building Intensity]            [Stripped Back]
[Minimal Arrangement]          [Layered Arrangement]
[Explosive Chorus]             [Massive Finale]
[Epic Choir Outro]             [Sudden Break]
[Crescendo]                    [Decrescendo]
[Orchestral Break]             [Cinematic Orchestra]
[High Energy]                  [Building Energy]
[Massive Choir]                [Cinematic Orchestra]
```

### Instrument Tags

Can be used in lieu of `[Solo]` or as part of one:

```
[Sad Trombone]          [Chugging Guitar]       [Overblown Flute]
[Trilling Pennywhistle] [808 sub bass]          [orchestral strings]
[60s jangly guitar rhythm]                      [sidechained synth bass]
[pedal steel guitar]
```

### Rhythm Modifiers for Instrumental Sections

Use `.` and `!` to shape pacing inside interlude/break sections:

```
[Melodic Interlude]
. . . ! . .
. ! . . ! .

[Intense Interlude]
!! . ! !! !
!! !! ! !!
```

### Lyric Formatting for Vocal Control

**Capitalization = intensity:**
```
MY WORLD'S BEEN LEFT IN SORROW FOR WAY TOO LONG!   ← loud/intense
My world's been left in sorrow for way too long.    ← calm/quiet
```

**Punctuation effects:**
- `...` = slower, more hesitant delivery
- `!` = emphasis
- `(parenthesis)` = call-and-response or backing vocal
- `Oooooohhh whoaaa` = explicit vocalizations (system won't generate these without being told)
- `mmmmm oh...` = dampening effect

**Background vocals:**
```
(fading away...)       ← quiet background
(RISE UP NOW!)         ← loud background
```

### Section Separation

Always separate sections with blank lines:

```
[Verse 1]
Lyrics here

[Chorus 1]
Lyrics here
```

### Closing the Structure

End your lyrics with `[End]` on its own line. Without a terminal tag, Suno will sometimes overrun the natural finish — fading mid-verse, tacking on an instrumental ramble, or hard-cutting at the length limit. This is a steer, not a guarantee: community testing puts it around 85–90% effective, and it tends to fail when the section right before it runs long or leaves a structure tag unclosed. If `[End]` keeps getting ignored, shorten the final section and make sure every `[Section]` above it is well-formed.

**On tag placement (unverified):** A widely-shared claim is that modifier tags should sit *before* a section header, not after — the theory being that Suno reads top-to-bottom, so tags placed under the header arrive after generation for that section has already started:

```
[Anthemic]
[Powerful Drums]
[Chorus]
We rise up, we fall down
```

The source is a promotional post and the claim is untested here, so treat it as an A/B to run, not a rule. Note that pipe-stacking (`[Chorus | Anthemic | Powerful Drums]`, see [Meta Tag Stacking](#meta-tag-stacking) and Trick #11) puts the modifiers and the header in the same token span and sidesteps the ordering question entirely — it's the more reliable pattern when section-level control matters.

### Structure Control: START_ON

Skip the intro and begin immediately on lyrics:

```
[START_ON: TRUE]
[START_ON: "write the first few words of your lyrics here"]
```

### Duet Voice Control

```
[DUET_START_ON: TRUE]
[MALE_START_ON: "first words of male vocal"]
[FEMALE_START_ON: "first words of female vocal"]
```

### Build Progressions

Common section sequences that work:

```
Ballad:    Gentle → Building → Powerful → Emotional
EDM:       Ambient → Rising → Drop → Explosive
Rock:      Clean → Driven → Heavy → Epic
Choir Epic: Solo → Backing Choir → Massive Choir → Decrescendo
```

### Full Cinematic Choir Example

**Style prompt:**
```
Overwhelming awe and life-or-death tension. Starts as a distant haunting prophecy, 
building into a world-shattering climactic clash. Verses feel intimate but urgent, 
a lone voice carrying the weight of a desperate plea. Chorus is a tidal wave of 
collective fury and righteous power, voices layered to the heavens. Operatic grandeur, 
shifting from a fragile lone soprano to a booming relentless mass choir. Sudden drops 
into absolute quiet, leaving only a single voice before the storm returns.
```

**Structure:**
```
[Intro]
[Massive Choir]
[War Chant]

[Verse 1]
[Operatic Soprano Female Vocal]
[Stripped Back]

[Pre-Chorus]
[Building Intensity]
[Backing Choir]

[Chorus]
[Explosive Chorus]
[Massive Choir]
[Cinematic Orchestra]

[Verse 2]
[Baritone Vocal]
[Minimal Arrangement]

[Pre-Chorus]
[Building Intensity]
[Call and Response]
(Solo) line...
(Choir) response...

[Bridge]
[Sudden Break]
[Soft Vocal]
[Female Vocal]
[Crescendo]
[Choir Chant]

[Orchestral Break]

[Final Chorus]
[Massive Finale]
[Massive Choir]

[Outro]
[Epic Choir Outro]
[Decrescendo]
[Fade Out]
```

---

## 5. Style Field

### What Goes in the Style Field

Comma-separated descriptive phrases **without brackets**. Priority order:

```
Genre → Mood → Lead instrument → Vocal style → Production → Atmosphere
```

### Tag Count Guidelines

| Use Case | Tag Count |
|----------|-----------|
| Simple songs | 3–5 core tags |
| Detailed control | 8–15 tags |
| Maximum | ~20 (beyond this confuses the model) |

### Genre Templates

**Pop Ballad:**
```
Pop, Emotional, Gentle, Building, Piano, String Section, Female Vocal, 
Vulnerable Vocals, Reverb Heavy, Hall Reverb, I-V-vi-IV, Clean Production
```

**EDM Anthem:**
```
EDM, Progressive House, Energetic, Explosive, Anthemic, Synth Lead, 
Synth Bass, TR-808, Electronic Drums, Vocoder, Vocal Chops, 
Sidechain Compression, Wall of Sound
```

**Indie Folk:**
```
Indie Folk, Peaceful, Contemplative, Organic, Acoustic Guitar, 
Fingerpicking, Harmonica, Male Vocal, Gentle Vocals, Natural Reverb, 
Dorian Mode
```

**Dark Electronic:**
```
Dark Electronic, Mysterious, Atmospheric, Synthesizer, Minor Key, 
Reverb Heavy, Delay, Ambient Pads, No Vocals, Cinematic, Building
```

**Trap/Hip-Hop:**
```
Hip-Hop, Trap Beat, 808 Bass, Male Rap Vocal, Aggressive, Urban Atmosphere
```

**Classic Rock:**
```
Rock, Distorted Electric Guitar Riffs, Acoustic Drums, Male Vocals, 
High Energy, Anthemic
```

**Stoner/Space Rock (high-performing example):**
```
space rock, psychedelic rock, desert rock, stoner rock, shoegaze
```
*Note: Taxonomy order matters — first genre has most influence, each subsequent tag has diminishing weight.*

### Style Field Pro Tips

- **Genre taxonomy:** List genres in order of desired influence
- **Emotive modifiers work better than technical ones** in the style field
- **Avoid stacking conflicting moods** in the same section (`[Calm]` + `[Aggressive]` requires intentional contrast framing)
- **Specific beats generic:** `Distorted Electric Guitar` > `Guitar`

---

## 6. Lyric Writing for Suno

### Plan Before Writing

Effective lyric structure requires three decisions upfront:

1. **Genre** — worship, EDM, pop-rock, country, experimental
2. **Emotional space** — intimate/vulnerable, stadium/anthemic, nostalgic, raw/angry
3. **Constraints** — tempo requirements, length, content restrictions

Constraints are creative assets, not limitations. They prevent generic "AI soup."

### Song Structure Building Blocks

```
Intro         → Establishes sonic world, often instrumental
Verse         → Tells the story; lyrics change each time
Pre-Chorus    → Builds tension toward chorus
Chorus        → Main thesis and emotional punchline — what everyone remembers
Bridge        → Different angle or emotional shift; break from repetition
Outro/Tag     → Final repeated idea or wind-down
Instrumental  → Musical breathing space
```

### Syllable Discipline

Suno aligns syllables against beats. Inconsistent line lengths cause awkward phrasing.

- **Target:** 6–10 syllables per line for most genres
- **Lines per section:** 4 standard, 8 for longer sections
- **Variance:** ±1–2 syllables for lines in the same structural position
- **Test:** Read aloud rhythmically; clap syllables

**One syllable short fixes:**
- Repeat last word: "Stay with me" → "Stay with me, stay"
- Add natural filler: "Stay with me tonight" → "Stay with me tonight, love"

### The One Metaphor Rule

Pick one central metaphor and go deep. Water, fire, architecture, light — pick one and explore all its facets. Songs that switch metaphors (river → fire → clouds → earth) give listeners no visual anchor.

This applies at album level too: each song gets its own motif for focus, variety comes from the collection, not from within a single track.

### Narrative Arc

Verses tell a story with progression. Abstract themes need specific narrative beats:

**Example progression for "falling in love":**
- Verse 1: First glance, specific physical detail
- Pre-Chorus: Internal reaction, nerves
- Chorus: Central thesis
- Verse 2: Complexity emerging, a crack in the facade
- Bridge: Vulnerability on both sides, twist
- Final Chorus: Same hook, deeper meaning after the journey

---

## 7. Realism & Production Quality

### "Realistic" Is a Weak Descriptor

Suno often ignores the word "realistic." **Descriptors of physical recording reality** are strong:

### Acoustic Realism Stack

```
Small room acoustics / room tone (air, faint hiss) / close mic presence
off-axis mic placement / proximity effect / single-mic capture
one-take performance / natural timing drift (human micro-rubato)
natural dynamics (no brickwall feel) / breath detail (inhales, exhales)
```

### Performance Detail Descriptors

```
Mouth noise (subtle lip noise, saliva clicks)
Pick noise (attack, scrape) / fret squeak (string slides)
Finger movement noise on strings / chair creak and body shift
```

### Analog Character

```
Tape saturation / analog warmth and harmonic grit
Slight wow and flutter (tape pitch wobble)
Gentle preamp drive (edge without distortion)
```

### Spatial / Mixing

```
Limited stereo (mono or narrow image)
Realistic reverb type (short room, early reflections)
Background noise floor consistent (not dead-silent)
Imperfections kept (tiny pitch drift, tiny buzz, slight rasp)
```

### Complete Acoustic Prompt Example

```
genre: "Acoustic folk, one singer and one guitar, intimate bedroom recording"

instruments: "Single acoustic guitar with fingerpicking, baritone vocals with 
emotional phrasing, vocal grit, blue note bends"

recording: "one person, one guitar, single-source path, natural dynamics"

style: "authentic take, tape recorder, close-up, raw performance texture, 
handheld device realism, narrow mono image, small-bedroom acoustics, unpolished, dry"

sound: "small room acoustics, close mic presence, proximity effect, one-take 
performance, natural timing drift, natural dynamics, breath detail, pick noise, 
fret squeak, finger movement noise, tape saturation, analog warmth, limited stereo, 
background noise floor consistent, imperfections kept"
```

### Eliminating the Generic Sawtooth Synth

Suno defaults to sawtooth synths constantly. You can't just say "no saws" — give it something else to latch onto.

**Replace with specific synthesis types:**
```
FM synthesis bass / wavetable movement / formant-driven bass
granular textures / spectral morphing / resonant bandpass motion
```

**Describe motion, not size:**
```
evolving modulation / LFO-driven movement / non-repeating bass cycles
```

**Shape harmonics directly:**
```
rounded harmonic profile / asymmetric waveforms / odd-harmonic emphasis
```

**Bass archetypes that avoid saws:**
```
Reese bass movement / neuro bass texture / growl bass modulation / sub-driven bass design
```

**Control high end and stereo:**
```
smooth top end / controlled high harmonics / center-focused bass / mono-stable low end
```

**Complete anti-saw prompt:**
```
FM and wavetable bass design, evolving modulation, non-repeating harmonic motion, 
rounded harmonic profile, controlled high end, phase-coherent low end, clean punch.
```

### Electronic Production Stack

For electronic, hip-hop, trap — shift away from realism language entirely:

```
genre: "Synthwave EDM, driving bassline, intense energy"

instruments: "Synthesizer lead, wavetable movement, FM synthesis bass with evolving 
modulation, electronic drums with punchy dynamics, TR-808 sub bass with sidechain compression"

style: "dark atmosphere, mysterious, nostalgic 80s, wide stereo image, wall of sound, 
synth-heavy, reverb with early reflections"

production: "modern mastering, high fidelity, clean transients, dynamic range, 
punchy compression, polished professional sound"
```

---

## 8. Advanced Parameters

### Weirdness (0–100%, default 50%)

| Goal | Setting | Effect |
|------|---------|--------|
| Safe, commercial, covers | 10–30% | Follows tags closely |
| Balanced (default) | 40–60% | Mix of control and surprise |
| Experimental / avant-garde | 70–100% | Unusual, unpredictable |

**Genre recommendations:**
- Classic genres (Pop, Rock, Country): 30–50%
- Experimental (Ambient, IDM): 60–80%
- Unusual fusions: 70–90%
- Covers / tributes: 10–30%

### Style Influence (0–100%)

| Goal | Setting | Effect |
|------|---------|--------|
| Tags as loose inspiration | 10–30% | Maximum freedom |
| Balanced | 40–60% | Flexibility with fidelity |
| Strict adherence | 70–100% | Very close to tags |

**When to use high Style Influence:** Vague tags (pop, rock) benefit from 70–90% to compensate for ambiguity.

**When to use moderate Style Influence:** Specific tags (Progressive Djent Metal, 7/8 time) work better at 40–60% — specificity already constrains output.

### Exclude Styles

Two distinct mechanisms — don't confuse them:

**1. The Exclude field (recommended, more reliable)**
In Custom Mode → Advanced Options there's a dedicated Exclude input. Type instruments, styles, or vocal types you don't want there. Suno renders excluded items with a leading `-` in the song's sidebar. This is a first-class feature — more reliable than any inline negation.

**2. Inline negation in the Style field (unofficial fallback)**
You can add `no [element]` inline in the style field — e.g. `Indie folk, acoustic guitar, 95 BPM, no drums, no autotune`. Caveats from community testing:
- **Use `no X`, not `don't add X`.** Suno doesn't reliably parse "don't."
- **Place exclusions at the end of the style prompt.** Positives are processed first; exclusions are applied after.
- **The `-X` minus-sign form** (e.g. `-scraping sounds`) is how Suno *displays* exclusions in the UI — its behavior as *input* in the style field is undocumented. The verified inline form is `no X`.
- **Less reliable than the Exclude field.** Use inline when you're already at the prompting stage and don't want to switch contexts, or when an exclusion targets a specific descriptor that the Exclude field doesn't accept cleanly.

**Combined approach** (used in this guide's [Getting Guitar + Voice Only](#getting-guitar--voice-only-no-drums-no-bass) example): put `no drums, no percussion` inline *and* `drums, percussion` in the Exclude field. Belt and braces — the most reliable way to eliminate something stubborn.

**Don't over-exclude.** Keep total exclusions to ~2–3 maximum. Too many makes the arrangement unstable — Suno needs *something* to build with.

**Common exclusion targets:**

| Goal | Exclude |
|------|---------|
| Only female vocals | Male Vocal |
| Only rap | Singing, Melodic Vocals |
| Acoustic only | Electronic, Synthesizer, Drum Machine |
| Pure rock | Electronic, Hip Hop, Pop |
| Pure classical | Modern, Electronic, Pop |

### Killing Intro Hums & Vocalized Intros

V5's most-reported annoyance: songs that open with a wordless hum, an "ooh/ahh" vocalise, or a spoken/ambient intro before the actual lyrics start (this is the "insists on weird intro vocals" behavior flagged in [Section 2](#2-model-selection)). The fix is a dense **negative-style** list aimed specifically at intro vocalizations — stack these in the Exclude Styles box:

```
humming, hums, mmm vocals, ooh vocals, ahh vocals, la la vocals, wordless vocals,
vocalizing before lyrics, intro vocals, spoken intro, whispered intro, ad-lib intro,
instrumental intro, ambient intro, cinematic intro, long intro
```

Combine with `[START_ON: TRUE]` and `[START_ON: "first words of your lyrics"]` (see [Section 4](#4-song-structure--meta-tags)) to push Suno straight into the first line. The exclude list removes the *hum*; START_ON removes the *runway* in front of the vocal. Using both is more reliable than either alone.

### Quick Reference by Genre

| Genre | Weirdness | Style Influence | Key Excludes |
|-------|-----------|-----------------|--------------|
| R&B/Hip-Hop | 30–50% | 70–90% | Opposite genre elements |
| Rock | 30–50% | 80–100% | Electronic, Hip Hop, Pop |
| Pop | 20–40% | 70–90% | Heavy Distortion, Screaming |
| Country | 20–40% | 80–100% | Synthesizer, EDM, Hip Hop |
| Electronic/EDM | 40–60% | 80–90% | Acoustic, Organic |
| Classical | 20–30% | 90–100% | Drum Set, Electric, Synth |
| Jazz | 40–60% | 70–90% | Electronic, Synth |
| Experimental | 60–80% | 40–60% | Minimal or none |

### Troubleshooting

| Problem | Fix |
|---------|-----|
| Too soft / melodic | Add `[Aggressive]`, `[Intense]`, `[Distortion]`; raise Style Influence |
| Too chaotic | Reduce Weirdness to 20–40%; remove conflicting tags |
| Wrong vocals | Set Vocal Gender explicitly; use Exclude for unwanted voice type |
| Too electronic | Exclude: Electronic, Synth, Drum Machine; Add: Acoustic Guitar, Acoustic Drums |
| Too acoustic | Exclude: Acoustic Guitar, Acoustic Drums; Add: Synthesizer, Electronic Drums |

---

## 9. Advanced Techniques

### Section-by-Section Building (The Extend Method)

The most important workflow improvement for quality output:

1. **Generate** intro only (15–20 seconds)
2. **Listen** — approve or discard before proceeding
3. **Extend** into Verse 1, explicitly directing that section
4. Continue: Pre-Chorus → Chorus → Verse 2 → Bridge → Final Chorus → Outro

Each extend reads from existing audio context — you're continuing something already approved, not prompting blind.

**Extend timing:** Extend from points of **building momentum**, not after natural endings. Mid-verse produces better continuations than post-chorus.

### Assemble From Parts (Studio Workflow)

A different philosophy than the Extend method. Instead of building one continuous track by extending forward, you generate 4–6 parallel candidates and **stitch the best sections of each together in Suno Studio (or any DAW)**. Best for full-length tracks with instrumental breaks, solos, and dynamic arrangement — things Suno rarely produces in a single pass.

**The core constraint that makes this work:**

> **Lock BPM and key in the style field, and reuse the same values for every generation in the project.**

Put them at the very start of the style field, e.g. `108 BPM, key of G minor, ...`. Once every candidate shares BPM and key, sections from different generations align on the beat grid and you can splice freely.

**Why this works:**
- Suno's first 30–60 seconds adhere most tightly to your prompt; later sections drift (see Section 1). Treating each generation as a *source of one good section* is more efficient than hoping any single generation is good end-to-end.
- Long instrumental breaks, solos, and extended bridges almost never appear in a one-pass generation. You build them later.
- Mashups and short-lyric tracks won't exceed the length of their lyrics — Suno doesn't reliably invent instrumental filler to stretch runtime. `[Instrumental]` tags may add a measure or two but won't carry a real solo section.

**The workflow:**

1. **Seed with a strong lyrical hook.** Idioms and common phrases ("come as you are," "wildest dreams") anchor the generation; awkward AI lyrics can be fixed later.
2. **Lock BPM + key in the style field.** Same values in every generation for this project.
3. **Generate 4–6 candidates.** You're not hunting for *the* song — you're collecting verses, choruses, bridges, intros, and outros.
4. **Assemble in Suno Studio (or your DAW).** Drag in the candidates, cut on the downbeat, swap sections freely: Verse from Gen 1, Chorus from Gen 2, Bridge from Gen 4.
5. **Generate a dedicated instrumental track for breaks.** Take your best generation, split stems (Suno's built-in splitter is ~10 credits and integrates with Studio), download the instrumental, run it through **Create → Remix → Cover** with prompts like *"fully expressive instrumental arrangement, active fills, evolving textures, build into a lead solo, never static."* Extract the solos, breakdowns, and reprises from the result and drop them into the arrangement.
6. **Export.**

**Lyric fixes inside Studio:**

| Method | When to use | Settings |
|--------|-------------|----------|
| **Replace** (click section, edit lyrics, open Advanced) | Changing a few words, keeping the music | Weirdness 20–30, Style Influence 60–70, Audio Influence 70–80. Set Replace to **Song**, not Vocals Only — keeps instruments synced to new phrasing. |
| **Sample** (Create → Sample → highlight stanza) | Replace gave weird results | Same influence settings as Replace |
| **Cover** (best structure → Remix → Cover, paste new lyrics) | Rewriting the whole song over the same arrangement | Same influence settings |

**When to use this vs. Extend:**
- **Extend method** — you have a clear single-take vision and want continuity. Best for short-form, tight arrangements, tracks where vibe matters more than structural variety.
- **Assemble method** — you want a full-length release with real dynamics, instrumental sections, and section-level variety. Best when you'd rather spend 1–2 hours in Studio than burn 30 generations chasing a one-shot win.

### Quality Over Volume

From community analysis:
> Out of 500 generated tracks, 485 were complete noise. One refined prompt outperformed the other 500 combined.

**Workflow:** Spend real time on one prompt → test 7+ instrument combinations → calibrate BPM specifically → tweak emotional descriptors → run that one prompt 20 times → pick the best.

**Curation forcing function:** Make music videos. Visualizing a song scene-by-scene immediately reveals which tracks have real structure vs. which are just vibes.

### Remastering with Metadata Tags

Add tags to **Song Details → Displayed Lyrics** before hitting Remaster:

```
[high_fidelity]    [studio_mix]       [analog_warmth]
[crystal_clarity]  [punchy_dynamics]  [tape_saturation]
[vocal_depth]      [smooth_transients]
```

**Vintage warmth stack:** `[tape_saturation]` + `[analog_warmth]` + `[smooth_transients]`

**Modern clarity stack:** `[crystal_clarity]` + `[transient_detail]` + `[punchy_dynamics]`

### Cover-Based Remastering

More effective than built-in Remaster. Use Cover with minimal prompt to improve quality without changing character:

```
Genre: Original genre with high fidelity recording and professional mastering.
Instrumentation: Acoustic drums with realistic sound.
Mastering: Clean, modern, professional sound.
```

Set: Weirdness 0, Style Influence 100, Audio Influence 100. Avoid style descriptors unless you want drift from the original.

### Song-to-Song Transplant

Generate a specific section (e.g., a bridge that sounds like a different band) as its own standalone song. Extract that section. Insert into your main track. Run Cover with Weirdness 0, Style Influence 100, Audio Influence 100.

**Warning:** Seams usually show — slight quality drop at the join. Use when nothing else achieves the contrast you need.

### Building Effective Personas

A persona must be a character dossier, not a label. Build in four layers:

1. **Demographics/timbre:** Age, gender, voice type, fundamental character
2. **Technical delivery:** Enunciation, phrasing, breath control, technique
3. **Emotional context:** Detached, passionate, vulnerable, aggressive
4. **Sonic anchor:** Artist reference points

**Example persona:**
```
Female contralto, androgynous, cold, monotone delivery, sharp enunciation, 
emotionally numb, sinister tone, reminiscent of Grimes with HEALTH-like atmosphere.
```

Multiple overlapping constraints dramatically reduce vocal variance between generations.

### Thumbnail Quality Predicts Audio Quality

When batch generating, thumbnail visual appeal predicts audio quality more accurately than other preliminary signals. Use for quick triage before listening.

### Simple Mode vs. Custom Mode

**Simple Mode** — best for rapid exploration, rough ideas, or when you don't care about exact structure. One compact paragraph prompt, generate a few versions.

**Custom Mode** — recommended whenever you care about sections, want a strong intentional hook, or need repeatable results you can iterate on. If your goal is "I want this to sound like the thing in my head," Custom Mode is the better default.

### The 5 Questions Before Writing Any Prompt

Answer these before touching the style field. If you can answer all five, you can write a good prompt almost every time:

1. **Instrumental or vocal?** — changes everything downstream
2. **Vibe in 3–5 adjectives** — uplifting, tense, nostalgic, dreamy, aggressive
3. **Primary genre + optional secondary flavor** — "synthwave with a little modern pop edge"
4. **Tempo and groove** — BPM or slow/mid/fast, plus feel ("four-on-the-floor," "half-time," "swing")
5. **What carries the hook?** — lead synth? guitar riff? piano motif? big vocal chorus?

### Getting Guitar + Voice Only (No Drums, No Bass)

The trick is to make the arrangement identity a **solo/duo performance** and explicitly exclude the two most common additions:

```
Style of Music:
Stripped-down acoustic singer-songwriter duet, intimate and warm, 84 BPM, 
close-mic natural vocal, fingerstyle acoustic guitar as the only accompaniment, 
no percussion and no bass, dry minimal production, clean mix, quiet room ambience, 
emotional and honest performance, dynamic but subtle

Lyrics / Structure:
[Intro: acoustic guitar only, fingerstyle, no percussion, no bass]
[Verse 1: vocal + guitar only, close and intimate, minimal dynamics]
(keep arrangement sparse: only one acoustic guitar + one lead vocal)
[Chorus: bigger vocal emotion but still only guitar + voice, no added instruments]
[Outro: guitar-only, let last chord ring]
[Fade Out]
```

**If drums keep appearing:** Add `no drums, no percussion, no beat, no rhythm section` to style + Exclude: `drums, percussion`

**If bass appears anyway:** Add `no bass, no sub bass` to style + Exclude: `bass`

**If pads/strings sneak in at the chorus:** Add `(do not add any instruments; keep only guitar + lead vocal)` before `[Chorus]` + Exclude: `strings, pads, keyboard`

### Hard Genre-Switch Within a Song ("Two Songs in One")

For tracks that need genuinely different vibes per section (e.g., dark rap verses + bright pop chorus):

**One-shot attempt:** Use language that signals duality without blending — "schizophrenic pop," "dynamic mood shifts," "aggressive dark electronic verses, bright upbeat piano rock chorus." Add `[Stop]` or `[Break]` before the chorus to force a reset.

**Splicing workflow (more reliable):**
1. Generate the verse separately with dark style prompt
2. Hit **Extend**, clear the style box completely
3. Enter the new contrasting style for the chorus section
4. Suno uses the audio tail of the verse to morph into the new style

This creates a distinct "two songs in one" feel that single-shot generation rarely achieves cleanly.

### Vocal Type Tags: Style Field, Not Lyrics Box

A key finding from metal/extreme genre users: **vocal type tags work significantly better in the Style field than embedded in the lyrics box.**

```
✅ Style field: "female screaming vocals, guttural growl, raspy delivery"
❌ Lyrics box: [screaming vocals] — often ignored
```

For extreme genres, describe the sound paragraph-style rather than stacking genre tags:

```
✅ "70s doom metal, sludgy and fast, deep distorted guitar and bass, 
   heavy deliberate drums, vocals with a powerful mournful quality, dark and menacing"

❌ "melodic death metal, nu-metalcore, deathcore, angry, screaming vocalist"
```

The paragraph approach gives Suno more context to triangulate from. Genre stacking above 2-3 tags causes confusion.

### Official Suno v5 Guidance (still applies to v5.5)

From the Suno team directly:

**Starting out with v5:**
- Begin with 1–2 genre tags in Style of Music; keep lyric structures simple
- Layer in complexity one or two tags at a time — don't port over v4.5 prompts wholesale
- Use Exclude tags to push away unwanted elements
- Prompts that worked in earlier models **may not translate** — treat v5 as fresh territory

**Sliders:**
- Tweak one slider at a time; don't jump above 50% immediately
- Adjust, test, roll back or forward carefully
- Double-click any slider to reset to default if results go wrong

**Song length:**
- Shorter lyrical inputs cause shorter generations
- Cover, Inspo, and Persona features naturally reduce generated length — this is by design, not a bug
- Extend is the correct tool for building out length

**Inspo feature:**
- Works best with 1–4 reference songs
- Be selective with playlists — too many songs dilutes the reference
- Drag and drop directly from your workspace into the Create panel: two options appear — *Drop here to Remix* or *Drop here to use as Inspiration*

### v5.5: Voices, Custom Models, My Taste

v5.5 (released March 26, 2026) is primarily a **personalization** release rather than a new prompt engine. Suno describes the model itself as "more expressive" and more responsive to prompt quality — meaning prompt discipline matters more, not less. No documented changes to style-field tag budget or meta-tag behavior; assume parity with v5 until field-tested otherwise.

**Voices (paid tiers)** — evolution of Personas. Capture your own singing voice and use it as the vocal in any generation. Solves the "same singer across multiple tracks" problem Personas only half-solved.

- **Upload spec (from Suno docs):** 15 seconds to 4 minutes of audio; Suno keeps the best 2 minutes. Acapella produces the cleanest results, but tracks with backing music are accepted — Suno runs stem extraction automatically.
- **Verification step:** Suno displays a short phrase for you to read aloud. The spoken recording is matched against the singing recording to confirm same person, and against the displayed text to confirm you said the right words. You then check a rights-confirmation box before saving.
- **Tier gating:** Suno's announcement and community sources put Voices on paid tiers; the help center doesn't enumerate plans explicitly. Confirm in your account before counting on it.
- **Style-bleed gotcha:** A Voice created from a *stylistically distinctive* source track will inject that style back into subsequent generations at **Audio Influence above ~20%**, usually around the midpoint or verse 2, in roughly 80% of attempts (community-reported, Cenn73 on r/SunoAI). If you want the Voice's *timbre* without its *style*, train Voices on cleaner / less style-heavy source recordings, and keep Audio Influence low when generating in a non-matching genre.

**Custom Models (Pro/Premier)** — upload your own finished tracks to fine-tune a personal v5.5 variant.

- **Minimum:** as few as 6 tracks. **Maximum:** 3 custom models per account.
- **Training time:** ~2–5 minutes per model.
- **Ownership:** you must own the rights to every track you upload.
- **Sharing:** Custom Models are private and cannot be shared between users.
- **Bulk Upload** is supported for batches.
- **Community guidance** (not from official docs): keep training tracks **stylistically consistent** — feeding the model a random genre mix produces a model that learned nothing in particular.

**My Taste (all users)** — adapts to your generation and listening patterns over time, biasing future outputs toward your preferred genres and moods. Also powers the **Style Augmentation** magic-wand button on the style field, which expands a short prompt using your taste profile.

**Practical implications:**
- If you've been using a hand-built Persona for vocal consistency on owned-vocal projects, **a Voice will probably replace it**.
- Custom Models reduce how much per-prompt detail you need to reach an "on-brand" sound — but only after they've been trained on consistent material.
- My Taste is silent influence: if your outputs start drifting toward something you don't want, check whether My Taste has been learning from generations you didn't actually like.

### MILO-1080 (Adjacent Release)

Launched alongside the v5.5 cycle (March 2026, currently in Suno's Labs area): **MILO-1080** — a 16-track step sequencer and synth designer with MIDI support. Tracks can be populated with prompted sample generations, clips from your Suno library, or sounds designed in MILO's synth engine. This is a production tool for people who know what a step sequencer is, not a generative shortcut. Out of scope for this guide, but worth knowing it exists if you're moving from Suno generations into a more hands-on production workflow.

### Hybrid Model Workflow: Generate on v4.5/v5, Cover on v5.5

Community-converged workaround for v5.5's mid-track issues: do the **composition and structure** on v4.5 or v5, then **Cover on v5.5** to inherit v5.5's stronger vocals without inheriting its audio degradation, pop reversion, or unreliable extend.

**When to reach for it:**
- v5.5 keeps falling apart mid-track and tighter prompting hasn't helped
- You want v5.5-quality vocals on a v4.5 instrumental groove
- Extend on v5.5 is producing different songs instead of continuations

**Why it works:** Cover takes structural and audio context from the source, so the v4.5/v5 track anchors the arrangement while v5.5 contributes the vocal performance. Reported independently by multiple users in the r/SunoAI v5.5 master-prompt thread.

### v5.5 Intro & Structure Fixes

Two specific community fixes for known v5.5 quirks (Cenn73 on r/SunoAI):

**Killing the "stuttering intro":** v5.5 sometimes opens with hesitation or repeated phrasing. Replace `[Intro]` with `[Short Instrumental Intro]` and follow it with `[Hook]` rather than `[Verse]`. The opening becomes a defined musical event instead of an ambiguous lead-in.

Also tried: `[Urgent Intro]` (community-suggested, less verified).

**Killing the stripped-down verse-ending drift:** Once your lyrics are rhythmically and lyrically locked, **remove all `[Verse]` and `[Chorus]` tags**. Keep `[Bridge]`, `[Drop]`, and `[Hook]` only. **Remove every empty line** — Suno reads blank lines as a pause or instrumental break and will hollow out the section. Crush stanzas together.

Fragile (Suno is fickle) but reproducible enough to be worth trying before burning more credits on the same prompt.

### Persona Workaround: Remaster → Persona

A song generated with a persona doesn't let you directly create another persona from it. Workaround: **remaster the song first**, then create a persona from the remaster.

Additional method: Crop the song and select the entire crop to create a persona — holds the same vocal style intact.

### Complete Dark Trap Template

A worked example for minimal cinematic trap (Carti/Ken Carson adjacent) — the key is keeping the global style and lyric structure separate and specific:

**Global Style:**
```
Dark, Restrained Trap In A Minor Key. Short, Consistent 808s With Clear Kick Separation.
Minimal Loop-Based Motifs Using Root Notes And Nearby Half-Steps.
Rolling Hats, Clean Snare Transients, Steady Momentum.
Sidechained Reverb/Delay For Space, Filtered Low End For Clarity.
Dense But Controlled Output, Gentle Peak Control.
```

**Lyric Structure (Instrumental Control):**
```
[INTRO]
[no vocals | vintage analog arpeggiator only | soft attack | filter drift | looping | background motion]

[VERSE 1]
[no arpeggiator | no chiptune | focus on groove and space]
[808s prominent | rolling hats | minimal motifs]

[PRE-CHORUS]
[subtle lift through texture | no new motifs | prepare transition]

[CHORUS]
[single small chiptune loop | clean | repetitive | abstract | no development]
[no analog arpeggiator | trap elements forward]

[VERSE 2]
[arpeggiator and chiptune absent | return to groove | maintain space]

[BRIDGE]
[thin arrangement | sidechained reverb/delay | no arpeggiator | no chiptune]
[pause before final section]

[FINAL CHORUS]
[chiptune returns | same loop | no variation | steady intensity | no analog arpeggiator]

[OUTRO]
[vintage analog arpeggiator returns | soft attack | filter drift | strip elements gradually]
[no chiptune | fade or clean stop]
```

This pattern — alternating presence of two elements (analog arp and chiptune) so they never overlap — is a reusable template for any two-element instrumentation contrast.

### Post-Processing & Remastering

Suno outputs almost always benefit from some post-processing before release. The degree of work depends on whether you want a quick loudness fix or a proper mix. This section covers both paths and the specific problems you will actually encounter.

---

#### The Fundamental Constraint

Suno audio is **already heavily compressed** and mixed internally. This is not a blank slate — it is a finished mix that happens to need polish. The most common beginner mistake is treating it like raw stems and stacking more compression on top, which crushes transients, removes dynamics, and makes everything sound worse.

**The correct mindset:** You are mastering a finished mix, not producing one from scratch. Small adjustments. Surgical EQ. A light limiter to hit platform targets. That is usually all a good Suno track needs.

---

#### Common Problems and Fixes

**Problem: Too much headroom / output is quiet**

Suno WAV downloads are intentionally conservative — the peak level sits well below 0dBFS. This is not a defect; it is intentional headroom for you to master up.

Fix: Use a limiter (not a compressor) to bring the output to -1dBFS true peak. Then target -14 LUFS for Spotify/streaming. Spotify normalizes to -14 LUFS anyway — going louder does not help and costs you dynamics.

Tools: Any DAW limiter, iZotope Ozone (Maximizer module), FabFilter Pro-L, Newfangled Elevate, or any AI mastering service.

---

**Problem: Muddiness in the low-mids**

This is the most common tonal problem in Suno output. The 200–500Hz range tends to accumulate energy, especially in dense arrangements with multiple instruments.

Fix: A surgical EQ cut in the 200–350Hz range — typically 2–4dB with a moderate Q. Listen while cutting and find where the "boxiness" lives for your specific track. Do not apply a blanket cut; find the frequency first.

Tools: FabFilter Pro-Q 3, Reaper's ReaEQ, Audacity's built-in EQ (less precise but functional for simple cuts).

---

**Problem: Harsh or brittle high end**

V5 in particular can produce slightly harsh sibilance or bright harshness in the 5–10kHz range, especially on vocals.

Fix: A gentle shelf cut above 8kHz (1–2dB) or a dynamic EQ / de-esser targeting the specific harsh frequency. Avoid broad cuts — they kill the air and make the track sound dull.

Tools: FabFilter Pro-DS (de-esser), Soothe 2 (dynamic resonance suppression), or a simple EQ shelf.

---

**Problem: Reverb tail gets cut off at the end**

Suno consistently clips the reverb and sustain tails at the end of tracks. The audio just stops.

Fix: In your DAW, add 1–2 seconds of silence at the end of the track before applying reverb as a send. Alternatively, use a short fade-out automation curve over the last 1–2 seconds to let tails decay naturally rather than hard-stopping.

Tools: Any DAW with automation. In Audacity: Effect → Fade Out on the last 1–2 seconds.

---

**Problem: Track is too saturated / distorted (especially V4/V4.5)**

Earlier models over-saturate. This is baked into the audio — you cannot fully remove it. What you can do is reduce the perceived harshness.

Fix: A gentle mid-side EQ to reduce saturation in the sides (where it is often most aggressive), or a multiband approach that gently tames the 2–5kHz range where saturation artifacts tend to be most audible. Do not try to "clean up" all saturation — some is character. Identify what is bothering you specifically and target that.

Tools: bx_digital V3 (M/S EQ), FabFilter Pro-Q 3 in M/S mode, iZotope Ozone's Spectral Shaper.

---

**Problem: Specific artifact frequencies / hiss on stems**

Suno stems (downloaded from the Song Editor) carry noise and artifacts, particularly on synth and SFX tracks. The WAV files themselves have a hard frequency cutoff around 17kHz despite being labeled as WAV — they are essentially MP3 quality with WAV packaging.

Fix: Lowpass filter at 18–20kHz to remove the artifact shelf. For hiss specifically, use spectral repair or noise reduction on the offending stem before incorporating it into your mix.

Tools: UVR5 (free, open source) with GPU processing produces significantly cleaner stems than Suno's built-in stem export. iZotope RX for spectral repair and noise reduction. Adobe Audition has noise reduction built in.

---

**Problem: No punch / transients feel soft**

Suno's internal compression rounds off transients, particularly on drums.

Fix: A **transient shaper** can restore attack without adding more compression. Increase attack shape on kick and snare. Subtle — do not overdo it.

Tools: Transient Designer Plus (Waves), SPL Transient Designer, or the free FLUX:: Bitter available through various freeware collections.

---

#### In-Suno Remaster with Metadata Tags

Before exporting at all, try the built-in Remaster with seeded metadata tags. Go to **Song Details → Displayed Lyrics**, add quality tags above your lyrics, save, then hit Remaster:

```
[high_fidelity]
[studio_mix]
[analog_warmth]
[crystal_clarity]
[punchy_dynamics]
[tape_saturation]
[vocal_depth]
[smooth_transients]
```

Stack tags to target a specific character:

| Target Sound | Tag Stack |
|---|---|
| Vintage warmth | `[tape_saturation]` + `[analog_warmth]` + `[smooth_transients]` |
| Modern clarity | `[crystal_clarity]` + `[transient_detail]` + `[punchy_dynamics]` |
| Vocal presence | `[vocal_depth]` + `[vocal_centered]` + `[clear_vocals]` |
| Wide stereo | `[true_stereo]` + `[stereo_depth]` + `[air_in_mix]` |

This is the lowest-effort remastering path. Results are inconsistent but sometimes surprisingly good, especially for acoustic and singer-songwriter tracks.

---

#### Cover-Based Remastering

More reliable than the built-in Remaster button. Use **Cover** with a minimal quality-focused prompt:

```
Genre: [original genre] with high fidelity recording and professional mastering.
Instrumentation: Acoustic drums with realistic sound.
Mastering: Clean, modern, professional sound.
```

Settings: Weirdness 0, Style Influence 100, Audio Influence 100.

**Critical:** Avoid style or mood descriptors — they cause the cover to drift from the original's character. The goal here is quality improvement, not reinterpretation. Covers preserve structure and genre when the style field is kept minimal.

---

#### Streaming Platform Targets

| Platform | Target LUFS | True Peak |
|---|---|---|
| Spotify | -14 LUFS | -1 dBFS |
| Apple Music | -16 LUFS | -1 dBFS |
| YouTube | -14 LUFS | -1 dBFS |
| Tidal | -14 LUFS | -1 dBFS |
| SoundCloud | -14 LUFS | -1 dBFS |

Spotify normalizes louder tracks down to -14 LUFS on playback. Mastering louder than -14 LUFS does not make your track louder on Spotify — it just reduces your dynamic range unnecessarily.

---

#### Stem Splitting

If you want to remix, replace, or individually process parts of a Suno track:

**Option 1 — Suno Song Editor (Pro/Premier):** Splits into up to 12 stems. Convenient but quality is MP3-grade (hard cutoff around 17kHz). Usable for light EQ or reverb additions but not ideal for detailed mixing.

**Option 2 — UVR5 (free, open source):** Community consensus is that UVR5 with GPU processing produces significantly cleaner stems. Best for extracting vocals for re-processing or when Suno's stems are too noisy to use.

**Option 3 — Spectralayers (paid):** Higher quality separation than UVR5 for complex material. Used by community members working on distorted or heavily layered tracks.

---

#### Tool Recommendations by Budget

| Budget | Path | Tools |
|---|---|---|
| Free | AI mastering service | Bandlab Mastering, iceestudios.com |
| Low ($0–$15/mo) | AI mastering with reference track | LANDR, emastered.com, Neural Analog (suno-specific) |
| Mid ($10–$30/mo) | AI mastering + stem control | Masterchannel, Kliga |
| DAW (one-time) | Full control | Reaper ($60), Audacity (free but limited) |
| DAW + plugins | Professional results | Reaper + FabFilter bundle, iZotope Ozone |

**On Audacity vs. Reaper:** Audacity applies effects destructively — baked into the waveform, hard to undo. Reaper is non-destructive, has automation, supports VST plugins without crashes, and costs $60 (free to evaluate indefinitely). If you are going to spend any time in a DAW, Reaper is the clear choice.

**Using AI to guide DAW work:** Gemini and ChatGPT can guide you through plugin settings step-by-step if you upload screenshots of your mixer and describe what you want. This is a legitimate and effective learning path — provide the AI with your plugin list and tell it specifically what is wrong with the track.

---

## 10. Grenar's Dirty Tricks — Lyric-Level Control

These techniques operate at the typography and syntax level inside the lyrics field. They give you fine-grained control over vocal delivery, timing, pronunciation, and atmosphere that structural tags alone can't provide. Source: Grenar's Suno AI Dirty Tricks series (Days 1–10).

---

### Trick #4 — Live Concert Mode *(Reliability: High)*

**What it does:** Environmental cues inside lyrics bias Suno toward live recordings, audience noise, imperfect timing, and raw vocal delivery. You are not "adding sound effects" — you are changing the performance context.

**Syntax:**

Section tags set the context:
```
[Intro: Live Crowd Cheering]
[Stage Ambience]
[Outro: Applause, Crowd Going Wild]
```

Asterisks add non-sung sound events:
```
*crowd roaring*
*audience cheering*
*applause*
```

Asterisks tell Suno: "this is not to be sung."

**Sound Effects Reference:**

| Syntax | Effect |
|--------|--------|
| `*crowd cheering*` | Applause and screams |
| `*audience singalong*` | Crowd singing along |
| `*festival roar*` | Stadium/festival roar |
| `*thunder*` | Epic moments |
| `*explosion*` | EDM drops |
| `*glass breaking*` | Breaking glass |
| `*café ambience*` | Coffee shop chatter |
| `*rain falling*` | Melancholic atmosphere |
| `*wind howling*` | Isolated, cold feeling |
| `*phone ringing*` | Narrative element |
| `*heartbeat*` | Pulse effect |

**When it fails:** Polished genres (clean pop, EDM), combined with "clean/studio/pristine" keywords, or too many cues stacked. Result: half-studio, half-live confusion.

**Rules:**
- Less is more — one or two live cues are enough
- Match genre — works best with rock, folk, singer-songwriter
- Place cues at structural moments: intro, between sections, outro
- Think "atmosphere," not "specific sound effect"

---

### Trick #5 — Phonetic Respelling *(Reliability: High)*

**What it does:** When Suno consistently mispronounces a word, respell it phonetically. Suno processes text via pattern matching, not linguistic understanding. Homographs (same spelling, different sound) default to the statistically common pronunciation.

**Technique 1 — Simple Respelling:**

| Standard | Phonetic | Why |
|----------|----------|-----|
| read (present tense) | reed | Forces "ree-d" |
| live (as in concert) | lyve | Forces "laiv" |
| bass (instrument) | bahss | Avoids "base" |
| tear (crying) | teer | Forces "teer" |
| wound (injury) | woond | Forces "woond" |
| lead (metal) | led | Forces "led" |

**Technique 2 — Syllable Splitting:**
```
extraordinary → ex-traor-din-ary
catastrophe → ca-tas-tro-phe
pneumonia → new-moan-ya
```

**Technique 3 — IPA (Last Resort):**

Use for one or two stubborn words only. Full IPA for entire lyrics confuses the model.
```
I'm out of /brɛθ/ again     ← forces "breth" not "breathe"
```

**Iteration order:** Simple respelling → hyphen splitting → IPA. Fix one word at a time. Keep a clean "human-readable" version saved separately.

---

### Trick #6 — Homophone Substitution for Content Filters *(Reliability: High when understood)*

**What it does:** Suno's content filter uses exact string matching before vocal synthesis. The vocal model prioritizes phonetic similarity over spelling. A word that sounds identical but is spelled differently can clear the filter while producing the intended sound.

**Why it works — three-stage pipeline:**
1. Content filter scans text for prohibited strings
2. Lyric tokenization converts text to phonetic patterns
3. Vocal synthesis generates audio from phonetic patterns (spelling irrelevant at this stage)

**Common substitutions:**
- `whole/wholes` → sounds like target when sung aggressively
- `dam` → one syllable, matches the target's cadence
- `faux king` → blends at 150+ BPM with fast/punk delivery

**Genre effectiveness:**
- Hip-hop/rap: fast delivery blurs pronunciation — works very well
- Punk/rock: shouted vocals mask exact pronunciation — very effective
- Metal: growled/screamed vocals make pronunciation ambiguous — reliable
- Pop/ballad: clear enunciation makes this difficult — least reliable

**Failure modes:** Phonetic distance too large, syllable count mismatch, filter updates. Expect 70–80% success rate with well-chosen homophones.

**Combining with other tricks:**
- With ALL CAPS (Trick #8): emphasis reinforces the delivery
- With Live Concert Mode (Trick #4): crowd noise further masks pronunciation
- With fast tempo tags: `rapid-fire delivery, breathless, punk speed, aggressive tempo 150+ BPM`

---

### Trick #7 — Rap Cadence Control: Hyphen-Runs + Breath Punctuation *(Reliability: Medium)*

**What it does:** Controls delivery inside a given tempo by manipulating word boundaries and micro-pauses. This is not BPM control — it's delivery control.

**Two levers:**
- **Hyphen-runs** → fewer natural gaps → faster/tighter flow
- **Punctuation** → breath resets → more deliberate cadence

**Syntax reference:**

```
i-hit-the-street-and-i-never-look-back       ← hyphens = speed/tightness
i hit the street, and i never look back       ← comma = micro-breath
i hit the street. i never look back.          ← period = hard reset
i hit the street... and i never look back...  ← ellipsis = drag/suspense
```

Line breaks = phrase/bar resets.

**The knob:**

| Goal | Action |
|------|--------|
| Faster flow | More hyphen-runs, remove commas/periods, shorten lines |
| Slower/clearer | Remove hyphens, add comma every ~6–9 syllables, insert line breaks before punchlines |

> **Rule of thumb:** Hyphens = speed. Punctuation = breath. Line breaks = phrases.

**Failure modes:** Too many hyphens → words smear/become unintelligible. Model starts singing (melisma kills cadence control). Too many control signals stacked simultaneously.

Make 2–3 generations before changing the template. Change **one lever at a time**.

---

### Trick #8 — ALL CAPS for Emotional Spikes *(Reliability: Very High)*

**What it does:** Capital letters signal emphasis, urgency, and intensity. Suno interprets ALL CAPS as "increase vocal energy" — affects loudness, aggressiveness, phrasing, and articulation.

**Why it works:** Training data strongly associates ALL CAPS and exclamation points with emotional outbursts. Extremely stable across models.

**Syntax reference:**

| Format | Effect |
|--------|--------|
| ALL CAPS + ! | Powerful, shouted |
| ALL CAPS + ? | Dramatic, desperate |
| ALL CAPS + ?! | Intense questioning |
| Mixed case + ! | Normal emphasis |
| lowercase | Delicate (when tagged) |

**Example:**
```
[Verse: whispered, intimate]
I tried to keep my voice down
the memories fade away...

[Chorus: explosive]
I still LOVE YOU!
WHY did you go?!
I NEED YOU here with me!
```

**Key rule:** Contrast is everything. CAPS work *because* they differ from surrounding text. If everything is in caps, nothing is emphatic. Use only for climax moments.

**Failure modes:** Only breaks when everything is in caps (no contrast) or genre discourages strong vocals (ambient, minimal).

---

### Trick #9 — Vocal Stretching and Stuttering *(Reliability: High)*

**What it does:** Hyphens and letter repetition stretch syllables and bias Suno toward longer notes, melismatic delivery, or stuttered phrasing.

**Technique 1 — Stretching (Long Notes):**
```
I lo-o-o-o-ve you
Forever-e-e-e-r-r-r
Sta-a-a-a-ay with me-e-e-e
```
Each hyphen = extended note duration. Best for: ballads, soul, gospel.

**Technique 2 — Stutter Effect:**
```
I-I-I-I want want want you
B-b-b-baby baby b-b-baby
```
Repetitions = autotune-style stutter. Best for: trap, hip-hop, electronic pop.

**Technique 3 — Melodic Spelling:**
```
L-O-V-E spells love
G-O-O-D-B-Y-E means the end
```
Each letter = separate note. Best for: pop hooks, jingles.

**Rules:** Use sparingly on key words only. Match tempo — stretching works better slow. Trap loves stuttering; folk doesn't.

**Failure modes:** Overuse throughout entire song, fast tempos, dense lyrics, or genre that doesn't support extended vocals → garbled or ignored.

---

### Trick #10 — Parentheses for Background Vocals *(Reliability: Very High)*

**What it does:** Text in `(parentheses)` is treated as secondary/background. Suno renders it as backing vocals, harmonies, call-and-response, or adlibs. Parentheses act as a syntactic hierarchy signal: everything inside is subordinate to the main vocal line.

**By genre:**

```
Hip-Hop/R&B (call-and-response):
I'm walking alone (walking alone)
Through the city lights (ooh ooh ooh)

Gospel/Soul:
He lifted me up (lifted me up!)
Out of the darkness (hallelujah!)

Trap adlibs:
I got that money (yeah!)
Stacking it high (skrrt skrrt!)
```

**Multiple layers:**
```
I'm breaking free (breaking free) (ooh) (yeah yeah)
```
Each parenthesis = separate vocal layer. More = more backing vocal density.

**Rules:** Use consistently to establish the pattern early. Vary content — mix echoes, adlibs, harmonies. Max 1–2 parentheses per line. Match genre conventions: hip-hop = adlibs, soul = harmonies.

**Failure modes:** Collapses to unison at extremely fast tempos, overly dense lyrics, or too many parentheses per line.

---

### Trick #11 — Tag Stacking with Pipe Symbol *(Reliability: High)*

**What it does:** The `|` pipe symbol separates tag instructions cleanly inside a section header. Without it, long comma-separated adjective lists blur together and the model doesn't know which to prioritize.

**The problem:**
```
[Chorus: Anthemic, Powerful, Epic, Orchestral, Dramatic, Climactic, Intense, Explosive, Grandiose]
```
9 adjectives = AI doesn't know what to prioritize. Instructions collapse.

**The solution:**
```
[Chorus | Anthemic | Stacked Harmonies | Brass Section | Bass Drop | Wide Stereo | Heavy Compression]
```
Each element is processed as a discrete instruction. Priority flows left to right — most important first.

**Optimal element order:**
```
[Section | Mood/Energy | Vocal Style | Key Instruments | Dynamic/Movement | Spatial/Effects | Production Style]
```

**Genre examples:**

```
EDM Drop:
[Drop | Explosive | Vocoder Vocals | Synth Lead | Bass Drop | Sidechain Compression | Wall of Sound]

Jazz Ballad:
[Verse | Intimate | Smooth Female Vocals | Piano Solo | Gentle Dynamics | Close Mic | Warm Analog]

Rock Anthem:
[Chorus | Anthemic | Full Band | Distorted Guitars | Building Energy | Wide Stereo | Arena Sound]
```

**Full example combining with ALL CAPS (Trick #8):**
```
[Intro | Atmospheric | Soft Synth Pad | Building]

[Verse | Intimate | Whispered Male Vocals | Minimal Beat | Lo-Fi Texture | Close and Personal]
I've been lost in the city lights
Searching for meaning every night

[Chorus | Explosive | Full Voice | Layered Synths | Driving Beat | Wide Stereo | Euphoric]
BUT NOW I'VE FOUND MY WAY!
NOTHING CAN STOP ME TODAY!

[Outro | Fading | Reverb Decay | Peaceful Resolution]
```

**Rules:**
- **Max 7 elements per tag** — beyond that, confusion increases
- **Priority order matters** — most important element goes first
- **Be specific** — "Guitar Solo" not just "Guitar"
- **Tags should reinforce the style prompt**, not contradict it

**Failure modes:** More than 7 elements, contradictory elements in the same tag (e.g., `Intimate | Explosive`), or used without clear section structure.

---

### Trick #12 — Vocal Adlibs with Inline Brackets *(Reliability: High)*

**What it does:** Square brackets `[...]` *inside* a lyric line (not at section start) produce adlibs, interjections, and non-melodic vocal sounds.

**Syntax:**
```
I'm on top [yeah!] of the world [uh!]
Living my life [let's go!] no regrets [ayy!]
```

**Adlibs by genre:**

| Genre | Common Adlibs |
|-------|--------------|
| Hip-Hop/Trap | `[yeah!]`, `[uh!]`, `[ayy!]`, `[skrrt!]`, `[woah!]`, `[let's go!]` |
| R&B/Soul | `[ooh]`, `[ahhh]`, `[mmm]`, `[oh yeah]`, `[baby]` |
| Rock/Metal | `[hey!]`, `[go!]`, `[yeah!]`, `[whoa-oh]`, `[come on!]` |
| Pop | `[oh oh oh]`, `[na na na]`, `[la la la]`, `[hey hey]` |
| Latin | `[dale!]`, `[oye!]`, `[vamos!]` |

**Rules:** Adlibs = punctuation, not sentences. 2–3 per verse is plenty. Place at end of lines or between phrases, not mid-word.

**Failure modes:** Every word has a bracket, lyrics are overcrowded, genre doesn't use adlibs (classical, ambient).

---

### Trick #13 — "Broadway" as Vocal Clarity Override *(Reliability: High)*

**What it does:** Adding `Broadway` or `Broadway musical clarity` to the style prompt biases Suno toward:
- Hard consonants (T/D/K/P/S land clean)
- Fully formed vowels (less smearing)
- Tighter rhythm (syllables align to the beat grid)
- Dry, forward vocal presence (less wash, more speech-band clarity)

**What it does NOT do:** Turn the track into a show tune. It is strictly a clarity multiplier for the vocals.

**Why it works:** "Broadway" in training data is strongly associated with dialogue-like intelligibility and vocal projection — even during high-emotion moments. The model stops "smearing" and starts treating lyrics like actual sentences that happen to be sung.

**Usage:**
```
Broadway musical clarity (story-first enunciation, hard consonants, centered vowels)
```

**Prompt variants — keep Broadway, swap texture:**

```
Variant A (industrial dark pop):
Broadway clarity, dry forward vocal, industrial pop, dark electro, distorted synth bass, minimal reverb, tight drums

Variant B (cinematic trip-hop noir):
Broadway clarity, close-mic vocal, minimal reverb, trip-hop, noir cinematic, vinyl crackle, slow tension, sub bass

Variant C (hyperpop precision):
Broadway clarity, crisp consonants, dry vocal, hyperpop textures, bright synths, tight quantized drums
```

**Failure modes:** Highly anti-theatrical genre tags (`grunge`, `lo-fi mumble`, `shoegaze`), stacked with heavy reverb keywords, too-dense lyrics, or combined with distortion/screaming (clarity conflicts with distortion).

**Troubleshooting:**
- Too theatrical → reinforce texture: add `dark electronic`, `alt-pop noir`
- Still murky → add `dry vocal, close mic, upfront`, remove reverb-heavy words
- Too emotionally flat → add `controlled anger / restrained grief / contained intensity`
- Consonants too harsh → add `smooth sibilance`
- Trick ignored → move "Broadway" earlier in the style line, delete conflicting tags

---

### Grenar's Dirty Tricks — Summary Table

| Trick | Tool | Reliability | Best For |
|-------|------|-------------|----------|
| #4 Live Concert | `*asterisks*`, `[Intro: Live Crowd]` | High | Rock, folk, singer-songwriter |
| #5 Phonetic Respelling | `reed`, `lyve`, `/brɛθ/` | High | Any genre with pronunciation issues |
| #6 Homophone Filter Bypass | `whole`, `dam`, `faux king` | High (70–80%) | Punk, metal, hip-hop |
| #7 Rap Cadence | `hyphen-runs`, `,` `.` `...` | Medium | Hip-hop, trap, rap |
| #8 ALL CAPS | `WORD!`, `WORD?!` | Very High | Any genre with dynamic contrast |
| #9 Stretching/Stutter | `lo-o-ove`, `B-b-baby` | High | Ballads, trap, soul |
| #10 Background Vocals | `(echo)`, `(yeah!)` | Very High | R&B, soul, hip-hop |
| #11 Pipe Tag Stacking | `[Section \| Mood \| Instrument \| ...]` | High | Any genre needing section-level control |
| #12 Inline Adlibs | `[yeah!]`, `[ayy!]` | High | Trap, hip-hop, rock |
| #13 Broadway Clarity | `Broadway musical clarity` | High | Any genre needing intelligible vocals |

---

## 11. Album Art & Thumbnails

### Format

Always start with `album art` to establish context:

```
album art: [main subject/vibe], [background/scene], [visual style], [colors/mood], text reads "TITLE"
```

### Working Examples

```
album art: psychedelic desert scene, super trippy, cactus and iguana and hot sun, vivid colors

album art: statuesque feminine figure emerging from cosmic darkness, classical features barely 
visible beneath translucent hood, wings of pure void, flowing dress moving like living nebula, 
stars visible through her ethereal form, text "NYX" written where starlight meets shadow

album art: ethereal night sky scene, deep purple and blue cosmos, massive spiral galaxy center 
frame, small silhouetted figure on mountainous ridge, telescope pointing upward, subtle lens 
flare effects, cosmic dust wisping across background, minimalist text "STARGAZER" in thin 
glowing font at bottom
```

### Rules

1. Start with `album art` for context
2. Main style/genre next (context first)
3. Main subject and scene construction
4. Colors, emotions, details, text last
5. **Less text is better** — one or two words maximum
6. Generate 4–5 variations per song; mix detailed and minimal

---

## 12. Quick Reference

### Prompt Anatomy
```
genre: "[Genre], [BPM] BPM, [Emotion], [Specific Instrument], [Vocal Type], [Era/Reference]"
```

### Reliable Descriptors
```
anthemic chorus / building intensity / desperate / haunting / bittersweet
operatic / gritty vocals / driving rhythm / punishing drums / emotional build-up
```

### Unreliable / Ignored
```
ethereal / lush / shimmering / crescendo (use "building intensity") / realistic
```

### Most Useful Structure Tags
```
[Verse] [Chorus] [Bridge] [Building Intensity] [Stripped Back]
[Explosive Chorus] [Sudden Break] [Fade Out] [Orchestral Break]
[Massive Choir] [Call and Response] [Melodic Interlude]
```

### Extend Workflow
```
1. Engineer one precise prompt (emotion + instrument + BPM + structure intent)
2. Generate intro section only
3. Extend section-by-section, directing each
4. Run best prompt 20 times, pick the top result
5. Master before publishing
```

### Assemble Workflow
```
1. Lock BPM + key in the style field (same values every generation)
2. Generate 4–6 candidates from the same prompt
3. Assemble best sections in Suno Studio: verse from one, chorus from another
4. For instrumental breaks: stem-split best gen, Cover-Remix the instrumental
5. Extract solos/breakdowns, drop into arrangement, export
```

### The Success Formula
```
BASE GENRE + DOMINANT MOOD + LEAD INSTRUMENT + VOCAL STYLE +
ATMOSPHERE + PRODUCTION + STRUCTURE + PERSONAL TOUCH = TARGET TRACK
```

---

## 13. AI Lyric Anti-Patterns

### Words to Avoid or Use Very Sparingly

**Overused nouns:**
crown, throne, kingdom, neon, halo, wings, angels, ghost(s), chains, shackles,
ashes, embers, flames, void, abyss, maze, armor, sword, canvas, storm, mirror,
demons, echoes, ruins, shadows, silence

**Banned phrases:**
"ghost in the machine" / "rise from the ashes" / "broken wings" / "dance with the devil"
"weight of the world" / "heart of gold" / "lost in the dark" / "find the light"
"walls come crashing down" / "demons inside" / "fire and ice" / "written in the stars"
"against all odds" / "eye of the storm" / "chasing dreams" / "through the fire"

**Lazy rhyme pairs to avoid:**
fire/desire / heart/apart/start / night/light/fight/sight / pain/rain/vain
soul/whole/control / time/mind/blind / breath/death / tears/fears/years
burn/turn/learn / fall/all/wall

**Adjective crutches:**
endless, eternal, hollow, broken, shattered, fading, crimson, golden, empty, frozen

### What Makes Lyrics Feel Human

- **Concrete sensory imagery:** "cold wind through an open door" beats "feeling so alone"
- **Verbs of subtle motion:** linger, drift, shift, slip, pull, settle, press
- **Temporal/spatial specificity:** "3 AM kitchen light" beats "the darkness of night"
- **Earned complexity:** tension between competing emotions, not simple declarations
- **Imperfect moments:** small, unglamorous details that feel lived-in
- **Subverted expectations:** set up a cliché, then pivot away
- **Rhythmic variation:** mix line lengths, don't make every line symmetrical

### Formatting Standards

- Use hyphens `(-)` for lyrical breaks, not em dashes `(—)`
- Correct: "I lose you - then find you again"
- Avoid excessive ellipses `(...)` — one per song maximum
- Let lines breathe; not every bar needs dense metaphor

### AI Lyric Generation Prompt Template

```
Write lyrics for a song with the following concept:

**Song Title:** [TITLE]
**Core Theme:** [e.g., "confronting self-deception"]
**Emotional Arc:** [e.g., "denial → confrontation → painful acceptance → quiet resolve"]
**Perspective:** [e.g., "first person, present tense, internal monologue"]

**Musical Context:**
- Genre: [GENRE]
- Tempo/Feel: [TEMPO AND FEEL]
- Structure: [e.g., "Verse 1 / Pre-Chorus / Chorus / Verse 2 / Pre-Chorus / Chorus / Bridge / Final Chorus / Outro"]

**Lyrical Style Requirements:**
1. Use concrete sensory imagery over abstract statements
2. Prefer verbs of subtle motion (linger, drift, shift, slip, settle, press)
3. Include at least one specific, unglamorous detail that feels lived-in
4. Vary line lengths — not every line should be symmetrical
5. Use hyphens (-) for breaks, not em dashes
6. Subvert at least one expected rhyme or phrase

**CRITICAL — Avoid:** [paste avoid lists above]

**Additional Context:** [specific imagery, references, constraints]

Write the complete lyrics with section markers. Make every line earn its place.
```

### Post-Generation Checklist

- [ ] No words from the avoid noun list used lazily
- [ ] No banned phrases
- [ ] No more than one lazy rhyme pair (ideally zero)
- [ ] At least 3 concrete sensory images
- [ ] At least 1 specific, unglamorous detail
- [ ] Hyphens used correctly (not em dashes)
- [ ] Line lengths vary naturally
- [ ] Emotional arc progresses — end differs from beginning
- [ ] Could a human have written this? Does it feel lived-in?

---

---

## 14. Sources

This guide synthesizes material from the following sources. Each is worth reading in full for context this guide couldn't fully capture.

---

### Primary Sources

**[1] r/SunoAI — "After mass-generating 200+ tracks I finally figured out which prompt words actually work"**
- Author: u/Budget_Coach9124
- URL: https://www.reddit.com/r/SunoAI/comments/1ruey5m/
- Content: Field-tested prompt word effectiveness from 200+ generation run. Source for Section 3 prompt word table and the quality-over-volume principle in Section 9.

**[2] Suno AI Song Syntax**
- Author: daveshap (GitHub)
- URL: https://github.com/daveshap/suno
- Content: Comprehensive tag syntax reference — base tags, modifier behavior, period/exclamation rhythm system, vocal and instrument tags, style field construction, album art prompting. Primary source for Section 4 (structure/tags) and Section 10 (album art).

**[3] SunoAI Complete Meta Tags Guide**
- Author: Shawn McAllister (@stonedoubt / entrepeneur4lyf)
- URL: https://github.com/entrepeneur4lyf/suno_ai_meta_tags_guide
- Content: Full meta tag taxonomy covering structure, mood, energy, instruments, genre, vocal, production, effects, keys/scales, chord progressions, rhythm, and advanced techniques. Genre-specific templates and parameter settings. Primary source for Sections 5, 8, and the AI lyric generation template in Section 13.

**[4] The Complete Guide to Mastering Suno: Advanced Strategies for Professional Music Generation**
- Author: Unknown (community-compiled Notion document)
- URL: https://www.notion.so/The-Complete-Guide-to-Mastering-Suno-Advanced-Strategies-for-Professional-Music-Generation-2d6ae744ebdf8024be42f6645f884221
- Content: Deep-dive on co-occurrence mechanics, genre clouds, the pop gravity well, structured prompt formats, MAX Mode, lyric bleed, model comparisons, realism vocabulary, anti-sawtooth strategy, cover-based remastering, song transplant technique, persona building, and lyric writing methodology. Primary source for Sections 1, 2, 3, 7, and 9.

---

### Grenar's Dirty Tricks Series

All posts by u/Grenar, published on r/AiPizza. Primary source for Section 10.

**[5] Day #1 — Trick #4: Live Concert Mode with Sound Effects**
- URL: https://www.reddit.com/r/AiPizza/comments/1rasrea/
- Content: Asterisk syntax for environmental sound cues, section tags for live atmosphere, full reference table of sound effects.

**[6] Day #2 — Trick #5: Phonetic Respelling for Pronunciation Control**
- URL: https://www.reddit.com/r/AiPizza/comments/1rc0b9r/
- Content: Simple respelling, syllable splitting with hyphens, IPA usage for stubborn words.

**[7] Day #3 — Trick #6: Alternative Spelling for Content Filters**
- URL: https://www.reddit.com/r/AiPizza/comments/1rcfs3o/
- Content: How Suno's three-stage pipeline (filter → tokenization → synthesis) creates a phonetic gap that homophones can exploit.

**[8] Day #4 — Trick #7: Rap Cadence Control**
- URL: https://www.reddit.com/r/AiPizza/comments/1rd8ks2/
- Content: Hyphen-runs for flow tightening, punctuation for breath control, one-lever-at-a-time iteration method.

**[9] Day #5 — Trick #8: ALL CAPS for Emotional Spikes**
- URL: https://www.reddit.com/r/AiPizza/comments/1rf9bw7/
- Content: Typography as vocal intensity control, contrast principle, punctuation modifiers.

**[10] Day #6 — Trick #9: Vocal Stretching and Stuttering**
- URL: https://www.reddit.com/r/AiPizza/comments/1rgj8ra/
- Content: Hyphen stretching for long notes, repetition for stutter/autotune effect, letter-by-letter melodic spelling.

**[11] Day #7 — Trick #10: Parentheses for Background Vocals**
- URL: https://www.reddit.com/r/AiPizza/comments/1rhgwid/
- Content: Parentheses as hierarchical subordination signal, multiple layers, genre-specific applications.

**[12] Day #8 — Trick #11: Tag Stacking with Pipe Symbol**
- URL: https://www.reddit.com/r/AiPizza/comments/1rjvdlq/
- Content: Pipe operator for clean multi-element section tags, optimal element order, max 7 elements rule, genre examples.

**[13] Day #9 — Trick #12: Vocal Adlibs with Brackets**
- URL: https://www.reddit.com/r/AiPizza/comments/1rnjduv/
- Content: Inline square brackets for adlibs and interjections, genre-specific adlib reference table.

**[14] Day #10 — Trick #13: "Broadway" as a Clarity Override**
- URL: https://www.reddit.com/r/AiPizza/comments/1rr1eon/
- Content: Broadway as vocal diction shorthand, prompt variants, troubleshooting murky vocals.

*Note: Grenar's full series (Days 1–10+) is also available as paid books on Amazon and Etsy. The posts above are the free Reddit versions.*

---

### Additional References

**[15] LedgerNote — Music Production Columns**
- URL: https://ledgernote.com/columns/
- Use: Genre and instrument vocabulary reference for prompt construction.

**[16] Music Theory Academy — Musical Structures**
- URL: https://www.musictheoryacademy.com/understanding-music/musical-structures/
- Use: Song structure reference for understanding section roles and arrangement logic.

**[17] Fantasy Bands End-to-End Walkthrough (Patreon)**
- Author: u/MrAndyPuppy
- URL: https://www.patreon.com/posts/fantasy-bands-137134953
- Use: Full walkthrough of creating a complete project in Suno from concept to finished track. May require free Patreon membership.

**[18] How to Write Great Prompts for Suno**
- Author: u/AnalystAI
- URL: https://www.reddit.com/r/SunoAI/comments/1qk766q/
- Content: Simple vs. Custom Mode framework, 5-question pre-prompt checklist, guitar+voice isolation technique, two-songs-in-one splicing workflow, troubleshooting table. Source for new subsections in Section 9.

**[19] Tips & Tricks for Using the New Model (Official Suno Post)**
- Author: u/Suno_helper (Suno team)
- URL: https://www.reddit.com/r/SunoAI/comments/1nudoue/
- Content: Official v5 guidance — slider usage, prompt complexity scaling, Inspo 1–4 songs, Drag & Drop Remix/Inspo, Cover/Persona length behavior. Source for "Official Suno v5 Guidance" in Section 9.

**[20] What Mastering Do You Perform on Finished Tracks?**
- Author: u/Key-Call2867 (thread), multiple contributors
- URL: https://www.reddit.com/r/SunoAI/comments/1rdu5e5/
- Content: Community consensus on Suno's over-compression problem, LUFS targets, AI mastering services, DAW recommendations (Reaper), UVR5 for stems. Source for "Post-Processing / Mastering" in Section 9.

**[21] Every Iteration of Suno Gets Worse for Metal**
- Author: u/BunnyPig1 (thread), multiple contributors
- URL: https://www.reddit.com/r/SunoAI/comments/1nop8q6/
- Content: Key insight that vocal type tags work better in Style field than lyrics box; descriptive paragraph prompting beats genre stacking for extreme genres; ChatGPT keyword extraction trick. Source for "Vocal Type Tags" in Section 9.

**[22] Personal Tips and Tricks**
- Author: u/Lucide_Reveur
- URL: https://www.reddit.com/r/SunoAI/comments/1in79lh/
- Content: Persona creation workaround (remaster → persona); public domain lyrics as source material; Wikipedia genre taxonomy for finding accurate style descriptors. Source for "Persona Workaround" in Section 9.

**[23] Sharing a Global + Lyric Prompt for Trap Music**
- Author: u/Scared-Emergency4157
- URL: https://www.reddit.com/r/SunoAI/comments/1qrf21y/
- Content: Complete dark trap template demonstrating separate global style + lyric structure with alternating instrumentation control. Source for "Complete Dark Trap Template" in Section 9.

**[24] My Complete Workflow for Writing Authentic Lyrics**
- Author: u/doomsboygamingv2
- URL: https://www.reddit.com/r/SunoAI/comments/1of8wt9/
- Content: Supplement to Section 13 — AI cliché word list, specific time trap (3 AM, 2 AM), academic language red flags, multi-stage writing process. Partial overlap with existing Section 13.

**[25] Guide: Stop Brute-Forcing Suno Like a Slot Machine**
- Author: u/budmaniak
- URL: https://www.reddit.com/r/SunoAI/comments/1tooamm/
- Content: Studio-centric assembly workflow — BPM/key locking, 4–6 generation rule, Replace/Sample/Cover lyric-fix methods with specific slider settings, stem-split + Cover-Remix for instrumental breaks. Source for "Assemble From Parts" in Section 9 and the prompt-adherence-decays note in Section 1.

**[26] Suno v5.5: More Expressive. More You. (Official Blog)**
- Author: Suno team
- URL: https://suno.com/blog/v5-5
- Content: Official v5.5 release announcement (March 26, 2026). Voices, Custom Models, My Taste. Source for v5.5 entries in Sections 2 and 9.

**[27] What's New in v5.5 (Help Center)**
- Author: Suno team
- URL: https://help.suno.com/en/articles/11362305
- Content: Help-center overview of v5.5 features. Confirms Custom Models tier gating (Pro/Premier), 3-model maximum, and My Taste availability to all users.

**[28] Voices: Use Your Voice in Suno (Help Center)**
- Author: Suno team
- URL: https://help.suno.com/en/articles/11362369
- Content: Voices upload spec (15s–4min, acapella preferred, auto stem extraction), verification process (spoken-phrase matching), rights-confirmation requirement.

**[29] Custom Models in v5.5 (Help Center)**
- Author: Suno team
- URL: https://help.suno.com/en/articles/11362497
- Content: Custom Models requirements — minimum 6 tracks, 3 models max, Pro/Premier tier, ~2–5 min training time, ownership requirement, bulk upload, no inter-user sharing.

**[30] MILO-1080 Coverage**
- Sources: Music Ally (https://musically.com/2026/03/24/sunos-latest-move-is-milo-1080-an-ai-driven-step-sequencer/), MusicRadar, RouteNote
- Content: 16-track step sequencer + synth designer with MIDI support, launched March 2026 in Suno Labs. Background for the MILO-1080 mention in Section 9.

**[31] Cracking the v5.5 Code: Community Master Prompt Thread**
- Author: u/BuffaloConscious7919 (OP), with contributions from u/Cenn73, u/treidan, u/West-Negotiation-716, u/CevapciciAllin, u/pixellegolas, u/atth3bottom, u/Andeva82, others
- URL: https://www.reddit.com/r/SunoAI/comments/1sfqdlj/
- Content: Field reports on v5.5 problems (mid-track audio degradation, pop reversion, broken Extend). Sources for: "V5.5 known issues" callout in Section 2, "Hybrid Model Workflow" subsection in Section 9, "Voice style-bleed gotcha" in v5.5 subsection, and "v5.5 Intro & Structure Fixes" subsection.

**[32] Suno Help Center: "How do I exclude elements of a song?"**
- Author: Suno team
- URL: https://help.suno.com/en/articles/3161921
- Content: Official documentation of the Exclude field in Custom Mode → Advanced Options. Source for clarifying official-vs-inline exclusion mechanisms in the expanded Section 8 "Exclude Styles" subsection.

**[33] r/SunoAI — "Get rid of the intro hums"**
- Author: u/PureRely
- URL: https://www.reddit.com/r/SunoAI/comments/1u1vxz8/
- Content: Negative-style list targeting wordless intros, hums, and spoken/ambient intros. Source for "Killing Intro Hums & Vocalized Intros" in Section 8.

**[34] r/SunoAI — "Your Suno song has a structure problem — here's the 3-part fix"**
- Author: u/sunoarchitect
- URL: https://www.reddit.com/r/SunoAI/comments/1u7asio/
- Content: Promotional post for a free structure-checking tool (sunoarchitect.com). Treated skeptically — its syllable-consistency and genre-order points duplicate Sections 6 and 5, and the comment thread flags it as AI-generated marketing. Only the honest `[End]` reliability caveat (surfaced by the author in replies) and the unverified tag-placement claim were kept. Source for "Closing the Structure" in Section 4.

---

## 15. Contributing

This guide is a living document. If you have a technique, prompt pattern, or finding that isn't covered here — or if something is wrong or outdated — contributions are welcome.

**To contribute:**
- Open a PR with your addition or correction
- Include a brief note on what you tested, how many generations, and which model version
- Link the source if the technique comes from a community post or external guide
- Keep the voice consistent — practical and direct, no fluff

**What makes a good addition:**
- Field-tested, not theoretical — describe what actually happened
- Specific enough to reproduce — vague tips like "be more creative" don't help
- Placed in the right section — or propose a new one if nothing fits

**What to flag as an issue rather than a PR:**
- Techniques that have stopped working (model may have changed)
- Broken source links
- Sections that need updating for a new model version

---

*Last updated: June 2026. Suno is a rapidly evolving platform — techniques may change as models update. When something stops working, the model has likely changed, not you.*
