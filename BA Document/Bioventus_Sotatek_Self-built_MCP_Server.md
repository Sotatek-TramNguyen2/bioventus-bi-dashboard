# Bioventus <> Sotatek — Self-built MCP Server

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

---

## TOOL DEFINITIONS

---

tool_get_jira_dashboard_data

Lấy toàn bộ dữ liệu tổng hợp và chi tiết để kết xuất Dashboard phân tích hiệu suất team BI (Jira Tickets). Trả về: kpi_metrics (Total Tickets, Average Days to Complete), chart_data (số liệu group sẵn theo Assignee, Status, RequestType, Year, Month), detail_table (danh sách ticket chi tiết gồm IssueKey, Summary, Assignee, Reporter, Status, RequestType, DaysToComplete).
Output: Chart + Table.

**AI Response Template:**
"[Context] Tôi đã tổng hợp dữ liệu Jira từ [date_from] đến [date_to].

[KPI] [key metrics]

[Chart] Mô tả chart đã gen + "để bạn có cái nhìn tổng quan về [topic]"

[Table] "Bảng [mô tả nội dung]"

[Suggestion] "Bạn có muốn xem chi tiết về [related topic] không?""

**DA Guide (Power BI):**

*KPI Card:*
- Value: `COUNTROWS(Fact_JiraTicket)`
- Subtitle/Trend: `AVERAGE(Fact_JiraTicket[DaysToComplete])`

*Bar Chart (Ticket by Assignee/Status/RequestType):*
- Axis (Y): `Dim_User[UserName]` hoặc `Dim_Status[StatusName]` hoặc `Dim_RequestType[RequestTypeName]`
- Values: `COUNTROWS(Fact_JiraTicket)`
- Legend: `Dim_Status[StatusCategory]` (nếu cần phân nhóm)

*Column Chart (Ticket by Month/Year):*
- Axis (X): `Dim_Date[MonthName]` hoặc `Dim_Date[Year]`
- Values: `COUNTROWS(Fact_JiraTicket)`
- Small Multiples (optional): `Dim_Status[StatusCategory]`

*Detail Table:*
- Columns: `Fact_JiraTicket[IssueKey]`, `Fact_JiraTicket[Summary]`, `Dim_User[UserName]` (Assignee), `Dim_User[UserName]` (Reporter - dùng USERELATIONSHIP), `Dim_Status[StatusName]`, `Dim_RequestType[RequestTypeName]`, `Fact_JiraTicket[DaysToComplete]`

Params:
- assignee_name: Tên người xử lý (từ Dim_User). Để trống = tất cả.
- reporter_name: Tên người tạo yêu cầu (từ Dim_User). Để trống = tất cả.
- status_name: Trạng thái ticket (từ Dim_Status), ví dụ "Done", "In Progress". Để trống = tất cả.
- request_type: Loại yêu cầu (từ Dim_RequestType), ví dụ "New Report", "App Permission". Để trống = tất cả.
- date_from: Lọc từ ngày tạo ticket, định dạng "YYYY-MM-DD". Để trống = không lọc.
- date_to: Lọc đến ngày tạo ticket, định dạng "YYYY-MM-DD". Để trống = không lọc.
- detail_limit: Số lượng bản ghi tối đa trả về trong detail_table, mặc định 50.

tool_get_jira_dashboard_data

---

tool_get_sales_dashboard_data

Lấy toàn bộ dữ liệu tổng hợp và chi tiết để kết xuất Dashboard Kinh doanh và Đối tác y tế. Trả về: kpi_metrics (Total Revenue, Total Quantity), chart_data (doanh thu & số lượng group theo AccountType, Region, Year, Month), detail_table (danh sách giao dịch gồm TransactionID, PartnerName, AccountType, Region, OrderDate, Quantity, NetValue).
Output: Chart + Table.

**AI Response Template:**
"[Context] Tôi đã tổng hợp dữ liệu kinh doanh từ [date_from] đến [date_to].

