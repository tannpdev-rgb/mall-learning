## 📚 Học tập không đi đường vòng

👉 **[Theo dõi公众号](#公众号)** và **trả lời “学习路线”** để nhận **lộ trình học RIÊNG cho dự án mall**!

---

# ⚡ Dự án mall: Tích hợp Redis để triển khai cơ chế cache

> Bài viết này sẽ **dẫn bạn từng bước** tích hợp **Redis vào dự án mall**,
> thông qua một ví dụ **rất thực tế**:
>
> 👉 **Lưu và xác thực mã OTP (SMS verification code)**.

💡 Head First nói thẳng:

> *Cái gì đọc nhiều – ghi ít → cho vào Redis!*
> OTP chính là ví dụ kinh điển.

---

## 🧠 Redis là gì? Vì sao dùng Redis?

> **Redis** là một **database key–value** hiệu năng cao,
> được viết bằng **C**, cực kỳ nhanh.

Redis thường dùng để:

* Cache dữ liệu
* Lưu session
* Lưu OTP / token
* Giảm tải cho database chính

👉 Trong bài này, Redis đóng vai:

> **“Nơi giữ OTP tạm thời”**

---

## 🧱 1. Cài đặt và khởi động Redis (Windows)

### 📥 Bước 1: Tải Redis

🔗 Link tải:
[https://github.com/MicrosoftArchive/redis/releases](https://github.com/MicrosoftArchive/redis/releases)

![Image](https://user-images.githubusercontent.com/515784/215540157-65f55297-cde2-49b3-8ab3-14dca7e11ee0.png)

![Image](https://opengraph.githubassets.com/8ed7824a0bd4327adfd32af2e8585aedd2c06b82ff044c5569279c4ed6431c17/redis-windows/redis-windows)

---

### 📂 Bước 2: Giải nén Redis

> Giải nén Redis vào **bất kỳ thư mục nào bạn muốn**

![Image](https://docs.servicestack.net/img/pages/redis/install-wsl.png)

![Image](https://i.sstatic.net/I0Btt.png)

---

### ▶️ Bước 3: Khởi động Redis

> Mở **CMD tại thư mục Redis**, chạy lệnh:

```bash
redis-server.exe redis.windows.conf
```

![Image](https://i.sstatic.net/I0Btt.png)

![Image](https://i.sstatic.net/RVHvS.png)

👉 Thấy Redis chạy → OK
👉 Không thấy lỗi → sẵn sàng dùng

---

## 🔌 2. Tích hợp Redis vào Spring Boot

### 📦 Bước 1: Thêm dependency Redis

> Mở `pom.xml` và thêm 👇

```xml
<!-- Redis starter -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

💡 Head First note:

> *Spring Boot + Redis Starter = gần như không cần config phức tạp*

---

### ⚙️ Bước 2: Cấu hình Redis trong `application.yml`

#### 🔹 Cấu hình Redis server (trong `spring:`)

```yml
redis:
  host: localhost        # Địa chỉ Redis
  database: 0            # Database index
  port: 6379             # Port Redis
  password:              # Mật khẩu (thường để trống)
  jedis:
    pool:
      max-active: 8
      max-wait: -1ms
      max-idle: 8
      min-idle: 0
  timeout: 3000ms
```

👉 Phần này nói cho Spring biết:

> *Redis của tao đang ở đâu và kết nối thế nào*

---

#### 🔹 Cấu hình key Redis tùy chỉnh (ở root)

```yml
# Custom redis key
redis:
  key:
    prefix:
      authCode: "portal:authCode:"
    expire:
      authCode: 120   # OTP hết hạn sau 120s
```

💡 Head First nhớ:

> *Key rõ ràng + prefix hợp lý = Redis gọn gàng, dễ quản lý*

---

## 🧩 3. Tạo RedisService – gói thao tác Redis lại

> Ta **KHÔNG dùng Redis trực tiếp ở Service nghiệp vụ**
> 👉 Tạo một lớp trung gian: `RedisService`

### 📄 RedisService interface

```java
public interface RedisService {

    // Lưu dữ liệu
    void set(String key, String value);

    // Lấy dữ liệu
    String get(String key);

    // Set thời gian hết hạn
    boolean expire(String key, long expire);

    // Xóa key
    void remove(String key);

    // Tăng giá trị
    Long increment(String key, long delta);
}
```

💡 Head First note:

> *RedisService = Adapter cho Redis*

---

## 🧠 4. Cài đặt RedisService bằng StringRedisTemplate

> Spring đã chuẩn bị sẵn `StringRedisTemplate` cho bạn.

### 🛠️ RedisServiceImpl

```java
@Service
public class RedisServiceImpl implements RedisService {

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @Override
    public void set(String key, String value) {
        stringRedisTemplate.opsForValue().set(key, value);
    }

    @Override
    public String get(String key) {
        return stringRedisTemplate.opsForValue().get(key);
    }

    @Override
    public boolean expire(String key, long expire) {
        return stringRedisTemplate.expire(key, expire, TimeUnit.SECONDS);
    }

    @Override
    public void remove(String key) {
        stringRedisTemplate.delete(key);
    }

    @Override
    public Long increment(String key, long delta) {
        return stringRedisTemplate.opsForValue().increment(key, delta);
    }
}
```

👉 Redis lúc này **đã sẵn sàng phục vụ nghiệp vụ**

---

## 🌐 5. Controller: API lấy & xác thực OTP

### 📄 UmsMemberController

> Ta tạo 2 API:

* Lấy OTP theo số điện thoại
* Kiểm tra OTP

```java
@Api(tags = "UmsMemberController", description = "Đăng ký & đăng nhập")
@RequestMapping("/sso")
public class UmsMemberController {

    @ApiOperation("Lấy mã OTP")
    @GetMapping("/getAuthCode")
    public CommonResult getAuthCode(@RequestParam String telephone) {
        return memberService.generateAuthCode(telephone);
    }

    @ApiOperation("Xác thực mã OTP")
    @PostMapping("/verifyAuthCode")
    public CommonResult verify(
        @RequestParam String telephone,
        @RequestParam String authCode) {

        return memberService.verifyAuthCode(telephone, authCode);
    }
}
```

---

## 🧠 6. Service xử lý logic OTP

### 📄 UmsMemberService

```java
public interface UmsMemberService {

    // Sinh OTP
    CommonResult generateAuthCode(String telephone);

    // Kiểm tra OTP
    CommonResult verifyAuthCode(String telephone, String authCode);
}
```

---

### 🛠️ UmsMemberServiceImpl – logic chính

> Đây là **trái tim của bài học**

#### 🧠 Logic sinh OTP:

1. Sinh 6 số ngẫu nhiên
2. Key = `prefix + telephone`
3. Value = OTP
4. Set expire = 120s

```java
redisService.set(prefix + telephone, otp);
redisService.expire(prefix + telephone, 120);
```

---

#### 🧠 Logic kiểm tra OTP:

1. Lấy OTP từ Redis
2. So sánh với OTP người dùng nhập
3. Đúng → OK
4. Sai → Fail

```java
String realAuthCode =
    redisService.get(prefix + telephone);
```

💡 Head First kết luận:

> *Redis = bộ nhớ tạm, OTP đúng nghĩa “sống ngắn”*

---

## ▶️ 7. Chạy project & test API

### 🌐 Truy cập Swagger UI

📍

```
http://localhost:8080/swagger-ui.html
```

![Image](https://redis.io/docs/latest/images/rv/api/swagger-post-edit-body.png)

![Image](https://static1.smartbear.co/swagger/media/images/tools/opensource/swaggerhub-swaggerui.png)

👉 Test:

* Lấy OTP
* Nhập OTP
* Xác thực

---

## 📦 Source code dự án

🔗 GitHub:
[https://github.com/macrozheng/mall-learning/tree/master/mall-tiny-03](https://github.com/macrozheng/mall-learning/tree/master/mall-tiny-03)

---

## 📢 公众号

![Image](https://opengraph.githubassets.com/0e4358626612706b3d9867e82818afa40c744572ddb56dcd795566d96379e1ae/macrozheng/mall)

![Image](https://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/banner/qrcode_for_macrozheng_258.jpg)

👉 Theo dõi để:

* Có lộ trình học rõ ràng
* Hiểu Redis + Spring Boot bài bản
* Không đi đường vòng ❌

👉 Cứ nói, mình sẽ **đi cùng bạn từng bước, đúng chất Head First** 💙
