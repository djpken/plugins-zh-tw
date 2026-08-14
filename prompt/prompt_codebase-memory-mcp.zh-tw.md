# 程式碼搜尋與導覽：結構化查詢 vs 語意查詢 vs 精確比對

搜尋程式碼時，依查詢類型選對工具：

- **精確 / 關鍵字搜尋**（function 名稱、變數、字串、file path）：
  用 Bash 的 `grep` 或 Explore agent。

- **結構化 / 關聯性查詢**（誰呼叫 X、X 依賴什麼、改 X 會壞什麼、call chain、import graph）：
  **必須**先用 `mcp__codebase-memory-mcp__*` 工具。依賴或呼叫關係的問題不要手動 grep 或讀檔案。

## codebase-memory-mcp 工具（結構化查詢）

| 工具                                          | 使用時機                                           |
| --------------------------------------------- | -------------------------------------------------- |
| `mcp__codebase-memory-mcp__search_graph`     | 依名稱或 label 找 function/class/route            |
| `mcp__codebase-memory-mcp__trace_path`       | 追蹤 call chain、data flow、跨 service 路徑        |
| `mcp__codebase-memory-mcp__get_code_snippet` | 取得指定 symbol 的精確原始碼                       |
| `mcp__codebase-memory-mcp__query_graph`      | 對程式碼圖跑複雜的 Cypher 查詢                     |
| `mcp__codebase-memory-mcp__get_architecture` | 專案層級的架構總覽                                 |
| `mcp__codebase-memory-mcp__search_code`      | 圖譜輔助的文字 / pattern 搜尋                      |
| `mcp__codebase-memory-mcp__index_repository` | 建立或重新整理結構索引                             |
| `mcp__codebase-memory-mcp__index_status`     | 確認索引是否為最新狀態                             |

## 合併使用模式

深度分析時串接兩者：先用 **codebase-memory-mcp** 畫出結構範圍（呼叫者、影響範圍）。
