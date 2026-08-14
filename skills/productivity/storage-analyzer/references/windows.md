# Windows 資料佈局與分級參考

分析 Windows 掃描結果時讀這份。講"東西存在哪、怎麼辨認、歸哪一級"。
注意：Windows 程式碼路徑在 macOS 上無法驗證，分析時對路徑存在性保持謹慎。

## 多磁碟機代號

Windows 通常多個盤（C:、D:…）。磁碟總覽會列出所有盤，但**分析和清理聚焦系統盤 C:**——快取、AppData、臨時檔案幾乎都在 C:。其他盤（D: 等）一般是使用者自存的資料/遊戲，歸 🟡 讓使用者自己判斷，不要自動給刪除按鈕。

## 關鍵目錄

| 目錄（環境變數） | 裝什麼 | 典型分級 |
|---|---|---|
| `%LOCALAPPDATA%`（`C:\Users\<u>\AppData\Local`） | 瀏覽器快取、應用資料、Temp，最大頭 | 快取 🟢 / 應用資料 🟡 |
| `%LOCALAPPDATA%\Temp`、`%TEMP%` | 臨時檔案 | 🟢 |
| `%APPDATA%`（Roaming） | 應用配置/資料 | 🟡 |
| 瀏覽器快取 `%LOCALAPPDATA%\Google\Chrome\User Data\*\Cache`、Edge 同構 | 瀏覽器快取 | 🟢 |
| 瀏覽器 `User Data\<Profile>`（非 Cache 部分） | 書籤/登入態 | 🟡 |
| `%USERPROFILE%\.cache`、`.npm`、`.gradle`、`.m2`、`.nuget\packages`、`%LOCALAPPDATA%\pip\Cache`、`Yarn` | 開發快取 | 🟢 |
| `C:\Program Files`、`Program Files (x86)` | 應用本體 | 🔴 僅重複/想卸時上燈，否則歸藍色 |
| `%USERPROFILE%\Downloads` 的安裝包 | exe/msi 殘留 | 🟢 |
| `C:\$Recycle.Bin` | 資源回收筒 | 🟡 提示使用者清空 |

## 系統佔用（不上燈，歸藍色"系統及其他"，間接釋放寫 long_term）

- `C:\Windows\WinSxS`：元件儲存，**絕不能手刪**，用 `DISM /Online /Cleanup-Image /StartComponentCleanup`
- `C:\Windows\SoftwareDistribution\Download`：Windows Update 快取，用磁碟清理處理
- `hiberfil.sys`（休眠）、`pagefile.sys`（虛擬記憶體）：系統管理，別手動刪
- 間接釋放：設定 > 系統 > 儲存 > 儲存感知；`cleanmgr`（磁碟清理）；擴充套件磁碟清理選 Windows 更新清理

## 刪除機制

`server.py` 在 Windows 用 ctypes 調 `SHFileOperationW`(FOF_ALLOWUNDO) 送進資源回收筒；純標準庫。🟢 項的 `trash_paths` 應在使用者配置檔案（`%USERPROFILE%`）目錄內，便於白名單與 HOME 越界校驗透過。
