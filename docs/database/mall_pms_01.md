Học tập **không đi đường vòng** 🧭
👉 [Theo dõi公众号](#公众号) và **trả lời “学习路线”** để nhận **lộ trình học riêng cho dự án mall**!

---

# Phân tích bảng cơ sở dữ liệu của module Sản phẩm (Phần 1)

> Bài viết này tập trung phân tích **3 chức năng cốt lõi của module sản phẩm**:
>
> * Phân loại sản phẩm
> * Quản lý thương hiệu
> * Loại sản phẩm (thuộc tính sản phẩm)
>
> Cách trình bày:
> 👉 **chức năng ↔ bảng dữ liệu ↔ ví dụ thực tế**
>
> ⚠️ Chỉ giải thích **trường quan trọng**, các trường đơn giản tự đọc comment SQL.

---

## 1️⃣ Phân loại sản phẩm

### Bảng phân loại sản phẩm

```sql
create table pms_product_category
(
   id                   bigint not null auto_increment,
   parent_id            bigint comment 'ID danh mục cha: 0 nghĩa là danh mục cấp 1',
   name                 varchar(64) comment 'Tên danh mục',
   level                int(1) comment 'Cấp danh mục: 0->cấp 1; 1->cấp 2',
   product_count        int comment 'Số lượng sản phẩm',
   product_unit         varchar(64) comment 'Đơn vị sản phẩm',
   nav_status           int(1) comment 'Hiển thị trên thanh điều hướng',
   show_status          int(1) comment 'Trạng thái hiển thị',
   sort                 int comment 'Thứ tự sắp xếp',
   icon                 varchar(255) comment 'Icon',
   keywords             varchar(255) comment 'Từ khóa',
   description          text comment 'Mô tả',
   primary key (id)
);
```

### 🧠 Tư duy Head First (kèm ví dụ)

Hãy tưởng tượng bạn đang xây **một trung tâm thương mại online** 🏬

#### Ví dụ dữ liệu thật

| id | parent_id | name                 | level |
| -- | --------- | -------------------- | ----- |
| 1  | 0         | Điện thoại           | 0     |
| 2  | 1         | Smartphone           | 1     |
| 3  | 1         | Điện thoại phổ thông | 1     |
| 4  | 0         | Laptop               | 0     |
| 5  | 4         | Laptop gaming        | 1     |

👉 Điều này có nghĩa là:

* `parent_id = 0` → danh mục **gốc**
* `level = 0` → menu chính
* `level = 1` → submenu

📱 **Mobile / Web sẽ dùng bảng này để vẽ menu dạng cây**.

---

### Hiển thị trên trang quản trị

* Danh sách phân loại sản phẩm
  ![](../images/database_screen_02.png)

* Thêm phân loại sản phẩm
  ![](../images/database_screen_01.png)

---

### Hiển thị trên mobile

![](../images/database_screen_03.png)

👉 **Một bảng – nhiều cách hiển thị**
DB không đổi, chỉ khác **frontend**.

---

## 2️⃣ Quản lý thương hiệu

### Bảng thương hiệu sản phẩm

```sql
create table pms_brand
(
   id                   bigint not null auto_increment,
   name                 varchar(64) comment 'Tên thương hiệu',
   first_letter         varchar(8) comment 'Chữ cái đầu',
   sort                 int comment 'Thứ tự sắp xếp',
   factory_status       int(1) comment 'Có phải nhà sản xuất không',
   show_status          int(1) comment 'Có hiển thị không',
   product_count        int comment 'Số lượng sản phẩm',
   product_comment_count int comment 'Số lượng bình luận',
   logo                 varchar(255) comment 'Logo',
   big_pic              varchar(255) comment 'Ảnh banner',
   brand_story          text comment 'Câu chuyện thương hiệu',
   primary key (id)
);
```

### 🧠 Head First: hãy nghĩ như người dùng

#### Ví dụ dữ liệu

| id | name    | first_letter | factory_status |
| -- | ------- | ------------ | -------------- |
| 1  | Apple   | A            | 1              |
| 2  | Samsung | S            | 1              |
| 3  | Baseus  | B            | 0              |

👉 Ý nghĩa:

* `factory_status = 1`
  → **hãng sản xuất gốc** (Apple, Samsung)
* `factory_status = 0`
  → **hãng phụ kiện / phân phối**

📱 Mobile dùng `first_letter` để:

* Hiển thị **A–Z**
* Scroll nhanh theo chữ cái

---

### Hiển thị trên trang quản trị

* Danh sách thương hiệu
  ![](../images/database_screen_04.png)

* Thêm thương hiệu
  ![](../images/database_screen_05.png)

---

### Hiển thị trên mobile

![](../images/database_screen_06.png)

---

## 3️⃣ Loại sản phẩm (Thuộc tính sản phẩm)

> **Loại sản phẩm = Thuộc tính sản phẩm**
>
> Gồm **2 loại khác nhau nhưng rất hay bị nhầm** 👇

---

### 🔹 规格 (Specification – Quy cách)

👉 **Người dùng phải chọn khi mua**
👉 Dùng để **tạo SKU**

**Ví dụ:**

* Màu sắc
* Dung lượng
* Size

---

### 🔹 参数 (Parameter – Tham số)

👉 **Chỉ để xem & lọc**
👉 Không tạo SKU

**Ví dụ:**

* CPU
* RAM
* Hệ điều hành

---

### Bảng phân loại thuộc tính

```sql
create table pms_product_attribute_category
(
   id                   bigint not null auto_increment,
   name                 varchar(64) comment 'Tên',
   attribute_count      int comment 'Số lượng thuộc tính',
   param_count          int comment 'Số lượng tham số',
   primary key (id)
);
```

### 🧠 Ví dụ

| id | name       |
| -- | ---------- |
| 1  | Điện thoại |
| 2  | Laptop     |

👉 Mỗi nhóm sẽ có **bộ thuộc tính riêng**

---

### Bảng thuộc tính sản phẩm

```sql
create table pms_product_attribute
(
   id                   bigint not null auto_increment,
   product_attribute_category_id bigint,
   name                 varchar(64),
   select_type          int(1),
   input_type           int(1),
   input_list           varchar(255),
   sort                 int,
   filter_type          int(1),
   search_type          int(1),
   related_status       int(1),
   hand_add_status      int(1),
   type                 int(1),
   primary key (id)
);
```

### 🧠 Ví dụ cực kỳ quan trọng (đọc chậm)

#### Ví dụ 1: Màu sắc (Specification)

| field       | value          |
| ----------- | -------------- |
| name        | Màu sắc        |
| type        | 0 (规格)         |
| select_type | 1 (đơn chọn)   |
| input_list  | Đen,Trắng,Xanh |

👉 Khi người dùng mua:

* Phải chọn **1 màu**
* Mỗi màu → **SKU khác nhau**

---

#### Ví dụ 2: RAM (Specification)

| field       | value    |
| ----------- | -------- |
| name        | RAM      |
| type        | 0        |
| select_type | 1        |
| input_list  | 8GB,16GB |

👉 Kết hợp với màu → tạo **nhiều SKU**

---

#### Ví dụ 3: CPU (Parameter)

| field       | value       |
| ----------- | ----------- |
| name        | CPU         |
| type        | 1 (参数)      |
| search_type | 1 (keyword) |

👉 Chỉ để:

* Hiển thị chi tiết
* Lọc khi tìm kiếm

---

### Bảng giá trị thuộc tính

```sql
create table pms_product_attribute_value
(
   id                   bigint not null auto_increment,
   product_id           bigint,
   product_attribute_id bigint,
   value                varchar(64),
   primary key (id)
);
```

### 🧠 Ví dụ

Sản phẩm: **iPhone 15**

| product_id | attribute | value |
| ---------- | --------- | ----- |
| 101        | CPU       | A17   |
| 101        | RAM       | 8GB   |
| 101        | Màu       | Đen   |

👉 Bảng này giống như:

> **“Bảng ghi chú thuộc tính của từng sản phẩm”**

---

### Bảng quan hệ danh mục – thuộc tính

```sql
create table pms_product_category_attribute_relation
(
   id                   bigint not null auto_increment,
   product_category_id  bigint,
   product_attribute_id bigint,
   primary key (id)
);
```

### 🧠 Ví dụ

| category   | attribute   |
| ---------- | ----------- |
| Điện thoại | RAM         |
| Điện thoại | CPU         |
| Laptop     | Card đồ họa |

👉 Khi người dùng:

> **Chọn danh mục → hệ thống biết phải hiện bộ lọc nào**

---

### Hiển thị trên trang quản trị

* Danh sách nhóm thuộc tính
  ![](../images/database_screen_07.png)

* Thêm nhóm thuộc tính
  ![](../images/database_screen_08.png)

* Danh sách quy cách
  ![](../images/database_screen_09.png)

* Danh sách tham số
  ![](../images/database_screen_10.png)

* Thêm thuộc tính
  ![](../images/database_screen_11.png)

* Sinh SKU khi chọn thuộc tính
  ![](../images/database_screen_12.png)

* Nhập tham số sản phẩm
  ![](../images/database_screen_13.png)

---

### Hiển thị trên mobile

* Chọn quy cách
  ![](../images/database_screen_14.png)

* Xem tham số
  ![](../images/database_screen_15.png)

* Lọc khi tìm kiếm
  ![](../images/database_screen_16.png)

---

## 公众号

![公众号图片](http://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/banner/qrcode_for_macrozheng_258.jpg)
