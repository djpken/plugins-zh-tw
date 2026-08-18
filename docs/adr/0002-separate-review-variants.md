# ADR-0002：在同一個 Base contract 後分離 review variants

- 狀態：接受
- 日期：2026-08-18

我們保留 Standards／Spec 雙軸流程作為 `/code-review`，並將同步 OCR contract
放到 `/code-review-ocr`。兩個入口共用 `/base` 建立的 `base_sha`、task source
與 `base_sha...HEAD` range；`/implement` 走穩定的 `/code-review`，OCR findings
與 counter 只由 OCR variant 擁有。兩個 review 入口維持獨立的執行契約；skill
文件語言與 canonical 檔案政策另由 ADR-0003 定義。

## 考慮過的選項

- 讓 OCR 版本繼續佔用 `/code-review`，會使 `/implement` 綁定尚未完成的依賴。
- 只保留原本的 OCR 版本，會丟失已定義的 Standards／Spec review 流程。
- 使用兩個明確命名的 skill 入口，讓使用者選擇穩定的雙軸 review 或 OCR review。

## 後果

- `/code-review` 的輸出保持 `Standards` 與 `Spec` 分區，不產生 OCR JSON，也不管理 finding counter。
- `/code-review-ocr` 獨立完成自己的 OCR session，並擁有 `.scratch/finding-counts.json`。
- `/implement` 不讀取 OCR 專屬欄位，改以雙軸 review findings 收斂。
- 空的 user summary 只供 `/code-review` 從 reviewed commit subjects 建立記憶中的 task source，不能驅動 `/implement`。
