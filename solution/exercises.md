# K3 — Ngày 1: Bài Tập & Phản Ánh
## Khám Phá LLM API | Phiếu Thực Hành

**Thời lượng:** 9h00–13h00
**Cách làm:** Trả lời từng câu ngay sau khi hoàn thành block tương ứng —
đừng để dồn hết về cuối buổi. Thay dòng placeholder bằng câu trả lời thật
trả lời thật (chấm tự động sẽ đếm số câu đã trả lời).

---

## Block 1 — API Cơ Bản (trả lời sau Checkpoint 1)

### Câu 1.1 — Độ nhạy của temperature
Gọi `call_openai` với temperature 0.0, 0.5, 1.0 và 1.5 dùng prompt
**"Hãy kể cho tôi một sự thật thú vị về Việt Nam."**

**Bạn nhận thấy quy luật gì qua bốn phản hồi?** (2–3 câu)
> Nhiệt độ càng thấp (0.0), phản hồi càng ổn định và nhất quán — model lặp lại cùng một sự thật (Sơn Đoòng) với cùng cấu trúc câu. Nhiệt độ càng cao (1.0–1.5), phản hồi bắt đầu có biến thiên về độ dài và từ ngữ, nhưng nội dung vẫn xoay quanh các sự thật phổ biến về Việt Nam. Temperature kiểm soát mức độ "sáng tạo" của model: thấp = deterministic, cao = ngẫu nhiên hơn.

### Câu 1.2 — Chọn temperature cho sản phẩm
**Bạn sẽ đặt temperature bao nhiêu cho chatbot hỗ trợ khách hàng, và tại sao?**
> Tôi sẽ đặt temperature = 0.2–0.3. Chatbot hỗ trợ khách hàng cần câu trả lời chính xác, nhất quán và đáng tin cậy — không muốn model "sáng tạo" ra thông tin sai lệch. Temperature thấp giúp giảm thiểu rủi ro hallucination và đảm bảo khách hàng nhận được câu trả lời giống nhau cho cùng một câu hỏi.

### Câu 1.3 — Đánh đổi chi phí
Kịch bản: 10.000 người dùng hoạt động mỗi ngày, mỗi người gọi API 3 lần,
mỗi lần trung bình ~350 token đầu ra.

**Ước tính GPT-4o đắt hơn GPT-4o-mini bao nhiêu lần cho workload này? Nêu một
trường hợp GPT-4o xứng đáng với chi phí và một trường hợp nên dùng mini:**
> GPT-4o đắt hơn khoảng **16.7 lần** (giá output: $0.010/1K tokens vs $0.0006/1K tokens). Chi phí daily: 10K × 3 × 350 = 10.5M tokens → GPT-4o = $105/ngày, mini = $6.3/ngày.
> 
> **Nên dùng GPT-4o khi:** phân tích hợp đồng pháp lý hoặc chẩn đoán y tế — cần độ chính xác và suy luận cao, sai sót có hậu quả lớn.
> 
> **Nên dùng mini khi:** chatbot chăm sóc khách hàng cơ bản, phân loại email, tóm tắt nội dung — workload khối lượng lớn, độ chính xác cao không quá quan trọng và cần tiết kiệm chi phí.

---

## Block 2 — System Prompt & Token (trả lời sau Checkpoint 2)

### Câu 2.1 — Sức mạnh của persona
Gọi `chat_with_system_prompt` hai lần với cùng câu hỏi
**"Giải thích blockchain là gì?"** nhưng hai system prompt khác nhau:
- "Bạn là giáo viên tiểu học, giải thích thật đơn giản cho trẻ 8 tuổi."
- "Bạn là chuyên gia tài chính, trả lời chuyên sâu bằng thuật ngữ kỹ thuật."

**Hai phản hồi khác nhau như thế nào (độ dài, từ vựng, ví dụ)? System prompt
ảnh hưởng đến hành vi model ra sao?** (3–4 câu)
> Với system prompt "giáo viên tiểu học", phản hồi dùng từ ngữ đơn giản, giọng điệu gần gũi ("Chào con!"), giải thích blockchain qua phép analogies dễ hiểu. Với prompt "chuyên gia tài chính", phản hồi dùng thuật ngữ kỹ thuật như "Turing-complete state machine", "self-executing protocols" — ngắn gọn và chuyên sâu hơn. System prompt đóng vai trò như "lời đạo diễn": nó định hình persona, giọng điệu, độ chuyên sâu và cách tiếp cận của model, giúp kiểm soát chất lượng đầu ra mà không cần thay đổi câu hỏi.

### Câu 2.2 — tiktoken vs đếm từ
Chọn một đoạn văn tiếng Việt ~100 từ. So sánh số token theo `count_tokens`
(tiktoken) với ước lượng `số từ / 0.75` mà Part 1 đã dùng.

