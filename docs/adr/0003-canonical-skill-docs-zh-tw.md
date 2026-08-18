# ADR-0003：root-level skill 以中文 `SKILL.md` 為唯一 canonical 文件

- 狀態：接受
- 日期：2026-08-18

## 背景

root-level skills 原本以英文 `SKILL.md` 加上 `SKILL.zh-TW.md` sibling file
維護。這會讓同一個 skill 有兩份需要同步的內容，也讓接手者必須先判斷哪一份
才是主要文件。遷入的 bucket skills 已直接使用台灣正體中文 `SKILL.md`，兩種
文件策略不一致。

## 決策

- root-level skill 的 `SKILL.md` 是唯一 canonical 文件，內容以台灣正體中文為主。
- 移除 root-level skill 的 `SKILL.zh-TW.md` sibling file，不再建立新的 sibling。
- `name`、slash command、API、class、function、package、protocol、schema 欄位與
 其他 runtime literals 保留原文，避免改變 skill contract。
- `README.md`、`README.zh-TW.md` 與 `README.en.md` 可以依讀者維持各自語言；這項決策
 只約束 skill runtime 文件，不取消 README 的雙語入口。
- `translating-skill-docs` 仍可服務其他 repo 或明確要求翻譯 sibling 的工作，不改變本
  repo root-level skill 的 canonical 文件政策。

## 原因

單一中文 canonical 文件會降低同步成本，讓 runtime 讀取的檔案與人類閱讀的主要
說明一致，也符合這個 repo 的台灣正體中文定位。保留 technical literals 可以維持
slash command 與 runtime contract 的相容性。

## 後果

- `base`、`code-review`、`code-review-ocr`、`implement` 與 `wait-what` 只保留
  `SKILL.md`。
- 新增或修改 root-level skill 時，直接編輯中文 `SKILL.md`；不得新增
  `SKILL.zh-TW.md`。
- ADR-0001 僅作為歷史記錄，現行規則以本 ADR 與 `AGENTS.md` 為準。
