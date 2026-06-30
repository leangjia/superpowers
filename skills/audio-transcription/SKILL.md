---
name: audio-transcription
description: Use when converting audio to MIDI, simplified notation (简谱), or staff notation (五线谱). Includes pitch detection, melody extraction, and MIDI export.
---

# Audio Transcription Skill

将音频转换为简谱、五线谱或MIDI。一站式音乐转录工具。

## 环境配置

- **Python**: `D:/spleeter_env/Scripts/python.exe` (推荐) 或 `D:/ACE-Step-1.5/python_embeded/python.exe`
- **虚拟环境**: `D:/spleeter_env/`
- **MuseScore**: `C:/Program Files/MuseScore 3/bin/MuseScore3.exe`

## 方法1: midi-extractor (推荐)

支持三种模式: mono(单旋律), poly(和弦), drums(鼓点)。

### 安装

```bash
D:/spleeter_env/Scripts/pip.exe install crepe pretty_midi pydub
```

### 使用

```bash
python convert.py input.wav output.mid --mode mono --bpm 90
```

| 参数 | 说明 |
|------|------|
| `--mode mono` | 单旋律 (人声/独奏) |
| `--mode poly` | 多声部 (和弦) |
| `--mode drums` | 鼓点 |
| `--bpm` | BPM (可选) |

## 方法2: librosa 内置

适合简单场景，直接提取旋律。

### 旋律提取脚本

```python
import librosa
import numpy as np

# 加载音频
y, sr = librosa.load('input.wav', sr=44100)

# 方法1: pyin (推荐)
f0, voiced, probs = librosa.pyin(
    y,
    fmin=librosa.note_to_hz('C3'),
    fmax=librosa.note_to_hz('C6'),
    sr=sr,
    hop_length=512
)

# 方法2: yin
f0 = librosa.yin(y, fmin=librosa.note_to_hz('C3'), fmax=librosa.note_to_hz('C6'), sr=sr)

# 转换为音符
melody_midi = librosa.hz_to_midi(f0)
melody_midi = np.nan_to_num(melody_midi, 0)
```

## 方法3: CREPE (高精度)

需要安装 crepe 库。

```bash
D:/spleeter_env/Scripts/pip.exe install crepe
```

```python
import crepe
import librosa

audio, sr = librosa.load('input.wav', sr=16000)
frequencies, activations, periodicity, pitch = crepe.predict(audio)
# 转换为MIDI
midi_notes = 12 * np.log2(frequencies / 440) + 69
```

## 转换为简谱

### 音符频率表

```python
NOTE_FREQS = {
    'C3': 130.81, 'C#3': 138.59, 'D3': 146.83,
    'C4': 261.63, 'D4': 293.66, 'E4': 329.63,
    'F4': 349.23, 'G4': 392.00, 'A4': 440.00, 'B4': 493.88,
    'C5': 523.25, 'D5': 587.33, 'E5': 659.25,
}

def find_nearest_note(freq):
    if freq < 100 or freq > 1200:
        return None
    nearest = min(NOTE_FREQS.items(), key=lambda x: abs(x[1] - freq))
    if abs(nearest[1] - freq) / nearest[1] < 0.05:
        return nearest[0]
    return None
```

### 简化版脚本位置

- `D:/spleeter_env/vocal_to_midi.py` - 从人声生成简谱+MIDI+PDF
- `D:/spleeter_env/accomp_to_midi.py` - 从伴奏生成简谱+MIDI+PDF

## 转换为五线谱PDF

需要MuseScore配合music21。

### 脚本

```python
from music21 import stream, note, tempo, meter, clef
import subprocess

# 创建乐谱
score = stream.Score()
part = stream.Part()
part.append(tempo.MetronomeMark(number=95))
part.append(meter.TimeSignature('4/4'))
part.append(clef.TrebleClef())

# 添加音符
for pitch, duration in notes:
    n = note.Note(pitch)
    n.quarterLength = duration
    part.append(n)

score.insert(0, part)

# 导出MusicXML
score.write('musicxml', fp='output.xml')

# 转换为PDF
subprocess.run(['C:/Program Files/MuseScore 3/bin/MuseScore3.exe', '-o', 'output.pdf', 'output.xml'])
```

## 完整工作流

### 1. Demucs分离人声

```bash
D:/spleeter_env/Scripts/demucs-infer.exe -n htdemucs -o OUTPUT --two-stems vocals --shifts 1 input.wav
```

### 2. 提取旋律生成MIDI

```bash
python convert.py vocals.wav output.mid --mode mono --bpm 95
```

### 3. 生成简谱txt

```python
# 使用 vocal_to_midi.py 脚本
D:/spleeter_env/Scripts/python.exe D:/spleeter_env/vocal_to_midi.py
```

### 4. 生成五线谱PDF

```python
# 使用 generate_score_pdf.py 脚本
D:/spleeter_env/Scripts/python.exe D:/spleeter_env/generate_score_pdf.py
```

## 工具对比

| 方法 | 精度 | 适用场景 | 备注 |
|------|------|----------|------|
| librosa.pyin | 中 | 人声/独奏 | 内置，无需额外安装 |
| CREPE | 高 | 人声 | 需要GPU |
| Melodia | 高 | 复杂音乐 | 需要Vamp插件 |
| midi-extractor mono | 高 | 旋律 | 需安装依赖 |
| midi-extractor poly | 中 | 和弦 | 包含多音符 |
| hf_midi_transcription | 高 | 特定乐器 | 支持 Sax/Bass/Guitar/Piano |

## 输出命名规范

| 类型 | 格式 |
|------|------|
| 简谱txt | `简谱_人声.txt` |
| 五线谱pdf | `五线谱_人声.pdf` |
| MIDI | `歌曲名_人声.mid` |
| 分离人声 | `歌曲名_分离人声.wav` |
| 分离伴奏 | `歌曲名_分离伴奏.wav` |