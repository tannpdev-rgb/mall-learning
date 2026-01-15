Học tập **không đi đường vòng** 🧭
👉 [Theo dõi公众号](#公众号) và **trả lời “学习路线”** để nhận **lộ trình học riêng cho dự án mall**!

---

# Phân tích bảng cơ sở dữ liệu của module Marketing (Phần 1)

> Bài viết này tập trung phân tích **chức năng Giờ vàng – Flash Sale (秒杀)**,
> bằng cách đối chiếu:
>
> 👉 **chức năng thực tế ↔ bảng dữ liệu tương ứng**

🧠 **Head First mindset**:
Đừng học bảng DB như danh sách cột ❌
Hãy học như **một câu chuyện người dùng đang “săn sale”** ✅

---

## Bức tranh tổng thể: Flash Sale hoạt động thế nào?

Hãy tưởng tượng scenario quen thuộc 👇

> 📅 Từ 01/10 → 07/10
> ⏰ Mỗi ngày có các khung giờ: **10h – 12h**, **20h – 22h**
> 📱 Mỗi khung giờ có **những sản phẩm khác nhau**
> 👤 Mỗi người chỉ được mua **giới hạn số lượng**

➡️ Để làm được chuyện đó, ta cần **4 bảng**.

---

## 1️⃣ Bảng Flash Sale – `sms_flash_promotion`

> Lưu **thông tin tổng của chiến dịch Flash Sale**
> (KHÔNG phải từng khung giờ)

```sql
create table sms_flash_promotion
(
   id                   bigint not null auto_increment,
   title                varchar(200) comment '标题',
   start_date           date comment '开始日期',
   end_date             date comment '结束日期',
   status               int(1) comment '上下线状态',
   create_time          datetime comment '创建时间',
   primary key (id)
);
```

### 🧠 Head First: bảng này đại diện cho CÁI GÌ?

👉 **Một chiến dịch lớn**, ví dụ:

> 🎉 “Flash Sale Quốc Khánh 2/9”

### Ví dụ dữ liệu

| field      | giá trị               |
| ---------- | --------------------- |
| id         | 1                     |
| title      | Flash Sale Quốc Khánh |
| start_date | 2024-09-01            |
| end_date   | 2024-09-07            |
| status     | 1 (đang online)       |

👉 Bảng này **KHÔNG quan tâm**:

* bán lúc mấy giờ
* bán sản phẩm nào

➡️ Những việc đó để bảng khác lo.

---

## 2️⃣ Bảng Flash Sale Session – `sms_flash_promotion_session`

> Lưu **các khung giờ cố định trong ngày**

```sql
create table sms_flash_promotion_session
(
   id                   bigint not null auto_increment comment '编号',
   name                 varchar(200) comment '场次名称',
   start_time           time comment '每日开始时间',
   end_time             time comment '每日结束时间',
   status               int(1) comment '启用状态',
   create_time          datetime comment '创建时间',
   primary key (id)
);
```

### 🧠 Head First: tại sao tách bảng này?

Vì **một khung giờ có thể dùng cho nhiều ngày**.

### Ví dụ dữ liệu

| id | name | start_time | end_time |
| -- | ---- | ---------- | -------- |
| 1  | Sáng | 10:00      | 12:00    |
| 2  | Tối  | 20:00      | 22:00    |

👉 Khi hiển thị mobile:

* “Đang抢购” → giờ hiện tại nằm trong session
* “即将开始” → giờ chưa tới

---

## 3️⃣ Bảng quan hệ Flash Sale – Sản phẩm

`sms_flash_promotion_product_relation`

> Đây là **bảng quan trọng nhất**
> 👉 nơi gắn **SẢN PHẨM + GIỜ + GIÁ FLASH SALE**

```sql
create table sms_flash_promotion_product_relation
(
   id                   bigint not null auto_increment,
   flash_promotion_id   bigint comment '限时购id',
   flash_promotion_session_id bigint comment '编号',
   product_id           bigint comment '商品id',
   flash_promotion_price decimal(10,2) comment '限时购价格',
   flash_promotion_count int comment '限时购数量',
   flash_promotion_limit int comment '每人限购数量',
   sort                 int comment '排序',
   primary key (id)
);
```

### 🧠 Head First: đọc bảng này như một câu nói

> “Trong **Flash Sale A**,
> tại **khung giờ B**,
> bán **sản phẩm C**,
> với **giá D**,
> số lượng **E**,
> mỗi người mua tối đa **F**.”

### Ví dụ dữ liệu

| field                      | ví dụ           |
| -------------------------- | --------------- |
| flash_promotion_id         | 1               |
| flash_promotion_session_id | 2 (20h–22h)     |
| product_id                 | 101 (iPhone 15) |
| flash_promotion_price      | 19990000        |
| flash_promotion_count      | 100             |
| flash_promotion_limit      | 1               |

👉 Nghĩa là:

* Chỉ bán **100 cái**
* Mỗi user mua **tối đa 1**
* Hết là **HẾT THẬT**

---

### ⚠️ Lưu ý QUAN TRỌNG (rất hay sai)

> **Sản phẩm tham gia Flash Sale phải:**

```text
pms_product.promotion_type = 5
```

👉 Vì:

* Hệ thống **tính giá khác**
* Checkout phải **ưu tiên flash_promotion_price**

🧠 Nếu quên bước này → **giá hiển thị sai**

---

## 4️⃣ Bảng log đặt lịch nhắc – `sms_flash_promotion_log`

> Cho phép user **đặt lịch nhắc trước khi sale bắt đầu**

```sql
create table sms_flash_promotion_log
(
   id                   int not null auto_increment,
   member_id            int comment '会员id',
   product_id           bigint comment '商品id',
   member_phone         varchar(64) comment '会员电话',
   product_name         varchar(100) comment '商品名称',
   subscribe_time       datetime comment '会员订阅时间',
   send_time            datetime comment '发送时间',
   primary key (id)
);
```

### 🧠 Head First: luồng người dùng

1. User thấy: **“即将开始”**
2. Bấm: **预约提醒**
3. Hệ thống lưu log
4. Đến giờ → gửi thông báo (SMS / push / email)

### Ví dụ dữ liệu

| member_id | product_id | subscribe_time   |
| --------- | ---------- | ---------------- |
| 1001      | 101        | 2024-09-01 18:30 |

👉 19:55 → hệ thống gửi nhắc
👉 20:00 → bắt đầu抢购

---

## Quản lý phía Admin (hiểu luồng, không học vẹt)

### Flash Sale List

![](../images/database_screen_72.png)

### Tạo / sửa chiến dịch

![](../images/database_screen_73.png)

### Quản lý khung giờ

![](../images/database_screen_74.png)
![](../images/database_screen_75.png)

### Thêm sản phẩm vào Flash Sale

![](../images/database_screen_76.png)
![](../images/database_screen_77.png)
![](../images/database_screen_78.png)

### Chỉnh giá & số lượng

![](../images/database_screen_79.png)

---

## Mobile – đúng trải nghiệm người dùng

### Đã bắt đầu

![](../images/database_screen_80.png)

### Đang抢购

![](../images/database_screen_81.png)

### Sắp bắt đầu

![](../images/database_screen_82.png)

### Đặt lịch nhắc

![](../images/database_screen_83.png)

---

## 🧠 Tổng kết Head First (rất quan trọng)

Nếu bạn nhớ được **4 câu này**, bạn đã hiểu Flash Sale chuẩn backend:

1️⃣ `sms_flash_promotion` = **chiến dịch lớn (theo ngày)**
2️⃣ `sms_flash_promotion_session` = **khung giờ cố định**
3️⃣ `relation` = **giá + số lượng + giới hạn mua**
4️⃣ `log` = **trải nghiệm user & marketing**

---

## 公众号

![公众号图片](http://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/banner/qrcode_for_macrozheng_258.jpg)
