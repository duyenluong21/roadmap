1. Query Optimization (Tối ưu truy vấn)

* Là quá trình viết hoặc cải thiện câu SQL để chạy nhanh hơn và tốn ít tài nguyên hơn.

Mục đích

* Giảm thời gian truy vấn
* Giảm CPU, RAM
* Giảm tải database

Các hướng tối ưu phổ biến

* Thêm index
* Tránh SELECT *
* Giảm JOIN không cần thiết

Khi 1 query chậm, nên nghĩ theo thứ tự

1. SQL viết đúng chưa?
2. Có dùng index chưa?
3. Có lấy quá nhiều dữ liệu không?
4. Join có hợp lý không?
5. Execution Plan nói gì?
6. Có cần cache hoặc pre-aggregation không?

Các cách tối ưu query

1. Chỉ lấy dữ liệu cần thiết

Ví dụ: chỉ cần email thì không nên SELECT *.

Vì sao nhanh hơn?

* Đọc ít dữ liệu hơn
* Truyền ít dữ liệu hơn
* PHP xử lý ít dữ liệu hơn

2. Thêm index

* Không có index thì DB có thể phải quét rất nhiều dòng.
* Có index thì DB tra index trước rồi đến đúng record.

3. Giảm số lượng join

* Join quá nhiều bảng sẽ chậm.
* Database phải:
    * đọc nhiều index
    * so sánh nhiều record
    * dùng nhiều RAM

4. WHERE càng sớm càng tốt

* Giúp Database chỉ xử lý dữ liệu cần thiết.

5. LIMIT

* Chỉ lấy số lượng record cần thiết thay vì lấy toàn bộ.

6. Tránh N+1 query

* Ví dụ trong ORM, lấy danh sách rồi lặp từng record để query dữ liệu liên quan.
* Cần dùng eager loading (with() trong Laravel).

7. Cache kết quả

* Với các query đọc nhiều, ít thay đổi, có thể cache để giảm tải DB.

⸻

2. Execution Plan

* Là kế hoạch Database sử dụng để thực hiện 1 câu SQL.

Ví dụ:

SELECT *
FROM users
WHERE email = 'abc@gmail.com';

Database không chạy ngay, mà sẽ qua các bước:

SQL
↓
Parser (kiểm tra cú pháp)
↓
Optimizer (tìm cách chạy nhanh nhất)
↓
Execution Plan
↓
Thực thi

Execution Plan sẽ cho biết

* Có dùng index không?
* Quét toàn bộ bảng hay chỉ đọc một phần?
* Join theo cách nào?
* Đọc bảng nào trước?
* Ước tính bao nhiêu dòng?

Xem bằng EXPLAIN

EXPLAIN
SELECT *
FROM users
WHERE email = 'abc@gmail.com';

Ví dụ kết quả:

id	table	type	possible_keys	key	rows	Extra
1	users	ref	idx_email	idx_email	1	Using index

Ý nghĩa các cột quan trọng

id

* Thứ tự thực hiện từng phần của query.

table

* Bảng đang được đọc.

type

* Cách Database truy cập dữ liệu.
* Đây là cột rất quan trọng.

Một số giá trị thường gặp:

* const: chỉ đọc đúng 1 record, rất tốt
* ref: dùng index, tốt
* range: quét theo khoảng, ví dụ >, <, BETWEEN
* index: quét toàn bộ index
* ALL: full table scan, thường là dấu hiệu query chưa tối ưu

possible_keys

* Các index mà Database có thể dùng.

key

* Index mà Database thực sự dùng.
* Nếu NULL thì có thể query không dùng index.

rows

* Số dòng Database ước tính phải đọc.
* Số này càng lớn thì query càng có nguy cơ chậm.

Extra

Một số giá trị hay gặp:

* Using index: chỉ đọc index, tốt
* Using where: có lọc thêm bằng điều kiện WHERE
* Using temporary: tạo bảng tạm, có thể chậm
* Using filesort: phải sort ngoài index, có thể chậm nếu dữ liệu lớn

Khi đọc EXPLAIN, nên nhìn gì?

1. type: Có phải ALL (Full Table Scan) không?
2. key: Có dùng index không? Nếu NULL thì vì sao?
3. rows: Database phải đọc bao nhiêu dòng?
4. Extra: Có Using temporary hoặc Using filesort không?

⸻

3. Composite Index

* Là index trên nhiều cột.
* Tại sao lại dùng 
    - Vì trong thực thế query thường không lọc theo 1 cột duy nhất, mà lọc theo nhiều cột cùng lúc
