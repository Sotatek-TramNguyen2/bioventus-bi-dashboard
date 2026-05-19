# Bioventus <> Sotatek — Solution Design Document (SDD)

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

## 1. BỐI CẢNH & BÀI TOÁN CỦA BIOVENTUS

### 1.1. Về doanh nghiệp

**BioVentus** là tập đoàn đa quốc gia chuyên **Sản xuất và Phân phối Thiết bị Y tế / Dược phẩm sinh học kỹ thuật cao** (thiết bị hỗ trợ phẫu thuật, phục hồi và chấn thương chỉnh hình). Mô hình kinh doanh **B2B2C phức hợp** — sản phẩm cung ứng qua chuỗi đại lý, bệnh viện, người dùng cuối là bác sĩ và bệnh nhân.

### 1.2. Hạ tầng công nghệ hiện tại

* **ERP lõi:** SAP S/4 HANA
* **Data Warehouse:** SQL Server (legacy)
* **BI Tool:** Power BI (hơn 2,000 báo cáo tồn tại)
* **Tích hợp:** EMR/EHR, FDA compliance workflows
* **Staging tables:** `STG_SAP_BUT000`, `STG_SAP_KNA1`, `STG_SAP_KNVP`

### 1.3. Nỗi đau nghiệp vụ (Pain Points)

| # | Pain Point | Impact |
|---|-----------|--------|
| 1 | Hơn 2,000 báo cáo nhưng adoption rate cực thấp | Team BI quá tải ticket vụn vặt, không có thời gian cho strategic work |
| 2 | Thiếu KPI Cataloging & Data QA | Báo cáo trùng lặp, không nhất quán, mất tin tưởng từ business |
| 3 | SQL Server không scale | Bottleneck khi data volume tăng |
| 4 | Chưa có self-service analytics | Mọi request đều phải qua BI team |

### 1.4. Mục tiêu chiến lược

* **Ngắn hạn:** Managed Services Model — chuẩn hóa Dashboard/KPI, đạt Immediate ROI
* **Trung hạn:** Migrate lên SAP RISE (Cloud) + Databricks/Snowflake
* **Dài hạn:** Descriptive → Predictive/Prescriptive qua Digital Factory & Digital Twin

---

## 2. GIẢI PHÁP ĐỀ XUẤT CỦA SOTATEK

### 2.1. Tổng quan Solution

Sotatek đề xuất xây dựng **AI-powered BI Generation Pipeline** — cho phép tự động sinh dashboard từ database schema + business context, giảm thiểu workload thủ công của team BI Bioventus.

### 2.2. Kiến trúc giải pháp

```
┌─────────────────────────────────────────────────────────────────┐
│                    BIOVENTUS DATA SOURCES                        │
│  SAP S/4 HANA  │  Jira  │  SQL Server DW  │  EMR/EHR          │
└────────┬────────────┬──────────┬─────────────────────────────────┘
         │            │          │
         ▼            ▼          ▼
┌─────────────────────────────────────────────────────────────────┐
│              SOTATEK AI BI PLATFORM                              │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                     SotaAgent                            │   │
│  │  (AI Chatbot — giao diện hội thoại cho business user)   │   │
│  │                                                         │   │
│  │  ┌─────────────────────────────────────────────────┐    │   │
│  │  │  MCP Protocol                                  │    │   │
│  │  │  (Model Context Protocol — kết nối Agent ↔ DB) │    │   │
│  │  └─────────────────────────────────────────────────┘    │   │
│  │                         │                                │   │
│  │                         ▼                                │   │
│  │  ┌───────────────┐    ┌──────────────┐                  │   │
│  │  │  DB Connector │───▶│  AI Engine   │                  │   │
│  │  │  (MCP Client) │    │  (Query +    │                  │   │
│  │  │               │    │   Render)    │                  │   │
│  │  └───────────────┘    └──────────────┘                  │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────────────┐
│                    OUTPUT                                        │
│  NLP Response  │  Data Table  │  Chart Visualization            │
└─────────────────────────────────────────────────────────────────┘
```

### 2.3. Giải pháp đề xuất: MCP Protocol trong SotaAgent

**SotaAgent** là một AI chatbot có sẵn MCP Protocol — cho phép kết nối trực tiếp với database và trả về kết quả dạng NLP response kết hợp bảng và chart.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      MCP INTERACTION FLOW                                    │
└─────────────────────────────────────────────────────────────────────────────┘

  User                  SotaAgent                  MCP Server               Database
   │                         │                          │                      │
   │  "Top 10 user có ticket │                          │                      │
   │   pending cao nhất?"     │                          │                      │
   │────────────────────────▶│                          │                      │
   │                         │                          │                      │
   │              ┌───────────┴───────────┐              │                      │
   │              │ AI phân tích câu hỏi   │              │                      │
   │              │Nhận ra: cần gọi MCP    │              │                      │
   │              │để query database        │              │                      │
   │              └───────────┬───────────┘              │                      │
   │                         │                          │                      │
   │                         │  SSE Request (SQL query) │                      │
   │                         │─────────────────────────▶│                      │
   │                         │                          │                      │
   │                         │              ┌───────────┴──────────┐           │
   │                         │              │ MCP thực thi query    │           │
   │                         │              │ Lấy data từ DB      │           │
   │                         │              └───────────┬──────────┘           │
   │                         │                          │                      │
   │                         │         JSON Result      │                      │
   │                         │◀─────────────────────────│                      │
   │                         │                          │                      │
   │              ┌───────────┴───────────┐              │                      │
   │              │ AI tổng hợp + render   │              │                      │
   │              │ NLP + Table + Chart    │              │                      │
   │              └───────────┬───────────┘              │                      │
   │                         │                          │                      │
   │     "Top 10 user có ticket│                          │                      │
   │      pending cao nhất:   │                          │                      │
   │      1. Rahime: 25       │                          │                      │
   │      2. Koh: 18          │                          │                      │
   │      [Bar Chart]"         │                          │                      │
   │◀─────────────────────────│                          │                      │