[KPI] [key metrics]

[Chart] Mô tả chart đã gen + "để bạn có cái nhìn tổng quan về [topic]"

[Table] "Bảng [mô tả nội dung]"

[Suggestion] "Bạn có muốn xem thêm [related action] không?""

**DA Guide (Power BI):**

*KPI Card:*
- Value: `SUM(Fact_PartnerTransaction[NetValue])`
- Subtitle/Trend: `SUM(Fact_PartnerTransaction[Quantity])`

*Horizontal Bar Chart (Revenue by AccountType/Region):*
- Y-Axis: `Dim_Partner_Account[AccountType]` hoặc `Dim_Partner_Account[Region]`
- X-Axis: `SUM(Fact_PartnerTransaction[NetValue])`
- Legend: `Dim_Partner_Account[Region]` (nếu cần phân nhóm)

*Pie/Donut Chart (Revenue Distribution):*
- Legend: `Dim_Partner_Account[AccountType]`
- Values: `SUM(Fact_PartnerTransaction[NetValue])`
- Tooltips: `SUM(Fact_PartnerTransaction[Quantity])`

*Line Chart (Revenue Trend by Month):*
- Axis: `Dim_Date[MonthName]` hoặc `Dim_Date[Year]`
- Values: `SUM(Fact_PartnerTransaction[NetValue])`
- Small Multiples (optional): `Dim_Partner_Account[AccountType]`

*Clustered Bar (Revenue vs Quantity by AccountType):*
- Y-Axis: `Dim_Partner_Account[AccountType]`
- X-Axis: `SUM(Fact_PartnerTransaction[NetValue])` và `SUM(Fact_PartnerTransaction[Quantity])`
- Legend: Metrics

*Detail Table:*
- Columns: `Fact_PartnerTransaction[TransactionID]`, `Dim_Partner_Account[PartnerName]`, `Dim_Partner_Account[AccountType]`, `Dim_Partner_Account[Region]`, `Dim_Date[Date]` (OrderDate), `Fact_PartnerTransaction[Quantity]`, `Fact_PartnerTransaction[NetValue]`

Params:
- account_type: Phân loại đối tác (từ Dim_Partner_Account), ví dụ "Hospitals", "Patients", "Distributors". Để trống = tất cả.
- region: Khu vực phân phối (từ Dim_Partner_Account). Để trống = tất cả.
- partner_name: Tên đối tác cụ thể. Để trống = tất cả.
- date_from: Lọc từ ngày giao dịch, định dạng "YYYY-MM-DD". Để trống = không lọc.
- date_to: Lọc đến ngày giao dịch, định dạng "YYYY-MM-DD". Để trống = không lọc.
- detail_limit: Số lượng bản ghi tối đa trả về trong detail_table, mặc định 50.

tool_get_sales_dashboard_data

---

tool_get_partner_directory

Tra cứu và phân trang danh mục đối tác khách hàng từ bảng Dim_Partner_Account. Hữu ích khi AI cần tìm chính xác tên một bệnh viện/bác sĩ trước khi query giao dịch. Trả về: danh sách PartnerID, PartnerName, AccountType, Region.
Output: Table.

**AI Response Template:**
"[Context] Tôi đã tra cứu danh mục đối tác [search_term/account_type/region].

[Table] "Bảng [mô tả nội dung]"

[Suggestion] "Bạn có muốn xem chi tiết giao dịch của [partner_name] không?""

**DA Guide (Power BI):**

*Partner Directory Table:*
- Columns: `Dim_Partner_Account[PartnerID]`, `Dim_Partner_Account[PartnerName]`, `Dim_Partner_Account[AccountType]`, `Dim_Partner_Account[Region]`
- Sort by: `Dim_Partner_Account[PartnerName]`
- Visual Level Filter: `Dim_Partner_Account[AccountType]` = "Hospitals" (nếu cần)

