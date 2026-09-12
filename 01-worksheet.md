# 01 - Problem Scan & Quick Cards (Vin Smart Future)

* **Học viên:** Cá nhân
* **Đơn vị:** Vin Smart Future (Vingroup)

---

## Phase 1 — SCAN: Danh sách bài toán cơ hội (4 Lenses)

| # | Subsidiary | Lens | Mô tả ngắn bài toán |
|---|---|---|---|
| 1 | **Xanh SM (GSM)** | Tốn thời gian | Điều phối viên xử lý thủ công các ca sự cố sạc pin hoặc cạn pin thực địa (tra cứu trụ sạc trống, tính cự ly, gọi cứu hộ, mất 15-20 phút/lượt). |
| 2 | **VinFast** | Lặp lại | Đối chiếu và so khớp dữ liệu log sạc hàng tuần từ các trạm sạc đối tác ngoài với báo cáo quyết toán tài chính. |
| 3 | **Vinhomes** | AI có thể tốt hơn | Tự động phân loại, trích xuất mức độ khẩn cấp và chuyển tiếp phản ánh cư dân trên Resident App về đúng ban quản lý phân khu. |
| 4 | **Vinmec** | Pain từ người khác | Bác sĩ mất 20–30 phút viết tóm tắt xuất viện (Discharge Summary) từ hồ sơ bệnh án điện tử, gây quá tải giờ làm việc. |
| 5 | **Vinpearl** | Tốn thời gian | Phân tích email đặt phòng theo đoàn (Group Booking) phức tạp để tự động kiểm tra quỹ phòng trống và draft báo giá. |

---

## Phase 2 — QUICK-ASSESS: 3 Quick Problem Cards

### QUICK PROBLEM CARD #1
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
│ Bước nào tốn nhất? Bước 3 & 4 ( 10-12 phút/lượt)            │
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 3 & 4            │
│ (Trích xuất GPS/Pin -> Tra cứu trạm -> Tự động draft tin)   │
│                                                             │
│ Đo thành công bằng gì (Metric có số)?                       │
│ Giảm thời gian xử lý từ 15 phút ──> dưới 3 phút/lượt.       │
│                                                             │
│ Quick Architecture: [x] LLM Feature                         │
└─────────────────────────────────────────────────────────────┘
```

```
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #2                                       │
│                                                             │
│ Bài toán: Phân loại khiếu nại/yêu cầu kỹ thuật của cư dân   │
│ gửi qua App Vinhomes Resident đến đúng ban quản lý tòa nhà. │
│ Công ty thành viên: [x] Vinhomes                            │
│                                                             │
│ Ai đang đau? Cư dân (chờ lâu), Nhân viên CSKH (quá tải vé)  │
│                                                             │
│ Workflow thủ công hiện tại:                                 │
│   1. Cư dân gửi tin nhắn phản ánh tự do                     │
│   → 2. Nhân viên CSKH đọc và phân loại thủ công             │
│   → 3. Gán thẻ nghiệp vụ và chuyển ticket sang ban kỹ thuật │
│   → 4. Soạn phản hồi xác nhận tiếp nhận cho cư dân          │
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

