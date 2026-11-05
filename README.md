# SmartBank 智慧銀行系統

以 **JavaFX + FXML** 實作的模擬銀行系統，支援多銀行、開戶、網路銀行、線上 ATM 與管理員後台

## 技術棧

| 項目 | 版本 |
| --- | --- |
| Java | 24 |
| JavaFX | 21.0.6 |

## 功能特色

### 多銀行架構

- 內建兩家銀行：`A銀行`、`B銀行`，彼此帳戶與使用者資料獨立。
- 每種操作都需先選擇銀行，跨行轉帳會依金額收取手續費。

### 開戶

- 輸入姓名、帳號、密碼、6 位數 PIN 碼並選擇銀行。
- 開戶成功自動產生：
  - **14 碼純數字帳戶**
  - **16 碼純數字卡號**（DebitCard，與帳戶一對一綁定）
- 同一銀行內帳號不可重複；PIN 碼強制為 6 位數字。

### 網路銀行

- 以「銀行 + 帳號 + 密碼」登入。
- 來源帳戶可從下拉選單切換（同一使用者可有多個帳戶）。
- 支援：**存款、提款、查詢餘額、轉帳、交易紀錄、新增帳戶**。
- 新增帳戶時會跳出對話框要求設定 6 位數 PIN 碼，完成後產生新帳戶與新卡號。

### 線上 ATM

- 以「銀行 + 卡號 + PIN 碼」登入，直接對應卡片綁定的帳戶。
- 支援：**存款、提款、查詢餘額、跨行／同行轉帳、交易紀錄**。

### 管理員後台

- 以各銀行內建的 `admin` 帳號登入。
- 以 `TreeView` 呈現「使用者 → 帳戶 → 卡號」階層。
- 對帳戶或卡號節點按右鍵可**複製**純數字編號，方便測試。

### 交易紀錄

- 每筆交易記錄型別、金額、手續費、交易後餘額、時間（毫秒精度）與對方帳號。
- 型別包含 `DEPOSIT`、`WITHDRAW`、`TRANSFER_IN`、`TRANSFER_OUT`。

## 快速開始

```pwsh
# Windows
.\mvnw.cmd clean javafx:run
```

```bash
# macOS / Linux
./mvnw clean javafx:run
```

若已安裝全域 Maven，也可使用 `mvn clean javafx:run`

### 其他常用指令

```pwsh
.\mvnw.cmd clean package     # 建置
.\mvnw.cmd clean compile     # 編譯
.\mvnw.cmd test              # 測試
```

## 預設帳號

系統啟動時會為**每一家銀行**自動建立管理員帳號：

| 銀行 | 帳號 | 密碼 | 用途 |
| --- | --- | --- | --- |
| A銀行 | `admin` | `admin` | 登入管理員後台 |
| B銀行 | `admin` | `admin` | 登入管理員後台 |

一般使用者需先在登入頁點「開戶」自行註冊，註冊完成後畫面會顯示產生的**帳戶號碼**與**卡號**

## 使用流程

1. **開戶**：登入頁 →「開戶」→ 填寫資料 → 取得帳戶號碼與卡號
2. **網路銀行**：模式選「網路銀行」→ 選銀行 → 輸入帳號／密碼 → 存款、提款、轉帳、看交易紀錄
3. **線上 ATM**：模式選「線上 ATM」→ 選銀行 → 輸入卡號／PIN 碼 → 操作該卡綁定的帳戶
4. **管理員**：以 `admin`／`admin` 登入 → 瀏覽所有使用者、帳戶與卡號
5. **登出**：各頁面的「登出」會回到登入頁

## 轉帳手續費規則

手續費僅在**跨行轉帳**時收取，同行轉帳免費：

| 跨行轉帳金額 | 手續費 |
| --- | --- |
| ≤ 500 | 0 |
| 501 ～ 1000 | 10 |
| > 1000 | 15 |

先計算手續費，再從來源帳戶扣款 `金額 + 手續費`，成功後才將「金額」存入目標帳戶並寫入雙方交易紀錄，餘額不足時整筆交易失敗不會扣款

## 專案結構

```text
smart-bank/
├── pom.xml
├── mvnw / mvnw.cmd
└── src/main/
    ├── java/
    │   ├── module-info.java                  # 模組宣告（requires javafx.*、opens controller）
    │   └── io/github/swient/smartbank/
    │       ├── Launcher.java                 # JavaFX 進入點（Application.launch）
    │       ├── BankApplication.java          # 載入 login.fxml、設定主視窗
    │       ├── controller/
    │       │   ├── LoginController.java      # 模式切換、網銀／ATM／管理員登入、前往開戶
    │       │   ├── RegisterController.java   # 開戶與 PIN 碼驗證
    │       │   ├── NetBankController.java    # 網路銀行功能
    │       │   ├── ATMController.java        # 線上 ATM 功能
    │       │   └── AdminController.java      # 管理員使用者樹與複製功能
    │       ├── model/
    │       │   ├── account/                  # User、Account、Transaction
    │       │   ├── bank/                     # Bank、ATM
    │       │   └── card/                     # BankCard、DebitCard
    │       └── service/
    │           ├── BankService.java          # 管理多家銀行
    │           └── UserService.java          # 註冊／驗證／SHA-256 雜湊
    └── resources/io/github/swient/smartbank/view/
        ├── login.fxml
        ├── register.fxml
        ├── netbank.fxml
        ├── atm.fxml
        └── admin.fxml
```
