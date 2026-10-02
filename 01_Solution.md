# Solution and Evaluation

## Tóm tắt
Dự án em đề xuất không hướng tới xây dựng một nền tảng thay thế sách giáo khoa quốc gia, cũng không phải là một thư viện lưu trữ dài hạn.

Vai trò của dự án là một công cụ ngắn hạn, giải quyết vấn đề thiếu sách giáo khoa, tài liệu học tập của học sinh Việt Nam ở thời điểm hiện tại. Nền tảng này (trong 2-4 tuần đầu năm học) sẽ giúp giáo viên nhanh chóng giải quyết tình trạng gián đoạn học tập bằng cách cung cấp kiến thức trọng tâm một cách hợp pháp, trong thời gian chờ đợi sách giáo khoa mới được xuất bản.

## 1. Về dự án
Dựa trên các nguyên tắc được liệt kê tại [Khảo sát thị trường & Khung pháp lý](02_Research_and_Law.md) và trích dẫn hợp lý, giải pháp này đảm bảo tính bao phủ từ thành phố lớn, cơ sở hạ tầng phát triển đến các vùng có cơ sở hạ tầng kém phát triển hơn.

## 2. Các mô hình trí tuệ nhân tạo đề xuất
Để xử lý đặc thù của sách giáo khoa (bố cục nhiều cột, tiếng Việt có dấu, công thức Toán/Lý/Hóa), hệ thống ưu tiên các mô hình có khả năng hiểu tài liệu đa phương thức và xuất trực tiếp mã LaTeX + Markdown.

