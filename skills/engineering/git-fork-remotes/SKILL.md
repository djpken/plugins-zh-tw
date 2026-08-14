---
name: git-fork-remotes
description: 在 fork 一個 repo 之後設定 git remote——讓 origin 變成你的 fork、upstream 變成原始專案——並修正 branch tracking 使其對應。
disable-model-invocation: true
---

## 步驟

1. **分類 remote。** 執行 `git remote -v`。執行 `gh api user --jq .login` 取得你的 GitHub 使用者名稱,並用它比對每個 remote URL 裡的 `github.com/<owner>/<repo>` 片段。owner 與你相符的 remote 就是 **fork**;其他則是 **upstream**(原始專案)。如果 `gh` 無法使用,或某個 URL 的 owner 無法判斷(SSH alias、自架伺服器),應該直接詢問,而不是用猜的。
   - 完成條件:每個 remote 都被分類為 fork 或 upstream,沒有任何一個維持未知狀態。

2. **依慣例重新命名。** 目標:`origin` = fork,`upstream` = 原始專案。
   - 名為 `origin` 的 remote 其實是 upstream,而另一個 remote 是 fork:執行 `git remote rename origin upstream`,接著執行 `git remote rename <fork-remote> origin`。
   - 只存在一個 remote(原始專案,名為 `origin`),尚未有 fork remote:詢問 fork 的 URL,然後執行 `git remote rename origin upstream` 與 `git remote add origin <fork-url>`。
   - 命名已經正確:略過重新命名這一步。
   - 完成條件:`git remote -v` 顯示 `origin` → fork URL,`upstream` → 原始專案 URL。

3. **修正 branch tracking。** 用 `git branch --format='%(refname:short)'` 列出每一個本機 branch——不要只看 `git config --get-regexp branch\..*\.remote` 的輸出,因為沒有設定 tracking 的 branch 不會出現在那份清單裡。針對清單中每一個沒有 tracking `origin` 的 branch(除非使用者說明該 branch 本來就只該對應 upstream),執行 `git branch --set-upstream-to=origin/<branch> <branch>`。
   - 完成條件:完整清單中的每個本機 branch 都 tracking `origin`,沒有遺漏任何一個。

4. **驗證並回報。** 顯示最終的 `git remote -v` 與 `git config --get-regexp 'branch\..*\.(remote|merge)'`。說明哪個 remote 對應哪個角色:預設 push 會送到你的 fork(`origin`),而 `git fetch upstream` 則會拉取原始專案的更新。

## 注意事項

- 絕對不要刪除 remote——只做重新命名。
- 當 URL 的歸屬不明確時不要用猜的——直接詢問。
