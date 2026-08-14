# 當前全部精選同步

只在使用者明確要求拿到當前全部精選，或維護持久化精選鏡像時讀取本文件。普通資訊問答不要使用 snapshot。

## 首次建立鏡像

snapshot 是分頁的，一輪 bootstrap 需要多次請求。當前約 2900 條：`fields=minimal` 約 1.08MB（gzip 247KB），`fields=default` 約 3.1MB（gzip 1.05MB），且只增不減。

1. 選擇欄位模式：
   - 只維護 id、標題、站內連結和分類：`fields=minimal`（預設首選，省四倍流量）。
   - 需要摘要或第三方原文連結：`fields=default`。
2. 請求 `/api/v1/selected/snapshot?fields=<模式>&limit=500`。
3. 累積本頁 `items`；記下回應裡的 `cursor`（逐頁恆定）。
4. 只要 `hasMore=true`，就帶 `page=<上一回應的 nextPage>` 繼續請求下一頁，`fields` 不必也不能改。
5. `hasMore=false` 時本輪結束。把累積的完整集合與**第一頁就拿到的那個 `cursor`** 作為一個原子狀態寫入；不能只保存其中一半，也不能中途保存半份鏡像。

兩個游標別搞混：`cursor` 是同步水位、翻頁期間不變、翻完才用來調 changes；`nextPage` 只用於翻頁。用 `nextPage` 去調 changes，或者翻頁沒翻完就開始調 changes，都會造成鏡像缺條。

翻頁期間條目可能被編輯或撤選。不用處理——翻完之後第一次 `changes` 會用同一個水位把這些變化補齊：改過的條目再來一次 `upsert`（冪等），撤選的條目來一次 `remove`（本地沒有就是空操作）。

如果使用者只想在對話裡查看當前全部精選，不要翻完所有頁：取第一頁、報告 `count` 與 `hasMore`，再按使用者指定數量展示。未指定數量時仍只先展示最重要的 3—8 條。

## 持續接收變化

1. 請求 `/api/v1/selected/changes?cursor=<原樣回傳>&limit=100`。
2. 完整應用本頁每條 change：
   - `op=upsert`：按 id 新增或替換條目。
   - `op=remove`：按 id 刪除條目。
3. 整頁全部應用成功後，再原子保存回應中的新 cursor。
4. `hasMore=true` 時立即用新 cursor 繼續排空積壓；排空後再恢復正常輪詢。
5. 健康輪詢期間只呼叫 changes，不請求 snapshot、items 或 fingerprint。

## 恢復

changes 回傳 `409` 且 Problem `code=snapshot_required` 時：

1. 停止重試舊 cursor。
2. 用原來的欄位模式重新走一遍完整的 snapshot 分頁流程（從無 `page` 的第一頁開始）。
3. 原子替換本地完整集合與 cursor。
4. 後續恢復 changes 輪詢。

snapshot 的 `page` 游標同樣可能回傳 `409 snapshot_required`（本輪快照已失效）。此時丟掉半份結果，從第一頁重新開始，不要接著舊 `nextPage` 翻。

一次恢復仍失敗時停止並報告；不要循環下載 snapshot。

## 不變量

- cursor 不透明且綁定欄位模式、同步端點和伺服器端水位。
- 不解析、不遞增、不修改、不跨端點複用 cursor。
- snapshot 的 `cursor`（同步水位）與 `nextPage`（翻頁）是兩個東西，互相解不開，任何一方都不能替代另一方。
- 鏡像不完整（`hasMore=true` 還沒翻完）時不要開始 changes 輪詢，也不要對外聲稱已同步。
- `publishedAt` 和 `discoveredAt` 都不能表示編輯與撤選，不能充當完整同步水位。
- 不使用重疊時間窗替代 changes；時間窗無法可靠表達 remove。
- 不把 `/api/v1/items?mode=all` 當成「當前全部精選」。它是最近公開池，語義不同。
- 持久任務保存完整 URL 的 ETag，並在下次發送 `If-None-Match`；`304` 時保持本地狀態和 cursor。
- 正常輪詢至少間隔 60 秒；`hasMore=true` 的積壓分頁除外。
