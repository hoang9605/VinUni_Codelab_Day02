# 01 — Problem Scan & Quick Problem Cards

**Học viên:** Nam (branch `namhv521`)  
**Vai trò giả lập:** AI Product Engineer, Vin Smart Future  
**Phạm vi:** Tìm bài toán vận hành có thể thử nghiệm bằng AI trong các công ty thành viên Vingroup.

> **Lưu ý về dữ liệu:** Bài làm này là product-scoping prototype. Các con số về thời gian, khối lượng và tỷ lệ bên dưới là giả định làm baseline, không phải số liệu vận hành chính thức của Vingroup. Trước khi triển khai cần xác minh bằng log đã ẩn danh, đo time-on-task và phỏng vấn stakeholder.

---

# Phase 1 — SCAN

## Bảng quét cơ hội

| # | Công ty thành viên | Lens | Actor gặp vấn đề | Mô tả ngắn bài toán |
|---:|---|---|---|---|
| 1 | Xanh SM | Tốn thời gian | Tài xế, điều phối viên | Khi xe báo pin yếu giữa ca, điều phối viên phải mở nhiều màn hình để tra vị trí, loại xe, trạm sạc tương thích rồi soạn hướng dẫn hoặc gọi cứu hộ. |
| 2 | Vinhomes | Lặp lại | Nhân viên CSKH, ban quản lý tòa nhà | Phản ánh cư dân về điện, nước, an ninh và tiếng ồn được đọc, gắn nhãn rồi chuyển đơn vị xử lý bằng tay. |
| 3 | VinFast | Lặp lại | Nhân viên tài chính vận hành | Hóa đơn sạc từ đối tác cần được so khớp với phiên sạc, đơn giá và dữ liệu thanh toán; ngoại lệ phải được tìm thủ công. |
| 4 | Vinpearl | Pain từ stakeholder | Quản lý khách sạn, khách lưu trú | Review tiêu cực từ nhiều nền tảng không được phát hiện và ưu tiên đủ sớm, khiến vấn đề khẩn cấp có thể phản hồi chậm. |
| 5 | Vinmec | Tốn thời gian | Bác sĩ, điều dưỡng | Nhân viên y tế mất thời gian tổng hợp ghi chú, xét nghiệm và chỉ định thành bản nháp tóm tắt xuất viện dễ hiểu. |
| 6 | Xanh SM | AI-upgrade | Nhân viên phân tích vận hành | Ghi chú tự do và transcript cuộc gọi hủy chuyến khó tổng hợp nhất quán thành nhóm nguyên nhân để theo dõi xu hướng. |

## Sàng lọc ban đầu

Ba vấn đề được chọn để quick-assess là **#1, #2 và #4** vì đều có actor rõ, đầu vào dạng ngôn ngữ, quy trình hiện hữu có thể đo và có thể giới hạn quyền của AI. Bài toán #3 chủ yếu phù hợp với rule/reconciliation engine; bài toán #5 có rủi ro y khoa cao và đòi hỏi governance dữ liệu nghiêm ngặt; bài toán #6 hữu ích nhưng không tác động tức thời bằng #1.

---

# Phase 2 — QUICK-ASSESS

## Quick Problem Card 1 — Xanh SM hỗ trợ sự cố pin yếu

| Thuộc tính | Nội dung |
|---|---|
| **Bài toán một câu** | Rút ngắn thời gian điều phối phương án an toàn khi tài xế báo pin yếu mà không cho AI tự gửi tin hoặc tự điều cứu hộ. |
| **Công ty** | Xanh SM (GSM) |
| **Actor đang đau** | Điều phối viên bị quá tải tra cứu; tài xế phải chờ và có nguy cơ hết pin giữa đường. |
| **Workflow hiện tại** | (1) Tài xế báo sự cố → (2) điều phối viên tra vị trí, pin, loại xe → (3) tra trạm sạc còn chỗ và tương thích → (4) chọn phương án, soạn hướng dẫn → (5) gửi sau khi kiểm tra hoặc gọi cứu hộ. |
| **Bottleneck** | Bước 3–4 vì phải tổng hợp nhiều nguồn và viết hướng dẫn thủ công; baseline giả định 10/15 phút mỗi lượt. |
| **AI hỗ trợ** | Tổng hợp dữ liệu đã được rule kiểm tra và soạn tin nhắn nháp cho điều phối viên. |
| **Metric có số** | Median handling time ≤ 4 phút; ≥ 98% đề xuất đúng điều kiện pin/khoảng cách/cổng sạc; 100% nội dung được người duyệt trước khi gửi. |
| **Quick Architecture** | **Rule + LLM Feature + Human-in-the-loop**; không dùng agent tự trị. |

### Ranh giới sơ bộ

- AI chỉ tạo đề xuất và nội dung nháp có nhãn `[DRAFT_ONLY]`.
- Pin dưới 5% không được đề xuất trạm xa hơn 5 km; ưu tiên đề xuất `dispatch_mobile_charger`.
- Điều phối viên quyết định việc gửi tin và điều cứu hộ.
- Thiếu pin, vị trí, loại xe hoặc dữ liệu trạm thì chuyển xử lý thủ công.

