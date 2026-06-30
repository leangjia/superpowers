---
name: rvc-webui-training
description: Use when training voice conversion models with RVC-WebUI, especially on Windows with RTX 3080+ GPUs. Handles fairseq compilation failures, omegaconf metadata issues, ffmpeg PATH errors, USE_LIBUV distributed training crashes, and matplotlib API deprecations.
---

# RVC-WebUI Voice Model Training

## Overview
RVC-WebUI (Retrieval-based Voice Conversion) trains voice models using HuBERT + RMVPE + HiFi-GAN. On Windows with PyTorch 2.5+, several environment issues block training. This skill documents the fixes.

## Baseline Failures (from real session)
| Symptom | Root Cause | Fix |
|---------|-----------|-----|
| `ModuleNotFoundError: fairseq` on `infer-web.py` start | fairseq C extensions fail to compile on Windows (libbleu needs MSVC) | Patch setup.py: remove all C extensions, install from source with pip 24.0 |
| `omegaconf` dependency resolution fails | omegaconf 2.0.x uses invalid PyYAML metadata format (>=5.1.*) rejected by pip>=24.1 | Downgrade pip to 24.0 |
| `ModuleNotFoundError: pkg_resources` | setuptools>=76 removed pkg_resources | Downgrade setuptools to <72 |
| `echo` garbled / batch file crashes | `|` pipe character in `echo GPU: RTX 3080 | CUDA: OK` interpreted as CMD pipe | Escape with `^|` or remove pipes |
| `FileNotFoundError: ffmpeg` | ffmpeg not in system PATH | `pip install imageio-ffmpeg` bundles ffmpeg.exe; copy to project root + add to PATH |
| `RuntimeError: use_libuv was requested` | PyTorch 2.5 Windows build lacks libuv; `USE_LIBUV=0` env var needed | Add `os.environ["USE_LIBUV"] = "0"` before any torch import in train.py |
| `AttributeError: 'FigureCanvasAgg' has no attribute 'tostring_rgb'` | matplotlib >=3.8 removed tostring_rgb() | Replace with `np.asarray(fig.canvas.buffer_rgba())[:, :, :3]` |

## Training Workflow

### 1. Environment Setup
```bash
# Python 3.10, venv at D:\RVC-WebUI\venv
pip install pip==24.0   # must be <24.1 for omegaconf
pip install setuptools<72  # preserve pkg_resources
```

### 2. Install Dependencies
```bash
pip install torch==2.5.1+cu121 torchaudio==2.5.1+cu121 --index-url https://download.pytorch.org/whl/cu121
# Install everything else
pip install gradio==3.34.0 fastapi==0.88 pydantic==1.10.26
pip install librosa==0.9.1 faiss-cpu==1.7.3 numpy==1.26.4
# ffmpeg bundle
pip install imageio-ffmpeg
```

### 3. fairseq Patch (CRITICAL - will fail without this)
```python
# Download fairseq==0.12.2, extract, modify setup.py:
# Delete ALL extensions (lines 58-145), remove omegaconf from install_requires
extensions = []
cmdclass = {}
# Install from patched source
pip install /path/to/patched/fairseq --no-build-isolation
```

### 4. Download Models
- hubert_base.pt → `assets/hubert/hubert_base.pt`
- rmvpe.pt → `assets/rmvpe/rmvpe.pt`
- pretrained_v2/f0G40k.pth + f0D40k.pth
- Use hf-mirror.com if huggingface.co blocked

### 5. Preprocess + Extract
```bash
# Preprocess (split audio into slices)
python infer/modules/train/preprocess.py <DATA_DIR> 40000 <N_CPU> <EXP_DIR> False 3.7
# Extract f0
python infer/modules/train/extract/extract_f0_rmvpe.py 2 0 0 <EXP_DIR> True
# Extract HuBERT features
python infer/modules/train/extract_feature_print.py cuda:0 1 0 0 <EXP_DIR> v2 True
```

### 6. Train (with libuv fix)
**Before training, patch train.py:**
Add `os.environ["USE_LIBUV"] = "0"` as the FIRST line after `import os` (before any torch import).

If you add it later in the `run()` function, it won't work because multiprocessing child processes on Windows don't inherit the env var correctly.

```bash
python infer/modules/train/train.py -e "model-name" -sr 40k -f0 1 -bs 5 -g 0 -te 20 -se 5 \
  -pg assets/pretrained_v2/f0G40k.pth \
  -pd assets/pretrained_v2/f0D40k.pth \
  -l 0 -c 0 -sw 0 -v v2
```

### 7. Generate Index
```python
# Run after training to build Faiss index
big_npy = np.concatenate([np.load(f) for f in sorted(listdir)], 0)
n_ivf = min(int(16 * np.sqrt(big_npy.shape[0])), big_npy.shape[0] // 39)
index = faiss.index_factory(768, "IVF%s,Flat" % n_ivf)
index.train(big_npy); index.add(big_npy)
faiss.write_index(index, "logs/<EXP>/added_IVF%s_Flat_nprobe_1_<EXP>_v2.index" % n_ivf)
```

## Matplotlib Fix
If training crashes at epoch 1 with `tostring_rgb` error, patch `infer/lib/train/utils.py`:
```python
# Replace:
data = np.fromstring(fig.canvas.tostring_rgb(), dtype=np.uint8, sep="")
data = data.reshape(fig.canvas.get_width_height()[::-1] + (3,))
# With:
data = np.asarray(fig.canvas.buffer_rgba())[:, :, :3]
```

## Batch File Gotcha
```bat
REM WRONG - | is pipe operator:
echo GPU: RTX 3080 | CUDA: OK

REM RIGHT - escape with ^| or avoid pipes:
echo GPU: RTX 3080 ^| CUDA: OK
```

## Common Mistakes
| Mistake | Fix |
|---------|-----|
| Pointing training data to .wav file | Must be a DIRECTORY containing .wav files |
| Forgetting `set PATH=D:\RVC-WebUI;%PATH%` in batch file | ffmpeg won't be found |
| Setting `TORCH_DISTRIBUTED_USE_LIBUV` | Wrong name! Must be `USE_LIBUV` |
| Using pip >= 24.1 | omegaconf metadata validation fails. Use pip 24.0 |
| setuptools >= 76 | pkg_resources removed. Use <72 |
