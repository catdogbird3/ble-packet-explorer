# Bluetooth LE Packet Explorer · 藍牙低功耗封包視覺化工具

> Interactive Bluetooth Low Energy (BLE) advertising packet visualizer｜互動式 BLE 廣告封包結構教學工具

這是一個用 **疊加方塊 + 點擊彈出視窗** 方式，互動式展示 **BLE 廣告封包 (Advertising Packet)** 結構的教學小工具。  
透過點擊 PDU → Header → Payload → AD 結構 → AD Type / Flags，你可以一層一層看到：

- 每個欄位有幾個 bit / byte  
- 各欄位在實際 BLE 廣告封包中的作用  
- 常見 AD Type、Flags（例如 BT_LE_AD_NO_BREDR）怎麼表示  

設計風格偏科技感（藍綠螢光那種 😎），適合用在：
- 自己學 BLE 協議
- 上課／內訓時當示意 Demo
- 撰寫文件／簡報時附上互動連結

---

## 🔗 Demo

> （建議你部署到 GitHub Pages 後，把網址貼在這裡）

- GitHub Pages：  
  `https://你的帳號.github.io/ble-packet-explorer/`

---

## ✨ Features · 功能特色

- 🧱 **互動式疊加方塊視覺化**
  - 主畫面為 PDU 大方塊
  - 點擊 PDU → 展開 Header / Payload
  - 再點 Header / Payload → 展開更細部欄位

- 📦 **完整的廣告 PDU 結構說明**
  - PDU（Protocol Data Unit）
  - Header：PDU Type、RFU、ChSel、TxAdd、RxAdd、Length
  - Payload：AdvA（廣告地址）、AdvData（廣告資料）

- 🧩 **AD 結構（AD0, AD1, …）視覺化**
  - AD = **AD Length + AD Type + AD Data**
  - 清楚標示每一部分的大小與用途
  - 以範例說明如：Flags / BT_LE_AD_NO_BREDR

- 🏷 **常見 AD Type 教學**
  - Complete Local Name（BT_DATA_NAME_COMPLETE）
  - Shortened Local Name（BT_DATA_NAME_SHORTENED）
  - Uniform Resource Identifier（BT_DATA_URI）
  - Service UUID（多種 AD Type 編碼）
  - Manufacturer Specific Data（BT_DATA_MANUFACTURER_DATA）
  - Flags（0x01）

- 🚩 **Flags 詳細解析**
  - BT_LE_AD_LIMITED（有限可發現模式）
  - BT_LE_AD_GENERAL（通用可發現模式）
  - BT_LE_AD_NO_BREDR（不支援 BR/EDR）
  - 各 Flag 所在 bit 位置、用途說明，搭配實際 16 進位與二進位範例

---

## 🧠 BLE 結構總覽（對照本）

### 1. PDU（Protocol Data Unit）

PDU 是 BLE 廣告封包的核心，包含：

- **Header（2 bytes, 16 bits）**
- **Payload（6–37 bytes，可變長度）**

### 2. Header 欄位

Header 總長度 16 bits，主要欄位如下（以廣告 PDU 為例）：

- **PDU Type（4 bits）**
  - 決定廣告類型，例如：`ADV_IND`（可連接、無向廣告）
  - 功能：定義封包用途與行為模式
- **RFU（Reserved for Future Use，2 bits）**
  - 預留未來使用
  - 功能：協議擴展空間
- **ChSel（1 bit）**
  - 若裝置支援 **LE Channel Selection Algorithm #2** 則設為 `1`
  - 功能：告知是否使用改良通道選擇演算法
- **TxAdd（1 bit）**
  - `0`：Public Address  
  - `1`：Random Address  
  - 功能：指示「傳送端」地址類型
- **RxAdd（1 bit）**
  - `0`：Public Address  
  - `1`：Random Address  
  - 功能：指示「接收端」地址類型
- **Length（6 bits）**
  - 表示 Payload 長度（單位：byte）
  - 實際廣告資料部分為 6–37 bytes

