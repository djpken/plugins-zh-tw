# AI HOT v1 API 參考

只在需要完整參數、欄位、分頁或構建用戶端時讀取本文件。普通資訊問答優先遵循 `SKILL.md` 的預設路由。

## 共同合約

- Base URL：`https://aihot.virxact.com`
- 匿名唯讀，不需要 API Key，不發送 cookie。
- OpenAPI：`https://aihot.virxact.com/openapi-v1.json`
- 所有 cursor 都是不透明書籤：只原樣回傳給產生它的同一端點和同一查詢，不解析、不修改、不跨查詢複用。
- 未知參數、無效參數、損壞或跨查詢 cursor 都回傳明確的 Problem JSON，不會靜默回到第一頁。
- 對同一完整 URL 保存回應 `ETag`；下次發送 `If-None-Match`。`304` 表示內容未變化。
- items cursor 沒有按時間自動失效，但 24 小時／7 天是滾動窗口，較老條目可能在兩次翻頁之間自然離開窗口；需要精確鏡像時改用 selected snapshot + changes。

## 操作

### 最近資訊、分類與搜尋

`GET /api/v1/items`

| 參數 | 合約 |
|---|---|
| `mode` | `selected` 或 `all`；預設 `selected` |
| `window` | `24h` 或 `7d`；預設 `7d` |
| `by` | `timeline` 或 `published`；預設 `timeline`（見下方「時間口徑」） |
| `category` | `ai-models`、`ai-products`、`industry`、`paper`、`tip` |
| `q` | 2—200 字；使用伺服器端搜尋 |
| `limit` | 1—100；預設 50。只需要頭幾條時顯式調小，別預設拉滿 50 |
| `cursor` | 原樣回傳上一頁的 `page.nextCursor` |

#### 時間口徑

`window` 從哪個時間點往回算、結果按哪個時間排序，由 `by` 決定。兩個原始時間戳恆定隨每條回傳，可自行判斷。

- `by=timeline`（預設）：與 aihot.virxact.com 網頁看到的順序和集合一致。規則是——原文發布後 72 小時內被收錄，按收錄時間；超過 72 小時才收錄的歷史回填，歸位到原文發布日。所以官方部落格、公眾號、HuggingFace Daily 這類「原文兩三天前發、今天才抓到」的慢推信源，仍會出現在 `window=24h` 裡，同時舊文回填不會冒充最近。
- `by=published`：只按第三方原文發布時間。慢推信源會掉出短窗口——同一時刻 `window=24h` 下它比預設口徑少約兩成條目。需要嚴格按原文時間線對帳時才用。

切換 `by` 會讓已持有的 cursor 失效並回傳 `invalid_cursor`，這是有意的：換了口徑繼續用舊書籤會串頁。重新從第一頁開始即可。

回應外層：

```json
{
  "schemaVersion": 1,
  "query": {
    "mode": "selected",
    "category": null,
    "q": null,
    "window": "24h",
    "by": "timeline",
    "ordering": "timelineDesc"
  },
  "items": [],
  "page": {
    "count": 0,
    "hasMore": false,
    "nextCursor": null
  }
}
```

每個 item 必有以下鍵：

- `id`
- `title`
- `originalTitle`
- `summary`
- `source.name`
- `links.aihot`
- `links.original`
- `publishedAt`
- `discoveredAt`
- `category`
- `score`
- `selected`

其中 `originalTitle`、`summary`、`publishedAt`、`category` 和 `score` 的鍵始終存在，但值可以是 `null`；展示前必須判空。`id`、`title`、`source.name`、`links.aihot`、`links.original`、`discoveredAt` 和 `selected` 為非空值。回應還可能帶可選的 `attribution`，用戶端不得依賴它一定存在，也不得因未來新增未知欄位而報錯。`page.count` 是本頁條數，不是全庫總數。

示例：

```text
GET /api/v1/items?mode=selected&window=24h&limit=8
GET /api/v1/items?mode=selected&window=7d&category=paper&limit=20
GET /api/v1/items?mode=selected&window=7d&q=OpenAI&limit=20
GET /api/v1/items?mode=all&window=24h&limit=50
```

### 當前熱點

`GET /api/v1/hot-topics`

回應為 `{schemaVersion, count, items}`，不是可續頁集合。保持 API 熱度順序。item 包含 `sourceCount`、`signalCount`、`sourceNames` 與 `latestAt`；其中 `sourceCount` 是獨立信源數。熱點與普通資訊欄位不同，不得把兩種回應強行混成同一列表協定。

### 日報

```text
GET /api/v1/dailies?limit=7
GET /api/v1/dailies/latest
GET /api/v1/dailies/2026-07-24
```