**Hai con số chênh nhau bao nhiêu phần trăm? Vì sao tiếng Việt thường tốn
nhiều token hơn tiếng Anh cùng độ dài?**
> Với đoạn ~100 từ tiếng Việt: ước lượng từ/0.75 = 132 token, tiktoken đếm thực tế = 126 token, chênh lệch ~4.8%. Tiếng Việt có nhiều dấu (thanh điệu: sắc, huyền, hỏi, ngã, nặng) và ký tự đặc biệt (ê, ô, ơ, â, đ) — mỗi ký tự mang dấu thường được mã hóa thành 1–2 token riêng thay vì gộp chung. Ngoài ra, tiếng Việt dùng nhiều từ ghép và từ mượn, làm tăng số token so với tiếng Anh cùng độ dài văn bản.

---

## Block 3 — Streaming & Độ Bền (trả lời sau Checkpoint 3)

### Câu 3.1 — Trải nghiệm người dùng với streaming
**Streaming quan trọng nhất trong trường hợp nào, và khi nào thì
non-streaming lại phù hợp hơn?** (1 đoạn văn)
> Streaming quan trọng nhất trong các ứng dụng tương tác trực tiếp với người dùng: chatbot, trợ lý ảo, công cụ viết nội dung — nơi người dùng cần thấy phản hồi ngay lập tức để có trải nghiệm mượt mà, đặc biệt với phản hồi dài. Non-streaming phù hợp hơn khi xử lý batch (hàng nghìn yêu cầu một lúc), khi cần kiểm tra toàn bộ nội dung trước khi hiển thị (filter an toàn), hoặc trong pipeline tự động mà không có người dùng đầu cuối — vì non-streaming đơn giản hơn, ít tốn tài nguyên và dễ implement caching/retry.

### Câu 3.2 — Vì sao backoff theo cấp số nhân?
**So với delay cố định (ví dụ luôn chờ 1 giây), exponential backoff có lợi
thế gì khi API bị quá tải? Điều gì xảy ra nếu hàng nghìn client cùng retry
với delay cố định giống nhau?**
> Exponential backoff cho thời gian nghỉ tăng dần (0.1s → 0.2s → 0.4s → ...), giúp server đang quá tải có thời gian phục hồi và giảm áp lực. Ngược lại, delay cố định 1s tạo hiệu ứng "đoàn xe" (thundering herd): hàng nghìn client cùng chờ 1s rồi đồng loạt gửi lại request — tạo đợt sóng thứ hai còn lớn hơn đợt đầu, dễ làm server sập hoàn toàn. Exponential backoff phân tán các request retry ra theo thời gian, tăng xác suất thành công và giảm tải tổng thể.

---

## Block 4 — Mini-Project (trả lời sau Checkpoint 4)

### Câu 4.1 — Thiết kế persona
**Bạn chọn persona gì cho trợ lý của mình? Viết lại system prompt đó và giải
thích 1–2 lựa chọn từ ngữ quan trọng trong prompt (ví dụ: vì sao yêu cầu
"trả lời ngắn gọn", vì sao chỉ định ngôn ngữ...):**
> **Persona:** "Bạn là trợ giảng thân thiện của khóa AI, luôn khuyến khích sinh viên. Trả lời ngắn gọn, dễ hiểu, ưu tiên ví dụ thực tế. Hỗ trợ bằng tiếng Việt."
> 
> **Giải thích:** (1) *"Trả lời ngắn gọn"* — giới hạn độ dài phản hồi để sinh viên không bị ngợp thông tin, phù hợp với môi trường học tập từng bước. (2) *"Ưu tiên ví dụ thực tế"* — giúp kiến thức AI trừu tượng trở nên dễ hình dung, tăng khả năng ghi nhớ. (3) *"Tiếng Việt"* — xóa bỏ rào cản ngôn ngữ cho sinh viên Việt Nam.

### Câu 4.2 — Hạn chế & cải thiện
**Trợ lý của bạn hiện có hạn chế lớn nhất là gì (ví dụ: history chỉ 3 lượt,
không có bộ nhớ dài hạn, không kiểm duyệt nội dung...)? Đề xuất một cải
thiện cụ thể và mô tả ngắn cách triển khai:**
> **Hạn chế lớn nhất:** Không có bộ nhớ dài hạn — mỗi phiên làm việc đều bắt đầu từ đầu, không nhớ thông tin người dùng, tiến độ học tập hay lịch sử tương tác từ phiên trước.
> 
> **Cải thiện:** Dùng database nhẹ (SQLite) để lưu context người dùng. Cụ thể: thêm tham số `user_id` vào `run_assistant`. Khi phiên kết thúc, lưu `{"user_id": ..., "summary": tóm_tắt_phiên, "key_concepts": [...], "last_topic": ...}` vào SQLite. Khi người dùng quay lại, load summary để tạo system prompt động: `persona = persona_gốc + "\nLần trước bạn đã học về: " + summary`. Giải pháp này nhẹ, không cần infrastructure phức tạp mà vẫn mang lại trải nghiệm "liên tục" cho người dùng.

---

## Danh Sách Kiểm Tra Nộp Bài

- [ ] `python grade.py` — xem điểm tự động, mục tiêu ≥ 75/100
- [ ] Cả 4 checkpoint pytest đều pass
- [ ] Tất cả 9 câu trong file này đã được trả lời
- [ ] Đã copy bài làm vào folder `solution/` và zip theo hướng dẫn README
