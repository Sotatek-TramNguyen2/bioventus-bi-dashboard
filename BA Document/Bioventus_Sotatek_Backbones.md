# Bioventus <> Sotatek — Backbones (Decision & Progress Tracker)

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
| 18 May 2026 | 1.1 | Update execution plan sau sync meeting | Tram (Sofia) Nguyen |

### Approval

| Approved date | Version | Approved by | Position |
|---------------|---------|-------------|----------|
| 18 May 2026 | 1.0 | Bryan Du | CDO |

---

## 1. PROJECT OVERVIEW

| Field | Value |
|-------|-------|
| Project Name | Tái Cấu Trúc Hệ Thống Báo Cáo Quản Trị Hiệu Suất Vận Hành & Phân Tích Khách Hàng |
| Client | BioVentus |
| Delivery Team | Sotatek — GEX 1 |
| Start Date | 18 May 2026 |
| Current Phase | PoC — Data Structure & Pipeline Research |

---

## 2. TEAM & PIC (Person In Charge)

| PIC | Role | Responsibility |
|-----|------|----------------|
| Tram (Sofia) Nguyen | BA Lead | Document, data structure design, pipeline research |
| Huynh | Tech Lead | GenBI scout, DB setup, synthetic data generation, Sota Agent integration |
| Team BA | Business Analysts | Research pipeline, MCP tool conversion, testing |
| Bryan Du | CDO (Sponsor) | Approval, strategic direction |

---

## 3. TIMELINE & MILESTONES

| # | Milestone | Deadline | PIC | Status |
|---|-----------|----------|-----|--------|
| 1 | Data structure draft hoàn thành | 19/05/2026 (Thứ 2) | Team BA | 🔄 In Progress |
| 2 | Scout GenBI — xác định deploy strategy | 19/05/2026 (Thứ 2) | Huynh | 🔄 In Progress |
| 3 | Gen synthetic records + DB connection | 20/05/2026 (Thứ 3) | Huynh | ⏳ Pending |
| 4 | Pipeline output (MCP tool) | 21/05/2026 (Thứ 4) | Team BA | ⏳ Pending |
| 5 | GenBI test output | 21/05/2026 (Thứ 4) | Huynh | ⏳ Pending |
| 6 | Internal Demo | 22/05/2026 (Thứ 5) | All | ⏳ Pending |

### Dependencies Flow

```
[Team BA: Data Structure] ──→ [Huynh: Gen Records + DB Connection] ──→ [Team BA: Pipeline/MCP Tool]
         (19/05)                        (20/05)                              (21/05)
                                                                     ──→ [Huynh: GenBI Test]
                                                                              (21/05)
                                                                                │
                                                                                ▼
                                                                     [Internal Demo - 22/05]
```

---

## 4. KEY DECISIONS LOG

| # | Date | Decision | Context / Reason | Decided by | Status |
|---|------|----------|------------------|-----------|--------|
| D-001 | 18/05/2026 | Evaluate 2 approaches song song: MCP Tool vs GenBI | Cần so sánh output thực tế trước khi commit vào 1 hướng | Team sync (BA + Huynh) | ✅ Confirmed |
| D-002 | 18/05/2026 | GenBI cần deploy instance riêng (KHÔNG dùng chung instance hiện tại) | Instance hiện tại gắn HR ERP data → nếu add DB Bioventus vào sẽ gây loạn output AI, không đúng expect | Huynh (đang scout) | 🔄 Pending confirmation |
| D-003 | — | Chọn approach chính: MCP Tool hay GenBI? | Sẽ quyết định sau Internal Demo dựa trên so sánh output | All | ⏳ Pending (target: 22/05) |

---

## 5. RISKS & BLOCKERS

| # | Risk/Blocker | Impact | Mitigation | Owner | Status |
|---|-------------|--------|-----------|-------|--------|
| R-001 | GenBI instance hiện tại dùng HR ERP data | Mix domain → AI output loạn, không mapping đúng Bioventus context | Deploy instance riêng hoặc tìm cách isolate DB | Huynh | 🔄 Investigating |
| R-002 | Data structure chưa xong → block toàn bộ downstream | Delay gen records, delay pipeline, delay demo | Team BA commit deadline 19/05 | Team BA | 🔄 In Progress |
| R-003 | Pipeline research chưa có precedent rõ ràng | Có thể cần thêm thời gian nếu approach không work | Chạy song song 2 hướng để hedge risk | Team BA + Huynh | ⏳ Monitoring |

---

## 6. MEETING NOTES & UPDATES

### 18/05/2026 — Sync Meeting (Team BA + Huynh)

**Attendees:** Tram (Sofia), Huynh, Team BA

**Key outcomes:**
1. Plan confirmed: Mai (19/05) team BA draft data structure dựa vào biz của Bioventus
2. Huynh đang scout GenBI của SotaLab — cần xác định: add thẳng DB vào được không hay phải deploy instance riêng
3. Sau khi có data structure → Huynh gen records + làm DB connection gắn vào Sota Agents
4. Team BA research pipeline gen BI từ database + dataset mock → convert thành MCP tool → test
5. Song song Huynh làm tương tự với GenBI → so sánh output

**Action items:**

| # | Action | PIC | Due |
|---|--------|-----|-----|
| 1 | Draft data structure (7 tables Star Schema) | Team BA | 19/05 |
| 2 | Confirm GenBI deploy strategy | Huynh | 19/05 |
| 3 | Gen synthetic records + DB connection | Huynh | 20/05 |
| 4 | Pipeline research + MCP tool test | Team BA | 21/05 |
| 5 | GenBI test + output evaluation | Huynh | 21/05 |

---

## 7. DOCUMENT REFERENCES

| Document | Purpose | Location |
|----------|---------|----------|
| Bioventus_Sotatek_SDD_Solution-Design-Document.md | Tổng quan solution & kiến trúc | ./Backbones/ |
| Bioventus_Sotatek_DataTable.md | Chi tiết từng table, setup guide, DAX measures | ./Backbones/ |
| Bioventus_Sotatek_Backbones.md | Decision log, timeline, PIC, risks (file này) | ./Backbones/ |

---

*Document Version: 1.1*  
*Last Updated: 18 May 2026*  
*Author: Tram (Sofia) Nguyen — BA Lead, GEX 1*  
*Approved by: Bryan Du — CDO*
