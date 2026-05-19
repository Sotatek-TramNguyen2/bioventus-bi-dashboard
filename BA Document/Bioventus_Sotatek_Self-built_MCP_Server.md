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

## 1. MCP SERVER OVERVIEW

### 1.1. Mục đích

Self-built MCP Server đóng vai trò bridge giữa **SotaAgent (AI Chatbot)** và **Synthetic Database (SQLite/PostgreSQL)** — cho phép AI Agent truy vấn dữ liệu Bioventus một cách an toàn và chính xác.

### 1.2. Kiến trúc

```
┌─────────────┐      SSE Request       ┌────────────────────┐      SQL Query      ┌────────────┐
│  SotaAgent  │ ──────────────────────▶ │  Self-built MCP    │ ──────────────────▶  │  SQLite /  │
│  (AI Chat)  │                       │  Server             │                      │  PostgreSQL│
└─────────────┘      JSON Response     └────────────────────┘      Result Set       └────────────┘
```

### 1.3. Database Schema

MCP Server kết nối với Synthetic Database theo mô hình Star Schema:

| # | Table | Type | Mô tả |
|---|-------|------|-------|
| 1 | Dim_User | Dimension | Nhân sự (Assignee + Reporter) |
| 2 | Dim_Status | Dimension | Trạng thái ticket |
| 3 | Dim_RequestType | Dimension | Loại yêu cầu |
| 4 | Dim_Date | Dimension | Lịch (dùng chung) |
| 5 | Dim_Partner_Account | Dimension | Đối tác SAP |
| 6 | Fact_JiraTicket | Fact | Ticket Jira |
| 7 | Fact_PartnerTransaction | Fact | Giao dịch SAP |

---

## 2. TOOL DEFINITIONS

### 2.1. tool_get_jira_dashboard_data

**Mục đích:** Lấy toàn bộ dữ liệu tổng hợp và chi tiết để kết xuất Dashboard phân tích hiệu suất team BI (Jira Tickets).

**Return Structure (JSON):**

```json
{
  "kpi_metrics": {
    "total_tickets": 661,
    "avg_days_to_complete": 4.5
  },
  "chart_data": [
    { "group_by": "Assignee", "label": "Rahime", "value": 71 },
    { "group_by": "Status", "label": "Done", "value": 412 },
    { "group_by": "RequestType", "label": "New Report", "value": 185 }
  ],
  "detail_table": [
    {
      "issue_key": "BI-1411",
      "summary": "Update dashboard sales report",
      "assignee": "Rahime Cvetkovic",
      "reporter": "Koh Higaki",
      "status": "Done",
      "request_type": "Update Existing Report",
      "days_to_complete": 3
    }
  ]
}
```

**Parameters:**

| Param | Type | Required | Default | Mô tả |
|-------|------|----------|---------|-------|
| assignee_name | String | No | "" (all) | Tên người xử lý (từ Dim_User) |
| reporter_name | String | No | "" (all) | Tên người tạo yêu cầu (từ Dim_User) |
| status_name | String | No | "" (all) | Trạng thái ticket (từ Dim_Status), VD: "Done", "In Progress" |
| request_type | String | No | "" (all) | Loại yêu cầu (từ Dim_RequestType), VD: "New Report", "App Permission" |
| date_from | String | No | null | Lọc từ ngày tạo, định dạng "YYYY-MM-DD" |
| date_to | String | No | null | Lọc đến ngày tạo, định dạng "YYYY-MM-DD" |
| detail_limit | Integer | No | 50 | Số bản ghi tối đa trong detail_table |

---

### 2.2. tool_get_sales_dashboard_data

**Mục đích:** Lấy toàn bộ dữ liệu tổng hợp và chi tiết để kết xuất Dashboard Kinh doanh và Đối tác y tế.

**Return Structure (JSON):**

```json
{
  "kpi_metrics": {
    "total_revenue": 15000000.00,
    "total_quantity": 45000
  },
  "chart_data": [
    { "group_by": "AccountType", "label": "Hospitals", "value": 8500000.00 },
    { "group_by": "Region", "label": "Northeast", "value": 4200000.00 }
  ],
  "detail_table": [
    {
      "transaction_id": "SO-2026-001234",
      "partner_name": "Mayo Clinic",
      "account_type": "Hospitals",
      "region": "Midwest",
      "order_date": "2026-05-15",
      "quantity": 150,
      "net_value": 75000.00
    }
  ]
}
```

