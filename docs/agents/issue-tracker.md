# Issue tracker: GitHub

此 repo 的 issues 與 specs 使用 GitHub Issues。所有操作使用 `gh` CLI。

## 慣例

- **建立 issue**：`gh issue create --title "..." --body "..."`。多行內容使用 heredoc。
- **讀取 issue**：`gh issue view <number> --comments`，並用 `jq` 過濾 comments 及取得 labels。
- **列出 issues**：`gh issue list --state open --json number,title,body,labels,comments --jq '[.[] | {number, title, body, labels: [.labels[].name], comments: [.comments[].body]}]'`，搭配適當的 `--label` 與 `--state` 篩選。
- **留言**：`gh issue comment <number> --body "..."`
- **套用或移除 labels**：`gh issue edit <number> --add-label "..."` / `--remove-label "..."`
- **關閉 issue**：`gh issue close <number> --comment "..."`

從 clone 內執行時，`gh` 會從 `git remote -v` 自動判斷 repo。

## Pull requests 作為 triage 請求來源

**PRs as a request surface: no。**

若 repo 將外部 PR 視為 feature request，可將設定改為 `yes`。`/triage` 會讀取這個 flag。

設定為 `yes` 後，PR 會使用與 issues 相同的 labels 與 states，並使用對應的 `gh pr` 指令：

- **讀取 PR**：`gh pr view <number> --comments` 與 `gh pr diff <number>`
- **列出外部 PR**：`gh pr list --state open --json number,title,body,labels,author,authorAssociation,comments`，只保留 `authorAssociation` 為 `CONTRIBUTOR`、`FIRST_TIME_CONTRIBUTOR` 或 `NONE` 的 PR，排除 `OWNER`、`MEMBER` 與 `COLLABORATOR`
- **留言、套用 labels、關閉**：使用 `gh pr comment`、`gh pr edit --add-label` / `--remove-label`、`gh pr close`

GitHub 的 issues 與 PR 共用編號空間。裸的 `#42` 可能指 issue 或 PR，請先執行 `gh pr view 42`，失敗後再執行 `gh issue view 42`。

## Skill 要求發布到 issue tracker 時

建立 GitHub issue。

## Skill 要求取得相關 ticket 時

執行 `gh issue view <number> --comments`。

## Wayfinding 操作

`/wayfinder` 使用一個包含 child issues 的 GitHub issue 作為 map。

- **Map**：建立一個標有 `wayfinder:map` 的 issue，內容包含 Notes、Decisions-so-far 與 Fog。使用 `gh issue create --label wayfinder:map`。
- **Child ticket**：將 issue 連結為 map 的 GitHub sub-issue，使用 `gh api` 操作 sub-issues endpoint。若 repo 未啟用 sub-issues，改在 map body 的 task list 加入 child，並在 child body 頂端加入 `Part of #<map>`。Labels 使用 `wayfinder:<type>`，其中 type 為 `research`、`prototype`、`grilling` 或 `task`。認領後，將 ticket 指派給負責的 dev。
- **Blocking**：使用 GitHub 原生 issue dependencies，這是 UI 可見的標準表示法。使用 `gh api --method POST repos/<owner>/<repo>/issues/<child>/dependencies/blocked_by -F issue_id=<blocker-db-id>` 建立關係。`<blocker-db-id>` 必須是 blocker 的 numeric database id，可用 `gh api repos/<owner>/<repo>/issues/<n> --jq .id` 取得，不能使用 `#number` 或 `node_id`。GitHub 會回報 `issue_dependencies_summary.blocked_by`，只計算仍開啟的 blocker。若 repo 不支援 dependencies，改在 child body 頂端加入 `Blocked by: #<n>, #<n>`。所有 blocker 都關閉後，ticket 才算解除阻塞。
- **Frontier query**：列出 map 的 open children，範圍依 map 的 sub-issues 或 task list，移除有 open blocker 或已指派者，依 map 順序取第一個。
- **Claim**：`gh issue edit <n> --add-assignee @me`，這是 session 的第一次寫入。
- **Resolve**：先執行 `gh issue comment <n> --body "<answer>"`，再執行 `gh issue close <n>`，最後將 context pointer，包含 gist 與連結，附加到 map 的 Decisions-so-far。
