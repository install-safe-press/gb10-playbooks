
步驟 1

存取 DGX 控制面板
選擇以下方法之一存取 DGX 控制面板 Web 介面：

選項 A：桌面捷徑（本機存取）
如果您可以本地存取 DGX Spark 設備：
登入 DGX Spark 裝置上的 Ubuntu 桌面環境
點擊螢幕左下角開啟 Ubuntu 應用程式啟動器
點擊應用啟動器中的 DGX 控制面板快捷方式
控制台將在您的預設 Web 瀏覽器中打開，位址為 http://localhost:11000

接螢幕鍵盤直接操作或遠端桌面


選項 B：NVIDIA Sync（建議用於遠端存取）
如果您已在本機上安裝 NVIDIA Sync：
點選系統托盤中的 NVIDIA Sync 圖標
從設備清單中選擇您的 DGX Spark 設備
點擊“連接”
點選「DGX 控制面板」啟動控制面板
控制面板將透過自動 SSH 隧道在您的預設 Web 瀏覽器中打開，位址為 http://localhost:11000
沒有安裝 NVIDIA Sync？在此安裝 https://build.nvidia.com/spark/connect-to-your-spark/sync

啟動Nvidia Sync
![nv-sync-1](images/nsync-1.jpg)<br>
加入GB10 IP 帳號
![nv-sync-2](images/nsync-2.jpg)<br>

啟動Nvidia Sync可能會遇到的錯誤--檢查是否有重覆執行或改用選項 C：手動下指令建立 SSH 隧道
![nv-sync-3](images/nsync-3.jpg)<br>

選項 C：手動下指令建立 SSH 隧道
如需在不使用 NVIDIA Sync 的情況下手動遠端訪問，您必須先手動設定 SSH 隧道。
您必須為 Dashboard 伺服器（連接埠 11000）和 JupyterLab 開啟一個隧道（如果您想遠端存取 JupyterLab）。每個使用者帳戶都會被指派一個不同的 JupyterLab 連接埠號碼。
```text
ssh -L 11000:localhost:11000 user@GB10-IP
```
把GB10 port 對映到本機PC的port
![jupyter](images/ji-1.jpg)<br>

這個畫面是本機開啟的DGX Dashboard,注意URL是本機PC Windows 的 localhost port 已經透過ssh tunnel到GB10,<br>
接著可以啟動JupterLAB <Start>
![jupyter](images/ji-2.jpg)<br>
按下open in browser
![jupyter](images/ji-3.jpg)<br>
開啟的網頁是空白？JupterLAB WEB port 11002並沒有對映過來PC Windows <br>


![jupyter](images/ji-4.jpg)<br>
開一個新CMD 加一筆對映<br>
```text
ssh -L 11000:localhost:11000 user@GB10-IP
```
刷新網頁之後就會出現 JupterLAB工作區網頁
![jupyter](images/ji-5.jpg)<br>

開一個新的python Notebook<br>
![jupyter](images/jupyter-create.png)<br>


https://build.nvidia.com/spark/dgx-dashboard/instructions Step 4 Test with sample AI workload 

底下這段Python程式貼上<br>
程式說明：此程式會載入 Stable Diffusion XL 模型並自動使用 GPU（CUDA）或 CPU 執行。
根據設定的文字提示（Prompt）生成一張 1024x1024 高品質 AI 圖像。
最後顯示圖片並儲存為 sdxl_output.png。
<br>

```text
import warnings
warnings.filterwarnings('ignore', message='.*cuda capability.*')
import tqdm.auto
tqdm.auto.tqdm = tqdm.std.tqdm

from diffusers import DiffusionPipeline
import torch
from PIL import Image
from datetime import datetime
from IPython.display import display

# --- Model setup ---
MODEL_ID = "stabilityai/stable-diffusion-xl-base-1.0"
dtype = torch.float16 if torch.cuda.is_available() else torch.float32

pipe = DiffusionPipeline.from_pretrained(
    MODEL_ID,
    torch_dtype=dtype,
    variant="fp16" if dtype==torch.float16 else None,
)
pipe = pipe.to("cuda" if torch.cuda.is_available() else "cpu")

# --- Prompt setup ---
prompt = "a cozy modern reading nook with a big window, soft natural light, photorealistic"
negative_prompt = "low quality, blurry, distorted, text, watermark"

# --- Generation settings ---
height = 1024
width = 1024
steps = 30
guidance = 7.0

# --- Generate ---
result = pipe(
    prompt=prompt,
    negative_prompt=negative_prompt,
    num_inference_steps=steps,
    guidance_scale=guidance,
    height=height,
    width=width,
)

# --- Save to file ---
image: Image.Image = result.images[0]
display(image)
image.save(f"sdxl_output.png")
print(f"Saved image as sdxl_output.png")

```






Test with sample AI workload , 貼上SDXL範例 prompt = "一個舒適的現代閱讀角落，配有大窗戶，柔和的自然光，照片級真實感"   <br>
![jupyter](images/jupyter-post.png)<br>
執行SDXL範例生成圖片<br>
![jupyter](images/jupyter-run-sd.png)<br>


檢視SDXL範例生成圖片實際存在的目錄與開啟圖片<br>
![sdxl-output](images/sdzl-output.jpg)<br>