**Parameters:**

| Param | Type | Required | Default | Mô tả |
|-------|------|----------|---------|-------|
| account_type | String | No | "" (all) | Phân loại đối tác (từ Dim_Partner_Account), VD: "Hospitals", "Patients", "Distributors" |
| region | String | No | "" (all) | Khu vực phân phối (từ Dim_Partner_Account) |
| partner_name | String | No | "" (all) | Tên đối tác cụ thể |
| date_from | String | No | null | Lọc từ ngày giao dịch, định dạng "YYYY-MM-DD" |
| date_to | String | No | null | Lọc đến ngày giao dịch, định dạng "YYYY-MM-DD" |
| detail_limit | Integer | No | 50 | Số bản ghi tối đa trong detail_table |

---

### 2.3. tool_get_partner_directory

**Mục đích:** Tra cứu và phân trang danh mục đối tác khách hàng từ bảng Dim_Partner_Account. Hữu ích khi AI cần tìm chính xác tên một bệnh viện/bác sĩ trước khi query giao dịch.

**Return Structure (JSON):**

```json
{
  "partners": [
    {
      "partner_id": "BP-001234",
      "partner_name": "Mayo Clinic",
      "account_type": "Hospitals",
      "region": "Midwest"
    }
  ],
  "total_count": 350,
  "returned_count": 100
}
```

**Parameters:**

| Param | Type | Required | Default | Mô tả |
|-------|------|----------|---------|-------|
| search_term | String | No | null | Tìm kiếm gần đúng theo PartnerName (chuỗi text) |
| account_type | String | No | "" (all) | Phân loại đối tác |
| region | String | No | "" (all) | Khu vực |
| limit | Integer | No | 100 | Số bản ghi tối đa |

---

### 2.4. tool_get_system_metadata

**Mục đích:** Lấy danh sách các giá trị hợp lệ (Valid values) của các bộ lọc trong toàn bộ hệ thống. Đây là tool "cứu cánh" — giúp AI tránh ảo giác (bịa ra tên user sai, status không tồn tại).

> **Ghi chú quan trọng:** AI Agent sẽ tự động gọi tool này khi nó không chắc chắn về tên của một Trạng thái, Loại request, hoặc Loại đối tác có tồn tại trong Database hay không.

**Return Structure (JSON):**

```json
{
  "users": [
    { "user_name": "Rahime Cvetkovic", "department": "BI Engineering" },
    { "user_name": "Koh Higaki", "department": "BI Engineering" }
  ],
  "statuses": [
    { "status_name": "New", "status_category": "Open" },
    { "status_name": "In Progress", "status_category": "In Work" },
    { "status_name": "Development", "status_category": "In Work" },
    { "status_name": "QA UAT", "status_category": "In Work" },
    { "status_name": "PROD/UAT", "status_category": "In Work" },
    { "status_name": "On Hold", "status_category": "Open" },
    { "status_name": "Done", "status_category": "Closed" }
  ],
  "request_types": [
    { "request_type_name": "Update Existing Report/Dataset" },
    { "request_type_name": "App Permission" },
    { "request_type_name": "New Report" },
    { "request_type_name": "Report an Issue" },
    { "request_type_name": "New Dataset" },
    { "request_type_name": "Data Quality Investigation" }
  ],
  "account_types": [
    "Patients",
    "Hospitals",
    "Doctors/Physicians",
    "Clinics",
    "Insurance/Payer",
    "Distributor",
    "ASC"
  ],
  "regions": [
    "Northeast",
    "Midwest",
    "Southeast",
    "Southwest",
    "West"
  ]
}
```

**Parameters:** Không yêu cầu tham số.

---

### 2.5. tool_get_jira_ticket_deepdive

**Mục đích:** Truy xuất lịch sử và thông tin siêu chi tiết của MỘT ticket duy nhất nhằm phục vụ cho mục đích điều tra (Investigate) trực tiếp.

**Return Structure (JSON):**

