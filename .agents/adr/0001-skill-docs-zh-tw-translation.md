# 0001: SKILL.md 系列改為直接覆寫成繁體中文,頂層 README 例外保留英文兄弟檔

## 狀態

已採用

## 背景

這個 repo 收錄的 8 個 skill 語言不一致:`neat-freak`、`git-fork-remotes`、`translating-skill-docs` 是英文;`hv-analysis`、`storage-analyzer`、`aihot`、`khazix-writer` 是從另一個簡體中文 skills repo 遷移過來的簡體中文;`writing-router-skill` 已經是繁體中文。維護者主要用 zh-TW 使用這些 skill,希望統一成繁體中文以方便自己理解與維護,而不是為了對外發布雙語版本。

repo 裡已經存在 `translating-skill-docs` 這個 skill,它定義了另一套翻譯策略:替每個 `SKILL.md` 新增 `SKILL.<locale>.md` 兄弟檔,原始檔案逐字不動。那套策略是為了「fork 一個 skill repo 之後加上翻譯、同時保留跟上游合併的能力」而設計的。這次任務的情境不同——repo 是維護者自己的,沒有上游合併的顧慮,強行套用兄弟檔策略只會多出一份很快就會跟正本脫節的檔案(頂層 `README.md`/`README.zh.md` 這對兄弟檔本身就已經出現內容過期不同步的狀況,可作為前車之鑑)。

## 決策

1. **7 個 `SKILL.md`(不含 `writing-router-skill`,它已是繁中)一律直接覆寫成繁體中文,不保留英文/簡體版本、不建立 sibling 檔。**
   - 原本是簡體中文的(`hv-analysis`、`storage-analyzer`、`aihot`、`khazix-writer`):做簡體轉繁體的字形與慣用語轉換(如「软件」→「軟體」),不是重新翻譯,語意、語氣、資訊內容不變。
   - 原本是英文的(`neat-freak`、`git-fork-remotes`、`translating-skill-docs`):正常翻譯成繁體中文。
   - `description:` frontmatter 一併翻譯/轉換,理由是這些 skill 主要在中文對話情境下被使用,一致性優先於英文情境下的邊際觸發命中率。
   - `name:` frontmatter 逐字不動,因為它是 harness 呼叫用的 key。
   - 每個 skill 指向的 `references/*.md` 一併處理,理由跟 SKILL.md 本體相同——避免使用者點進去卻看到沒翻譯的參考文件,體驗斷裂。
   - `translating-skill-docs` 本身也照樣直接覆寫,即使它描述的是另一套(兄弟檔)策略、跟這次任務用的策略不同——這份文件的翻譯與否跟它教的方法論是否被採用是兩回事,硬要為了自洽而重寫它的內容反而超出這次任務範圍。
   - `scripts/`、`assets/`、非 `.md` 的參考檔(如 `schema.json`)、`evals/` 測試 fixture、`LICENSE` 全部不動。

2. **對外發布的 `docs/<bucket>/<skill>.md` 頁面排除在外**,維持英文。這些是給外部使用者看的公開頁面,語言決策是獨立話題,不該被這次「整理自己 repo」的任務動到。

3. **頂層 `README.md` 是唯一的例外,採用「覆寫 + 兄弟檔角色互換」而非純粹覆寫**:
   - `README.md` 換成繁體中文內容(取代原本 `README.zh.md` 的角色,且以更新過的英文版內容重新翻譯,因為原本的 `README.zh.md` 內容已經跟英文版脫節)。
   - 原本的英文內容改存成 `README.en.md`。
   - 舊的 `README.zh.md` 移除,合併進新的 `README.md`。
   - 之所以跟其他 SKILL.md 不同、選擇保留英文兄弟檔,是因為頂層 README 先前就已經在用雙語兄弟檔慣例,直接刪掉英文版是丟棄既有的維護投資;而 SKILL.md 系列從一開始就沒有這層顧慮。
   - 各 bucket 底下的 `README.md`(`engineering/`、`productivity/`、`misc/`、`personal/`、`in-progress/`、`deprecated/`)沒有既存的雙語慣例,所以跟 SKILL.md 一樣直接覆寫,不留英文版。

4. **register 限制**:全部翻譯/轉換成台灣標準書面繁體中文,不得出現粵語白話用字(嘅、喺、唔、咗、呢個、嗰、佢哋等)。

## 後果

- 這個 repo 裡以後新增或翻新 skill,若要跟現有慣例一致,SKILL.md 本體與其 references 應直接維護繁體中文版本,不用再另外想「要不要建兄弟檔」。
- `translating-skill-docs` 這個 skill 描述的兄弟檔策略仍然有效——是給「翻譯別人 fork 來的 skill repo」情境用的,跟這個 repo 自己內部的翻譯慣例是兩回事,兩者並存不衝突,未來使用者不要混淆。
- 頂層 README 的雙語兄弟檔(`README.md` + `README.en.md`)之後如果又不同步,參考這份 ADR 的背景說明即可理解成因,不用重新推敲一次。
- CLAUDE.md、`.claude-plugin/plugin.json`、對外 `docs/*.md` 頁面不受這次決策影響。
