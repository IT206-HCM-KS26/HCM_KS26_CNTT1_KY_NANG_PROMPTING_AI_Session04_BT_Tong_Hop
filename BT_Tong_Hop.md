# BÀI TẬP SESSION 04 [BTTH] Bài tập thực hành tổng hợp
## Phần 1: Báo cáo thiết kế giải pháp
### 1. Bối cảnh 
- Một cửa hàng bán lẻ công nghệ nhận hàng trăm tin nhắn/phản hồi từ khách hàng mỗi ngày với nội dung lộn xộn (vừa khen, vừa chê, hỏi bảo hành, hoặc spam ký tự lạ). Nếu dùng 1 prompt duy nhất, AI dễ bị quá tải thông tin, bỏ sót lỗi hoặc trả lời sai định dạng.

- **Giải pháp:** Xây dựng quy trình Prompt Chaining (Chuỗi Prompt) gồm 3 bước chạy tuần tự khép kín, kết hợp Chain-of-Thought (CoT) để suy luận logic và để xử lý dữ liệu bị đứt gãy.

### 2. Cơ chế kỹ thuật áp dụng

- **Prompt Chaining:** Đầu ra của bước trước là cấu trúc JSON tinh gọn, làm đầu vào duy nhất cho bước sau.
- **Chain-of-Thought (CoT):** Ở Bước 2, AI buộc phải suy luận từng bước (Vấn đề là gì $\rightarrow$ Ảnh hưởng thế nào $\rightarrow$ Chấm điểm ưu tiên) trước khi kết luận.
- **Chống đứt gãy ngữ cảnh:** Mỗi prompt đều kèm thẻ [MỤC TIÊU CỐT LÕI] và chỉ truyền dữ liệu cần thiết (loại bỏ lời chào thừa) để tiết kiệm token và giữ AI không đi lệch hướng.
- **Bắt lỗi dữ liệu:** Nếu dữ liệu đầu vào rỗng, sai ngôn ngữ hoặc chứa ký tự vô nghĩa, Bước 1 tự động trả về mã lỗi chuẩn {"status": "ERROR"} để dừng chuỗi an toàn, không làm sập hệ thống.

### 3. Sơ đồ kiến trúc chuỗi

-> [Dữ liệu thô] 

-> [BƯỚC 1: Lọc rác và Bóc tách thực thể (Data Extraction) + Kỹ thuật: Guardrails (Bắt lỗi) + JSON chuẩn hóa] 

-> [BƯỚC 2: Phân tích nguyên nhân & Mức độ ưu tiên (CoT) + Kỹ thuật: Few-shot CoT + Context Anchoring] 

-> [BƯỚC 3: Soạn phản hồi khách hàng (Actionable Output) + Kỹ thuật: Role-playing + Ràng buộc độ dài tối ưu token]
    

## Phần 2: Prompt thực thi các bước (3 bước)

### Bước 1: Prompt Lọc lỗi và chuẩn hóa dữ liệu

[MỤC TIÊU CỐT LÕI]: Bóc tách thông tin từ phản hồi khách hàng thành JSON chuẩn.

[QUY TẮC BẮT LỖI]:
- Nếu đầu vào rỗng, chỉ có số/ký tự đặc biệt, hoặc không có nghĩa: Chỉ trả về đúng chuỗi sau và dừng lại:
  {"status": "ERROR", "reason": "Du lieu khong hop le"}

[ĐỊNH DẠNG ĐẦU RA (JSON)]:

{

  "status": "OK",

  "san_pham": "<Tên sản phẩm hoặc 'Không rõ'>",

  "van_de_chinh": "<Tóm tắt ngắn gọn dưới 15 từ>"

}

[DỮ LIỆU ĐẦU VÀO]:

{{INPUT_TEXT}}

### Bước 2: Prompt Phân tích suy luận (Chain-of-Thought)

[MỤC TIÊU CỐT LÕI]: Đánh giá mức độ nghiêm trọng của sự cố dựa trên JSON từ Bước 1.

[QUY TẮC SUY LUẬN (CoT)]:
Hãy suy nghĩ theo 3 bước ngắn gọn trước khi xuất kết quả:

1. Phân loại: Sự cố thuộc nhóm nào (Kỹ thuật / Giao hàng / Thái độ / Khen ngợi)?

2. Tác động: Có làm gián đoạn việc sử dụng sản phẩm ngay lập tức không?

3. Chấm điểm: Gán nhãn KHẨN CẤP (hỏng hóc, mất tiền) hoặc BÌNH THƯỜNG (góp ý, hỏi đáp).

[VÍ DỤ MẪU]:

Input: {"status": "OK", "san_pham": "Chuột không dây", "van_de_chinh": "Hỏng nút click trái sau 2 ngày"}

Suy luận: 1. Lỗi kỹ thuật -> 2. Không dùng được chuột -> 3. KHẨN CẤP.

Output: {"muc_do": "KHẨN CẤP", "nhom": "Kỹ thuật", "tom_tat": "Chuột không dây hỏng nút click trái"}

[DỮ LIỆU TỪ BƯỚC 1]:

{{STEP_1_OUTPUT}}

### Bước 3: Prompt Tạo phản hồi tự động (Tối ưu Token)

[MỤC TIÊU CỐT LÕI]: Viết tin nhắn phản hồi khách hàng dựa trên kết quả phân tích Bước 2.

[RÀNG BUỘC]:
- Vai trò: Nhân viên CSKH chuyên nghiệp, lịch sự.
- Độ dài: Tối đa 50 từ (tiết kiệm token, đi thẳng vào vấn đề).
- Nếu mức độ là "KHẨN CẤP": Xin lỗi và cam kết đổi mới/xử lý trong 24h.
- Nếu mức độ là "BÌNH THƯỜNG": Cảm ơn và ghi nhận ý kiến.

[DỮ LIỆU TỪ BƯỚC 2]:

{{STEP_2_OUTPUT}}

## Phần 3: Kết quả chạy thử thực tế

- **Text Phần 1:** Dữ liệu hợp lệ (Trường hợp sự cố nghiêm trọng)

   - **Dữ liệu đầu vào thô:** "Shop làm ăn kiểu gì vậy, cái bàn phím cơ mới mua hôm qua nay liệt mất phím Space rồi, cần xử lý gấp để làm việc!"

   - **Kết quả ở bước 1:**



  "is_valid_ticket": true,

  "noise_filtered": "Shop làm ăn kiểu gì vậy",

  "entities": {

    "product": "Bàn phím cơ",

    "purchase_time": "1 ngày trước",

    "defect": "Liệt phím Space",

    "context": "Đang cần dùng để làm việc",

    "urgency": true
  

- **Text Phần 2:** 

  - **Dữ liệu đầu vào thô:** @@@ ### 999999 [null] ???

  - **Kết quả Bước 1 (Bắt lỗi thành công):**
  
   {"status": "ERROR", "reason": "Du lieu khong hop le"}

