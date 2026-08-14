# Productivity

通用工作流程工具,不限於程式碼。

## User-invoked

只有輸入時才會觸發(`disable-model-invocation: true`)。

- **[translating-skill-docs](./translating-skill-docs/SKILL.md)** — 幫每個 skill 加上翻譯後的 `SKILL.<locale>.md` 兄弟檔,不改變 runtime 行為。

## Model-invoked

可由使用者觸發,也可由 model 觸發(有豐富的觸發語句讓 model 可以自動找到它們)。

- **[hv-analysis](./hv-analysis/SKILL.md)** — 執行橫縱分析法——一套深度研究框架,追蹤主題完整的歷史脈絡,並與同期競品做系統性比較——最後產出排版精美的 PDF 報告。
- **[storage-analyzer](./storage-analyzer/SKILL.md)** — 唯讀的 macOS/Windows 儲存空間分析工具,掃描磁碟用量、依風險分級標出可清理項目,並產出可一鍵清理的互動式 HTML 報告。
