## 補充：如何把 `my-app.tar.gz` 從 host 放進 sandbox

如果你已經有一份打包好的 `my-app.tar.gz`（例如放在 `/home/user/` 底下，或是像你 repo 裡
[`my-app.tar.gz`](https://github.com/install-safe-press/gb10-playbooks/blob/main/2c-2-NemoClaw-Example/my-app.tar.gz)
這樣先準備好的檔案），不需要重新打包，直接照下面步驟操作即可。

### 1. 確認檔案確實在 host 上，並檢查內部結構

```bash
ls -la /home/user/my-app.tar.gz
tar tzf /home/user/my-app.tar.gz | head -10
```

**這一步很重要**：確認打包時有沒有多包一層外層資料夾。如果 `tar tzf` 印出來的每一行都是 `my-app/...` 開頭（例如 `my-app/app.py`、`my-app/tests/`），代表解壓縮時需要用 `--strip-components=1` 去掉這層外殼，否則專案會變成巢狀的 `/sandbox/project/my-app/app.py`，跟後續 prompt 裡假設的路徑 `/sandbox/project/app.py` 不一致。

### 2. 用 `nemoclaw upload` 把檔案傳進 sandbox（不要用 tar 管道直接串流）

今天實測發現，官方文件建議的做法：

```bash
tar czf - -C ~/nemoclaw-projects/my-app . \
  | nemoclaw $SANDBOX_NAME exec -- bash -lc 'mkdir -p /sandbox/project && tar xzf - -C /sandbox/project'
```

在這個環境會出現 `gzip: stdin: unexpected end of file` 的錯誤，管道中途被截斷。**改用官方本來就有的 `nemoclaw upload` 指令**更穩定，因為它讓檔案先完整落地在兩端，不受管道緩衝區大小或逾時影響：

```bash
nemoclaw gb10-assistant upload /home/user/my-app.tar.gz /sandbox/my-app.tar.gz
```

成功會看到：

```
Uploading /home/user/my-app.tar.gz -> sandbox:/sandbox/my-app.tar.gz
✓ Upload complete
```

### 3. 在 sandbox 裡解壓縮到正確路徑

如果第 1 步確認過打包時多包了一層 `my-app/` 外殼，解壓縮時加上 `--strip-components=1`：

```bash
nemoclaw gb10-assistant exec -- bash -lc 'mkdir -p /sandbox/project && tar xzf /sandbox/my-app.tar.gz -C /sandbox/project --strip-components=1 && rm /sandbox/my-app.tar.gz'
```

如果打包時本來就沒有外層資料夾（直接對內容打包，不是對資料夾本身打包），就不需要 `--strip-components=1`：

```bash
nemoclaw gb10-assistant exec -- bash -lc 'mkdir -p /sandbox/project && tar xzf /sandbox/my-app.tar.gz -C /sandbox/project && rm /sandbox/my-app.tar.gz'
```

### 4. 驗證解壓縮後的結構是否正確（沒有巢狀目錄）

```bash
nemoclaw gb10-assistant exec -- ls -la /sandbox/project
```

**預期看到**（直接是專案內容，不是又一層 `my-app/`）：

```
app.py
requirements.txt
tests/
README.md
.gitignore
.git/
```

**如果看到的是**：

```
my-app/
```

代表 `--strip-components=1` 沒有正確處理，需要先清空重來：

```bash
nemoclaw gb10-assistant exec -- rm -rf /sandbox/project
```

回到步驟 3，重新解壓縮並補上 `--strip-components=1`。

### 5. 最後確認 git 歷史與檔案內容都完整

```bash
nemoclaw gb10-assistant exec -- bash -lc 'cd /sandbox/project && git --no-pager log --oneline'
nemoclaw gb10-assistant exec -- cat /sandbox/project/app.py
```

`git log` 應該要看到 `Initial commit`；`app.py` 內容應該只有最初的 `/` 路由（如果這份 tar.gz 是乾淨的初始版本）。

---

### 常見錯誤對照表

| 現象 | 原因 | 解法 |
|---|---|---|
| `gzip: stdin: unexpected end of file` | 用 tar 管道直接串流進 `nemoclaw exec`，管道不穩定 | 改用 `nemoclaw upload` 分兩步（先傳檔案、再解壓縮） |
| `ls /sandbox/project` 顯示又一層 `my-app/` | 打包時對資料夾本身打包（`tar czf x.tar.gz my-app/`），而非對資料夾內容打包 | 解壓縮時加 `--strip-components=1` |
| `git log` 沒有任何輸出 | 其實只是 `git log` 預設呼叫 pager，在非互動環境下輸出被吃掉，不是真的沒有 commit | 一律用 `git --no-pager log`，不要用純 `git log` |
