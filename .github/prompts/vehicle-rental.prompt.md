---
mode: 'agent'
description: '車輛租用命令：依照 RentalCar 系統需求，產生車輛租用相關的領域模型、應用服務、CQRS Command、Controller 及 View。'
---

# 車輛租用命令

## 目標
依照 `README.md` 的 RentalCar 線上預先車輛租用系統需求，實作「車輛租用」功能模組，涵蓋車型選擇、租用時間區間設定、租金計算與確認租用等核心功能。

## 系統背景
- 租車用戶（Actor）已完成帳號註冊並登入後，可透過呼叫 `ToRentalCar()` 方法線上租用車輛。
- 可選擇的車型與對應日租金如下：
  | 車型 | 英文識別碼 | 日租金（元/天） |
  |------|-----------|--------------|
  | 轎車 | Car | 1,000 |
  | 休旅車 | SUV | 1,500 |
  | 貨車 | Truck | 2,000 |
  | 跑車 | SportsCar | 3,000 |
  | 電動車 | ElectricCar | 2,800 |
- 整體架構採用 **ASP.NET Core 9 MVC** + **CQRS** 模式，UI 使用 **Bootstrap 5.1** 實作 RWD 介面。

## 請完成下列工作項目

### 1. 領域模型（Domain Model）

#### 車型列舉（VehicleType Enum）
```
Car = 1000, SUV = 1500, Truck = 2000, SportsCar = 3000, ElectricCar = 2800
```

#### RentalOrder（租用訂單 Entity）
屬性：
- `OrderId`（Guid）
- `AccountId`（Guid）：關聯租車用戶
- `VehicleType`（VehicleType enum）：選擇的車型
- `StartDate`（DateOnly）：租用起始日期
- `EndDate`（DateOnly）：租用結束日期
- `TotalDays`（int）：計算租用天數，唯讀
- `TotalFee`（decimal）：計算總租金，唯讀
- `CreatedAt`（DateTime）

領域方法：
- `ToRentalCar(accountId, vehicleType, startDate, endDate)`：建立租用訂單，驗證日期區間合法性。
- `CalculateFee()`：依車型日租金 × 租用天數計算總租金，回傳 `TotalFee`。

### 2. CQRS Command 與 Handler
- 建立 **CreateRentalOrderCommand**：
  - 屬性：`AccountId`、`VehicleType`、`StartDate`、`EndDate`
  - Handler（`CreateRentalOrderCommandHandler`）：
    1. 驗證輸入（日期區間、車型合法性）
    2. 呼叫 `RentalOrder.ToRentalCar()` 建立訂單
    3. 呼叫 `RentalOrder.CalculateFee()` 計算租金
    4. 儲存訂單至資料庫
    5. 回傳 `OrderId` 與 `TotalFee`

- 建立 **ConfirmRentalOrderCommand**：
  - 屬性：`OrderId`、`AccountId`
  - Handler（`ConfirmRentalOrderCommandHandler`）：查詢並確認指定訂單，更新訂單狀態為「已確認」。

### 3. Application Service
- 建立 `RentalCarApplicationService`，對外公開下列方法：
  - `CreateRentalOrderAsync(CreateRentalOrderCommand command)`：建立租用訂單並計算租金。
  - `ConfirmRentalOrderAsync(ConfirmRentalOrderCommand command)`：確認訂單。
  - `GetVehicleTypesAsync()`：查詢可用車型清單（含日租金）。

### 4. MVC Controller
- 建立 `RentalCarController`，包含以下 Action：
  - `GET  /RentalCar/Index`   → 顯示車型選擇頁（需登入）
  - `POST /RentalCar/Create`  → 接收車型、起訖日期，呼叫 `CreateRentalOrderAsync()`，導向租金確認頁
  - `GET  /RentalCar/Confirm/{orderId}` → 顯示訂單明細與總租金確認頁
  - `POST /RentalCar/Confirm` → 呼叫 `ConfirmRentalOrderAsync()`，確認租用，導向完成頁
  - `GET  /RentalCar/Complete/{orderId}` → 顯示租用完成頁

### 5. Razor View（cshtml）
使用 **Bootstrap 5.1** 設計以下 RWD 頁面：
- `Index.cshtml`：車型選擇頁，以卡片（Card）方式展示每種車型名稱與日租金，含「選擇」按鈕。
- `Create.cshtml`：租用時間設定頁，包含「起始日期」與「結束日期」日期選擇器（Date Picker），並即時顯示預估天數與租金（JavaScript 動態計算）。
- `Confirm.cshtml`：確認租用頁，顯示訂單摘要（車型、起訖日期、天數、總租金），提供「確認租用」與「取消」按鈕。
- `Complete.cshtml`：租用完成頁，顯示訂單編號與成功訊息。

### 6. Mermaid 圖表（請附加至 README.md 或獨立文件）
- **Use Case Diagram**：顯示租車用戶（Actor）與「選擇車型」、「設定租用時間」、「計算租金」、「確認租用」Use Case 的關係。
- **Sequence Diagram**：描述 `ToRentalCar()` 的完整呼叫流程，從 Actor → `RentalCarController` → `RentalCarApplicationService` → `CreateRentalOrderCommandHandler` → `RentalOrder`（Domain）→ Repository → 回傳 `TotalFee`。
- **Class Diagram**：顯示 `RentalOrder`、`VehicleType`、`CreateRentalOrderCommand`、`ConfirmRentalOrderCommand`、`RentalCarApplicationService`、`RentalCarController` 之間的關聯。

## 技術規範
- 框架：ASP.NET Core 9 MVC
- UI 元件庫：Bootstrap 5.1
- 架構模式：CQRS（Command Query Responsibility Segregation）
- 語言：C#
- 所有 Mermaid 圖表須以 ` ```mermaid ``` ` 語法包裹，放置於 Markdown 文件中。
- 程式碼需符合 Clean Architecture 分層原則（Domain / Application / Infrastructure / Presentation）。
- `RentalCarController` 的每個需要登入的 Action 須套用 `[Authorize]` 屬性。
