## Team

- **Team:** Fournity
- **Members:**
  - 2A202602853 — Hoàng Văn Nam
  - 2A202602489 — Nguyễn Hải Hoàng
  - 2A202602977 — Dương Hà Đức Anh
  - 2A202603018 — Tạ Đăng Dương

---

**Bài toán được chọn (từ `01-problem-scan.md`):** Card #1 — **Vinmec: Chuẩn hóa chẩn đoán sang mã ICD-10**

> **Lưu ý về dữ liệu:** Các số liệu về khối lượng hồ sơ, thời gian xử lý và tỷ lệ sai mã trong báo cáo là giả định phục vụ scoping. Nhóm cần xác minh bằng log đã ẩn danh và khảo sát thực tế trước khi kết luận ROI. “Có dữ liệu” trong phạm vi prototype nghĩa là có danh mục ICD-10 chính thức và bộ dữ liệu mẫu được phê duyệt quyền sử dụng; không mặc định rằng dữ liệu bệnh nhân thật đã sẵn sàng.

---

# 🏗️ Phase 3 — DEEP-DIVE

## 3.1. Current-State Workflow

### Mô tả quy trình

```text
┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│ Bước 1       │     │ Bước 2       │     │ Bước 3       │     │ Bước 4       │
│ Bác sĩ ghi   │     │ Coder đọc    │     │ Coder tra    │     │ Nhập mã vào  │
│ chẩn đoán    │ ──→ │ free-text    │ ──→ │ cứu danh mục │ ──→ │ hệ thống BHYT│
│ (free-text)  │     │ chẩn đoán    │     │ ICD-10 🔴    │     │              │
│ Ai: Bác sĩ   │     │ Ai: Coder    │     │ Ai: Coder    │     │ Ai: Coder    │
│ ⏱ 1 phút     │     │ ⏱ 1 phút     │     │ ⏱ 5 phút 🔴  │     │ ⏱ 1 phút     │
│ In: Kết quả  │     │ In: Hồ sơ    │     │ In: Tên bệnh │     │ In: Mã ICD-10│
│ Out: Hồ sơ   │     │ Out: Khái niệm│    │ Out: Mã ứng  │     │ Out: Claim   │
│ chẩn đoán    │     │ bệnh cần mã  │     │ viên         │     │ BHYT         │
└──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘
                                                                      │
                                                                      ▼
                                                               ┌──────────────┐
                                                               │ Bước 5 🔄    │
                                                               │ Handoff:     │
                                                               │ Coder → BHYT │
                                                               │ Nếu bị từ   │
                                                               │ chối: tra và│
                                                               │ sửa lại 🔴  │
                                                               │ ⏱ 10–15 phút│
                                                               └──────────────┘

🔴 = Bottleneck   🔄 = Handoff (Medical coder ↔ Hệ thống BHYT)
⏱ Tổng thời gian thủ công giả định: khoảng 8 phút/hồ sơ,
  chưa tính 10–15 phút làm lại nếu claim bị từ chối vì mã không phù hợp.
```

Sơ đồ trực quan được nộp tại `04-workflow-diagram.png`.

### Bảng phân tích từng bước

| Bước | Actor / hệ thống | Input | Hoạt động | Output | Thời gian giả định |
|---:|---|---|---|---|---:|
| 1 | Bác sĩ | Kết quả khám và bằng chứng lâm sàng | Ghi chẩn đoán dạng free-text vào hồ sơ | Chẩn đoán đã ghi nhận | 1 phút |
| 2 | Medical coder | Hồ sơ chẩn đoán | Đọc, chuẩn hóa viết tắt/chính tả và xác định khái niệm cần mã hóa | Tên bệnh/khái niệm chuẩn hóa sơ bộ | 1 phút |
| 3 | Medical coder + danh mục ICD-10 | Tên bệnh, thông tin ngữ cảnh | Tra cứu danh mục và đối chiếu các mã gần giống | Mã ICD-10 ứng viên | 5 phút 🔴 |
| 4 | Medical coder + hệ thống nghiệp vụ | Mã ứng viên | Chọn mã cuối, nhập và kiểm tra claim | Claim sẵn sàng gửi | 1 phút |
| 5 | Hệ thống BHYT + medical coder | Claim | Tiếp nhận; nếu từ chối thì trả lý do để coder tra và sửa | Claim chấp nhận hoặc yêu cầu sửa | 10–15 phút nếu làm lại 🔴 |

### Handoff và bottleneck

- **Handoff 1:** Bác sĩ → medical coder qua hồ sơ bệnh án.
- **Handoff 2:** Medical coder → hệ thống BHYT qua claim.
- **Handoff ngược:** BHYT → medical coder khi claim bị từ chối.
- **Bottleneck chính:** Bước 3 vì free-text có viết tắt, lỗi chính tả, từ đồng nghĩa và các mã ICD-10 gần nhau.
- **Bottleneck thứ cấp:** Rework tại bước 5 khi mã không đủ đặc hiệu hoặc không phù hợp ngữ cảnh được ghi nhận.

