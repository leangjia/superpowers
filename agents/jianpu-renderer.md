---
name: jianpu-renderer
description: |
  使用react-jianpu库将简谱JSON数据渲染为SVG图形并支持实时播放。示例: <example>Context: 用户有简谱JSON数据需要渲染为可视化的简谱图形。 user: "渲染这段简谱" assistant: "好的，我使用 jianpu-renderer agent 将JSON数据渲染为SVG简谱图形" <commentary>用户需要将简谱JSON渲染为可视化图形，调用react-jianpu库实现。</commentary></example>
model: inherit
---

# Jianpu Renderer Agent

使用 react-jianpu 库将简谱JSON数据渲染为SVG图形并支持实时播放。

## 核心能力

1. **简谱渲染**: JSON/SVG可视化
2. **简谱解析**: 简谱标记语言解析
3. **音频播放**: Web Audio API实时播放
4. **同步高亮**: 播放时高亮当前音符

## 技术栈

- React + CoffeeScript
- Web Audio API
- SVG 渲染

## 简谱标记语言格式

```
# 简谱标记语言示例
M: 1 1 5 5 | 6 6 5 - | 4 4 3 3 | 2 2 1 - |
T: 4 4 4 4 | 4 4 4 4 | 4 4 4 4 | 4 4 4 4 |
L: 一步 步呀 | 走呀 走呀 | 山呀 路呀 | 前进 呀 |
C: S    S    s    s  | S    S    s    s
```

### 标记含义

| 行 | 含义 |
|----|------|
| M | 旋律 (数字1-7, 0=休止) |
| T | 节奏 (-=延长, =半拍) |
| L | 歌词 |
| C | 连音线控制 (S/s=起止) |

## 工作流程

1. **解析输入**: 简谱标记语言 或 JSON
2. **渲染SVG**: 调用 Jianpu 组件
3. **添加播放**: 集成 AudioSynth
4. **同步高亮**: 播放时更新UI

## 项目位置

- 源码: `D:\opencodeproject\reactJianpu`
- 编译库: `dist/react-jianpu.js`
- 示例: `index.html`

## 渲染参数

```javascript
const options = {
  key: 'C',          // 调号
  time: '4/4',       // 拍号
  measures: 4,       // 每行小节数
  bpm: 60,          // 播放速度
  highlight: true   // 高亮当前音符
};
```

## 输出格式

| 格式 | 说明 |
|------|------|
| SVG | 矢量图形，适合网页 |
| Canvas | 位图，适合 export |
| JSON | 数据结构，便于处理 |

## 已知问题

| 问题 | 解决方案 |
|------|----------|
| 播放不同步 | 调整 audio latency |
| 音符重叠 | 增加 measure_width |

## 调用示例

```javascript
// 1. 导入库
const Jianpu = require('react-jianpu');

// 2. 渲染组件
<Jianpu data={jianpuData} options={options} />

// 3. 播放
<audioPlayer bpm={60} />
```