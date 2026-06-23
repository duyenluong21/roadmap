1. LEMP stack là gì?

- L: Linux => Hệ điều hành
- E: Nginx => Web server tốc độ cao chịu trách nhiệm xử lý các yêu cầu HTTP đầu vào
- M: MySQL => Cơ sở dữ liệu
- P: PHP => Ngôn ngữ lập trình phía máy chủ

- LEMP sử dụng PHP-FPM cho phép nginx duy trì các tiến trình hoạt động sẵn sàng xử lý

2. Tại sao cần đến nginx?

- Xử lý lưu lượng khủng: Ngix rất nhẹ, tiêu tốn ít bộ nhớ. Nó xử lý hàng chục nghìn yêu cầu mượt mà
- Ngoài làm web server, nginx còn được sử dụng như reverse proxy, load balancer, caching và API gateway
- Bảo mật: Hỗ trợ access control, IP blocking, chống DDoS và nhiều tính năng bảo vệ dữ liệu
- Nginx liên tục được cập nhật để hỗ trợ HTTP/2, HTTP/3, gRPC và Websocket
