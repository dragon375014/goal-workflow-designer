# goal-workflow-designer（繁體中文說明）

> 兩支 Claude Code skill，會「反問你」直到把你腦中模糊的任務，變成一份**精確、能直接執行的指令** —— 一份深度的 `/goal` prompt，或一支廣度的 workflow。它是**幫你想清楚要交代什麼的設計助手，不是幫你做事的引擎**。

[English README →](README.md) ·  [![授權 MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

---

## 先講最重要的一句：它不是 AI，是「幫你下指令的教練」

很多人一看到 `/goal`、看到 workflow，以為要再裝一個新的 AI。**不是。**

- 它**不取代** Claude Code 本來就有的 `/goal` 跟 workflow 功能。
- 你**不裝它也能用** `/goal`。

那它做什麼？它解決一個你一定遇過的問題 ——

> **難的從來不是「叫 AI 去做」，難的是「把你要什麼講清楚」。**

你心裡知道想要什麼，但寫成指令時總是寫得太模糊（「幫我弄好一點」「順一點」「專業一點」）。AI 拿到模糊指令，就會自己亂猜一個「完成」的定義，然後三分鐘就交差了事。

這兩支 skill 就是站在你前面的**教練**：它會一題一題反問你，逼你把「好一點」講成「具體要長怎樣、怎麼算過關」，最後吐出一份精準的指令，你再貼給 AI 去跑。

---

## 它有兩支，對應兩種情況：一件事 vs 很多件事

同一個模糊任務，其實有兩種「做法」。這個工具一邊各給你一支 skill：

| Skill | 它幫你塑形成… | 什麼時候用 |
|---|---|---|
| **`goal`（深度）** | 一份精準的 `/goal` 指令 —— 像**一位師傅照著一張評分表，反覆磨同一件事到夠好** | 你只有**一件事**，但要它做到位、會自己越改越好 |
| **`workflow-shaper`（廣度）** | 一支能跑的 workflow —— 像**一個工頭發組織圖，叫一整組人同時開工，再交叉檢查** | **同一個檢查/同一種改動，要套到很多件**彼此獨立的事情上 |

### 一秒判斷該用哪個：數一數「有幾件」

- **一件**事要磨到完美 → 用 `goal`（一個師傅 + 評分表）
- **很多件**要套同一個檢查 → 用 `workflow-shaper`（工頭 + 一整組人）
- **很多件不一樣的事、而且有先後依賴**（A 做完 B 才能開始）→ 那是「規格分解」，見下方姊妹 repo **spec-sonar**（建築師畫施工順序圖）

> 用咖啡來比喻：
> - 把**一支配方**調到對味 → 師傅 + 評分表（深度）
> - **一整批 30 支生豆**都要照同一張表驗一遍 → 工頭叫一整組人（廣度）

而且兩個可以疊起來用：workflow 是那個「重複很多次」的迴圈，goal 是迴圈裡每個人實際在做的那件事。

---

## 為什麼需要它？（白話版）

2026 年，Claude Code、OpenAI Codex、Manus Agent 幾乎同一時間推出了 `/goal`；Claude Code 後來又加了 workflow（背景同時派很多個 AI 分身做事）。它們都會「自己一直做，直到達標」。

但 Anthropic 自己的研究發現一個現象：**當 AI 的記憶體（context）快被塞滿時，它會焦慮、會抄捷徑草草收尾 —— 哪怕事情還沒做完。** 這叫 context anxiety（脈絡焦慮）。

對付這個的唯一方法，就是**一開始就把「怎樣算做完」定義得很清楚**。

- 指令模糊（「把這個專案弄好一點」）→ AI 自己選一個「好」的定義，三分鐘就停。
- 指令精準（「把結帳頁回應時間壓到 0.2 秒以下，用速度測試工具驗證，過程不准動其他模組」）→ AI 會乖乖磨上好幾個小時。

問題是 —— **寫出這種精準指令本身就很難**。這兩支 skill 就是把「逼你想清楚」這件事自動化，而且它不會放你用模糊答案蒙混過去。

> 順帶一提：表面上你是在「幫 AI 寫評分表」，**真正的價值是它逼你把原本只活在你腦中、說不出口的那套標準，第一次變成白紙黑字。**

---

## 裝了有什麼好處？

- ✅ **不再「AI 做一半就停」** —— 它幫你寫出讓 AI 願意跑好幾個小時的指令。
- ✅ **不再「跑完發現不是我要的」** —— 反問階段就把標準對齊，省下重跑的時間跟額度。
- ✅ **主觀的事也能交給 AI** —— 寫文案、做設計這種「說不清楚但看得出來好壞」的活，它會帶你做一張**評分表**，AI 才有東西可以照著評。
- ✅ **它會誠實擋你** —— 不是每件事都該開 workflow（很燒額度）。`workflow-shaper` 會先過一關「值不值得」的閘門，不值得就直接叫你別開，幫你省錢。
- ✅ **記憶體快滿了也能接力** —— session 快爆時，它會把進度打包，讓你開新對話無痛接續（見下方「接力機制」）。
- ✅ **全程中英對照** —— 每個術語第一次出現都標 `英文（中文）`，看不懂英文也不卡。
- ✅ **裝一次，所有專案通用** —— 它是裝在你帳號層級（`~/.claude/`），不綁特定專案。

---

## 怎麼安裝

### 方法 A —— 讓 AI 幫你自動裝（推薦，最省事）

打開你的 Claude Code，把下面這段**整段貼進去**，AI 就會自己把兩支 skill 裝到對的位置：

```
請讀 https://raw.githubusercontent.com/dragon375014/goal-workflow-designer/main/INSTALL.md

然後照它的步驟，把 goal + workflow-shaper 兩支 skill 裝到我帳號層級的 ~/.claude/skills/。
寫任何檔案前先給我看 diff 或摘要，如果我已經改過某個 SKILL.md，先問我再覆蓋。
```

它會自動抓檔、放到 `~/.claude/skills/` 底下、並把兩個觸發設定加進你的 `~/.claude/CLAUDE.md`。

### 方法 B —— 手動裝

```bash
git clone https://github.com/dragon375014/goal-workflow-designer.git /tmp/gwd
cp -r /tmp/gwd/skills/goal           ~/.claude/skills/goal
cp -r /tmp/gwd/skills/workflow-shaper ~/.claude/skills/workflow-shaper
```

（`~` 就是你的家目錄：Windows 是 `C:\Users\你的帳號`、Mac/Linux 是 `/home/你的帳號`。）

然後把觸發設定（[INSTALL.md](INSTALL.md) 第 3 步那兩段）貼進你的 `~/.claude/CLAUDE.md`。裝完後，你設計出來的 goal 會存在 `~/.claude/goals/<名字>/`。

> ⏫ 從舊名 `skill-to-goal`（還只有一支 skill 時的 repo 名）升級上來？repo 已改名並重整結構：goal skill 從 repo 根目錄搬進 `skills/goal/`，還新增了 `workflow-shaper`。重跑一次方法 A，或 `git pull` 後重新複製一遍即可。

---

## 怎麼使用

### `goal`（深度 —— 一件事磨到好）

```
/goal <一句話描述你的任務>     # 從頭開始引導問答
/goal --use <名字>            # 載入之前存好的 goal
/goal --resume <草稿名>       # 接續一個被中斷的問答
```

它會帶你走過五個元素：**完成定義 / 怎麼驗證 / 不能動什麼 / 每輪記錄什麼 / 什麼時候該停下來回報**；如果是主觀類任務（設計、寫作、行銷），還會多帶你做一張評分表。最後產出一份能讓 AI 持續磨好幾小時、而不是三分鐘就放棄的指令。

用中文講也會觸發，例如「幫我寫個 goal prompt」。

> ⚠️ 故意不支援空的 `/goal`（不給描述）—— 模糊的開頭只會問出模糊的問題，所以至少給它一句話。

### `workflow-shaper`（廣度 —— 很多件同時做）

描述一個「很多件要套同一個檢查」的任務，或直接說「用 workflow 做」。它會**先過一關「值不值得開」的閘門**（5 個條件 → 紅黃綠燈），再決定要不要動：

- 🟢 **綠燈** → 推薦一個並行的形狀，產出可以直接貼上去跑的 `workflow ...` 指令。
- 🟡 **黃燈** → 給你一條中間路（拆成幾支小 workflow / 先跑一件試水溫），不逼你二選一。
- 🔴 **紅燈**（其實只有一件事、需要中途人工確認、需要把中間結果留著追問）→ **直接拒絕產出 workflow 指令**，把你導回普通做法或 `/goal`，幫你省下亂燒的額度。

它**只幫你塑形、不會自己去跑** —— 產出交回給你，你決定要不要貼上去執行。

---

## `/goal` skill 和一般 prompt skill 有什麼不同？

Claude Code 圈子裡已經有幾支做 prompt 設計的 skill（[prompt-architect](https://github.com/ckelsoe/prompt-architect)、[prompt-improver](https://github.com/ndpvt-web/prompt-improver)、[prompt-master](https://github.com/nidhinjs/prompt-master)），它們走的是通用路線。這一支則是死咬著 `/goal` 的執行模型在做：

| 功能 | 本 repo | 一般 prompt skill |
|---|---|---|
| 對準 `/goal` 的五元素框架 | ✅ | ❌（用 CO-STAR、RISEN 這類通用框架）|
| 幫主觀任務建一張評分表 | ✅（6 步 SOP）| ❌ |
| 記憶體（context）健康度閘門 | ✅（Phase 0a）| ❌ |
| **「適不適合做成 goal」檢查（三燈號）** | ✅（Phase 0c）| ❌ |
| KICK-OFF 自我提示（AI 真的會開工，不用你再打「continue」）| ✅ | ❌ |
| 記憶體吃緊時的接力打包 | ✅（`--resume`）| ❌ |
| 用隔離的 subagent 掃描程式碼庫 | ✅ | ❌ |
| **多件事任務有廣度搭檔** | ✅（`workflow-shaper`）| ❌ |
| 術語中英對照（EN + 中文） | ✅ | 不一定 |

---

## 設計原則

這九條寫死在 skill 裡，每一輪問答都照著走：

1. **模糊答案一定追問。**「好一點／順一點／專業一點」→ 必須追到一個可以衡量的標準，放過模糊答案就是失職。
2. **聰明跳過。** 每輪先掃描已經收集到的答案，已知的元素不重複問。
3. **每輪只問 1–3 題。** 一次丟一長串題目，人就放棄了。
4. **舉例幫你起頭** —— 但會提醒你別照抄。
5. **主觀任務一定要有評分表。** 沒有評分表，負責審核的 AI 就沒有東西可以評。
6. **記憶體健康度最優先**（Phase 0a 守門）。
7. **不是每件事都適合 `/goal`**（Phase 0c）—— 也**不是每件事都值得開 workflow**（`workflow-shaper` 的「值不值得」閘門）。兩邊都寧可拒絕，也不送出一場注定失敗的執行。
8. **KICK-OFF 是自我提示。** 印出 kick-off 區塊後，skill 會在同一回合直接呼叫工具開工 —— 光靠文字不一定能讓 AI 真的動起來。
9. **術語中英對照。** 每個術語第一次出現都寫成 `English（中文）`。

---

## 接力機制（Handoff）

一個很實用的小設計（細節見 [docs/handoff-pattern.md](docs/handoff-pattern.md)）：當你的對話記憶體已經快滿，這時硬把問答做完、再去跑 `/goal`，只會讓品質一起下滑。

所以這支 skill 會在記憶體吃緊時，**把目前進度打包成一個檔案**，叫你 `/clear` 清空、再用 `/goal --resume <草稿名>` 在一個全新、記憶體滿格的對話裡接續 —— 真正要跑的那一段，就會落在最清醒的狀態下。

---

## 生態系（Ecosystem）

本 repo 是 [dragon375014](https://github.com/dragon375014) **五個公開 repo 的 AI 開發工具鏈**裡的**「塑形層」**（shaping layer）。完整地圖、分流規則、各 skill 的歸屬 → [**ECOSYSTEM.md**](https://github.com/dragon375014/specmit/blob/main/ECOSYSTEM.md)

**一個指令裝整套**（五個工具一次裝到對的位置）：
```bash
npx specmit init
```

姊妹 repo：[spec-sonar](https://github.com/dragon375014/spec-sonar) · [specmit](https://github.com/dragon375014/specmit) · [claude-skills-governance-meta](https://github.com/dragon375014/claude-skills-governance-meta) · [agent-work-board](https://github.com/dragon375014/agent-work-board) —— 每個都能獨立使用，要哪個裝哪個。

---

## 姊妹 repo

### spec-sonar —— 規格側

這個 repo 是**「執行側」**（一件事磨深、或很多件平行跑）。[`spec-sonar`](https://github.com/dragon375014/spec-sonar) 是**「規格側」**（要做的到底是什麼）：`idea-to-spec` 用暗區問答把模糊的產品想法收斂成規格，`goal-decomposer` 再把規格編譯成**帶依賴順序的 goal 圖**——它產出的 `G*.md` 用的正是本 repo `/goal` 的**同一套五元素格式**，可以直接用 `/goal` 式的 KICK-OFF 執行。

| 你要的是 | 用 | 在哪個 repo |
|---|---|---|
| 把完整產品想法收斂成規格 | `idea-to-spec` | spec-sonar |
| 把規格拆成帶依賴順序的執行單元 | `goal-decomposer` | spec-sonar |
| 把一件事磨成可迭代的 goal prompt | `/goal` | 本 repo |
| 同一個檢查跑 N 件獨立的事 | `workflow-shaper` | 本 repo |

完整鏈路：`idea-to-spec → goal-decomposer → goals/G*.md → 每個用 /goal 式 KICK-OFF 執行（深度），同型子任務用 workflow-shaper 扇出（廣度）`。

### claude-skills-governance-meta —— 結構治理

這個 repo 管的是**「要做什麼」**（任務怎麼講清楚）。治理姊妹 repo 管的是**「做出來的東西結構別爛掉」**：

> ### [`claude-skills-governance-meta`](https://github.com/dragon375014/claude-skills-governance-meta)
> 一套防守 + 進攻的治理機制：架構完整性閘門（在你「我要做 X」還沒動手前就先攔下來檢查）、跨層 trace-lock（把「資料從寫入到顯示」的鏈路鎖死，避免「改了 A 忘了 B」）、還有可以實際跑的防禦型檢查器。

**心智模型：這個 repo = 把任務講清楚的「需求單」；那個 repo = 不讓成果走樣的「護欄」。**

只裝一個也完全沒問題，兩個各自能獨立運作。但如果你是一個獨立開發者、在一個真實專案上跑很長的 AI 對話 —— 這一對剛好覆蓋了「把工作塑形好」跟「別讓工作把結構搞爛」兩件事。

---

## 素材來源

這套方法論的出處：Anthropic 關於 **context anxiety（脈絡焦慮）**與「實作者／審核者」架構的研究、Anthropic **frontend-design** skill 的案例（博物館實驗 → 6 步評分表 SOP）、Karpathy 的「評估比 prompt 工程更重要」、加上社群的共同體會 ——「`/goal` 的成敗完全取決於規格有多精準」。廣度那一半則參考 Claude Code 的 **Dynamic Workflows**（背景多 agent 編排）。原始對談的逐字稿在 [examples/source-transcript.md](examples/source-transcript.md)。

---

## 貢獻

歡迎開 PR 跟 issue。幾個原則：

- **不要往五元素框架裡加新元素。** 它的穩定是刻意的，想加請先開 issue 討論。
- **不要把 `workflow-shaper` 的「值不值得」閘門放寬成自動放行。** 它存在的意義就是擋掉不划算的任務。
- **歡迎往 `examples/` 加範例輸出。**
- **歡迎翻譯**（目前有英文／繁中）。
- **送 PR 前先測過** —— 在自己的機器上把 skill 完整跑一遍。

---

## 授權

[MIT](LICENSE)。隨你怎麼用都行，標註出處感激但不強制。

---

## 相關專案

- [Claude Code skills 官方文件](https://code.claude.com/docs/en/skills)
- [Claude Code Dynamic Workflows](https://code.claude.com/docs/en/workflows)
- [awesome-claude-skills](https://github.com/travisvn/awesome-claude-skills)
- [prompt-architect](https://github.com/ckelsoe/prompt-architect) —— 通用型的 prompt 結構器

---

如果這兩支 skill 幫你省下了一次 debug、或一次白跑的 `/goal`，請我喝杯咖啡不強求但很開心。這裡全部 MIT、無條件免費使用。

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/Z8C420A0VI)
