---
name: sd-webui-forge-deployment
description: Use when deploying Stable Diffusion WebUI Forge on Windows with NVIDIA GPU, setting up AI绘画环境, or configuring low VRAM optimization for 4G-6G graphics cards
---

# Skill: SD WebUI Forge 部署

## Overview
Stable Diffusion WebUI Forge 是 lllyasviel 开发的 SD WebUI 增强版，优化显存占用并提升推理速度，支持 4G 显存流畅运行 AI 绘画。

## When to Use
- 在 Windows + NVIDIA GPU 环境部署 AI 绘画工具
- 显存 ≤ 6G 需要优化参数配置
- 需要运行 FLUX / SDXL / SD 1.5 模型
- 新手首次安装 Forge（一键包方式）

## 硬件要求
| 配置 | 显卡示例 | 显存 |
|------|---------|------|
| 入门 | GTX 1060 6G / RTX 2060 | 6G |
| 推荐 | RTX 3060 12G / RTX 4060 | 8-12G |
| 理想 | RTX 3090 / RTX 4090 | 24G |

本机：RTX 3080 10G（满足要求）

## 部署步骤（一键包方式）

### 1. 下载一键包

**国内用户（GitHub 访问慢/超时）：**
- 镜像站：https://ghproxy.com/https://github.com/lllyasviel/stable-diffusion-webui-forge/releases/latest
- 或设置代理后下载

**直接下载地址（需替换镜像前缀）：**
```
原地址：https://github.com/lllyasviel/stable-diffusion-webui-forge/releases/download/1.0.0/webui_forge_cu121_torch2_win.7z
镜像：https://ghproxy.com/https://github.com/lllyasviel/stable-diffusion-webui-forge/releases/download/1.0.0/webui_forge_cu121_torch2_win.7z
```

**PowerShell 下载（需代理）：**
```powershell
$proxy = "http://127.0.0.1:7890"  # 替换为你的代理端口
[System.Net.WebRequest]::DefaultWebProxy = New-Object System.Net.WebProxy($proxy)
Invoke-WebRequest -Uri "https://github.com/lllyasviel/stable-diffusion-webui-forge/releases/latest" -OutFile "D:\AI\forge_page.html"
```

**国外/直连用户：**
访问 https://github.com/lllyasviel/stable-diffusion-webui-forge
点击 Releases，下载 Windows 一键包（通常是 `webui_forge_cu121_torch2_win.7z` 或类似）

### 2. 解压到非中文路径
```powershell
# 示例路径（不要有中文或空格）
D:\AI\stable-diffusion-webui-forge\
```

### 3. 运行 update.bat（必须！）
双击 `update.bat`，等待显示 `already updated` 后关闭
**跳过此步会导致依赖缺失**

### 4. 启动 Forge
双击 `run.bat`，等待启动完成后访问：
```
http://127.0.0.1:7860
```

## 模型存放路径
```
stable-diffusion-webui-forge/
└── models/
    ├── Stable-diffusion/   # 主模型 (.safetensors)
    ├── VAE/                # VAE 模型
    ├── Lora/               # LoRA 模型
    ├── ControlNet/         # ControlNet 模型
    └── embeddings/         # Embeddings
```

## 模型下载站
| 站点 | 地址 |
|------|------|
| Civitai | https://civitai.com |
| HuggingFace | https://huggingface.co |
| LiblibAI | https://www.liblib.ai |

## 显存优化参数
编辑 `webui-user.bat`，修改 `COMMANDLINE_ARGS`：

```batch
set COMMANDLINE_ARGS=--xformers --opt-sdp-attention --medvram
```

| 参数 | 作用 | 适用显存 |
|------|------|---------|
| `--xformers` | xformers 加速 | 所有 |
| `--opt-sdp-attention` | SDP 注意力优化 | 所有 |
| `--medvram` | 中等显存模式 | 6-8G |
| `--lowvram` | 低显存模式 | 2-4G |
| `--no-half` | 禁用半精度（部分显卡修复） | 特殊 |

## 生成速度优化
- 启用 TensorRT（RTX 30/40 系列）
- 使用 DPM++ 2M Karras 采样器
- 采样步数 20-30 步即可

## 常见错误

| 问题 | 解决方案 |
|------|---------|
| 启动闪退 | 运行 update.bat 更新依赖 |
| 显存不足 OOM | 加 `--lowvram`，降低分辨率到 512x512 |
| 无法访问 GitHub | 配置代理或科学上网 |
| 中文路径报错 | 迁移到纯英文路径 |

## RED Flags - 不要这样做
- 跳过 update.bat 直接启动
- 路径包含中文或空格
- 6G 显存未加 `--medvram` 运行 SDXL
- 未下载模型就尝试生成图片
