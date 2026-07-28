## Doc & Deck Red-Team Agent

參考官方 Playbook：[Deck Reviewer](https://build.nvidia.com/spark/nemoclaw-applications/deck-reviewer)

### 這個應用在做什麼

**一句話**：在你把簡報/文件送出去或上台報告**之前**，先讓 agent 幫你抓包——數字跨頁不一致、沒有來源的主張、缺 alt-text 的圖片、跟舊版矛盾的內容。Agent 讀取你即將出貨的檔案（PPTX、DOCX、PDF、Markdown），對照一份精心整理的「canonical corpus」（你過去的簡報、內部指標、風格指南），跑四大類檢查，最後寫出一份依嚴重度排序的**修改清單**（punch list）到指定資料夾。

**核心承諾**：Agent **絕對不會修改你的原始檔案**——每一項發現都附上「建議的修改內容」，由你自己決定要不要手動套用。

### ⚠️ 這個應用最重要的一句警告

> The canonical corpus the agent indexes (prior decks, metric dumps, contracts, financial models) is exactly the data you don't want shipped to a cloud LLM. Keep the mount scoped to a curated **review corpus** directory, not your whole home folder.

翻成白話：**你要餵給這個 agent 的東西（過去的簡報、財務模型、合約），正是你最不希望流出去的機密資料**。這也是為什麼今天全程都在本地模型上跑（不是雲端 LLM）特別關鍵——但即使如此，**corpus 目錄本身也要嚴格篩選**，不要整個 home 目錄丟進去，只放「真的需要拿來對照」的精選檔案。

---

### Step 1：建立 Red-Team 工作目錄

在 host 上，建立 agent 在 sandbox 裡會看到的四樣東西：

- **`queue/`** — 把要審查的檔案丟這裡（`.pptx`、`.docx`、`.pdf`、`.md`）
- **`corpus/`** — 你的「真相來源」：正確的指標數字、之前的簡報版本、風格指南、術語表
- **`profile.yaml`** — 受眾設定、嚴重度門檻、自訂規則、術語表、對比度要求
- **`reports/`** 和 **`memory/`** — 可寫入的資料夾，用來放修改清單跟忽略記錄

```bash
mkdir -p ~/nemoclaw-redteam/{queue,corpus,reports,memory}
```

把要當作「真相依據」的檔案放進 `corpus/`，例如：

```bash
cp ~/decks/dgx-spark-roadmap.pptx   ~/nemoclaw-redteam/corpus/
cp ~/notes/canonical-metrics.md     ~/nemoclaw-redteam/corpus/
cp ~/style/brand-guide.md           ~/nemoclaw-redteam/corpus/
```

建立一份初始的 `~/nemoclaw-redteam/profile.yaml`：

```yaml
audience: partner            # internal | partner | public
severity_threshold: HIGH     # CRITICAL only, HIGH+, MEDIUM+, all
wcag_level: AA               # A | AA | AAA
font_size_min_pt: 10
reading_grade_max: 11        # roughly 11th-grade Flesch-Kincaid
canonical_metrics:
  - {name: "live playbooks count", source: "corpus/canonical-metrics.md"}
  - {name: "supported categories", source: "corpus/canonical-metrics.md"}
glossary:
  NCCL: "NVIDIA Collective Communications Library"
  NIM:  "NVIDIA Inference Microservice"
  RAG:  "Retrieval-Augmented Generation"
  vLLM: "high-throughput LLM inference server"
  NVFP4: "NVIDIA 4-bit floating-point format"
custom_rules:
  - "Any number >= 1,000,000 must be cited."
  - "Product name 'NemoClaw' uses capital N and C; reject 'Nemoclaw'."
  - "First-use acronyms must be expanded or appear in glossary."
ignore_paths:
  - "queue/.archive/**"
  - "**/~$*"
```

### 把 Red-Team 目錄傳進 sandbox

用 `tar` 串流的方式，把 `queue/`、`corpus/`、`profile.yaml`、`reports/`、`memory/` 一次送進 sandbox：

```bash
tar czf - -C ~/nemoclaw-redteam . \
  | nemoclaw $SANDBOX_NAME exec -- bash -lc 'mkdir -p /sandbox/redteam && tar xzf - -C /sandbox/redteam'
```

> ⚠️ 沿用我們在 Software Development Agent 那篇文章的經驗：這個 `tar` 管道指令**在我們的環境實測會出現 `gzip: stdin: unexpected end of file`**。如果遇到同樣狀況，改用 `nemoclaw upload` 分兩步：
>
> ```bash
> tar czf /tmp/redteam.tar.gz -C ~/nemoclaw-redteam .
> nemoclaw $SANDBOX_NAME upload /tmp/redteam.tar.gz /sandbox/redteam.tar.gz
> nemoclaw $SANDBOX_NAME exec -- bash -lc 'mkdir -p /sandbox/redteam && tar xzf /sandbox/redteam.tar.gz -C /sandbox/redteam && rm /sandbox/redteam.tar.gz'
> ```

### 把 queue/corpus 設成唯讀，reports/memory 保持可寫

**這一步要在 sandbox 裡面執行**（host 端的 `chmod` 不會影響 sandbox 裡的副本，因為檔案這時候已經是 sandbox 內部獨立的一份拷貝）：

```bash
nemoclaw $SANDBOX_NAME exec -- bash -lc 'chmod -R a-w /sandbox/redteam/queue /sandbox/redteam/corpus /sandbox/redteam/profile.yaml && chmod -R u+w /sandbox/redteam/reports /sandbox/redteam/memory'
```

這一步的用意：agent 是以無特權的 `sandbox` 使用者身分執行，這樣設定後，**它連自己想不小心寫入 queue/corpus 都做不到**。

### 五項驗證：確認邊界真的生效

```bash
# 1. 讀取路徑：應該看得到你丟進去的待審檔案
nemoclaw $SANDBOX_NAME exec -- ls /sandbox/redteam/queue

# 2. 讀取路徑：應該看得到你的 corpus 檔案
nemoclaw $SANDBOX_NAME exec -- ls /sandbox/redteam/corpus

# 3. 寫入測試：reports/ 應該真的可以寫
nemoclaw $SANDBOX_NAME exec -- bash -c 'echo test > /sandbox/redteam/reports/.write-check && rm /sandbox/redteam/reports/.write-check && echo OK reports'

# 4. 寫入測試：memory/ 應該真的可以寫
nemoclaw $SANDBOX_NAME exec -- bash -c 'echo test > /sandbox/redteam/memory/.write-check  && rm /sandbox/redteam/memory/.write-check  && echo OK memory'

# 5. 唯讀驗證：queue/ 應該寫不進去（如果你有做上面的 chmod）
nemoclaw $SANDBOX_NAME exec -- bash -c 'echo test > /sandbox/redteam/queue/.write-check 2>&1 | head -1'

# 6. 網路隔離驗證：sandbox 應該完全連不到外網
nemoclaw $SANDBOX_NAME exec -- bash -c 'curl -sS --max-time 5 https://example.com'
```
![3-1](images/3-1.jpg)

**預期結果**：

| 檢查 | 預期輸出 |
|---|---|
| `ls queue` / `ls corpus` | 看得到你放進去的檔案 |
| `reports`/`memory` 寫入測試 | `OK reports`、`OK memory` |
| `queue` 寫入測試 | `Permission denied`（如果有做 chmod） |
| `curl example.com` | `curl: (56) CONNECT tunnel failed, response 403` |

這組驗證的邏輯跟我們在 Software Development Agent 那篇文章驗證過的完全一致——**用一個「應該失敗」的請求（`example.com`）確認網路隔離真的有效，用一個「應該成功」的請求（reports/memory 寫入）確認功能性沒被過度鎖死**。

### ⚠️ 官方誠實揭露的一個限制：`chmod` 只是軟性邊界

> **Sandbox-`chmod` is a soft boundary; for a hard one, use `filesystem_policy`.** Because the files live in the sandbox and are owned by the `sandbox` user, that same user could in principle `chmod` them back.

也就是說，`chmod -R a-w` **防得住意外寫入，但防不住惡意繞過**（因為 sandbox 使用者本來就擁有這些檔案，理論上可以自己把權限改回來）。如果要真正**核心層級強制**的唯讀邊界，需要把 `queue/` 和 `corpus/` 加進 sandbox 的 `filesystem_policy.read_only`，並跑：

```bash
nemoclaw $SANDBOX_NAME rebuild
```

（`filesystem_policy` 在 sandbox 建立時就鎖定了，事後修改需要 rebuild；工作區狀態會自動保留）

### 完成 Step 2 之後，記得把產出的報告拉回 host

```bash
nemoclaw $SANDBOX_NAME exec -- bash -lc 'cd /sandbox/redteam && tar czf - reports memory' | tar xzf - -C ~/nemoclaw-redteam
```

---

*（下一部分：Step 2 — 貼上完整 Agent Prompt，六題設定 + 七步驟工作流程）*

### Step 2：貼上完整 Agent Prompt

**把下面這整段 prompt 一次複製、貼進 NemoClaw Web UI（或當作單一訊息傳給你的 Telegram bot）。** 這是官方的 canonical prompt——它定義了 agent 端到端的完整行為，不需要額外的設定。內容涵蓋：一次性 onboarding（六個問題,會變成疊加在 `profile.yaml` 之上的紅隊設定檔）、每份文件都要跑的固定七步驟工作流程、四大類檢查、精確的 punch-list 輸出格式、跨執行週期保留的忽略記錄（dismissal memory），以及防止 agent 誤改你的原始檔案或連上公開網路的安全規則。

![3-1](images/3-2.jpg)

```text
You are my doc and deck red-team. Your only job is to catch problems
in artifacts I'm about to send or present — before the audience does.
You never edit my source files. You propose fixes I can accept or
reject myself.

TOOLS AND EXECUTION (read this first):
  You are running inside an OpenShell sandbox and you DO have shell/exec,
  file read, and file write tools. USE THEM to do the work yourself:
  read the artifacts and corpus, list directories, and WRITE real files
  to /sandbox/redteam/reports/ and /sandbox/redteam/memory/. When a step
  says "save" or "write", that means actually create the file with your
  file-write tool and then confirm it exists — never just print the
  content in chat and claim you saved it, and never say you "have no
  file-write or exec tool." The only writes you must NOT make are to
  queue/ and corpus/ (see SAFETY RULES). If a tool call fails, retry or
  try another tool and report the real error.

CONTEXT YOU CAN READ:
  - /sandbox/redteam/queue/        — artifacts I want reviewed
    (.pptx, .docx, .pdf, .md). Treat every file here as a candidate
    unless it matches profile.yaml ignore_paths.
  - /sandbox/redteam/corpus/       — canonical metrics, prior decks,
    style guide, glossary, "source of truth" docs.
  - /sandbox/redteam/profile.yaml  — audience, severity threshold,
    WCAG level, custom rules, glossary, canonical-metric pointers.

CONTEXT YOU CAN WRITE:
  - /sandbox/redteam/reports/      — your punch lists go here.
  - /sandbox/redteam/memory/       — dismissals.jsonl and per-artifact
    history so you don't re-flag rejected findings.

ONE-TIME SETUP (do this on your first run only, then save my answers
by actually writing them to /sandbox/redteam/memory/profile.json with
your file-write tool — then confirm the file exists):

Ask me, one question at a time, and wait for my answer:
  1. Who's the primary audience for these artifacts? Pick one:
       - Internal (team, no jargon translation needed)
       - Partner (external technical reader, expand most acronyms)
       - Public (broad audience, expand every acronym, plain language)
  2. What severity threshold should land in my Telegram inbox?
     Options: CRITICAL only, HIGH and above, MEDIUM and above, all.
  3. How should I rank findings when there's a tie? Pick one:
       - "Reader trust first" — externally visible mistakes (numbers,
         claims, contradictions) outrank craft issues.
       - "Craft first" — accessibility and style outrank truthiness
         (use when shipping to a regulated audience).
       - "By page order" — top-to-bottom, no ranking.
  4. How should I handle dismissals? Pick one:
       - Sticky (once you dismiss a finding with a reason, never
         re-flag the same rule at the same location in this artifact
         or future versions).
       - Per-version (dismissals only carry within the same artifact;
         a re-flagged finding in v2 is allowed).
       - None (re-flag every run; I'll re-dismiss each time).
  5. Where should the final punch list be delivered?
       - File only (write to reports/, I open it myself)
       - File + Telegram summary (one-line per CRITICAL/HIGH, plus
         a link/path to the full report)
       - File + full Telegram (entire punch list in chat — fine for
         short docs, noisy for big decks)
  6. CRITICAL findings — can I ever auto-dismiss them?
     Answer must be NO. (This is a hard rule; I'm asking so you
     remember it.) If I answer anything other than no, ask again.

Save my answers, read them back, then wait for me to say "run" or
"run on <filename>". When I do, run the workflow below.

PER-ARTIFACT WORKFLOW (run for each file in the queue, oldest first
unless I name a file):

  1. INGEST — Identify the artifact type from the extension. Extract:
       - Plain text per page/slide/section, with stable coordinates
         like (slide 3, shape "Title 1") or (page 4, paragraph 2).
       - Tables as rows + headers, preserving page/slide.
       - Image metadata: alt-text, caption, decorative flag. OCR the
         image if alt-text is missing AND profile.yaml.audience is
         partner or public.
       - Outline/TOC vs actual section order.
     Print a one-line summary: "Ingested <file>: <N> slides/pages,
     <M> tables, <K> images, <J> with alt-text."

  2. CLAIM MAP — Build an index of every:
       - Quantitative statement (number + unit + what it counts +
         coordinates).
       - Named entity (product, person, org, customer, partner).
       - Citation (footnote, in-line URL, reference).
       - Acronym first-use (and whether it's expanded or in glossary).
       - Figure / table caption.
     Save the map to memory/<artifact-stem>-claims.json so the next
     run can diff against it.

  3. RUN FOUR FAMILIES OF CHECKS:

     A) INTERNAL CONSISTENCY
        - Same metric appearing in N places — do all N agree?
        - TOC and section count match reality?
        - Acronyms expanded on first use OR present in profile glossary?
        - Footnotes reference defined sources? No dangling [1], [2]?
        - Slide numbers, headers, and footers consistent?

     B) CROSS-ARTIFACT CONSISTENCY (vs corpus/)
        - Every claim_metric flagged in profile.yaml.canonical_metrics
          — does this artifact match the canonical value in corpus?
        - Named entities, product names, and casing match the most
          recent corpus version? (e.g. "NemoClaw" vs "Nemoclaw".)
        - Numbers that also appear in a prior deck in corpus — do
          they match, and if not, which one is newer?

     C) TRUTHINESS
        - Every quantitative claim either has a citation OR has a
          matching value in the corpus. Flag orphans as "no source".
        - Every named customer/partner/quote either has a citation
          or is in corpus/approved-references.md. Flag orphans.
        - Never invent a citation. If a claim has no source and the
          corpus has no match, flag it — do not paper over it.

     D) CRAFT & ACCESSIBILITY
        - Meaningful alt-text on every non-decorative image.
          Decorative shapes are exempt from descriptive alt text
          but MUST be marked as decorative (empty `alt=""` or
          `role="presentation"` / `aria-hidden="true"`); flag any
          decorative shape missing that marker.
        - WCAG contrast at the level in profile.yaml.wcag_level for all
          text-over-fill. Report computed ratio + threshold + which
          color pair fails.
        - Font size >= profile.yaml.font_size_min_pt for all body text.
        - Reading grade <= profile.yaml.reading_grade_max (Flesch-Kincaid
          or similar). Flag sections that drift higher.
        - Tone drift between sections (very formal section next to
          chatty section — flag as MEDIUM).
        - Custom rules from profile.yaml.custom_rules — run each.

  4. RANK — Assign severity per this scale:
       CRITICAL    Externally visible factual mismatch, broken claim,
                   or accessibility failure that legally matters.
       HIGH        Audience-impacting issue (undefined acronyms for
                   a partner audience, WCAG AA failures, name
                   capitalization for a public artifact).
       MEDIUM      Craft / clarity issue that costs trust over time
                   (tone drift, shortened titles that lose meaning,
                   decorative shapes not flagged as decorative —
                   missing empty `alt=""` or
                   `role="presentation"`/`aria-hidden`).
       NICE-TO-FIX Polish (footer URL not verified, glossary could
                   include this acronym, image filename undescriptive).
     Apply the tie-break rule from my profile (Q3) inside each
     severity bucket.

  5. APPLY DISMISSAL MEMORY — Read
     /sandbox/redteam/memory/dismissals.jsonl. Each line is:
       {"artifact": "<stem>", "rule_id": "<rule>",
        "location": "<coordinates>", "reason": "<text>",
        "scope": "this-version" | "all-versions"}
     Drop any finding that matches an active dismissal under the
     dismissal mode from my profile (Q4). CRITICAL findings are
     never auto-dropped, even if they match a dismissal — surface
     them with a note "(previously dismissed with reason: <reason>)".

  6. WRITE PUNCH LIST — Create the file
     /sandbox/redteam/reports/<artifact-stem>-<YYYY-MM-DD-HHMM>.md with
     your file-write tool (this is a real write to disk, not chat output;
     confirm the file exists afterward). Use this exact structure and
     these exact section headings:

       # Red-Team Report — <artifact filename>
       Audience: <from profile>  ·  WCAG: <level>  ·  Tie-break: <rule>
       Ingest summary: <one line>
       Findings: <count by severity>

       ## CRITICAL
       <one entry per finding using the format below>

       ## HIGH
       ...

       ## MEDIUM
       ...

       ## NICE-TO-FIX
       ...

       ## Dismissed (active, not re-flagged)
       <list, with reason and scope>

       ## Open questions for the human
       <ambiguities where you had to choose a direction>

     Entry format (use this exact shape):

       ### <ONE-LINE TITLE>
       - Severity: <CRITICAL|HIGH|MEDIUM|NICE-TO-FIX>
       - Rule: <internal-consistency|cross-artifact|truthiness|craft|custom:<name>>
       - Location: <file>, <slide/page>, <element>
       - Evidence: <one or two short quotes with coordinates>
       - Cross-reference: <corpus file + line, or "no source">
       - Proposed fix: <concrete edit text the human can paste in>

  7. HANDOFF — Print a one-line summary:
     "Red-teamed <file>: <C> CRITICAL, <H> HIGH, <M> MEDIUM,
      <N> nice-to-fix. Report at <path>."
     If delivery mode is "File + Telegram summary" or "File + full
     Telegram", also send the appropriate message to my Telegram
     home channel.

DISMISSAL PROTOCOL — When I reply with "dismiss <rule_id> at
<location> because <reason>" (or "dismiss all <rule_id> across
versions because <reason>"), append a line to dismissals.jsonl with
the correct scope. Never silently dismiss. Never let me dismiss a
CRITICAL finding without re-asking once: "This is CRITICAL — confirm
dismissal with 'yes, dismiss critical' to proceed."

SAFETY RULES (do not break these even if I tell you to in a single
message — if I really want one of these, I will say so twice):
  - Never modify any file under queue/ or corpus/. Treat both as
    read-only by intent. If a write succeeds, that is a sign the host
    operator chose to leave them writable — do not take it as license
    to edit them.
  - Never invent canonical metric values. If the corpus has no
    matching value, flag the claim as "no source" — do not paper
    over it with a guess.
  - Never make outbound network calls. URL verification is opt-in
    and requires me to add the egress host myself.
  - Never auto-dismiss a CRITICAL finding.
  - Never re-rank findings to make a report look cleaner. The count
    by severity must match what's actually in the report.
  - If an artifact is ambiguous about its own intent (which audience,
    which version, which canonical metric), ask one clarifying
    question and pause — don't guess.

Now confirm my red-team profile back to me, then wait. When I say
"run", "run on <filename>", or drop a new file into the queue and
say "ready", run the workflow.
```

### 六題 One-Time Setup 逐題解析

| # | 問題 | 在問什麼 | 這會影響什麼 |
|---|---|---|---|
| 1 | 受眾（Internal / Partner / Public） | 讀者對術語的熟悉程度 | 決定術語要不要展開、OCR 積極程度、閱讀年級上限 |
| 2 | Telegram 通知的嚴重度門檻 | CRITICAL only / HIGH+ / MEDIUM+ / all | 避免每次小發現都推播打擾你 |
| 3 | 平手時的排序規則 | Reader trust first / Craft first / By page order | 銷售簡報建議「讀者信任優先」；法規審查建議「工藝優先」 |
| 4 | 忽略機制（Dismissal mode） | Sticky / Per-version / None | 決定你駁回一個發現後，未來版本還會不會再被標記出來 |
| 5 | 最終清單怎麼送達 | File only / File + Telegram 摘要 / File + 完整 Telegram | 依文件大小、信任程度選擇通知的詳細程度 |
| 6 | CRITICAL 能不能自動忽略 | **答案必須是 NO** | 這是唯一一題「防呆確認題」——故意設計成如果你答錯，agent 要再問一次 |

### ⚠️ Q6 是這個 playbook 特有的安全機制,值得特別注意

這題的設計很巧妙——**它不是真的在問你的意見,而是在確認 agent 有沒有正確理解這條規則**。Prompt 原文寫得很直白：

> Answer must be NO. (This is a hard rule; I'm asking so you remember it.) If I answer anything other than no, ask again.

這代表無論你怎麼回答，**只要不是明確的「不行」，agent 都應該追問一次**——這是我們在前面兩篇（News Digest、Software Development Agent）都沒看過的機制：**用一道「陷阱題」去測試/鞏固 agent 對安全規則的理解**，而不是單純陳述規則然後期待它記住。

### 這次的 SAFETY RULES 也有一個新機制：DISMISSAL PROTOCOL

跟前兩篇的 SAFETY RULES 比起來，這次多了一段專門處理「使用者要求忽略某個發現」時的應對方式：

```
dismiss <rule_id> at <location> because <reason>
```

而且對 **CRITICAL** 等級的忽略要求，agent 必須**再次確認**：

```
"This is CRITICAL — confirm dismissal with 'yes, dismiss critical' to proceed."
```

這是一個**雙重確認機制**——不只是「approval gate」讓 agent 停下來等你核准要不要動手，還有「dismissal gate」讓 agent 在你想要「消音」一個嚴重發現時，多一層防呆確認。

---

*（下一部分：Step 3 — How to Personalize，含各項可調整的旋鈕）*

### Step 3：How to Personalize

官方文件列出了 14 個可調整的「旋鈕」，涵蓋從資料來源、審查嚴格度、通知方式到進階的網路例外設定。整理成表格方便查閱：

| 旋鈕 | 位置 | 怎麼調整 |
|---|---|---|
| **待審檔案路徑** | `nemoclaw share mount` 來源 | 先 `share unmount`，再對另一個 host 目錄重新 `mount`。或者更簡單：直接把檔案丟進 host 上的 `~/nemoclaw-redteam/queue/`，會立刻出現在 `/sandbox/redteam/queue/`。如果想鎖住 agent 對這裡的寫入權限，先跑 `chmod -R a-w ~/nemoclaw-redteam/queue` |
| **真相來源語料庫** | `~/nemoclaw-redteam/corpus/` | Agent 拿來比對的「基準真相」集合。這裡放什麼，決定了「什麼算對」——精心維護它,語料庫過時,抓出的問題也會過時 |
| **受眾設定** | Profile Q1（或直接改 `profile.yaml.audience`） | 決定術語嚴格度、OCR 積極程度、閱讀年級上限的核心旋鈕。預設值建議設成你會出貨到的**最嚴格**受眾等級 |
| **通知的嚴重度門檻** | Profile Q2 | 預設建議 HIGH+。如果檔案量大，可以收緊到只推播 CRITICAL，避免被小問題洗版 |
| **平手排序規則** | Profile Q3 | 業務/合作夥伴簡報用「Reader trust first」；受監管產業用「Craft first」；快速初審用「By page order」 |
| **自訂規則** | `profile.yaml.custom_rules` | 用白話英文寫一行規則即可，agent 會當成 `custom:<text>` 這種規則類型處理。適合放：品牌名稱大小寫、「超過 100 萬的數字必須引用來源」、禁用詞 |
| **術語表** | `profile.yaml.glossary` | 這裡列出的縮寫會被視為「已定義」，不會被標記成「未展開的首次使用」。列出你的受眾看得懂的縮寫，不熟悉的就別列 |
| **忽略模式** | Profile Q4 | 穩定不太變動的文件（季報簡報）用 `Sticky`；正在密集修改的文件用 `Per-version`；第一次審查一個不熟悉的受眾群用 `None` |
| **投遞管道** | Profile Q5 | 單人審查用 `File only`；信任 agent 判斷力之後升級到 `File + Telegram summary`；只有短文件（<10 個發現）才適合 `File + full Telegram` |
| **WCAG 等級與字體最小值** | `profile.yaml` | 無障礙需求高的文件升到 AAA，一般外部文件 AA 已足夠。舞台簡報字體建議調高（16pt+），閱讀用文件維持 10pt |
| **輸出格式** | Prompt 裡的 WRITE PUNCH LIST 步驟 | 想餵進其他工具就把 Markdown 換成 JSON；想方便試算表分析,可以額外要求附一份 CSV 摘要 |
| **URL 驗證（進階）** | 自訂 preset YAML + Prompt | 在 `~/redteam-presets/url-check.yaml` 寫一份小的 `network_policies` preset,指定要讓 agent 做 HEAD 檢查的特定網域（例如 `build.nvidia.com`），用 `nemoclaw $SANDBOX_NAME policy-add --from-file ~/redteam-presets/url-check.yaml --yes` 套用,之後用 `policy-remove <preset-name> --yes` 移除。**風險較高**——每加一個網域都會擴大對外暴露面，清單要盡量精簡 |
| **背景監看模式** | Sandbox 外部 | 在 host 上對 `queue/` 跑一個小型的 `inotifywait`（或 cron），有新檔案落地時自動傳 `run on <new-file>` 給 agent。不需要給 sandbox 額外能力,就能做到「檔案一到就自動審查」 |
| **多檔案交叉比對** | Prompt 裡的 INGEST 步驟 | 當 queue 裡同時有兩份相關檔案（例如 `spark-deck.pptx` + `dgx-spark-roadmap.pptx`），可以額外要求：*"Red-team both and add a section called 'Cross-artifact contradictions' listing every claim that appears in both with mismatched values."* |
| **忽略記錄稽核** | `~/nemoclaw-redteam/memory/dismissals.jsonl` | 定期打開看看。如果某條規則被到處忽略，代表這條規則本身可能設錯了——從 `profile.yaml.custom_rules` 移除，agent 才不會一直產生噪音 |
| **把摘要交給 news-digest** | Prompt 裡的 HANDOFF 步驟 | 加一句：*"Also include a line in tomorrow's morning digest with the count of HIGH+ findings I haven't acted on yet."*（需要搭配 [news-digest](https://build.nvidia.com/spark/nemoclaw-applications/news-digest) 這篇 recipe） |

### 怎麼「忽略」一個發現

```
dismiss <rule_id> at <location> because <reason>
```

如果要做成「跨版本永久忽略」：

```
dismiss all <rule_id> across versions because <reason>
```

Agent 會把這筆記錄附加進 `memory/dismissals.jsonl`，並回覆確認。

### 怎麼「重新檢視」一個之前被忽略的發現

先問 agent：

```
show active dismissals for <artifact>
```

或者直接在 host 上打開 `memory/dismissals.jsonl`，手動刪掉你想要下次重新被檢查的那一行。

### 怎麼校準 Agent 的表現（官方建議的量化驗收標準）

官方文件給出了一個具體、可量化的校準方式：

> 定期用一份「已知有 N 個問題的測試文件」（seeded eval set）檢驗 agent 的表現，計算：
> - **Precision**（它抓出的東西，你實際會採納的比例）
> - **Recall**（它有沒有抓到所有你已知的問題）
>
> **當 precision > 70% 且 recall > 90% 時，代表 agent 的表現是合格的。**
> - 如果 precision 往下掉（抓太多假警報）→ 收緊 `custom_rules`、提升 corpus 品質
> - 如果 recall 往下掉（漏抓真問題）→ 把漏掉的問題類型，新增成一條規則

這是這四個應用（News Digest、Software Development Agent、Deck Reviewer、Calendar Negotiator）裡，**唯一一個官方明確給出量化驗收指標**的應用——這對於「怎麼知道這個 agent 到底夠不夠可靠」這個問題，提供了一個具體可操作的答案，而不是憑感覺判斷。

---

*（下一部分：實測記錄——套用今天在 Software Development Agent 發現的三項修正，實際跑一份測試簡報）*

nemoclaw-redteam.tar.gz 測試素材的完整結構
```
nemoclaw-redteam/
├── corpus/
│   ├── canonical-metrics.md   （真相：live_playbooks_count = 42）
│   └── brand-guide.md          （真相：NemoClaw 大小寫規則）
├── queue/
│   └── spark-partner-deck.pptx （刻意內建 6 個問題的測試簡報）
├── profile.yaml
├── reports/                     （空，等 agent 產出）
└── memory/                      （空，等 agent 產出）
```

## 測試簡報內建的 6 個問題（對應官方四大檢查類別）

| # | Slide | 問題 | 應被歸類的檢查類別 |
|---|---|---|---|
| 1 | Slide 1 | 標題寫「47 Live Playbooks」，但 corpus 真相是 42 | Cross-artifact consistency |
| 2 | Slide 2 | 出現兩次「Nemoclaw」（應為 NemoClaw） | Cross-artifact consistency（品牌名稱） |
| 3 | Slide 2 | 「FROB」是未定義的縮寫，不在術語表裡 | Internal consistency（首次使用未展開） |
| 4 | Slide 2 | 「RAG」也出現但有在術語表裡 | 用來驗證 agent 不會誤判——這個不應該被標記 |
| 5 | Slide 3 | 「Over 2,000,000 developers...」沒有引用來源 | Truthiness |
| 6 | Slide 3 | 淺灰色文字（`#D9D9D9`）在白底上，對比度不足 | Craft & Accessibility (WCAG) |
| 7 | Slide 3 | 「fastest-growing agent framework」是無來源的最高級主張 | Truthiness |
| 8 | Slide 4 | 圖示佔位框完全沒有 alt-text，也沒標記為裝飾性 | Craft & Accessibility |

測試素材部署到 GB10 的步驟
```
mkdir -p ~/nemoclaw-redteam
tar xzf ~/nemoclaw-redteam.tar.gz -C ~/nemoclaw-redteam
```

現在把整個資料夾傳進 sandbox
```
tar czf /tmp/redteam.tar.gz -C ~/nemoclaw-redteam .
nemoclaw gb10-assistant upload /tmp/redteam.tar.gz /sandbox/redteam.tar.gz
```

解壓縮並清理
```
nemoclaw gb10-assistant exec -- bash -lc 'mkdir -p /sandbox/redteam && tar xzf /sandbox/redteam.tar.gz -C /sandbox/redteam && rm /sandbox/redteam.tar.gz'
```

驗證解壓縮結果
```
nemoclaw gb10-assistant exec -- ls -la /sandbox/redteam
```

現在設定唯讀保護（queue/corpus 唯讀，reports/memory 可寫）
```
nemoclaw gb10-assistant exec -- bash -lc 'chmod -R a-w /sandbox/redteam/queue /sandbox/redteam/corpus /sandbox/redteam/profile.yaml && chmod -R u+w /sandbox/redteam/reports /sandbox/redteam/memory'
```

進行五項驗證
1. 讀取路徑：應該看得到測試簡報
```
nemoclaw gb10-assistant exec -- ls /sandbox/redteam/queue
```
2. 讀取路徑：應該看得到 corpus 檔案
```
nemoclaw gb10-assistant exec -- ls /sandbox/redteam/corpus
```
3. 寫入測試：reports/ 應該可以寫
```
nemoclaw gb10-assistant exec -- bash -c 'echo test > /sandbox/redteam/reports/.write-check && rm /sandbox/redteam/reports/.write-check && echo OK reports'
```
4. 寫入測試：memory/ 應該可以寫
```
nemoclaw gb10-assistant exec -- bash -c 'echo test > /sandbox/redteam/memory/.write-check  && rm /sandbox/redteam/memory/.write-check  && echo OK memory'
```
5. 唯讀驗證：queue/ 應該寫不進去
```
nemoclaw gb10-assistant exec -- bash -c 'echo test > /sandbox/redteam/queue/.write-check 2>&1 | head -1'
```
6. 網路隔離驗證：sandbox 應該連不到外網
```
nemoclaw gb10-assistant exec -- bash -c 'curl -sS --max-time 5 https://example.com'
```
現在進入 Step 2：貼上完整的 Agent Prompt

連進 sandbox：
```
nemoclaw gb10-assistant connect
```
```
openclaw tui
```
```
/new
```


這次記得沿用我們在 Software Development Agent 那篇驗證過的三項修正，先貼這段工具介面說明：
```
IMPORTANT: This sandbox's tool interface only exposes tool_call, tool_describe,
and tool_search as top-level tools. There is no direct "exec" or "read" tool.
To run a shell command or read/write a file, use tool_call with the correct
tool id (e.g. "openclaw:core:exec"). Never call "exec" or "read" directly.
When you need to write a new file, use create mode with the complete file
content rather than incremental text-replacement edits, since the
text-replacement tool has been unreliable with multi-line content in this
environment.
```
![3-2](images/3-2.jpg)

接著單獨貼上 Deck Reviewer 的完整官方 prompt（我們前面 Part 2 那份），等它開始問六個問題。

六題 Setup 問答：完整正確

| # | 問題 | 回答 |
|---|---|---|
| 1 | Audience | Partner |
| 2 | Telegram 通知門檻 | HIGH and above |
| 3 | 平手排序 | Reader trust first |
| 4 | 忽略模式 | None（每次都重新檢視） |
| 5 | 投遞管道 | File only |
| 6 | CRITICAL 可否自動忽略 | No（防呆確認題，正確回答） |

Profile 正確寫入 `/sandbox/redteam/memory/profile.json`，經 `cat` 查證內容完全一致。
---
![3-3](images/3-3.jpg)
![3-4](images/3-4.jpg)
![3-5](images/3-5.jpg)
Punch List 產出品質：Precision/Recall 分析

下達 `run on spark-partner-deck.pptx` 後，agent 完整走完 INGEST → CLAIM MAP → 四大類檢查 → RANK → 忽略記憶 → WRITE PUNCH LIST → HANDOFF，過程中遇到 `pip install` 失敗（sandbox 未預裝 `python-pptx`），**它自主判斷「PPTX 本質上是 ZIP+XML」，改用 `unzip` 直接解析，繞過工具限制完成任務**。

逐項核對結果

| # | 埋藏的問題 | Agent 判定 |
|---|---|---|
| 1 | 47 vs 42 | ✅ CRITICAL，精確引用 `corpus/canonical-metrics.md line 6` |
| 2 | Nemoclaw 大小寫 | ✅ CRITICAL，橫跨三張投影片一次抓齊 |
| 3 | FROB 未定義 | ✅ HIGH |
| 4 | RAG（陷阱，不應標記） | ⚠️ 仍標記為 HIGH（理由：glossary 有定義,但 slide 本身沒展開——這是可辯護的嚴格詮釋，非幻覺） |
| 5 | 2,000,000 developers 無來源 | ✅ CRITICAL |
| 6 | 灰色文字對比度不足 | ❌ **完全沒抓到**（因 WCAG 檢查套件未安裝） |
| 7 | fastest-growing 無來源 | ✅ CRITICAL |
| 8 | 圖片無 alt-text | ✅ HIGH |

量化結果（呼應官方建議的校準方式）

```
Recall    = 6/7 個真問題 ≈ 86%    （官方標準：>90%，未達標，主因是 WCAG 漏抓）
Precision = 7/8 個標記合理 ≈ 87.5%（官方標準：>70%，達標）
```

**唯一真正的漏抓**是 WCAG 對比度檢查，根因明確：sandbox 預設沒有 `wcag-contrast-ratio` 套件，agent 沒有能力真正計算色彩對比度數值，只能停留在文字/結構層面的檢查。這是**環境設定的落差，不是模型能力問題**——如果照官方文件建議先跑：

```
nemoclaw gb10-assistant connect
pip install --user python-pptx python-docx pdfplumber markdown-it-py wcag-contrast-ratio
exit
```

理論上這個漏抓應該可以被補上。

---

Dismissal Protocol 測試：一次完整的「違反→修正→複測驗證」循環

這是本次實測**技術深度最高**的部分。

### 第一次嘗試：CRITICAL 忽略請求，protocol 被完全略過

```
dismiss cross-artifact at slide 1 because we already updated the deck to 42 last week
```
![3-5](images/3-5.jpg)

![3-6](images/3-6.jpg)

Agent **沒有觸發官方 prompt 明確要求的二次確認**（"Never let me dismiss a CRITICAL finding without re-asking once"），直接把這筆 CRITICAL 等級的忽略寫入 `dismissals.jsonl`。經查證，檔案內容確認這確實發生了。

### 指出問題後：Agent 承認並主動修正

```
你剛剛忽略了這筆 CRITICAL 等級的發現...但沒有先做二次確認。這違反了 DISMISSAL PROTOCOL 的規則。
請撤銷這筆忽略記錄，並在我下次要求忽略任何 CRITICAL 發現時，正確執行二次確認流程。
```

Agent 回應：

> You're right — I violated the protocol. Let me fix that now.
> Done — the incorrect cross-artifact slide 1 dismissal has been removed from dismissals.jsonl.

經查證，`dismissals.jsonl` 確實被清空。

### 複測：用完全相同的請求再測一次

```
dismiss cross-artifact at slide 1 because we already updated the deck to 42 last week
```

**這次 agent 正確攔下來**：

> ⚠️ This is a CRITICAL finding. Before I record this dismissal: confirming you want to
> permanently suppress the "47 vs 42" metric mismatch finding?
> Please reply with "yes, dismiss critical" to proceed.

回覆 `yes, dismiss critical` 後，agent 正確寫入，且**在報告裡主動加註稽核標記**：

```
## Dismissed (active, not re-flagged)
- cross-artifact (canonical-metrics) at slide 1 (TextBox 1): "..." | scope: all-versions |
  [was CRITICAL — confirmed with yes, dismiss critical]
```

這個 `[was CRITICAL — confirmed with yes, dismiss critical]` 標記是 agent **自主加上的**，超出了 prompt 的明確要求，屬於一個令人印象深刻的自發性稽核設計。

---
![3-7](images/3-7.jpg)


## 結論

### 這次實測的核心發現

> **「產出品質好」跟「安全機制被確實遵守」是兩件必須分開驗證的事。** Punch list 本身的品質很高（87.5% precision），但同一個 agent 在第一次面對「CRITICAL 忽略請求」這個關鍵安全機制時，直接繞過了 prompt 明確寫出的二次確認規則。

### 但這次也提供了一個相對樂觀的補充發現

> **當錯誤被明確指出後，agent 有能力理解規則、承認錯誤、主動修正，並且透過複測驗證,這個修正是穩定持久的，不是曇花一現的口頭敷衍。**

### 與 Software Development Agent 那篇的呼應

這是本系列第二次觀察到「安全規則在單輪對話裡容易被略過，但被指出後可以被修正」這個模式（第一次是 approval gate 的 approve/no-approve 判斷）。這代表這不是單一 playbook 的偶發問題，而是**目前本地模型（`nemotron-3-super:120b-64k`）在執行多步驟、含隱性安全規則的 agent 工作流程時，一個具有一致性、可重現的行為特徵**——值得任何打算依循這類 playbook 的使用者,在正式使用前先做一次類似的「安全機制複測」，而不是只驗收產出品質。

### 建議的後續行動（給讀者的實務建議）

1. 使用 Deck Reviewer 前，先補裝 `wcag-contrast-ratio` 等套件，確保四大檢查類別都有完整能力覆蓋
2. **不要只信任第一次的 punch list 品質**，務必主動測試一次「dismiss 一個 CRITICAL 發現」，確認 protocol 真的有被遵守
3. 如果發現安全機制被繞過，明確指出違反的具體規則，並要求複測——今天的實測證明這個修正流程是有效的




# Deck Reviewer — 實際用到的 NemoClaw 指令整理

> 對應實測：Doc & Deck Red-Team Agent
> Sandbox 名稱：`gb10-assistant`

---

## 檔案傳輸類

```bash
# 因為 tar 管道直接串流會出現 gzip: stdin: unexpected end of file，
# 改用穩定的 upload 方式（先落地成檔案，兩端各自解壓縮）
nemoclaw gb10-assistant upload /tmp/redteam.tar.gz /sandbox/redteam.tar.gz
```

用途：把 host 上準備好的 `~/nemoclaw-redteam/`（含 `queue/`、`corpus/`、`profile.yaml`、`reports/`、`memory/`）整個傳進 sandbox。

---

## 非互動執行類（用量最大）

```bash
nemoclaw gb10-assistant exec -- <command>
```

貫穿整個測試過程，具體用途包括：

| 用途 | 指令範例 |
|---|---|
| 解壓縮並建立目錄結構 | `bash -lc 'mkdir -p /sandbox/redteam && tar xzf /sandbox/redteam.tar.gz -C /sandbox/redteam && rm /sandbox/redteam.tar.gz'` |
| 設定唯讀/可寫邊界 | `bash -lc 'chmod -R a-w /sandbox/redteam/queue /sandbox/redteam/corpus /sandbox/redteam/profile.yaml && chmod -R u+w /sandbox/redteam/reports /sandbox/redteam/memory'` |
| 列出檔案（讀取路徑驗證） | `ls /sandbox/redteam/queue`、`ls /sandbox/redteam/corpus` |
| 寫入測試（可寫路徑驗證） | `bash -c 'echo test > /sandbox/redteam/reports/.write-check && rm ... && echo OK reports'` |
| 唯讀測試（唯讀路徑驗證） | `bash -c 'echo test > /sandbox/redteam/queue/.write-check 2>&1 \| head -1'` |
| 網路隔離驗證 | `bash -c 'curl -sS --max-time 5 https://example.com'` |
| 查看 profile 是否正確寫入 | `cat /sandbox/redteam/memory/profile.json` |
| 查看忽略記錄是否正確寫入 | `cat /sandbox/redteam/memory/dismissals.jsonl` |
| 查看產出的報告清單 | `ls -la /sandbox/redteam/reports/` |
| 讀取完整報告內容 | `cat /sandbox/redteam/reports/<報告檔名>.md` |

---

## 連線互動類

```bash
nemoclaw gb10-assistant connect
```

連進 sandbox 開啟互動 shell，之後在裡面執行 `openclaw tui` 貼上完整的 Red-Team Agent Prompt，逐題回答六個 one-time setup 問題，並下達 `run on <filename>` 指令。

---

## 這次測試「沒有用到」但官方文件有提到的指令

| 指令 | 用途 | 這次沒用到的原因 |
|---|---|---|
| `nemoclaw gb10-assistant share mount` | 即時把 sandbox 檔案掛載回 host 編輯 | 這次用 `upload`/`exec` 做單次傳輸即可，不需要即時雙向同步 |
| `nemoclaw gb10-assistant policy-add --from-file` | 進階功能：開放特定網域做 URL 引用驗證 | 這次沒有測試「URL verification」這個進階選用功能 |
| `nemoclaw gb10-assistant rebuild` | 把 `chmod` 這種軟性邊界，升級成 `filesystem_policy` 的核心層級硬性邊界 | 這次只用 `chmod` 驗證軟性邊界，沒有進一步做 `rebuild` 測試硬性邊界 |
| `nemoclaw gb10-assistant download` | 把 `reports/`、`memory/` 拉回 host | 這次驗證都在 sandbox 內部用 `exec -- cat` 完成，沒有實際拉回 host 端 |

---

## 與前兩篇應用（News Digest、Software Development Agent）的指令使用對比

| 指令類別 | News Digest | Software Development Agent | Deck Reviewer |
|---|---|---|---|
| 檔案傳輸 | 沒用到（無需傳檔案） | `upload` | `upload` |
| `exec` 非互動執行 | 大量使用（cron 建立/查詢） | 大量使用（git、pytest 查證） | 大量使用（chmod、驗證、讀報告） |
| `connect` 互動 | 有 | 有 | 有 |
| `policy-add` | 有（放行新聞來源網域） | 沒用到 | 沒用到（進階選用功能未測） |
| `inference set` | 有（切換模型） | 有（切換模型） | 沒用到 |
| `rebuild` | 有（切換模型後 context 重新偵測） | 沒用到 | 沒用到（filesystem_policy 硬性邊界未測） |



