# cgmh_oracle
# 📦 Oracle 作業說明書 — 外購暫估成本資料產生

## 🧾 作業名稱
**產生外購收料的暫估費用資料**

## 🧠 程式目的
本程式主要針對兩個月前的外購收料資料 (`RSUPRECV`)，依照特定條件與費用項目 (`FID`)，檢查是否已有暫估記錄，若無則依規則寫入 `RFPRMAIN` 表中，並補上暫估金額。

---

## 📂 程式架構

### 1. 宣告區（Declarations）
- `SYS`: 儲存系統日期
- `CurRecv`: 查詢兩個月前符合條件的收料資料（條件：`NL = '9'`、`ORDID = '1'`）
- `CurEITM`: 組合 GPA1、GPAF 及手動定義的暫估項目（AK、AX、AL）
- `CurTPEST`: 查詢已存在但暫估金額為 0 的紀錄

---

### 2. `Function checkEst(...)`
- **用途**：檢查是否已有暫估費用資料
- **傳回值**：
  - `0`: 未存在
  - `>0`: 已存在

---

### 3. `Function getTpEstAmt(...)`
- **用途**：依據 `FID` 取得預設的暫估金額
- **來源資料表**：`RFprexch`
- **例外處理**：查不到資料時回傳 `-2`

---

### 4. `Procedure insertEst(...)`
- **用途**：根據邏輯產生並寫入暫估金額至 `RFPRMAIN`
- **判斷邏輯**：
  - 若為 `AK`、`AF`、`AX`、`AL`：使用 `getTpEstAmt` 抓預設金額
  - 若為 `A2`、`AA`、`AB`、`An`、`AM`、`AS`：設為需 PES（不進行寫入）
  - 其他：從 `RSUPIMSD` 抓取金額，乘上匯率 (`EXR`)，寫入暫估

---

### 5. 主程式流程
1. 記錄當前系統日期至 `SYS`
2. 逐筆讀取 `CurRecv` 中的收料資料
3. 對每筆收料資料，跑所有暫估項目 `CurEITM`
   - 呼叫 `checkEst()` 判斷是否已有資料
   - 若無則呼叫 `insertEst()` 插入資料
   - 若已有資料且暫估金額為 0，則用 `CurTPEST` 更新金額為 `TPBRWAMT`

---

## 📝 輸出內容（`DBMS_OUTPUT`）
- 印出處理進度，如：
  - `12345678暫估AK`（新建立）
  - `12345678AK已經暫估`（已存在）

---

## ⚠️ 錯誤處理
- 使用 `WHEN OTHERS THEN` 捕捉任何例外，並印出錯誤訊息 (`SQLERRM`)

---

## 📌 補充說明
- 所有日期欄位統一使用 `2024/05/31` 作為暫估日期 (`TPESTDAT`)
- 查詢的收料日期格式為民國年月（例如：`11303%`）
- 本程式主要與以下資料表互動：
  - `RSUPRECV`（收料主檔）
  - `RFPRMAIN`（暫估主檔）
  - `RFPREXCH`（費用代碼與匯率）
  - `RSUPIMSD`（收料明細）

---

> 📅 作者：中山醫醫資系  
> 💾 執行環境：Oracle PL/SQL  
> 🛠️ 建議使用工具：SQL Developer / TOAD / PL/SQL Developer
