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
> Cách trình bày sẽ theo kiểu:
> 👉 **chức năng ↔ cấu trúc bảng dữ liệu**
>
> ⚠️ Lưu ý:
> Chỉ những **trường quan trọng cần hiểu** mới được giải thích.
> Các trường đơn giản, bạn hãy **tự đối chiếu với comment trong bảng**.

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
   nav_status           int(1) comment 'Hiển thị trên thanh điều hướng: 0->không; 1->có',
   show_status          int(1) comment 'Trạng thái hiển thị: 0->ẩn; 1->hiện',
   sort                 int comment 'Thứ tự sắp xếp',
   icon                 varchar(255) comment 'Icon',
   keywords             varchar(255) comment 'Từ khóa',
   description          text comment 'Mô tả',
   primary key (id)
);
```

🧠 **Tư duy Head First**:
Hãy hình dung đây là **một cây danh mục** 🌳

* `parent_id = 0` → danh mục gốc
* Các danh mục con trỏ ngược về danh mục cha
* `level` giúp frontend biết đang ở tầng nào

---

### Hiển thị trên trang quản trị

* Danh sách phân loại sản phẩm
  ![](../images/database_screen_02.png)

* Thêm phân loại sản phẩm
  ![](../images/database_screen_01.png)

---

### Hiển thị trên mobile

![](../images/database_screen_03.png)

👉 Cùng một bảng, nhưng **UI khác nhau** tùy nền tảng.

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
   factory_status       int(1) comment 'Có phải nhà sản xuất không: 0->không; 1->có',
   show_status          int(1) comment 'Có hiển thị không',
   product_count        int comment 'Số lượng sản phẩm',
   product_comment_count int comment 'Số lượng bình luận sản phẩm',
   logo                 varchar(255) comment 'Logo thương hiệu',
   big_pic              varchar(255) comment 'Ảnh lớn khu vực thương hiệu',
   brand_story          text comment 'Câu chuyện thương hiệu',
   primary key (id)
);
```

🧠 **Hiểu nhanh**:

* `factory_status` → dùng để phân biệt **hãng sản xuất** và **nhãn hiệu phân phối**
* `first_letter` → dùng cho **sắp xếp A–Z** trên mobile

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

> **Loại sản phẩm = thuộc tính sản phẩm**
>
> Gồm 2 nhóm chính:
>
> * **规格 (Specification – Quy cách)** → người dùng chọn khi mua (màu, size…)
> * **参数 (Parameter – Tham số)** → mô tả sản phẩm, dùng để lọc & tìm kiếm

---

### Cấu trúc bảng liên quan

#### Bảng phân loại thuộc tính sản phẩm

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

👉 Dùng để **gom nhóm thuộc tính** (ví dụ: Điện thoại, Laptop…)

---

#### Bảng thuộc tính sản phẩm

> Trường `type` quyết định đây là **quy cách** hay **tham số**

```sql
create table pms_product_attribute
(
   id                   bigint not null auto_increment,
   product_attribute_category_id bigint comment 'ID nhóm thuộc tính',
   name                 varchar(64) comment 'Tên thuộc tính',
   select_type          int(1) comment 'Cách chọn: 0->duy nhất; 1->đơn chọn; 2->đa chọn',
   input_type           int(1) comment 'Cách nhập: 0->nhập tay; 1->chọn từ danh sách',
   input_list           varchar(255) comment 'Danh sách giá trị, cách nhau bằng dấu phẩy',
   sort                 int comment 'Thứ tự (cao nhất có thể upload ảnh)',
   filter_type          int(1) comment 'Kiểu lọc: 0->thường; 1->màu sắc',
   search_type          int(1) comment 'Kiểu tìm kiếm: 0->không; 1->keyword; 2->range',
   related_status       int(1) comment 'Sản phẩm cùng thuộc tính có liên kết không',
   hand_add_status      int(1) comment 'Có cho thêm thủ công không',
   type                 int(1) comment '0->quy cách; 1->tham số',
   primary key (id)
);
```

🧠 **Cách nhớ nhanh**:

* **规格 (type=0)** → tạo **SKU**
* **参数 (type=1)** → hiển thị & lọc

---

#### Bảng giá trị thuộc tính sản phẩm

> Tùy từng trường hợp mà bảng này lưu:
>
> * Giá trị **quy cách thêm thủ công**
> * Hoặc **giá trị tham số**

```sql
create table pms_product_attribute_value
(
   id                   bigint not null auto_increment,
   product_id           bigint comment 'ID sản phẩm',
   product_attribute_id bigint comment 'ID thuộc tính',
   value                varchar(64) comment 'Giá trị (quy cách nhiều giá trị cách nhau bằng dấu phẩy)',
   primary key (id)
);
```

---

#### Bảng quan hệ giữa danh mục và thuộc tính

> Dùng để **tạo bộ lọc khi tìm kiếm theo danh mục**

```sql
create table pms_product_category_attribute_relation
(
   id                   bigint not null auto_increment,
   product_category_id  bigint comment 'ID danh mục',
   product_attribute_id bigint comment 'ID thuộc tính',
   primary key (id)
);
```

👉 Đây chính là thứ giúp:

> “Chọn danh mục → hiện bộ lọc phù hợp”

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

* Khi thêm sản phẩm, chọn nhóm thuộc tính → hiển thị quy cách để tạo SKU
  ![](../images/database_screen_12.png)

* Khi thêm sản phẩm, hiển thị tham số để nhập
  ![](../images/database_screen_13.png)

---

### Hiển thị trên mobile

* Chọn quy cách sản phẩm
  ![](../images/database_screen_14.png)

* Xem tham số sản phẩm
  ![](../images/database_screen_15.png)

* Lọc sản phẩm khi tìm kiếm theo danh mục
  ![](../images/database_screen_16.png)

---

## 公众号

![公众号图片](http://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/banner/qrcode_for_macrozheng_258.jpg)
