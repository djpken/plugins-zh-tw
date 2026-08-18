---
name: base
description: "建立或重設固定的 BASE_SHA 與 task source，供 implement 與兩個 code-review variant 使用。"
disable-model-invocation: true
---

請明確使用 `/base`，再使用 `/implement`、`/code-review` 或
`/code-review-ocr`。這個 skill 會建立或驗證 repo-local `.scratch/base`
manifest。它不會自動呼叫其他 skill。

## 指令

外部 task source：

```text
/base <full-40-character-BASE_SHA> --source <provider> --task-ref <ref>
```

使用者提供的 task source：

```text
/base <full-40-character-BASE_SHA> --source user --summary <one-line summary>
```

只供 review 使用的空 task source：

```text
/base <full-40-character-BASE_SHA> --source user --summary ""
```

空的 user summary 是只供 review 使用的 placeholder。`/base` 不會讀取或
填入 commit message；遇到這個格式時，`/code-review` 會從被 review 的
commit subject 在記憶體中建立 task source。單一 commit review 請把目標
commit 的 parent 當作 `BASE_SHA`，並讓 `HEAD` 停在目標 commit。

要刻意替換既有 baseline 或 task source，請用相同參數執行 `/base reset`。
使用不同值的一般呼叫必須停止，不能默默覆寫既有狀態。

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
summary: <one-line summary or empty>
```

`base_sha` 與 `source` 必填。External source 必須有非空的 `ref`。User
source 必須有 `summary` 欄位，只有 caller 明確傳入
`--summary ""` 代表只供 review 的 fallback 時，才允許它為空。`ref` 與
`summary` 互斥。Provider key 使用小寫，且可以擴充而不改變這個格式。未知
欄位、重複欄位、必填且非 summary 欄位的空值、無效 SHA、空白行，以及含有
`ref` 的 user source 都視為無效。

以 atomic 方式寫入檔案。不要加入 commit trailer，也不要修改
`.git/info/exclude`。兩個 review variant 都會把 `.scratch/**` 排除在
review 輸入之外。只有 OCR adapter 會在完整 OCR review 後建立
`.scratch/finding-counts.json`，`/base reset` 會清除該 counter。

執行期間，baseline 會保留在記憶體中；中斷後會從這個檔案重新載入。不要把
task payload 複製到 state。Host agent 每 round 讀取 `ref` 一次，並把完整
raw response 傳給 `implement`、`code-review` 或 `code-review-ocr`。