```json
{
  "ticket": {
    "ticket_key": 1001,
    "issue_key": "BI-1411",
    "summary": "Update dashboard sales report",
    "assignee": {
      "user_name": "Rahime Cvetkovic",
      "department": "BI Engineering",
      "email": "rahime@bioventus.com"
    },
    "reporter": {
      "user_name": "Koh Higaki",
      "department": "Operations",
      "email": "koh@bioventus.com"
    },
    "status": {
      "status_name": "Done",
      "status_category": "Closed"
    },
    "request_type": {
      "request_type_name": "Update Existing Report"
    },
    "created_date": "2026-05-01",
    "resolved_date": "2026-05-04",
    "days_to_complete": 3
  }
}
```

**Parameters:**

| Param | Type | Required | Default | Mô tả |
|-------|------|----------|---------|-------|
| issue_key | String | Yes | — | Mã ticket thực tế trên Jira (VD: "BI-1411", "BI-1429") |

---

### 2.6. tool_get_partner_transaction_deepdive

**Mục đích:** Truy xuất chi tiết tất cả các giao dịch của MỘT đối tác cụ thể để làm báo cáo công nợ hoặc sao kê chi tiết.

**Return Structure (JSON):**

```json
{
  "partner": {
    "partner_id": "BP-001234",
    "partner_name": "Mayo Clinic",
    "account_type": "Hospitals",
    "region": "Midwest"
  },
  "transactions": [
    {
      "transaction_id": "SO-2026-001234",
      "order_date": "2026-05-15",
      "quantity": 150,
      "net_value": 75000.00
    }
  ],
  "summary": {
    "total_transactions": 12,
    "total_quantity": 1800,
    "total_net_value": 900000.00
  }
}
```

**Parameters:**

| Param | Type | Required | Default | Mô tả |
|-------|------|----------|---------|-------|
| partner_id | String | Yes | — | PartnerID của đối tác cần sao kê |
| date_from | String | No | null | Lọc từ ngày, định dạng "YYYY-MM-DD" |
| date_to | String | No | null | Lọc đến ngày, định dạng "YYYY-MM-DD" |
| limit | Integer | No | 100 | Số bản ghi tối đa |

---

## 3. DEVELOPER NOTES

### 3.1. Xử lý chart_data

Khi query cho phần `chart_data`, sử dụng GROUP BY để gom nhóm dữ liệu:

```sql
-- Ví dụ: Group by Assignee
SELECT u.user_name AS label, COUNT(*) AS value
FROM Fact_JiraTicket f
JOIN Dim_User u ON f.assignee_key = u.user_key
GROUP BY u.user_name
ORDER BY value DESC;
```

**Format trả về:** Key-Value pairs để AI của SotaAgent dễ dàng map vào trục X (label) và trục Y (value):

```json
[
  { "label": "Rahime", "value": 71 },
  { "label": "Koh", "value": 58 },
  { "label": "Shankar", "value": 45 }
]
```

### 3.2. Xử lý Date Filter

- `date_from` và `date_to` luôn filter theo `CreatedDate` (bảng Fact_JiraTicket)
- Format chuẩn: "YYYY-MM-DD" (VD: "2026-05-19")
- Nếu để trống → không áp dụng filter

### 3.3. Giới hạn (Limits)

- `detail_limit` mặc định = 50, tối đa = 500
- `limit` trong partner_directory mặc định = 100, tối đa = 1000
- Luôn trả về `total_count` để AI biết có bao nhiêu bản ghi thực tế

### 3.4. Error Handling

| Error Code | Message | Giải thích |
|------------|---------|------------|
| E001 | "Invalid date format" | date_from/date_to không đúng "YYYY-MM-DD" |
| E002 | "Invalid filter value" | Giá trị filter không tồn tại trong DB → gợi ý gọi tool_get_system_metadata |
| E003 | "Partner not found" | partner_id không tồn tại |
| E004 | "Ticket not found" | issue_key không tồn tại |

---

## 4. TESTING CHECKLIST

Trước khi demo, đảm bảo:

- [ ] Tất cả 6 tools trả về đúng JSON structure
- [ ] date_from/date_to filter hoạt động đúng
- [ ] detail_limit giới hạn đúng số bản ghi
- [ ] tool_get_system_metadata trả về đầy đủ valid values
- [ ] Error codes E001-E004 hoạt động đúng
- [ ] GROUP BY cho chart_data trả về format Key-Value

---

*Document Version: 1.0*  
*Created: 18 May 2026*  
*Author: Tram (Sofia) Nguyen — BA Lead, GEX 1*