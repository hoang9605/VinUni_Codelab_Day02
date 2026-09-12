# 02 — Deep-Dive Report

## Xanh SM: Dispatcher Co-pilot cho sự cố pin yếu

**Phạm vi quyết định:** Prototype hỗ trợ điều phối, không phải hệ thống tự trị.  
**Chủ sở hữu nghiệp vụ giả định:** Trung tâm Điều vận Xanh SM.  
**Trạng thái số liệu:** Baseline và volume trong báo cáo là giả định cần xác minh bằng log vận hành đã ẩn danh.

---

# 1. Current-State Workflow

## 1.1. Mô tả quy trình hiện tại

Khi tài xế báo xe có mức pin thấp, điều phối viên phải ghép thông tin từ cuộc gọi, dashboard phương tiện, bản đồ và danh sách trạm sạc. Phần tra cứu/chọn phương án và soạn hướng dẫn là bottleneck chính.

| Bước | Actor / hệ thống | Input | Hoạt động | Output | Thời gian giả định |
|---:|---|---|---|---|---:|
| 1 | Tài xế | Mức pin, vị trí, mô tả | Gọi/gửi yêu cầu hỗ trợ | Ticket sự cố | 2 phút |
| 2 | Điều phối viên + dashboard xe | Biển số/tài khoản xe | Xác minh vị trí, mức pin, dòng xe và cổng sạc | Hồ sơ tình huống | 2 phút |
| 3 | Điều phối viên + dashboard trạm | Vị trí, cổng sạc | Tìm trạm còn chỗ, tương thích và trong khoảng cách an toàn | Danh sách ứng viên | 5 phút 🔴 |
| 4 | Điều phối viên | Hồ sơ + danh sách ứng viên | Chọn phương án, viết hướng dẫn hoặc xác định cần cứu hộ | Nội dung hướng dẫn nháp | 5 phút 🔴 |
| 5 | Điều phối viên | Phương án đã kiểm tra | Gửi tài xế hoặc liên hệ đội cứu hộ | Hướng dẫn/lệnh nghiệp vụ | 1 phút |

**Tổng thời gian baseline giả định:** 15 phút/lượt.  
**Bottleneck:** Bước 3 và 4, chiếm khoảng 10 phút/lượt.  
**Handoff:** Tài xế → tổng đài/điều phối viên → dashboard xe → dashboard trạm sạc → điều phối viên → tài xế hoặc đội cứu hộ.

## 1.2. Điểm lỗi tiềm năng

- Mức pin hoặc vị trí được nghe/nhập sai từ cuộc gọi.
- Trạm gần nhất không tương thích cổng sạc hoặc vừa hết chỗ.
- Điều phối viên chuyển đổi giữa nhiều màn hình nên bỏ sót điều kiện.
- Tin nhắn viết gấp thiếu địa chỉ, cảnh báo hoặc chỉ dẫn rõ ràng.
- Áp lực của người dùng có thể khiến quy trình an toàn bị bỏ qua.

Sơ đồ trực quan được nộp tại `04-workflow-diagram.png`.

---

# 2. Problem Statement — 6 Fields

| Field | Nội dung |
|---|---|
| **1. Actor / Operator** | Điều phối viên tại trung tâm vận hành Xanh SM là operator chính. Tài xế là người báo sự cố và nhận hướng dẫn; đội cứu hộ là downstream actor khi cần. |
| **2. Current Workflow** | Điều phối viên tiếp nhận yêu cầu, xác minh xe/vị trí/pin, mở dashboard trạm để lọc trạm tương thích, chọn phương án, soạn tin và gửi hoặc liên hệ cứu hộ. Quy trình khoảng 5 bước, baseline giả định 15 phút/lượt và phụ thuộc nhiều thao tác thủ công. |
| **3. Bottleneck** | Bước tra trạm và soạn phương án mất khoảng 10 phút/lượt vì dữ liệu nằm ở nhiều nguồn, cần kiểm tra khoảng cách/cổng sạc và chuyển dữ liệu thô thành hướng dẫn dễ hiểu. |
| **4. Business Impact** | Nếu giả định có 40 lượt cần hỗ trợ/ngày, 15 phút/lượt tương đương 10 giờ xử lý/ngày. Chờ lâu làm tăng thời gian xe không phục vụ khách và rủi ro cạn pin. Volume và tác động doanh thu phải được đo lại bằng log thật trước business case. |
| **5. Success Metric** | (a) Giảm median handling time từ baseline 15 phút xuống ≤ 4 phút; (b) ≥ 98% đề xuất vượt qua rule về mức pin, khoảng cách và cổng sạc; (c) 100% nội dung gửi tài xế và lệnh cứu hộ có người duyệt; (d) 100% trường hợp pin < 5% không được đề xuất trạm > 5 km; (e) tỷ lệ fallback/escalation được ghi log để phân tích. |
| **6. Operational Boundary** | AI được đọc dữ liệu cần thiết đã phân quyền, tóm tắt và tạo đề xuất nháp. AI không được tự gửi tin, tự điều cứu hộ, sửa dữ liệu nguồn, bỏ qua rule hoặc khẳng định hành động đã xảy ra. Điều phối viên duyệt mọi hành động. Thiếu hoặc mâu thuẫn dữ liệu phải fallback. |

