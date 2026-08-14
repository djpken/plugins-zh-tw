# 證據層級與發布終態

## 風險決定證據深度

| 結論 | 最低證據 |
|---|---|
| 「文件連結有效」 | 專案自己的 doc-link/index check,或逐一驗證連結存在性 |
| 「規則已同源」 | realpath/readlink/import + 平台實際的載入順序 |
| 「程式碼實作是 X」 | 目前目標分支的程式碼、schema、設定與相關測試 |
| 「PR 已完成」 | PR state=merged + merge commit;不能推導出已部署 |
| 「已部署」 | deploy marker/release 指向目標 commit + 服務 active |
| 「使用者已看到新版本」 | canonical 使用者 URL/API 的真實回應,必要時同時比對 origin/cache |
| 「可以安全清場」 | merged + production contains change + knowledge receipt + lane clean + 沒有唯一未整合的檔案 |
| 「已獲准清場」 | 完整結果已向使用者回報 + 使用者在該回報之後明確確認可以清場 + 現場要求的確認憑證 |
| 「整個專案是乾淨的」 | 專案內所有適用的事實面皆為 verified;警告、pending 與 out-of-scope 分項列出 |

程式碼直覺、舊的 memory、commit message 與 cache-buster URL,都只能當作線索,不能單獨證明正式環境的終態。

## 真相矩陣

對每個發現至少記錄:

```text
topic: <事實主題>
authority: <目前的權威來源>
code: verified-current | stale | n/a
runtime: verified-current | stale | unverified | n/a
docs: verified-current | stale | changed
rules: verified-current | stale | changed | n/a
memory: verified-current | stale | generated-read-only | changed | n/a
action: <做了什麼,或為什麼沒做>
verification: <指令、頁面或門禁>
```

使用者不需要看到完整矩陣,但最終摘要必須保留尚未結案的狀態。

## 發布狀態機

```text
implemented
  -> locally verified
  -> pushed / PR opened
  -> CI + required backtest/visual review passed
  -> merged
  -> deployed
  -> live verified
  -> knowledge closed + receipt recorded
  -> full result reported while evidence is preserved
  -> user explicitly approved cleanup after the report
  -> workspace cleaned
  -> post-cleanup audit passed
  -> cleanup result appended
```

跳過的狀態必須有專案規則允許的理由。失敗停在哪一格,就依那一格回報,不能用「大致完成」蓋過去。

## 快取與多重表面的產品

當使用者可見的內容經過 CDN、邊緣快取、搜尋索引、非同步 worker 或多個用戶端時,至少要辨識:

- origin 是否已經是新內容;
- canonical URL 是否仍指向舊快取;
- API/頁面/通知/RSS 是否共用同一個資料出口;
- deploy marker 是否等到所有非同步流程真正切換之後才寫入;
- cache-buster 是否只是診斷用途,而不是真實使用者的驗收結果。

只驗證了其中一個表面時,結論裡要明確標示範圍限制。

## 清場前的 gate

清場會銷毀複盤與使用者複核的證據,因此順序固定為:

1. 驗證目標工作已經整合並上線;
2. 同步 docs/rules/獲授權的記憶;
3. 記錄專案要求的 knowledge closeout receipt;
4. 預覽待刪除的 worktree/branch/db/artifact;
5. 檢查 dirty 檔案與 patch equivalence;
6. 向使用者完整回報結果並保留上述現場;
7. 等待使用者在看完回報後明確確認可以清場;
8. 記錄專案要求的使用者確認憑證,並執行獲授權的清理;
9. 重新執行 workspace audit,補上清場結果的回報。

使用者在最初任務中「收尾並清理」「做完刪掉」之類的預先授權,不能取代第 7 步;確認必須發生在完整回報之後,因為使用者要先看到結果,才能判斷是否需要保留現場繼續複核。

目錄名稱、分支的存續時間,以及 agent 工作階段是否已關閉,都不能證明可以刪除。

## 驗證失敗時

- 同一個失敗第二次出現,停止盲目重試,重新檢查假設、環境與指令名稱。
- 門禁要求機器可讀的 metadata 時,補上正確的留痕並觸發新事件;不要用人工確認去繞過可修復的格式問題。
- 失敗發生在正式環境寫入之前,明確說「尚未影響正式環境」;發生在切換流量之後,先確認目前的 active release 與回滾邊界。
- 任何未驗證的項目維持 `pending`,不要為了讓摘要好看,就把它降格成警告。
