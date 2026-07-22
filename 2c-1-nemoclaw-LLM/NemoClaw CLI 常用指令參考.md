# NemoClaw CLI 常用指令參考

> 版本：`NemoClaw v0.0.81`（`nemoclaw` 指令本身可執行 `nemoclaw` 查看完整說明）
> 官方文件：https://docs.nvidia.com/nemoclaw/user-guide/openclaw/home

## 指令格式說明

NemoClaw 指令分兩種：

- **全域指令**：不需要 sandbox 名稱前綴，例如 `nemoclaw status`、`nemoclaw list`
- **Sandbox 指令**：需要以 sandbox 名稱開頭，例如 `nemoclaw gb10-assistant status`

---

## 入門：建立與連線

| 用途 | 指令 |
|---|---|
| 啟動互動精靈，設定推論來源與憑證 | `nemoclaw onboard` |
| 用自訂 Dockerfile 建置 sandbox image | `nemoclaw onboard --from <Dockerfile>` |
| 列出可用的 agent runtime | `nemoclaw agents list` |
| 連進 sandbox（互動 shell） | `nemoclaw <name> connect` |
| 只探測 sandbox 是否就緒，不進入 shell | `nemoclaw <name> connect --probe-only` |

**範例：**
```bash
nemoclaw onboard
nemoclaw gb10-assistant connect
```

---

## Sandbox 管理

| 用途 | 指令 |
|---|---|
| 列出所有 sandbox | `nemoclaw list` |
| 設定預設 sandbox | `nemoclaw use <name>` |
| 查看單一 sandbox 健康狀態 | `nemoclaw <name> status` |
| 查看全域整體狀態 | `nemoclaw status` |
| 印出 Dashboard 網址 | `nemoclaw <name> dashboard-url --quiet` |
| 修復停止的 gateway/host forward | `nemoclaw <name> recover` |
| 非互動執行單一指令 | `nemoclaw <name> exec -- <cmd>` |
| 非互動執行一次 agent 對話 | `nemoclaw <name> agent -m "訊息"` |
| 從 sandbox 下載檔案到 host | `nemoclaw <name> download <sandbox-path> [host-dest]` |
| 從 host 上傳檔案到 sandbox | `nemoclaw <name> upload <host-path> [sandbox-dest]` |
| 健康診斷（可自動修復） | `nemoclaw <name> doctor --fix` |
| 串流 sandbox log | `nemoclaw <name> logs --follow` |
| 升級 sandbox 到目前 agent 版本 | `nemoclaw <name> rebuild` |
| 印出 sandbox agent 的認證 token | `nemoclaw <name> gateway-token --quiet` |
| 重啟 sandbox gateway | `nemoclaw <name> gateway restart` |
| 停止並刪除 sandbox | `nemoclaw <name> destroy --yes` |

**範例：**
```bash
nemoclaw list
nemoclaw gb10-assistant status
nemoclaw gb10-assistant logs --follow
nemoclaw gb10-assistant exec -- ollama list
nemoclaw gb10-assistant download /sandbox/.openclaw/workspace/memory/daily-news-summary.md ./
nemoclaw gb10-assistant rebuild
```

---

## Snapshot（快照備份/還原）

| 用途 | 指令 |
|---|---|
| 建立快照 | `nemoclaw <name> snapshot create --name <name>` |
| 列出所有快照 | `nemoclaw <name> snapshot list` |
| 還原快照 | `nemoclaw <name> snapshot restore <selector>` |

**範例：**
```bash
nemoclaw gb10-assistant snapshot create --name before-model-switch
nemoclaw gb10-assistant snapshot list
```

---

## 檔案系統掛載（Share）

| 用途 | 指令 |
|---|---|
| 把 sandbox 檔案系統掛載到 host | `nemoclaw <name> share mount [sandbox-path] [local-mount-point]` |
| 卸載掛載點 | `nemoclaw <name> share unmount [local-mount-point]` |
| 查看掛載狀態 | `nemoclaw <name> share status` |

---

## Skills（技能）

| 用途 | 指令 |
|---|---|
| 部署技能到 sandbox | `nemoclaw <name> skill install <path>` |
| 移除已安裝的技能 | `nemoclaw <name> skill remove <skill>` |

---

## Policy（網路/檔案系統權限）★ 今天最常用的一組

| 用途 | 指令 |
|---|---|
| 匯出完整、可回填的 base policy | `nemoclaw <name> policy-get --raw` |
| 新增內建 preset（網路或檔案系統） | `nemoclaw <name> policy-add` |
| 移除已套用的 preset | `nemoclaw <name> policy-remove` |
| 列出目前套用的 preset | `nemoclaw <name> policy-list` |
| 解釋目前生效的完整 policy 內容 | `nemoclaw <name> policy-explain --json` |
| 新增 /etc/hosts 別名 | `nemoclaw <name> hosts-add <hostname> <ip>` |
| 列出 hosts 別名 | `nemoclaw <name> hosts-list` |
| 移除 hosts 別名 | `nemoclaw <name> hosts-remove` |

**範例：**
```bash
nemoclaw gb10-assistant policy-list
nemoclaw gb10-assistant policy-add
nemoclaw gb10-assistant policy-explain --json
```

> ⚠️ **重要限制（今天實測發現）**：`policy-add` 只能從**固定的內建 preset 清單**（npm、brew、telegram、discord 等約 11 個）中選擇，**不支援輸入任意自訂網域**。如果要放行自訂新聞網站這類非內建服務，`policy-add` 這條路走不通，需要改用底層的 `openshell policy get --full` / `openshell policy set --policy <file> --wait` 手動編輯完整 YAML 來達成（見下方「與 openshell 的關係」）。

