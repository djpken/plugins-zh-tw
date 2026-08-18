---
name: code-review
description: "沿 Standards 與 Spec 軸 review `/base` 到 `HEAD` 的範圍。"
---

只 review Git 範圍。固定起點來自 `.scratch/base`；不要要求會移動的
`HEAD~N` ref，也不要用 workspace 或單一 commit review 執行這個 skill。
這是穩定的雙軸 review；OCR workflow 由 `/code-review-ocr` 提供。

## 前置條件

1. 必須存在且非空的 `.scratch/base`。不要建立它，也不要自動呼叫 `/base`。
   Manifest 建立由 `/base` 負責，嚴格格式驗證由這個 review 負責。
2. 讀取本次 review 所需的固定 `base_sha`、`source` 與 source value。
3. 在 review 開始時讀取一次 task source。外部 `source/ref` 透過 host
   integration 讀取；`source: user` 使用非空的 `summary`。當
   `source: user` 明確有空的 `summary` 時，延後 task source 建立，直到第
   5 步擷取 commit list 後，再從 commit subject 建立。外部 source 讀取失敗
   時，不得使用 cache 或摘要替代。只把有效 payload 保留在記憶體中，不要
   寫入 repo state、commit 或其他 log。
4. 如果無法讀取 task source、內容無法序列化，或超過可用 context，停止並
   回傳 `needs_human`，不要開始任一 review 軸。

## 範圍 review

1. 從 `.scratch/base:base_sha` 解析 `from`，並將 `to` 解析為目前 `HEAD`。
   呼叫者明確提供目前 head 的 commit ref 時，才使用該 ref。開始 review
   前，將兩者轉成完整 commit SHA。
2. 要求 `from` 是 `to` 的 ancestor。範圍無效時停止 review。
3. 自動排除 `.scratch/**`，不納入 review 輸入。
4. 排除上述檔案後若範圍為空，停止並回傳 `needs_human`，不要開始任一
   review 軸。
5. 只擷取一次 `git diff from...to` 與一次
   `git log from..to --oneline`。將產生的 diff command 與 commit list 原樣
   傳給兩個 review 軸。

## Task source

從 `.scratch/base` 載入的 task source，或在空 summary 情況下從擷取的
commit subject 建立的 task source，是 Spec 的輸入。不要搜尋其他 issue、
要求第二個固定起點，或用摘要替換完整的 external payload。

External source 必須把完整 provider payload 保留在記憶體中：

```text
<task-source>
source: github
ref: https://github.com/org/repo/issues/123

content:
<complete provider payload>
</task-source>
```

User source 使用已保存的 summary：

```text
<task-source>
source: user
summary: <one-line summary>
</task-source>
```

User source 明確使用空 summary 時，保持 manifest 不變，從擷取的
`git log from..to --oneline` 每行 subject 部分在記憶體中建立 task source，
並保留原本順序：

```text
<task-source>
source: user
summary:
commit_subjects:
<subject line 1>
<subject line 2>
...
</task-source>
```

這個 fallback 只會在固定範圍解析完成後讀取 commit subject，不會寫回
`.scratch/base`。

## Standards source

Repo 中記錄程式碼寫法的文件，例如 `CODING_STANDARDS.md` 或
`CONTRIBUTING.md`，都屬於 standards source。

除了 repo 文件規範之外，Standards 軸固定使用下列來自 _Refactoring_ 第 3
章的 Fowler code smell baseline。遵循兩項規則：

- Repo 規範優先。文件明確支持某種做法時，即使 baseline 會標記，也要抑制
  該 smell。
- 一律採判斷式檢查。每個 smell 都是有標籤的 heuristic，不是硬性違規。工具
  已經檢查的內容跳過。

每個 smell 先說明定義，再說明修正方式：

- **Mysterious Name**：函式、變數或型別名稱無法表達它做什麼或保存什麼。
  重新命名；如果找不到誠實的名稱，表示設計不清楚。
- **Duplicated Code**：變更中有兩個以上 hunk 或檔案出現相同的邏輯形狀。
  抽出共用形狀，讓兩處都呼叫它。
- **Feature Envy**：method 讀取另一個 object 的資料多於自己的資料。把
  method 移到它依賴的資料所在處。
- **Data Clumps**：相同幾個欄位或參數反覆一起傳遞。把它們包成一個 type。
- **Primitive Obsession**：primitive 或 string 代替了值得有獨立型別的 domain
  concept。為這個 concept 建立小型 type。
- **Repeated Switches**：相同 type 的 `switch` 或 `if` cascade 在變更中重複。
  用 polymorphism，或讓兩處共用同一張 map。
- **Shotgun Surgery**：一個邏輯變更迫使 diff 分散修改許多檔案。把會一起
  變動的內容集中到一個 module。
- **Divergent Change**：同一個檔案或 module 因為多個無關原因被修改。拆開，
  讓每個 module 只因一個原因變更。
- **Speculative Generality**：加入 spec 沒要求的 abstraction、parameter 或
  hook。刪除它們，inline 回真正有需求的形狀。
- **Message Chains**：caller 不該依賴很長的 `a.b().c().d()` 導覽。把這段
  walk 藏在第一個 object 的一個 method 裡。
- **Middle Man**：class 或 function 主要只做 delegation。移除它，直接呼叫
  真正的 target。
- **Refused Bequest**：subclass 或 implementer 忽略或覆寫繼承內容的大部分。
  移除 inheritance，改用 composition。

## 執行兩個軸

平行啟動 Standards 與 Spec review sub-agent，讓兩個 context 保持獨立。

Standards sub-agent 收到：

- 完整 diff command 與 commit list；
- 找到的所有 standards-source 檔案；
- 上述 smell baseline。

它的任務是：逐檔案或依相關 hunk 回報 diff 違反的每個文件規範，引用
standard file 與 rule，也回報任何 baseline smell。區分文件規範的硬性違規
與判斷式問題。文件規範優先於 baseline。跳過工具已經檢查的內容。報告少於
400 字。

Spec sub-agent 收到：

- 完整 diff command 與 commit list；
- 從 `.scratch/base` 載入的有效 task source，或 user summary 為空時從
  commit subject 建立的 task source。

它的任務是：回報 task source 要求但缺少或不完整的需求、diff 中未被要求的
行為，以及看似已實作但實作可能錯誤的需求。每個 finding 引用相關的
task-source 行。報告少於 400 字。

## 彙整

在 `## Standards` 與 `## Spec` 標題下呈現兩份報告，原樣或輕微清理。不要
合併或重新排序 findings；兩個軸必須保持分開。

最後用一行寫出每個軸的 finding 總數，以及各軸最嚴重的問題，若有的話。不要
在兩個軸之間選出單一優勝者。
