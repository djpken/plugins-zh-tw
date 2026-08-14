# Claude / Codex 路徑、載入與規模速查

平台機制會改變。先探測目前的環境與本機規則;涉及寫入或大小上限時,優先核對目前的官方文件或本機工具輸出,不要把這張表當成永遠不變的事實。

## 通用原則

- 區分三類檔案:人工維護的規則、Agent 自動記憶、機器產生的歷史/索引。它們不能套用同一套寫入規則。
- `MEMORY.md` 只是檔名,不代表跨平台的語意相同。大小門檻必須綁定平台與檔案類型。
- 規則的真身可能是 CLAUDE.md、AGENTS.md、override、匯入或符號連結;以目前工作區的宣告與實際載入鏈為準。
- 發現多個平台目錄不代表每個平台都在使用中。只稽核目前運行的平台,以及使用者明確納入的安裝面。

## Claude Code

| 用途 | 常見路徑 / 規則 |
|---|---|
| 使用者指令 | `~/.claude/CLAUDE.md` |
| 專案指令 | `./CLAUDE.md`、`./.claude/CLAUDE.md`、`CLAUDE.local.md` |
| 路徑規則 | `.claude/rules/**/*.md` |
| 自動記憶 | `~/.claude/projects/<project>/memory/` |
| 自動記憶索引 | 上述目錄的 `MEMORY.md` |
| Skills | `~/.claude/skills/<name>/SKILL.md` 或專案的 `.claude/skills/` |

目前的官方口徑:

- CLAUDE.md 會全量載入,但建議目標少於約 200 行;越長越消耗注意力,也越降低遵守度。這是品質預算,不是硬性截斷線。
- Claude 自動記憶的 `MEMORY.md` 在工作階段啟動時只載入前 200 行或 25KB(先到者為準);主題檔案則按需讀取。這個硬性限制只屬於 Claude 自動記憶,不適用於 Codex 產生的記憶。
- Claude 原生讀取 CLAUDE.md。已有 AGENTS.md 的專案可以用匯入或符號連結同源;方向由專案規則決定,不要擅自反轉。

## OpenAI Codex

| 用途 | 常見路徑 / 規則 |
|---|---|
| Codex home | `$CODEX_HOME`,預設為 `~/.codex` |
| 全域指令 | `$CODEX_HOME/AGENTS.override.md`,不存在時讀 `AGENTS.md` |
| 專案指令 | 從專案根目錄到目前目錄逐層尋找 `AGENTS.override.md`、`AGENTS.md`、設定的 fallback |
| 全域 Skills | `$CODEX_HOME/skills/<name>/SKILL.md` |
| 專案 Skills | 專案的 `.codex/skills/<name>/`(以目前 Codex 版本與環境為準) |

目前的官方口徑:專案指令鏈合併後預設最多 32KiB,由 `project_doc_max_bytes` 控制;越靠近目前目錄的指令越晚載入。要檢查 override 與 fallback,不能只找根目錄的 AGENTS.md。

某些 Codex 環境還提供 `~/.codex/memories/`、rollout summaries 或 Chronicle 衍生索引。這類檔案可能由宿主管線產生:

- 先讀目前環境給出的 memory instructions;沒有明確授權時只做唯讀。
- 不要直接修改產生的 `MEMORY.md`、`memory_summary.md`、`raw_memories.md` 或 rollout summary。
- 使用者明確要求更新記憶且環境允許時,只使用該 Codex 環境規定的 correction input,或透過官方的 `/memories`、設定與 `memories.*` 設定項來控制產生與使用,再等待宿主 consolidation;不要自訂檔案大小目標、compact candidate,或專案層級的產生記憶門禁。

發現 `TEAM_GUIDE.md`、`.agents.md` 等檔案時,只有當它們出現在 Codex 的 fallback 設定裡,才把它們當成指令檔。

## 其他 Agent Skills 平台(Qoder、Kimi Code、iFlow、CodeBuddy、Cursor、Gemini CLI 等)

Agent Skills 是開放標準(2025 年 12 月由 Anthropic 開放),已有約 40 個產品相容本 skill 的發布格式。Claude Code 與 Codex 以外的平台不逐一維護速查表,改用通用探測法:

1. **規則檔**:在專案根目錄與上層目錄找 `AGENTS.md`(跨平台的事實標準)、`CLAUDE.md`,以及平台專屬的形態(例如 `.cursor/rules/`、`.cursorrules`、平台設定裡的專案指令)。哪一份實際被載入,以目前平台的文件與診斷入口為準,不要用猜的。
2. **三分法歸類**:把發現的每個知識檔案歸入三類之一——人工維護的規則、Agent 自動記憶、機器產生的歷史/索引。歸類不明時按機器產生處理(最保守)。
3. **記憶邊界**:未知平台的記憶機制找不到官方控制面時,預設當作唯讀;不要把其他平台的大小門檻或寫入規則套用過來。
4. **降級用法**:宿主不支援 Agent Skills 時,本 skill 仍可使用——把 SKILL.md 全文當成規則檔或對話指令交給 Agent,references 內容再按需跟進;執行邊界不變。

## 共存檢查

1. 列出實際存在的平台目錄與 skill 的 realpath。
2. 核對同名 skill 是否符號連結到同一個真身、被複製安裝,或被更高優先權的版本覆蓋。
3. 只修改權威真身;複製安裝需要明確的同步機制,不能假設它會自動更新。
4. 符號連結在 Windows 或受限環境可能無法使用,允許專案採用匯入或產生鏡像,只要現場規則明確且有一致性門禁。
5. 驗證載入狀態,而不是只驗證檔案存在:使用平台提供的 instruction/skill list、`/memory`、status 或等效的診斷入口。

## 官方複核入口

- Agent Skills specification: <https://agentskills.io/specification>
- Agent Skills 相容產品名錄: <https://agentskills.io>(showcase)
- Claude Code memory and CLAUDE.md: <https://code.claude.com/docs/en/memory>
- OpenAI Codex AGENTS.md: <https://developers.openai.com/codex/guides/agents-md/>
- OpenAI Codex Memories: <https://learn.chatgpt.com/docs/customization/memories>
