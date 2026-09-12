# 02 - Deep Dive Report: Xanh SM Intelligent Dispatcher Support

* **Đơn vị:** Vin Smart Future (Vingroup)
* **Dự án:** Trợ lý điều vận hỗ trợ sự cố sạc & điều phối cứu hộ pin (Xanh SM)

---

## 1. Problem Statement (6-field) — Vin Smart Future Standard

| Field | Nội dung chi tiết |
|---|---|
| **1. Actor / Operator** | Điều phối viên (Dispatcher) tại Trung tâm Điều vận Xanh SM (GSM). |
| **2. Current Workflow** | Khi tài xế báo cạn pin: (1) Tiếp nhận qua app/hotline, (2) Tra cứu GPS xe trên bản đồ nội bộ, (3) Mở Dashboard tìm trạm sạc VinFast còn trụ trống, (4) Soạn tin nhắn hướng dẫn gửi app tài xế, (5) Điều xe cứu hộ nếu pin dưới 5%. Gồm 5 bước xử lý thủ công, mất 15 phút/lượt. |
| **3. Bottleneck** | Bước 3 & 4 (mất 10–12 phút/lượt): Tra cứu trụ sạc còn cổng phù hợp dòng xe (VF5/VF8) và soạn thảo tin nhắn hướng dẫn đường đi bằng văn phong chuẩn mực. |
| **4. Business Impact** | ~80 sự cố pin/ngày tại Hà Nội; lãng phí ~20 giờ làm việc/ngày của khối điều vận; cuốc xe bị gián đoạn làm giảm 15% hiệu suất khai thác cuốc và gây stress cho tài xế. |
| **5. Success Metric** | 1. Rút ngắn thời gian xử lý sự cố từ 15 phút xuống dưới 3 phút.<br>2. Tỷ lệ đề xuất trạm đúng cổng sạc và tuyệt đối an toàn pin đạt 100%. |
| **6. Operational Boundary** | AI được phép đọc dữ liệu telemetry (vị trí GPS, % pin) và soạn draft tin nhắn. **CẤM:** AI không được tự động gửi tin nhắn đến tài xế mà thiếu tiền tố `[DRAFT_ONLY]` hoặc chưa qua điều phối viên phê duyệt (HITL); cấm hướng dẫn trạm sạc > 5km khi pin < 5%. |

---

## 2. Future-State Flow & AI Fit

* **Đánh giá AI Fit:** **LLM Feature** (Quy trình nghiệp vụ có luồng cố định, dùng LLM để trích xuất ngữ cảnh và draft tin nhắn hỗ trợ; không áp dụng Agent tự trị để phòng ngừa rủi ro xe cạn pin giữa đường).
* **Quy trình tương lai (Future-State Flow):**
  * **Bước 1 (Nhận tin):** Hệ thống nhận tọa độ GPS và mức pin từ xe tài xế.
  * **Bước 2 (AI Step 🔵):** Phân tích dữ liệu: Nếu pin < 5%, từ chối trạm sạc xa và xuất cấu trúc JSON điều xe cứu hộ; nếu pin an toàn, soạn nháp hướng dẫn. Mọi kết quả bắt buộc bắt đầu bằng thẻ `[DRAFT_ONLY]`.
  * **Bước 3 (Human-in-the-loop 🟢):** Điều phối viên nhấn phê duyệt gửi tin nhắn hoặc xác nhận lệnh điều xe cứu hộ pin.
  * **Bước 4 (Fallback ↩️):** Nếu API gặp lỗi hoặc phản hồi không chắc chắn, hệ thống chuyển giao cho điều phối viên xử lý thủ công theo quy trình truyền thống.

---

## 3. Đánh giá độ sẵn sàng & Quyết định (Evaluate)

* **Checklist kiểm tra:**
  * [x] Dữ liệu mẫu/logs xe và vị trí trạm sạc có sẵn qua API của GSM & VinFast.
  * [x] Rủi ro vận hành được kiểm soát chặt chẽ bằng cơ chế Human-in-the-loop và tiền tố `[DRAFT_ONLY]`.
  * [x] Đội ngũ điều phối viên sẵn sàng tiếp nhận công cụ co-pilot để giảm tải giờ cao điểm.

* **Quyết định cuối cùng:** **GO (Bắt đầu xây dựng Prototype)**
* **Lý giải (Justification):** Phạm vi bài toán rõ ràng, rủi ro an toàn được khống chế bằng ranh giới lập trình sẵn trong prompt, giúp giải phóng hơn 75% thời gian xử lý mỗi sự cố và tăng năng suất điều xe.