*Slicer (for filtering):*
- Fields: `Dim_Partner_Account[AccountType]`, `Dim_Partner_Account[Region]`
- Type: Dropdown hoặc List

Params:
- search_term: Tìm kiếm gần đúng theo PartnerName (chuỗi text). Để trống = tất cả.
- account_type: Phân loại đối tác. Để trống = tất cả.
- region: Khu vực. Để trống = tất cả.
- limit: Số bản ghi tối đa, mặc định 100.

tool_get_partner_directory

---

tool_get_system_metadata

Lấy danh sách các giá trị hợp lệ (Valid values) của các bộ lọc (Filters/Dimensions) trong toàn bộ hệ thống. Trả về: danh sách mảng JSON chứa Users (từ Dim_User), Statuses (từ Dim_Status), RequestTypes (từ Dim_RequestType), AccountTypes và Regions (từ Dim_Partner_Account).
Output: Reference Data (không trực tiếp sinh chart/table, AI dùng để validate trước khi gọi tool khác).

**AI Response Template:**
"[Context] Danh sách giá trị hợp lệ trong hệ thống:

[Reference Data] Liệt kê các valid values theo nhóm

[Suggestion] "Bạn có muốn tra cứu đối tác cụ thể hoặc xem dashboard không?""

**DA Guide (Power BI):**

*Dùng để tạo Slicer với các giá trị chuẩn:*
- Status Slicer: `Dim_Status[StatusName]`
- Request Type Slicer: `Dim_RequestType[RequestTypeName]`
- Account Type Slicer: `Dim_Partner_Account[AccountType]`
- Region Slicer: `Dim_Partner_Account[Region]`
- User Slicer: `Dim_User[UserName]`

*Ghi chú:* Khi tạo Slicer, dùng field từ Dimension table, không dùng giá trị hard-code.

Params:
- (Không yêu cầu tham số)

Ghi chú: AI Agent sẽ tự động gọi tool này khi nó không chắc chắn về tên của một Trạng thái, Loại request, hoặc Loại đối tác có tồn tại trong Database hay không.

tool_get_system_metadata

---

tool_get_jira_ticket_deepdive

Truy xuất lịch sử và thông tin siêu chi tiết của MỘT ticket duy nhất nhằm phục vụ cho mục đích điều tra (Investigate) trực tiếp. Trả về: toàn bộ trường dữ liệu có trong Fact_JiraTicket và các bảng Dim liên quan tương ứng với Ticket đó (IssueKey, Summary, Assignee, Reporter, Status, RequestType, CreatedDate, ResolvedDate, DaysToComplete).
Output: Detail View.

**AI Response Template:**
"[Context] Chi tiết ticket [issue_key]:

[Detail View] Hiển thị toàn bộ fields của ticket

[Insight] "Ticket này đã [action] trong [days] ngày. [So sánh với average]"

[Suggestion] "Bạn có muốn xem các ticket cùng Assignee/Reporter không?""

**DA Guide (Power BI):**

*Detail Card (Multi-row Card):*
- Fields: `Fact_JiraTicket[IssueKey]`, `Fact_JiraTicket[Summary]`, `Dim_Status[StatusName]`, `Dim_RequestType[RequestTypeName]`, `Fact_JiraTicket[DaysToComplete]`

* hoặc Table với 1 row:*
- Columns: `Fact_JiraTicket[IssueKey]`, `Fact_JiraTicket[Summary]`, `Dim_User[UserName]` (Assignee), `Dim_User[UserName]` (Reporter - USERELATIONSHIP), `Dim_Status[StatusName]`, `Dim_RequestType[RequestTypeName]`, `Dim_Date[Date]` (CreatedDate), `Dim_Date[Date]` (ResolvedDate - USERELATIONSHIP), `Fact_JiraTicket[DaysToComplete]`
- Filter: `Fact_JiraTicket[IssueKey]` = "BI-1411"

