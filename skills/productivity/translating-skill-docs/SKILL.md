---
name: translating-skill-docs
description: 為目標 repo 中的每個 skill，依指定的地區代碼與書面語體，新增一份翻譯後的 SKILL.<locale>.md 對應檔案，且不改變執行期行為。
disable-model-invocation: true
---

# 翻譯 Skill 文件

## 輸入

在寫任何一個檔案之前，以下兩件事都必須先確定：

- **地區代碼（locale）** — 目標代碼（`zh-TW`、`ja-JP`、`pt-BR`……）。
- **書面語體（register）** — 該地區代碼的標準*書面*形式，以及會顯示偏移到方言或口語的標記詞。範例：標準書面 `zh-TW` 排除粵語口語標記詞 `嘅 喺 唔 咗 呢個 嗰 佢哋`——單靠地區代碼並不能保證這點，因為它固定的是地區與文字系統，不是語體，而粵語口語同樣是用繁體字書寫的。不要臆測這件事——要問清楚，或從人類夥伴指定的風格中推導出來。

## 哪些要翻譯／哪些保持原樣

| 翻譯 | 保持原樣 |
| --- | --- |
| `SKILL.md` 本文的敘述文字 | `name:` frontmatter（逐位元組相同——它是呼叫用的 key） |
| `description:` frontmatter | 程式碼區塊、指令、檔案路徑、URL |
| `SKILL.md` 透過 context pointer 指向的每個參考檔案 | `CLAUDE.md`、`RELEASE-NOTES.md` |
| `README.md` | plugin manifest（`.claude-plugin/plugin.json` 等）——它們指向資料夾而非檔案，所以新增的 `.<locale>.md` 對應檔案永遠不會破壞它們 |

## 步驟

1. **釘死地區代碼與書面語體。** 在寫任何東西之前，明確講出地區代碼和它的禁用標記詞清單。完成標準：兩者都被明確指名，或者人類夥伴對你不確定的地區代碼確認「沒有已知的偏移風險」。
2. **只記錄一次決策。** 檢查 `docs/adr/` 是否已有關於雙語並存的 ADR（對應檔翻譯 vs. 取代英文內容）。如果沒有，就寫一份：對應檔案、原始 `SKILL.md` 永不編輯、`name:` 永不翻譯，讓上游合併保持無衝突。完成標準：恰好一份 ADR 涵蓋這個決策——不要為第二個地區代碼再寫一份。
3. **翻譯每一個 skill。** 對每一份 `skills/*/SKILL.md`，在旁邊建立 `SKILL.<locale>.md`：翻譯本文和 `description:` 的值，逐字複製 `name:`。完成標準：`skills/` 底下每個目錄都有一份 `SKILL.<locale>.md`，且對每一個都執行 `diff <(grep '^name:' SKILL.md) <(grep '^name:' SKILL.<locale>.md)` 結果為空。
4. **翻譯每個 skill 所指向的內容。** `SKILL.md` 揭露指向的任何參考檔案（指向 `references/` 或類似目錄的 context pointer）都採用同樣的對應檔處理：`<file>.<locale>.md`。完成標準：從已翻譯 skill 可觸及的每個已揭露參考檔案都有一份翻譯對應檔。
5. **翻譯 README，然後連結它。** 建立 `README.<locale>.md` 作為完整翻譯。在 `README.md` 開頭加一行連結到它。完成標準：連結可正確解析，且 `README.<locale>.md` 存在。
6. **書面語體檢查——用指令跑，不要用肉眼看。** 對每個新增的 `.<locale>.md` 檔案 grep 步驟 1 釘死的禁用標記詞。必須零命中。任何地方命中都代表要重新翻譯整份檔案，而不是修補那一句——語體偏移通常是整份檔案層級的，不是單行層級的。

   ```bash
   grep -nE '嘅|喺|唔|咗|呢個|嗰|佢哋' -- $(git ls-files '*.zh-TW.md')
   ```

   （把這個模式和檔案 glob 換成步驟 1 釘死的地區代碼／書面語體——
   拿掉任何在標準書面文字裡也會出現的標記詞，例如 `關係` 裡單獨的 `係`）
7. **其餘保持原樣。** 確認 diff 中 `CLAUDE.md`、`RELEASE-NOTES.md` 和 plugin manifest 都未被改動——它們刻意排除在範圍之外（見該份 ADR）。

## 完成的定義

每一份 `skills/*/SKILL.md` 及其指向的每個參考檔案都有一份 `<locale>` 對應檔，`README.<locale>.md` 存在且從 `README.md` 連結過去，步驟 6 的書面語體 grep 沒有任何回傳結果，且 `git diff --stat` 顯示 `name:` 欄位、`CLAUDE.md`、`RELEASE-NOTES.md` 或 plugin manifest 都沒有變動。