```

#### Luồng tương tác 5 bước (RAG / Tool Calling):

| # | Step | Mô tả |
|---|------|-------|
| 1 | **User đặt câu hỏi** | "Top 10 user có ticket pending cao nhất?" |
| 2 | **AI Phân tích (Reasoning)** | SotaAgent nhận ra: "Cần gọi MCP để query database, không có sẵn trong context" |
| 3 | **Gửi Request qua MCP** | SotaAgent gửi SSE request chứa SQL query do AI tự sinh ra |
| 4 | **MCP Server Thực thi** | Server nhận request, chạy SQL vào Database, trả JSON result ngược lại |
| 5 | **Tổng hợp & Hiển thị** | SotaAgent nhận JSON → tổng hợp thành NLP response kết hợp Table + Chart |

**Output mẫu:**
```
Top 10 user có ticket pending cao nhất:

| Rank | User        | Pending Tickets |
|------|-------------|------------------|
| 1    | Rahime      | 25               |
| 2    | Koh         | 18               |
...

[Bar Chart]

→ NLP: "Rahime đang có 25 ticket pending, cao nhất team BI. Bạn có muốn xem chi tiết không?"
```

#### Hai hướng triển khai song song:

**Hướng 1: Self-built MCP Server**
- SotaAgent (AI Chatbot) nhận câu hỏi → gọi MCP Server tự build, kết nối trực tiếp với SQLite/PostgreSQL (dataset nằm ngoài)
- MCP Server nhận SSE request, thực thi SQL query, trả JSON result về cho SotaAgent
- SotaAgent tổng hợp dữ liệu → trả về NLP response kết hợp Table + Chart trên giao diện Chatbot

**Hướng 2: Power BI MCP**
- SotaAgent (AI Chatbot) nhận câu hỏi → gọi MCP Server của Power BI
- MCP Server đóng vai trò bridge, kết nối với Power BI Dataset (đã import sẵn)
- SotaAgent nhận kết quả → trả về NLP response kết hợp Table + Chart trên giao diện Chatbot

| Tiêu chí | Hướng 1: Self-built MCP | Hướng 2: Power BI MCP |
|----------|------------------------|----------------------|
| Độ phức tạp | Cần build from scratch | Có sẵn, deploy nhanh |
| Tính linh hoạt | Cao — custom business logic | Phụ thuộc Power BI |
| Dataset | SQLite/PostgreSQL (external DB) | Power BI Dataset (import mode) |
| Output | NLP + Table + Chart trên Chatbot | NLP + Table + Chart trên Chatbot |
| Chi phí license | Không phụ thuộc | Phụ thuộc Power BI license |

> **Quyết định cuối cùng:** Sẽ được xác định sau Internal Demo ngày 22/05/2026, dựa trên so sánh output thực tế của cả 2 hướng.

### 2.4. Phạm vi Demo (Proof of Concept)

Demo thực hiện trên **SotaAgent** với cùng 1 câu hỏi nghiệp vụ nhưng sử dụng 2 MCP Server khác nhau để so sánh:

| Demo | MCP Server | Dataset |
|------|------------|---------|
| **Demo 1** | Self-built MCP | SQLite (external DB) |
| **Demo 2** | Power BI MCP | Power BI Dataset (import) |

So sánh output để đánh giá độ chính xác và performance.

### 2.5. Deliverables

| # | Deliverable | Mô tả |
|---|-------------|--------|
| 1 | Synthetic Database | Mock data matching Bioventus schema (7 tables, Star Schema) |
| 2 | AI BI Chatbot | SotaAgent với MCP Protocol — trả về NLP response + Table + Chart |
| 3 | Comparison Report | So sánh output giữa Self-built MCP và Power BI MCP |

---

## 3. GIÁ TRỊ MANG LẠI CHO BIOVENTUS

| Pain Point | Hiện tại | Sau khi áp dụng Solution |
|------------|----------|--------------------------|
| Hơn 2,000 báo cáo nhưng adoption rate thấp | Business user phải chờ BI team tạo report | AI Chatbot tự trả lời câu hỏi ngay, không cần qua BI team |
| Thiếu KPI Cataloging & Data QA | Report trùng lặp, không nhất quán | AI tự động sinh metric chuẩn hóa từ 1 nguồn dữ liệu |
| SQL Server không scale | Bottleneck khi data tăng | MCP Server query trực tiếp, không phụ thuộc SQL Server legacy |
| Chưa có self-service analytics | Mọi request đều qua BI team → quá tải | Business user tự hỏi AI → giảm ticket backlog cho BI team |

---

*Document Version: 1.0*  
*Created: 18 May 2026*  
*Author: Tram (Sofia) Nguyen — BA Lead, GEX 1*  
*Approved by: Bryan Du — CDO*
