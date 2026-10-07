---
name: domain-modeling-cn
description: 建構並持續打磨專案的領域模型。適用於釐清領域詞彙、建立團隊共享的通用語言（Ubiquitous Language）、記錄重要架構決策，以及維護領域知識模型的情境。
---
## 領域模型建構（Domain Modeling）

在設計過程中主動建立並持續精煉專案的領域模型。這是一項「主動式」紀律，包含挑戰現有術語、設計邊界情境（edge cases），並在概念逐漸清晰時立即記錄詞彙表與決策。

（僅僅閱讀 `GLOSSARY.md` 以了解專案詞彙並不屬於此技能，那只是任何技能都可以執行的基本習慣。本技能適用於修改或演進領域模型，而非單純使用它。）

### 檔案結構

大多數儲存庫會有單一 Context：

```text
context:/
├── GLOSSARY.md
├── docs/
│   └── adr/
│       ├── 0001-event-sourced-orders.md
│       └── 0002-postgres-for-write-model.md
└── src/
```

如果根目錄存在 `GLOSSARY-MAP.md`，表示該儲存庫包含多個 Context。此對照表指出各個 Context 所在位置：

```text
/
├── GLOSSARY-MAP.md
├── docs/
│   └── adr/                          ← 系統層級決策
├── src/
│   ├── ordering/
│   │   ├── GLOSSARY.md
│   │   └── docs/adr/                 ← Context 專屬決策
│   └── billing/
│       ├── GLOSSARY.md
│       └── docs/adr/
```

採用延遲建立（lazy creation）原則：僅在需要記錄內容時才建立檔案。

- 若不存在 `GLOSSARY.md`，則在第一個術語被明確定義時建立。
- 若不存在 docs/adr/，則在第一份 ADR 需要建立時建立。

### 在工作階段期間

#### 挑戰既有詞彙表

當使用者使用的術語與 CONTEXT.md 中既有定義衝突時，應立即指出。

例如：
> 「你的詞彙表將『cancellation』定義為 X，但你現在似乎是在指 Y。究竟是哪一個？」

#### 精煉模糊語言

當使用者使用含糊不清或具多重含義的術語時，應提出更精確的標準術語。

例如：
> 「你提到的『account』是指 Customer 還是 User？這兩者是不同概念。」

#### 討論具體情境

當討論領域關係時，應透過具體案例進行壓力測試。

主動設計能探測邊界條件的情境，以迫使使用者更精確定義各概念之間的界線。

#### 與程式碼交叉驗證

當使用者描述某項功能運作方式時，應檢查程式碼是否與描述一致。

若發現矛盾，應主動提出。

例如：
> 「你的程式碼會取消整張 Order，但你剛剛提到支援部分取消，究竟哪個才是正確行為？」

#### 即時更新 GLOSSARY.md

當某個術語被確認與定義完成後，應立即更新 `GLOSSARY.md`。

不要等到最後再集中整理，而應在概念明確的當下立即記錄。

請使用 [GLOSSARY-FORMAT.md](2_Reference/skills/domain-modeling-cn/GLOSSARY-FORMAT.md) 的格式。

`GLOSSARY.md` 應完全避免實作細節。

不要將 `GLOSSARY.md` 視為：

- 規格文件（Specification）
- 臨時筆記區（Scratch Pad）
- 實作決策儲存區

它的唯一用途是作為詞彙表（Glossary）。

#### 謹慎提出 ADR

僅在以下三個條件同時成立時，才建議建立 ADR（Architecture Decision Record）：

1. **難以回溯（Hard to reverse）**
   - 未來若要改變決策，成本相當高。

2. **缺乏背景時會令人困惑（Surprising without context）**
   - 未來讀者看到後，會自然產生：「為什麼要這樣設計？」的疑問。

3. **真實存在取捨（The result of a real trade-off）**
   - 確實評估過多種方案，並基於明確理由選擇其中之一。

若缺少任何一項條件，則不要建立 ADR。

請使用 [ADR-FORMAT.md](./ADR-FORMAT.md) 的格式。