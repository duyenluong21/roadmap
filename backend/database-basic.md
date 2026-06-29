1. SQL basic?

- (Structured Query Language) là ngôn ngữ làm việc với CSDL
- Dùng để

* Tạo database
* Tạo bảng
* Thêm / sửa / xóa / truy vấn dữ liệu

- Các câu lệnh cơ bản

* Select # Lấy dữ liệu
* Insert # Thêm dữ liệu
* Update # Cập nhật dữ liệu
* Delete # Xóa dữ liệu
* Create table # Tạo bảng
* Alter table # Thay đổi cấu trúc bảng
* Drop table # Xóa bảng

2. Join

- Phép kết hợp dữ liệu từ bảng thông qua khóa liên kết
- Dùng để: Lấy dữ liệu nằm ở nhiều bảng trong 1 câu SQL
- Các loại join

* Inner join # Chỉ lấy dữ liệu khớp ở 2 bảng
* Left join # Lấy toàn bộ bảng trái và dữ liệu khớp ở bảng phải
* Right join # Lấy toàn bộ bảng phải và dữ liệu khớp ở bảng trái
* Cross join # Kết hợp mọi dòng của 2 bảng

3. Aggregate

- Là các hàm dùng để tính toán trên nhiều dòng dữ liệu
- Các hàm phổ biến:

* Count() # Đếm số dòng
* Sum() # Tính tổng
* Avg() # Tính trung bình
* Min() # Giá trị nhỏ nhất
* Max() # Giá trị lớn nhất

- Dùng để thống kê và tính toán dữ liệu

4. Database design

- Là quá trình thiết kế cấu trúc dữ liệu
- Dùng để:

* Dữ liệu ít bị trùng lặp
* Dễ mở rộng
* Dễ bảo trì
* Truy vấn hiệu quả

5. Migration

- Là cách quản lý việc thay đổi cấu trúc database bằng code
- Dùng để

* Tạo bảng
* Thêm cột
* Xóa cột
* Đồng bộ database giữa các môi trường

6. Constraint

- Là các quy tắc ràng buộc dữ liệu trong database

* Khóa chính (Primary key)
* Khóa ngoại (Foreign key)
* Unique (Không cho phép trùng lặp dữ liệu)
* Not null (Không được để trống)
* Check (Kiểm tra điều kiện dữ liệu)
* Default (Gán giá trị mặc định)

7. Index

- Là cấu trúc dữ liệu giúp database tìm kiếm nhanh hơn
- Dùng để tăng tốc

* Where
* Join
* Order by
* Group by
  VD:
  CREATE INDEX idx_name
  ON users(name);

- Ưu điểm:

* Select nhanh

- Nhược điểm:

* Insert chậm hơn
* Update chậm hơn
* Tốn bộ nhớ

8. N+1 problem

- Là tình trạng ứng dụng thực hiện quá nhiều câu SQL không cần thiết
- Các khắc phục:

* JOIN
* Eager Loading (Laravel with())
* Batch Query

ĐẶC BIỆT INDEX

- Index là một cấu trúc dữ liệu được tạo trên một hoặc nhiều cột của bảng để giúp Database tìm kiếm dữ liệu nhanh hơn mà không cần quét toàn bộ bảng
- Dùng cấu trúc B-Tree

1. Có những loại index nào
   a. MySQL

- Primary Index => Tự tạo khi khai báo Primary key
- Unique Index => Đảm bảo không có giá trị trùng
  VD: Create unique INDEX idx_email on users(email);
  => Dùng cho email, username, số điện thoại
- Normal Index => Index thông thường
  VD: Create INDEX idx_name on users(name)
  => Dùng để tăng tốc độ truy vấn
- Composite Index => Index nhiều cột
  => Hiệu quả khi truy vấn theo đúng thứ tự cột
  VD: CREATE INDEX idx_user
  ON users(last_name, first_name);
- Fulltext Index: Dùng để tìm kiếm văn bản
  => Không dùng cho điều kiện “=” thông thường
  VD: MATCH(content)
  AGAINST('mysql')
- Spatial Index: Cho dữ liệu tọa độ

b. PostgreSQL

- B-Tree:

* Cuốn từ điển sắp xếp dữ liệu theo thứ tự và tạo thành 1 cây để tìm kiếm rất nhanh
* Dùng khi >, <, BETWEEN, ORDER BY, GROUP BY (mặc định, dùng nhiều nhất)

- Hash index:

* Danh bạ tra bằng mã băm
* Chỉ truy vấn bằng =

- GiST

* Bản đồ biết khoảng cách và vị trí
* Dùng khi: GPS, hình học, dữ liệu không gian

- SP-GiST

* Bản đồ được chia thành nhiều khu vực
* Dùng khi: Dữ liệu không gian rất lớn, có thể phân vùng

- GIN:

* Mục lục từ khóa ở cuối sách
* Dùng khi: Full text search, JSONB, Array, tìm kiếm nhiều giá trị

- BRIN: Dùng cho bảng cực lớn àm dữ liệu có tính liên tục

* log
* timestampTree
* sensor
  => BRIN rất nhỏ, tiết kiệm dung lượng nhưng không chính xác bằng B-Tree

FULLTEXT SEARCH

- Tìm kiếm theo nội dung văn bản thay vì tìm kiếm giá trị chính xác
- Database sẽ tạo mục lục từ khóa để biết mỗi từ xuất hiện ở bản ghi nào, giúp tìm kiếm rất nhanh trên dữ liệu lớn
