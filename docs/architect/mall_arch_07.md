Học tập **không đi đường vòng** 🧭
👉 [Theo dõi公众号](#公众号) và **trả lời “学习路线”** để nhận **lộ trình học riêng cho dự án mall**!

---

# mall tích hợp Elasticsearch – Hiểu đúng ngay từ đầu (Head First Style)

> Nếu MySQL giỏi **lưu trữ & giao dịch**
> thì Elasticsearch giỏi **tìm kiếm & phân tích text**

👉 mall **KHÔNG dùng Elasticsearch thay MySQL**
👉 mà dùng nó như **một SEARCH ENGINE chuyên nghiệp**

---

## 🧠 Bức tranh tổng thể (rất quan trọng)

Trước khi xem code, hãy nhìn **luồng dữ liệu**:

```
MySQL (pms_product, pms_product_attribute, ...)
        |
        |  (import / sync)
        v
Elasticsearch (EsProduct index)
        |
        |  (search)
        v
Frontend / Mobile
```

👉 **MySQL là nguồn dữ liệu gốc (Source of Truth)**
👉 **Elasticsearch chỉ là bản sao phục vụ tìm kiếm**

---

## 1️⃣ Vì sao mall cần Elasticsearch?

Nếu chỉ dùng MySQL:

```sql
LIKE '%iphone%'
```

❌ chậm
❌ không phân tích tiếng Trung
❌ không scoring (độ liên quan)

Elasticsearch thì:

* ✅ full-text search
* ✅ phân词 (IK Analyzer)
* ✅ relevance score
* ✅ filter + sort + aggregation

👉 **Search là thế mạnh tuyệt đối của ES**

---

## 2️⃣ Spring Data Elasticsearch – vì sao dùng?

🧠 Head First so sánh:

| Cách               | Đặc điểm                      |
| ------------------ | ----------------------------- |
| REST thuần         | Phải tự viết DSL JSON         |
| Transport Client   | API dài, phức tạp             |
| **Spring Data ES** | ✔ giống JPA, ✔ ít boilerplate |

👉 mall chọn **Spring Data Elasticsearch** để:

* code gọn
* dễ đọc
* dễ học

---

## 3️⃣ Tư duy thiết kế Document (EsProduct)

### ❓ Vì sao không dùng pms_product trực tiếp?

🧠 Vì **Search cần dữ liệu khác Transaction**

Ví dụ:

* MySQL: normalize (chuẩn hóa)
* ES: denormalize (phi chuẩn hóa)

👉 **EsProduct là một VIEW MODEL cho tìm kiếm**

---

### 🧩 EsProduct = “Sản phẩm để search”

```java
@Document(indexName = "pms", type = "product")
public class EsProduct {
    @Id
    private Long id;
```

🧠 Mapping trong đầu bạn:

| MySQL    | Elasticsearch |
| -------- | ------------- |
| database | index         |
| table    | type          |
| row      | document      |
| column   | field         |

---

### 🔑 Keyword vs Text (rất hay bị nhầm)

```java
@Field(type = FieldType.Keyword)
private String brandName;
```

👉 `Keyword`:

* không phân词
* dùng cho filter / exact match

```java
@Field(analyzer = "ik_max_word", type = FieldType.Text)
private String name;
```

👉 `Text`:

* có phân词
* dùng cho full-text search

🧠 **Quy tắc nhớ nhanh**:

> ID, mã, tên hãng → `Keyword`
> Tên sản phẩm, mô tả → `Text + analyzer`

---

### 🧠 attrValueList dùng Nested – vì sao?

```java
@Field(type = FieldType.Nested)
private List<EsProductAttributeValue> attrValueList;
```

👉 Vì:

* mỗi sản phẩm có nhiều thuộc tính
* mỗi thuộc tính có key + value

Nested giúp:

* search chính xác theo cặp key–value
* tránh sai logic khi filter

---

## 4️⃣ Repository – Search không cần viết SQL

```java
Page<EsProduct> findByNameOrSubTitleOrKeywords(...)
```

🧠 Head First giải thích:

* Spring **đọc tên method**
* → tự sinh DSL Elasticsearch
* → không cần implement

👉 **Giống JPA**, nhưng chạy trên ES.

---

## 5️⃣ Service – tách rõ trách nhiệm

### 1️⃣ importAll()

```java
productDao.getAllEsProductList(null);
productRepository.saveAll(...)
```

🧠 Ý nghĩa:

* Lấy dữ liệu từ MySQL
* Build EsProduct
* Đẩy sang ES

👉 Thường dùng khi:

* lần đầu chạy hệ thống
* rebuild index

---

### 2️⃣ create(id)

👉 Khi admin **tạo / update sản phẩm**

Flow:

```
MySQL save
→ build EsProduct
→ save vào ES
```

👉 Đảm bảo **search data luôn mới**

---

### 3️⃣ delete(id / ids)

👉 Khi sản phẩm bị xóa / off sale
→ phải xóa khỏi ES
→ tránh search ra “hàng ma”

---

### 4️⃣ search(keyword)

```java
findByNameOrSubTitleOrKeywords(keyword, keyword, keyword)
```

🧠 Đây là:

* **simple search**
* demo cho người mới

👉 Sau này có thể mở rộng:

* filter theo brand
* filter theo price
* sort theo sale / score

---

## 6️⃣ Controller – API rất “đúng vai”

Controller **KHÔNG biết ES hoạt động thế nào**

Nó chỉ:

* gọi service
* trả CommonResult

👉 Kiến trúc **rất sạch**

---

## 7️⃣ Test – thứ tự bắt buộc

🧠 Rất nhiều người test sai thứ tự:

❌ search trước khi import
→ ES rỗng → không có kết quả

### Đúng thứ tự:

1️⃣ `/importAll`
2️⃣ `/search/simple?keyword=xxx`

👉 Kết quả mới đúng

---

## 8️⃣ Tổng kết Head First (phần này cực quan trọng)

### 🎯 mall dùng Elasticsearch đúng chỗ

* ❌ không thay MySQL
* ✅ chỉ phục vụ search

---

### 🎯 Kiến trúc chuẩn enterprise

* MySQL → source of truth
* ES → read model
* Service → đồng bộ dữ liệu

---

### 🎯 Tư duy bạn cần nhớ

> **Elasticsearch không khó**
> Cái khó là **biết dùng nó ở đâu, và dùng tới mức nào**
