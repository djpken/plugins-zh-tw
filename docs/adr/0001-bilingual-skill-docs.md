# ADR-0001：skill 文件曾採用雙語並存

- 狀態：已取代
- 日期：2026-08-14

本決策已由 [ADR-0003：root-level skill 以中文 `SKILL.md` 為唯一 canonical 文件](./0003-canonical-skill-docs-zh-tw.md) 取代。以下內容保留當時的背景與決策，供追溯歷史。

## 當時的適用範圍

本 ADR 適用於原本已存在的 root-level skills：`base`、`code-review`、`implement` 與 `wait-what`。它們以英文 `SKILL.md` 加上 `SKILL.zh-TW.md` sibling file 的方式維護。

從 `/Users/kunkun/Projects/skills` 遷入的 bucket skills 已依來源 repo 的決策直接維護為台灣正體中文 `SKILL.md`，不另外建立 sibling file。完整背景記錄在 `.agents/adr/0001-skill-docs-zh-tw-translation.md`。

## 當時的決策

翻譯文件採用同層 sibling file，來源 `SKILL.md` 保持不變，新增 `SKILL.<locale>.md`。每個翻譯檔的 `name:` 維持來源內容，`description:` 依目標語言翻譯。

README 當時也採用同層語言檔；英文 `README.md` 保留原內容，並在檔案頂端連結 `README.zh-TW.md`。後續 README 入口改由 `README.en.md` 與 `README.zh-TW.md` 分工，這不影響本 ADR 已被 ADR-0003 取代的狀態。

## 當時的原因

這種配置不會改變 skill 的執行期行為，也能讓上游合併持續套用到英文來源檔。翻譯內容與來源內容分開管理，降低後續同步時的衝突。

## 歷史影響

- 上游更新先套用到來源檔，再同步翻譯檔。
- `name:` 不翻譯，避免改變 skill 的 invocation key。
- 每個 locale 只建立一份雙語並存決策 ADR。
