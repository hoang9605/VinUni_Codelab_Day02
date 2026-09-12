## Team

- Team: Fournity
- Members:  
      2A202602853 Hoàng Văn Nam  
      2A202602489 Nguyễn Hải Hoàng  
      2A202602977 Dương Hà Đức Anh  
      2A202603018 Tạ Đăng Dương  

---

# 02 - Deep Dive Report: Xanh SM Intelligent Dispatcher Support

* **Đơn vị:** Vin Smart Future (Vingroup)
* **Dự án:** Trợ lý điều vận hỗ trợ sự cố sạc & điều phối cứu hộ pin (Xanh SM)

---

## 1. Problem Statement (6-field) — Vin Smart Future Standard

| Field | Nội dung chi tiết |
|---|---|
| **1. Actor / Operator** | Điều phối viên (Dispatcher) tại Trung tâm Điều vận Xanh SM (GSM). |
| **2. Current Workflow** | Khi tài xế báo cạn pin: (1) Nhận thông tin qua hotline/app, (2) Tra cứu GPS xe trên bản đồ nội bộ, (3) Tìm trạm sạc VinFast còn trụ trống, (4) Soạn tin nhắn hướng dẫn gửi tài xế, (5) Điều xe cứu hộ nếu pin cạn kiệt. Tổng cộng 5 bước xử lý thủ công. |
| **3. Bottleneck** | Bước 3 & 4 (mất 10–12 phút/lượt): Tra cứu trụ sạc còn cổng phù hợp với loại xe và gõ văn bản hướng dẫn chi tiết kèm cảnh báo an toàn. |
| **4. Business Impact** | Khoảng 80 ca sự cố pin/ngày tại Hà Nội; lãng phí ~20 giờ làm việc/ngày của nhân sự điều vận; xe dừng đón khách gây giảm 15% hiệu suất khai thác cuốc và gây áp lực tâm lý cho tài xế. |
| **5. Success Metric** | 1. Giảm thời gian xử lý sự cố từ 15 phút xuống dưới 3 phút (Hiệu suất).<br>2. Tỷ lệ điều hướng đúng loại trụ sạc và an toàn pin đạt 100% (An toàn vận hành). |
| **6. Operational Boundary** | AI được phép đọc dữ liệu telemetry (vị trí GPS, % pin) và draft tin nhắn hỗ trợ. **CẤM:** AI không được tự động gửi tin nhắn đến tài xế mà thiếu nhãn `[DRAFT_ONLY]` hoặc chưa qua phê duyệt của Điều phối viên (HITL); cấm hướng dẫn trạm sạc > 5km khi pin < 5%. |

---

## 2. Future-State Flow & AI Fit

* **Đánh giá AI Fit:** **LLM Feature** (Quy trình nghiệp vụ có luồng cố định, cần xử lý ngôn ngữ tự nhiên để draft tin nhắn và bám sát ranh giới an toàn; không dùng Agent tự trị để tránh rủi ro xe cạn pin giữa đường).
* **Quy trình tương lai (Future-State):**
  * **Bước 1 (Nhận tin):** Hệ thống tiếp nhận tọa độ GPS và mức pin từ xe tài xế.
  * **Bước 2 (AI Step 🔵):** Mô hình đối chiếu dữ liệu: Nếu pin < 5%, từ chối trạm xa và tạo cấu trúc JSON điều xe cứu hộ; nếu pin an toàn, soạn nháp hướng dẫn đường đi. Toàn bộ bắt buộc bắt đầu bằng thẻ `[DRAFT_ONLY]`.
  * **Bước 3 (Human-in-the-loop 🟢):** Điều phối viên nhấn xác nhận duyệt tin nhắn hoặc xác nhận lệnh điều xe cứu hộ pin.
  * **Bước 4 (Fallback ↩️):** Nếu API gặp lỗi mạng hoặc AI không thể xác định trạm, hệ thống chuyển giao cho điều phối viên xử lý thủ công theo quy trình truyền thống.

---

## 3. Đánh giá độ sẵn sàng & Quyết định (Evaluate)

* **Checklist kiểm tra:**
  * [x] Dữ liệu mẫu/logs xe và vị trí trạm sạc có sẵn qua API của GSM & VinFast.
  * [x] Rủi ro vận hành được kiểm soát chặt chẽ bằng cơ chế Human-in-the-loop và tiền tố `[DRAFT_ONLY]`.
  * [x] Đội ngũ điều phối viên sẵn sàng tiếp nhận công cụ co-pilot để giảm tải giờ cao điểm.
* **Quyết định cuối cùng:** **GO (Bắt đầu xây dựng Prototype)**
* **Lý giải (Justification):** Bài toán có phạm vi rõ ràng, rủi ro được cô lập qua ranh giới an toàn đã được kiểm chứng bằng thực nghiệm mã nguồn, mang lại giá trị định lượng trực tiếp về thời gian xử lý và tỷ lệ khả dụng của đội xe.