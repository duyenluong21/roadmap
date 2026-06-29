# Câu hỏi phỏng vấn

## 1. Dùng AI như nào? Dùng phương pháp gì?

**Cách dùng AI trong công việc:**
- Hỗ trợ code: sinh code mẫu, refactor, viết unit test, giải thích code lạ, tìm bug nhanh
- Tra cứu & học: thay cho việc đọc tài liệu dài, hỏi trực tiếp để hiểu nhanh khái niệm mới
- Xử lý dữ liệu/văn bản: tóm tắt, phân loại, trích xuất thông tin, viết tài liệu
- Tích hợp vào sản phẩm: chatbot, gợi ý, tìm kiếm thông minh

**Phương pháp:**
- **Prompt engineering**: viết prompt rõ ràng, có ngữ cảnh, đưa ví dụ (few-shot), yêu cầu output theo định dạng cụ thể
- **RAG (Retrieval-Augmented Generation)**: nạp dữ liệu nội bộ (tài liệu, DB) cho AI để trả lời đúng theo nghiệp vụ, giảm bịa đặt
- **Function calling / Agent**: cho AI gọi tool, API để tự thực hiện tác vụ
- **Fine-tuning**: huấn luyện thêm trên dữ liệu riêng khi cần model chuyên biệt (chi phí cao, cân nhắc dùng RAG trước)

**Nguyên tắc:** luôn kiểm chứng lại kết quả AI, không tin tuyệt đối, đặc biệt với code và số liệu.

## 2. Tính năng khó về mặt công nghệ?

**Bối cảnh:** Dự án migrate hệ thống chứng khoán từ Fuel sang Laravel gặp một vấn đề: các API key của bên thứ 3 được hardcode trong code. Khi chạy test để kiểm tra tính đúng đắn của quá trình migrate, các request thật sẽ bị gửi ra ngoài tới các service thật — điều này không thể chấp nhận được trong môi trường test. Để giải quyết, nhóm xây dựng một hệ thống proxy đặt tên là **Blackhole**.

Thách thức kỹ thuật lớn nhất của Blackhole là **chặn và ghi log HTTPS traffic mà không forward request ra ngoài** — tức là thực hiện MITM (Man-in-the-Middle) proxy cho HTTPS.

Cụ thể, vấn đề là:

**1. HTTPS không thể chặn thông thường.** Khi **Fuel** gọi một API bên thứ 3 qua HTTPS, client gửi lệnh `HTTP CONNECT` để thiết lập tunnel TLS mã hóa tới server đích. Một forward proxy bình thường chỉ đục tunnel đó qua — không đọc được nội dung bên trong. Blackhole cần đọc được plaintext request bên trong tunnel đó mà không forward đi đâu.

**2. Giải pháp: MITM với CA tự ký nhúng trong binary.** Blackhole nhúng một CA certificate và private key trực tiếp vào source code Go. Khi nhận được lệnh `CONNECT`, thay vì đục tunnel ra ngoài, Blackhole:
- Hijack TCP connection (`http.Hijacker`) để giành quyền kiểm soát socket
- Gửi `200 Connection Established` để client tiếp tục
- Tự mình đóng vai TLS server, sinh certificate leaf động cho đúng hostname, ký bằng CA nội bộ
- Thực hiện TLS handshake với client
- Đọc plaintext HTTP request bên trong tunnel và lưu vào SQLite

**3. Vấn đề trust certificate ở PHP.** PHP/OpenSSL không tin CA tự ký theo mặc định, sẽ báo lỗi SSL. Giải pháp là thêm CA cert vào bundle CA của Homebrew PHP và inject biến môi trường `http_proxy`/`https_proxy` vào một PHP-FPM pool riêng biệt — chỉ ảnh hưởng đến site cần test, không ảnh hưởng các site khác trên Valet.

**Tóm lại**, cái khó không phải là "bắt request" — mà là bắt được **HTTPS request đã mã hóa**, đọc nội dung bên trong mà không làm vỡ TLS handshake của client, và làm điều đó hoàn toàn transparent với ứng dụng đang chạy.

