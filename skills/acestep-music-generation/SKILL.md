---
name: acestep-music-generation
description: Use when generating AI music with ACE-Step V1.5, creating lyrics with brainstorming, or using music-arrangement skills for emotional instrument selection
---

# ACE-Step Music Generation

## Overview

Generate AI music using ACE-Step V1.5 with lyrics and instrumental versions. Combines brainstorming for lyrics, music-arrangement for emotional instrument selection, and ACE-Step for local music generation.

## Workflow

### 1. Lyrics Creation (Brainstorming)

1. Load `superpowers:brainstorming` skill
2. Ask clarifying questions about emotional tone, target audience, music style
3. Create design spec in `docs/superpowers/specs/`
4. Write lyrics file

### 2. Music Arrangement (Music-Arrangement)

1. Load `superpowers:music-arrangement` skill
2. Map emotion to instruments using the emotion-instrument table
3. Generate English prompt for ACE-Step

**Emotion-Instrument Mapping:**

| Emotion | Instruments | Keywords |
|---------|-------------|----------|
| 悲伤/孤独 | 钢琴、大提琴 | Sparse arrangement, Minor key, Solo piano |
| 燃/激昂 | 铜管、定音鼓、合成器 | Epic orchestral, Powerful brass |
| 温馨/治愈 | 木吉他、钢片琴、长笛 | Warm acoustic, Gentle strumming |
| 梦幻/空灵 | 合成器Pad、八音盒 | Ambient, Ethereal, Washed out reverb |
| 快乐/活泼 | 尤克里里、钢片琴 | Upbeat, Plucky, Major key |

### 3. ACE-Step Generation

**Required Parameters:**
- `task_type`: "text2music"
- `caption`: English music description
- `lyrics`: Chinese lyrics text
- `duration`: Duration in seconds
- `vocal_language`: "zh" for Chinese
- `bpm`: Beats per minute
- `instrumental`: True/False

**Generation Command:**
```python
$env:PYTHONIOENCODING="utf-8"
& "D:/ACE-Step-1.5/python_embeded/python.exe" -X utf8 generate_music.py
```

### 4. Python Script Template

```python
import os
import sys
os.chdir(r"D:/ACE-Step-1.5")
sys.path.insert(0, r"D:/ACE-Step-1.5")

for v in ['http_proxy', 'https_proxy', 'HTTP_PROXY', 'HTTPS_PROXY', 'ALL_PROXY']:
    os.environ.pop(v, None)

from acestep.handler import AceStepHandler
from acestep.llm_inference import LLMHandler
from acestep.inference import GenerationParams, GenerationConfig, generate_music
from acestep.gpu_config import get_gpu_config, set_global_gpu_config

gpu_config = get_gpu_config()
set_global_gpu_config(gpu_config)

lyrics = """你的歌词"""

params = GenerationParams()
params.task_type = "text2music"
params.caption = "Warm acoustic ballad, tender nostalgic..."
params.lyrics = lyrics
params.duration = 180
params.vocal_language = "zh"
params.bpm = 75
params.instrumental = False
params.thinking = False

config = GenerationConfig()
config.batch_size = 1
config.audio_format = "wav"
config.seeds = [42]

output_dir = r"D:/music_project/output"
os.makedirs(output_dir, exist_ok=True)
config.save_dir = output_dir

dit_handler = AceStepHandler()
dit_handler.initialize_service(project_root=r"D:/ACE-Step-1.5", config_path="acestep-v15-turbo", device="cuda")

llm_handler = LLMHandler()
llm_handler._skip_prompt_edit = True

result = generate_music(params=params, config=config, dit_handler=dit_handler, llm_handler=llm_handler)

if result.audios:
    import torchaudio
    torchaudio.save(os.path.join(output_dir, "output.wav"), result.audios[0]['tensor'].cpu(), 48000)
```

### 5. Output Files

| Type | Filename Pattern |
|------|------------------|
| Vocal | `*_vocal.wav` |
| Instrumental | `*_instrumental.wav` |

## Quick Reference

- **Chinese lyrics**: Use `\n` for line breaks, no quotes
- **English caption**: Describe instruments, mood, vocals
- **BPM range**: 70-90 for ballad, 120+ for upbeat
- **Duration**: 180s standard, up to 600s max
- **Encoding**: Always use `-X utf8` flag on Windows

## Common Issues

| Issue | Solution |
|-------|----------|
| UnicodeEncodeError | Use `-X utf8` flag |
| Model not initialized | Call `initialize_service()` explicitly |
| Chinese chars garbled | Use UTF-8 encoding throughout |
| KV cache error | Restart Python process, clear GPU memory |
