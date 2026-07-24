# 在 Dell Pro Max GB10 / DGX Spark 上安裝 IndexTTS2 完整紀錄

## 硬體與系統資訊

- 硬體：Dell Pro Max with GB10（與 NVIDIA DGX Spark 同款硬體）
- 晶片：NVIDIA GB10 Grace Blackwell Superchip
  - CPU：ARM64 (aarch64)，20 核心（10x Cortex-X925 + 10x Cortex-A725）
  - GPU：Blackwell 架構，運算能力 sm_121，6,144 CUDA 核心
  - 記憶體：128GB LPDDR5x 統一記憶體
- 作業系統：NVIDIA DGX OS（基於 Ubuntu 24.04，aarch64）
- CUDA Toolkit：13.0（系統內建）

這台機器跟一般 x86_64 的 WSL/Linux 環境**架構完全不同**，需要 ARM64 專用的套件與 PyTorch build，且 Blackwell 是全新架構，許多套件的預編譯二進位檔還沒跟上，導致安裝過程需要多處手動處理。

---

## 完整安裝流程

### 1. 基礎工具準備

```bash
# git-lfs（用來抓 repo 裡的大型檔案）
sudo apt update
sudo apt install -y git-lfs
git lfs install

# clone 專案
git clone https://github.com/index-tts/index-tts.git
cd index-tts
git lfs pull
```

### 2. 安裝 uv（後來改用 conda，見下方說明）

一開始嘗試用 `uv` 建立虛擬環境，但因為 `pynini`（文字正規化用的相依套件）在 ARM64 上編譯會失敗（見下方問題 1），最後改用 **conda（Miniforge）** 作為主要環境管理工具。

### 3. 安裝 Miniforge（支援 aarch64 的 conda 發行版）

```bash
cd /tmp
wget https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-aarch64.sh
bash Miniforge3-Linux-aarch64.sh -b -p $HOME/miniforge3
source $HOME/miniforge3/bin/activate

# 讓每次開新終端機都能直接用 conda 指令
$HOME/miniforge3/bin/conda init bash
source ~/.bashrc
```

### 4. 建立專用環境並安裝 pynini（解決問題 1）

```bash
conda create -n indextts python=3.11 -y
conda activate indextts

# 用 conda-forge 裝相容版本的 pynini（自動配對正確版本的 OpenFst 預編譯檔）
conda install -c conda-forge pynini=2.1.6 -y

# WeTextProcessing 裝的時候要跳過它想重裝 pynini
pip install WeTextProcessing --no-deps
```

### 5. 安裝相容 GB10 的 PyTorch（cu130）

```bash
cd ~/index-tts
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu130

# 驗證
python3 -c "import torch; print(torch.__version__, torch.cuda.is_available(), torch.cuda.get_device_name(0))"
# 預期輸出：2.13.0+cu130 True NVIDIA GB10
```

### 6. 安裝 index-tts 本體與其餘依賴

```bash
# 裝本體，--no-deps 避免它想重裝/降版 torch
pip install -e . --no-deps

# 裝其餘依賴（排除 torch 系列與 pynini/WeTextProcessing，皆已裝好）
pip install gradio==5.45.0 accelerate==1.8.1 cn2an==0.5.22 cython==3.0.7 \
  descript-audiotools==0.7.2 einops ffmpeg-python==0.2.0 g2p-en==2.1.0 \
  jieba==0.42.1 json5==0.10.0 keras==2.9.0 librosa==0.10.2.post1 \
  matplotlib==3.10.0 modelscope==1.27.0 munch==4.0.0 numba==0.63.0 \
  numpy==2.2.6 omegaconf opencv-python==4.9.0.80 pandas==2.3.2 \
  safetensors==0.5.2 sentencepiece tensorboard==2.20.0 textstat \
  tokenizers==0.21.0 transformers==4.52.1
```

### 7. 下載模型權重並啟動

```bash
export HF_HUB_DISABLE_XET=1
export HF_HUB_DOWNLOAD_TIMEOUT=120

python webui.py
```

`webui.py` 第一次執行時會自動偵測 `checkpoints/` 資料夾缺哪些檔案，並自動從 Hugging Face 下載（IndexTTS-2 主模型 + w2v-bert-2.0 + MaskGCT semantic codec + CAMPPlus + BigVGAN，總計約 8GB）。

---

## 遇到的問題與解決方式

### 問題 1：`pynini` 在 ARM64 上編譯失敗

**現象：** `uv sync` / `pip install WeTextProcessing` 時，`pynini==2.1.7` 編譯失敗，錯誤訊息顯示 `fst/util.h: No such file or directory`，即使手動編譯 OpenFst 1.8.2 後再試，仍出現大量 C++ 編譯錯誤（`Properties`、`GetSeed`、`kDefaultSeed`、`CompileInternal` 等符號找不到）。

**原因：** pynini 的 Cython 產生的 C++ 程式碼跟 OpenFst 版本綁得很緊，pip 預設抓到的 `pynini 2.1.7` 需要跟它精確配對的 OpenFst 版本，手動編譯的 1.8.2 版本 API 不相容。

