# Bioventus <> Sotatek — Demo Scenario

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

## 15 DEMO SCENARIOS

---

### Scenario 1: Dashboard tổng quan Jira

**User Question:**
> "Cho tôi xem tổng quan tình hình ticket Jira hiện tại"

**MCP Tool Used:**
`tool_get_jira_dashboard_data`

**Parameters:**
```
- assignee_name: (để trống)
- reporter_name: (để trống)
- status_name: (để trống)
- request_type: (để trống)
- date_from: 2026-01-01
- date_to: 2026-05-19
- detail_limit: 50
```

**Expected AI Response:**

```
Tôi đã tổng hợp dữ liệu Jira từ 2026-01-01 đến 2026-05-19.

📊 KPIs:
- Total Tickets: 661
- Avg Days to Complete: 4.5

📈 Chart (Bar Chart - Ticket by Status):
[Bar chart: Done 412, In Progress 98, New 75, QA UAT 45, Development 20, On Hold 8, PROD/UAT 3]
→ Để bạn có cái nhìn tổng quan về phân bổ trạng thái ticket

📋 Bảng chi tiết 50 ticket gần nhất:
| IssueKey | Summary | Assignee | Status | DaysToComplete |
|----------|---------|----------|--------|----------------|
| BI-1411 | Update dashboard... | Rahime | Done | 3 |
| BI-1410 | Fix report... | Koh | In Progress | 12 |
...

📝 Power BI Guide:
- KPI Card 1: Value = COUNTROWS(Fact_JiraTicket), Subtitle = AVG(Fact_JiraTicket[DaysToComplete])
- KPI Card 2: Trend = vs Last Year (CALCULATE + SAMEPERIODLASTYEAR)
- Stacked Bar Chart: Y-Axis = Dim_Status[StatusName], X-Axis = COUNTROWS(Fact_JiraTicket), Legend = Dim_Status[StatusCategory]
- Detail Table: Columns = Fact_JiraTicket[IssueKey], Fact_JiraTicket[Summary], Dim_User[UserName] (Assignee), Dim_Status[StatusName], Fact_JiraTicket[DaysToComplete]

Bạn có muốn drill down vào trạng thái cụ thể nào không?
```

**Output Type:** Chart + Table

---

### Scenario 2: Top 10 user có nhiều ticket nhất

**User Question:**
> "Ai đang xử lý nhiều ticket nhất?"

**MCP Tool Used:**
`tool_get_jira_dashboard_data`

**Parameters:**
```
- assignee_name: (để trống)
- status_name: (để trống)
- date_from: 2026-01-01
- date_to: 2026-05-19
```

**Expected AI Response:**

```
Tôi đã tổng hợp dữ liệu Jira từ 2026-01-01 đến 2026-05-19.

📊 KPIs:
- Top Assignee: Rahime Cvetkovic (71 tickets)
- Total Assignees: 15

📈 Chart (Horizontal Bar - Ticket by Assignee):
[Chart: Rahime 71, Koh 58, Shankar 45, Robert 38, ...]
→ Để bạn có cái nhìn tổng quan về workload theo từng người xử lý

📝 Power BI Guide:
- Horizontal Bar Chart: Y-Axis = Dim_User[UserName], X-Axis = COUNTROWS(Fact_JiraTicket), Sort = Descending
- Tooltips: AVG(Fact_JiraTicket[DaysToComplete]) để xem ai xử lý nhanh hơn
- Visual Filter: status_name = "Done" (nếu muốn xem riêng ticket đã done)

Bạn có muốn xem chi tiết ticket của Rahime hoặc so sánh với người khác không?
```

**Output Type:** Chart + Table

---

### Scenario 3: Ticket pending lâu nhất

**User Question:**
> "Những ticket nào đang bị kẹt lâu chưa xử lý?"

**MCP Tool Used:**
`tool_get_jira_dashboard_data`

