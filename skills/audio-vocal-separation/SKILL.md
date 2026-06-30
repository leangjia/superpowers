---
name: audio-vocal-separation
description: Use when separating vocals and accompaniment from audio files using Demucs, extracting melody for sheet music, or converting audio to MIDI
---

# Audio Vocal Separation

使用 Demucs 从音频文件中分离人声和伴奏，或提取旋律生成简谱/五线谱。

## 环境配置

- **虚拟环境**: `D:/spleeter_env/`
- **Python**: `D:/spleeter_env/Scripts/python.exe`
- **Demucs**: `D:/spleeter_env/Scripts/demucs-infer.exe`
- **模型**: htdemucs (Meta Demucs v4, 效果好于 Spleeter)

## 分离人声和伴奏

### 命令格式

```bash
D:/spleeter_env/Scripts/demucs-infer.exe -n htdemucs -o OUTPUT_DIR --two-stems vocals --shifts 1 INPUT_WAV
```

### 参数说明

| 参数 | 说明 | 推荐值 |
|------|------|-------|
| `-n htdemucs` | 使用 htdemucs 模型 | 必须 |
| `-o` | 输出目录 | 自定义 |
| `--two-stems vocals` | 分离为人声+伴奏 | 必须 |
| `--shifts` | 增强质量(时间翻倍) | 1 |

### 输出文件

分离后在 `OUTPUT_DIR/htdemucs/文件名/` 目录下:
- `vocals.wav` - 人声
- `no_vocals.wav` - 伴奏

### 完整示例

```bash
# 分离音频
D:/spleeter_env/Scripts/demucs-infer.exe -n htdemucs -o D:/music_project/output/demucs_song --two-stems vocals --shifts 1 "D:/music_project/song.wav"

# 复制到项目目录
D:/ACE-Step-1.5/python_embeded/python.exe -c "import shutil; shutil.copy('D:/music_project/output/demucs_song/htdemucs/song/vocals.wav', 'D:/project/song_分离人声.wav'); shutil.copy('D:/music_project/output/demucs_song/htdemucs/song/no_vocals.wav', 'D:/project/song_分离伴奏.wav')"
```

## 从人声提取简谱

### 1. 提取旋律 (Python)

```python
import librosa
import numpy as np

y, sr = librosa.load('vocals.wav', sr=44100)
f0, voiced, probs = librosa.pyin(y, fmin=librosa.note_to_hz('C3'), fmax=librosa.note_to_hz('C6'), sr=sr, hop_length=512)

# 转换为音符
NOTE_FREQS = {'C4': 261.63, 'D4': 293.66, 'E4': 329.63, 'F4': 349.23, 'G4': 392.00, 'A4': 440.00, 'B4': 493.88}
# ... (见下方脚本)
```

### 2. 生成五线谱 PDF

需要 MuseScore 3:
- 安装路径: `C:/Program Files/MuseScore 3/bin/MuseScore3.exe`
- music21 → MusicXML → PDF

```python
from music21 import stream, note, tempo, meter, clef
import subprocess

# 1. 创建 music21 stream
score = stream.Score()
part = stream.Part()
part.append(tempo.MetronomeMark(number=95))
part.append(meter.TimeSignature('4/4'))
part.append(clef.TrebleClef())
# 添加音符...
score.insert(0, part)

# 2. 导出 MusicXML
score.write('musicxml', fp='output.xml')

# 3. 用 MuseScore 转换 PDF
subprocess.run(['C:/Program Files/MuseScore 3/bin/MuseScore3.exe', '-o', 'output.pdf', 'output.xml'])
```

### 3. 简化脚本位置

```
D:/spleeter_env/vocal_to_midi.py     - 人声转 MIDI + PDF
D:/spleeter_env/accomp_to_midi.py   - 伴奏转 MIDI + PDF
```

## 已知问题

| 问题 | 原因 | 解决 |
|------|------|------|
| demucs RuntimeError | torch 2.11+ 不兼容 | 使用 torch 2.10 或等待修复 |
| music21 PDF不支持 | 需MuseScore | 导出MusicXML再转PDF |

## 快速调用

### 分离单文件

```bash
D:/spleeter_env/Scripts/demucs-infer.exe -n htdemucs -o OUTPUT --two-stems vocals --shifts 1 INPUT.wav
```

### 批量分离

```bash
for f in *.wav; do
  D:/spleeter_env/Scripts/demucs-infer.exe -n htdemucs -o "../demucs_${f%.wav}" --two-stems vocals --shifts 1 "$f"
done
```

## 输出文件命名规范

| 类型 | 命名格式 |
|------|---------|
| 分离人声 | `歌曲名_分离人声.wav` |
| 分离伴奏 | `歌曲名_分离伴奏.wav` |
| 简谱��本 | `简谱_人声.txt`, `简谱_伴奏.txt` |
| 五线谱PDF | `五线谱_人声.pdf`, `五线谱_伴奏.pdf` |
| MIDI | `歌曲名_人声.mid`, `歌曲名_伴奏.mid` |