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

## 與我們今天實測的對照

| 項目 | 官方標準做法 | 我們今天實際走的路 |
|---|---|---|
| 新增網域方式 | `nemoclaw policy-add --from-file <preset.yaml> --yes`（增量、熱重載） | `openshell policy get --full` 匯出 → 手動編輯 → `openshell policy set --policy <file> --wait`（整份覆蓋） |
| 為什麼繞遠路 | — | 當時只測試過**互動式** `nemoclaw policy-add`（只顯示 11 個內建 preset），沒發現 `--from-file` 這個非互動參數能吃自訂 YAML |
| Endpoint 存取模式 | 建議用 `access: full` + `tls: skip`（原始穿透，適合唯讀抓取） | 用了 `protocol: rest` + `enforcement: enforce` + `rules`（L7 過濾），需要 proxy 終止 TLS，對唯讀抓取來說其實不必要，但today一樣測試成功 |
| Preset 命名 | 必須是小寫連字號（RFC 1123），如 `news-sources` | 用了底線 `news_sources`——因為我們是直接改整份 `network_policies` 底下的 key，不是走 `policy-add` 的 preset 驗證流程，剛好沒有被擋下來 |
| 排程註冊方式 | `openclaw cron add`，操作端執行，`--no-deliver` 因為沒接通道時必要 | `openclaw cron create`，同樣是操作端在 sandbox shell 內執行，直接指定 `--announce --channel discord --to <id>` 投遞 |
| `Version:` 大小寫 bug | 官方文件明確記錄為 OpenShell 0.0.44 已知 bug，並給出 `sed` 修正法 | 我們今天實際踩到同一個錯誤，也是用同樣的 `sed -i 's/^Version:/version:/'` 方式修正 |

兩條路殊途同歸，都能達成放行自訂新聞來源網域的目的，但官方 `policy-add --from-file` 這條路**風險更低**（增量合併，不會誤刪其他既有設定），值得作為之後的標準做法。

---

*（下一部分：Software Development Agent）*