**Parameters:**
```
- status_name: In Progress, Development, QA UAT, On Hold
- date_from: 2026-01-01
- date_to: 2026-05-19
```

**Expected AI Response:**

```
Tôi đã tổng hợp các ticket đang open từ 2026-01-01 đến 2026-05-19.

📋 Bảng ticket đang pending (sắp xếp theo ngày tạo):
| IssueKey | Summary | Assignee | Status | CreatedDate | DaysPending |
|----------|---------|----------|--------|-------------|-------------|
| BI-1301 | Fix dashboard... | Koh | In Progress | 2026-04-15 | 34 days |
| BI-1289 | Update report... | Shankar | QA UAT | 2026-04-20 | 29 days |
| BI-1275 | New dataset... | Robert | Development | 2026-04-25 | 24 days |
...

⚠️ Có 15 ticket đang open hơn 2 tuần chưa được xử lý.

📝 Power BI Guide:
- Table: Columns = Fact_JiraTicket[IssueKey], Fact_JiraTicket[Summary], Dim_User[UserName] (Assignee), Dim_Status[StatusName], Dim_Date[Date] (CreatedDate), Fact_JiraTicket[DaysToComplete]
- Conditional Formatting: DaysToComplete > 14 → Background = Red
- Sort by: Dim_Date[Date] (CreatedDate) Ascending (ticket cũ nhất lên đầu)
- Visual Filter: Dim_Status[StatusCategory] <> "Closed"

Bạn có muốn tôi thông báo cho assignee hoặc xem chi tiết ticket cụ thể không?
```

**Output Type:** Table

---

### Scenario 4: Tra cứu đối tác cụ thể

**User Question:**
> "Tìm bệnh viện Mayo Clinic trong hệ thống"

**MCP Tool Used:**
`tool_get_partner_directory`

**Parameters:**
```
- search_term: Mayo Clinic
- limit: 10
```

**Expected AI Response:**

```
Tôi đã tra cứu danh mục đối tác với từ khóa "Mayo Clinic".

📋 Bảng kết quả tìm kiếm:
| PartnerID | PartnerName | AccountType | Region |
|-----------|-------------|-------------|--------|
| BP-001234 | Mayo Clinic | Hospitals | Midwest |

📍 Thông tin bổ sung:
- Địa chỉ: Rochester, Minnesota
- Tổng giao dịch: 12
- Tổng doanh thu: $900,000

📝 Power BI Guide:
- Table: Columns = Dim_Partner_Account[PartnerID], Dim_Partner_Account[PartnerName], Dim_Partner_Account[AccountType], Dim_Partner_Account[Region]
- Search: Dùng Visual Filter với "Contains" hoặc FILTER() DAX
- Slicer bổ sung: Dim_Partner_Account[AccountType], Dim_Partner_Account[Region]

Bạn có muốn xem chi tiết giao dịch của Mayo Clinic không?
```

**Output Type:** Table

---

### Scenario 5: Revenue theo đối tác

**User Question:**
> "Doanh thu theo loại đối tác năm 2026?"

**MCP Tool Used:**
`tool_get_sales_dashboard_data`

**Parameters:**
```
- date_from: 2026-01-01
- date_to: 2026-05-19
- detail_limit: 50
```

**Expected AI Response:**

