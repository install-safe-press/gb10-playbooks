# GB10 / DGX Spark 安裝 IndexTTS2 標準流程（已避開已知問題）

適用於：Dell Pro Max with GB10、NVIDIA DGX Spark，或其他 GB10 (Grace Blackwell, aarch64, sm_121) 機器。

本流程已根據前一台機器的除錯經驗調整，跳過所有已知的失敗路徑，照著做應該能一次到位。

---

## 前置確認

```bash
nvidia-smi
nvcc --version   # 應顯示 CUDA 13.x
```

---

## 步驟 1：安裝 git-lfs 並 clone 專案

```bash
sudo apt update
sudo apt install -y git-lfs
git lfs install

git clone https://github.com/index-tts/index-tts.git
cd index-tts
git lfs pull
```

---

## 步驟 2：安裝 Miniforge（直接用 conda，不要用 uv）

> ⚠️ 不要用 `uv sync` 或官方 README 建議的 uv 安裝方式——`pynini` 在 ARM64 上用 pip/uv 會編譯失敗，`torch` 版本也會抓錯（官方鎖定 x86_64 專用的 cu128）。直接跳過，全部用 conda 處理。

```bash
cd /tmp
wget https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-aarch64.sh
bash Miniforge3-Linux-aarch64.sh -b -p $HOME/miniforge3
$HOME/miniforge3/bin/conda init bash
source ~/.bashrc

conda create -n indextts python=3.11 -y
conda activate indextts
```

---

## 步驟 3：用 conda-forge 裝 pynini（跳過編譯地獄）

```bash
conda install -c conda-forge pynini=2.1.6 -y
pip install WeTextProcessing --no-deps
```

---

## 步驟 4：裝相容 GB10 的 PyTorch（cu130，不是官方預設的 cu128）

```bash
cd ~/index-tts
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu130

# 驗證，務必看到 True 跟 NVIDIA GB10
python3 -c "import torch; print(torch.__version__, torch.cuda.is_available(), torch.cuda.get_device_name(0))"
```

---

## 步驟 5：裝 index-tts 本體與其餘依賴（跳過會失敗的 extras）

> ⚠️ 不要用 `--extra accel`（會拉入 Windows 專用的 `triton-windows`，在 Linux 上必壞）。`deepspeed` 也先不裝（在 ARM 上編譯問題多，非必需）。

```bash
pip install -e . --no-deps

pip install gradio==5.45.0 accelerate==1.8.1 cn2an==0.5.22 cython==3.0.7 \
  descript-audiotools==0.7.2 einops ffmpeg-python==0.2.0 g2p-en==2.1.0 \
  jieba==0.42.1 json5==0.10.0 keras==2.9.0 librosa==0.10.2.post1 \
  matplotlib==3.10.0 modelscope==1.27.0 munch==4.0.0 numba==0.63.0 \
  numpy==2.2.6 omegaconf opencv-python==4.9.0.80 pandas==2.3.2 \
  safetensors==0.5.2 sentencepiece tensorboard==2.20.0 textstat \
  tokenizers==0.21.0 transformers==4.52.1
```

---

## 步驟 6：修正 `torchaudio.save` 在 ARM64 上會壞掉的 bug（關鍵！）

> ⚠️ 這是最容易忽略、但影響最大的一步。`torchaudio.save()` 在 PyTorch 2.13(開發版) + ARM64 這個組合下存檔會壞掉，模型本身生成的音訊其實是正常的，但存檔會把資料寫成幾乎全滿格的雜訊。**務必在下載模型、跑第一次推論之前就先做這個修正**，才不會被雜訊誤導以為是模型或硬體問題。

```bash
pip install soundfile

python3 << 'EOF'
path = "indextts/infer_v2.py"
with open(path, "r") as f:
    lines = f.readlines()

def indent_of(line):
    return line[:len(line) - len(line.lstrip())]

target_idx = None
for i, line in enumerate(lines):
    if "torchaudio.save(output_path" in line:
        target_idx = i
        break

if target_idx is None:
    print("找不到 torchaudio.save 那一行，可能官方原始碼已經改了，請手動檢查 infer_v2.py")
else:
    indent = indent_of(lines[target_idx])
    lines[target_idx] = (
        f'{indent}import soundfile as _sf\n'
        f'{indent}_sf.write(output_path, (wav.type(torch.int16).squeeze(0)).cpu().numpy().astype("int16"), '
        f'sampling_rate, subtype="PCM_16")\n'
    )
    with open(path, "w") as f:
        f.writelines(lines)
    print(f"Patched line {target_idx + 1} successfully")
EOF
```

跑完確認一下有正確替換：
```bash
grep -n "soundfile\|torchaudio.save" indextts/infer_v2.py
```

應該只看到 `soundfile` 那兩行，`torchaudio.save` 不應該再出現。

---

## 步驟 7：下載模型並啟動

```bash
export HF_HUB_DISABLE_XET=1
export HF_HUB_DOWNLOAD_TIMEOUT=120

python webui.py
```

第一次執行會自動下載 `checkpoints/`（IndexTTS-2 主模型 + w2v-bert-2.0 + MaskGCT semantic codec + CAMPPlus + BigVGAN，共約 8GB）。若網路不穩導致中斷，直接重跑 `python webui.py` 即可從中斷點接續，通常重試 2-3 次會全部到齊。

啟動成功後會看到：
```
Running on local URL: http://0.0.0.0:7860
```

用瀏覽器打開該網址（或如果是遠端機器，改用 SSH port forward 或直接用區網 IP）即可使用。

---

## 快速檢查清單

- [ ] 全程用 `conda activate indextts`，不要跟 uv/venv 混用
- [ ] pynini 用 `conda install -c conda-forge pynini=2.1.6`，不要用 pip/uv 裝
- [ ] torch 用 cu130 索引安裝，不要用官方 `uv sync` 預設的 cu128
- [ ] 不要加 `--extra accel`
- [ ] **跑第一次推論之前先做完步驟 6 的 soundfile patch**，避免被雜訊誤導
- [ ] 下載模型前先 `export HF_HUB_DISABLE_XET=1` 和 `HF_HUB_DOWNLOAD_TIMEOUT=120`