Params:
- issue_key: Bắt buộc. Mã ticket thực tế trên Jira, ví dụ "BI-1411", "BI-1429".

tool_get_jira_ticket_deepdive

---

tool_get_partner_transaction_deepdive

Truy xuất chi tiết tất cả các giao dịch (Transactions) của MỘT đối tác cụ thể để làm báo cáo công nợ hoặc sao kê chi tiết. Trả về: danh sách TransactionID, OrderDate, Quantity, NetValue và summary (total_transactions, total_quantity, total_net_value).
Output: Table + Summary.

**AI Response Template:**
"[Context] Sao kê giao dịch của [partner_name]:

[Summary] Tổng quan: total_transactions, total_quantity, total_net_value

[Table] Chi tiết từng giao dịch

[Suggestion] "Bạn có muốn so sánh với đối tác khác hoặc xem trend không?""

**DA Guide (Power BI):**

*KPI Summary Cards (3 cards horizontal):*
- Card 1: `COUNTROWS(Fact_PartnerTransaction)` (Total Transactions)
- Card 2: `SUM(Fact_PartnerTransaction[Quantity])` (Total Quantity)
- Card 3: `SUM(Fact_PartnerTransaction[NetValue])` (Total Revenue)

*Transaction Table:*
- Columns: `Fact_PartnerTransaction[TransactionID]`, `Dim_Date[Date]` (OrderDate), `Fact_PartnerTransaction[Quantity]`, `Fact_PartnerTransaction[NetValue]`
- Filter: `Dim_Partner_Account[PartnerID]` = "BP-001234" (hoặc PartnerKey tương ứng)

*Line Chart (Transaction Trend):*
- Axis: `Dim_Date[MonthName]` hoặc `Dim_Date[Date]`
- Values: `SUM(Fact_PartnerTransaction[NetValue])`
- Filter: `Dim_Partner_Account[PartnerID]` = "BP-001234"

Params:
- partner_id: Bắt buộc. PartnerID của đối tác cần sao kê.
- date_from: Lọc từ ngày, định dạng "YYYY-MM-DD". Để trống = không lọc.
- date_to: Lọc đến ngày, định dạng "YYYY-MM-DD". Để trống = không lọc.
- limit: Số bản ghi tối đa, mặc định 100.

tool_get_partner_transaction_deepdive

---

## OUTPUT RESPONSE RULES

### AI Response Structure

Mỗi response từ AI phải có 4 phần:

| Phần | Mô tả | Ví dụ |
|------|-------|-------|
| **Context** | Giải thích AI đã làm gì, data range nào | "Tôi đã tổng hợp dữ liệu Jira từ 2025-01-01 đến 2026-05-19" |
| **KPIs/Chart** | Key metrics + chart đã gen | "Tổng 661 ticket, chart top 10 assignee" |
| **Table** | Mô tả rõ bảng đang hiển thị | "Bảng top 10 partner có doanh thu cao nhất" |
| **DA Guide** | Hướng dẫn Power BI fields cho user tự tạo | (Xem section bên dưới) |
| **Suggestion** | Gợi ý action liên quan | "Bạn có muốn xem chi tiết giao dịch của partner này không?" |

### DA Guide (Power BI) - Hướng dẫn tạo chart/table

Dưới mỗi chart/table, AI phải thêm 1 đoạn note ngắn theo format:

**DA Guide Format:**
```
📝 Power BI Guide:
- [Chart Type]: Y-Axis: [Dim/Field], X-Axis: [Dim/Field hoặc Measure], Values: [Measure]
- [Legend] (optional): [Field]
- [Tooltips] (optional): [Field/Measure]
```

**Ví dụ 1 - Bar Chart (Jira):**
```
📝 Power BI Guide:
- Bar Chart: Y-Axis: Dim_Status[StatusName], X-Axis: COUNTROWS(Fact_JiraTicket)
- Legend (optional): Dim_Status[StatusCategory]
- Tooltips: AVG(Fact_JiraTicket[DaysToComplete])
```