```
Tôi đã tổng hợp dữ liệu kinh doanh từ 2026-01-01 đến 2026-05-19.

💰 KPIs:
- Total Revenue: $15,000,000
- Total Quantity: 45,000

📈 Chart (Horizontal Bar - Revenue by AccountType):
[Chart: Hospitals $8.5M (57%), Distributor $3.2M (21%), Clinics $1.8M (12%), Doctors $1M (7%), Insurance $0.5M (3%)]
→ Để bạn có cái nhìn tổng quan về phân bổ doanh thu theo loại đối tác

📋 Bảng top 10 partner có doanh thu cao nhất:
| PartnerName | AccountType | TotalRevenue |
|-------------|-------------|--------------|
| Mayo Clinic | Hospitals | $900,000 |
| Johns Hopkins | Hospitals | $850,000 |
...

📝 Power BI Guide:
- KPI Card: Value = SUM(Fact_PartnerTransaction[NetValue]), Subtitle = SUM(Fact_PartnerTransaction[Quantity])
- Horizontal Bar Chart: Y-Axis = Dim_Partner_Account[AccountType], X-Axis = SUM(Fact_PartnerTransaction[NetValue])
- Pie Chart: Legend = Dim_Partner_Account[AccountType], Values = SUM(Fact_PartnerTransaction[NetValue])
- Clustered Bar (Revenue vs Quantity): Y-Axis = Dim_Partner_Account[AccountType], X-Axis = SUM(NetValue) và SUM(Quantity)
- Detail Table: Columns = Dim_Partner_Account[PartnerName], Dim_Partner_Account[AccountType], SUM(Fact_PartnerTransaction[NetValue]), Sort = NetValue Descending

Bạn có muốn so sánh với loại đối tác khác hoặc xem theo khu vực không?
```

**Output Type:** Chart + Table

---

### Scenario 6: Chi tiết 1 ticket để điều tra

**User Question:**
> "Chi tiết ticket BI-1411"

**MCP Tool Used:**
`tool_get_jira_ticket_deepdive`

**Parameters:**
```
- issue_key: BI-1411
```

**Expected AI Response:**

```
Chi tiết ticket BI-1411:

📋 Thông tin ticket:
| Field | Value |
|-------|-------|
| Issue Key | BI-1411 |
| Summary | Update dashboard sales report |
| Status | Done ✅ |
| Priority | High |

👥 Nhân sự:
- Assignee: Rahime Cvetkovic (BI Engineering)
- Reporter: Koh Higaki (Operations)

📁 Phân loại:
- Request Type: Update Existing Report

📅 Timeline:
- Created: 2026-05-01
- Resolved: 2026-05-04
- Days to Complete: 3 days

💡 Ticket này được xử lý trong 3 ngày, nhanh hơn average (4.5 days) → SLA OK.

📝 Power BI Guide:
- Multi-row Card: Fields = Fact_JiraTicket[IssueKey], Fact_JiraTicket[Summary], Dim_Status[StatusName], Fact_JiraTicket[DaysToComplete]
- Table (1 row): Columns = Fact_JiraTicket[IssueKey], Fact_JiraTicket[Summary], Dim_User[UserName] (Assignee - USERELATIONSHIP), Dim_User[UserName] (Reporter - USERELATIONSHIP), Dim_Status[StatusName], Dim_RequestType[RequestTypeName], Dim_Date[Date] (CreatedDate), Dim_Date[Date] (ResolvedDate - USERELATIONSHIP), Fact_JiraTicket[DaysToComplete]
- Visual Filter: Fact_JiraTicket[IssueKey] = "BI-1411"

Bạn có muốn xem các ticket cùng Assignee (Rahime) hoặc cùng Request Type không?
```

**Output Type:** Detail View

---

### Scenario 7: Sao kê giao dịch đối tác

**User Question:**
> "Liệt kê tất cả giao dịch của Mayo Clinic năm 2026"

**MCP Tool Used:**
`tool_get_partner_transaction_deepdive`

**Parameters:**
```
- partner_id: BP-001234
- date_from: 2026-01-01
- date_to: 2026-05-19
```

**Expected AI Response:**