### Điểm lỗi tiềm năng

1. Chẩn đoán quá chung chung hoặc thiếu bằng chứng để chọn mã đủ đặc hiệu.
2. Viết tắt có nhiều nghĩa và khác nhau giữa chuyên khoa.
3. Nhầm mã bệnh gần giống, mã biến chứng hoặc mã nguyên nhân bên ngoài.
4. Dùng sai phiên bản/danh mục mã đang có hiệu lực.
5. AI hoặc coder suy diễn thêm chẩn đoán không có trong hồ sơ.
6. Dữ liệu nhạy cảm bị đưa ra ngoài phạm vi hệ thống được phê duyệt.

---

## 3.2. Problem Statement (6-field)

| Field | Nội dung |
|---|---|
| **1. Actor / Operator** | Nhân viên hành chính y tế/medical coder tại cơ sở Vinmec là operator chính. Bác sĩ là người tạo chẩn đoán nguồn; bộ phận thanh toán và hệ thống BHYT là downstream stakeholders. |
| **2. Current Workflow** | Bác sĩ ghi chẩn đoán tự do → coder đọc và chuẩn hóa nội dung → tra cứu thủ công danh mục ICD-10 đang áp dụng → chọn và nhập mã vào hệ thống nghiệp vụ → claim được chuyển sang BHYT; nếu bị từ chối vì mã, coder phải đọc lý do và làm lại. Quy trình cơ sở giả định mất khoảng 8 phút/hồ sơ. |
| **3. Bottleneck** | Bước tra cứu thủ công mất khoảng 5 phút: coder phải đối chiếu free-text có nhiều biến thể, viết tắt và lỗi chính tả với danh mục ICD-10 lớn; các mã gần giống đòi hỏi kiểm tra ngữ cảnh và độ đặc hiệu. |
| **4. Business Impact** | Với giả định khoảng 200 hồ sơ/ngày/cơ sở và tỷ lệ lỗi coding 10–15%, rework 10–15 phút/hồ sơ có thể gây chậm xử lý claim và tăng tải hành chính. Đây là giả thuyết cần xác minh bằng rejection reason, coding time và claim logs đã ẩn danh; không coi là số liệu chính thức của Vinmec/BHYT. |
| **5. Success Metric** | (a) Giảm median search time từ baseline 5 phút xuống dưới 30 giây/hồ sơ; (b) top-3 recall ≥ 95% và top-1 accuracy ≥ 90% trên golden set có chuyên gia xác nhận; (c) 100% mã cuối do coder duyệt; (d) 100% trường hợp confidence < 70%, thiếu ngữ cảnh hoặc có mâu thuẫn được gắn cờ review; (e) không tăng tỷ lệ claim bị từ chối liên quan tới coding so với baseline. |
| **6. Operational Boundary** | AI được trích xuất cụm chẩn đoán từ free-text và đề xuất tối đa top-3 mã từ danh mục ICD-10 đã phê duyệt, kèm confidence và bằng chứng văn bản. **CẤM:** tự chẩn đoán, thêm bệnh không có trong hồ sơ, tự ghi mã cuối, tự gửi claim BHYT hoặc thay đổi hồ sơ nguồn. Medical coder bắt buộc duyệt; confidence < 70%, dữ liệu thiếu/mâu thuẫn hoặc không có mã phù hợp phải chuyển tra cứu thủ công. |

### Phạm vi prototype

**Trong scope:**

- Văn bản chẩn đoán tiếng Việt đã loại bỏ/giả lập định danh.
- Một phiên bản danh mục ICD-10 được kiểm soát và ghi rõ ngày hiệu lực.
- Trích xuất cụm chẩn đoán, tìm kiếm semantic/fuzzy và xếp hạng top-3.
- Hiển thị mã, tên mã chính thức, confidence, đoạn bằng chứng và cảnh báo.
- Coder chấp nhận, sửa hoặc từ chối đề xuất; mọi quyết định được audit.

**Ngoài scope:**

- Chẩn đoán bệnh, tư vấn điều trị hoặc thay thế quyết định của bác sĩ.
- Tự động ghi mã vào hồ sơ hoặc tự gửi claim.
- Học trực tiếp từ dữ liệu mới mà chưa kiểm duyệt.
- Dùng thông tin bệnh nhân ngoài mục đích coding được phê duyệt.
- Mã hóa toàn bộ chuyên khoa ngay từ vòng prototype đầu tiên.

---

## 3.3. Future-State Flow & AI Fit

### AI Fit

