# Skill: jianpu-generator

# 简谱生成器 (基于ScoreTextMaker + MIDI)

简谱生成器有两种方法：FFT频谱分析和MIDI数据提取。

## 方法对比

| 方法 | 精度 | 适用场景 |
|------|------|----------|
| FFT频谱分析 | 中 | 无MIDI源的wav音频 |
| MIDI数据提取 | 高 | 有ACE-Step生成的MIDI |

### 1. FFT频谱分析 (ScoreTextMaker方法)
- **核心文件**: `D:/opencodeproject/ScoreTextMaker/jianpu_generator.py`
- **原理**: 将WAV音频按500ms切割→FFT→频谱特征→匹配钢琴88音频率
- **问题**: 噪音干扰大，识别准确率低

### 2. MIDI数据提取 (推荐)
- **来源**: ACE-Step生成的MIDI文件
- **优势**: 精确的音符数据，无噪音干扰
- **转换**: 使用convert_jianpu.py将MIDI转换为简谱数据

## 环境配置

- **Python**: `D:/spleeter_env/Scripts/python.exe`
- **频率字典**: `D:/opencodeproject/ScoreTextMaker/Fcsv_final.csv`

## 简谱格式说明

简谱数字对应音高:
- `1` = do (C)
- `2` = re (D)  
- `3` = mi (E)
- `4` = fa (F)
- `5` = sol (G)
- `6` = la (A)
- `7` = si (B)

八度标记:
- `1(-1)` = 低音do (C3)
- `1` = 中央do (C4)
- `1(+1)` = 高音do (C5)

## 使用方法

### 命令行 (FFT方法)

```bash
python D:/opencodeproject/ScoreTextMaker/jianpu_generator.py <wav文件> [输出文件]
```

示例:
```bash
python D:/opencodeproject/ScoreTextMaker/jianpu_generator.py "D:/泥娃娃，泥巴巴/泥娃娃，泥巴巴_童声_人声.wav" "D:/泥娃娃，泥巴巴/jianpu.txt"
```

### Python API

```python
import sys
sys.path.insert(0, 'D:/opencodeproject/ScoreTextMaker')
from jianpu_generator import generate_jianpu, generate_jianpu_json

# 生成文本简谱
generate_jianpu("input.wav", "output.txt")

# 生成JSON简谱
generate_jianpu_json("input.wav", "output.json")
```

## 输出格式

- **TXT**: 简谱文本文件 (每500ms一个片段)
- **JSON**: 简谱JSON数据 (含频率信息)
- **HTML**: 带播放功能的简谱页面 (使用generate_jianpu_html.py)
- **PDF**: PDF简谱 (使用reportlab)

## 核心算法

```python
# 1. 静音过滤
audio_power = np.mean(wave_data[0]**2)
if audio_power < 1000:
    return ["休止"]

# 2. FFT分析
N = framerate
wave_data2 = wave_data[0][start:start+N]
original_array = np.fft.fft(wave_data2) * 2 / N
c = MMNormalization(original_array)  # 归一化

# 3. 找频率峰值
f_arr, a_arr = FindFrequency(c)

# 4. 分类匹配
f_index_arr = Classify(f_arr, FDic)

# 5. 转换为简谱
jianpu = KeyList[f_index_arr[j]]
```

## 注意事项

- 音频采样率应为44100Hz
- 建议先使用Demucs分离人声再生成简谱
- 噪音过多的音频识别可能不准确
- 推荐使用MIDI数据提取方法，精度更高