Học tập **không đi đường vòng** 🧭
👉 [Theo dõi公众号](#公众号) và **trả lời “学习路线”** để nhận **lộ trình học riêng cho dự án mall**!

---

# Phân tích bảng cơ sở dữ liệu của module Sản phẩm (Phần 2)

> Tiếp nối bài trước, bài viết này tập trung phân tích **3 khối chức năng lớn**:
>
> 1. Chỉnh sửa sản phẩm
> 2. Đánh giá sản phẩm & phản hồi
> 3. Duyệt sản phẩm & ghi nhận thao tác
>
> Cách tiếp cận vẫn là:
> 👉 **Chức năng ↔ Bảng dữ liệu tương ứng**

---

## 1️⃣ Chỉnh sửa sản phẩm

### Các bảng liên quan

---

### 📦 Bảng sản phẩm (pms_product)

> Thông tin sản phẩm được chia thành **4 nhóm lớn**:
>
> 1. Thông tin cơ bản
> 2. Thông tin khuyến mãi
> 3. Thông tin thuộc tính
> 4. Quan hệ liên kết sản phẩm
>
> 👉 Bảng `pms_product` chính là **“trái tim”** của toàn bộ module sản phẩm.

```sql
create table pms_product
(
   id                   bigint not null auto_increment,
   brand_id             bigint comment 'ID thương hiệu',
   product_category_id  bigint comment 'ID danh mục sản phẩm',
   feight_template_id   bigint comment 'ID mẫu phí vận chuyển',
   product_attribute_category_id bigint comment 'ID nhóm thuộc tính',
   name                 varchar(64) not null comment 'Tên sản phẩm',
   pic                  varchar(255) comment 'Hình đại diện',
   product_sn           varchar(64) not null comment 'Mã sản phẩm',
   delete_status        int(1) comment 'Trạng thái xóa: 0->chưa xóa; 1->đã xóa',
   publish_status       int(1) comment 'Trạng thái bán: 0->ngừng bán; 1->đang bán',
   new_status           int(1) comment 'Sản phẩm mới: 0->không; 1->có',
   recommand_status     int(1) comment 'Trạng thái đề xuất',
   verify_status        int(1) comment 'Trạng thái duyệt',
   sort                 int comment 'Thứ tự sắp xếp',
   sale                 int comment 'Số lượng bán',
   price                decimal(10,2) comment 'Giá bán',
   promotion_price      decimal(10,2) comment 'Giá khuyến mãi',
   gift_growth          int default 0 comment 'Điểm tăng trưởng tặng kèm',
   gift_point           int default 0 comment 'Điểm thưởng',
   use_point_limit      int comment 'Giới hạn điểm được dùng',
   sub_title            varchar(255) comment 'Tiêu đề phụ',
   description          text comment 'Mô tả sản phẩm',
   original_price       decimal(10,2) comment 'Giá gốc',
   stock                int comment 'Tồn kho',
   low_stock            int comment 'Ngưỡng cảnh báo tồn kho',
   unit                 varchar(16) comment 'Đơn vị',
   weight               decimal(10,2) comment 'Trọng lượng (gram)',
   preview_status       int(1) comment 'Sản phẩm xem trước',
   service_ids          varchar(64) comment 'Dịch vụ kèm theo',
   keywords             varchar(255) comment 'Từ khóa',
   note                 varchar(255) comment 'Ghi chú',
   album_pics           varchar(255) comment 'Ảnh album',
   detail_title         varchar(255) comment 'Tiêu đề chi tiết',
   detail_desc          text comment 'Mô tả chi tiết',
   detail_html          text comment 'Chi tiết web',
   detail_mobile_html   text comment 'Chi tiết mobile',
   promotion_start_time datetime comment 'Bắt đầu khuyến mãi',
   promotion_end_time   datetime comment 'Kết thúc khuyến mãi',
   promotion_per_limit  int comment 'Giới hạn mua',
   promotion_type       int(1) comment 'Loại khuyến mãi',
   product_category_name varchar(255) comment 'Tên danh mục',
   brand_name           varchar(255) comment 'Tên thương hiệu',
   primary key (id)
);
```

🧠 **Head First tip**:
Hãy tưởng tượng `pms_product` giống như **hồ sơ gốc của sản phẩm**.
Các bảng khác chỉ là **phần mở rộng** xoay quanh nó.

---

### 🧩 Bảng SKU sản phẩm

> **SKU (Stock Keeping Unit)** = đơn vị tồn kho
> **SPU (Standard Product Unit)** = đơn vị sản phẩm chuẩn
>
> Ví dụ:
>
> * *iPhone XS* → SPU
> * *iPhone XS | 64GB | Bạc | Bản quốc tế* → SKU

```sql
create table pms_sku_stock
(
   id                   bigint not null auto_increment,
   product_id           bigint comment 'ID sản phẩm',
   sku_code             varchar(64) not null comment 'Mã SKU',
   price                decimal(10,2) comment 'Giá',
   stock                int default 0 comment 'Tồn kho',
   low_stock            int comment 'Tồn kho cảnh báo',
   sp1                  varchar(64) comment 'Thuộc tính 1',
   sp2                  varchar(64) comment 'Thuộc tính 2',
   sp3                  varchar(64) comment 'Thuộc tính 3',
   pic                  varchar(255) comment 'Ảnh hiển thị',
   sale                 int comment 'Số lượng bán',
   promotion_price      decimal(10,2) comment 'Giá khuyến mãi SKU',
   lock_stock           int default 0 comment 'Tồn kho bị khóa',
   primary key (id)
);
```

👉 Một sản phẩm (`pms_product`)
👉 có **nhiều SKU** (`pms_sku_stock`)

---

### 📉 Bảng giá bậc thang (Ladder Price)

> Mua càng nhiều → giá càng rẻ
> Ví dụ: mua 2 sản phẩm → giảm 20%

```sql
create table pms_product_ladder
(
   id                   bigint not null auto_increment,
   product_id           bigint comment 'ID sản phẩm',
   count                int comment 'Số lượng đạt',
   discount             decimal(10,2) comment 'Tỷ lệ giảm',
   price                decimal(10,2) comment 'Giá sau giảm',
   primary key (id)
);
```

---

### 💰 Bảng giảm giá theo số tiền (Full Reduction)

> Mua đủ tiền → được giảm
> Ví dụ: mua đủ 1.000.000đ → giảm 100.000đ

```sql
create table pms_product_full_reduction
(
   id                   bigint not null auto_increment,
   product_id           bigint comment 'ID sản phẩm',
   full_price           decimal(10,2) comment 'Số tiền đạt',
   reduce_price         decimal(10,2) comment 'Số tiền giảm',
   primary key (id)
);
```

---

### 👑 Bảng giá theo cấp độ thành viên

> Mỗi cấp độ thành viên → một mức giá khác nhau
> ⚠️ Thiết kế này còn hạn chế, có thể mở rộng theo % hoặc mức giảm linh hoạt hơn.

```sql
create table pms_member_price
(
   id                   bigint not null auto_increment,
   product_id           bigint comment 'ID sản phẩm',
   member_level_id      bigint comment 'ID cấp độ thành viên',
   member_price         decimal(10,2) comment 'Giá thành viên',
   member_level_name    varchar(100) comment 'Tên cấp độ',
   primary key (id)
);
```

---

### Hiển thị trên trang quản trị

#### Nhập thông tin sản phẩm

![](../images/database_screen_22.png)

#### Cấu hình khuyến mãi

![](../images/database_screen_17.png)

##### Khuyến mãi đặc biệt

![](../images/database_screen_18.png)

##### Giá theo thành viên

![](../images/database_screen_19.png)

##### Giá bậc thang

![](../images/database_screen_20.png)

##### Giảm giá theo mức tiền

![](../images/database_screen_21.png)

#### Nhập thuộc tính sản phẩm

![](../images/database_screen_23.png)
![](../images/database_screen_24.png)
![](../images/database_screen_25.png)

#### Chọn sản phẩm liên quan

![](../images/database_screen_26.png)

---

### Hiển thị trên mobile

#### Giới thiệu sản phẩm

![](../images/database_screen_27.png)

#### Chi tiết hình ảnh & nội dung

![](../images/database_screen_28.png)

#### Chuyên đề liên quan

![](../images/database_screen_29.png)

---

## 2️⃣ Đánh giá sản phẩm & phản hồi

### Các bảng liên quan

---

### ⭐ Bảng đánh giá sản phẩm

```sql
create table pms_comment
(
   id                   bigint not null auto_increment,
   product_id           bigint comment 'ID sản phẩm',
   member_nick_name     varchar(255) comment 'Tên người dùng',
   product_name         varchar(255) comment 'Tên sản phẩm',
   star                 int(3) comment 'Số sao (0–5)',
   member_ip            varchar(64) comment 'IP người đánh giá',
   create_time          datetime comment 'Thời gian tạo',
   show_status          int(1) comment 'Có hiển thị hay không',
   product_attribute    varchar(255) comment 'Thuộc tính lúc mua',
   collect_couont       int comment 'Lượt thích',
   read_count           int comment 'Lượt xem',
   content              text comment 'Nội dung',
   pics                 varchar(1000) comment 'Ảnh đính kèm',
   member_icon          varchar(255) comment 'Avatar',
   replay_count         int comment 'Số phản hồi',
   primary key (id)
);
```

---

### 💬 Bảng phản hồi đánh giá

```sql
create table pms_comment_replay
(
   id                   bigint not null auto_increment,
   comment_id           bigint comment 'ID đánh giá',
   member_nick_name     varchar(255) comment 'Tên người phản hồi',
   member_icon          varchar(255) comment 'Avatar',
   content              varchar(1000) comment 'Nội dung',
   create_time          datetime comment 'Thời gian',
   type                 int(1) comment '0->người dùng; 1->admin',
   primary key (id)
);
```

---

### Hiển thị trên mobile

#### Danh sách đánh giá

![](../images/database_screen_30.png)

#### Chi tiết đánh giá

![](../images/database_screen_31.png)

#### Danh sách phản hồi

![](../images/database_screen_32.png)

---

## 3️⃣ Duyệt sản phẩm & ghi nhận thao tác

---

### 📋 Bảng ghi nhận duyệt sản phẩm

```sql
create table pms_product_vertify_record
(
   id                   bigint not null auto_increment,
   product_id           bigint comment 'ID sản phẩm',
   create_time          datetime comment 'Thời gian',
   vertify_man          varchar(64) comment 'Người duyệt',
   status               int(1) comment '0->không duyệt; 2->đã duyệt',
   detail               varchar(255) comment 'Chi tiết phản hồi',
   primary key (id)
);
```

👉 Dùng cho **workflow kiểm duyệt sản phẩm**.

---

### 📝 Bảng log thao tác sản phẩm

```sql
create table pms_product_operate_log
(
   id                   bigint not null auto_increment,
   product_id           bigint comment 'ID sản phẩm',
   price_old            decimal(10,2) comment 'Giá cũ',
   price_new            decimal(10,2) comment 'Giá mới',
   sale_price_old       decimal(10,2) comment 'Giá KM cũ',
   sale_price_new       decimal(10,2) comment 'Giá KM mới',
   gift_point_old       int comment 'Điểm cũ',
   gift_point_new       int comment 'Điểm mới',
   use_point_limit_old  int comment 'Giới hạn điểm cũ',
   use_point_limit_new  int comment 'Giới hạn điểm mới',
   operate_man          varchar(64) comment 'Người thao tác',
   create_time          datetime comment 'Thời gian',
   primary key (id)
);
```

🧠 **Hiểu nhanh**:
Bảng này giống như **audit log** – để:

* Truy vết thay đổi
* Kiểm soát rủi ro
* Debug & kiểm toán

---

## 公众号

![公众号图片](http://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/banner/qrcode_for_macrozheng_258.jpg)

---