```
Sao kê giao dịch của Mayo Clinic (BP-001234):

📊 Summary:
- Total Transactions: 12
- Total Quantity: 1,800
- Total Revenue: $900,000
- Avg Order Value: $75,000

📋 Chi tiết giao dịch:
| TransactionID | OrderDate | Quantity | NetValue |
|---------------|-----------|----------|----------|
| SO-2026-001234 | 2026-05-15 | 150 | $75,000 |
| SO-2026-001189 | 2026-04-28 | 200 | $100,000 |
...

📝 Power BI Guide:
- KPI Summary (3 Cards): Card1 = COUNTROWS(Fact_PartnerTransaction), Card2 = SUM(Quantity), Card3 = SUM(NetValue)
- Transaction Table: Columns = Fact_PartnerTransaction[TransactionID], Dim_Date[Date] (OrderDate), Fact_PartnerTransaction[Quantity], Fact_PartnerTransaction[NetValue], Sort = OrderDate Descending
- Visual Filter: Dim_Partner_Account[PartnerID] = "BP-001234"
- Line Chart (Trend): X-Axis = Dim_Date[MonthName], Y-Axis = SUM(NetValue), Filter = PartnerID = "BP-001234"

Bạn có muốn so sánh với đối tác khác (ví dụ Johns Hopkins) hoặc xem trend theo tháng không?
```

**Output Type:** Table + Summary

---

### Scenario 8: Validate trước khi query

**User Question:**
> "Trạng thái nào đang có trong hệ thống Jira?"

**MCP Tool Used:**
`tool_get_system_metadata`

**Parameters:**
```
- (không có)
```

**Expected AI Response:**

```
Danh sách giá trị hợp lệ trong hệ thống:

📋 Statuses:
| StatusName | Category |
|------------|----------|
| New | Open |
| In Progress | In Work |
| Development | In Work |
| QA UAT | In Work |
| PROD/UAT | In Work |
| On Hold | Open |
| Done | Closed |

📋 Request Types:
| RequestTypeName |
|------------------|
| Update Existing Report/Dataset |
| App Permission |
| New Report |
| Report an Issue |
| New Dataset |
| Data Quality Investigation |

📝 Power BI Guide:
- Status Slicer: Field = Dim_Status[StatusName], Type = Dropdown
- Request Type Slicer: Field = Dim_RequestType[RequestTypeName], Type = List
- Account Type Slicer: Field = Dim_Partner_Account[AccountType]
- Region Slicer: Field = Dim_Partner_Account[Region]
- Ghi chú: Dùng Dimension fields cho slicer, KHÔNG hard-code giá trị

💡 Danh sách này giúp bạn biết giá trị nào hợp lệ khi lọc.

Bạn có muốn tra cứu đối tác cụ thể hoặc xem dashboard tổng quan không?
```

**Output Type:** Reference Data

---

### Scenario 9: Tìm ticket theo người tạo

**User Question:**
> "Những ai đang gửi nhiều yêu cầu nhất?"

**MCP Tool Used:**
`tool_get_jira_dashboard_data`

**Parameters:**
```
- date_from: 2026-01-01
- date_to: 2026-05-19
```

**Expected AI Response:**

```
Tôi đã tổng hợp dữ liệu Jira từ 2026-01-01 đến 2026-05-19.

📊 KPIs:
- Top Reporter: Koh Higaki (42 tickets)
- Total Reporters: 25

📈 Chart (Bar - Ticket by Reporter):
[Chart: Koh 42, Conrad 38, Faeza 35, ...]
→ Để bạn có cái nhìn tổng quan về ai đang gửi nhiều request nhất

💡 Koh Higaki gửi nhiều request nhất (42), chủ yếu là Update Existing Report và New Report.

📝 Power BI Guide:
- Bar Chart: Y-Axis = Dim_User[UserName] (Reporter - dùng USERELATIONSHIP), X-Axis = COUNTROWS(Fact_JiraTicket)
- Measure: [Tickets by Reporter] = CALCULATE(COUNTROWS(Fact_JiraTicket), USERELATIONSHIP(Dim_User[UserKey], Fact_JiraTicket[ReporterKey]))
- Table: Columns = Dim_User[UserName] (Reporter), COUNTROWS(Fact_JiraTicket), Sort = Descending
- Visual Filter: Dùng ReporterKey (Inactive relationship)

Bạn có muốn xem Koh gửi yêu cầu về loại nào nhiều nhất không?
```

