---
name: writing-router-skill
description: 詢問這個 repo 裡哪個 skill 適合你目前的情況——整個 skill 集合的路由器。
disable-model-invocation: true
---

# Skill 路由器

你不會記住這個 repo 裡每個 skill，所以就問吧。

## Engineering — 日常程式碼工作

- **`/git-fork-remotes`** — fork 一個 repo 之後，設定好 `origin`/`upstream`，並修正每個 local branch 的 tracking。User-invoked，要用就打它的名字。
- **`neat-freak`** — 收尾知識與治理債務：讓 `CLAUDE.md`/`AGENTS.md`、authorized agent memory、workspace 殘留物對齊程式碼與 runtime 的現況，下一個 session 或下一個人才能從同一個答案開始。Model-invoked，agent 偵測到知識落後的訊號會自己接手，也能直接提「neat-freak」、「洁癖」（觸發詞照 `SKILL.md` 原文，簡體）或打 `/neat` 觸發。

## Productivity — 日常非程式碼工作流程

- **`/translating-skill-docs`** — 幫一個 repo 裡每個 skill 加上 `SKILL.<locale>.md` 翻譯版本，不動 runtime 行為。User-invoked。
- **`hv-analysis`** — 橫縱分析法：系統性研究一個產品、公司、概念、技術或人物。縱軸追歷史脈絡、橫軸比同期競品，交叉兩軸產出洞察，最後生成排版精美的 PDF 研究報告。Model-invoked，說「研究一下」、「深度研究」、「竞品分析」（觸發詞照 `SKILL.md` 原文，簡體）之類的話會觸發，也能直接打 `/hv-analysis`。
- **`storage-analyzer`** — 唯讀掃描 macOS/Windows 磁碟空間，把占用大戶按風險分成可自動清理／需人工判斷／謹慎清理三級，生成可一鍵清理的互動式 HTML 報告。Model-invoked，說「存储分析」、「磁盘满了」、「清理空间」（觸發詞照 `SKILL.md` 原文，簡體）會觸發。

## 保留但不推廣

`skills/misc/` 與 `skills/personal/` 底下還有幾個 skill（例如 `aihot`、`khazix-writer`），能用但沒進 `engineering/`／`productivity/` 這兩個推廣 bucket，也沒登錄在頂層 README 或 `.claude-plugin/plugin.json`。要找就直接看該 bucket 自己的 `README.md`——`in-progress/`（草稿）和 `deprecated/`（已停用）同理。

## 撰寫或維護 skill 本身

要新增、修改，或審視某個 skill 寫得好不好，走 `writing-great-skills`（全域 skill，不在這個 repo 裡）——它是 in-skill step、reference、leading word 這套寫作原則的 single source of truth。把一個 skill 推進 `engineering/` 或 `productivity/`（或搬出去、改名），`CLAUDE.md` 訂的登錄點要一起動：頂層 `README.md`、`.claude-plugin/plugin.json`、該 bucket 自己的 `README.md`，還有 `docs/<bucket>/<name>.md`——照 `.agents/writing-docs.md` 的樣板寫。