---

## Messaging Channels（Telegram / Discord 等）

| 用途 | 指令 |
|---|---|
| 列出支援的通訊管道 | `nemoclaw <name> channels list` |
| 新增管道憑證並重建 | `nemoclaw <name> channels add <channel>` |
| 移除管道憑證並重建 | `nemoclaw <name> channels remove <channel>` |
| 暫停管道（保留憑證） | `nemoclaw <name> channels stop <channel>` |
| 重新啟用已停用的管道 | `nemoclaw <name> channels start <channel>` |
| 查看管道狀態 | `nemoclaw <name> channels status --channel <channel>` |

**範例：**
```bash
nemoclaw gb10-assistant channels list
nemoclaw gb10-assistant channels status --channel telegram
```

---

## MCP Servers

| 用途 | 指令 |
|---|---|
| 列出已設定的 MCP server | `nemoclaw <name> mcp list` |
| 新增受 OpenShell 管控的 MCP HTTP server | `nemoclaw <name> mcp add <server> --url <url>` |
| 查看 MCP server 健康狀態 | `nemoclaw <name> mcp status` |
| 重新整理 MCP server 註冊 | `nemoclaw <name> mcp restart` |
| 移除 MCP server | `nemoclaw <name> mcp remove <server>` |

---

## 推論模型設定（Inference）

| 用途 | 指令 |
|---|---|
| 查看目前生效的推論路由 | `nemoclaw inference get --json` |
| 切換推論模型/provider | `nemoclaw inference set --model <model> --provider <provider> --sandbox <name>` |

**範例（今天實際用過）：**
```bash
nemoclaw inference get --json
nemoclaw inference set --model qwen3.6:35b --provider ollama-local --sandbox gb10-assistant
```

---

## 服務管理（Tunnel / 全域狀態）

| 用途 | 指令 |
|---|---|
| 啟動 cloudflared 對外 tunnel | `nemoclaw tunnel start` |
| 停止 tunnel | `nemoclaw tunnel stop` |
| 查看 tunnel 狀態 | `nemoclaw tunnel status` |
| 查看全域 sandbox/host 服務狀態 | `nemoclaw status --json` |

---

## Credentials（憑證）

| 用途 | 指令 |
|---|---|
| 新增 provider 憑證 | `nemoclaw credentials add <PROVIDER> --type <TYPE>` |
| 列出已儲存的憑證 provider | `nemoclaw credentials list` |
| 移除憑證 | `nemoclaw credentials reset <PROVIDER> --yes` |

---

## Backup / Upgrade / Resources / Cleanup

| 用途 | 指令 |
|---|---|
| 升級前備份所有 sandbox 狀態 | `nemoclaw backup-all` |
| 執行官方安裝程式的更新流程 | `nemoclaw update` |
| 偵測並重建過期的 sandbox | `nemoclaw upgrade-sandboxes --check` |
| 查看硬體資源（CPU/RAM/GPU VRAM） | `nemoclaw resources --json` |
| 清除孤兒 Docker image | `nemoclaw gc --dry-run` |
| 解除安裝 NemoClaw | `nemoclaw uninstall` |

**範例：**
```bash
nemoclaw resources --json
nemoclaw gc --dry-run
```

---

## 疑難排解

| 用途 | 指令 |
|---|---|
| 收集除錯資訊供回報問題 | `nemoclaw debug --quick -o debug.log` |

---

## 常見重新設定情境對照（官方內建提示整理）

| 想做的事 | 對應指令 |
|---|---|
| 檢查目前推論路由 | `nemoclaw inference get` |
| 換推論模型 | `nemoclaw inference set --model <model> --provider <provider>` |
| 新增網路 preset（僅限內建服務） | `nemoclaw <name> policy-add` |
| 換憑證 | `nemoclaw credentials reset <PROVIDER>`，再重跑 `nemoclaw onboard` |
| 鎖定設定（用於敏感工作負載） | `nemoclaw <name> shields up` |

---

## 與 `openshell` 底層指令的關係

NemoClaw 的 `policy-*` 系列指令是**建立在 `openshell` 之上的高階封裝**，但功能範圍較窄（僅支援內建 preset）。如果需要更精細的控制（例如自訂網域白名單），需要直接呼叫底層的 `openshell` 指令：

| 情境 | 用 nemoclaw 還是 openshell |
|---|---|
| 加開/移除官方內建服務（npm、GitHub、Telegram 等） | `nemoclaw <name> policy-add` / `policy-remove` |
| 加開自訂網域（如一般新聞網站） | `openshell policy get <name> --full` 取得完整 YAML → 手動編輯 → `openshell policy set <name> --policy <file> --wait` |
| 只是想看目前完整 policy 內容 | `nemoclaw <name> policy-get --raw` 或 `openshell policy get <name> --full` 兩者皆可 |

---

## 延伸閱讀

- 官方 User Guide：https://docs.nvidia.com/nemoclaw/user-guide/openclaw/home
- Network Policies 參考：https://docs.nvidia.com/nemoclaw/latest/reference/network-policies.html
- Customize Network Policy：https://docs.nvidia.com/nemoclaw/latest/network-policy/customize-network-policy.html
- NemoClaw GitHub Repo：https://github.com/NVIDIA/NemoClaw