Chọn **LLM Feature kết hợp retrieval/ranking xác định**, không dùng LLM sinh mã tự do.

| Phương án | Điểm mạnh | Hạn chế | Vai trò trong giải pháp |
|---|---|---|---|
| **Rule-based thuần** | Dễ audit, tốt với từ điển và mapping chính xác | Khó bao phủ viết tắt, lỗi chính tả và cách diễn đạt đa dạng | Chuẩn hóa cơ bản, validation và enforcement boundary |
| **Fuzzy/Semantic Retrieval** | Chỉ tìm trong danh mục hợp lệ, hỗ trợ biến thể ngôn ngữ | Cần corpus và cách đánh giá ranking | Sinh danh sách mã ứng viên |
| **LLM Feature** | Trích xuất khái niệm, hiểu ngữ cảnh free-text và giải thích đề xuất | Có thể suy diễn/hallucinate | NER, chuẩn hóa ngôn ngữ và rerank có kiểm soát |
| **Agentic Loop** | Có thể tự gọi nhiều công cụ | Không cần thiết, tăng quyền và rủi ro trong workflow cố định | Không chọn |

Không chọn Rule-based thuần vì ngôn ngữ chẩn đoán có nhiều biến thể. Không để LLM tự tạo mã vì có nguy cơ hallucination. Danh sách ứng viên phải được truy hồi từ danh mục ICD-10 đã phê duyệt và mã cuối luôn do coder quyết định.

### Future-State Flow

```text
┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│ Bước 1       │     │ 🔵 AI trích  │     │ 🔵 Retrieval │     │ 🟢 Coder     │
│ Bác sĩ ghi   │ ──→ │ xuất & chuẩn │ ──→ │ + AI rerank  │ ──→ │ review top-3 │
│ chẩn đoán    │     │ hóa khái niệm│     │ mã + score   │     │ và chọn mã   │
└──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘
                                                                      │
                                      ┌───────────────────────────────┤
                                      │                               ▼
                                      │                        [Validate mã/version]
                                      │                               │
                                      │                               ▼
                                      │                        [Coder xác nhận ghi mã]
                                      │
                                      ▼
                               ↩️ FALLBACK
                     Confidence < 70%, thiếu/mâu thuẫn dữ liệu,
                     không tìm được mã hoặc coder từ chối:
                     quay lại tra cứu thủ công và/hoặc yêu cầu
                     bác sĩ làm rõ — không tự suy diễn chẩn đoán.
```

### Structured output đề xuất

```json
{
  "extracted_diagnosis": "viêm phổi cộng đồng",
  "candidates": [
    {
      "code": "<mã lấy từ danh mục được phê duyệt>",
      "official_label": "<tên mã chính thức>",
      "confidence": 0.86,
      "evidence": "<đoạn văn bản nguồn hỗ trợ đề xuất>"
    }
  ],
  "requires_human_review": true,
  "warning": "Đây là đề xuất coding, không phải chẩn đoán y khoa."
}
```

### Human-in-the-loop

Medical coder phải:

1. Kiểm tra cụm chẩn đoán được trích xuất có đúng hồ sơ nguồn không.
2. Kiểm tra mã và tên mã trong danh mục đúng phiên bản.
3. Xác định mức độ đặc hiệu phù hợp với bằng chứng đã ghi nhận.
4. Chấp nhận, sửa hoặc từ chối đề xuất.
5. Xác nhận trước khi mã được ghi vào hệ thống nghiệp vụ.

Nếu chẩn đoán nguồn chưa đủ rõ, coder không được dùng AI để suy diễn mà phải yêu cầu bác sĩ làm rõ theo quy trình nghiệp vụ.

### Fallback

| Tình huống | Hành vi fallback |
|---|---|
| Confidence cao nhất < 70% | Không preselect mã; gắn cờ và mở công cụ tra cứu thủ công. |
| Thiếu ngữ cảnh hoặc các đoạn hồ sơ mâu thuẫn | Chuyển coder review; nếu cần, yêu cầu bác sĩ làm rõ. |
| Không có ứng viên trong danh mục | Không để model tạo mã mới; tra cứu thủ công. |
| Danh mục sai version/hết hiệu lực | Chặn đề xuất và yêu cầu quản trị cập nhật nguồn mã. |
| Model/API timeout hoặc lỗi | Giữ workflow thủ công; không retry vô hạn. |
| Output không đúng schema/mã không nằm trong danh mục | Chặn output, ghi log kỹ thuật và chuyển review. |
| Coder từ chối đề xuất | Ghi lý do để đánh giá; không tự động học từ một phản hồi đơn lẻ. |

---

## 3.4. Data, Privacy, Risk & Measurement Plan

### Dữ liệu cần thiết