**Ví dụ 2 - Table (Sales):**
```
📝 Power BI Guide:
- Table Columns: Dim_Partner_Account[PartnerName], Dim_Partner_Account[AccountType], SUM(Fact_PartnerTransaction[NetValue])
- Sort by: NetValue (Descending)
- Visual Filter: AccountType = "Hospitals"
```

**Ví dụ 3 - KPI Card:**
```
📝 Power BI Guide:
- KPI Card: Value = COUNTROWS(Fact_JiraTicket), Goal (optional) = 600
- Trend Indicator: vs Last Year = CALCULATE(COUNTROWS(...), SAMEPERIODLASTYEAR(...))
```

### Context Formatting

| Output Type | Context Example |
|-------------|-----------------|
| Chart + Table | "để bạn có cái nhìn tổng quan về [topic]" |
| Table only | "Dưới đây là bảng [mô tả nội dung]" |
| Detail View | "Chi tiết ticket [issue_key]" |
| Reference Data | "Danh sách giá trị hợp lệ trong hệ thống" |

### Suggestion Patterns

| Khi user hỏi | AI suggest |
|---------------|------------|
| Top N items | "Bạn có muốn xem [N+1] hoặc filter thêm không?" |
| Một loại đối tác | "Bạn có muốn so sánh với loại đối tác khác không?" |
| Một khu vực | "Bạn có muốn xem các khu vực khác không?" |
| Chi tiết 1 item | "Bạn có muốn xem các item cùng [Assignee/Type/Region] không?" |
| Tổng quan | "Bạn có muốn drill down vào [segment] cụ thể không?" |

---

## DEVELOPER NOTES

### Xử lý chart_data

Khi query cho phần chart_data, sử dụng GROUP BY để gom nhóm dữ liệu. Trả về dữ liệu dạng Key-Value (VD: [{"label": "Rahime", "value": 71}]) để AI của SotaAgent có thể dễ dàng map vào trục X và trục Y khi sinh ra UI vẽ Chart.

```sql
-- Ví dụ: Group by Assignee
SELECT u.user_name AS label, COUNT(*) AS value
FROM Fact_JiraTicket f
JOIN Dim_User u ON f.assignee_key = u.user_key
GROUP BY u.user_name
ORDER BY value DESC;
```

### Xử lý Date Filter

- date_from và date_to luôn filter theo CreatedDate (bảng Fact_JiraTicket) hoặc OrderDate (bảng Fact_PartnerTransaction)
- Format chuẩn: "YYYY-MM-DD" (VD: "2026-05-19")
- Nếu để trống → không áp dụng filter

### Giới hạn (Limits)

- detail_limit mặc định = 50, tối đa = 500
- limit trong partner_directory mặc định = 100, tối đa = 1000

---

## POWER BI CHART TYPE REFERENCE

| Chart Type | Y-Axis | X-Axis | Values | Legend | Tooltip |
|------------|--------|--------|--------|--------|---------|
| Bar (Horizontal) | Dimension | Measure | Measure | Dimension (optional) | Measure |
| Bar (Vertical) | Measure | Dimension | Measure | Dimension (optional) | Measure |
| Column | Dimension | Measure | Measure | Dimension (optional) | Measure |
| Line | Dimension (Date) | Measure | Measure | Dimension (optional) | Measure |
| Pie/Donut | Legend | - | Measure | Dimension | Measure |
| KPI Card | - | - | Measure | - | Goal measure |
| Table | - | - | Dimension/Measure | - | - |
| Clustered Bar | Dimension | Measure + Measure | - | Series | Measure |

---

*Document Version: 1.0*  
*Created: 18 May 2026*  
*Author: Tram (Sofia) Nguyen — BA Lead, GEX 1*