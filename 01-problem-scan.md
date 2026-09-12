# 01 — Problem Scan & Quick Problem Cards

## Bối cảnh lựa chọn

Tôi đóng vai AI Product Engineer tại Vin Smart Future, tập trung vào vận hành đội xe điện Xanh SM. Các con số trong tài liệu này là **baseline giả định để thiết kế prototype**, chưa phải số liệu production; cần được đối chiếu với log điều vận trước khi ra quyết định đầu tư.

## Phase 1 — SCAN

| # | Subsidiary | Lens | Bài toán / bottleneck | Tín hiệu cần đo |
|---|---|---|---|---|
| 1 | **Xanh SM** | Tốn thời gian | Điều phối viên xử lý sự cố pin giữa đường bằng cách nghe điện thoại, tra GPS, tìm trạm và soạn hướng dẫn thủ công. | Thời gian xử lý, số lần gọi lại, thời gian xe nằm chờ |
| 2 | **Xanh SM** | Pain từ người khác | Tài xế báo điểm đón khó tiếp cận qua tin nhắn tự do; điều phối viên phải đọc và chuẩn hóa mô tả trước khi điều xe lại. | Tỷ lệ điều xe lại, số cuốc hủy, thời gian đọc tin |
| 3 | **VinFast** | Lặp lại | Nhân sự đối chiếu dữ liệu phiên sạc từ nhiều đối tác với hóa đơn theo tuần. | Số dòng cần đối chiếu, tỷ lệ lệch, giờ công xử lý |
| 4 | **Vinhomes** | AI-upgrade | Phản ánh cư dân qua app được phân loại và chuyển ban quản lý thủ công, khiến các ca khẩn cấp bị trộn với yêu cầu thông thường. | SLA phân loại, tỷ lệ route đúng, thời gian tới người xử lý |
| 5 | **Vinmec** | Tốn thời gian | Bác sĩ soạn bản nháp tóm tắt xuất viện từ bệnh án, xét nghiệm và ghi chú ở nhiều màn hình. | Thời gian soạn, tỷ lệ bổ sung/sửa, thời gian chờ ký |

## Phase 2 — QUICK-ASSESS

### Quick Problem Card #1 — Xanh SM: xử lý pin tới hạn

**Bài toán:** Khi tài xế báo pin thấp hoặc xe dừng giữa đường, điều phối viên mất nhiều thời gian xác minh vị trí, tìm phương án và viết hướng dẫn an toàn.

- **Công ty:** Xanh SM (GSM)
- **Actor chịu ảnh hưởng:** Điều phối viên, tài xế, khách đang chờ chuyến
- **Workflow hiện tại:**
  1. Tài xế gọi hoặc nhắn tin báo sự cố.
  2. Điều phối viên tra cứu biển số và GPS trên hệ thống.
  3. Đọc mức pin, tìm trạm phù hợp hoặc đội hỗ trợ.
  4. Soạn hướng dẫn và gọi lại cho tài xế.
  5. Ghi nhận kết quả và điều chỉnh cuốc nếu cần.
- **Bước chậm/lỗi nhất:** Tra cứu phương án và soạn hướng dẫn, baseline **10 phút/lượt**.
- **AI hỗ trợ:** Chuẩn hóa thông tin sự cố và tạo bản nháp điều phối sau khi rule kiểm tra pin, khoảng cách và loại xe.
- **Metric:** Giảm thời gian xử lý từ **15 xuống dưới 3 phút/lượt**; 100% ca pin dưới 5% không được gợi ý trạm trên 5 km.
- **Quick architecture:** Rule/state-machine + LLM feature + HITL; không dùng agent tự trị.

### Quick Problem Card #2 — Vinhomes: phân loại phản ánh cư dân

**Bài toán:** Phản ánh tự do như “mất nước”, “thang máy kêu” hoặc “ồn sau 22h” cần được phân loại và chuyển đúng ban quản lý.

- **Công ty:** Vinhomes
- **Actor chịu ảnh hưởng:** Nhân viên CSKH, ban quản lý tòa nhà, cư dân
- **Workflow hiện tại:**
  1. Cư dân gửi nội dung qua app.
  2. CSKH đọc và gắn nhóm thủ công.
  3. CSKH tìm tòa/khu và chuyển ticket.
  4. Ban quản lý xác nhận và cập nhật trạng thái.
- **Bước chậm/lỗi nhất:** Đọc, hiểu và chọn route, baseline **8 phút/ticket**.
- **AI hỗ trợ:** Trích xuất intent, địa điểm, mức khẩn cấp và draft route; con người duyệt các ca nhạy cảm.
- **Metric:** 90% ticket được phân loại dưới **30 giây**, route đúng tối thiểu **95%**.
- **Quick architecture:** Rule cho mức khẩn cấp và SLA; LLM cho phân loại ngôn ngữ; HITL cho tranh chấp/pháp lý.

### Quick Problem Card #3 — VinFast: đối chiếu hóa đơn sạc

**Bài toán:** Nhân sự tài chính đối chiếu dữ liệu phiên sạc từ nhiều đối tác với hóa đơn, thường phải tìm nguyên nhân lệch bằng tay.

- **Công ty:** VinFast
- **Actor chịu ảnh hưởng:** Nhân viên tài chính, quản trị đối tác, đội vận hành trạm
- **Workflow hiện tại:**
  1. Nhận file phiên sạc và hóa đơn từ đối tác.
  2. Chuẩn hóa mã trạm, thời gian và đơn giá.
  3. So khớp dòng dữ liệu.
  4. Tách các dòng lệch và gửi yêu cầu xác minh.
- **Bước chậm/lỗi nhất:** Giải thích nguyên nhân lệch và viết email truy vấn, baseline **20 phút cho mỗi nhóm lệch**.
- **AI hỗ trợ:** Tóm tắt nhóm lệch và draft yêu cầu xác minh; phép tính và quyết định thanh toán vẫn do rule/người duyệt.
- **Metric:** Giảm thời gian phân tích lệch từ **20 xuống 5 phút/nhóm**; không tự động phê duyệt thanh toán.
- **Quick architecture:** Rule/data pipeline + LLM explanation feature; không dùng agent.

## Lựa chọn để Deep-Dive

Chọn **Card #1 — Xử lý sự cố pin tới hạn của Xanh SM** vì đây là quy trình có tác động thời gian thực, metric đo được và ranh giới an toàn rõ. Card #2 cần thêm dữ liệu phân loại đa ngôn ngữ và có rủi ro SLA/pháp lý; Card #3 phù hợp tự động hóa dữ liệu có cấu trúc trước khi dùng LLM.
