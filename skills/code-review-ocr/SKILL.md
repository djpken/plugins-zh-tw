---
name: code-review-ocr
description: "用同步 OCR MCP review 固定的 /base baseline 到目前 HEAD 的 Git 範圍。"
---

只 review Git 範圍。固定起點來自 `.scratch/base`；不要要求會移動的
`HEAD~N` ref，也不要用 workspace 或單一 commit review 執行這個 skill。

## 前置條件

1. 必須存在且非空的 `.scratch/base`。不要建立它，也不要自動呼叫
   `/base`。Manifest 建立由 `/base` 負責，嚴格格式驗證由 MCP adapter 負責。
2. 讀取本次 review 所需的固定 `base_sha`、`source` 與 source value。
3. 在 review 開始時完整讀取 task source 一次。外部 `source/ref` 透過 host
   integration 讀取；`source: user` 使用 `summary`。外部 source 讀取失敗時，
   不得使用 cache 或摘要替代。只把 payload 保留在記憶體中，不要寫入 repo
   state、commit 或其他 log。
4. 如果無法讀取 task source、內容無法序列化，或超過可用 context，停止並
   回傳 `needs_human`，不要呼叫 OCR。

## 範圍 review

1. 從 `.scratch/base:base_sha` 解析 `from`，並將 `to` 解析為目前 `HEAD`。
   呼叫者明確提供目前 head 的 commit ref 時，才使用該 ref。呼叫 MCP 前，
   將兩者轉成完整 commit SHA。
2. 要求 `from` 是 `to` 的 ancestor。範圍無效時停止 review。
3. 自動排除 `.scratch/**`，不納入 review 輸入。
4. 排除上述檔案後若範圍為空，回傳 OCR 原生的 empty-review JSON，不要呼叫
   OCR。
5. 在下方 deterministic `background` wrapper 中帶入該 Git 範圍與 raw task
   source，且只呼叫一次 `ocr_review`。呼叫必須持續阻塞，直到取得 terminal
   result。

```text
<task-source>
source: github
ref: https://github.com/org/repo/issues/123

content:
<complete provider payload>
</task-source>
```

不要啟動 review sub-agent、執行 OCR CLI、檢查 progress event，或輪詢完成
狀態。Host transport 中斷時，呼叫一次 `ocr_review_wait`。MCP process 結束
時，使用相同 target 恢復同一個 persisted range session 一次。Unavailable
session 回傳 `needs_human`。

## 結果契約

回傳 OCR 原生 JSON，包含既有的頂層 `status`、`summary`、`comments`、
`warnings`、coverage 與 session 欄位。不要加入第二種報告格式、
`review_handoff` 或 `recommended_disposition`。

完成的 range result 會由 forked MCP adapter 為每個 comment 加上
`finding_id`、`consecutive_review_count` 與 `automation_status`，並更新
`.scratch/finding-counts.json`。同一 finding 連續出現在三次完成的 review 時，
會回傳 `deferred_for_human`；implement 會跳過它，繼續處理其他 findings。
Partial、failed、cancelled 或 incomplete 的 OCR result 不會更新 counter。