* Điểm quan trọng nhất: Thứ tự cột
    - Chỉ hoạt động tốt nếu query dùng cột bên trái trước
* Nguyên tắc:
    - Cột lọc nhiều / quan trọng đặt trước
    - Cột có độ phân biệt cao thường đáng ưu tiên
    - Nếu query có ORDER BY, cân nhắc đưa cột vào sort sau
* Nhược Điểm
    - Tốn bộ nhớ / Disk: Mỗi index là một cấu trúc dữ liệu riêng
    - Làm chậm insert/update/delete
    - Nếu thiết kế sai thứ tự cột thì hiệu quả kém

Ví dụ:

INDEX(first_name, last_name)

Tốt cho:

WHERE first_name = 'A'

và

WHERE first_name = 'A'
AND last_name = 'B'

Không tối ưu cho:

WHERE last_name = 'B'

vì nguyên tắc Left-most Prefix.

Left-most Prefix

Với index:

(first_name, last_name)

thì DB tối ưu tốt cho:

* first_name
* first_name + last_name

nhưng không tối ưu nếu chỉ search theo last_name.

Từ khóa nhớ:

* Một index cho nhiều cột

⸻

4. Transactions

* Là nhóm nhiều câu SQL thành 1 đơn vị công việc.
* Kết quả:
    * hoặc thành công tất cả
    * hoặc rollback tất cả

Ví dụ

Chuyển tiền:

* Trừ tiền tài khoản A
* Cộng tiền tài khoản B

Nếu A bị trừ tiền nhưng B chưa được cộng thì dữ liệu sai.

Transaction giúp đảm bảo:

* hoặc cả 2 câu lệnh đều thành công
* hoặc nếu lỗi thì rollback toàn bộ

ACID

Atomicity

* Tất cả hoặc không gì cả.

Consistency

* Dữ liệu luôn ở trạng thái hợp lệ trước và sau transaction.

Isolation

* Transaction này không làm hỏng transaction kia.

Durability

* Khi đã commit thì dữ liệu phải được lưu bền vững.

* Kết quả của transation chỉ có 2 khả năng:
    - Commit: tất cả các câu lệnh đều thành công
    - Rollback: nếu có lỗi ở giữa, hủy toàn bộ những gì đã làm

⸻

5. Locking

* Là cơ chế Database khóa dữ liệu để tránh nhiều transaction sửa cùng lúc gây xung đột.

Mục đích

* Tránh mất dữ liệu
* Tránh ghi đè sai
* Đảm bảo dữ liệu nhất quán

Ví dụ

Tài khoản có balance = 100.

* Transaction A trừ 50
* Transaction B trừ 30 cùng lúc

Nếu không có locking thì có thể tính sai số dư.

Một số loại lock hay gặp

Shared Lock

* Cho phép đọc nhưng không cho ghi.

Exclusive Lock

* Chỉ transaction đang giữ lock mới được sửa dữ liệu.

* Lock trong SQL: For update

`START TRANSACTION
SELCT *
FROM accounts
where id = 1
FOR UPDATE`

    => Ý nghĩa:
        - Transaction hiện tại khóa record đó
        - Transaction khác không được sửa record này cho đến khi COMMIT hoặc ROLLBACK

* Lock trong Laravel dùng lockForUpdate()
    - Tức là:
        - lấy record
        - đồng thời khóa record đó trong transaction
        - request khác muốn lock/sửa record đó phải chờ

⸻

6. Isolation Level

* Quy định transaction nhìn thấy dữ liệu của transaction khác đến mức nào.
* Mục tiêu là cân bằng giữa tính đúng đắn và hiệu năng.

Nói dễ hiểu: Nó quyết định transaction A có được nhìn thấy dữ liệu đang dở dang hoặc thay đổi của transaction B không

- 3 mục tiêu:
    - tính đúng đắn của dữ liệu
        - tránh đọc dữ liệu sai
        - tránh bị thay đổi dữ liệu bất ngờ giữa transaction
        - tránh race condition kiểu đọc không ổn định
    - Hiệu năng
        - Càng cô lập mạnh thì càng an toàn
        - Nhưng thường sẽ chậm hơn, lock nhiều hơn, block nhiều hơn
        
Các mức phổ biến

1. Read Uncommitted

* Có thể đọc cả dữ liệu chưa commit của transaction khác.
* Có thể xảy ra dirty read.

