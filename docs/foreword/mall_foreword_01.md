# 🧭 Mall – Tổng quan kiến trúc & chức năng

> Trước khi đọc code, **hãy nhìn bản đồ**.
> Phần này giúp bạn hiểu: **mall là gì – gồm những gì – tại sao nó được thiết kế như vậy**.

---

## 🏬 Mall là dự án gì?

**mall** là một **hệ thống thương mại điện tử hoàn chỉnh**, gồm **2 phần lớn**:

### 🛒 1. Hệ thống FRONTEND (dành cho người dùng)

Đây là phần **khách hàng sử dụng hằng ngày**, bao gồm:

* Trang chủ
* Gợi ý sản phẩm
* Tìm kiếm sản phẩm
* Xem chi tiết sản phẩm
* Giỏ hàng
* Quy trình đặt hàng
* Trung tâm thành viên
* Chăm sóc khách hàng
* Trung tâm trợ giúp

👉 Nói đơn giản:

> **Tất cả những gì bạn thấy khi mua hàng online**.

---

### 🧑‍💼 2. Hệ thống BACKEND ADMIN (dành cho quản trị)

Đây là **hệ thống quản lý phía sau**, bao gồm:

* Quản lý sản phẩm
* Quản lý đơn hàng
* Quản lý thành viên
* Quản lý khuyến mãi
* Quản lý vận hành
* Quản lý nội dung
* Thống kê – báo cáo
* Quản lý tài chính
* Phân quyền & bảo mật
* Cấu hình hệ thống

👉 Đây chính là **phần bạn – một Java Backend Developer – sẽ làm việc nhiều nhất**.

---

## ▶️ Demo hệ thống (rất nên xem)

> Xem demo giúp bạn **hiểu chức năng trước khi đọc code**.

* 🔧 Backend Admin:
  [http://www.macrozheng.com/admin/index.html](http://www.macrozheng.com/admin/index.html)

* 📱 Frontend (Mobile):
  [http://www.macrozheng.com/app/index.html](http://www.macrozheng.com/app/index.html)

👉 Khuyên thật:

> **Mở demo – bấm thử vài chức năng – rồi quay lại học**.
> Bạn sẽ hiểu code nhanh hơn rất nhiều.

---

## 🧰 Công nghệ được sử dụng trong mall

> mall **không phải demo nhỏ**, mà là **stack công nghệ chuẩn doanh nghiệp**.

| Công nghệ         | Phiên bản | Dùng để làm gì          |
| ----------------- | --------- | ----------------------- |
| Spring Boot       | 2.3.0     | Khung xương backend     |
| Spring Security   | 5.1.4     | Đăng nhập & phân quyền  |
| MyBatis           | 3.4.6     | ORM – thao tác DB       |
| MyBatis Generator | 1.3.3     | Sinh code DAO           |
| PageHelper        | 5.1.8     | Phân trang              |
| Swagger-UI        | 2.9.2     | Tài liệu API            |
| Elasticsearch     | 7.6.2     | Tìm kiếm sản phẩm       |
| RabbitMQ          | 3.7.14    | Message Queue           |
| Redis             | 5.0       | Cache                   |
| MongoDB           | 4.2.5     | NoSQL                   |
| Docker            | 18.09.0   | Container hóa           |
| Druid             | 1.1.10    | Connection Pool         |
| OSS               | 2.5.0     | Lưu file                |
| JWT               | 0.9.0     | Token đăng nhập         |
| Lombok            | 1.18.6    | Giảm code getter/setter |

👉 Nhận xét thẳng:

> **Học xong mall = bạn đã chạm gần hết tech stack backend Java hiện đại**.

---

## 🧩 Các chức năng chính của mall

> Nếu bạn thấy nhiều → đúng rồi, vì **hệ thống thật nó phải như vậy**.

### 🛍️ Module Sản phẩm (PMS)

* Quản lý sản phẩm
* Quản lý danh mục
* Quản lý loại sản phẩm
* Quản lý thương hiệu

---

### 📦 Module Đơn hàng (OMS)

* Quản lý đơn hàng
* Cấu hình đơn hàng
* Xử lý yêu cầu trả hàng
* Thiết lập lý do trả hàng

---

### 🎯 Module Marketing (SMS)

* Quản lý flash sale
* Quản lý giá ưu đãi
* Gợi ý thương hiệu
* Gợi ý sản phẩm mới
* Gợi ý sản phẩm hot
* Quản lý chuyên đề
* Quản lý banner trang chủ

👉 Đây là nơi **nghiệp vụ phức tạp nhất**.

---

## 🗄️ Tổng quan Database của mall

> mall **KHÔNG đơn giản** – hiện tại có **71 bảng dữ liệu**.

![Image](https://www.tutorials24x7.com/sites/default/files/uploads/2020-04-27/files/tutorials24x7-mysql-online-shopping-cart-database-design.png)

![Image](https://databasesample.com/_next/image?q=75\&url=%2Fdatabase%2Fshopping-mall-database.png\&w=3840)

👉 Điều này có nghĩa là:

* Có quan hệ phức tạp
* Có nhiều bảng trung gian
* Rất sát với hệ thống thật

---

### 📌 Quy ước tiền tố bảng

Để **nhìn tên bảng là biết thuộc module nào**:

* `cms_*` → Content (nội dung)
* `oms_*` → Order (đơn hàng)
* `pms_*` → Product (sản phẩm)
* `sms_*` → Sale / Marketing
* `ums_*` → User / Member

👉 Đây là **best practice rất nên học theo**.

---

## 🎯 Bạn nên học phần này như thế nào?

💡 Gợi ý:

1️⃣ Xem **demo hệ thống**
2️⃣ Đọc **tổng quan kiến trúc (bài này)**
3️⃣ Nhớ:

* Có **2 hệ thống**
* Có **nhiều module**
* DB **rất lớn**

👉 Sau đó **mới bắt đầu đọc từng module**.

---

## 📢 公众号 (WeChat Official Account)

Học không đi đường vòng, theo dõi公众号 **macrozheng**
👉 Trả lời **「学习路线」** để nhận **lộ trình học mall chi tiết**

![Image](https://opengraph.githubassets.com/0e4358626612706b3d9867e82818afa40c744572ddb56dcd795566d96379e1ae/macrozheng/mall)

![Image](https://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/banner/qrcode_for_macrozheng_258.jpg)

---

## ✅ Kết luận (rất quan trọng)

> mall **không phải project để copy code**
> mall là **bản đồ giúp bạn hiểu cách xây dựng một hệ thống lớn**
