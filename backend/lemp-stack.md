1. LEMP stack là gì?

- L: Linux => Hệ điều hành
- E: Nginx => Web server tốc độ cao chịu trách nhiệm xử lý các yêu cầu HTTP đầu vào
- M: MySQL => Cơ sở dữ liệu
- P: PHP => Ngôn ngữ lập trình phía máy chủ

- LEMP sử dụng PHP-FPM cho phép nginx duy trì các tiến trình hoạt động sẵn sàng xử lý

2. Tại sao cần đến nginx?

- Trước đó: PHP có thể tự chạy bằng built-in server (`php -S`), không cần web server riêng. Nhưng nhược điểm khiến nó chỉ hợp để dev, không dùng production:
    - Chỉ xử lý 1 request tại 1 thời điểm (blocking) => đông người là nghẽn
    - Không có HTTPS/TLS
    - Phục vụ file tĩnh (ảnh, CSS, JS) rất kém, không cache, không tối ưu
    - Không có reverse proxy, load balancer, chống DDoS, IP blocking
- => Production cần nginx đứng trước: nhận request, phục vụ file tĩnh, lo SSL/bảo mật, rồi mới đẩy phần động cho PHP-FPM
- Xử lý lưu lượng khủng: Ngix rất nhẹ, tiêu tốn ít bộ nhớ. Nó xử lý hàng chục nghìn yêu cầu mượt mà
- Ngoài làm web server, nginx còn được sử dụng như reverse proxy, load balancer, caching và API gateway
- Bảo mật: Hỗ trợ access control, IP blocking, chống DDoS và nhiều tính năng bảo vệ dữ liệu
- Nginx liên tục được cập nhật để hỗ trợ HTTP/2, HTTP/3, gRPC và Websocket

3. Nginx khác gì Apache?
- Kiến trúc: Apache hoạt động dựa trên kiến trúc hướng quy trình. Nginx hoạt động dựa trên kiến trúc hướng sự kiện

=> Việc xử lý các truy vấn từ client đến server bằng các thread, mỗi truy vấn client sẽ được server xử lý bằng 1 thread riêng biệt cho đến khi nó hoàn thành nhiệm vụ sẽ làm ổ cứng trên server mất nhiều thời gian hơn để giải phóng các thread đã hoàn thành nhiệm vụ, và việc tạo ra nhiều thread cũng sẽ làm server tiêu hao rất nhiều tài nguyên

Apache:
    - Mỗi request đến có thể được 1 process xử lý => Tốn nhiều RAM, ưu điểm không ảnh hưởng đến các process khác
    - Mỗi request đến có thể được xử lý bởi 1 thread => tiết kiệm RAM hơn

Nginx:
    - Hướng sự kiện có nghĩa là các thông báo hoặc tín hiệu được sử dụng để đánh dấy sự khởi đầu hay hết thúc của 1 process. Như vậy, các tài nguyên có thể được sử dụng bởi các process khác cho đến khi 1 process khởi tạo sự kiện được kích hoạt và tài nguyên có thể được phân bố và phát hành tự động -> Dẫn đến việc sử dụng tối ưu hóa bộ nhớ và CPU
    - Không đồng bộ có nghĩa là các thread có thể xử lý đồng thời, giúp tăng cường việc chia sẻ tài nguyên không bị chặn
    - Đơn luồng nghĩa là nhiều client có thể được xử lý bởi 1 worker process duy nhất với các tài nguyên không bị chặn
    - Nginx sinh ra các worker process, mỗi worker process có thể xử lý hàng ngàn kết nối bằng cách thực hiện 1 cơ chế lặp nhanh chóng để liên tục kiểm tra và xử lý các sự kiện
    - Mỗi kết nối được xử lý bởi worker được đặt trong vòng lặp. Bên trong vòng lặp, các sự kiện được xử lý đồng bộ, cho phép công việc được xử lý 1 cách không chờ đợi. Khi kết nối đóng lại thì worker process bị xóa khỏi vòng lặp
- Với nội dung tĩnh và động:
    - Apache: 
        - Các Apache server có thể xử lý nội dung tĩnh bằng cách sử dụng các phương pháp dựa trên các file thông thường của nó => Hiệu suất chủ yếu phụ thuộc vào các module đa xử lý MPMs được trình bày ở trên
        - Apache cũng có thể xử lý nội dung động bằng cách nhúng bộ xử lý của nó
    - Nginx:
        - Không có khả năng xử lý các nội dung động mà chỉ phục vụ các nội dung tĩnh
        - Với nội dung động, Nginx phải gọi đến bộ vi xử lý bên ngoài để thực thi và chờ kết quả gửi lại
- Nginx không có file .htaccess (được đọc trên mọi yêu cầu => thực thi ngay lập tức), nhược điểm là giảm hiệu suất, bảo mật kém. Nginx không có nhưng có lợi thế về bảo mật, hiệu suất
- Nginx: Có ít module hơn nhưng cấu hình để làm proxy ngược, cân bằng tải hiệu quả. Apache: Có hệ sinh thái module khổng lồ, rất thân thiện cho lập trình viên nếu thiết lập các cấu hình phức tạp

4. http1 vs http2
- Định dạng dữ liệu:
    - http1: Văn bản thuần túy (Plain text)
    - http2: Nhị phân - máy tính hiểu và xử lý nhanh hơn, ít lỗi hơn
- Request:
    - http1: Tuần tự => Đợi request
    - http2: đa kênh => Nhiều request mà không bị chặn
- Số lượng kết nối TCP
    - hppt1: Cần nhiều kết nối TCP song song
    - http2: 1 kết nối TCP duy nhất cho toàn bộ phiên làm việc
- Nén header
    - http1: không nén, lãng phí băng thông
    - http2: sử dụng thuật toán HPACK để nén header, giảm thiểu dữ liệu thừa
- Cơ chế server push
    - http1: Không hỗ trợ. Server chỉ phản hồi khi client yêu cầu
    - http2: Có hỗ trợ. Server có thể chủ động gửi các tệp (như CSS/JS) về trước khi Client kịp yêu cầu

5. Bare Metal Server vs VPS
Bare Mertal
- Ưu điểm:
    - Hiệu suất tối đa: cung cấp toàn bộ tài nguyên phần cứng cho 1 ứng dụng
    - Toàn quyền kiểm soát: Người dùng có thể tự thiết lập và quản lý hệ điều hành, phần mềm, phân vùng lưu trữ và cấu hình phần cứng
    - Bảo mật: Dữ liệu và ứng dụng được cô lập hoàn toàn
    - Độ ổn định cao: Môi trường vận hành ít chịu tác động bên ngoài, giảm nguy cơ xung đột hệ thống và ảnh hưởng hiệu năng giữa các ứng dụng
- Nhược điểm:
    - Khó mở rộng nhanh chóng
    - Chi phí đầu tư lớn
    - Thiếu tính linh hoạt
    - Thời gian triển khai dài

VPS:
- Ưu điểm:
    - Chi phí thấp
    - Dễ mở rộng: Nâng cấp CPU, RAM, storage chỉ với vài click
    - Dễ quản lý: Nhà cung cấp hỗ trợ quản lý
    - Toàn quyền cấu hình hệ điều hành và ứng dụng
- Nhược điểm:
    - Hiệu năng phụ thuộc vào tài nguyên chia sẻ
    - Không toàn quyền kiểm soát phần cứng
    - Quản lý nâng cao: vẫn yêu cầu kiến thức hệ điều hành, ảo hóa và network