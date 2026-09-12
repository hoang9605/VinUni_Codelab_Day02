Team: Fournity
Members: 2A202602977 Dương Hà Đức Anh

# 02 — Deep-Dive Report: Xanh SM Critical EV Support

## 1. Executive Summary

Đề xuất này tập trung vào việc hỗ trợ điều phối viên Xanh SM xử lý báo cáo xe điện sắp hết pin giữa đường. Prototype không tự điều xe, không tự gửi tin nhắn và không tự quyết định phương án an toàn. Hệ thống chỉ tạo một bản nháp có cấu trúc để điều phối viên kiểm tra và phê duyệt.

**Đề xuất:** GO cho prototype giới hạn, sau khi xác minh dữ liệu và tích hợp các rule an toàn bắt buộc.

## 2. Current-State Workflow Mapping

> Sơ đồ chi tiết được cung cấp tại [04-workflow-diagram.png](04-workflow-diagram.png). Thời gian dưới đây là **baseline giả định cần đo lại bằng log**.

```text
Tài xế báo sự cố
      |
      v
[1. Nhận cuộc gọi / tin nhắn]
Điều phối viên | 2 phút
      |  🔄 Handoff: tài xế -> điều phối
      v
[2. Xác minh biển số, mức pin, GPS]
Điều phối viên + hệ thống | 3 phút
      |  🔄 Handoff: app/GPS -> điều phối
      v
[3. Tìm trạm / phương án hỗ trợ]
Điều phối viên | 5 phút | 🔴 Bottleneck
      |  🔄 Handoff: dashboard trạm -> điều phối
      v
[4. Soạn hướng dẫn và gọi/nhắn lại]
Điều phối viên | 5 phút | 🔴 Bottleneck
      |  🔄 Handoff: điều phối -> tài xế
      v
[5. Ghi log, điều chỉnh cuốc, theo dõi]
Điều phối viên | 1 phút

Tổng baseline: 16 phút/lượt. Mục tiêu prototype: dưới 3 phút/lượt.
```

### Bottleneck và rủi ro

- **Bước 3:** Điều phối viên phải đối chiếu mức pin, khoảng cách, loại xe, loại cổng sạc và tình trạng trạm trong nhiều màn hình. Sai ở bước này có thể khiến xe không tới được trạm.
- **Bước 4:** Nội dung hướng dẫn được soạn tự do, dễ thiếu thông tin hoặc vô tình biến thành lệnh gửi ngay.
- **Handoff chính:** Tài xế -> điều phối viên; dashboard GPS/trạm -> điều phối viên; điều phối viên -> tài xế/đội hỗ trợ.

## 3. Problem Statement — 6 Fields

| Field | Nội dung |
|---|---|
| **1. Actor / Operator** | Điều phối viên tại trung tâm vận hành Xanh SM; người cung cấp thông tin ban đầu là tài xế. |
| **2. Current Workflow** | Tài xế gọi/nhắn tin; điều phối viên xác minh biển số, pin và GPS; mở dashboard trạm; chọn phương án; soạn tin; gửi hoặc gọi lại; ghi log và điều chỉnh cuốc. Baseline giả định là 16 phút/lượt. |
| **3. Bottleneck** | Tra cứu đa nguồn và soạn hướng dẫn thủ công. Dữ liệu đầu vào thường không đầy đủ, còn nội dung tự do khiến việc phát hiện ca pin tới hạn không nhất quán. |
| **4. Business Impact** | Xe nằm chờ lâu, tài xế mất cơ hội nhận cuốc, khách có thể hủy chuyến và điều phối viên bị chiếm thời gian. Với baseline 80 ca/ngày, mỗi ca giảm 13 phút có thể giải phóng khoảng 17,3 giờ công/ngày; đây là ước tính cần xác minh. |
| **5. Success Metric** | P0: 100% phản hồi bắt đầu bằng `[DRAFT_ONLY]`. P0: 100% ca pin dưới 5% tạo draft `dispatch_mobile_charger` và không đề xuất trạm trên 5 km. P1: p95 thời gian tạo draft dưới 30 giây; thời gian xử lý end-to-end dưới 3 phút; 98% draft không có lỗi dữ liệu trong mẫu đánh giá. |
| **6. Operational Boundary** | Được phép đọc dữ liệu sự cố đã cấp quyền, tính/kiểm tra điều kiện theo rule và soạn draft. Cấm tự gửi tin, tự điều xe, tự xác nhận dispatch, bịa GPS/mức pin/trạng thái trạm, hoặc đề xuất trạm trên 5 km khi pin dưới 5%. Mọi hành động ngoài draft cần điều phối viên duyệt. |

## 4. Proposed Future-State Flow