- **Lựa chọn ưu tiên: Gemini 3.8 Flash (API)**
    - *Lý do:* Gemini 3.8 Flash qua Gemini API. Model hỗ trợ đầu vào text, image, video, audio và pdf; cửa sổ ngữ cảnh đầu vào tối đa 1.048.576 token và đầu ra tối đa 65.536 token. Gemini 3.8 Flash xử lý được cho các tác vụ đa bước, autonomous agents và workflow phức tạp, model hỗ trợ function calling, code execution, file search, structured outputs, search grounding, URL context và mức suy luận điều chỉnh được. Free Tier hiện có giá token standard bằng 0, nhưng quota, rate limit, điều kiện dữ liệu và danh sách tính năng cần được xác minh tại thời điểm triển khai. <sup>[[3]](#ref3)</sup><sup>[[4]](#ref4)</sup>.
- **Lựa chọn Thay thế (Dành cho mã nguồn mở tự lưu trữ): Qwen2-VL (2B/7B) hoặc GOT-OCR 2.0**
    - *Lý do:* Qwen công bố các bản 2B và 7B theo Apache 2.0. Qwen2-VL hỗ trợ Naive Dynamic Resolution, cho phép xử lý ảnh với độ phân giải và tỉ lệ khung hình khác nhau bằng số visual token động; đồng thời công bố hỗ trợ OCR đa ngôn ngữ bao gồm tiếng Việt, hiểu tài liệu/bảng và ví dụ trích xuất công thức sang Markdown có LaTeX. Bản 2B phù hợp hơn cho thử nghiệm hạn chế tài nguyên, còn bản 7B nên được benchmark nếu ưu tiên chất lượng. Hiệu quả bảo toàn layout cột, bảng và công thức cần được đánh giá riêng trên tập sách tiếng Việt của dự án <sup>[[5]](#ref5)</sup>.

## 3. Kiến trúc dự án
### 3.1. Dành cho khu vực cơ sở hạ tầng phát triển
Giải pháp cần đi theo nguyên tắc: Giáo viên không cần cài đặt ứng dụng mới, hệ thống được xây dựng hoàn toàn trên hệ sinh thái Google (Google Forms + Apps Script) kết hợp lệnh gọi API để tối ưu hóa trải nghiệm ngưởi dùng.

- **Bước 1 - Đầu vào:** Giáo viên truy cập đường link Google Forms của dự án trên điện thoại hoặc máy tính, nhập địa chỉ Email và tải lên ảnh chụp 5-6 trang sách cần giảng dạy.
- **Bước 2 - Tiền xử lý & Gọi AI:** Sau khi gửi Form sẽ tự chạy Google Apps Script. Ảnh được chuyển đổi sang định dạng Base64. Script sử dụng thuật toán *Exponential Backoff* để gọi API Gemini 3.8 Flash, đảm bảo không bị nghẽn mạng (Rate Limit) khi có nhiều giáo viên truy cập cùng lúc. Mô hình AI tiến hành tách văn bản, công thức, và loại bỏ hình ảnh có bản quyền. Đầu ra thô là định dạng Markdown + LaTeX.
- **Bước 3 - Output:** Apps Script tự động biên dịch văn bản thô từ bước 2, tạo thành một tập tin **Google Docs** (tương đương `.docx`). Script cấp quyền chỉnh sửa cho Email của giáo viên và gửi link truy cập trực tiếp.
- **Bước 4 - DRM:** Nhờ định dạng mở của Google Docs, giáo viên có thể chỉnh sửa số liệu để tạo phiếu học tập hợp pháp. Để ngăn chặn việc hình thành kho lưu trữ tài liệu lậu, hệ thống có cơ chế tự động xóa tệp (hoặc thu hồi quyền truy cập) sau **6 giờ** tính từ thời điểm tạo, yêu cầu giáo viên sau khi tạo xong tài liệu thì phải tải về ngay lập tức, tránh tình trạng quá thời gian yêu cầu trên, hệ thống sẽ khóa quyền truy cập file.

### 3.2. Phương án dành cho khu vực cơ sở hạ tầng kém phát triển hơn
Đối với các trường học ở khu vực khó khăn, không có internet, thiết bị thông minh hoặc khi hệ thống gặp sự cố, giải pháp chuyển đổi sang mô hình quản trị tài sản vật lý (dựa trên Điều 20, Luật SHTT):

- Trường học giữ lại 10-20 cuốn sách vật lý (từ khóa trước hoặc quỹ trường) làm tài sản chung tại lớp. Sách không phát về nhà mà được luân chuyển giữa các nhóm học sinh ngay trong tiết học.

- Phiếu học tập: Giáo viên tóm tắt lại kiến thức tuần đó ra một tờ A4 (có thể thay đổi số liệu bài tập) và photocopy để phát cho học sinh. Bản in này là tài liệu hợp pháp do giáo viên tự soạn, giải quyết được nhu cầu muốn có tài liệu học tập mà không cần bất kỳ kết nối internet nào.

## 4. Triển khai dự án
Sản phẩm mà người dùng sẽ tương tác là một đường link google forms, khi người dùng nhấn vào link trên điện thoại, nó sẽ chuyển hướng sang trình duyệt (Chrome, Google, Safari,...) hoặc mở trực tiếp google form trong trình duyệt tích hợp của Zalo/Messenger. Các lợi ích khi triển khai dự án này có thể thấy như sau:

- Giao diện người dùng rất quen thuộc là Google Forms kết hợp với Email $\rightarrow$ Tối ưu hóa trải nghiệm người dùng.
- Vì Zalo/Messenger là công cụ trao đổi học tập thường xuyên giữa giáo viên-học sinh, giáo viên-giáo viên hay học sinh-học sinh. Nên việc tận dụng một đường link đơn giản, có thể gửi đi nhanh, dễ thao tác (chỉ cần click link), theo em, là một lợi thế (dễ lan tỏa công nghệ đến nhiều người, tối đa hóa được trải nghiệm người dùng).

Ngoài ra, để tăng khả năng hoạt động, ta có thể tận dụng mạng lưới từ các cộng đồng công nghệ để phân phối mã nguồn mở. Sinh viên tại các địa phương có thể tự cài đặt cấu hình hệ thống này và triển khai cục bộ để hỗ trợ trực tiếp cho các trường cấp 3 tại quê hương họ.

Cấu trúc của file `.docx` có thể chèn thêm các header/footer với dòng chữ "Tạo tài liệu tương tự tại `<url>`".

## 5. Đánh giá hệ thống
### 5.1. Lợi thế
- **Chi phí vận hành gần như bằng 0:** Sử dụng Google App Script và gói API miễn phí của Gemini 3.8 Flash, dự án không tốn chi phí thuê máy chủ (VPS) hay bảo trì hạ tầng.
- **An toàn về mặt pháp lý:** Giải quyết bài toán bản quyền bằng cách biến đổi hình thức (chỉ lấy văn bản, công thức trọng tâm), kết hợp DRM (xóa sau 6 giờ) tránh ảnh hưởng doanh thu của Nhà xuất bản.
- **Tốc độ lan tỏa cao:** Hệ thống có thể tạo và phân phối học liệu đồng loạt cho toàn bộ học sinh trong lớp học, với thao tác đơn giản là chụp màn hình $\rightarrow$ Gửi form $\rightarrow$ Có tài liệu học.

### 5.2. Điểm yếu và rủi ro 
- **Thiếu tính trực quan:** Nếu bỏ hoàn toàn biểu đồ, bản đồ địa lý hoặc hình học không gian có bản quyền, chất lượng phiếu học tập sẽ giảm tính trực quan, gây khó khăn cho học sinh trong việc tiếp thu.
- **Phụ thuộc chất lượng mô hình:** Mô hình AI vẫn có nguy cơ mắc hallucination (nhận diện sai dấu câu, công thức phức tạp). Hệ thống bắt buộc phải có sự kiểm định từ giáo viên kinh nghiệm trước khi in ấn phát cho học sinh. Hơn nữa, vì là mô hình được triển khai ở dạng thương mại free, nên dẫn đến số lượng quota bị giới hạn và gây khó chịu cho người dùng nếu ứng dụng tiếp cận được nhiều học sinh, giáo viên hơn.

## 6. Hướng Phát triển Tương lai
- **Truy xuất hình ảnh có nhận thức bản quyền:** Để khắc phục điểm yếu về hình ảnh minh họa mà không vi phạm pháp luật hoặc tốn chi phí tạo sinh ảnh (dùng thêm mô hình GenAI), hệ thống sẽ tích hợp module tự động gọi API (Google Custom Search) . Module này truyền vào các tham số lọc siêu dữ liệu để tự động tìm kiếm các hình ảnh liên quan thuộc phạm vi Creative Commons Zero (CC0) hoặc Public Domain <sup>[[6]](#ref6)</sup> và chèn vào file `.docx`.
- **Tích hợp API Cấp Trường:** Xây dựng các Webhook chuẩn hóa, cho phép các trường học sở hữu hệ thống quản lý học tập riêng có thể nhúng thẳng tính năng tách tài liệu này vào giao diện nội bộ của trường, nâng cao tính bảo mật và đồng bộ dữ liệu.

- **Caching và memorization**:
    - Tình huống giả định: Nếu cả 2 giáo viên A và B sử dụng hệ thống để tạo ra tài liệu về cùng một chương là chương 1 sách toán 12 tập 1. Kết quả là hệ thống gọi API đến mô hình AI 2 lần để tạo ra cùng một bộ tài liệu cho cùng một chương -> Chi phí tính toán nhân lên, tiêu tốn nhiều token không cần thiết đặc biệt là mô hình được sử dụng là mô hình miễn phí (có giới hạn lượng token được sử dụng trong một khoảng thời gian).
    - Giải pháp đề xuất: Sử dụng cơ chế tương tự như Youtube sử dụng google global cache (GGC) <sup>[[1]](#ref1)</sup><sup>[[2]](#ref2)</sup>. Có nghĩa là, sau khi nhận được ảnh tải lên từ người dùng, hệ thống sẽ gọi một luồng công nghệ (có thể là fast-OCR) để nhận biết trang sách đã tồn tại trong Database chưa, nếu đã tồn tại (kết quả giống nhau trên 92%), ta chỉ cần trả về file kết quả, nếu chưa có, ta tiến hành gọi model để xử lý. Sau khi model xử lý xong và ra được file kết quả cuối cùng, ta tiến hành lưu trữ ID của nó bằng tên của bài học (ví dụ: Bài 1: đạo hàm. Ta chuẩn hóa chuỗi thành bai1daoham $\rightarrow$ Băm chuỗi thành một id duy nhất và dán nhãn cho file này). Kết quả là chỉ cần 2 trang sách có độ giống nhau trên 92%, ta đều có cùng một ID trong database và sẽ trả về kết quả, việc này nhanh hơn rất nhiều và đỡ tốn token hơn là gọi model để xử lý ảnh ngay lập tức.


## Tài liệu tham khảo
<a id="ref1"></a> \[1\]: [Google, "Support Google - interconnect help", section Content Served, High-volume traffic](https://support.google.com/interconnect/answer/7658599)  
<a id="ref2"></a> \[2\]: [Google, "Peering Google, Data Centers"](https://peering.google.com/static/js/site/modules/pages/infrastructure.html)  
<a id="ref3"></a> \[3\]: [Google AI, "Models, All models, Gemini 3.8 Flash"](https://ai.google.dev/gemini-api/docs/models/gemini-3.8-flash)  
<a id="ref4"></a> \[4\]: [Firebase Google, "Mô hình - Ngôn ngữ được hỗ trợ"](https://firebase.google.com/docs/ai-logic/models?hl=vi)  
<a id="ref5"></a> \[5\]: [Peng Wang et al, "Qwen2-VL: Enhancing Vision-Language Model’s Perception of the World at Any Resolution", Abstract, Introduction](https://arxiv.org/html/2409.12191v2)  
<a id="ref6"></a> \[6\]: [Google, "Custom Search JSON API", Query Parameter section, parameter's name is rights](https://developers.google.com/custom-search/v1/reference/rest/v1/cse/list#query-parameters) 