**解決方式：** 改用 **conda-forge** 安裝 `pynini==2.1.6`，它會自動配對好正確版本的預編譯 OpenFst，不需要自己編譯：
```bash
conda install -c conda-forge pynini=2.1.6 -y
pip install WeTextProcessing --no-deps
```

### 問題 2：PyTorch 版本與 CUDA 架構不相容

**現象：** 官方 `pyproject.toml` 寫死 `torch==2.8.*` 搭配 cu128 索引，這是給 x86_64 用的，且 cu128 不支援 Blackwell(sm_121) 需要的 CUDA 13.0。

**解決方式：** 不使用 `uv sync`／官方預設安裝方式，改為手動安裝相容 GB10 的版本：
```bash
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu130
```

### 問題 3：`accel` extra 裡的 `triton-windows` 在 Linux 上必壞

**現象：** 官方 `pyproject.toml` 的 `accel` extra 裡寫死了 `triton-windows==3.1.0.post17`，沒有平台判斷式，導致在任何 Linux 環境（含 WSL 和 GB10）安裝時都會失敗。

**解決方式：** 安裝時不要加 `--extra accel`，直接跳過。

### 問題 4：下載模型時網路頻繁斷線

**現象：** 下載 Hugging Face 上的模型檔案（`gpt.pth`、`s2mel.pth`、`conformer_shaw.pt` 等大型檔案）時經常出現 `LocalEntryNotFoundError`、`RemoteDisconnected` 等網路錯誤。

**解決方式：**
```bash
export HF_HUB_DISABLE_XET=1
export HF_HUB_DOWNLOAD_TIMEOUT=120
huggingface-cli download IndexTeam/IndexTTS-2 --local-dir=checkpoints --max-workers=1
```
直接重跑指令即可從中斷點接續下載，通常重試幾次就會全部到齊。

### 問題 5：生成的語音充滿嚴重雜訊（本次除錯重點）

**現象：** 模型能成功載入、推論流程跑得完全正常（GPT 生成、s2mel、BigVGAN 皆無錯誤），但輸出的 `.wav` 檔案播放起來充滿嚴重雜訊，幾乎無法辨識人聲。這個問題與社群回報的 [GitHub Issue #496](https://github.com/index-tts/index-tts/issues/496)（Nvidia DGX-Spark 部署生成的語音充滿雜音，目前尚無官方或社群解答）描述的症狀一致。

**排查過程：**
1. 用程式分析輸出的 wav 檔案數值，發現 **99.8% 以上的取樣點都精準卡在 int16 的最大/最小值**（嚴重削波），代表寫入檔案的數值幾乎全部溢位。
2. 依序排除以下假設，逐一測試皆未改善：
   - 關閉 cuDNN 與 TF32（排除 GPU 精度問題）
   - 完全跑在純 CPU（排除 GPU 架構特有問題；CPU 上甚至直接因為 ARM JIT bug 而崩潰報錯 `Xbyak::Error`）
   - 關閉 CPU 的 mkldnn 後端（仍然一樣削波）
   - 完全重新下載一份乾淨的模型權重（排除下載過程檔案損毀）
3. 在原始碼裡插入診斷用的 `print`，分別印出：
   - **BigVGAN 輸出、乘以 32767 之前**的數值範圍 → 結果完全正常（`min -1.0, max 1.0, mean_abs 0.11~0.13`）
   - 但**存檔後**的實際 wav 檔案分析卻仍然是 99%+ 削波

**真正的根本原因：** 模型本身生成的音訊資料從頭到尾都是正常、健康的訊號，問題出在**存檔這一步** —— `torchaudio.save()` 在這台機器的 PyTorch 2.13.0（開發版）+ ARM64 組合下，儲存邏輯本身有 bug（原本應該退回的 `torchcodec` 後端在此環境下也不可用），把原本正常的訊號寫壞成幾乎全滿格的雜訊檔案。

**解決方式：** 繞過壞掉的 `torchaudio.save`，改用 `soundfile` 手動存檔。修改 `indextts/infer_v2.py` 中負責存檔的那一行：

```python
# 原本（會產生雜訊）：
torchaudio.save(output_path, wav.type(torch.int16), sampling_rate)

# 改成：
import soundfile as _sf
_sf.write(output_path, (wav.type(torch.int16).squeeze(0)).cpu().numpy().astype("int16"),
          sampling_rate, subtype="PCM_16")
```

修改後重新測試，輸出檔案完全乾淨，0% 削波，音質正常。因為 `webui.py` 呼叫的是同一個 `infer_v2.py` 裡的推論函式，這個修正對 WebUI 介面生成的音訊同樣有效，不需要額外修改 `webui.py`。

---

## 附註：其他順帶發現的小問題

- 直接呼叫 `IndexTTS2` 類別時，若環境缺少 `ninja`，會自動 fallback 掉「BigVGAN 自訂 CUDA 加速核心」（`use_cuda_kernel`），退回純 PyTorch 實作，這其實意外地讓輸出更穩定（純 PyTorch 路徑在 GB10 上是可信賴的，自訂 CUDA kernel 在 sm_121 上未必已驗證過）。
- 儲存音訊時 `torchaudio` 提示需要 `torchcodec` 套件；但如上所述，最終方案是完全改用 `soundfile`，不需要再處理 `torchcodec` 的安裝。