- 索引回應為 `{schemaVersion, count, items}`，不是可續頁集合。
- 最新或指定日報回應為 `{schemaVersion, report}`。
- 保留 report 的 `lead`、`sections` 與 `flashes` 結構，不把日報重排成普通 items。
- 日報索引項和 report 頂層的 `links.aihot` 必有。sections／flashes 中 `links.aihot` 可能為 `null`；此時使用必有的 `links.original`，不要再尋找舊欄位 `permalink` 或 `sourceUrl`。
- 最新日報或指定日期回傳 404 時，索引只查一次有界的 `/api/v1/dailies?limit=7`。索引有結果時，從中選擇實際回傳的最近日期，再請求一次對應的 `/api/v1/dailies/{date}` 取得完整日報並如實說明日期；索引為空就報告當前沒有可用日報。絕不猜「昨天」或自行拼接日期。

### 正文與週期報告邊界

- `items` 只回傳標題、摘要、來源、時間、評分和連結，不回傳正文，也沒有 `/api/v1/items/{id}`。使用者要深入閱讀時提供 `links.aihot` 和 `links.original`，不要抓網頁或全文 RSS 冒充單篇正文 API。
- AI HOT 編輯成品週報與月報目前只有 `/weekly` 和 `/monthly` 網頁，沒有 v1、Skill 或 RSS 端點。「最近一週精選」仍是滾動 7 天 items 查詢，不得稱為正式週報。

### 完整精選同步

```text
GET /api/v1/selected/snapshot?fields=minimal&limit=500
GET /api/v1/selected/snapshot?fields=minimal&limit=500&page=<opaque>
GET /api/v1/selected/changes?cursor=<opaque>&limit=100
```

只有使用者明確要求當前全部精選或持久鏡像時才使用。完整算法見 [sync.md](sync.md)；不要僅憑本文件實現同步狀態機。

snapshot 是**分頁**的，一次請求拿不到全部：

| 參數 | 合約 |
|---|---|
| `fields` | `default` 或 `minimal`；預設 `default`。`minimal` 去掉摘要與原文連結，體積約為 default 的四分之一 |
| `limit` | 1—1000；預設 500 |
| `page` | 原樣回傳上一回應的 `nextPage`；續頁的 `fields` 由游標鎖定，傳不同值無效 |

回應裡有兩個不同的游標，**不要混用**：

- `cursor`：同步游標，逐頁恆定，指向第一頁取到的水位。**翻完所有頁之後**才拿它調 `changes`。
- `nextPage`：翻頁游標，只在本輪快照內有效。`hasMore=true` 時必須繼續翻，否則鏡像不完整。

規模參考：當前約 2900 條，`fields=default` 全量約 3.1MB（gzip 1.05MB），`fields=minimal` 約 1.08MB（gzip 247KB）。條目只增不減，會逐年變大——不確定就用 `minimal`，需要摘要時再取 `default`。

## 分頁

1. 處理當前頁全部 items。
2. `page.hasMore=true` 時，原樣回傳 `page.nextCursor` 請求下一頁。
3. 達到使用者指定數量即可停止；無需為了「完整」耗盡所有頁。
4. `page.hasMore=false` 時結束。
5. cursor 報錯就報告或按對應恢復合約處理，絕不刪掉 cursor 後假裝翻頁成功。
6. 普通 items 分頁不是一致性快照；新條目不會造成已翻頁內容重複，但滾動窗口內的編輯、撤選和自然過期可能改變後續頁。完整、可恢復同步只使用 selected snapshot + changes。

## 欄位語義

- `links.aihot`：AI HOT 站內中文閱讀頁，預設主連結。
- `links.original`：第三方原文，僅在使用者要出處時附加。
- `originalTitle`：來源原標題，可能不是英文。
- `publishedAt`：第三方原文發布時間。展示前把 ISO 時間轉換到 `Asia/Shanghai`。
- `discoveredAt`：AI HOT 首次收到時間。`publishedAt` 為空時可回退使用，但必須標為「AI HOT 收錄時間」。
- `score`：0—100 總分，可能為空，不表示當前回應按它排序。
- `selected`：是否屬於當前精選。
- `category`：允許未來增加新值；不要把未知值當成回應損壞。

## 時間範圍

v1 只承諾 `24h` 和 `7d` 兩個伺服器端窗口：

- 今天、過去 24 小時：用 `24h`。
- 最近、最近一週：用 `7d`。
- 使用者要 2 天、3 天等其他七天內範圍：取 `7d` 後本地收窄。**收窄用的欄位必須與請求的 `by` 口徑一致**，否則會切掉伺服器端本來算在窗口內的條目：
  - 預設 `by=timeline`：用時間軸值——`publishedAt` 為空取 `discoveredAt`；`discoveredAt - publishedAt > 72 小時`（歷史回填）取 `publishedAt`；其餘取 `discoveredAt`。
  - 顯式 `by=published`：才直接用 `publishedAt`。
  - 拿 `publishedAt` 去收窄預設口徑，會把官方部落格、公眾號、HuggingFace Daily 這類慢推信源整批誤刪（見上方「時間口徑」）。
- 超過 7 天的普通公開池不承諾可用；不要用 selected snapshot 冒充歷史搜尋。
