# macOS 資料佈局與分級參考

分析 macOS 掃描結果時讀這份。講"東西存在哪、怎麼辨認、歸哪一級"。

## 關鍵目錄

| 目錄 | 裝什麼 | 典型分級 |
|---|---|---|
| `~/Library/Caches/*` | 應用/工具快取（瀏覽器、Homebrew、pip、playwright） | 🟢 可自動清 |
| `~/.cache/*`、`~/.npm`、`~/.cargo`、`~/.gradle`、`~/.m2` | 開發快取 | 🟢 |
| `~/Library/Developer/Xcode/DerivedData`、`CoreSimulator` | Xcode 構建/模擬器 | 🟢 |
| `~/Library/Containers/<UUID 或 bundleid>` | 沙盒應用資料（聊天記錄、離線影片、設定） | 🟡 多為使用者資料 |
| `~/Library/Application Support/*` | 應用資料（Chrome Profile、Claude VM、飛書） | 🟡 |
| `~/Downloads` 裡的 .dmg/.pkg | 安裝包殘留 | 🟢 |
| `/Applications/*.app` | 應用本體 | 🔴 僅當重複/想卸時上燈，否則歸藍色 |
| 系統檔案、APFS 本地快照 | 系統 | 不上燈，歸藍色"系統及其他" |

## 辨認"神秘 UUID 容器"

`~/Library/Containers/` 下 UUID 命名的大目錄，要查清屬於哪個 App：
- `ls` 進 `Data/Documents/`、`Data/Library/`，找帶 bundle id 的子目錄（如 `com.bilibili.bbad` → 嗶哩嗶哩）
- 大頭常藏在隱藏目錄（如 `.Downloads/` 裡的 `.bilitask` 離線影片）
- 仍只讀，別動檔案

## 間接釋放（寫進 long_term，不上紅燈）

- 系統"可清除空間"磁碟緊張時自動回收
- 重啟釋放部分 swap / 臨時快照
- `brew cleanup --prune=all`、清 Xcode DerivedData
- 調整 Time Machine 本地快照保留策略

## 刪除機制

`server.py` 在 macOS 用 osascript 調 Finder 入垃圾桶；首次彈自動化授權，點允許。
