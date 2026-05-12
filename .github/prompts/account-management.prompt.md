---
mode: 'agent'
description: '用戶登入帳號管理命令：依照 RentalCar 系統需求，產生帳號管理相關的領域模型、應用服務、CQRS Command、Controller 及 View。'
---

# 用戶登入帳號管理命令

## 目標
依照 `README.md` 的 RentalCar 線上預先車輛租用系統需求，實作「用戶帳號管理」功能模組，涵蓋帳號註冊（RegisterAccount）與帳號登入（LoginAccount）兩項核心功能。

## 系統背景
- 租車用戶（Actor）在使用線上租車服務之前，必須先完成帳號註冊，再進行登入，才能預先租用車輛。
- 帳號管理對應的領域物件為 **Account**（或 **Customer**）。
- 整體架構採用 **ASP.NET Core 9 MVC** + **CQRS** 模式，UI 使用 **Bootstrap 5.1** 實作 RWD 介面。

## 請完成下列工作項目

### 1. 領域模型（Domain Model）
- 建立 `Account` 領域物件（Domain Entity），包含以下屬性：
  - `AccountId`（Guid）
  - `UserName`（string）
  - `Email`（string）
  - `PasswordHash`（string）
  - `CreatedAt`（DateTime）
- 提供以下領域方法：
  - `RegisterAccount(userName, email, password)`：建立新帳號，對密碼進行雜湊處理。
  - `ValidatePassword(password)`：驗證密碼是否正確。

### 2. CQRS Command 與 Handler
- 建立 **RegisterAccountCommand**：
  - 屬性：`UserName`、`Email`、`Password`、`ConfirmPassword`
  - Handler（`RegisterAccountCommandHandler`）：驗證輸入、檢查 Email 是否重複、呼叫 `Account.RegisterAccount()`、儲存至資料庫。
- 建立 **LoginAccountCommand**：
  - 屬性：`Email`、`Password`
  - Handler（`LoginAccountCommandHandler`）：查詢帳號、呼叫 `Account.ValidatePassword()`、建立登入 Session 或 JWT Token。

### 3. Application Service
- 建立 `AccountApplicationService`，對外公開下列方法：
  - `RegisterAsync(RegisterAccountCommand command)`
  - `LoginAsync(LoginAccountCommand command)`

### 4. MVC Controller
- 建立 `AccountController`，包含以下 Action：
  - `GET  /Account/Register` → 顯示註冊表單 View
  - `POST /Account/Register` → 接收表單資料，呼叫 `RegisterAsync()`，成功後導向登入頁
  - `GET  /Account/Login`    → 顯示登入表單 View
  - `POST /Account/Login`    → 接收表單資料，呼叫 `LoginAsync()`，成功後導向車輛租用頁
  - `GET  /Account/Logout`   → 清除登入狀態，導向首頁

### 5. Razor View（cshtml）
使用 **Bootstrap 5.1** 設計以下 RWD 頁面：
- `Register.cshtml`：帳號註冊表單，欄位包含「使用者名稱」、「電子郵件」、「密碼」、「確認密碼」，含用戶端驗證。
- `Login.cshtml`：帳號登入表單，欄位包含「電子郵件」、「密碼」，含「記住我」選項。

### 6. Mermaid 圖表（請附加至 README.md 或獨立文件）
- **Use Case Diagram**：顯示租車用戶（Actor）與「註冊帳號」、「登入帳號」Use Case 的關係。
- **Sequence Diagram**：描述 `RegisterAccount()` 與 `LoginAccount()` 的完整呼叫流程，從 Actor → Controller → ApplicationService → CommandHandler → Domain → Repository。
- **Class Diagram**：顯示 `Account` 領域物件、`RegisterAccountCommand`、`LoginAccountCommand`、`AccountApplicationService`、`AccountController` 之間的關聯。

## 技術規範
- 框架：ASP.NET Core 9 MVC
- UI 元件庫：Bootstrap 5.1
- 架構模式：CQRS（Command Query Responsibility Segregation）
- 語言：C#
- 所有 Mermaid 圖表須以 ` ```mermaid ``` ` 語法包裹，放置於 Markdown 文件中。
- 程式碼需符合 Clean Architecture 分層原則（Domain / Application / Infrastructure / Presentation）。