1. Danh mục ICD-10 chính thức/phiên bản đang được nghiệp vụ áp dụng.
2. Bộ ánh xạ chẩn đoán free-text → mã đã được coder xác nhận.
3. Các biến thể viết tắt/chính tả có kiểm duyệt.
4. Claim rejection reasons liên quan tới coding để thiết lập baseline.
5. Dữ liệu phải được tối thiểu hóa, ẩn danh/giả lập và phê duyệt quyền sử dụng.

### Rủi ro và kiểm soát

| Rủi ro | Mức độ | Kiểm soát |
|---|---|---|
| Đề xuất mã sai hoặc thiếu đặc hiệu | Cao | Top-3, evidence, confidence, danh mục đóng và coder duyệt |
| LLM tạo mã không tồn tại | Cao | Retrieval-only candidates + kiểm tra mã bằng code |
| AI suy diễn thành chẩn đoán mới | Cao | Prompt boundary, chỉ trích xuất từ source, evidence bắt buộc và HITL |
| Lộ dữ liệu sức khỏe cá nhân | Cao | Data minimization, de-identification, RBAC, encryption và audit log |
| Automation bias của coder | Trung bình | Không preselect dưới ngưỡng, hiển thị evidence, theo dõi override/error |
| Dataset lệch chuyên khoa | Trung bình | Đánh giá riêng theo chuyên khoa, giới hạn scope pilot |

### Kế hoạch đo lường

1. Chọn một chuyên khoa/phạm vi mã hẹp cho prototype.
2. Tạo golden set đã loại định danh, được ít nhất hai coder review và xử lý bất đồng.
3. Đo baseline: median/P90 search time, top coding errors và rejection rate.
4. Chạy offline: top-1 accuracy, top-3 recall, coverage, calibration theo confidence.
5. Chạy shadow mode: AI đề xuất nhưng coder vẫn làm theo quy trình hiện tại.
6. Chạy pilot có giám sát và so sánh time, acceptance/override, safety errors và claim rejection.

---

# 🏁 Phase 5 — EVALUATE

## AI Readiness Checklist

- [x] **Có nguồn dữ liệu để bắt đầu prototype?** — Có điều kiện: danh mục ICD-10 chính thức và dataset chẩn đoán tiếng Việt **chỉ được dùng sau khi xác minh license, phiên bản, chất lượng và phê duyệt privacy**. Dữ liệu công khai không tự động đồng nghĩa với dữ liệu phù hợp cho Vinmec.
- [x] **Rủi ro khi AI sai có thể kiểm soát?** — Có cho prototype scope hẹp: model chỉ đề xuất top-3; mã phải thuộc danh mục đóng; confidence dưới 70% fallback; coder luôn duyệt mã cuối. Việc kiểm soát phải được chứng minh qua golden set, không chỉ dựa trên thiết kế.
- [x] **Thay đổi workflow có khả thi?** — Giả thuyết khả thi vì coder chuyển từ “tự tra cứu toàn bộ” sang “review đề xuất”, nhưng vẫn cần phỏng vấn và usability test trước pilot để xác nhận stakeholder readiness.

## Quyết định cuối cùng: **GO — Prototype scope hẹp, có điều kiện**

Nhóm không phê duyệt triển khai production hoặc tự động gửi claim. Quyết định GO chỉ dành cho prototype offline/shadow mode trong một phạm vi chuyên khoa nhỏ.

### Justification

> Bài toán có đầu vào, operator và bottleneck rõ: chuẩn hóa free-text chẩn đoán thành danh sách mã ICD-10 ứng viên. Danh mục mã là tập đóng nên có thể ngăn LLM tạo mã tự do; semantic retrieval và LLM hỗ trợ xử lý biến thể ngôn ngữ tốt hơn rule thuần. Rủi ro được giới hạn bằng top-3 recommendation, confidence threshold 70%, evidence từ hồ sơ nguồn, validation theo danh mục và bắt buộc medical coder duyệt trước khi ghi mã. Các metric về thời gian, top-1 accuracy, top-3 recall và claim rejection cho phép đánh giá rõ ràng. Tuy nhiên, dữ liệu và số liệu business hiện vẫn cần xác minh, nên GO chỉ áp dụng cho prototype có kiểm soát, chưa áp dụng cho production.

### Exit criteria trước khi pilot

1. Xác minh license và phiên bản danh mục/dataset.
2. Privacy/security review thông qua.
3. Golden set đạt top-3 recall ≥ 95% và top-1 accuracy ≥ 90%.
4. 100% output chỉ chứa mã tồn tại trong danh mục được phê duyệt.
5. 100% hồ sơ có coder xác nhận mã cuối.
6. Không tăng coding-related claim rejection so với baseline trong shadow test.
7. Medical coder xác nhận giao diện và workflow có thể sử dụng.