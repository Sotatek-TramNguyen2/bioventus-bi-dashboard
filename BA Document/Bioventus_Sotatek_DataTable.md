# Bioventus <> Sotatek — Data Table Specification

---

## Document Information

### Author

| Author | Position | Division |
|--------|----------|----------|
| Tram (Sofia) Nguyen | BA Lead | GEX 1 |

### Adjust History

| Date | Version | Description | Editor |
|------|---------|-------------|--------|
| 18 May 2026 | 1.0 | New | Tram (Sofia) Nguyen |

### Approval

| Approved date | Version | Approved by | Position |
|---------------|---------|-------------|----------|
| 18 May 2026 | 1.0 | Bryan Du | CDO |

---

## 1. TỔNG QUAN MÔ HÌNH DỮ LIỆU

### 1.1. Kiến trúc Star Schema

Mô hình chia 2 vùng nghiệp vụ, dùng chung `Dim_Date`:

```
═══════════════════════════════════════════════════════════════════════
  VÙNG 1: JIRA PERFORMANCE                    VÙNG 2: SAP PARTNERS
═══════════════════════════════════════════════════════════════════════

┌─────────────┐                               ┌─────────────────────┐
│  Dim_User   │──┐                            │ Dim_Partner_Account │
└─────────────┘  │                            └──────────┬──────────┘
                 │  ┌──────────────────┐                 │
┌─────────────┐  ├──┤ Fact_JiraTicket  │                 │
│ Dim_Status  │──┘  └────────┬─────────┘                 │
└─────────────┘              │              ┌────────────┴──────────┐
                             │              │Fact_PartnerTransaction│
┌───────────────┐            │              └────────────┬──────────┘
│Dim_RequestType│────────────┘                           │
└───────────────┘                                        │
                    ┌──────────────┐                     │
                    │   Dim_Date   │─────────────────────┘
                    └──────────────┘
                     (Dùng chung)
```

### 1.2. Tổng hợp tables

| # | Table Name | Type | Vùng nghiệp vụ | Số dòng dự kiến |
|---|-----------|------|-----------------|-----------------|
| 1 | Dim_User | Dimension | Jira | ~20–30 |
| 2 | Dim_Status | Dimension | Jira | 7 |
| 3 | Dim_RequestType | Dimension | Jira | 4–6 |
| 4 | Dim_Date | Dimension | Shared | ~3,650 (10 năm) |
| 5 | Dim_Partner_Account | Dimension | SAP | ~200–500 |
| 6 | Fact_JiraTicket | Fact | Jira | ~600–1,000 |
| 7 | Fact_PartnerTransaction | Fact | SAP | ~5,000–10,000 |

---

## 2. DIMENSION TABLES — CHI TIẾT

### 2.1. `Dim_User`

**Ý nghĩa:** Lưu thông tin nhân sự, đóng vai trò **Role-Playing Dimension** — cùng 1 bảng phục vụ cả vai trò Assignee (người xử lý) và Reporter (người tạo ticket).

**Phục vụ charts:** Ticket by Assignee, Ticket by Reporter, Detail Summary Table

| Column Name | Data Type | Key | Nullable | Description |
|-------------|-----------|-----|----------|-------------|
| UserKey | Integer | PK | No | Surrogate Key tự tăng |
| UserName | String(150) | | No | Tên nhân sự (VD: Rahime Cvetkovic, Koh Higaki) |
| Department | String(100) | | Yes | Phòng ban (VD: BI Engineering, Finance, Operations) |
| Email | String(200) | | Yes | Email nhân sự |

**Setup trong Power BI:**
- Tạo relationship Active tới `Fact_JiraTicket[AssigneeKey]`
- Tạo relationship Inactive tới `Fact_JiraTicket[ReporterKey]`
- Hoặc: Duplicate thành bảng `Dim_Reporter` (Reference query trong Power Query) để user kéo thả dễ hơn

<!-- PLACEHOLDER_DIM_STATUS -->

