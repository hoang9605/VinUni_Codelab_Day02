## Team

- Team: Fournity
- Members:  
      2A202602853 Hoàng Văn Nam  
      2A202602489 Nguyễn Hải Hoàng  
      2A202602977 Dương Hà Đức Anh  
      2A202603018 Tạ Đăng Dương  

---y

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
│ Bước nào tốn nhất? Bước 3-4 (⏱ 10-12 phút/lượt)             │
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 3 & 4            │
│ (Trích xuất GPS/Pin -> Tra cứu trạm -> Tự động draft tin)   │
│                                                             │
│ Đo thành công bằng gì (Metric có số)?                        │
│ Giảm thời gian xử lý từ 15 phút ──> dưới 3 phút/lượt.       │
│                                                             │
│ Quick Architecture: [x] LLM Feature                         │
└─────────────────────────────────────────────────────────────┘