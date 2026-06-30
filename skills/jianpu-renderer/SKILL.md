---
name: jianpu-renderer
description: Use when rendering numbered music notation (简谱) to SVG graphics from JSON data. Includes pitch rendering, rhythm display, slur handling, and playback synchronization.
---

# Jianpu Renderer Skill

简谱数字记谱法渲染器。将简谱JSON数据渲染为SVG图形。

## 简谱基础

### 音符表示

| 数字 | 音高(简谱) | MIDI |
|------|------------|------|
| 1 | do | 60 |
| 2 | re | 62 |
| 3 | mi | 64 |
| 4 | fa | 65 |
| 5 | sol | 67 |
| 6 | la | 69 |
| 7 | ti | 71 |
| 0 | 休止 | - |

### 拍号与时值

```
duration: 8     # 四分音符
duration: 12    # 三连音
duration: 4     # 八分音符
duration: 16    # 二分音符
duration: 24    # 全音符
```

## JSON 数据结构

```json
{
  "pitch": {
    "base": 60,
    "accidental": 0
  },
  "duration": 12,
  "options": {
    "slur": "start"
  },
  "lyrics": {
    "exists": true,
    "content": "Row, ",
    "hyphen": false
  }
}
```

## 渲染组件

### 核心渲染元素

1. **调号** - 显示1=C等调式
2. **拍号** - 显示4/4等节拍
3. **音符数字** - 1-7的SVG渲染
4. **时值线** - 表示时值长度的横线
5. **附点** - 增加50%时值
6. **八度标记** - 点(上方=高音,下方=低音)
7. **连音线** - 连接音符的弧线
8. **临时升降号** - #/b显示

### SVG 绘制

```
音符位置计算:
x = measure_index * measure_width + note_index * note_width
y = (7 - (pitch % 7)) * line_height + octave_offset
```

## 播放同步

### 高亮当前音符

```javascript
// 当前播放时高亮
currentNote.classList.add('playing');
```

### 播放进度

```javascript
// BPM转换
const beatDuration = 60000 / bpm; // ms per beat
const noteDuration = beatDuration * (duration / 4);
```

## 实际项目参考

react-jianpu 项目位置: `D:\opencodeproject\reactJianpu`

核心文件:
- `dist/react-jianpu.js` - 编译后的渲染库
- `source/index.coffee` - 解析器源码

## 常见问题

| 问题 | 解决方案 |
|------|----------|
| 音符位置重叠 | 调整 measure_width |
| 八度标记错误 | 根据 pitch 判断: >= 65 为高音点 |
| 连音线不显示 | 检查 options.slur 参数 |

## 与五线谱转换

简谱转五线谱需要 music21 库:

```python
from music21 import stream, note, converter

# 简谱JSON转五线谱
s = stream.Stream()
for n in jianpu_notes:
    m21_note = note.Note()
    m21_note.pitch.midi = n['pitch']['base']
    s.append(m21_note)
s.write('pdf', 'output.pdf')
```

## 使用场景

1. 将用户输入的简谱数据渲染为可视化乐谱
2. 在网页中显示简谱并支持实时播放
3. 导出简谱为图片/PDF
4. 与五线谱互相转换