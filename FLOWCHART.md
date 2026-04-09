# 個人記帳簿系統 — 流程圖文件

> 本文件根據 [PRD.md](file:///c:/Users/MB07/111/melody6968806-hue/PRD.md) 與 [ARCHITECTURE.md](file:///c:/Users/MB07/111/melody6968806-hue/ARCHITECTURE.md) 產出，涵蓋系統主要用戶流程與技術流程。

---

## 1. 系統總覽流程

### 1.1 應用啟動流程

```mermaid
flowchart TD
    A([使用者開啟應用]) --> B{LocalStorage<br/>是否有資料?}

    B -->|有| C[讀取 ledger_records]
    B -->|無| D[初始化預設分類]

    C --> E[計算當前餘額]
    D --> F[顯示歡迎畫面]

    E --> G[渲染首頁儀表板]
    F --> H[引導新增第一筆紀錄]

    G --> I([應用就緒])
    H --> I

    style A fill:#4ECDC4,color:#fff,stroke:none
    style I fill:#4ECDC4,color:#fff,stroke:none
    style B fill:#FF8A5C,color:#fff,stroke:none
    style F fill:#A8E6CF,color:#2D3436,stroke:none
    style G fill:#A8E6CF,color:#2D3436,stroke:none
```

---

### 1.2 頁面導航流程

```mermaid
flowchart LR
    subgraph NAV["底部導航列"]
        direction LR
        N1["🏠 首頁"]
        N2["📋 紀錄"]
        N3["📊 統計"]
    end

    subgraph PAGES["頁面視圖"]
        direction TB
        P1["首頁儀表板<br/>#home"]
        P2["歷史紀錄清單<br/>#records"]
        P3["月度統計圖表<br/>#stats"]
    end

    N1 --> P1
    N2 --> P2
    N3 --> P3

    subgraph MODALS["彈窗覆蓋層"]
        M1["新增紀錄 Modal"]
        M2["編輯紀錄 Modal"]
        M3["刪除確認 Dialog"]
    end

    P1 -- "FAB ＋" --> M1
    P2 -- "點擊紀錄" --> M2
    P2 -- "長按/左滑" --> M3

    style NAV fill:#FFF8F0,stroke:#E8E8E8
    style PAGES fill:#FFF8F0,stroke:#E8E8E8
    style MODALS fill:#FFF8F0,stroke:#E8E8E8
```

---

## 2. 核心用戶流程

### 2.1 新增收支紀錄

```mermaid
flowchart TD
    A([點擊 FAB ＋ 按鈕]) --> B[開啟新增紀錄 Modal]
    B --> C{選擇類型}

    C -->|支出| D["載入支出分類<br/>🍜🛒🏠🚗🏥🎓🎉📦"]
    C -->|收入| E["載入收入分類<br/>💰🎁💵📈📦"]

    D --> F[選擇分類]
    E --> F

    F --> G["輸入金額<br/>(數字鍵盤)"]
    G --> H["選擇日期<br/>(預設今天)"]
    H --> I["填寫備註<br/>(選填, ≤50字)"]
    I --> J[點擊「儲存」]

    J --> K{表單驗證}
    K -->|金額為空或 ≤ 0| L["❌ 顯示錯誤<br/>「請輸入有效金額」"]
    K -->|未選分類| M["❌ 顯示錯誤<br/>「請選擇分類」"]
    K -->|驗證通過| N["✅ 建立 Record 物件"]

    L --> G
    M --> F

    N --> O["呼叫 storage.addRecord()"]
    O --> P{LocalStorage<br/>寫入成功?}

    P -->|成功| Q[重新計算餘額]
    P -->|空間不足| R["⚠️ 提示匯出備份<br/>並清理舊紀錄"]

    Q --> S[更新首頁儀表板]
    S --> T[關閉 Modal]
    T --> U[顯示成功動畫 ✨]
    U --> V([回到首頁])

    style A fill:#4ECDC4,color:#fff,stroke:none
    style V fill:#4ECDC4,color:#fff,stroke:none
    style L fill:#FF6B6B,color:#fff,stroke:none
    style M fill:#FF6B6B,color:#fff,stroke:none
    style R fill:#FF6B6B,color:#fff,stroke:none
    style N fill:#6BCB77,color:#fff,stroke:none
    style U fill:#6BCB77,color:#fff,stroke:none
```

---

### 2.2 編輯紀錄

```mermaid
flowchart TD
    A([在紀錄清單中<br/>點擊某筆紀錄]) --> B[開啟編輯 Modal]
    B --> C["預填原有資料<br/>類型/金額/分類/日期/備註"]

    C --> D{使用者修改欄位}
    D --> E[點擊「儲存變更」]

    E --> F{表單驗證}
    F -->|失敗| G[顯示錯誤提示]
    F -->|通過| H{資料是否有變動?}

    G --> D
    H -->|無變動| I[關閉 Modal]
    H -->|有變動| J["呼叫 storage.updateRecord(id, data)"]

    J --> K[重新計算餘額]
    K --> L[更新紀錄清單 UI]
    L --> M[更新首頁儀表板]
    M --> N[關閉 Modal]
    N --> O([回到紀錄頁面])

    D --> P[點擊「取消」]
    P --> I
    I --> O

    style A fill:#4ECDC4,color:#fff,stroke:none
    style O fill:#4ECDC4,color:#fff,stroke:none
    style G fill:#FF6B6B,color:#fff,stroke:none
```

---

### 2.3 刪除紀錄

```mermaid
flowchart TD
    A([長按或左滑紀錄]) --> B[顯示刪除按鈕 🗑️]
    B --> C[點擊刪除按鈕]
    C --> D["彈出確認對話框<br/>「確定要刪除這筆紀錄嗎？<br/>此操作無法復原。」"]

    D --> E{使用者選擇}
    E -->|取消| F[關閉對話框]
    E -->|確認刪除| G["呼叫 storage.deleteRecord(id)"]

    G --> H[從紀錄陣列中移除]
    H --> I[重新計算餘額]
    I --> J[紀錄淡出動畫]
    J --> K[更新紀錄清單]
    K --> L[更新首頁儀表板]
    L --> M([刪除完成])

    F --> N([返回紀錄清單])

    style A fill:#4ECDC4,color:#fff,stroke:none
    style M fill:#FF6B6B,color:#fff,stroke:none
    style N fill:#4ECDC4,color:#fff,stroke:none
    style D fill:#FF8A5C,color:#fff,stroke:none
```

---

### 2.4 查看歷史紀錄

```mermaid
flowchart TD
    A([點擊「📋 紀錄」]) --> B[載入所有紀錄]
    B --> C["按日期降序排列<br/>(最新在上)"]
    C --> D["按日期分組顯示<br/>＋ 每日小計"]

    D --> E{使用者操作}

    E -->|篩選月份| F["選擇月份<br/>(年月選擇器)"]
    E -->|篩選分類| G["選擇分類<br/>(下拉選單)"]
    E -->|篩選類型| H["收入 / 支出<br/>(切換)"]
    E -->|搜尋| I["輸入關鍵字<br/>(備註搜尋)"]
    E -->|向下滾動| J["載入更多紀錄<br/>(每次 20 筆)"]
    E -->|點擊紀錄| K[進入編輯流程]

    F --> L[套用篩選條件]
    G --> L
    H --> L
    I --> M["debounce 300ms"]
    M --> L

    L --> N[重新渲染紀錄列表]
    N --> E

    J --> O[追加渲染紀錄]
    O --> E

    style A fill:#4ECDC4,color:#fff,stroke:none
    style L fill:#A8E6CF,color:#2D3436,stroke:none
```

---

## 3. 資料處理流程

### 3.1 餘額計算流程

```mermaid
flowchart TD
    A[觸發餘額計算] --> B["讀取 storage.getRecords()"]
    B --> C{紀錄數量}

    C -->|0 筆| D["餘額 = NT$ 0"]
    C -->|> 0 筆| E["遍歷所有紀錄<br/>records.reduce()"]

    E --> F{紀錄類型}
    F -->|income| G["balance += amount"]
    F -->|expense| H["balance -= amount"]

    G --> I{還有下一筆?}
    H --> I
    I -->|是| F
    I -->|否| J[計算完成]

    J --> K{餘額正負}
    K -->|≥ 0| L["顯示綠色 🟢<br/>NT$ X,XXX"]
    K -->|< 0| M["顯示紅色 🔴<br/>-NT$ X,XXX"]

    D --> N[更新首頁顯示]
    L --> N
    M --> N

    style A fill:#4ECDC4,color:#fff,stroke:none
    style L fill:#6BCB77,color:#fff,stroke:none
    style M fill:#FF6B6B,color:#fff,stroke:none
```

---

### 3.2 月度摘要計算

```mermaid
flowchart TD
    A["取得當前年月<br/>(year, month)"] --> B["讀取所有紀錄"]
    B --> C["篩選該月紀錄<br/>filter by date"]

    C --> D["分組計算"]

    D --> E["收入總計<br/>Σ income.amount"]
    D --> F["支出總計<br/>Σ expense.amount"]
    D --> G["各分類統計<br/>groupBy category"]

    E --> H["顯示 📈 收入卡片<br/>+NT$ XX,XXX"]
    F --> I["顯示 📉 支出卡片<br/>-NT$ XX,XXX"]
    G --> J["圓餅圖資料<br/>(P1 統計頁)"]

    style H fill:#6BCB77,color:#fff,stroke:none
    style I fill:#FF6B6B,color:#fff,stroke:none
    style J fill:#FF8A5C,color:#fff,stroke:none
```

---

### 3.3 資料存取層流程

```mermaid
flowchart TD
    subgraph APP["業務模組 (records.js / categories.js)"]
        A1["addRecord()"]
        A2["updateRecord()"]
        A3["deleteRecord()"]
        A4["getRecords()"]
    end

    subgraph DAL["資料存取層 (storage.js)"]
        S1["JSON.stringify()"]
        S2["localStorage.setItem()"]
        S3["localStorage.getItem()"]
        S4["JSON.parse()"]
    end

    subgraph LS["LocalStorage"]
        L1["ledger_records"]
        L2["ledger_categories"]
        L3["ledger_settings"]
    end

    A1 --> S1
    A2 --> S1
    A3 --> S1
    S1 --> S2
    S2 --> L1

    A4 --> S3
    S3 --> L1
    L1 --> S4
    S4 --> A4

    S2 --> ERR{寫入失敗?}
    ERR -->|QuotaExceeded| WARN["⚠️ 容量滿提示"]
    ERR -->|成功| OK["✅ 回傳 true"]

    style APP fill:#A8E6CF,color:#2D3436,stroke:none
    style DAL fill:#FFF8F0,color:#2D3436,stroke:#E8E8E8
    style LS fill:#4ECDC4,color:#fff,stroke:none
    style WARN fill:#FF6B6B,color:#fff,stroke:none
    style OK fill:#6BCB77,color:#fff,stroke:none
```

---

## 4. 統計圖表流程（P1）

### 4.1 圓餅圖生成

```mermaid
flowchart TD
    A([切換至統計頁面]) --> B["取得當月紀錄<br/>getMonthSummary()"]
    B --> C["篩選支出紀錄"]
    C --> D["依分類 groupBy"]
    D --> E["計算各分類金額"]

    E --> F{有支出資料?}
    F -->|無| G["顯示空狀態<br/>「本月尚無支出紀錄」"]
    F -->|有| H["計算各分類佔比 %"]

    H --> I["取得 Canvas 2D Context"]
    I --> J["繪製圓餅圖弧形"]
    J --> K["繪製分類圖例"]
    K --> L([圖表渲染完成])

    style A fill:#4ECDC4,color:#fff,stroke:none
    style L fill:#4ECDC4,color:#fff,stroke:none
    style G fill:#636E72,color:#fff,stroke:none
```

---

### 4.2 長條圖生成

```mermaid
flowchart TD
    A["計算近 6 個月資料"] --> B["Month -5"]
    A --> C["Month -4"]
    A --> D["Month -3"]
    A --> E["Month -2"]
    A --> F["Month -1"]
    A --> G["當月"]

    B --> H["各月收入/支出總計"]
    C --> H
    D --> H
    E --> H
    F --> H
    G --> H

    H --> I["計算 Y 軸最大值"]
    I --> J["繪製 X 軸（月份標籤）"]
    J --> K["繪製 Y 軸（金額刻度）"]
    K --> L["繪製收入長條 🟢"]
    L --> M["繪製支出長條 🔴"]
    M --> N([圖表完成])

    style N fill:#4ECDC4,color:#fff,stroke:none
    style L fill:#6BCB77,color:#fff,stroke:none
    style M fill:#FF6B6B,color:#fff,stroke:none
```

---

## 5. 匯出功能流程（P2）

```mermaid
flowchart TD
    A([點擊「匯出紀錄」]) --> B{選擇匯出範圍}

    B -->|全部| C["讀取所有紀錄"]
    B -->|指定月份| D["選擇年月"]

    D --> E["篩選該月紀錄"]
    C --> F["組建 CSV 內容"]
    E --> F

    F --> G["加入 CSV 標頭<br/>日期,類型,分類,金額,備註"]
    G --> H["逐筆轉換為 CSV 行"]
    H --> I["加入 BOM 標記<br/>(Excel 相容)"]
    I --> J["建立 Blob 物件"]
    J --> K["產生下載連結"]
    K --> L["觸發瀏覽器下載<br/>記帳簿_2026-04.csv"]
    L --> M([下載完成 ✅])

    style A fill:#4ECDC4,color:#fff,stroke:none
    style M fill:#6BCB77,color:#fff,stroke:none
```

---

## 6. 錯誤處理流程

```mermaid
flowchart TD
    A[操作觸發] --> B{操作類型}

    B -->|讀取資料| C["storage.getRecords()"]
    B -->|寫入資料| D["storage.saveRecords()"]
    B -->|表單送出| E["validateRecord()"]

    C --> F{JSON.parse 成功?}
    F -->|成功| G["✅ 回傳紀錄陣列"]
    F -->|失敗| H["⚠️ console.error()<br/>回傳空陣列 []"]

    D --> I{容量足夠?}
    I -->|足夠| J["✅ 寫入成功"]
    I -->|QuotaExceeded| K["🔴 彈出提示<br/>「儲存空間已滿」"]

    K --> L["引導匯出 CSV"]
    L --> M["引導清理舊紀錄"]

    E --> N{驗證結果}
    N -->|通過| O["✅ 繼續儲存"]
    N -->|失敗| P["🔴 顯示欄位錯誤"]

    P --> Q["高亮錯誤欄位<br/>+ 提示文字"]

    style G fill:#6BCB77,color:#fff,stroke:none
    style J fill:#6BCB77,color:#fff,stroke:none
    style O fill:#6BCB77,color:#fff,stroke:none
    style H fill:#FF8A5C,color:#fff,stroke:none
    style K fill:#FF6B6B,color:#fff,stroke:none
    style P fill:#FF6B6B,color:#fff,stroke:none
```

---

## 7. 完整用戶旅程

### 7.1 首次使用旅程

```mermaid
flowchart TD
    A([首次開啟應用]) --> B["🎉 歡迎畫面<br/>「開始記錄你的每一筆帳」"]
    B --> C["引導提示<br/>「點擊 ＋ 新增第一筆紀錄」"]
    C --> D[使用者點擊 FAB ＋]
    D --> E[開啟新增 Modal]
    E --> F["選擇：支出 → 🍜 餐飲"]
    F --> G["輸入金額：120"]
    G --> H["備註：午餐便當"]
    H --> I[點擊儲存]
    I --> J["🎊 首筆紀錄完成動畫"]
    J --> K["首頁顯示<br/>餘額 -NT$ 120"]
    K --> L([開始日常使用])

    style A fill:#4ECDC4,color:#fff,stroke:none
    style L fill:#4ECDC4,color:#fff,stroke:none
    style J fill:#FF8A5C,color:#fff,stroke:none
```

---

### 7.2 日常使用旅程

```mermaid
flowchart TD
    A([打開應用]) --> B["查看餘額<br/>NT$ 12,350"]

    B --> C{今天要做什麼?}

    C -->|記帳| D[點擊 ＋ 新增紀錄]
    C -->|查帳| E[切換至紀錄頁]
    C -->|看統計| F[切換至統計頁]

    D --> G["快速記帳<br/>🛒 超市 -580"]
    G --> H["餘額更新<br/>NT$ 11,770"]
    H --> B

    E --> I["依日期查看紀錄"]
    I --> J{需要修改?}
    J -->|是| K[點擊編輯/刪除]
    J -->|否| L[返回首頁]
    K --> H

    F --> M["查看圓餅圖<br/>餐飲 40% 購物 25% ..."]
    M --> N["查看趨勢<br/>近 6 個月收支"]
    N --> L

    L --> B

    style A fill:#4ECDC4,color:#fff,stroke:none
    style H fill:#6BCB77,color:#fff,stroke:none
    style M fill:#FF8A5C,color:#fff,stroke:none
```

---

### 7.3 月底結算旅程

```mermaid
flowchart TD
    A([月底打開應用]) --> B["查看本月摘要<br/>收入 +25,000<br/>支出 -18,200"]

    B --> C[切換至統計頁]
    C --> D["查看支出分佈<br/>🍜 餐飲 35%<br/>🏠 日常規費 30%<br/>🛒 購物 20%<br/>📦 其他 15%"]

    D --> E{分析結果}
    E -->|花費合理| F["✅ 繼續保持"]
    E -->|需要節省| G["🎯 下月目標<br/>減少外食開銷"]

    F --> H{是否備份?}
    G --> H
    H -->|是| I["匯出 CSV<br/>記帳簿_2026-04.csv"]
    H -->|否| J([結束])

    I --> J

    style A fill:#4ECDC4,color:#fff,stroke:none
    style J fill:#4ECDC4,color:#fff,stroke:none
    style D fill:#FF8A5C,color:#fff,stroke:none
    style I fill:#6BCB77,color:#fff,stroke:none
```

---

## 圖表索引

| 編號 | 圖表名稱 | 類型 | 對應功能 |
|------|----------|------|----------|
| 1.1 | 應用啟動流程 | Flowchart | 系統初始化 |
| 1.2 | 頁面導航流程 | Flowchart | SPA 路由 |
| 2.1 | 新增收支紀錄 | Flowchart | F2 |
| 2.2 | 編輯紀錄 | Flowchart | F5 |
| 2.3 | 刪除紀錄 | Flowchart | F5 |
| 2.4 | 查看歷史紀錄 | Flowchart | F4 |
| 3.1 | 餘額計算流程 | Flowchart | F1 |
| 3.2 | 月度摘要計算 | Flowchart | F1 |
| 3.3 | 資料存取層流程 | Flowchart | 架構 |
| 4.1 | 圓餅圖生成 | Flowchart | F6 |
| 4.2 | 長條圖生成 | Flowchart | F6 |
| 5 | 匯出功能流程 | Flowchart | F7 |
| 6 | 錯誤處理流程 | Flowchart | 非功能需求 |
| 7.1 | 首次使用旅程 | Flowchart | 用戶旅程 |
| 7.2 | 日常使用旅程 | Flowchart | 用戶旅程 |
| 7.3 | 月底結算旅程 | Flowchart | 用戶旅程 |
