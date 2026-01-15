Học tập **không đi đường vòng** 🧭
👉 [Theo dõi公众号](#公众号) và **trả lời “学习路线”** để nhận **lộ trình học riêng cho dự án mall**!

---

# Phân tích bảng cơ sở dữ liệu của module Marketing (Phần 3)

> Bài viết này tập trung phân tích **chức năng đề xuất nội dung trang chủ**,
> trình bày theo cách:
>
> 👉 **Khối nội dung trang chủ ↔ Bảng dữ liệu ↔ Cách hệ thống sử dụng**

🧠 **Head First mindset**
Trang chủ **không phải code cứng** ❌
Trang chủ là **kết quả của cấu hình dữ liệu + logic hiển thị** ✅

---

## Bức tranh tổng thể: Trang chủ được “lắp ráp” thế nào?

Hãy tưởng tượng homepage mobile 👇

1️⃣ Banner (quảng cáo, sự kiện)
2️⃣ Hãng sản xuất trực tiếp
3️⃣ Sản phẩm mới
4️⃣ Sản phẩm hot
5️⃣ Chủ đề nổi bật

👉 Mỗi block = **1 bảng riêng**
👉 Admin bật / tắt / sắp xếp = **không cần deploy code**

---

## 1️⃣ Bảng thương hiệu đề xuất – `sms_home_brand`

> Quản lý **khu vực “Brand Manufacturer Direct”** trên trang chủ

```sql
create table sms_home_brand
(
   id                   bigint not null auto_increment,
   brand_id             bigint comment '商品品牌id',
   brand_name           varchar(64) comment '商品品牌名称',
   recommend_status     int(1) comment '推荐状态',
   sort                 int comment '排序',
   primary key (id)
);
```

### 🧠 Head First: bảng này đại diện cho điều gì?

👉 **Danh sách brand được chọn lọc để show ở homepage**,
KHÔNG phải toàn bộ brand trong hệ thống.

### Ví dụ dữ liệu

| brand   | recommend_status | sort |
| ------- | ---------------- | ---- |
| Apple   | 1                | 1    |
| Samsung | 1                | 2    |
| Xiaomi  | 0                | 0    |

👉 Frontend chỉ query:

```sql
where recommend_status = 1
order by sort
```

🧠 **Tư duy marketing**:
Brand ở đây = **uy tín + bảo chứng chất lượng**

---

## 2️⃣ Bảng sản phẩm mới – `sms_home_new_product`

> Quản lý block **“新鲜好物 – Sản phẩm mới”**

```sql
create table sms_home_new_product
(
   id                   bigint not null auto_increment,
   product_id           bigint,
   product_name         varchar(64),
   recommend_status     int(1),
   sort                 int(1),
   primary key (id)
);
```

### 🧠 Head First: tại sao không dùng `create_time`?

Vì:

* Không phải sản phẩm mới nào cũng muốn show
* Marketing muốn **chủ động chọn**

### Ví dụ

| product   | recommend_status |
| --------- | ---------------- |
| iPhone 15 | 1                |
| iPhone 14 | 0                |

👉 Backend:

```text
Homepage != danh sách mới nhất
Homepage = danh sách được CHỌN
```

---

## 3️⃣ Bảng sản phẩm hot – `sms_home_recommend_product`

> Quản lý block **“人气推荐 – Sản phẩm được yêu thích”**

```sql
create table sms_home_recommend_product
(
   id                   bigint not null auto_increment,
   product_id           bigint,
   product_name         varchar(64),
   recommend_status     int(1),
   sort                 int(1),
   primary key (id)
);
```

### 🧠 Head First: “hot” không nhất thiết là bán nhiều

👉 “Hot” có thể là:

* Bán chạy
* Được marketing đẩy
* Có lợi nhuận cao

🧠 Vì vậy:

> **KHÔNG tính tự động bằng sale_count**

---

### Ví dụ

| product     | lý do         |
| ----------- | ------------- |
| AirPods Pro | lợi nhuận cao |
| iPhone SE   | dễ bán        |

👉 Marketing quyết định, DB chỉ lưu **kết quả chọn**

---

## 4️⃣ Bảng专题精选 – `sms_home_recommend_subject`

> Quản lý block **“专题精选 – Chủ đề đặc biệt”**

```sql
create table sms_home_recommend_subject
(
   id                   bigint not null auto_increment,
   subject_id           bigint,
   subject_name         varchar(64),
   recommend_status     int(1),
   sort                 int,
   primary key (id)
);
```

### 🧠 Head First: Subject là gì?

👉 Subject = **landing page theo chủ đề**

Ví dụ:

* “Top đồ công nghệ cho dân IT”
* “Back to school”
* “Apple week”

🧠 Khi user click:

```text
Homepage → Subject page → Product list
```

---

## 5️⃣ Bảng quảng cáo banner – `sms_home_advertise`

> Quản lý **banner / carousel** trên homepage

```sql
create table sms_home_advertise
(
   id                   bigint not null auto_increment,
   name                 varchar(100),
   type                 int(1),
   pic                  varchar(500),
   start_time           datetime,
   end_time             datetime,
   status               int(1),
   click_count          int,
   order_count          int,
   url                  varchar(500),
   note                 varchar(500),
   sort                 int default 0,
   primary key (id)
);
```

---

### 🧠 Head First: bảng này KHÔNG chỉ để hiển thị

👉 Nó còn dùng để:

* Đếm click
* Đếm đơn hàng
* Đánh giá hiệu quả chiến dịch

---

### Ví dụ banner Flash Sale

| field      | ví dụ            |
| ---------- | ---------------- |
| name       | Flash Sale 9.9   |
| type       | 1 (app)          |
| start_time | 2024-09-09 00:00 |
| end_time   | 2024-09-09 23:59 |
| url        | /flash-sale      |

🧠 Khi user click:

```text
click_count++
```

Khi order thành công từ link:

```text
order_count++
```

👉 **Marketing đo ROI ngay trong DB**

---

## Quản trị phía Admin (hiểu luồng)

### Quản lý brand

![](../images/database_screen_93.png)
![](../images/database_screen_94.png)

### Quản lý sản phẩm mới

![](../images/database_screen_95.png)
![](../images/database_screen_96.png)

### Quản lý sản phẩm hot

![](../images/database_screen_97.png)
![](../images/database_screen_98.png)

### Quản lý chủ đề

![](../images/database_screen_99.png)
![](../images/database_screen_100.png)

### Quản lý banner

![](../images/database_screen_101.png)
![](../images/database_screen_102.png)

---

## Mobile – kết quả cuối cùng

### Banner

![](../images/database_screen_103.png)

### Brand trực tiếp

![](../images/database_screen_104.png)

### Sản phẩm mới

![](../images/database_screen_105.png)

### Sản phẩm hot

![](../images/database_screen_106.png)

### Chủ đề精选

![](../images/database_screen_107.png)

---

## 🧠 Tổng kết Head First (cực kỳ quan trọng)

> Trang chủ **KHÔNG phải dữ liệu động phức tạp**,
> mà là **tập hợp các danh sách được marketing chọn trước**.

### Ghi nhớ 5 dòng này 👇

1️⃣ Homepage = **nhiều block độc lập**
2️⃣ Mỗi block = **1 bảng riêng**
3️⃣ `recommend_status` = công tắc bật/tắt
4️⃣ `sort` = quyền kiểm soát hiển thị
5️⃣ Backend chỉ **assemble dữ liệu**, frontend chỉ render

---

## 公众号

![公众号图片](http://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/banner/qrcode_for_macrozheng_258.jpg)