---

### 2.2. `Dim_Status`

**Ý nghĩa:** Quản lý luồng trạng thái công việc. Cho phép phát hiện bottleneck (ticket bị kẹt ở trạng thái nào lâu nhất).

**Phục vụ charts:** Ticket by Status (Bar Chart), Status Slicer

| Column Name | Data Type | Key | Nullable | Description |
|-------------|-----------|-----|----------|-------------|
| StatusKey | Integer | PK | No | Khóa chính |
| StatusName | String(50) | | No | Tên trạng thái |
| StatusCategory | String(30) | | No | Nhóm trạng thái logic |

**Giá trị mẫu:**

| StatusKey | StatusName | StatusCategory |
|-----------|-----------|----------------|
| 1 | New | Open |
| 2 | In Progress | In Work |
| 3 | Development | In Work |
| 4 | QA UAT | In Work |
| 5 | PROD/UAT | In Work |
| 6 | On Hold | Open |
| 7 | Done | Closed |

**Setup trong Power BI:**
- Relationship 1:* tới `Fact_JiraTicket[StatusKey]`, Single direction, Active
- Dùng làm Slicer filter trên tất cả pages

---

### 2.3. `Dim_RequestType`

**Ý nghĩa:** Phân loại công việc để biết team BI đang tốn nguồn lực vào đâu. Giúp management quyết định resource allocation.

**Phục vụ charts:** Ticket by Request Type (Horizontal Bar), Avg Days to Complete by Request Type

| Column Name | Data Type | Key | Nullable | Description |
|-------------|-----------|-----|----------|-------------|
| RequestTypeKey | Integer | PK | No | Khóa chính |
| RequestTypeName | String(100) | | No | Tên loại request |

**Giá trị mẫu:**

| RequestTypeKey | RequestTypeName |
|----------------|-----------------|
| 1 | Update Existing Report/Dataset |
| 2 | App Permission |
| 3 | New Report |
| 4 | Report an Issue |
| 5 | New Dataset |
| 6 | Data Quality Investigation |

**Setup trong Power BI:**
- Relationship 1:* tới `Fact_JiraTicket[RequestTypeKey]`, Single direction, Active

---

### 2.4. `Dim_Date`

**Ý nghĩa:** Master Calendar dùng chung cho cả 2 vùng nghiệp vụ. Là **Role-Playing Dimension** cho Date — cùng 1 bảng phục vụ cả CreatedDate và ResolvedDate.

**Phục vụ charts:** Ticket by Created Date (Column Chart), Time slicers (Year, Month), Trend analysis

| Column Name | Data Type | Key | Nullable | Description |
|-------------|-----------|-----|----------|-------------|
| DateKey | Integer | PK | No | Định dạng YYYYMMDD (VD: 20260519) |
| Date | Date | | No | Ngày cụ thể |
| Year | Integer | | No | Năm (2017–2026) |
| MonthNo | Integer | | No | Số thứ tự tháng (1–12) |
| MonthName | String(20) | | No | Tên tháng (January, February...) |
| Quarter | Integer | | No | Quý (1–4) |
| WeekOfYear | Integer | | No | Tuần trong năm |
| DayOfWeek | Integer | | No | Ngày trong tuần (1=Monday) |
| FiscalYear | Integer | | Yes | Năm tài chính (nếu lệch calendar year) |
| IsWeekday | Boolean | | No | True nếu ngày làm việc |

**Setup trong Power BI:**
- Relationship Active tới `Fact_JiraTicket[CreatedDateKey]` — mặc định cho Year/Month slicer
- Relationship Inactive tới `Fact_JiraTicket[ResolvedDateKey]` — dùng `USERELATIONSHIP()` khi cần
- Relationship Active tới `Fact_PartnerTransaction[OrderDateKey]`
- Mark as Date Table trong Power BI để enable Time Intelligence functions
- Có thể generate bằng DAX: `CALENDARAUTO()` hoặc Power Query script