> 註：實際 Header bit 配置依 Bluetooth Core Specification 定義，本工具以教學視覺化為主。

---

### 3. Payload 結構

Payload 主要包含：

- **AdvA（Advertiser Address）**
  - 長度：6 bytes（48 bits）
  - 功能：發送廣告的 BLE 裝置地址
- **AdvData（Advertising Data）**
  - 0–31 bytes，可變長度
  - 由多組 **AD 結構（AD0, AD1, AD2, …）** 組成  
    每一組 AD 都有自己的 Length / Type / Data

---

### 4. AD 結構：AD Length + AD Type + AD Data

每一組 AD（例如 AD0）都是：

- **AD Length（1 byte）**
  - 表示後面 `AD Type + AD Data` 的總長度
  - 功能：讓解析器知道這一筆 AD 的範圍，才能找到下一筆

- **AD Type（1 byte）**
  - 表示 AD Data 的類型
  - 功能：告訴解析端「這筆資料應該怎麼解讀」
  - 由藍牙規範定義，在 nRF Connect SDK 中可參考  
    `EIR/AD data type definitions`

- **AD Data（N bytes）**
  - 長度：`AD Length - 1`
  - 內容格式視 AD Type 而定

#### 範例：Flags AD 結構設定 BT_LE_AD_NO_BREDR

| 欄位      | 值   | 說明           |
|----------|------|----------------|
| Length   | 0x02 | 1 byte Type + 1 byte Data |
| Type     | 0x01 | Flags          |
| Data     | 0x04 | BT_LE_AD_NO_BREDR |

---

### 5. 常見 AD Type

本工具中示範了幾個常見 AD Type：

- **Complete Local Name – `BT_DATA_NAME_COMPLETE (0x09)`**
  - 完整的裝置名稱
  - 使用者掃描裝置時看到的名字（例如手機掃描附近藍牙）

- **Shortened Local Name – `BT_DATA_NAME_SHORTENED (0x08)`**
  - 完整名稱的縮寫版本
  - 用於節省廣告空間

- **Uniform Resource Identifier – `BT_DATA_URI (0x24)`**
  - 廣告 URI，例如網站 URL

- **Service UUID – 多種 Type 值**
  - 對特定服務具有全球唯一性的 ID
  - 可幫助掃描器判斷是否要連線

- **Manufacturer Specific Data – `BT_DATA_MANUFACTURER_DATA (0xFF)`**
  - 製造商自訂資料
  - 例如：iBeacon 格式即使用此類型

- **Flags – `0x01`**
  - 封裝在一個 byte 中的一組 bit flags（最多 8 個）

---

### 6. 廣告 Flags

Flags 放在 AD Data 中，常見有：

- **BT_LE_AD_LIMITED**
  - 設定「有限可發現模式 (LE Limited Discoverable Mode)」
  - 用於可連接廣告，代表裝置僅在一段有限時間提供連線

- **BT_LE_AD_GENERAL**
  - 設定「通用可發現模式 (LE General Discoverable Mode)」
  - 用於可連接廣告，代表長時間可被掃描（time-out = 0）

- **BT_LE_AD_NO_BREDR**
  - 代表裝置不支援經典藍牙（BR/EDR），只支援 BLE

> BT_LE_AD_LIMITED 和 BT_LE_AD_GENERAL 主要是給 **Peripheral 角色** 使用，用來控制「裝置有多容易被發現」。

---

## 🏗 Tech Stack · 技術棧

- **HTML5**：單一頁面（Single Page, 靜態）
- **Tailwind CSS（CDN 版）**：快速打造科技感 UI
- **原生 JavaScript**：
  - `openModal()` / `closeModal()` 控制浮窗
  - 透過 `modal-overlay` + CSS transition 做動畫
- **無後端 / 無 build 步驟**
  - 直接用靜態伺服器（或 GitHub Pages）就能跑

---

## 📁 專案結構

目前專案可以非常簡單：

```bash
.
├─ index.html   # 互動式 BLE 封包視覺化主頁
└─ README.md    # 專案說明文件（本檔）
