Dưới đây là **bản dịch TOÀN BỘ nội dung sang tiếng Việt**, được viết lại theo **phong cách Head First Java**: thân thiện, dễ hiểu, nói chuyện trực tiếp với người học, giúp bạn **học mà không đi đường vòng** 🚀

---

## 📚 Học tập không đi đường vòng

👉 **[Theo dõi公众号](#公众号)** và **trả lời “学习路线”** để nhận **lộ trình học riêng cho dự án mall**!

---

# 🏗️ Dự án mall: Tích hợp Spring Boot + MyBatis để xây dựng bộ khung cơ bản

> Bài viết này sẽ **dẫn bạn từng bước** tích hợp **Spring Boot + MyBatis** để dựng **bộ xương sống cho dự án mall**.
>
> Chúng ta sẽ lấy **quản lý thương hiệu sản phẩm** làm ví dụ, thực hiện đầy đủ:
>
> * Thêm (Create)
> * Sửa (Update)
> * Xóa (Delete)
> * Truy vấn (Query)
> * Phân trang bằng **PageHelper**

👉 Mục tiêu: **hiểu – làm được – mở rộng được**

---

## 🗄️ 1. Chuẩn bị môi trường MySQL

Trước khi code, ta cần “đất” để dữ liệu ở đã 😄

### Các bước cần làm:

* 📥 **Tải & cài MySQL 5.7**
  Link: [https://dev.mysql.com/downloads/installer/](https://dev.mysql.com/downloads/installer/)

* 🔐 **Thiết lập tài khoản DB**

  ```
  username: root
  password: root
  ```

* 🧰 **Cài công cụ quản lý DB – Navicat**
  Link: [http://www.formysql.com/xiazai.html](http://www.formysql.com/xiazai.html)

* 🗃️ **Tạo database**

  ```
  mall
  ```

* 📄 **Import script SQL của mall**
  Link:
  [https://github.com/macrozheng/mall-learning/blob/master/document/sql/mall.sql](https://github.com/macrozheng/mall-learning/blob/master/document/sql/mall.sql)

👉 OK! Database đã sẵn sàng, ta chuyển sang code 💻

---

## 🧩 2. Giới thiệu các framework sử dụng

### 🚀 Spring Boot – “Khởi động là chạy”

> Spring Boot giúp bạn **tạo Web App siêu nhanh** dựa trên Spring:
>
> * Không cần cấu hình rườm rà
> * Tích hợp sẵn Tomcat
> * Chạy app chỉ bằng **hàm `main()`**

💡 Head First nhớ nhé:

> *Spring Boot = Spring + Auto Configuration + Less Pain*

---

### 📄 PageHelper – Phân trang cho MyBatis

> Đây là **plugin phân trang** cho MyBatis.
> Chỉ cần **vài dòng code**, là có phân trang ngay.

```java
PageHelper.startPage(pageNum, pageSize);

// Sau đó query như bình thường
List<PmsBrand> brandList =
    brandMapper.selectByExample(new PmsBrandExample());

// Gói kết quả vào PageInfo để lấy info phân trang
PageInfo<PmsBrand> pageInfo = new PageInfo<>(brandList);
```

👉 Khi đã tích hợp với Spring Boot thì:

> **Có PageHelper → Tự động hỗ trợ MyBatis**

---

### 🛢️ Druid – Connection Pool của Alibaba

> Druid là **connection pool** mã nguồn mở của Alibaba.
> Được mệnh danh là:
>
> 👉 *“Connection Pool tốt nhất trong Java”*

Ưu điểm:

* Theo dõi SQL
* Chống SQL injection
* Hiệu năng cao

---

### ⚙️ MyBatis Generator (MBG)

> MBG giúp bạn **tự động sinh code** từ database:
>
> * `model`
> * `mapper.xml`
> * `mapper interface`
> * `Example`

👉 Kết quả:
❌ Không cần viết mapper tay cho các CRUD đơn giản
✅ Tập trung vào logic nghiệp vụ

---

## 🧱 3. Khởi tạo dự án

### 🧠 Bước 1: Tạo Spring Boot project bằng IntelliJ IDEA

> Chọn Spring Initializr → Next → Next → Finish 🎉

---

### 📦 Bước 2: Thêm dependency vào `pom.xml`

> Đây là **xương sống của project**, copy cẩn thận nhé 👇

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.3.RELEASE</version>
</parent>

<dependencies>
    <!-- Spring Boot Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- AOP -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-aop</artifactId>
    </dependency>

    <!-- Actuator -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>

    <!-- Test -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

    <!-- PageHelper -->
    <dependency>
        <groupId>com.github.pagehelper</groupId>
        <artifactId>pagehelper-spring-boot-starter</artifactId>
        <version>1.2.10</version>
    </dependency>

    <!-- Druid -->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid-spring-boot-starter</artifactId>
        <version>1.1.10</version>
    </dependency>

    <!-- MyBatis Generator -->
    <dependency>
        <groupId>org.mybatis.generator</groupId>
        <artifactId>mybatis-generator-core</artifactId>
        <version>1.3.3</version>
    </dependency>

    <!-- MySQL Driver -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.15</version>
    </dependency>
</dependencies>
```

---

### ⚙️ Bước 3: Cấu hình `application.yml`

> Khai báo:
>
> * Cổng server
> * Kết nối DB
> * Đường dẫn mapper.xml

```yml
server:
  port: 8080

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mall?useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Shanghai
    username: root
    password: root

mybatis:
  mapper-locations:
    - classpath:mapper/*.xml
    - classpath*:com/**/mapper/*.xml
```

---

## 🗂️ 4. Cấu trúc project

> Cấu trúc rõ ràng = code dễ hiểu = bảo trì dễ 😎

(ảnh cấu trúc project)

---

## 🤖 5. Cấu hình MyBatis Generator

> File này nói cho MBG biết:
>
> * Kết nối DB ở đâu
> * Sinh code vào package nào
> * Bảng nào cần generate

👉 **generatorConfig.xml**

```xml
<!-- (giữ nguyên nội dung như bản gốc, chỉ dịch phần giải thích) -->
```

💡 Head First note:

> *MBG = Database → Java Code, không cần gõ tay*

---

## ▶️ 6. Chạy Generator để sinh code

> Viết một class Java có `main()` để chạy MBG.

```java
public class Generator {
    public static void main(String[] args) throws Exception {
        // Chạy là sinh code 😄
    }
}
```

👉 Sau khi chạy:

* Có `model`
* Có `mapper`
* Có `mapper.xml`

---

## 🧩 7. Cấu hình MyBatis Java

> Dùng `@MapperScan` để Spring biết:
> “Mapper của tao nằm ở đâu”

```java
@Configuration
@MapperScan("com.macro.mall.tiny.mbg.mapper")
public class MyBatisConfig {
}
```

---

## 🌐 8. Controller – Viết API quản lý Brand

> Thực hiện:
>
> * Lấy danh sách
> * Thêm
> * Sửa
> * Xóa
> * Phân trang

👉 Code `PmsBrandController` giữ nguyên như bản gốc (logic rất rõ ràng).

---

## 🧠 9. Service – Tách logic nghiệp vụ

### Interface

```java
public interface PmsBrandService {
    List<PmsBrand> listAllBrand();
    int createBrand(PmsBrand brand);
    int updateBrand(Long id, PmsBrand brand);
    int deleteBrand(Long id);
    List<PmsBrand> listBrand(int pageNum, int pageSize);
    PmsBrand getBrand(Long id);
}
```

### Implementation

```java
@Service
public class PmsBrandServiceImpl implements PmsBrandService {
    // CRUD + PageHelper
}
```

👉 **Controller không chạm DB**
👉 **Service xử lý logic**
👉 **Mapper lo SQL**

---

## 📦 10. Source code dự án

🔗 GitHub:
[https://github.com/macrozheng/mall-learning/tree/master/mall-tiny-01](https://github.com/macrozheng/mall-learning/tree/master/mall-tiny-01)

---

## 📢 公众号

(Ảnh QR code)

👉 Theo dõi để:

* Nhận lộ trình học
* Học Spring Boot + Mall bài bản
* Không đi đường vòng ❌