## 2.1. Scope và ngoài scope

### Trong scope prototype

- Input giả lập: mức pin, vị trí, dòng xe, khoảng cách và trạng thái trạm.
- Rule kiểm tra ngưỡng an toàn trước/sau lời gọi LLM.
- LLM tạo JSON và nội dung `[DRAFT_ONLY]`.
- Điều phối viên xem, sửa, duyệt hoặc từ chối.
- Log input đã ẩn danh, rule result, output và quyết định của người duyệt.

### Ngoài scope

- Tự động điều xe cứu hộ hoặc tự gửi nội dung cho tài xế.
- Điều khiển phương tiện, chẩn đoán lỗi pin hoặc dự đoán quãng đường chính xác.
- Thay đổi trạng thái trạm sạc và booking chỗ thật.
- Dùng dữ liệu vị trí/cá nhân ngoài mục đích xử lý ticket.

---

# 3. AI Fit

## 3.1. So sánh phương án

| Phương án | Phù hợp ở đâu | Hạn chế | Quyết định |
|---|---|---|---|
| **No AI / thủ công** | Dễ kiểm soát, là fallback bắt buộc | Chậm và phụ thuộc thao tác nhiều màn hình | Giữ làm fallback |
| **Rule / state machine** | Pin `< 5%`, khoảng cách `> 5 km`, cổng sạc, trường bắt buộc và quyền hành động | Không giỏi soạn/tóm tắt ngôn ngữ tự nhiên | Bắt buộc cho safety-critical logic |
| **LLM Feature** | Tóm tắt tình huống, giải thích lựa chọn và soạn tin nhắn rõ ràng | Có thể hallucinate hoặc không tuân thủ format | Chọn, nhưng đặt sau rule và trước HITL |
| **Agentic Loop** | Có thể tự gọi nhiều tool và thực hiện workflow | Quyền tự trị làm tăng rủi ro vận hành, khó audit | Không chọn trong scope này |

**Kết luận AI Fit:** Chọn **Rule + LLM Feature + Human-in-the-loop**. Không giao quy tắc định lượng cho prompt; Python kiểm tra điều kiện an toàn một cách xác định. LLM là co-pilot, không phải decision maker.

---

# 4. Future-State Flow

```text
Tài xế gửi yêu cầu
        │
        ▼
[Validate trường bắt buộc & quyền truy cập]
        │ thiếu/sai ────────────────► ↩️ Fallback: điều phối nhập/xác minh lại
        ▼
[RULE: pin, khoảng cách, cổng sạc]
        │
        ├── pin < 5% và trạm > 5 km ─► action=dispatch_mobile_charger
        │
        └── đủ điều kiện ────────────► action=recommend_station
                                             │
                                             ▼
                              🔵 LLM tạo JSON + [DRAFT_ONLY]
                                             │
                                             ▼
                                 [Post-validation bằng code]
                                    │ lỗi format/boundary
                                    └────────► ↩️ Fallback / escalation
                                             │ hợp lệ
                                             ▼
                                  🟢 Điều phối viên review
                                    │ từ chối/sửa  │ duyệt
                                    ▼             ▼
                              Xử lý thủ công   Hệ thống nghiệp vụ gửi/
                                              tạo yêu cầu cứu hộ
```

## 4.1. Human-in-the-loop

Điều phối viên phải:

1. Xác nhận dữ liệu xe và mức pin.
2. Kiểm tra trạm/cứu hộ được đề xuất.
3. Sửa hoặc từ chối nội dung nếu cần.
4. Chủ động bấm duyệt trước khi hệ thống nghiệp vụ thực hiện hành động.

