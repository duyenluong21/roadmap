1. Query Optimize (Tối ưu truy vấn)

- Là quá trình viết hoặc cải thiện câu SQL để chạy nhanh hơn và tốn ít tài nguyên hơn
- Mục đích:

* Giảm thời gian truy vấn
* Giảm CPU, RAM
* Giảm tải database
  hoặc
* Thêm index
* Tránh select \*
* Giảm join không cần thiết

Khi 1 query chậm phải nghĩ theo thứ tự

1. SQL viết đúng không?
2. Có dùng Index chưa?
3. Có lấy quá nhiều dữ liệu không?
4. Join có hợp lý không?
5. Execution Plan nói gì?
6. Có cần cache hoặc pre-aggregation không?

Các cách để tối ưu

1. Chỉ lấy dữ liệu cần thiết
   VD: Chỉ cần lấy ra email thì không cần phải select \*
   Vì sao nhanh hơn?

- đọc ít dữ liệu hơn
- truyền ít dữ liệu hơn
- php xử lý ít dữ liệu hơn

2. Thêm index

- Không có index thì khi tìm kiếm thì sẽ phải tìm kiếm từ trên xuống
  VD: Có 1 triệu bản ghi thì phải quét hết
- Dùng index => tra index => đến đúng record

3. Giảm số lượng join
   => Join quá nhiều bảng sẽ chậm
   Database sẽ phải đọc nhiều index, so sánh nhiều record, dùng nhiều RAM

4. Where càng sớm càng tốt => Database chỉ xử lý dữ liệu cần thiết

5. LIMIT

6. Tránh n+1 query
7. Cache kết quả

8. Execution plan

- Là kế hoạch Database sử dụng để thực hiện 1 câu SQL
  Khi bạn viết một câu SQL:
  SELECT \*
  FROM users
  WHERE email = 'abc@gmail.com';
  Database không chạy ngay.
  Nó sẽ trải qua bước:
  SQL
  ↓
  Parser (kiểm tra cú pháp)
  ↓
  Optimizer (tìm cách chạy nhanh nhất)
  ↓
  Execution Plan
  ↓
  Thực thi
  Nó sẽ quyết định:
- Có dùng index không?
- Quét toàn bộ bảng hay chỉ đọc 1 phần?
- JOIN theo cách nào
- Đọc bảng nào trước
- Ước tính bao nhiêu dòng

`EXPLAIN
SELECT *
FROM users
WHERE email = 'abc@gmail.com';`

id
table
type
possible_keys
key
rows
Extra
1
users
ref
idx_email
idx_email
1
Using index

3. Composite Index

- Index trên nhiều cột
- Ví dụ
  INDEX(first_name,last_name)
  Tốt cho
  WHERE first_name='A'
  và
  WHERE first_name='A'
  AND last_name='B'
  Không tối ưu
  WHERE last_name='B'
  vì nguyên tắc Left-most Prefix.
  Từ khóa nhớ
  Một index cho nhiều cột.

4. Transactions

- Nhóm nhiều câu SQL thành 1 đơn vị công việc
  Hoặc
- Thành công tất cả
- Hoặc rollback tất cả
  VD: A chuyển tiền B nhận tiền
  Acid

* Atomicity
* Consistency
* Isolation
* Durability

5. Locking

- Database khóa dữ liệu để tránh nhiều người sửa cùng lúc => Tránh xung đột

6. Isolation Level

- Quy định Transaction nhìn thấy dữ liệu của transaction khác đến mức nào
  4 mức
- Read committed

* Chỉ đọc dữ liệu đã commit
* Phổ biến

- Repeatable Read

* Đọc nhiều lần luôn giống nhau
* MySQL InnoDB mặc định

- Serializable

* An toàn nhất
* Chậm nhất

7. Deadlock

- Hai transaction chờ nhau
  Ví dụ
  A khóa User
  đợi Order
  Trong khi
  B khóa Order
  đợi User
  Không ai chạy tiếp được.
  => Deadlock.
  Database sẽ rollback một transaction.
  Từ khóa nhớ
  Hai transaction chờ nhau vô thời hạn.

8. Pagination

- Chia dữ liệu thành từng trang
- Mục đích:

* Nhanh hơn
* Tiết kiệm RAM
* UI đẹp hơn

9. Full text search

- TÌm kiếm theo nội dung

10. Pre-aggregation

- Tính toán trước dữ liệu thống kê
  => Lưu trước vào DB để đọc nhanh

10. Monitoring

- Theo dõi tình trạng Database
- Theo dõi:

* Slow query
* CPU
* RAM
* Connections
* Lock
* Deadlock
* Query time
  => Giám sát và xử lý sự cố