**Output Type:** Chart + Table

---

### Scenario 10: Loại request nào chiếm nhiều công việc nhất?

**User Question:**
> "Team BI đang tốn nguồn lực vào loại request nào?"

**MCP Tool Used:**
`tool_get_jira_dashboard_data`

**Parameters:**
```
- date_from: 2026-01-01
- date_to: 2026-05-19
```

**Expected AI Response:**

```
Tôi đã tổng hợp dữ liệu Jira từ 2026-01-01 đến 2026-05-19.

📊 KPIs:
- Total Tickets: 661

📈 Chart (Horizontal Bar - Ticket by Request Type):
[Chart: Update Report 285 (43%), New Report 185 (28%), App Permission 95 (14%), Report Issue 52 (8%), New Dataset 25 (4%), Data Quality 19 (3%)]
→ Để bạn có cái nhìn tổng quan về phân bổ workload theo loại request

📋 Avg Days to Complete by Type:
| RequestType | AvgDays |
|-------------|---------|
| Report an Issue | 6.5 days (cao nhất) |
| New Report | 5.2 days |
| Update Existing Report | 3.1 days |
...

💡 "Report an Issue" có SLA cao nhất (6.5 days) → có thể cần ưu tiên xử lý.

📝 Power BI Guide:
- Horizontal Bar Chart: Y-Axis = Dim_RequestType[RequestTypeName], X-Axis = COUNTROWS(Fact_JiraTicket)
- Clustered Bar (Tickets + Avg Days): Y-Axis = Dim_RequestType[RequestTypeName], X-Axis = COUNTROWS(Fact_JiraTicket) và AVG(Fact_JiraTicket[DaysToComplete])
- Table: Columns = Dim_RequestType[RequestTypeName], COUNTROWS(Fact_JiraTicket), AVG(Fact_JiraTicket[DaysToComplete])
- Legend (optional): Dim_RequestType[RequestTypeName]

Bạn có muốn xem chi tiết ticket "Report an Issue" hoặc so sánh với tháng trước không?
```

**Output Type:** Chart + Table

---

### Scenario 11: Filter theo trạng thái Done

**User Question:**
> "Có bao nhiêu ticket đã hoàn thành trong tháng 5?"

**MCP Tool Used:**
`tool_get_jira_dashboard_data`

**Parameters:**
```
- status_name: Done
- date_from: 2026-05-01
- date_to: 2026-05-19
```

**Expected AI Response:**

```
Tôi đã tổng hợp ticket Done từ 2026-05-01 đến 2026-05-19.

📊 KPIs:
- Done Tickets (May 2026): 45
- Avg Days to Complete: 3.2 days

📈 Chart (Line - Done tickets by day):
[Line chart showing trend]
→ Để bạn có cái nhìn tổng quan về tốc độ xử lý trong tháng

📋 Bảng ticket Done gần nhất:
| IssueKey | Summary | Assignee | ResolvedDate |
|----------|---------|----------|--------------|
| BI-1405 | Update... | Rahime | 2026-05-18 |
...

📝 Power BI Guide:
- KPI Card: Value = COUNTROWS(Fact_JiraTicket) (với filter status = "Done"), Subtitle = AVG(Fact_JiraTicket[DaysToComplete])
- Line Chart: X-Axis = Dim_Date[Date] (CreatedDate), Y-Axis = COUNTROWS(Fact_JiraTicket)
- Visual Filter: Dim_Status[StatusName] = "Done"
- Slicer bổ sung: Dim_Date[MonthName], Dim_Date[Year]

Bạn có muốn so sánh với tháng 4 hoặc xem ticket chưa Done không?
```

**Output Type:** Chart + Table

---

### Scenario 12: Tìm kiếm đối tác không rõ tên