---

## Quick Problem Card 2 — Vinhomes phân loại phản ánh cư dân

| Thuộc tính | Nội dung |
|---|---|
| **Bài toán một câu** | Phân loại và đề xuất tuyến xử lý cho phản ánh cư dân, đồng thời giữ nhân viên phụ trách các trường hợp nhạy cảm hoặc không chắc chắn. |
| **Công ty** | Vinhomes |
| **Actor đang đau** | Nhân viên CSKH phải đọc nội dung không cấu trúc; cư dân chờ phản ánh được chuyển đúng bộ phận. |
| **Workflow hiện tại** | (1) Cư dân gửi phản ánh → (2) CSKH đọc và kiểm tra tòa/căn → (3) chọn nhóm sự cố → (4) chuyển ban quản lý/kỹ thuật/an ninh → (5) đơn vị nhận xác nhận. |
| **Bottleneck** | Bước 2–4; baseline giả định 5 phút/ticket, dễ chuyển nhầm khi nội dung mơ hồ. |
| **AI hỗ trợ** | Trích xuất loại sự cố, mức khẩn cấp, tòa nhà và đề xuất queue; nhân viên duyệt trường hợp nhạy cảm. |
| **Metric có số** | ≥ 90% ticket được route đúng; thời gian phân loại trung vị ≤ 30 giây; 100% ticket an ninh/tranh chấp được người duyệt. |
| **Quick Architecture** | **Rule + LLM Feature**; rule cho từ khóa khẩn cấp và quyền truy cập, LLM cho phân loại ngôn ngữ. |

### Ranh giới sơ bộ

- Không tự kết luận trách nhiệm pháp lý, mức bồi thường hoặc phí dịch vụ.
- Không tiết lộ thông tin căn hộ cho người không có quyền.
- Ticket có nguy cơ an toàn, tranh chấp hoặc confidence thấp phải chuyển người xử lý.

---

## Quick Problem Card 3 — Vinpearl ưu tiên review tiêu cực

| Thuộc tính | Nội dung |
|---|---|
| **Bài toán một câu** | Phát hiện, tóm tắt và ưu tiên review có dấu hiệu sự cố khẩn cấp để quản lý phản hồi sớm hơn. |
| **Công ty** | Vinpearl |
| **Actor đang đau** | Quản lý cơ sở phải theo dõi nhiều kênh; khách có trải nghiệm xấu không được hỗ trợ kịp thời. |
| **Workflow hiện tại** | (1) Nhân viên mở từng nền tảng → (2) đọc review mới → (3) xác định cơ sở/chủ đề → (4) đánh giá mức ưu tiên → (5) báo quản lý và soạn phản hồi. |
| **Bottleneck** | Bước 1–4; việc giám sát rời rạc có thể làm review khẩn cấp bị chìm. |
| **AI hỗ trợ** | Tóm tắt, gắn nhãn chủ đề/cảm xúc và đề xuất ưu tiên để quản lý review. |
| **Metric có số** | Recall ≥ 95% với nhóm review khẩn cấp; precision ≥ 90%; cảnh báo được tạo trong ≤ 5 phút; 100% phản hồi công khai được người duyệt. |
| **Quick Architecture** | **LLM Feature + Rule**; không cho AI tự đăng phản hồi. |

### Ranh giới sơ bộ

- AI không được hứa hoàn tiền, nâng hạng phòng hoặc thừa nhận trách nhiệm thay doanh nghiệp.
- Review có đe dọa an toàn hoặc dữ liệu cá nhân phải escalation ngay.
- Quản lý cơ sở duyệt nội dung trước khi phản hồi công khai.

---

# Quyết định lựa chọn

Tôi chọn **Card 1 — Xanh SM hỗ trợ sự cố pin yếu** để thực hiện deep-dive.

## Lý do lựa chọn

1. Actor, workflow và bottleneck đủ cụ thể để đo trước/sau.
2. Bài toán có tác động vận hành tức thời nhưng vẫn có thể thử nghiệm bằng dữ liệu giả lập và shadow mode.
3. Quy tắc an toàn về mức pin, khoảng cách và cổng sạc có thể thực thi bằng code xác định; LLM chỉ đảm nhận phần ngôn ngữ.
4. Mọi hành động thực tế đều giữ Human-in-the-loop nên giảm nguy cơ AI tự quyết định sai.
5. Có thể xây prototype nhỏ, stress-test bằng prompt đối nghịch rồi mới xin dữ liệu thật.

## Vì sao chưa chọn hai card còn lại

- **Card 2:** Có tiềm năng cao nhưng cần taxonomy phản ánh, phân quyền dữ liệu cư dân và tập ticket đã gắn nhãn trước khi đánh giá chính xác.
- **Card 3:** Hữu ích cho trải nghiệm khách hàng nhưng tác động vận hành tức thời thấp hơn; dữ liệu từ nền tảng bên ngoài cũng phụ thuộc điều khoản truy cập API.