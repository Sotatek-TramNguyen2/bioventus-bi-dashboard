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

Lấy toàn bộ dữ liệu tổng hợp và chi tiết để kết xuất Dashboard phân tích hiệu suất team BI (Jira Tickets). Trả về: kpi_metrics (Total Tickets, Average Days to Complete), chart_data (số liệu group sẵn theo Assignee, Status, RequestType, Year, Month), detail_table (danh sách ticket chi tiết gồm IssueKey, Summary, Assignee, Reporter, Status, RequestType, DaysToComplete). Output: Chart + Table.
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

Lấy toàn bộ dữ liệu tổng hợp và chi tiết để kết xuất Dashboard Kinh doanh và Đối tác y tế. Trả về: kpi_metrics (Total Revenue, Total Quantity), chart_data (doanh thu & số lượng group theo AccountType, Region, Year, Month), detail_table (danh sách giao dịch gồm TransactionID, PartnerName, AccountType, Region, OrderDate, Quantity, NetValue). Output: Chart + Table.
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

Tra cứu và phân trang danh mục đối tác khách hàng từ bảng Dim_Partner_Account. Hữu ích khi AI cần tìm chính xác tên một bệnh viện/bác sĩ trước khi query giao dịch. Trả về: danh sách PartnerID, PartnerName, AccountType, Region. Output: Table.
Params:
- search_term: Tìm kiếm gần đúng theo PartnerName (chuỗi text). Để trống = tất cả.
- account_type: Phân loại đối tác. Để trống = tất cả.
- region: Khu vực. Để trống = tất cả.
- limit: Số bản ghi tối đa, mặc định 100.

tool_get_partner_directory

---

tool_get_system_metadata

Lấy danh sách các giá trị hợp lệ (Valid values) của các bộ lọc (Filters/Dimensions) trong toàn bộ hệ thống. Trả về: danh sách mảng JSON chứa Users (từ Dim_User), Statuses (từ Dim_Status), RequestTypes (từ Dim_RequestType), AccountTypes và Regions (từ Dim_Partner_Account). Output: Reference Data (không trực tiếp sinh chart/table, AI dùng để validate trước khi gọi tool khác).
Params:
- (Không yêu cầu tham số)

Ghi chú: AI Agent sẽ tự động gọi tool này khi nó không chắc chắn về tên của một Trạng thái, Loại request, hoặc Loại đối tác có tồn tại trong Database hay không.

tool_get_system_metadata

---

tool_get_jira_ticket_deepdive

Truy xuất lịch sử và thông tin siêu chi tiết của MỘT ticket duy nhất nhằm phục vụ cho mục đích điều tra (Investigate) trực tiếp. Trả về: toàn bộ trường dữ liệu có trong Fact_JiraTicket và các bảng Dim liên quan tương ứng với Ticket đó (IssueKey, Summary, Assignee, Reporter, Status, RequestType, CreatedDate, ResolvedDate, DaysToComplete). Output: Detail View (hiển thị chi tiết 1 ticket, có thể render thành formatted card hoặc table row).
Params:
- issue_key: Bắt buộc. Mã ticket thực tế trên Jira, ví dụ "BI-1411", "BI-1429".

tool_get_jira_ticket_deepdive

---

tool_get_partner_transaction_deepdive

Truy xuất chi tiết tất cả các giao dịch (Transactions) của MỘT đối tác cụ thể để làm báo cáo công nợ hoặc sao kê chi tiết. Trả về: danh sách TransactionID, OrderDate, Quantity, NetValue và summary (total_transactions, total_quantity, total_net_value). Output: Table + Summary.
Params:
- partner_id: Bắt buộc. PartnerID của đối tác cần sao kê.
- date_from: Lọc từ ngày, định dạng "YYYY-MM-DD". Để trống = không lọc.
- date_to: Lọc đến ngày, định dạng "YYYY-MM-DD". Để trống = không lọc.
- limit: Số bản ghi tối đa, mặc định 100.

tool_get_partner_transaction_deepdive

---

## OUTPUT TYPES REFERENCE

| Output Type | Mô tả | Dùng khi |
|-------------|-------|----------|
| Chart + Table | KPI metrics + chart_data + detail_table | Dashboard tổng hợp |
| Chart | KPI metrics + chart_data | Câu hỏi về xu hướng, so sánh, phân bổ |
| Table | detail_table hoặc danh sách items | Tra cứu, tìm kiếm, sao kê |
| Table + Summary | detail_table + summary metrics | Báo cáo công nợ, sao kê đối tác |
| Detail View | Single record với đầy đủ fields | Điều tra ticket/giao dịch cụ thể |
| Reference Data | Danh sách valid values | AI validate trước khi query chính |

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

*Document Version: 1.0*  
*Created: 18 May 2026*  
*Author: Tram (Sofia) Nguyen — BA Lead, GEX 1*