2. Read Committed

* Chỉ đọc dữ liệu đã commit.
* Phổ biến ở nhiều hệ quản trị CSDL.

3. Repeatable Read

* Trong cùng 1 transaction, đọc nhiều lần cùng 1 bản ghi sẽ ra cùng kết quả.
* Là mặc định của MySQL InnoDB.

4. Serializable

* Mức an toàn cao nhất.
* Transaction gần như chạy tuần tự với nhau.
* Chậm nhất.

Các vấn đề liên quan

Dirty Read

* Đọc phải dữ liệu chưa commit.

Non-repeatable Read

* Cùng 1 query, đọc 2 lần trong transaction nhưng kết quả khác nhau.

Phantom Read

* Cùng 1 điều kiện query nhưng lần sau xuất hiện thêm hoặc mất đi các dòng mới.

⸻

7. Deadlock

* Là tình huống hai hoặc nhiều transaction chờ nhau vô thời hạn.

Ví dụ

* Transaction A khóa bảng / record User, rồi đợi Order
* Transaction B khóa Order, rồi đợi User

=> Không ai chạy tiếp được → Deadlock

Kết quả

* Database sẽ tự phát hiện deadlock
* Sau đó rollback một transaction để giải phóng lock

Cách giảm deadlock

* Luôn lock dữ liệu theo cùng một thứ tự
* Giữ transaction ngắn nhất có thể
* Tránh update nhiều bảng theo thứ tự khác nhau giữa các transaction

⸻

8. Pagination

* Là chia dữ liệu thành từng trang thay vì lấy toàn bộ một lần.

Mục đích

* Nhanh hơn
* Tiết kiệm RAM
* Giảm dữ liệu trả về
* UI dễ nhìn hơn

Ví dụ SQL

SELECT *
FROM users
LIMIT 20 OFFSET 40;

Ý nghĩa:

* LIMIT 20: lấy 20 dòng
* OFFSET 40: bỏ qua 40 dòng đầu

Trong Laravel

User::paginate(20);

⸻

9. Full Text Search

* Là tìm kiếm theo nội dung văn bản.
* Dùng khi cần search text tốt hơn LIKE.

Vì sao không phải lúc nào cũng dùng LIKE

Ví dụ:

WHERE name LIKE '%iphone%'

Cách này có thể chậm khi dữ liệu lớn, đặc biệt nếu có % ở đầu.

Full Text Search giải quyết gì?

* Tìm kiếm nhanh hơn trên dữ liệu text lớn
* Phù hợp cho:
    * tên sản phẩm
    * tiêu đề bài viết
    * mô tả
    * nội dung

Ví dụ

Tìm kiếm:

* "iphone 15 pro max"
* "laravel query optimization"

Một số khả năng của Full Text Search

* Tìm theo từ khóa
* Xếp hạng độ liên quan (relevance)
* Hỗ trợ tìm kiếm gần đúng tốt hơn LIKE

⸻

10. Pre-aggregation

* Là tính toán trước dữ liệu thống kê rồi lưu sẵn lại để đọc nhanh hơn.

Ví dụ

Dashboard cần hiển thị:

* tổng doanh thu theo ngày
* tổng số đơn theo tháng
* số user đăng ký mới theo tuần

Nếu mỗi lần mở dashboard đều chạy:

SUM(...)
COUNT(...)
GROUP BY ...

trên hàng triệu bản ghi thì sẽ chậm.

Cách làm

Tạo bảng thống kê sẵn, ví dụ:

* daily_sales
* monthly_orders
* weekly_new_users

Sau đó dashboard chỉ đọc từ bảng tổng hợp này.

Mục đích

* Tăng tốc độ đọc dữ liệu thống kê
* Giảm tải cho bảng dữ liệu gốc

⸻

11. Monitoring

* Là theo dõi tình trạng Database để phát hiện vấn đề và xử lý sớm.

Theo dõi những gì?

* Slow query
* CPU
* RAM
* Connections
* Lock
* Deadlock
* Query time

Mục đích

* Giám sát sức khỏe Database
* Tìm query chậm
* Phát hiện nghẽn tài nguyên
* Xử lý sự cố sớm
* Tối ưu hiệu năng hệ thống

Ví dụ thực tế

Nếu thấy:

* CPU tăng cao
* số lượng connection tăng bất thường
* slow query log xuất hiện nhiều câu SELECT mất vài giây

=> cần kiểm tra query, index, transaction hoặc tải hệ thống.