**User Question:**
> "Có bệnh viện nào ở Midwest không?"

**MCP Tool Used:**
`tool_get_partner_directory`

**Parameters:**
```
- account_type: Hospitals
- region: Midwest
- limit: 50
```

**Expected AI Response:**

```
Tôi đã tra cứu danh mục đối tác với filter: Hospitals | Midwest.

📋 Bảng kết quả (45 bệnh viện):
| PartnerID | PartnerName | AccountType | Region |
|-----------|-------------|-------------|--------|
| BP-001234 | Mayo Clinic | Hospitals | Midwest |
| BP-001456 | Methodist Hospital | Hospitals | Midwest |
| BP-002789 | Rush University | Hospitals | Midwest |
...

💡 Có 45 bệnh viện ở Midwest trong hệ thống.

📝 Power BI Guide:
- Table: Columns = Dim_Partner_Account[PartnerID], Dim_Partner_Account[PartnerName], Dim_Partner_Account[AccountType], Dim_Partner_Account[Region]
- Slicer: Dim_Partner_Account[AccountType] = "Hospitals", Dim_Partner_Account[Region] = "Midwest"
- Search: Dùng Visual Filter hoặc FILTER() DAX
- Sort: PartnerName Ascending

Bạn có muốn xem chi tiết giao dịch của bệnh viện cụ thể hoặc so sánh Midwest với region khác không?
```

**Output Type:** Table

---

### Scenario 13: Revenue theo region

**User Question:**
> "Doanh thu phân bổ theo khu vực như thế nào?"

**MCP Tool Used:**
`tool_get_sales_dashboard_data`

**Parameters:**
```
- date_from: 2026-01-01
- date_to: 2026-05-19
```

**Expected AI Response:**

```
Tôi đã tổng hợp dữ liệu kinh doanh từ 2026-01-01 đến 2026-05-19.

💰 KPIs:
- Total Revenue: $15,000,000

📈 Chart (Pie - Revenue by Region):
[Chart: Northeast $5.25M (35%), Midwest $4.5M (30%), Southeast $3M (20%), Southwest $1.5M (10%), West $0.75M (5%)]
→ Để bạn có cái nhìn tổng quan về phân bổ doanh thu theo khu vực

📋 Region Detail:
| Region | Partners | Transactions | Revenue |
|--------|----------|--------------|---------|
| Northeast | 120 | 1,500 | $5,250,000 |
| Midwest | 95 | 1,200 | $4,500,000 |
...

💡 Northeast dẫn đầu về doanh thu ($5.25M), chiếm 35%.

📝 Power BI Guide:
- Pie/Donut Chart: Legend = Dim_Partner_Account[Region], Values = SUM(Fact_PartnerTransaction[NetValue])
- Table: Columns = Dim_Partner_Account[Region], COUNTROWS(Dim_Partner_Account) (Partners), COUNTROWS(Fact_PartnerTransaction) (Transactions), SUM(Fact_PartnerTransaction[NetValue])
- Drillthrough: Region → Partner Details

Bạn có muốn drill down vào Northeast để xem top partners hoặc so sánh với region khác không?
```

**Output Type:** Chart + Table

---

### Scenario 14: KPI ngắn gọn

**User Question:**
> "Tổng số ticket hiện tại là bao nhiêu?"

**MCP Tool Used:**
`tool_get_jira_dashboard_data`

**Parameters:**
```
- date_from: 2026-01-01
- date_to: 2026-05-19
```

**Expected AI Response:**

