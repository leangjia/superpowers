---
name: music-arrangement
description: 使用音乐配器思维 - 根据情绪选择合适的乐器和技法来生成AI音乐
---

# 音乐配器思维

根据情绪选择正确的乐器和提示词来生成AI音乐。

## 情绪-乐器-技法映射表

### 1. 悲伤、孤独、失落
- **核心乐器**: 钢琴、大提琴、小提琴独奏
- **关键词**: `Sparse arrangement`, `Minor key`, `Solo piano`, `Dark reverb`, `Emotional`
- **避坑**: 避免铜管和太多打击乐

### 2. 燃、激昂、史诗感
- **核心乐器**: 铜管组、定音鼓、合成器锯齿波、电吉他失真
- **关键词**: `Epic orchestral`, `Powerful brass staccato`, `Rising tension`, `Dense arrangement`, `Full drums`
- **技巧**: 副歌前留 `Build-up` 段落

### 3. 温馨、治愈、希望
- **核心乐器**: 木吉他、钢片琴、长笛
- **关键词**: `Warm acoustic`, `Gentle strumming`, `Bright major key`, `Soft reverb`, `Heartwarming`
- **点缀**: 加入 `Music Box` 增加童真感

### 4. 紧张、悬疑、科技感
- **核心乐器**: 合成器Pad、电子琶音、低音提琴拨奏
- **关键词**: `Tension`, `Dissonant harmony`, `Filter sweep`, `Lo-fi texture`, `Minimalist repetitive`
- **秘技**: `Uneasy silence filled with low-end rumble`

### 5. 梦幻、空灵、迷幻
- **核心乐器**: 合成器Pad、八音盒、长笛
- **关键词**: `Ambient`, `Ethereal`, `Washed out reverb`, `Slow attack pad`

### 6. 快乐、活泼、俏皮
- **核心乐器**: 尤克里里、钢片琴、木箱鼓
- **关键词**: `Upbeat`, `Plucky`, `Major key`, `Sparse percussion`, `Glockenspiel`

### 7. 史诗、宏大、叙事
- **核心乐器**: 完整管弦乐、定音鼓
- **关键词**: `Cinematic orchestral`, `Sweeping strings`, `Heroic brass`, `Timpani rolls`

### 8. 暧昧、性感、深夜
- **核心乐器**: 萨克斯、电钢琴、爵士鼓刷
- **关键词**: `Smooth jazz`, `Sensual`, `Mellow saxophone`, `Brush drums`, `7th chords`

### 9. 诡异、荒诞、不稳定
- **核心乐器**: 不协和钢琴、水琴、随机电子噪音
- **关键词**: `Atonal`, `Glitchy`, `Unsettling`, `Dissonant clusters`, `Random rhythm`

## 万能提示词模板

### 悲伤/孤独
慢速钢琴独奏，只用一架钢琴。稀疏单音旋律和留白，混响开大。高音区孤立的单音。情绪：安静、失落。

### 燃/热血
史诗管弦乐。前15秒定音鼓滚奏+低音弦乐铺垫；15秒后铜管组齐奏主旋律，打击乐全开。密集编排。情绪：英雄登场。

### 温馨/治愈
木吉他扫弦独奏，偶尔加高音区单音旋律。无电子音色、无打击乐。大调，节奏轻快。情绪：放松、温暖。

### 梦幻/空灵
合成器Pad铺底，加八音盒高旋律。大量混响和延迟，节奏模糊或无明确节奏。情绪：像在做梦。

### 紧张/悬疑
低音提琴拨奏+合成器持续低音。偶尔不协和高音弦乐颤音。节奏不规则，大量留白。情绪：暴风雨前的宁静。

### 快乐/活泼
尤克里里或钢片琴为主，加轻快拍手或木箱鼓。大调，节奏跳跃，短促音符。情绪：阳光下的蹦跳。

### 史诗/宏大
完整管弦乐编制，弦乐群铺底+铜管旋律+定音鼓。慢速，起承转合。情绪：电影片头字幕。

## 三步修改法

### 第一步：换核心乐器
- 钢琴→大提琴 = 悲伤变深沉
- 木吉他→八音盒 = 温馨变童话

### 第二步：加细节
- "钢琴" → "音不准的旧钢琴"
- "弦乐" → "只有中提琴"
- "鼓" → "鼓皮调松的底鼓"

### 第三步：删多余
- "不要弦乐群"
- "不要打击乐"
- "不要电子音色"

## 使用场景

当用户要求生成AI音乐时：
1. 确定目标情绪
2. 选择对应的乐器组合
3. 应用关键词模板生成提示词
4. 使用Suno/MiniMax/Tunee等工具生成