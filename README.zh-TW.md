# plugins-zh-tw

[回到 README.md](./README.md) · [English](./README.en.md)

一套供 Claude Code 與其他相容 Agent-Skills 標準 harness 使用的 agent skills、plugins、slash commands 與行為集合。

## 目錄結構

Skills 放在 `skills/<bucket>/<skill-name>/SKILL.md` 下。Bucket 分類：

- `engineering/`：日常程式碼工作
- `productivity/`：日常非程式碼工作流程工具
- `misc/`：保留但很少用，不列入推廣
- `personal/`：綁定個人環境設定，不列入推廣
- `in-progress/`：草稿，尚未準備好發布
- `deprecated/`：已不再使用

`engineering/` 與 `productivity/` 的 promoted skills 會登錄在這份 README 與 `.claude-plugin/plugin.json`。

## 本機開發

```bash
scripts/link-skills.sh
```

把 `skills/` 底下每個 skill symlink 到 `~/.Codex/skills` 與 `~/.agents/skills`，供本機測試。新增、移除或重新命名 skill 後要重新執行。

## User-invoked skills

- [`/git-fork-remotes`](./skills/engineering/git-fork-remotes/SKILL.md)：Fork repo 後設定 fork/upstream remote，並修正 branch tracking。
- [`/translating-skill-docs`](./skills/productivity/translating-skill-docs/SKILL.md)：為每個 skill 加上翻譯後的 `SKILL.<locale>.md` sibling file，不改變 runtime 行為。
- [`/writing-router-skill`](./skills/engineering/writing-router-skill/SKILL.md)：判斷目前情境適合使用哪個 skill。

## Model-invoked skills

- [`/hv-analysis`](./skills/productivity/hv-analysis/SKILL.md)：執行橫縱分析法並產出 PDF 報告。
- [`/storage-analyzer`](./skills/productivity/storage-analyzer/SKILL.md)：唯讀分析 macOS/Windows 儲存空間並產出互動式 HTML 報告。
- [`/neat-freak`](./skills/engineering/neat-freak/SKILL.md)：整理專案文件、規則檔、agent memory 與 workspace 殘留物。

## 既有 plugin 與 prompt

- `prompt/`：獨立 prompt 收藏，收錄從各 skill / plugin repo 匯出的 standalone prompt 文字檔。
- `skills/base/`、`skills/code-review/`、`skills/implement/`、`skills/wait-what/`：原有的 root-level skills，保留原始文件與 `zh-TW` 翻譯。