**Power Query script tạo Dim_Date:**
```powerquery
let
    StartDate = #date(2017, 1, 1),
    EndDate = #date(2026, 12, 31),
    NumberOfDays = Duration.Days(EndDate - StartDate) + 1,
    DateList = List.Dates(StartDate, NumberOfDays, #duration(1, 0, 0, 0)),
    TableFromList = Table.FromList(DateList, Splitter.SplitByNothing(), {"Date"}),
    ChangedType = Table.TransformColumnTypes(TableFromList, {{"Date", type date}}),
    AddDateKey = Table.AddColumn(ChangedType, "DateKey", each Number.From(Date.ToText([Date], "yyyyMMdd")), Int64.Type),
    AddYear = Table.AddColumn(AddDateKey, "Year", each Date.Year([Date]), Int64.Type),
    AddMonthNo = Table.AddColumn(AddYear, "MonthNo", each Date.Month([Date]), Int64.Type),
    AddMonthName = Table.AddColumn(AddMonthNo, "MonthName", each Date.MonthName([Date]), type text),
    AddQuarter = Table.AddColumn(AddMonthName, "Quarter", each Date.QuarterOfYear([Date]), Int64.Type),
    AddWeekOfYear = Table.AddColumn(AddQuarter, "WeekOfYear", each Date.WeekOfYear([Date]), Int64.Type),
    AddDayOfWeek = Table.AddColumn(AddWeekOfYear, "DayOfWeek", each Date.DayOfWeek([Date], Day.Monday) + 1, Int64.Type),
    AddIsWeekday = Table.AddColumn(AddDayOfWeek, "IsWeekday", each Date.DayOfWeek([Date], Day.Monday) < 5, type logical)
in
    AddIsWeekday
```

---

### 2.5. `Dim_Partner_Account`

**Ý nghĩa:** Lưu thông tin đối tác trong hệ sinh thái Healthcare của Bioventus. Đồng bộ từ SAP tables `BUT000` (Business Partner Master) và `KNA1` (Customer Master).

**Phục vụ charts:** Partners by Account Type (Horizontal Bar Chart)

| Column Name | Data Type | Key | Nullable | Description |
|-------------|-----------|-----|----------|-------------|
| PartnerKey | Integer | PK | No | Surrogate Key |
| PartnerID | String(20) | | No | Mã đối tác trên SAP (VD: BP-001234) |
| PartnerName | String(200) | | No | Tên đối tác |
| AccountType | String(50) | | No | Phân loại đối tác |
| Region | String(50) | | Yes | Vùng địa lý |
| PartnerFunction | String(30) | | Yes | Vai trò SAP SD: Sold-to, Ship-to, Bill-to, Payer |

**Giá trị mẫu cho AccountType:**

| AccountType | Mô tả | Ví dụ |
|-------------|--------|-------|
| Patients | Bệnh nhân cuối | John Smith (end user) |
| Hospitals | Bệnh viện | Mayo Clinic, Johns Hopkins |
| Doctors/Physicians | Bác sĩ phẫu thuật | Dr. Sarah Johnson |
| Clinics | Phòng khám chuyên khoa | Orthopedic Associates |
| Insurance/Payer | Công ty bảo hiểm | UnitedHealth, Aetna |
| Distributor | Đại lý phân phối | McKesson, Cardinal Health |
| ASC | Ambulatory Surgery Center | SurgCenter of Atlanta |

**Setup trong Power BI:**
- Relationship 1:* tới `Fact_PartnerTransaction[PartnerKey]`, Single direction, Active
- Dùng `AccountType` làm trục Y trong Horizontal Bar Chart
- Count of PartnerKey làm trục X

---

## 3. FACT TABLES — CHI TIẾT

### 3.1. `Fact_JiraTicket`

**Ý nghĩa:** Bảng sự kiện chính cho phân hệ Jira — mỗi dòng = 1 ticket. Lưu toàn bộ lifecycle của ticket từ lúc tạo đến lúc đóng.