```
Tôi đã tổng hợp dữ liệu Jira từ 2026-01-01 đến 2026-05-19.

📊 KPIs:
┌─────────────────────┐
│  Total Tickets      │
│       661           │
│  ↑ 12% vs last year │
└─────────────────────┘

💡 Hiện có 661 ticket trong hệ thống, tăng 12% so với cùng kỳ năm ngoái.

📝 Power BI Guide:
- KPI Card: Value = COUNTROWS(Fact_JiraTicket), Goal (optional) = 600
- Trend Indicator: CALCULATE(COUNTROWS(Fact_JiraTicket), SAMEPERIODLASTYEAR(Dim_Date[Date]))
- Comparison: "vs Last Year" = DIVIDE(COUNTROWS(Fact_JiraTicket), CALCULATE(COUNTROWS(Fact_JiraTicket), SAMEPERIODLASTYEAR(Dim_Date[Date]))) - 1

Bạn có muốn xem chi tiết theo assignee, status hoặc request type không?
```

**Output Type:** Chart (minimal)

---

### Scenario 15: Cross-domain analysis

**User Question:**
> "So sánh workload Jira và doanh thu SAP — có tương quan gì không?"

**MCP Tool Used:**
`tool_get_jira_dashboard_data` + `tool_get_sales_dashboard_data`

**Parameters:**
```
(Jira Dashboard)
- date_from: 2026-01-01
- date_to: 2026-05-19

(Sales Dashboard)
- date_from: 2026-01-01
- date_to: 2026-05-19
```

**Expected AI Response:**

```
Tôi đã tổng hợp dữ liệu từ cả Jira và SAP từ 2026-01-01 đến 2026-05-19.

📊 KPIs - Jira Workload vs SAP Revenue:

| Jira | SAP |
|------|-----|
| Total Tickets: 661 | Total Revenue: $15,000,000 |
| Avg 4.5 days/ticket | Avg $75,000/order |
| ↑ 12% vs last year | ↑ 8% vs last year |

📈 Chart (Dual Axis - Ticket Volume vs Revenue):
[Bar + Line chart: ticket volume and revenue trend]
→ Để bạn có cái nhìn tổng quan về tương quan giữa workload và doanh thu

💡 Workload Jira tăng 12% trong khi Revenue chỉ tăng 8%. Team BI đang xử lý nhiều ticket hơn nhưng doanh thu không tăng tương xứng.

📝 Power BI Guide:
- KPI Cards (2 side by side): Card1 = COUNTROWS(Fact_JiraTicket), Card2 = SUM(Fact_PartnerTransaction[NetValue])
- Combo Chart: Axis = Dim_Date[MonthName], Bar Values = COUNTROWS(Fact_JiraTicket), Line Values = SUM(Fact_PartnerTransaction[NetValue])
- Cross-filter: Chọn Region/AccountType → filter cả Jira và SAP visuals
- Ghi chú: Cần tạo separate page cho mỗi domain, dùng sync slicers để link

Bạn có muốn drill down vào segment nào cụ thể (theo assignee, region, hoặc account type) để phân tích sâu hơn không?
```

**Output Type:** Chart + Table (cross-domain)

---

## SCENARIO COVERAGE MATRIX

| MCP Tool | Scenarios | Output Type |
|----------|-----------|-------------|
| `tool_get_jira_dashboard_data` | 1, 2, 3, 9, 10, 11, 14 | Chart + Table |
| `tool_get_sales_dashboard_data` | 5, 13, 15 | Chart + Table |
| `tool_get_partner_directory` | 4, 12 | Table |
| `tool_get_system_metadata` | 8 | Reference Data |
| `tool_get_jira_ticket_deepdive` | 6 | Detail View |
| `tool_get_partner_transaction_deepdive` | 7 | Table + Summary |

---

## NOTES FOR DEMO

1. **Scenario 1-2**: Mở đầu demo — show dashboard tổng quan
2. **Scenario 3-6**: Probe deeper — filter, investigate, validate
3. **Scenario 7-12**: Cross-check — sao kê, tra cứu
4. **Scenario 13-15**: Advanced — revenue analysis, cross-domain

---

*Document Version: 1.0*  
*Created: 18 May 2026*  
*Author: Tram (Sofia) Nguyen — BA Lead, GEX 1*