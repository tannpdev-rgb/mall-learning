## 📚 Học tập không đi đường vòng

👉 **[Theo dõi公众号](#公众号)** và **trả lời “学习路线”** để nhận **lộ trình học RIÊNG cho dự án mall**!

---

# 🔍 Dự án mall: Tích hợp Elasticsearch để tìm kiếm sản phẩm

> Bài viết này sẽ **dẫn bạn từng bước** tích hợp **Elasticsearch vào dự án mall**,
> nhằm thực hiện các chức năng:
>
> * Import dữ liệu sản phẩm vào Elasticsearch
> * Tìm kiếm sản phẩm (full-text search)
> * Thêm / sửa / xóa dữ liệu trong Elasticsearch

💡 Head First nói thẳng:

> *Database để lưu – Elasticsearch để tìm!*

---

## 🧩 1. Elasticsearch là gì?

> **Elasticsearch** là một **công cụ tìm kiếm & phân tích dữ liệu phân tán**,
> có khả năng:
>
> * Tìm kiếm **toàn văn (full-text search)**
> * Truy vấn cực nhanh
> * Phân tích dữ liệu theo thời gian thực

👉 Trong hệ thống mall:

> *MySQL = nguồn dữ liệu gốc*
> *Elasticsearch = công cụ tìm kiếm*

---

## ⚙️ 2. Cài đặt và chạy Elasticsearch

### 📥 Bước 1: Tải Elasticsearch

* Phiên bản sử dụng: **6.2.2**
* Link tải:
  [https://www.elastic.co/cn/downloads/past-releases/elasticsearch-6-2-2](https://www.elastic.co/cn/downloads/past-releases/elasticsearch-6-2-2)

![Image](https://cdn.sanity.io/images/me0ej585/production/df5883cb815395012d333f4415b1f21798493461-1280x800.png)

![Image](https://cdn2.percipio.com/public/b/f880c4e4-d484-4b94-81be-b6dd4240abca/image001.jpg)

---

### 🧩 Bước 2: Cài plugin phân tích tiếng Trung (IK Analyzer)

> Elasticsearch **KHÔNG hỗ trợ phân từ tiếng Trung mặc định**
> → Phải cài thêm plugin

Chạy lệnh trong thư mục `bin`:

```bash
elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.2.2/elasticsearch-analysis-ik-6.2.2.zip
```

![Image](https://opengraph.githubassets.com/efd97e24c04fe30c9e252ea96aca63660f238d73266d4515233202c7d1d6f8f0/infinilabs/analysis-ik)

![Image](https://opengraph.githubassets.com/f67730197c3f2b45a9519bf3fd3d2cdb9665d0a02f1625fbdd0f9ef103c97528/liuxun666/elasticsearch-analysis-ik)

💡 Head First nhớ:

> *Không có phân từ → tìm kiếm tiếng Trung coi như “mù chữ”*

---

### ▶️ Bước 3: Chạy Elasticsearch

```bash
elasticsearch.bat
```

![Image](https://i.sstatic.net/yYgmj.png)

![Image](https://www.exactsoftware.com/docs/DocBinBlob.aspx?ID=%7Be59b1ecb-8ec3-4834-a1bd-b6e789af4a69%7D)

---

### 🖥️ Bước 4: Cài Kibana (UI cho Elasticsearch)

* Phiên bản: **6.2.2**
* Link tải:
  [https://artifacts.elastic.co/downloads/kibana/kibana-6.2.2-windows-x86_64.zip](https://artifacts.elastic.co/downloads/kibana/kibana-6.2.2-windows-x86_64.zip)

![Image](https://vitalflux.com/wp-content/uploads/2018/03/kibana.png)

![Image](https://us1.discourse-cdn.com/elastic/original/3X/0/7/07bc9d2b1c690d576c55ecf5338f1201cb7a3e5a.png)

Chạy:

```bash
kibana.bat
```

Truy cập:

```
http://localhost:5601
```

![Image](https://static-www.elastic.co/v3/assets/bltefdd0b53724fa2ce/bltd01281e2aa656f58/6881472454ac0d3c9890ee66/illustrated-screenshot-hero-dashboards.png)

![Image](https://play.vidyard.com/5veanmC18pMFPpf4RBVvUR.jpg)

---

## 🌱 3. Spring Data Elasticsearch

> **Spring Data Elasticsearch** giúp bạn thao tác Elasticsearch
> **giống hệt JPA / MyBatis Repository**

👉 Ít code hơn
👉 Đọc dễ hơn
👉 Bảo trì sướng hơn 😄

---

### 🏷️ Các annotation quan trọng

#### `@Document` – tương đương bảng trong DB

```java
@Document(indexName = "pms", type = "product")
```

👉 index = database
👉 type = table

---

#### `@Id` – khóa chính

```java
@Id
private Long id;
```

---

#### `@Field` – mapping field

```java
@Field(type = FieldType.Keyword)
private String brandName;
```

👉 `Keyword` → không phân từ
👉 `Text + analyzer` → có phân từ

💡 Head First nhớ:

> *Field nào cần search → Text + analyzer*

---

## 🧠 4. Chiến lược mapping cho sản phẩm

* ❌ Không phân từ: mã SP, tên brand
* ✅ Có phân từ: tên SP, tiêu đề, keyword

```java
@Field(analyzer = "ik_max_word", type = FieldType.Text)
private String name;
```

---

## 🧱 5. EsProduct – document sản phẩm

> Đây là **phiên bản search** của Product
> (KHÔNG phải entity MySQL)

```java
@Document(indexName = "pms", type = "product")
public class EsProduct {
    @Id
    private Long id;

    @Field(type = FieldType.Keyword)
    private String productSn;

    @Field(analyzer = "ik_max_word", type = FieldType.Text)
    private String name;

    @Field(analyzer = "ik_max_word", type = FieldType.Text)
    private String subTitle;

    @Field(analyzer = "ik_max_word", type = FieldType.Text)
    private String keywords;
}
```

💡 Head First note:

> *Elasticsearch document ≠ MySQL entity*

---

## 🗃️ 6. EsProductRepository – thao tác ES

```java
public interface EsProductRepository
        extends ElasticsearchRepository<EsProduct, Long> {

    Page<EsProduct> findByNameOrSubTitleOrKeywords(
        String name,
        String subTitle,
        String keywords,
        Pageable page
    );
}
```

👉 **Derived Query** – không cần viết DSL
👉 IDE tự gợi ý field

![Image](https://developer.okta.com/assets-jekyll/blog/spring-data-elasticsearch/spring-data-collaboration-3a7aa7e4afe3d17ddbb14a785ae9b9dc6e57d44a73be00ae14fe3855d98c37a1.png)

![Image](https://i.sstatic.net/Xnhio.png)

---

## 🧠 7. EsProductService – logic tìm kiếm

### Các chức năng chính:

* Import toàn bộ sản phẩm từ DB
* Thêm / xóa / batch delete
* Tìm kiếm theo keyword

💡 Head First nhớ:

> *DB → ES = import*
> *Search → chỉ hỏi ES*

---

## ⚙️ 8. EsProductServiceImpl – triển khai

### Import toàn bộ dữ liệu

```java
List<EsProduct> list = productDao.getAllEsProductList(null);
productRepository.saveAll(list);
```

👉 1 lần import = ES có dữ liệu để search

---

### Search sản phẩm

```java
return productRepository.findByNameOrSubTitleOrKeywords(
    keyword, keyword, keyword, pageable
);
```

💡 Head First:

> *Search = OR nhiều field → trải nghiệm người dùng tốt hơn*

---

## 🌐 9. EsProductController – API cho search

### Import dữ liệu

```http
POST /esProduct/importAll
```

---

### Search đơn giản

```http
GET /esProduct/search/simple?keyword=iphone
```

---

## 🧪 10. Test API

### Import dữ liệu

![Image](https://i.sstatic.net/KxkK8.png)

![Image](https://i.sstatic.net/2Tupn.png)

---

### Search sản phẩm

![Image](https://www.elastic.co/guide/en/app-search/current/images/app-search/result-settings.png)

![Image](https://images.contentstack.io/v3/assets/bltefdd0b53724fa2ce/blt0dcc1204d3090052/5ed91a7d08d08473f007ab9a/app-search-analytics-dashboard-blog.jpg)

---

## 📦 Source code dự án

🔗 GitHub:
[https://github.com/macrozheng/mall-learning/tree/master/mall-tiny-06](https://github.com/macrozheng/mall-learning/tree/master/mall-tiny-06)

---

## 📢 公众号

![Image](https://opengraph.githubassets.com/0e4358626612706b3d9867e82818afa40c744572ddb56dcd795566d96379e1ae/macrozheng/mall)

![Image](https://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/banner/qrcode_for_macrozheng_258.jpg)

👉 Theo dõi để:

* Hiểu Elasticsearch **từ mapping → search**
* Áp dụng search cho dự án thật
* Không đi đường vòng ❌
