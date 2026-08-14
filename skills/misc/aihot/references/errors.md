# 錯誤與重試

請求失敗時讀取本文件。先保護使用者問題的原意，再考慮重試；不得靠放寬參數或換資料源偽裝成功。

## v1 應用錯誤

請求到達 v1 應用後，標準錯誤使用 `application/problem+json`，至少包含：

```json
{
  "type": "/problems/invalid-request",
  "title": "Invalid request",
  "status": 400,
  "detail": "Human-readable explanation",
  "code": "invalid_request",
  "requestId": "req_123"
}
```

按穩定 `code` 分支，不解析 `detail` 人話：

- `invalid_request`：修正明確參數；不要自動改成另一個問題。
- `invalid_cursor`：書籤已失效（cursor 不能跨窗口、端點或查詢條件，伺服器端演進也可能讓舊書籤作廢）。恢復方式是**顯式從第一頁重新發起同一查詢**，並如實說明列表已從頭開始；禁止的是靜默回退——把新第一頁悄悄當成上次的續頁拼下去。
- `snapshot_required`：僅 selected changes 按 [sync.md](sync.md) 重建一次。
- `rate_limited`：遵守 `Retry-After`，串行重試。
- `temporarily_unavailable`：有限退避後告訴使用者暫不可用。

未知 code 按 HTTP status 保守處理，並保留 `requestId` 供回饋。

CDN 安全層可能在請求到達應用前回傳 566／567 極小 JSON，不保證 Problem 格式或 CORS 頭。這不改變 v1 的匿名存取合約，也不要求自定義 User-Agent。保留回應中的 `requestId` 和 `help`，正常退避後最多重試一次；仍失敗就停止並回饋。不得循環換 UA、偽裝 Mozilla／Chrome，或申請長期 IP 白名單。

## 重試

- `400／409`：除明確的 selected snapshot 恢復外，不盲目重試。
- `404`：普通資源不重試；日報 latest／指定日期按 [API 參考](api.md) 只查一次有界索引，不猜日期。
- `429`：按 `Retry-After` 等待；沒有該頭時等待 60 秒。不要增加並發。
- `5xx` 或逾時：最多重試 2 次，採用指數退避。
- 仍失敗：說明 AI HOT 暫不可用，並提供 `https://aihot.virxact.com/feedback`；不得用訓練記憶冒充實時資料。
- 瀏覽器跨域錯誤：說明瀏覽器沒有讀到回應，不把它誤報為使用者帳號或 IP 被封。

持久輪詢使用條件請求：

```text
If-None-Match: <上次同一完整 URL 的 ETag>
```

`304` 表示內容未變化。保留已有資料與 cursor，不把它當空回應。