**Phục vụ charts:** Total Tickets (KPI Card), Ticket by Assignee, Ticket by Status, Ticket by Request Type, Ticket by Created Date, Ticket by Reporter, Avg Days to Complete, Detail Summary Table

| Column Name | Data Type | Key | Nullable | Description |
|-------------|-----------|-----|----------|-------------|
| TicketKey | Integer | PK | No | Surrogate Key |
| IssueKey | String(20) | | No | Mã ticket Jira (VD: BI-1411, BI-0523) |
| Summary | String(500) | | No | Tiêu đề ticket |
| AssigneeKey | Integer | FK → Dim_User | No | Người được giao xử lý |
| ReporterKey | Integer | FK → Dim_User | No | Người tạo/báo cáo ticket |
| StatusKey | Integer | FK → Dim_Status | No | Trạng thái hiện tại |
| RequestTypeKey | Integer | FK → Dim_RequestType | No | Loại yêu cầu |
| CreatedDateKey | Integer | FK → Dim_Date | No | Ngày tạo ticket |
| ResolvedDateKey | Integer | FK → Dim_Date | Yes | Ngày đóng ticket (NULL nếu chưa done) |
| DaysToComplete | Integer | | Yes | Số ngày xử lý = ResolvedDate - CreatedDate |

**Setup trong Power BI:**

1. **Relationships:**
   - `AssigneeKey` → `Dim_User.UserKey` (Active)
   - `ReporterKey` → `Dim_User.UserKey` (Inactive)
   - `StatusKey` → `Dim_Status.StatusKey` (Active)
   - `RequestTypeKey` → `Dim_RequestType.RequestTypeKey` (Active)
   - `CreatedDateKey` → `Dim_Date.DateKey` (Active)
   - `ResolvedDateKey` → `Dim_Date.DateKey` (Inactive)

2. **DAX Measures cần tạo:**

```dax
[Total Tickets] = COUNTROWS(Fact_JiraTicket)

[Avg Days to Complete] = AVERAGE(Fact_JiraTicket[DaysToComplete])

[Ticket Age (Days)] = 
    IF(
        ISBLANK(Fact_JiraTicket[ResolvedDateKey]),
        DATEDIFF(RELATED(Dim_Date[Date]), TODAY(), DAY),
        BLANK()
    )

// Kích hoạt Inactive relationship cho Reporter
[Tickets by Reporter Context] = 
    CALCULATE(
        COUNTROWS(Fact_JiraTicket),
        USERELATIONSHIP(Dim_User[UserKey], Fact_JiraTicket[ReporterKey])
    )

// Kích hoạt Inactive relationship cho Resolved Date
[Tickets Resolved in Period] = 
    CALCULATE(
        COUNTROWS(Fact_JiraTicket),
        USERELATIONSHIP(Dim_Date[DateKey], Fact_JiraTicket[ResolvedDateKey])
    )
```

3. **Lưu ý khi seed data:**
   - `DaysToComplete` chỉ có giá trị khi `ResolvedDateKey` IS NOT NULL
   - Phân bổ ticket theo năm: nhiều hơn ở 2024–2026 (matching trend tăng trong dashboard)
   - Tổng ~661 tickets khi filter all years (matching KPI card trong dashboard gốc)

---

### 3.2. `Fact_PartnerTransaction`

**Ý nghĩa:** Lưu giao dịch bán hàng đồng bộ từ SAP SD module. Mỗi dòng = 1 đơn hàng/giao dịch với đối tác.

**Phục vụ charts:** Partners by Account Type (khi cần drill từ count → revenue), Cross-domain analysis

