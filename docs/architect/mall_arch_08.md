## 📚 Học tập không đi đường vòng

👉 **[Theo dõi公众号](#公众号)** và **trả lời “学习路线”** để nhận **lộ trình học RIÊNG cho dự án mall**!

---

# 📄 Dự án mall: Tích hợp MongoDB để thao tác **document**

> Bài viết này sẽ **dẫn bạn từng bước** tích hợp **MongoDB vào dự án mall**,
> thông qua một ví dụ **rất đời thực**:
>
> 👉 **Lưu – xóa – truy vấn lịch sử xem sản phẩm của người dùng**

💡 Head First nói thẳng:

> *Dữ liệu ghi nhiều – đọc nhiều – không cần join → MongoDB sinh ra cho việc đó.*

---

## 🧩 1. MongoDB là gì?

> **MongoDB** là **CSDL NoSQL dạng document**,
> được thiết kế cho:
>
> * Hiệu năng đọc/ghi cao
> * Dữ liệu linh hoạt (schema-less)
> * Mở rộng ngang (scale out) tốt

👉 Trong mall:

> *MySQL* → dữ liệu nghiệp vụ chuẩn (order, product…)
> *MongoDB* → dữ liệu hành vi (history, log, click…)

---

## ⚙️ 2. Cài đặt & chạy MongoDB (Windows)

### 📥 Bước 1: Tải MongoDB

🔗 Link tải (v3.2.21 – bản dùng trong demo):
[https://fastdl.mongodb.org/win32/mongodb-win32-x86_64-2008plus-ssl-3.2.21-signed.msi](https://fastdl.mongodb.org/win32/mongodb-win32-x86_64-2008plus-ssl-3.2.21-signed.msi)

---

### 📦 Bước 2: Cài đặt MongoDB

> Chọn thư mục cài đặt và tiến hành cài như phần mềm bình thường.

![Image](https://media.geeksforgeeks.org/wp-content/uploads/20231218135411/Steps-to-install-MongoDB_2.png)

![Image](https://miro.medium.com/1%2AKg8EYalMWwNont39rihcwA.png)

---

### 📂 Bước 3: Tạo thư mục dữ liệu & log

Trong thư mục cài đặt MongoDB, tạo:

```
data\db
data\log
```

![Image](https://www.mongodb.com/community/forums/uploads/default/original/1X/b2c235be81e16580e05104f21bddcd384b60ec71.png)

![Image](https://www.mongodb.com/community/forums/uploads/default/original/1X/287e39d1ec5ae59ab30203555b26ca4ad1be71a8.jpeg)

---

### ⚙️ Bước 4: Tạo file cấu hình `mongod.cfg`

```yml
systemLog:
  destination: file
  path: D:\developer\env\MongoDB\data\log\mongod.log
storage:
  dbPath: D:\developer\env\MongoDB\data\db
```

👉 File này nói cho MongoDB biết:

* Log ghi ở đâu
* Data lưu ở đâu

---

### 🧰 Bước 5: Cài MongoDB thành service

> ⚠️ Chạy CMD **với quyền Administrator**

```bash
D:\developer\env\MongoDB\bin\mongod.exe --config "D:\developer\env\MongoDB\mongod.cfg" --install
```

![Image](https://mkyong.com/wp-content/uploads/2011/04/mongodb-as-windows-service.png)

![Image](https://media.geeksforgeeks.org/wp-content/uploads/20231218135411/Steps-to-install-MongoDB_2.png)

---

### ▶️ Bước 6: Quản lý service MongoDB

```text
Start  : net start MongoDB
Stop   : net stop MongoDB
Remove : mongod.exe --remove
```

💡 Head First nhớ:

> *Chạy dạng service → MongoDB tự khởi động cùng Windows*

---

### 🖥️ Bước 7: Cài client Robo 3T

🔗 Link tải:
[https://download.robomongo.org/1.2.1/windows/robo3t-1.2.1-windows-x86_64-3e50a65.zip](https://download.robomongo.org/1.2.1/windows/robo3t-1.2.1-windows-x86_64-3e50a65.zip)

> Mở `robo3t.exe` → kết nối `localhost:27017`

![Image](https://i.sstatic.net/qg7N6.png)

![Image](https://i.sstatic.net/ggdNm.png)

---

## 🌱 3. Spring Data MongoDB

> **Spring Data MongoDB** cho phép bạn:
>
> * Thao tác MongoDB **giống JPA / Repository**
> * Không cần viết query thủ công
> * Code ngắn – dễ đọc – dễ bảo trì

💡 Head First nhớ:

> *Spring Data = “ít code, nhiều sức mạnh”*

---

### 🏷️ Annotation thường dùng

* `@Document` → ánh xạ class → collection
* `@Id` → khóa chính document
* `@Indexed` → tạo index cho field

---

### 🧠 Repository & Derived Query

> Chỉ cần **đặt tên method đúng quy ước** → Spring tự sinh query.

```java
List<MemberReadHistory>
findByMemberIdOrderByCreateTimeDesc(Long memberId);
```

👉 Không viết 1 dòng query nào 😎

![Image](https://journaldev.nyc3.cdn.digitaloceanspaces.com/2018/01/Spring-Data-MongoDB-MongoRepository-Example-Create.png)

![Image](https://websparrow.org/wp-content/uploads/2020/03/spring-data-jpa-query-annotation-example-1.png)

---

## 🔌 4. Tích hợp MongoDB vào dự án mall

### 📦 Bước 1: Thêm dependency

```xml
<!-- MongoDB -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

---

### ⚙️ Bước 2: Cấu hình `application.yml`

```yml
mongodb:
  host: localhost
  port: 27017
  database: mall-port
```

👉 Thế là xong!
👉 Không cần datasource, không cần pool 😄

---

## 🧱 5. Document: Lịch sử xem sản phẩm

### 📄 MemberReadHistory

> Đây là **document MongoDB**, không phải entity MySQL.

```java
@Document
public class MemberReadHistory {

    @Id
    private String id;

    @Indexed
    private Long memberId;

    @Indexed
    private Long productId;

    private String productName;
    private String productPic;
    private Date createTime;
}
```

💡 Head First nhớ:

> *Field hay query → nhớ đánh @Indexed*

---

## 🗃️ 6. Repository thao tác MongoDB

```java
public interface MemberReadHistoryRepository
        extends MongoRepository<MemberReadHistory, String> {

    List<MemberReadHistory>
    findByMemberIdOrderByCreateTimeDesc(Long memberId);
}
```

👉 Có sẵn:

* save
* delete
* findAll
* findById

---

## 🧠 7. Service xử lý nghiệp vụ

### Interface

```java
int create(MemberReadHistory history);
int delete(List<String> ids);
List<MemberReadHistory> list(Long memberId);
```

---

### Implementation

#### 📝 Tạo lịch sử xem

```java
history.setId(null);
history.setCreateTime(new Date());
repository.save(history);
```

👉 MongoDB **tự sinh id**

---

#### 🗑️ Xóa batch

```java
repository.deleteAll(deleteList);
```

---

#### 📖 Lấy lịch sử xem

```java
return repository
    .findByMemberIdOrderByCreateTimeDesc(memberId);
```

💡 Head First note:

> *MongoDB rất hợp cho dữ liệu dạng timeline*

---

## 🌐 8. Controller – API cho MongoDB

### 📌 API tạo lịch sử xem

```http
POST /member/readHistory/create
```

---

### 📌 API xóa lịch sử xem

```http
POST /member/readHistory/delete
```

---

### 📌 API xem lịch sử

```http
GET /member/readHistory/list?memberId=1
```

---

## 🧪 9. Test API

### ➕ Thêm lịch sử xem

![Image](https://i.sstatic.net/Fv6HZ.png)

![Image](https://miro.medium.com/1%2Af8VkVgPPbodeKDtfObanog.png)

---

### 🔍 Truy vấn lịch sử xem

![Image](https://www.mongodb.com/docs/compass/static/2c8242f03b7aca7a2c24349162e12d2d/a1c7d/query-history-select.webp)

![Image](https://i.sstatic.net/KEGY3.png)

---

## 📦 Source code dự án

🔗 GitHub:
[https://github.com/macrozheng/mall-learning/tree/master/mall-tiny-07](https://github.com/macrozheng/mall-learning/tree/master/mall-tiny-07)

---

## 📢 公众号

![Image](https://opengraph.githubassets.com/0e4358626612706b3d9867e82818afa40c744572ddb56dcd795566d96379e1ae/macrozheng/mall)

![Image](https://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/banner/qrcode_for_macrozheng_258.jpg)

👉 Theo dõi để:

* Hiểu MongoDB **đúng vai trò**
* Kết hợp MongoDB + MySQL hiệu quả
* Không đi đường vòng ❌