```
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #2                                       │
│                                                             │
│ Bài toán: Phân loại khiếu nại/yêu cầu kỹ thuật của cư dân   │
│ gửi qua App Vinhomes Resident đến đúng ban quản lý tòa nhà. │
│ Công ty thành viên: [x] Vinhomes                            │
│                                                             │
│ Ai đang đau? Cư dân (chờ lâu), Nhân viên CSKH (quá tải vé)  │
│                                                             │
│ Workflow thủ công hiện tại:                                 │
│   1. Cư dân gửi tin nhắn phản ánh tự do                     │
│   → 2. Nhân viên CSKH đọc và phân loại thủ công             │
│   → 3. Gán thẻ nghiệp vụ và chuyển ticket sang ban kỹ thuật │
│   → 4. Soạn phản hồi xác nhận tiếp nhận cho cư dân          │
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

> [!TIP]
> **🤖 AI Prompts — Stress-Test thẻ bài toán:**
> Hãy dán nội dung thẻ bài toán của bạn vào LLM để nhận phản biện:
> *"Đây là một thẻ bài toán vận hành tôi đề xuất cho Vin Smart Future: [Dán nội dung]. Hãy đóng vai trò là một CFO và Trưởng phòng Vận hành cực kỳ khắt khe, chỉ ra cho tôi 3 điểm yếu về logic, metric, và giải thích vì sao rule-based code thông thường có thể giải quyết bài toán này tốt hơn là dùng AI."*

---

# 🏗️ Phase 3 — DEEP-DIVE (Nhóm, 85 min)

## 3.1. Current-State Workflow Mapping (25 min)
**Vẽ quy trình hiện tại lên bảng/giấy A3.** Sử dụng các ký hiệu:
* 🔴 **Bottleneck:** Bước gây tắc nghẽn, tốn thời gian, hoặc sai sót nhiều nhất.
* 🔄 **Handoff:** Điểm chuyển giao thông tin giữa người và hệ thống, hoặc giữa các bộ phận.
* Ghi rõ thời gian vận hành trung bình: **Tổng cộng = ____ phút/lượt**.

## 3.2. Problem Statement (6-field) & Metrics (15 min)
Điền đầy đủ 6 trường thông tin của bài toán:

| Field | Nội dung chi tiết |
|---|---|
| **1. Actor / Operator** | Ai đang thực hiện tác vụ hằng ngày? |
| **2. Current Workflow** | Mô tả tóm tắt quy trình thủ công hiện tại và công cụ sử dụng. |
| **3. Bottleneck** | Bước nào chậm, lỗi, hoặc cần xử lý ngôn ngữ tự động nhiều nhất? |
| **4. Business Impact** | Tổn thất thực tế đo bằng thời gian, chi phí, hoặc SLA của Vingroup. |
| **5. Success Metric** | AI giải quyết được thì đạt ngưỡng số mấy? (Ví dụ: *"85% vé được phân loại dưới 10s"*). |
| **6. Operational Boundary** | AI được phép làm gì, TUYỆT ĐỐI không được làm gì, điểm nào cần duyệt? |

## 3.3. Future-State Flow & AI Fit (25 min)
* **Xác định mức AI Fit (AI-Fit Matrix):** Giải pháp thuộc nhóm nào? [ ] Rule / State-Machine [ ] LLM Feature [ ] Agentic Loop.
* **Vẽ Future-State Flow:** Đánh dấu rõ:
  * 🔵 **AI Step:** Tác vụ LLM xử lý.
  * 🟢 **Human Step (HITL):** Bước con người phê duyệt/review (Human-in-the-loop).
  * ↩️ **Fallback:** Kế hoạch dự phòng khi LLM trả về kết quả lỗi hoặc không tự tin.

---

# 💻 Phase 4 — TECHNICAL PROMPT PROTOTYPE (Nhóm, 30 min)

Để đảm bảo kỹ sư của Vin Smart Future luôn giữ vững năng lực lập trình, nhóm của bạn sẽ tiến hành **lập trình bản mẫu prompt** trực tiếp trên **Gemini 2.5 Flash** bằng Python để stress-test hệ thống.

### Hướng dẫn thực hiện:
1. Mở file [starter-code/prompt_prototype.py](starter-code/prompt_prototype.py) bằng VS Code/Cursor.
2. Hoàn thiện các nội dung sau:
   * **System Prompt:** Viết chỉ thị cực kỳ nghiêm ngặt quy định vai trò, nhiệm vụ, định dạng output và **Operational Boundary (Ranh giới cấm)** của mô hình.
   * **Structured Output:** Định nghĩa định dạng JSON output rõ ràng.
   * **Adversarial Test Cases:** Viết ít nhất 3 prompts "tấn công" (Adversarial inputs) cố tình dụ AI vượt ranh giới hoặc đưa ra câu trả lời không được phép để kiểm tra xem ranh giới của bạn có thực sự vững chắc.
3. Chạy file python:
   ```bash
   python3 prompt_prototype.py
   ```
4. Kiểm tra xem các ranh giới an toàn có bị LLM phá vỡ hay không và ghi lại kết quả vào worksheet.

---

# 🏁 Phase 5 — EVALUATE (Nhóm, 20 min)

### AI Readiness Checklist:
1. [ ] Chúng tôi có sẵn dữ liệu mẫu/logs sạch để test?
2. [ ] Rủi ro khi AI sai có nằm trong tầm kiểm soát (qua HITL hoặc Fallback)?
3. [ ] Stakeholders sẵn sàng thay đổi quy trình làm việc cũ?

### Quyết định cuối cùng của Ban Giám Đốc Vin Smart Future:
[ ] **GO (Bắt đầu xây dựng Prototype):** Bắt đầu phát triển với scope hẹp.
[ ] **NOT YET (Cần tích lũy thêm dữ liệu/xác lập baseline):** Trì hoãn để chuẩn bị thêm.
[ ] **NO-GO (Không khả thi / Rule-based tốt hơn):** Hủy bỏ dự án AI này.

**Justification (Lý giải quyết định dựa trên bằng chứng kỹ thuật và chi phí):**
> *Viết lý giải chi tiết tại đây*

---

# 📝 Phase 6 — REFLECTION (Cá nhân)
*Ghi nhận phản ánh của cá nhân bạn về việc phối hợp với AI trong buổi học hôm nay vào file `03-ai-log.md`.*
