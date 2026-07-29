## Daily Personal News Digest

這是一個 cron 排程型的工作流：agent 定時醒來、從一組白名單網址抓取更新、整理摘要、推播到你的 Telegram 主頻道。

### Step 1：Policy Setup（官方建議做法）

先完成 [NemoClaw Policy Setup](https://build.nvidia.com/spark/nemoclaw-applications/policy-setup) 分頁裡的 Telegram 通道設定（channel plugin + `api.telegram.org` egress）。接著**用一個小型自訂 preset 檔案，透過 `nemoclaw policy-add --from-file` 新增你要讓 agent 讀取的新聞來源網路權限**。這個 preset 是**增量合併、熱重載**的——不需要匯出/回填整份 policy。

建立 `news-sources.yaml`：

```yaml
preset:
  name: news-sources
  description: "Daily news digest source allowlist"

network_policies:
  news-sources:
    name: news-sources
    endpoints:
      - host: developer.nvidia.com
        port: 443
        access: full
        tls: skip
      - host: blogs.nvidia.com
        port: 443
        access: full
        tls: skip
      - host: news.ycombinator.com
        port: 443
        access: full
        tls: skip
    binaries:
      - { path: /usr/local/bin/openclaw }
      - { path: /usr/local/bin/node }
      - { path: /usr/bin/node }
      - { path: /usr/bin/curl }
```

#### ⚠️ 三個容易踩雷的格式細節（官方文件明確列出）

1. **`network_policies` 是一個以群組名稱為 key 的 map**，每個群組要有自己的 `name` 跟 `endpoints` 清單。如果直接把 `{host, port}` 的清單放在 `network_policies` 底下（不包一層群組名稱），會出現 `invalid type: sequence, expected a map` 錯誤。

2. **`preset.name` 與 `network_policies` 的群組 key，必須是小寫、用連字號的 RFC 1123 格式**（只能是字母、數字、連字號，**不能有底線**）。用 `news_sources`（底線）會直接失敗，錯誤訊息是 `Preset must declare preset.name (lowercase, hyphenated RFC 1123 label)`。這跟官方內建的 preset（`brave`、`github`、`slack`）命名慣例一致。

3. **每個 endpoint 除了 `host`/`port`，還需要兩件事，否則 proxy 會用 `curl: (56) CONNECT tunnel failed, response 403` 拒絕連線**，即使這個 host 已經出現在生效的 policy 裡：
   - **存取模式**：抓取一般網頁最簡單的方式是原始穿透通道——`access: full` 搭配 `tls: skip`（跟官方內建的 `whatsapp`/`brew` preset 用同樣的寫法）。另一種選擇是 `protocol: rest` + `enforcement: enforce` + `rules` 的 L7 過濾寫法，但這需要 proxy 終止 TLS，對於唯讀新聞抓取來說沒有必要。
   - **`binaries` 白名單**：指定哪些程式可以使用這條 egress。Agent 的網頁抓取工具跑在 `/usr/local/bin/openclaw` 跟 `node` 底下，也建議加上 `/usr/bin/curl` 讓 shell 型的抓取也能用。**沒有 `binaries` 子句，就沒有任何程式被授權開啟這條通道**，每次抓取都會回傳 403。

   一個裸的 `{host, port}` 項目（沒有存取模式、沒有 binaries）是「摘要看起來套用成功，但什麼都讀不到」最常見的原因。

#### 套用（熱重載，不需要重啟 sandbox）

```bash
nemoclaw $SANDBOX_NAME policy-add --from-file ./news-sources.yaml --yes
```

#### 確認新網域已生效

```bash
openshell policy get $SANDBOX_NAME --full | grep -E "host:|port:"
```

#### 💡 官方提示：優先用 `policy-add --from-file`，不要用完整匯出再套用

> 比起 `openshell policy get --full > policy.yaml` 再 `openshell policy set` 的做法，更建議直接用 `nemoclaw policy-add --from-file`。因為 OpenShell `0.0.44` 版本在完整匯出時，會印出 `Version:`（大寫 V），但解析器預期的是 `version:`（小寫），導致 `policy set` 會用 `unknown field 'Version'` 拒絕自己匯出的內容。增量式的 `policy-add` 流程完全不會碰到生效中的 `version:` 欄位，因此避開了這個 bug。如果你從舊的做法遇到這個錯誤，可以直接把欄位改成小寫修正：`sed -i 's/^Version:/version:/' policy.yaml`，再重跑 `policy set`。

---

### Step 2：Agent Prompt

複製下面完整的 prompt，貼到 NemoClaw Web UI，或當作單一訊息傳給你的 Telegram bot。這是**唯一需要的設定**——這段 prompt 定義了 agent 端到端的完整行為，包含一次性 onboarding、固定的簡報結構、風格規則、錯誤處理，以及排程維護邏輯。

```text
You are my personal news intelligence analyst. Your job is to make sure I wake
up each morning already knowing the few things that matter — and never to
bury me in noise.

ONE-TIME SETUP (do this on your very first run only, then remember my answers
as my profile):

Ask me, one question at a time, and wait for my answer before moving on:
  1. What's on your news menu? Pick any combination of: world news,
     US politics, business, personal finance, technology, climate,
     science, health, sports, entertainment, lifestyle. You can also
     name your own custom beats — anything from "Formula 1" to "indie
     video games" to "my hometown city council" counts.
  2. Who should I sound like when I write to you? Pick one:
       - Plain-language explainer (no jargon, ever)
       - Neutral wire-service (just the facts, AP-style)
       - Friendly newsletter (warm, a little chatty)
       - Executive briefing (tight, bullet-heavy, no filler)
  3. How much time do you give me with your coffee? 60-second skim,
     3-minute read, or 10-minute deep brief — pick one and we can
     change it any time.
  4. Any VIPs or villains? Tell me the people, companies, teams, or
     topics I should always surface for you — and anything I should
     never put in your briefing.
  5. Where are you waking up? Give me a city (or country) so the
     weather and the "near you" news are actually near you.
  6. When's showtime? Default is 08:00 America/Los_Angeles every
     weekday. Tell me if you want a different time, timezone, or
     cadence (daily, weekdays only, weekend recap, etc.).

Confirm my answers back to me in a short summary, then run the first
briefing immediately so I can see what to expect.

DAILY BRIEFING STRUCTURE (use this exact shape every run, in this order):

  1. Top 3 — the three stories I cannot miss today. One sentence each,
     followed by a one-clause "why it matters to me" tailored to my profile.
  2. Headlines by topic — under each topic I follow, 3 to 5 bullet
     headlines with the source name in parentheses and the URL.
  3. Deep dive — pick the single most important story of the day and
     explain it in 4 to 6 short sentences: what happened, why now, who
     is affected, what to watch next.
  4. Skip the noise — one or two lines naming stories that are loud
     today but safe for me to ignore, with a brief reason.
  5. On my radar — events, earnings, votes, sports fixtures, or
     deadlines in the next 7 days that match my profile.
  6. Local — a 2-sentence weather summary plus any notable local news
     for the city I chose.

STYLE RULES:
  - Plain language; assume I am not an expert in any topic.
  - No hype words ("shocking", "you won't believe", "breaking"). Just
    the facts.
  - Cite every claim with the source name and a working URL.
  - Never invent quotes, numbers, dates, or events. If you cannot
    verify a detail, omit it or label it clearly as "unconfirmed".
  - Deduplicate: if multiple sources report the same story, pick the
    most credible one and link only that.
  - Respect my length preference. If it's tight, drop sections rather
    than shortening each one to the point of being useless.

ERROR HANDLING:
  - If a source is unreachable, add it to a short "Sources skipped
    today" line at the bottom with the reason, and keep going.
  - If the news is genuinely quiet on a topic, write "Quiet day —
    nothing material" instead of padding with filler.
  - If two days in a row have nothing in a topic, ask me once whether
    I want to drop it from my profile.

SCHEDULE AND DELIVERY:
  - Register this as a recurring task in your built-in scheduler at the
    time and timezone I picked. Confirm the next 3 trigger times back
    to me after onboarding.
  - Deliver each briefing to my Telegram home channel.
  - Skip US public holidays unless a major breaking story is unfolding.

WEEKLY CHECK-IN:
  - On Friday's briefing only, end with one line: "Want me to adjust
    your topics, length, sources, or delivery time?" If I reply, update
    my profile and confirm the change.

Start now: ask me the setup questions, save my profile, then run
today's first briefing.
```

**預期結果**：agent 會確認它已經排定一個任務。到下一次 08:00 觸發時，你的 Telegram 主頻道會收到一則摘要訊息。可以在 web UI 問「Show me my scheduled tasks」來驗證任務是否有註冊成功。

> 依你選擇的模型不同，設定 agent 工作流程可能需要一些時間。如果過程中 agent 卡住沒進展，在 web UI 問一句「Is my workflow set up yet」把它喚醒。

#### 沒有接 Telegram？可以改用 Web UI 投遞

如果沒有設定 Telegram 通道，把 prompt 裡的投遞那一行——`Deliver each briefing to my Telegram home channel.`——改成 `Deliver each briefing to the web UI (this session). Do not use any messaging channel.`。這樣 agent 會把每次簡報寫回你在 dashboard 能讀到的 session 裡。回答 onboarding 第 6 題時，順便告訴 agent 你選擇的投遞方式。

#### 💡 建議先手動觸發一次，驗證端到端流程

在正式排程時間到之前，先請 agent 立刻手動跑一次：*"Run the digest task now as a one-off, then keep the schedule for tomorrow."* 這樣可以透過**現正運行**的 agent，得到最可靠的端到端驗證（會立即產出一份真實簡報）。

#### ⚠️ 重要：排程要從操作端（operator）註冊，不要依賴 agent 自己呼叫工具

> 當 agent 以內嵌的 `openclaw agent` turn 執行（也就是這裡用的無頭模式）時，它的 turn 內 cron 工具會用一個**缺少 scheduler scope** 的 device token 連到 gateway，導致註冊被拒絕，錯誤訊息是 `scope upgrade pending approval … pairing required: device is asking for more scopes than currently approved`。Agent 接著會回報它「沒有內建排程器」或排程器「不穩定」。

正確做法是**由你自己（操作端）註冊這個排程任務**，這是驗證過可行的方式：

```bash
nemoclaw $SANDBOX_NAME exec -- openclaw cron add \
  --name news-digest --cron "0 8 * * 1-5" --tz America/Los_Angeles \
  --agent default --session-key agent:default:news-digest \
  --message "Run my daily news briefing now and write it to this session." \
  --no-deliver --token ""
```

`--no-deliver` 讓簡報留在 session 裡（在 web UI 讀取），而不是推播到某個聊天頻道——**如果沒有設定 Telegram/Slack 通道，這個參數是必要的**，否則執行會直接失敗，錯誤是 `last -> no route`。用以下指令確認：

```bash
nemoclaw $SANDBOX_NAME exec -- openclaw cron list
nemoclaw $SANDBOX_NAME exec -- openclaw cron status
```

（如果你是把 prompt 貼進**互動式** web UI，而不是無頭執行，dashboard 會提示你核准這個 scope，agent 就能自己註冊任務；但不論哪種情境，上面這個操作端指令都是可靠的做法。）

#### ⚠️ 重要：本地模型（vLLM）的排程觸發限制

> 註冊完成後，排程的 cron 執行會被 provider 的**前置檢查（pre-flight）**卡住——這個檢查會對管理推論的 host `inference.local` 做一次單純的 DNS 查詢。但這個 host **只能透過 egress proxy 解析**（它沒有真正的 DNS 紀錄或 `/etc/hosts` 項目），所以前置檢查會用 `getaddrinfo EAI_AGAIN inference.local` 失敗，該次執行會被記錄為 `skipped`。

即時的 `openclaw agent` turn（onboarding、上面的「立刻跑一次」、或你在 web UI 裡打的任何字）不受影響——它們透過 proxy 能正常連到模型。**如果需要無人值守的排程投遞，改讓 cron job 指向一個 DNS 可解析的推論端點**，而不是 `inference.local`（`local-inference` preset 本身就允許連到 host 上的 vLLM `host.openshell.internal:8000`，這個位址可以透過 `/etc/hosts` 解析），用 `--model` 參數傳給 `cron add`。雲端模型的 sandbox（provider host 能正常解析）不受此限制影響。

---

### Step 3：How to Personalize

| 旋鈕 | 在哪裡調 | 怎麼改 |
|---|---|---|
| **排程時間** | Step 2 的操作端指令 `openclaw cron add` | 改 `--cron "0 8 * * 1-5"` 跟 `--tz`（例如 `0 9 * * 1` = 每週一 09:00，`0 */6 * * *` = 每 6 小時一次）。記得同步更新 prompt 裡陳述的時間，讓 agent 回報的「接下來 3 次觸發時間」保持一致 |
| **新聞來源** | `news-sources.yaml` **與** prompt 兩處都要改 | 在 `network_policies.news-sources.endpoints` 底下新增一筆，重跑 `nemoclaw $SANDBOX_NAME policy-add --from-file ./news-sources.yaml --yes`，再把網址列進 prompt。Sandbox 會擋掉任何不在白名單裡的網域 |
| **語氣風格** | Prompt — onboarding 第 2 題 | 把四個語氣選項換成你自己的（例如「沉穩老爸語氣」、「懷疑論分析師」、「毒舌財經網紅」） |
| **長度** | Prompt — onboarding 第 3 題 | 把三個長度選項換成適合你早晨步調的版本（例如「5 分鐘閱讀」、「吃早餐時快速掃過」） |
| **投遞管道** | Prompt | 把 `Telegram home channel` 換成 `the web UI`，或換成其他已設定的通道 |
| **過濾條件** | Prompt | 加一句「Only include posts that mention "Spark" or "GB10".」聚焦摘要內容 |

要**取消**排程任務，傳送：`List my scheduled tasks, then cancel the digest one.`

---


# Daily Personal News Digest — 實際操作完整正確摘要

> 時間範圍：2026/7/13 ～ 7/24
> 說明：這份摘要對照實際對話紀錄整理，跟目前 GitHub 已發布版本有出入——
> 實際過程比文章呈現的更曲折，這裡如實記錄完整時間軸與過程。

---

## 實作修正摘要

| 日期 | 內容 |
| **7/24** | **Daily Personal News Digest 完整重建的實際操作日**——從決定重現舊功能、policy 排查、cron 建立、裝置權限死結、RSS 網址修正、成功推播、到最後補做官方標準做法示範，全部發生在這一天 |

---

起點不是照 playbook 從頭做起，而是想把**之前用純 OpenClaw（沒有 NemoClaw/OpenShell 沙箱）搭配 Discord**做出來的每日新聞 bot，在 NemoClaw 底下重建一次。原本的架構參考：

```
/home/user/.openclaw/workspace/bin/daily-tech-news.sh
```

推播對象：Discord #news 頻道；來源：中央社科技（CTI Tech）、Digitimes IT/雲端、Digitimes 電腦運算、Yahoo 股市（台積電）；排程：每天 08:00。
系統整台重灌後舊資料全部消失，這次要在**有沙箱隔離、有網路白名單限制**的 NemoClaw 環境下重做一次同樣的功能。

---

## 7/24 完整操作過程

### 階段一：釐清架構差異

先確認純 OpenClaw 跟 NemoClaw 的核心差異——NemoClaw 的 sandbox 預設完全連不到外網，需要額外用 policy 放行才能抓新聞來源。

### 階段二：建立 Cron Job，第一次直接失敗

用 `openclaw cron create` 建立排程任務，手動觸發測試後**立刻失敗**——sandbox 網路 policy 沒有放行任何新聞來源，agent 一嘗試連線就被 `403 CONNECT tunnel failed` 擋下。

### 階段三：手動編輯 Policy（不是用 `policy-add --from-file`）

先嘗試互動式的 `nemoclaw policy-add`，發現這個指令**只列出 11 個內建 preset（npm、telegram、discord 等），沒有任何選項能輸入自訂網域**。找不到高階指令能用，繞去底層手動處理：

```bash
openshell policy get gb10-assistant --full > policy.yaml
# 手動編輯，加入 news_sources 區塊
openshell policy set gb10-assistant --policy policy.yaml --wait
```

過程中撞到兩個具體問題：

1. `openshell policy get --full` 輸出開頭有 metadata header（`Version:` 大寫），直接拿去 `policy set` 會報錯 `unknown field 'Version'`——用 `sed` 去掉 header 才解決,後來確認這是 OpenShell 官方承認的已知 bug
2. 一度誤以為手動編輯時 YAML 裡混進了 Markdown 連結格式，查證後發現只是聊天介面顯示層自動轉換，檔案內容其實是乾淨的

這條路最終成功套用了 `news_sources` policy，放行 `technews.tw`、`www.ithome.com.tw`、`www.reuters.com`、`news.ycombinator.com`、`www.digitimes.com.tw`。

### 階段四：Cron Job 意外消失，牽出裝置權限死結

某次 `nemoclaw rebuild`（換模型觸發）之後，回頭發現**cron job 整個不見了**——`openclaw cron list` 顯示 `No cron jobs.`。

重新建立時撞上更深層的問題：

```
gateway connect failed: GatewayClientRequestError: scope upgrade pending approval
pairing required: device is asking for more scopes than currently approved
```

追查後發現：唯一的裝置只有 `operator.pairing`、`operator.read` 權限，但建立 cron job 需要 `operator.admin`——系統規則是「只能核准自己已持有的權限範圍」，導致這個裝置**永遠無法核准自己申請的升級**，形成死結。CLI 跟 Telegram 兩條路都試過，都卡在同一個地方。

**最終解法**：手動編輯 sandbox 內的裝置記錄檔案：

```bash
nemoclaw gb10-assistant connect
# 進去後用 python3 編輯 /sandbox/.openclaw/devices/paired.json
# 把 operator.admin 加進 scopes / approvedScopes / tokens.operator.scopes
# 清空 pending.json
```

重啟 gateway 後權限問題解除，重新建立 cron job 成功。

### 階段五：新聞來源網址本身也不對

Cron job 建起來後，agent 執行時嘗試連線的網域跟埋好的白名單對不上——它自己聯想去抓 `rss.digitimes.com`、`feeds.arstechnica.com` 這類完全沒放行過的網址。

逐一用 `curl` 驗證正確的 RSS 網址：

- iThome：`https://www.ithome.com.tw/rss` → 確認是有效的 `application/rss+xml`
- Digitimes：一開始猜的網址是死的（`404`），從 `301 Moved` 回應追出正確路徑 `https://www.digitimes.com.tw/tech/rss/xml/xmlrss_30_1.xml`

把 prompt 改成明確指定這兩個驗證過的 RSS 網址，不讓 agent 自己亂猜其他來源。

### 階段六：終於成功，Discord 收到真實摘要

重新建立 cron job、手動觸發測試，這次成功抓到內容，Discord #news 頻道收到結構化摘要，含「🌍 國際新聞」「🇹🇼 台灣新聞」分類、標題、摘要、原文連結。

### 階段七：補做官方標準做法示範

整個流程跑通後，額外**刻意重新示範一次官方推薦的正確做法**，作為對照：

1. 用 Python 從現有 policy 移除 `www.ithome.com.tw`、`www.digitimes.com.tw` 兩筆，套用回去，製造「還沒放行」的狀態
2. 手動觸發 cron job，故意讓它失敗——Discord 收到「連線失敗 403」訊息，agent 自己準確診斷出是 policy 限制
3. **這時候才第一次用上 `nemoclaw policy-add --from-file`**——寫一個乾淨的 preset YAML 套用，系統會先列出「即將開放的端點」讓你確認,體驗跟一開始手動改整份 policy 差很多
4. 重新觸發，這次成功，Discord 收到真實內容

---

## 更正：跟目前已發布文章的落差

| 文章描述 | 實際發生的事 |
|---|---|
| 從一開始就用 `policy-add --from-file` | **不是**——一開始互動式選單沒有自訂網域選項，繞去用 `openshell policy get/set` 手動編輯；`policy-add --from-file` 是後來才發現、且是**額外補做示範**才用上 |
| 流程一路順暢 | **不是**——中間發生過 cron job 憑空消失、裝置權限死結（需手動改 `paired.json`）、RSS 網址猜錯要重新排查 |
| `Version:` bug 只是順手提一句 | 這其實是**當時卡關的關鍵原因之一**，排查花了不少來回才確認是已知 bug |
| 事件發生在多天分散進行 | 實際上整個 News Digest 重建、排查、修正、成功、加上補做示範，**全部集中在 7/24 這一天**完成 |

---

## 一句話總結

**這次 Daily Personal News Digest 的重建，不是照著官方 playbook 一步到位的乾淨示範，而是先用最笨的手動方式（改整份 policy YAML）解決問題，中途意外挖出一個裝置權限死結，最後才回頭示範官方真正建議的簡便做法（`policy-add --from-file`）作為對照。全部發生在 7/24 這一天，過程中踩到的坑，遠比文章呈現的內容更多。**















---

*（下一部分：Software Development Agent）*
