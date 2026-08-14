# Domain Docs

Engineering skills 探索 repo 時，依下列規則讀取 domain 文件。

## 探索前先讀取

- 根目錄的 `CONTEXT.md`
- 若根目錄有 `CONTEXT-MAP.md`，依它讀取與主題相關的每個 context `CONTEXT.md`
- `docs/adr/` 中與目前工作範圍相關的 ADR

若檔案不存在，直接繼續，不需要指出缺少檔案，也不要預先建議建立檔案。`/domain-modeling` skill，經由 `/grill-with-docs` 與 `/improve-codebase-architecture` 進入時，會在 terms 或 decisions 真正確定後延遲建立這些檔案。

## 檔案結構

本 repo 採 single-context：

```
/
├── CONTEXT.md
├── docs/adr/
│   ├── 0001-event-sourced-orders.md
│   └── 0002-postgres-for-write-model.md
└── src/
```

若未來根目錄出現 `CONTEXT-MAP.md`，則改採 multi-context：

```
/
├── CONTEXT-MAP.md
├── docs/adr/                          ← system-wide decisions
└── src/
    ├── ordering/
    │   ├── CONTEXT.md
    │   └── docs/adr/                  ← context-specific decisions
    └── billing/
        ├── CONTEXT.md
        └── docs/adr/
```

## 使用 glossary 詞彙

在 issue title、重構提案、假設與 test name 中，使用 `CONTEXT.md` 定義的 domain terms。若需要的概念尚未出現在 glossary，這表示專案語彙可能存在缺口，請交由 `/domain-modeling` 處理。

## ADR 衝突

若輸出內容與既有 ADR 衝突，先明確指出衝突，不要靜默覆寫決策：

> _Contradicts ADR-0007 (event-sourced orders) — but worth reopening because…_