## 3. Tính năng khó về mặt nghiệp vụ?

**Tính năng chấm công & Tính công (HRM)**

Tính năng chấm công nghe có vẻ đơn giản, nhưng thực tế là một bài toán phụ thuộc dây chuyền — không có module nào hoạt động độc lập được.

Để tính được công, trước tiên phải có **quản lý nhân viên** để biết ai đang làm việc, thuộc ca nào, áp dụng quy chế nào. Tiếp theo phải có **quản lý ngày nghỉ lễ, ngày làm bù, ngày du lịch công ty** — vì đây là nền tảng để xác định một ngày cụ thể có được tính công hay không, có được nghỉ hưởng lương hay không. Chỉ khi có đủ hai lớp dữ liệu đó thì module **tính công** mới có thể vận hành chính xác.

Cái khó thực sự nằm ở nghiệp vụ: giờ chấm công **linh hoạt theo từng ca**, và khi nhân viên thực hiện chấm công, hệ thống không chỉ ghi nhận thời gian mà còn **kiểm tra IP và vị trí địa lý** để xác minh nhân viên đang thực sự ở tại công ty. Tuy nhiên, nếu nhân viên có đăng ký WFH thì điều kiện này được bỏ qua hoàn toàn — hệ thống phải tra cứu trạng thái WFH trước khi áp dụng bất kỳ ràng buộc địa lý nào.

Sau khi chấm công, hệ thống so sánh giờ check-in/check-out thực tế với ca làm việc, tính ra số phút đi muộn và về sớm, rồi **quy đổi thành phép năm** để trừ vào số ngày phép còn lại. Nếu nhân viên có **đơn nghỉ phép được duyệt**, công của ngày đó sẽ được tính theo đơn thay vì theo dữ liệu chấm công thực tế — tức là logic tính công phải luôn kiểm tra đơn từ trước khi ra kết quả cuối cùng.

Thêm vào đó, để ra được con số công cuối tháng, hệ thống phải đối chiếu với lịch nghỉ lễ, ngày làm bù, và ngày du lịch công ty — mỗi loại có cách tính công khác nhau và có thể giao thoa với nhau. Xử lý đúng tất cả các trường hợp này mà không để sót hay tính nhầm là phần tốn nhiều công sức nhất trong toàn bộ tính năng.

## 4. Dự án đem lại hiệu quả kinh doanh tốt nhất?

**Dự án mang lại hiệu quả kinh doanh tốt nhất: HRM**

**Trước khi có hệ thống:**

- Chấm công phải thuê bên thứ 3 lắp camera, cuối tháng HR tự đối chiếu và tính toán thủ công trên Excel — tốn thời gian và dễ sai
- Nghỉ phép xin qua nhóm chat, cuối tháng kế toán phải đọc lại toàn bộ tin nhắn để tổng hợp — không có gì lưu trữ bài bản
- Thiết bị công ty chỉ mượn qua miệng, không biên bản, không bằng chứng — thất lạc không truy được trách nhiệm

**Sau khi triển khai HRM:**

- Chấm công tự động, kiểm tra IP và vị trí địa lý, kết quả tính công cuối tháng ra tự động — công ty không còn cần thuê dịch vụ camera bên ngoài
- Nghỉ phép có luồng phê duyệt rõ ràng, lưu lại đầy đủ, kế toán chỉ cần xuất báo cáo thay vì đọc lại hàng trăm tin nhắn
- Thiết bị được ghi nhận từ lúc bàn giao đến lúc thu hồi, có ký nhận, truy được lịch sử — tránh tranh chấp và thất lạc

**Kết quả:**

- Cắt chi phí thuê dịch vụ bên thứ 3
- Giảm đáng kể thời gian xử lý thủ công của HR và kế toán mỗi tháng
- Minh bạch hóa tài sản nội bộ, giảm rủi ro thất lạc không rõ trách nhiệm