| Column Name | Data Type | Key | Nullable | Description |
|-------------|-----------|-----|----------|-------------|
| TransactionID | String(20) | PK | No | Mã giao dịch SAP (VD: SO-2026-001234) |
| PartnerKey | Integer | FK → Dim_Partner_Account | No | Đối tác (Sold-to Party) |
| OrderDateKey | Integer | FK → Dim_Date | No | Ngày đặt hàng |
| Quantity | Integer | | No | Số lượng thiết bị y tế |
| NetValue | Decimal(12,2) | | No | Giá trị doanh thu thuần (USD) |

**Setup trong Power BI:**

1. **Relationships:**
   - `PartnerKey` → `Dim_Partner_Account.PartnerKey` (Active)
   - `OrderDateKey` → `Dim_Date.DateKey` (Active)

2. **DAX Measures:**

```dax
[Total Partners] = DISTINCTCOUNT(Fact_PartnerTransaction[PartnerKey])
[Total Net Value] = SUM(Fact_PartnerTransaction[NetValue])
[Avg Transaction Value] = AVERAGE(Fact_PartnerTransaction[NetValue])
[Total Quantity Sold] = SUM(Fact_PartnerTransaction[Quantity])
```

3. **Lưu ý khi seed data:**
   - Phân bổ transactions theo AccountType: Hospitals chiếm nhiều nhất về value, Clinics nhiều về quantity
   - Dữ liệu 3 năm gần nhất (2024–2026)

---

## 4. HƯỚNG DẪN THIẾT LẬP RELATIONSHIPS TRONG POWER BI

### 4.1. Quy tắc chung

| Quy tắc | Mô tả |
|---------|--------|
| Cardinality | Luôn 1:* (Dim → Fact) |
| Cross Filter Direction | Luôn Single (Dim → Fact) |
| Không dùng Both direction | Tránh ambiguity và suy giảm hiệu năng RAM |

### 4.2. Xử lý Role-Playing Dimensions

**Vấn đề:** `Dim_User` có 2 FK trong Fact (Assignee + Reporter). `Dim_Date` có 2 FK trong Fact (Created + Resolved).

**Giải pháp A — USERELATIONSHIP (Recommended):**
- Đặt 1 relationship = Active, 1 = Inactive
- Trong DAX measure, gọi `USERELATIONSHIP()` khi cần dùng Inactive

**Giải pháp B — Physical duplicate table:**
- Tạo `Dim_Reporter` = Reference query từ `Dim_User` trong Power Query
- Tạo `Dim_ResolvedDate` = Reference query từ `Dim_Date`
- Mỗi bảng có relationship Active riêng → user nghiệp vụ kéo thả dễ hơn

**Khuyến nghị:** Dùng Giải pháp B cho demo vì trực quan hơn khi present cho Bioventus.

### 4.3. Tổng hợp Relationship Map

| # | From (1 side) | To (* side) | Status | Ghi chú |
|---|---------------|-------------|--------|---------|
| 1 | Dim_User.UserKey | Fact_JiraTicket.AssigneeKey | **Active** | Mặc định cho Assignee slicer |
| 2 | Dim_User.UserKey | Fact_JiraTicket.ReporterKey | **Inactive** | Dùng USERELATIONSHIP |
| 3 | Dim_Status.StatusKey | Fact_JiraTicket.StatusKey | Active | |
| 4 | Dim_RequestType.RequestTypeKey | Fact_JiraTicket.RequestTypeKey | Active | |
| 5 | Dim_Date.DateKey | Fact_JiraTicket.CreatedDateKey | **Active** | Mặc định cho Year/Month slicer |
| 6 | Dim_Date.DateKey | Fact_JiraTicket.ResolvedDateKey | **Inactive** | Dùng USERELATIONSHIP |
| 7 | Dim_Date.DateKey | Fact_PartnerTransaction.OrderDateKey | Active | |
| 8 | Dim_Partner_Account.PartnerKey | Fact_PartnerTransaction.PartnerKey | Active | |

---

*Document Version: 1.0*  
*Created: 18 May 2026*  
*Author: Tram (Sofia) Nguyen — BA Lead, GEX 1*  
*Approved by: Bryan Du — CDO*

