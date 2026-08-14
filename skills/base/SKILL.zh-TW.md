---
name: base
description: "建立或重設固定的 BASE_SHA 與 task source，供 implement 與 code-review 使用。"
disable-model-invocation: true
---

請明確使用 `/base`，再使用 `/implement` 或 `/code-review`。這個 skill 會建立或驗證 repo-local `.scratch/base` manifest。它不會自動呼叫 `implement` 或 `code-review`。

## 指令

外部 task source：

```text
/base <full-40-character-BASE_SHA> --source <provider> --task-ref <ref>
```

使用者提供的 task source：

```text
/base <full-40-character-BASE_SHA> --source user --summary <one-line summary>
```

要刻意替換既有 baseline 或 task source，請用相同參數執行 `/base
reset`。使用不同值的一般呼叫必須停止，不能默默覆寫既有狀態。

## Manifest

`.scratch/base` 必須完全符合下列其中一種格式：

```text
base_sha: <full 40-character SHA>
source: github
ref: https://github.com/org/repo/issues/123
```

```text
base_sha: <full 40-character SHA>
source: user
summary: <one-line summary>
```

`base_sha`、`source` 與選定的 source value 必填。`ref` 與 `summary` 互斥。Provider key 使用小寫，且可以擴充而不改變這個格式。未知欄位、重複欄位、空值、無效 SHA、空白行，以及含有 `ref` 的 user source 都視為無效。

以 atomic 方式寫入檔案。不要加入 commit trailer，也不要修改 `.git/info/exclude`。MCP adapter 會自動將 `.scratch/**` 排除在 review 輸入之外。完成 review 後，MCP 會另外建立 finding counter；只有 `/base reset` 會清除它。

執行期間，baseline 會保留在記憶體中；中斷後會從這個檔案重新載入。不要把 task payload 複製到 state。Host agent 會讀取 `ref` 一次，並把完整的 raw response 傳給 implement 與 OCR。
