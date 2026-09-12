## Team

- Team: Fournity
- Members:  
      2A202602853 Hoàng Văn Nam  
      2A202602489 Nguyễn Hải Hoàng  
      2A202602977 Dương Hà Đức Anh  
      2A202603018 Tạ Đăng Dương  

---

# 01 - Problem Scan & Quick Cards (Vin Smart Future)

* **Học viên:** Cá nhân
* **Đơn vị:** Vin Smart Future (Vingroup)

---

## Phase 1 — SCAN: Bảng quét cơ hội (4 Lenses)

| # | Subsidiary | Lens | Mô tả ngắn bài toán |
|---|---|---|---|
| 1 | **Xanh SM (GSM)** | Tốn thời gian | Điều phối viên mất 15–20 phút xử lý thủ công các sự cố pin yếu giữa đường (tra cứu trụ trống, tính cự ly, gọi cứu hộ). |
| 2 | **VinFast** | Lặp lại | Đối chiếu và so khớp dữ liệu log sạc hàng tuần từ các trạm sạc đối tác ngoài với báo cáo quyết toán tài chính. |
| 3 | **Vinhomes** | AI-upgrade | Hệ thống phân loại và trích xuất mức độ khẩn cấp các phản ánh cư dân trên App Vinhomes Resident để chuyển đúng bộ phận. |
| 4 | **Vinmec** | Pain từ người khác | Bác sĩ mất 20–30 phút viết tóm tắt xuất viện (Discharge Summary) từ hồ sơ bệnh án điện tử, gây quá tải giờ làm việc. |
| 5 | **Vinpearl** | Tốn thời gian | Phân tích email đặt phòng theo đoàn (Group Booking) phức tạp để tự động kiểm tra quỹ phòng trống và draft báo giá. |

---

## Phase 2 — QUICK-ASSESS: 3 Quick Problem Cards

### QUICK PROBLEM CARD #1: Xanh SM — Điều phối cứu hộ pin & trạm sạc khẩn cấp
```text
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #1                                       │
│                                                             │
│ Bài toán: Tài xế Xanh SM báo cạn pin giữa đường cần hướng   │
│ dẫn trạm sạc gần nhất hoặc điều xe sạc pin lưu động.        │
│ Công ty thành viên: [x] Xanh SM (GSM)                       │
│                                                             │
│ Ai đang đau? Tài xế (chờ đợi), Điều phối viên (quá tải)     │
│                                                             │
│ Workflow thủ công hiện tại (5 bước):                        │
│   1. Tài xế gọi tổng đài báo pin yếu                        │
│   → 2. Điều phối viên tra cứu tọa độ GPS xe                 │
│   → 3. Tra cứu dashboard trạm sạc VinFast tìm cổng trống    │
│   → 4. Soạn tin nhắn hướng dẫn gửi app tài xế               │
│   → 5. Liên hệ đội cứu hộ nếu pin dưới ngưỡng an toàn       │
│                                                             │
│ Bước nào tốn nhất? Bước 3-4 ( 10-12 phút/lượt)              │
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 3 & 4            │
│ (Trích xuất GPS/Pin -> Tra cứu trạm -> Tự động draft tin)   │
│                                                             │
│ Đo thành công bằng gì (Metric có số)?                       │
│ Giảm thời gian xử lý từ 15 phút ──> dưới 3 phút/lượt.       │
│                                                             │
│ Quick Architecture: [x] LLM Feature                         │
└─────────────────────────────────────────────────────────────┘

```

---

### QUICK PROBLEM CARD #2: Vinhomes — Phân loại và route phản ánh cư dân

```text
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #2                                       │
│                                                             │
│ Bài toán: Phân loại khiếu nại/yêu cầu sửa chữa của cư dân    │
│ gửi qua App Vinhomes Resident đến đúng ban quản lý tòa nhà. │
│ Công ty thành viên: [x] Vinhomes                            │
│                                                             │
│ Ai đang đau? Cư dân (chờ lâu), Nhân viên CSKH (quá tải vé)  │
│                                                             │
│ Workflow thủ công hiện tại:                                 │
│   1. Cư dân gửi form/tin nhắn phản ánh tự do                │
│   → 2. Nhân viên CSKH đọc và phân loại thủ công             │
│   → 3. Gán thẻ nghiệp vụ và chuyển ticket sang ban kỹ thuật │
│   → 4. Soạn email/tin nhắn xác nhận tiếp nhận cho cư dân    │
│                                                             │
│ Bước nào tốn nhất? Bước 2 & 3 ( 6-8 phút/yêu cầu)           │
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 2, 3 & 4         │
│ (Trích xuất vấn đề, gán tag tự động, soạn nháp phản hồi)    │
│                                                             │
│ Đo thành công bằng gì (Metric có số)?                       │
│ Rút ngắn thời gian phân luồng từ 12 phút ──> dưới 1 phút;   │
│ độ chính xác điều hướng đạt trên 95%.                       │
│                                                             │
│ Quick Architecture: [x] Rule + LLM Feature                  │
└─────────────────────────────────────────────────────────────┘

```

---

### QUICK PROBLEM CARD #3: Vinmec — Soạn thảo tóm tắt hồ sơ xuất viện

```text
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #3                                       │
│                                                             │
│ Bài toán: Tổng hợp hồ sơ điều trị, kết quả xét nghiệm thành │
│ bản tóm tắt xuất viện bằng ngôn ngữ dễ hiểu cho bệnh nhân.  │
│ Công ty thành viên: [x] Vinmec                              │
│                                                             │
│ Ai đang đau? Bác sĩ điều trị / Trợ lý y khoa                │
│                                                             │
│ Workflow thủ công hiện tại:                                 │
│   1. Rà soát kết quả cận lâm sàng và chỉ định thuốc         │
│   → 2. Viết diễn tiến lâm sàng và đánh giá phục hồi         │
│   → 3. Đánh máy dặn dò dùng thuốc và lịch tái khám          │
│   → 4. In ấn và ký phê duyệt hồ sơ ra viện                  │
│                                                             │
│ Bước nào tốn nhất? Bước 1 & 2 ( 20-25 phút/hồ sơ)           │
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 2 & 3            │
│ (Tổng hợp dữ liệu có cấu trúc từ EMR -> Soạn nháp tóm tắt)  │
│                                                             │
│ Đo thành công bằng gì (Metric có số)?                       │
│ Giảm thời gian soạn thảo từ 25 phút ──> dưới 5 phút/ca;     │
│ 100% hồ sơ phải được Bác sĩ kiểm duyệt (HITL).              │
│                                                             │
│ Quick Architecture: [x] LLM Feature                         │
└─────────────────────────────────────────────────────────────┘

```