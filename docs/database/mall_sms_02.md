Học tập **không đi đường vòng** 🧭
👉 [Theo dõi公众号](#公众号) và **trả lời “学习路线”** để nhận **lộ trình học riêng cho dự án mall**!

---

# Phân tích bảng cơ sở dữ liệu của module Marketing (Phần 2)

> Bài viết này tập trung phân tích **chức năng Coupon (优惠券)**,
> trình bày theo cách:
>
> 👉 **Tình huống người dùng → Luồng hệ thống → Bảng dữ liệu**

🧠 **Head First mindset**
Đừng học coupon như “một tờ giảm giá” ❌
Hãy học coupon như **một quy tắc kinh doanh có trạng thái** ✅

---

## Bức tranh tổng thể: Coupon hoạt động thế nào?

Hãy tưởng tượng tình huống quen thuộc 👇

> 👤 User đăng nhập
> 🎟️ Nhận coupon
> 🛒 Thêm sản phẩm vào giỏ
> 💰 Checkout
> ❓ Hệ thống hỏi:
>
> **“Coupon này có dùng được KHÔNG?”**

Để trả lời câu hỏi đó, backend cần **4 bảng**.

---

## 1️⃣ Bảng Coupon – `sms_coupon`

> Đây là **định nghĩa gốc của coupon**
> 👉 giống như “luật chơi” do admin tạo ra

```sql
create table sms_coupon
(
   id                   bigint not null auto_increment,
   type                 int(1) comment '优惠卷类型',
   name                 varchar(100) comment '名称',
   platform             int(1) comment '使用平台',
   count                int comment '数量',
   amount               decimal(10,2) comment '金额',
   per_limit            int comment '每人限领张数',
   min_point            decimal(10,2) comment '使用门槛',
   start_time           datetime comment '开始使用时间',
   end_time             datetime comment '结束使用时间',
   use_type             int(1) comment '使用类型',
   note                 varchar(200) comment '备注',
   publish_count        int comment '发行数量',
   use_count            int comment '已使用数量',
   receive_count        int comment '领取数量',
   enable_time          datetime comment '可以领取的日期',
   code                 varchar(64) comment '优惠码',
   member_level         int(1) comment '可领取的会员类型',
   primary key (id)
);
```

---

### 🧠 Head First: hãy đọc bảng này như một câu nói

> “Coupon **A**
> giảm **B tiền**,
> áp dụng cho **C đối tượng**,
> dùng trong **D thời gian**,
> với **E điều kiện**.”

---

### Ví dụ 1: Coupon toàn sàn

| field      | giá trị           |
| ---------- | ----------------- |
| name       | Giảm 50k toàn sàn |
| amount     | 50000             |
| min_point  | 500000            |
| use_type   | 0 (toàn sàn)      |
| per_limit  | 1                 |
| start_time | 2024-09-01        |
| end_time   | 2024-09-30        |

👉 Nghĩa là:

* Mua gì cũng được
* Đơn ≥ 500k
* Giảm 50k
* Mỗi người chỉ lấy 1 lần

---

### Ví dụ 2: Coupon chỉ dành cho mobile

| platform | ý nghĩa |
| -------- | ------- |
| 0        | tất cả  |
| 1        | mobile  |
| 2        | PC      |

👉 Backend sẽ check:

```text
request.platform == coupon.platform
```

---

## 2️⃣ Bảng lịch sử Coupon – `sms_coupon_history`

> Đây là **coupon thật của người dùng**,
> không phải coupon mẫu.

```sql
create table sms_coupon_history
(
   id                   bigint not null auto_increment,
   coupon_id            bigint comment '优惠券id',
   member_id            bigint comment '会员id',
   order_id             bigint comment '订单id',
   coupon_code          varchar(64) comment '优惠券码',
   member_nickname      varchar(64) comment '领取人昵称',
   get_type             int(1) comment '获取类型',
   create_time          datetime comment '创建时间',
   use_status           int(1) comment '使用状态',
   use_time             datetime comment '使用时间',
   order_sn             varchar(100) comment '订单号码',
   primary key (id)
);
```

---

### 🧠 Head First: TẠI SAO cần bảng này?

Vì:

* Một coupon có thể:

  * được phát **hàng nghìn bản**
  * mỗi user dùng **1 bản khác nhau**
* Trạng thái của từng bản:

  * chưa dùng
  * đã dùng
  * hết hạn

---

### Ví dụ luồng thực tế

1️⃣ User A bấm **“Nhận coupon”**
→ insert `sms_coupon_history`

| coupon_id | member_id | use_status |
| --------- | --------- | ---------- |
| 10        | 1001      | 0          |

2️⃣ User checkout thành công
→ update:

| use_status | order_id |
| ---------- | -------- |
| 1          | 50001    |

3️⃣ Hết hạn
→ cron job update:

```text
use_status = 2
```

---

## 3️⃣ Coupon – Sản phẩm

`sms_coupon_product_relation`

> Chỉ dùng khi:
>
> ```text
> use_type = 2 (指定商品)
> ```

```sql
create table sms_coupon_product_relation
(
   id                   bigint not null auto_increment,
   coupon_id            bigint,
   product_id           bigint,
   product_name         varchar(500),
   product_sn           varchar(200),
   primary key (id)
);
```

---

### 🧠 Ví dụ

Coupon: **“Giảm 100k cho iPhone”**

| coupon_id | product_id          |
| --------- | ------------------- |
| 20        | 101 (iPhone 15)     |
| 20        | 102 (iPhone 15 Pro) |

👉 Backend khi checkout sẽ check:

> “Giỏ hàng có **ít nhất 1 sản phẩm** thuộc danh sách này không?”

---

## 4️⃣ Coupon – Danh mục

`sms_coupon_product_category_relation`

> Chỉ dùng khi:
>
> ```text
> use_type = 1 (指定分类)
> ```

```sql
create table sms_coupon_product_category_relation
(
   id                   bigint not null auto_increment,
   coupon_id            bigint,
   product_category_id  bigint,
   product_category_name varchar(200),
   parent_category_name varchar(200),
   primary key (id)
);
```

---

### 🧠 Ví dụ

Coupon: **“Giảm 10% cho Laptop”**

| coupon_id | category      |
| --------- | ------------- |
| 30        | Laptop        |
| 30        | Laptop Gaming |

👉 Chỉ cần:

* Giỏ hàng có **1 sản phẩm thuộc danh mục này**
* Là coupon hợp lệ

---

## 🧠 Flow kiểm tra coupon khi checkout (rất quan trọng)

Backend thường làm theo thứ tự:

1️⃣ Coupon có tồn tại không?
2️⃣ Có nằm trong thời gian sử dụng không?
3️⃣ User đã dùng chưa?
4️⃣ Đơn hàng đạt `min_point` chưa?
5️⃣ `use_type` là gì?

* 0 → OK luôn
* 1 → check danh mục
* 2 → check sản phẩm
  6️⃣ Coupon có áp dụng đúng platform không?

👉 Chỉ cần **fail 1 bước → coupon không hợp lệ**

---

## Quản lý phía Admin

### Danh sách coupon

![](../images/database_screen_84.png)

### Tạo / sửa coupon

#### Toàn sàn

![](../images/database_screen_85.png)

#### Chỉ sản phẩm

![](../images/database_screen_86.png)

#### Chỉ danh mục

![](../images/database_screen_87.png)

### Xem chi tiết coupon

![](../images/database_screen_88.png)

---

## Mobile – góc nhìn người dùng

### Coupon của tôi

* Chưa dùng
  ![](../images/database_screen_89.png)

* Đã dùng
  ![](../images/database_screen_90.png)

* Hết hạn
  ![](../images/database_screen_91.png)

### Chi tiết coupon

![](../images/database_screen_92.png)

---

## 🧠 Tổng kết Head First (rất đáng nhớ)

> Nếu bạn nhớ được 4 câu này, bạn đã hiểu coupon ở mức **backend chuẩn**:

1️⃣ `sms_coupon` = **luật coupon**
2️⃣ `sms_coupon_history` = **coupon thật của user**
3️⃣ relation = **phạm vi áp dụng**
4️⃣ Checkout = **chuỗi check logic, không phải if-else đơn giản**

---

## 公众号

![公众号图片](http://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/banner/qrcode_for_macrozheng_258.jpg)