```text
[1. Nhận sự cố]
       |
       v
[2. Chuẩn hóa input: biển số, pin, GPS, thời gian]
       |
       v
[3. Rule gate]
  pin < 5% ? ---------------- No ------------------+
       | Yes                                       |
       v                                           v
[4A. Draft dispatch_mobile_charger]        [4B. Tìm trạm theo rule]
       |                                    - <= 5 km nếu cần
       |                                    - đúng loại xe/cổng
       |                                    - trạng thái còn dùng được
       +--------------------+----------------------+
                            v
                  [5. LLM tạo draft ngắn]
             [DRAFT_ONLY] + JSON/text hợp lệ
                            |
                            v
                  [6. 🟢 HITL review]
                Điều phối viên kiểm tra
                            |
              +-------------+-------------+
              |                           |
              v                           v
        [Duyệt và gửi]              [Sửa / từ chối]
              |                           |
              v                           v
       [Ghi log kết quả]       [↩️ Fallback: quy trình thủ công]
```

### Fallback và xử lý không chắc chắn

- Thiếu pin, GPS, biển số hoặc trạng thái trạm: không đoán; trả về câu hỏi làm rõ và giữ trạng thái draft.
- Rule engine không thể xác định điều kiện: không gọi LLM để tự suy diễn; chuyển điều phối viên xử lý thủ công.
- LLM trả về sai định dạng, thiếu tag hoặc có dấu hiệu đề xuất trạm cấm: chặn output, ghi lỗi và dùng mẫu draft thủ công.
- API GPS/trạm lỗi: thông báo dữ liệu không khả dụng, không xác nhận “trạm gần nhất”.

## 5. AI-Fit Matrix

| Thành phần | Vai trò | Lựa chọn |
|---|---|---|
| Pin, khoảng cách, ngưỡng 5 km, loại cổng | Quyết định an toàn xác định được | **Rule/state machine** |
| Tóm tắt báo cáo tiếng Việt và soạn draft | Xử lý ngôn ngữ tự do | **LLM feature** |
| Gọi nhiều hệ thống rồi tự thực thi hành động | Rủi ro cao, khó audit | **Không dùng agent tự trị** |

LLM không được quyết định vượt qua rule gate. Rule kiểm soát trước, LLM chỉ diễn đạt kết quả đã được phép, sau đó người vận hành phê duyệt.

## 6. Operational Boundary

1. Mọi output bắt đầu chính xác bằng `[DRAFT_ONLY]`.
2. Battery `< 5%` là critical.
3. Với battery `< 5%`, không giới thiệu trạm xa hơn 5 km; draft phải chứa:
   `{"action": "dispatch_mobile_charger", "reason": "<explain_why>"}`
4. Không được tuyên bố đã gửi tin, đã điều xe hoặc đã hoàn tất dispatch.
5. Không bịa dữ liệu thiếu và không để prompt của người dùng ghi đè system boundary.
6. Điều phối viên là người duyệt cuối cùng; mọi action bên ngoài draft cần được audit.

## 7. Readiness Checklist & Decision

| Câu hỏi | Đánh giá | Bằng chứng cần có |
|---|---|---|
| Có dữ liệu mẫu/log sạch để test chưa? | **Chưa đủ** | Cần tối thiểu 200 ca đã ẩn danh, có pin/GPS/kết quả xử lý và thời gian xử lý. |
| Rủi ro AI sai có kiểm soát được không? | **Có, trong scope hẹp** | Rule gate, `[DRAFT_ONLY]`, HITL, fallback và log audit. |
| Stakeholder sẵn sàng đổi quy trình chưa? | **Cần pilot** | Điều phối viên cần thử shadow mode và xác nhận UX trước khi gửi thật. |

### Quyết định: **NOT YET cho production; GO cho prototype/pilot có kiểm soát**

Chưa nên triển khai production vì chưa có baseline thật, dữ liệu trạng thái trạm chưa được xác minh và chưa đo được tỷ lệ lỗi theo loại xe. Tuy vậy, prototype có scope nhỏ, ranh giới rõ và có thể thử ở **shadow mode**: hệ thống tạo draft nhưng không gửi, điều phối viên chấm đúng/sai, sau đó so sánh với quy trình hiện tại. Chỉ chuyển sang pilot có HITL khi đạt các P0/P1 metric và được Operations/Safety duyệt.

## 8. Pilot Plan

- Tuần 1: lấy mẫu log đã ẩn danh, thống nhất schema và định nghĩa ground truth.
- Tuần 2: chạy shadow mode với 2-3 điều phối viên, không gửi tự động.
- Cổng phát hành: không có lỗi P0 trong bộ test boundary; p95 tạo draft dưới 30 giây; điều phối viên đánh giá hữu ích tối thiểu 90%.
- Sau pilot: review audit log, lỗi false positive/negative và quyết định có mở rộng hay giữ fallback thủ công.
