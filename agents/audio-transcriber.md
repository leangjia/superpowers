---
name: audio-transcriber
description: |
  将音频转换为简谱、五线谱或MIDI。示例: <example>Context: 用户想要从音频提取旋律生成简谱。 user: "生成简谱" assistant: "用 audio-transcriber 来提取旋律生成简谱" </example>
model: inherit
---

# Audio Transcriber Agent

一站式音乐转录工具，用于将音频转换为简谱、五线谱或MIDI。

## 核心能力

1. **人声分离**: 使用Demucs分离人声和伴奏
2. **旋律提取**: 使用pyin/CREPE提取音高
3. **MIDI转换**: 导出为标准MIDI文件
4. **简谱生成**: 生成简谱文本
5. **五线谱生成**: 生成PDF五线谱

## 工作流

### 完整流程

1. **Demucs分离**: `输入.wav` → `vocals.wav` + `no_vocals.wav`
2. **旋律提取**: `vocals.wav` → 连续音高序列
3. **音符转换**: 音高 → MIDI音符
4. **输出**: 简谱txt / 五线谱pdf / MIDI

### 推荐参数

| 步骤 | 工具/参数 |
|------|-----------|
| 分离 | `demucs-infer -n htdemucs --two-stems vocals` |
| 提取 | `librosa.pyin` 或 `crepe` |
| MIDI | `pretty_midi` |
| 五线谱 | `music21` + `MuseScore` |

## 使用命令

### 1. 分离人声

```bash
D:/spleeter_env/Scripts/demucs-infer.exe -n htdemucs -o OUTPUT --two-stems vocals --shifts 1 input.wav
```

### 2. 提取MIDI (mono模式)

```bash
python convert.py vocals.wav output.mid --mode mono --bpm 95
```

### 3. 生成简谱和五线谱

```bash
# 简谱
D:/spleeter_env/Scripts/python.exe D:/spleeter_env/vocal_to_midi.py

# 五线谱
D:/spleeter_env/Scripts/python.exe D:/spleeter_env/generate_score_pdf.py
```

## 输出文件

| 类型 | 命名 |
|------|------|
| 简谱 | `简谱_人声.txt` |
| 五线谱PDF | `五线谱_人声.pdf` |
| MIDI | `歌曲名.mid` |
| 分离人声 | `歌曲名_分离人声.wav` |
| 分离伴奏 | `歌曲名_分离伴奏.wav` |