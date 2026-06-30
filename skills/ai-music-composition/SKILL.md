---
name: ai-music-composition
description: Use when generating AI music, creating emotional soundtracks, or selecting instruments for specific moods in Suno, MiniMax Music, or Tunee
---

# AI 音乐创作技能

**语言：** 永远使用中文

## Overview

Bridge the gap between emotional intent and AI-generated music. Use instrument-emotion mapping to craft precise prompts that tell AI exactly what instrumentation and mood you want.

## When to Use

- Generating background music for videos, games, or podcasts
- Creating emotional soundtracks with specific moods
- Choosing instruments for scene descriptions
- Writing AI music prompts with professional terminology

## Emotion-Instrument Mapping

| Emotion | Core Instruments | AI Keywords |
|---------|------------------|-------------|
| Sad/Lonely | Piano, Cello, Solo Violin | `Sparse arrangement, Minor key, Solo piano, Dark reverb` |
| Epic/Thrilling | Brass, Timpani, Distortion Guitar | `Epic orchestral, Powerful brass staccato, Rising tension` |
| Warm/Healing | Acoustic Guitar, Glockenspiel, Flute | `Warm acoustic, Gentle strumming, Bright major key` |
| Tense/Sci-fi | Synth Pad, Arpeggio, Contrabass pizzicato | `Tension, Dissonant harmony, Filter sweep, Minimalist repetitive` |
| Dreamy/Ethereal | Synth Pad, Music Box, Flute | `Ambient, Ethereal, Washed out reverb, Slow attack pad` |
| Happy/Playful | Ukulele, Glockenspiel, Cajon | `Upbeat, Plucky, Major key, Sparse percussion` |
| Epic/Cinematic | Full Orchestra, Timpani | `Cinematic orchestral, Sweeping strings, Heroic brass` |
| Sensual/Jazz | Saxophone, Electric Piano, Brush drums | `Smooth jazz, Sensual, Mellow saxophone, 7th chords` |
| Ominous/Weird | Atonal piano, Waterphone, Glitch | `Atonal, Glitchy, Unsettling, Dissonant clusters` |

## Three-Step Prompt Customization

### Step 1: Replace Core Instrument

| Template | Alternative | Result |
|----------|-------------|--------|
| Piano | Cello | Sad becomes deeper |
| Acoustic Guitar | Music Box | Warm becomes nostalgic |
| Brass | Distortion Guitar | Epic becomes rock |

### Step 2: Add Detail Modifier

- "Piano" → "Out-of-tune old piano" (adds decay)
- "Strings" → "Viola only, no violin" (adds gloom)
- "Drums" → "Loose kick drum" (adds laziness)

### Step 3: Remove Unwanted Elements

```
不要弦乐群 (No string ensemble)
不要打击乐 (No percussion)
不要任何电子音色 (No electronic sounds)
```

## Quick Reference: 7 Emotion Templates

### 1. Sad/Lonely
Slow piano solo, sparse arrangement, minor key, dark reverb, isolated high notes

### 2. Epic/Thrilling
Timpani rolls first 15 seconds, then brass unison, full drums, dense orchestration

### 3. Warm/Healing
Acoustic guitar strumming, major key, no electronics, relaxed tempo

### 4. Dreamy/Ethereal
Synth pad foundation, music box melody, heavy reverb/delay, loose rhythm

### 5. Tense/Suspense
Contrabass pizzicato, sustained synth bass, dissonant high strings, irregular rhythm

### 6. Happy/Playful
Ukulele or glockenspiel lead, light percussion, major key, bouncy rhythm

### 7. Cinematic Narrative
Full orchestra, strings bed, brass melody, timpani accents, clear structure (intro-development-climax-outro)

## Common Mistakes

| Mistake | Correction |
|---------|------------|
| AI gives happy sad music | Add `Minor key` explicitly |
| Too many instruments | Use `Sparse arrangement` |
| Sounds generic | Add specific detail modifier |
| Missing "breathing room" | Add `Build-up` before climax |

## Pro Tips

1. **Describe the silence too**: "Uneasy silence filled with low-end rumble" creates better tension than just "Anxious"

2. **Even epic music needs rest**: Add `Build-up` before climax - momentary quiet amplifies impact

3. **Large ensembles lose emotion**: For sadness, solo instruments work better than string sections

4. **Specify what NOT to play**: AI often defaults to certain instruments - explicitly exclude unwanted sounds

## Full Prompt Templates

See `prompts.md` for complete prompt templates with Chinese and English versions.
