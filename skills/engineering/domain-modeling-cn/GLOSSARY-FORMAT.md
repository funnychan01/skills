# GLOSSARY.md 格式

## 結構

```
# {情境名稱}

{以一至兩句說明此情境是什麼，以及它存在的原因。}

## 語言

**Order（訂單）**：
{以一至兩句說明此術語。}
_避免使用_：Purchase（購買）、Transaction（交易）

**Invoice（發票）**：
於交付後發送給客戶的付款請求。
_避免使用_：Bill（帳單）、Payment Request（付款請求）

**Customer（客戶）**：
下訂單的個人或組織。
_避免使用_：Client（客戶）、Buyer（買方）、Account（帳戶）
```

## 規則
- **要有立場。** 當同一概念存在多個詞彙時，選定最恰當的一個作為標準術語，其餘列於 `_避免使用_`。
- **定義保持精簡。** 最多一至兩句話。描述它「是什麼」，而不是它「做什麼」。
- **僅納入此專案情境特有的術語。** 一般程式設計概念（例如逾時、錯誤類型、工具模式）即使被大量使用，也不應納入。新增術語前請先確認：這是此情境專屬概念，還是通用程式設計概念？只有前者才應收錄。
- **當自然形成群組時，使用子標題分類。** 若所有術語皆屬於同一個一致領域，則可維持平面列表。

## 單一情境與多重情境儲存庫

**單一情境（大多數儲存庫）：** 在儲存庫根目錄放置一個 `GLOSSARY.md`。

**多重情境：** 在儲存庫根目錄建立一個 `GLOSSARY-MAP.md`，用來列出各情境、其所在位置，以及彼此關係：

```
# 情境地圖

## 情境

- [Ordering](./src/ordering/CONTEXT.md) — 接收並追蹤客戶訂單
- [Billing](./src/billing/CONTEXT.md) — 產生發票並處理付款
- [Fulfillment](./src/fulfillment/CONTEXT.md) — 管理倉庫揀貨與出貨

## 關係

- **Ordering → Fulfillment**：Ordering 發出 `OrderPlaced` 事件；Fulfillment 消費此事件以開始揀貨。
- **Fulfillment → Billing**：Fulfillment 發出 `ShipmentDispatched` 事件；Billing 消費此事件以建立發票。
- **Ordering ↔ Billing**：共用 `CustomerId` 與 `Money` 等型別。
```

此技能會自動推斷應採用哪種結構：
- 若存在` GLOSSARY-MAP.md`，則先讀取該檔案以尋找各情境。
- 若僅存在根目錄 `GLOSSARY.md`，則視為單一情境。
- 若兩者皆不存在，則在首次解析術語時延遲建立根目錄 `GLOSSARY.md`。

當存在多個情境時，推斷目前主題所屬的情境；若無法判定，再進一步詢問。