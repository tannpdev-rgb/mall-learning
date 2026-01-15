Học tập **không đi đường vòng** 🚀
👉 [Theo dõi公众号](#公众号) và **trả lời “学习路线”** để nhận **lộ trình học riêng cho dự án mall**!

---

# mall tích hợp Elasticsearch để thực hiện tìm kiếm sản phẩm

> Bài viết này sẽ dẫn bạn từng bước **tích hợp Elasticsearch vào dự án mall**, với mục tiêu:
> 👉 **import – truy vấn – cập nhật – xoá** thông tin sản phẩm trong Elasticsearch.

Hãy tưởng tượng thế này 🧠:
👉 **MySQL** lo lưu trữ dữ liệu
👉 **Elasticsearch** lo **tìm kiếm siêu nhanh**
👉 **mall** kết hợp cả hai để tạo trải nghiệm tìm sản phẩm “nhanh như chớp ⚡”

---

## Giới thiệu framework được sử dụng trong dự án

### Elasticsearch

> **Elasticsearch** là một công cụ **tìm kiếm và phân tích dữ liệu**:

* Phân tán (distributed)
* Có thể mở rộng (scalable)
* Thời gian thực (real-time)

Ngay từ khi dự án bắt đầu, Elasticsearch đã cho phép bạn:

* 🔍 Tìm kiếm toàn văn (full-text search)
* 📊 Thống kê dữ liệu theo thời gian thực

---

### Cài đặt và sử dụng Elasticsearch

#### Bước 1: Tải Elasticsearch

* Tải **Elasticsearch 6.2.2 (zip)** và giải nén
* Link tải:
  [https://www.elastic.co/cn/downloads/past-releases/elasticsearch-6-2-2](https://www.elastic.co/cn/downloads/past-releases/elasticsearch-6-2-2)

![](../images/arch_screen_25.png)

---

#### Bước 2: Cài plugin phân tích tiếng Trung (IK)

Trong thư mục `elasticsearch-6.2.2/bin`, chạy lệnh:

```bash
elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.2.2/elasticsearch-analysis-ik-6.2.2.zip
```

👉 Plugin này giúp Elasticsearch **tách từ tiếng Trung** (giống như tokenizer cho tiếng Việt/Anh).

![](../images/arch_screen_26.png)

---

#### Bước 3: Khởi động Elasticsearch

Chạy file:

```text
bin/elasticsearch.bat
```

![](../images/arch_screen_27.png)

---

#### Bước 4: Cài Kibana (giao diện quản lý ES)

* Tải **Kibana 6.2.2**
* Link tải:
  [https://artifacts.elastic.co/downloads/kibana/kibana-6.2.2-windows-x86_64.zip](https://artifacts.elastic.co/downloads/kibana/kibana-6.2.2-windows-x86_64.zip)

![](../images/arch_screen_28.png)

---

#### Bước 5: Khởi động Kibana

Chạy:

```text
bin/kibana.bat
```

![](../images/arch_screen_29.png)

---

#### Bước 6: Truy cập giao diện Kibana

Mở trình duyệt và truy cập:

```
http://localhost:5601
```

![](../images/arch_screen_30.png)

🎉 Chúc mừng! Bạn đã có **bảng điều khiển Elasticsearch**.

---

## Spring Data Elasticsearch

> **Spring Data Elasticsearch** cho phép bạn thao tác Elasticsearch theo phong cách **Spring Data quen thuộc**, giúp:

* Giảm code lặp
* Không cần viết nhiều boilerplate
* Code gọn – dễ đọc – dễ bảo trì

---

### Các annotation thường dùng

#### `@Document`

👉 Tương đương **database + table** trong MySQL

```java
@Document(
  indexName = "pms", // giống database
  type = "product", // giống table
  shards = 1,
  replicas = 0
)
```

---

#### `@Id`

👉 Chính là **primary key** của document

```java
@Id
private Long id;
```

---

#### `@Field`

👉 Dùng để cấu hình **kiểu dữ liệu & cách lập chỉ mục**

```java
@Field(
  type = FieldType.Text,
  analyzer = "ik_max_word"
)
```

---

#### `FieldType`

Một số kiểu thường dùng:

* `Text` → có phân từ + lập index
* `Keyword` → **không phân từ**
* `Nested` → object lồng nhau
* `Auto` → ES tự đoán kiểu

---

## Thao tác dữ liệu với Spring Data Elasticsearch

### 1️⃣ Kế thừa `ElasticsearchRepository`

👉 Tự động có:

* save
* delete
* findById
* search

![](../images/arch_screen_31.png)

---

### 2️⃣ Derive Query – viết query bằng… tên hàm 😲

```java
Page<EsProduct> findByNameOrSubTitleOrKeywords(
    String name,
    String subTitle,
    String keywords,
    Pageable page
);
```

👉 Không cần SQL
👉 Không cần DSL
👉 Spring tự hiểu!

IDEA còn **auto suggest field** cho bạn nữa 👇

![](../images/arch_screen_32.png)

---

### 3️⃣ Dùng `@Query` để viết DSL

```java
@Query("{"bool":{"must":{"field":{"name":"?0"}}}}")
Page<EsProduct> findByName(String name, Pageable pageable);
```

👉 Khi cần **query phức tạp**, dùng cách này.

---

## Các bảng dữ liệu trong dự án

* `pms_product` – thông tin sản phẩm
* `pms_product_attribute` – thuộc tính sản phẩm
* `pms_product_attribute_value` – giá trị thuộc tính

---

## Tích hợp Elasticsearch để tìm kiếm sản phẩm

### Thêm dependency vào `pom.xml`

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```

---

### Cấu hình `application.yml`

```yml
data:
  elasticsearch:
    repositories:
      enabled: true
    cluster-nodes: 127.0.0.1:9300
    cluster-name: elasticsearch
```

---

### Tạo document `EsProduct`

💡 **Quy tắc vàng**:

* Không cần phân từ → `Keyword`
* Cần tìm kiếm → `Text + ik_max_word`

```java
@Document(indexName = "pms", type = "product", shards = 1, replicas = 0)
public class EsProduct implements Serializable {
    @Id
    private Long id;

    @Field(type = FieldType.Keyword)
    private String productSn;

    @Field(analyzer = "ik_max_word", type = FieldType.Text)
    private String name;

    @Field(type = FieldType.Nested)
    private List<EsProductAttributeValue> attrValueList;
}
```

---

### Repository thao tác Elasticsearch

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

---

### Service & ServiceImpl

👉 Chịu trách nhiệm:

* Import dữ liệu
* Search
* Create / Delete sản phẩm trong ES

(code giữ nguyên như bản gốc)

---

### Controller – định nghĩa API

👉 Các API:

* Import toàn bộ dữ liệu
* Xoá theo ID
* Xoá batch
* Tạo sản phẩm
* Tìm kiếm đơn giản

(code giữ nguyên)

---

## Test API

### Import dữ liệu vào Elasticsearch

![](../images/arch_screen_33.png)
![](../images/arch_screen_34.png)

---

### Tìm kiếm sản phẩm

![](../images/arch_screen_35.png)
![](../images/arch_screen_36.png)

---

## Mã nguồn dự án

👉 [https://github.com/macrozheng/mall-learning/tree/master/mall-tiny-06](https://github.com/macrozheng/mall-learning/tree/master/mall-tiny-06)

---

## 公众号

![公众号图片](http://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/banner/qrcode_for_macrozheng_258.jpg)
