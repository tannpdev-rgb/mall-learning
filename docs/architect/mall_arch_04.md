
## 📚 Học tập không đi đường vòng

👉 **[Theo dõi公众号](#公众号)** và **trả lời “学习路线”** để nhận **lộ trình học RIÊNG cho dự án mall**!

---

# 🔐 Dự án mall: Tích hợp Spring Security + JWT để xác thực & phân quyền (Phần 1)

> Bài viết này sẽ **dẫn bạn từng bước** cách dự án **mall kết hợp Spring Security và JWT**
> để thực hiện:
>
> * ✅ Đăng nhập cho user backend
> * ✅ Xác thực (Authentication)
> * ✅ Phân quyền (Authorization)
> * ✅ Nâng cấp Swagger-UI để **tự động gửi token**

💡 Head First nói thẳng:

> *Không có bảo mật → Backend chỉ là “API công cộng trá hình”* 😅

---

## 🧩 1. Các framework được sử dụng

### 🛡️ Spring Security là gì?

> **Spring Security** là framework **chuẩn mực về bảo mật** cho ứng dụng Spring:
>
> * Xác thực người dùng (Authentication)
> * Kiểm soát quyền truy cập (Authorization)
> * Cấu hình rất mạnh, rất sâu, rất “enterprise”

👉 Với Spring:

> *Muốn làm bảo mật nghiêm túc → Spring Security là con đường chính thống*

---

### 🔑 JWT (JSON Web Token) là gì?

> **JWT** là chuẩn **RFC 7519**, dùng để **truyền thông tin xác thực một cách an toàn**.

Đặc điểm:

* Không cần session
* Gọn nhẹ
* Phù hợp REST API
* Rất hợp với frontend SPA / mobile

---

### 🧠 JWT gồm những phần nào?

#### 🔹 Cấu trúc tổng quát

```
header.payload.signature
```

---

#### 🔹 Header – thuật toán ký

```json
{"alg": "HS512"}
```

👉 Nói cho server biết:

> *Token này được ký bằng thuật toán gì*

---

#### 🔹 Payload – dữ liệu chính

```json
{"sub":"admin","created":1489079981393,"exp":1489684781}
```

Chứa:

* Username
* Thời gian tạo
* Thời gian hết hạn

---

#### 🔹 Signature – chữ ký số

```java
HMACSHA512(
  base64(header) + "." + base64(payload),
  secret
)
```

👉 Chỉ cần **payload bị sửa 1 byte** → token **INVALID**

---

#### 🔹 Ví dụ JWT thực tế

```
eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJhZG1pbiIsImNyZWF0ZWQiOjE1NTY3NzkxMjUzMDksImV4cCI6MTU1NzM4MzkyNX0...
```

Bạn có thể **decode trực tiếp tại**:
👉 [https://jwt.io/](https://jwt.io/)

![Image](https://www.redevtools.com/blog/jwtdecode-how-to-decode-a-jwt-token-from-the-console/decode-a-jwt-token.jpg)

![Image](https://fusionauth.io/img/shared/json-web-token.png)

---

### 🔄 JWT hoạt động thế nào trong hệ thống?

1. User login → backend trả JWT
2. Frontend lưu JWT
3. Mỗi request → gửi JWT trong header
4. Backend:

   * Decode token
   * Verify chữ ký
   * Lấy user & quyền
   * Cho hoặc chặn request

💡 Head First nhớ:

> *JWT = “thẻ căn cước số” của user*

---

### 🧰 Hutool là gì?

> **Hutool** là bộ **utility Java cực kỳ mạnh**:
>
> * JSON
> * Date
> * String
> * IO
>
> 👉 Viết code **ngắn hơn – sạch hơn**

---

## 🗄️ 2. Các bảng dữ liệu liên quan

Hiểu bảng = hiểu phân quyền 👇

* `ums_admin` → user backend
* `ums_role` → vai trò (ADMIN, SALE, …)
* `ums_permission` → quyền cụ thể
* `ums_admin_role_relation` → user ↔ role (N-N)
* `ums_role_permission_relation` → role ↔ permission
* `ums_admin_permission_relation` → quyền cộng / trừ riêng cho user

💡 Head First note:

> *User có quyền = quyền role ± quyền custom*

---

## 🔌 3. Tích hợp Spring Security + JWT

### 📦 Bước 1: Thêm dependency

```xml
<!-- Spring Security -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-security</artifactId>
</dependency>

<!-- Hutool -->
<dependency>
  <groupId>cn.hutool</groupId>
  <artifactId>hutool-all</artifactId>
  <version>4.5.7</version>
</dependency>

<!-- JWT -->
<dependency>
  <groupId>io.jsonwebtoken</groupId>
  <artifactId>jjwt</artifactId>
  <version>0.9.0</version>
</dependency>
```

👉 Sau bước này:

* Project **đã bật chế độ bảo mật**
* Nhưng… **tất cả API sẽ bị khóa** 😅
  → Phải cấu hình tiếp

---

## 🔑 4. JwtTokenUtil – công cụ xử lý JWT

> Đây là **trái tim của JWT**

Nó làm được:

* Sinh token
* Parse token
* Kiểm tra hạn
* Refresh token

💡 Head First hiểu đơn giản:

> *JwtTokenUtil = nhà máy sản xuất & kiểm tra thẻ căn cước*

(Code giữ nguyên như bản gốc – bạn đã làm rất chuẩn 👍)

---

## 🛡️ 5. Cấu hình Spring Security

### 🔧 SecurityConfig

> File này quyết định:
>
> * API nào cần login
> * API nào public
> * JWT filter chạy ở đâu
> * Khi lỗi thì trả JSON gì

💡 Head First cực kỳ quan trọng:

> *SecurityConfig = luật giao thông của hệ thống*

---

### 🧠 Những điểm mấu chốt cần nhớ

* ❌ **Tắt CSRF** → vì dùng JWT
* ❌ **Không dùng Session** → Stateless
* ✅ Thêm **JWT filter trước UsernamePasswordAuthenticationFilter**
* ✅ Custom handler cho:

  * Chưa login
  * Không đủ quyền

---

## 🚫 6. Xử lý lỗi bảo mật theo REST

### 🔹 Không đủ quyền → `RestfulAccessDeniedHandler`

👉 Trả JSON thay vì HTML

### 🔹 Chưa login / token hết hạn → `RestAuthenticationEntryPoint`

👉 Frontend dễ xử lý hơn

💡 Head First nhớ:

> *REST API không trả trang lỗi – chỉ trả JSON*

---

## 👤 7. AdminUserDetails – user chuẩn Spring Security

> Spring Security **KHÔNG dùng entity trực tiếp**
> 👉 Phải bọc lại bằng `UserDetails`

`AdminUserDetails` chứa:

* Username
* Password
* Status
* Permission list

👉 Permission được map thành `GrantedAuthority`

---

## 🧱 8. JwtAuthenticationTokenFilter

> Đây là **cửa kiểm soát an ninh**

Nó làm gì?

1. Lấy token từ header
2. Decode username
3. Load user từ DB
4. Validate token
5. Gắn user vào `SecurityContext`

💡 Head First nhớ:

> *Không có filter này → JWT chỉ là chuỗi vô nghĩa*

---

## 📦 Source code dự án

🔗 GitHub:
[https://github.com/macrozheng/mall-learning/tree/master/mall-tiny-04](https://github.com/macrozheng/mall-learning/tree/master/mall-tiny-04)

---

## 📢 公众号

![Image](https://opengraph.githubassets.com/0e4358626612706b3d9867e82818afa40c744572ddb56dcd795566d96379e1ae/macrozheng/mall)

![Image](https://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/banner/qrcode_for_macrozheng_258.jpg)

👉 Theo dõi để:

* Có lộ trình học Spring Security rõ ràng
* Hiểu JWT từ gốc
* Không đi đường vòng ❌
