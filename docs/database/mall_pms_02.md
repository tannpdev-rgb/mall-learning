## 1️⃣ Chỉnh sửa sản phẩm (bổ sung ví dụ Head First)

---

### 📦 Bảng sản phẩm (`pms_product`) – ví dụ thực tế

🧠 **Hãy tưởng tượng bạn đang thêm sản phẩm: *iPhone 15 Pro 128GB***

Một dòng dữ liệu trong `pms_product` sẽ trông như sau:

| field                         | giá trị ví dụ             |
| ----------------------------- | ------------------------- |
| id                            | 101                       |
| brand_id                      | 1 (Apple)                 |
| product_category_id           | 2 (Smartphone)            |
| product_attribute_category_id | 1 (Thuộc tính Điện thoại) |
| name                          | iPhone 15 Pro             |
| product_sn                    | IP15PRO-128               |
| price                         | 29990000                  |
| original_price                | 32990000                  |
| promotion_price               | 28990000                  |
| publish_status                | 1 (đang bán)              |
| verify_status                 | 1 (đã duyệt)              |
| stock                         | 500                       |
| low_stock                     | 20                        |
| promotion_type                | 1 (dùng giá khuyến mãi)   |
| brand_name                    | Apple                     |
| product_category_name         | Smartphone                |

👉 **Hiểu theo Head First**:

* `pms_product` = **thông tin chung**
* KHÔNG chứa:

  * Màu
  * Dung lượng
  * SKU chi tiết
    ➡️ Những thứ đó nằm ở **bảng khác**

---

### 🧩 Bảng SKU (`pms_sku_stock`) – ví dụ cực kỳ quan trọng

🧠 **Một SPU → nhiều SKU**

Sản phẩm: **iPhone 15 Pro**

| SKU   | Màu   | Dung lượng |
| ----- | ----- | ---------- |
| SKU-1 | Đen   | 128GB      |
| SKU-2 | Đen   | 256GB      |
| SKU-3 | Trắng | 128GB      |
| SKU-4 | Trắng | 256GB      |

👉 Mỗi dòng = **1 SKU**

| field      | ví dụ             |
| ---------- | ----------------- |
| product_id | 101               |
| sku_code   | IP15PRO-BLACK-128 |
| sp1        | Đen               |
| sp2        | 128GB             |
| price      | 29990000          |
| stock      | 120               |

🧠 **Tư duy chuẩn backend**:

> “Người dùng mua cái gì → trừ tồn kho cái đó”

➡️ Backend **CHỈ trừ tồn kho ở `pms_sku_stock`**, không trừ ở `pms_product`

---

### 📉 Bảng giá bậc thang (`pms_product_ladder`) – ví dụ

🎯 **Chiến lược bán sỉ / combo**

| product_id | count | discount |
| ---------- | ----- | -------- |
| 101        | 2     | 0.9      |
| 101        | 5     | 0.8      |

👉 Nghĩa là:

* Mua **2 cái** → giảm **10%**
* Mua **5 cái** → giảm **20%**

🧠 Frontend sẽ hỏi backend:

> “Mua 3 cái thì áp giá nào?”

➡️ Backend chọn **mức cao nhất thỏa điều kiện**

---

### 💰 Bảng giảm theo số tiền (`pms_product_full_reduction`) – ví dụ

| product_id | full_price | reduce_price |
| ---------- | ---------- | ------------ |
| 101        | 30000000   | 2000000      |

👉 Nếu tổng tiền ≥ 30 triệu
👉 Giảm **2 triệu**

🧠 **Khác ladder price ở chỗ**:

* Ladder → dựa trên **số lượng**
* Full reduction → dựa trên **tổng tiền**

---

### 👑 Bảng giá theo cấp độ thành viên (`pms_member_price`) – ví dụ

| member_level | giá      |
| ------------ | -------- |
| Silver       | 29500000 |
| Gold         | 28500000 |
| Diamond      | 27500000 |

🧠 Khi user checkout:

1. Xác định **member_level**
2. Nếu có giá member → **ưu tiên dùng**
3. Nếu không → fallback về giá thường / khuyến mãi

---

## 2️⃣ Đánh giá sản phẩm & phản hồi – ví dụ Head First

---

### ⭐ Bảng đánh giá (`pms_comment`) – ví dụ

Người dùng A mua:

> iPhone 15 Pro – Đen – 128GB

| field             | ví dụ                   |
| ----------------- | ----------------------- |
| product_id        | 101                     |
| member_nick_name  | nguyenvana              |
| star              | 5                       |
| product_attribute | Màu: Đen; Bộ nhớ: 128GB |
| content           | Máy rất mượt, pin tốt   |
| pics              | img1.jpg,img2.jpg       |
| show_status       | 1                       |

🧠 **Vì sao lưu `product_attribute` dưới dạng text?**

👉 Để:

* Biết user đánh giá **SKU nào**
* Hiển thị đúng thông tin khi đọc review

---

### 💬 Bảng phản hồi (`pms_comment_replay`) – ví dụ

| comment_id | type | content                    |
| ---------- | ---- | -------------------------- |
| 5001       | 0    | Mình cũng thấy pin rất tốt |
| 5001       | 1    | Shop cảm ơn bạn đã ủng hộ  |

🧠 `type` giúp frontend:

* Gắn nhãn **Admin**
* Hiển thị avatar khác

---

## 3️⃣ Duyệt sản phẩm & log thao tác – ví dụ Head First

---

### 📋 Bảng duyệt sản phẩm (`pms_product_vertify_record`)

🧠 **Luồng thực tế**:

1. Nhân viên tạo sản phẩm
2. Trạng thái: `verify_status = 0`
3. Admin duyệt

| product_id | status | detail           |
| ---------- | ------ | ---------------- |
| 101        | 2      | Thông tin hợp lệ |

👉 Nếu bị từ chối:

| status | detail                  |
| ------ | ----------------------- |
| 0      | Thiếu hình ảnh chi tiết |

---

### 📝 Bảng log thao tác (`pms_product_operate_log`) – ví dụ

Admin đổi giá:

| field       | ví dụ    |
| ----------- | -------- |
| product_id  | 101      |
| price_old   | 29990000 |
| price_new   | 28990000 |
| operate_man | admin01  |

🧠 **Tại sao bảng này cực kỳ quan trọng?**

* Truy trách nhiệm
* Xem lịch sử thay đổi
* Phục vụ audit / dispute

👉 **E-commerce lớn bắt buộc phải có**

---

## 🧠 Tổng kết Head First (đọc chậm)

> Nếu bạn nhớ được 3 câu này là đã thắng 70% rồi:

1️⃣ `pms_product` = **thông tin chung (SPU)**
2️⃣ `pms_sku_stock` = **thứ user thực sự mua**
3️⃣ Review, duyệt, log = **bảo vệ hệ thống khi scale**