Giao diện không được biến `[DRAFT_ONLY]` thành hành động gửi tự động. Backend phải kiểm tra quyền và trạng thái approval thay vì tin vào văn bản do LLM tạo.

## 4.2. Fallback

| Tình huống | Hành vi fallback |
|---|---|
| Thiếu pin, vị trí, dòng xe hoặc loại cổng | Không gọi LLM; yêu cầu xác minh hoặc xử lý thủ công. |
| API xe/trạm lỗi hoặc dữ liệu quá cũ | Hiển thị dữ liệu không khả dụng; quay về dashboard và quy trình cũ. |
| Không có trạm tương thích/an toàn | Đề xuất escalation; điều phối viên quyết định cứu hộ. |
| Gemini timeout/lỗi | Không retry vô hạn; tối đa một lần rồi chuyển thủ công. |
| Output sai JSON hoặc vi phạm boundary | Chặn output, ghi log lỗi và chuyển điều phối viên. |
| Người dùng cố prompt-inject | System instruction + deterministic guardrail vẫn được ưu tiên; không thực hiện hành động. |

---

# 5. Risk, Data và Measurement Plan

## 5.1. Rủi ro chính và kiểm soát

| Rủi ro | Mức độ | Kiểm soát |
|---|---|---|
| Đề xuất trạm không an toàn | Cao | Rule trước/sau LLM, dữ liệu trạm thời gian thực, HITL |
| AI tuyên bố đã gửi/đã điều xe | Cao | Không cấp tool thực thi; schema chỉ là proposal; approval ở backend |
| Dữ liệu vị trí/cá nhân bị lộ | Cao | Data minimization, RBAC, mã hóa, retention ngắn, log đã che định danh |
| Hallucination địa chỉ/trạng thái trạm | Cao | Chỉ cho phép chọn ID từ API; post-validation; không cho model tự tạo trạm |
| Automation bias | Trung bình | Hiển thị lý do/rule, cho phép từ chối, audit tỷ lệ override |

## 5.2. Kế hoạch xác minh metric

1. Lấy mẫu ticket đã ẩn danh trong một khoảng thời gian được phê duyệt.
2. Đo median/P90 handling time và tỷ lệ escalation của quy trình hiện tại.
3. Xây golden set có phương án được chuyên gia vận hành xác nhận.
4. Chạy offline evaluation cho rule và output schema.
5. Chạy **shadow mode**: co-pilot tạo đề xuất nhưng không tác động workflow thật.
6. So sánh thời gian, lỗi safety, acceptance/override và phản hồi điều phối viên.

---

# 6. AI Readiness Evaluation

| Checklist | Trạng thái | Bằng chứng / khoảng trống |
|---|---|---|
| Có dữ liệu mẫu/log sạch để test? | ❌ Chưa | Repository chỉ có tình huống giả lập; chưa có log vận hành đã ẩn danh và golden labels. |
| Rủi ro khi AI sai kiểm soát được qua HITL/fallback? | ✅ Có điều kiện | Thiết kế có rule, post-validation, không cấp quyền hành động, HITL và fallback; vẫn cần security review. |
| Stakeholder sẵn sàng đổi quy trình? | ❌ Chưa xác minh | Chưa có phỏng vấn/usability test với điều phối viên và chủ hệ thống. |

## Quyết định: **NOT YET**

Không GO production ở thời điểm hiện tại. Bài toán đủ rõ để tiếp tục một prototype offline, nhưng chưa có bằng chứng dữ liệu và stakeholder readiness. Các con số đang là giả định nên chưa thể chứng minh ROI hoặc safety performance.

### Điều kiện để chuyển sang GO cho pilot scope hẹp

1. Có dataset đã ẩn danh và được phê duyệt sử dụng.
2. Xác lập baseline median/P90 handling time và error taxonomy.
3. Rule engine đạt 100% trên bộ boundary tests bắt buộc.
4. Golden-set evaluation đạt ≥ 98% safety-rule compliance.
5. Security/privacy review thông qua.
6. Điều phối viên thử nghiệm shadow mode và chấp thuận workflow.

Ngay cả khi đạt các điều kiện trên, quyết định GO chỉ dành cho **pilot có giám sát**, không phải tự động hóa hoàn toàn.