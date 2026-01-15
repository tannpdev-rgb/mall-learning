Học tập **không đi đường vòng** 🧭
👉 [Theo dõi公众号](#公众号) và **trả lời “学习路线”** để nhận **lộ trình học riêng cho dự án mall**!

---

# Ghi log truy cập API trong Spring Boot bằng AOP

> Bài viết này trình bày **cách sử dụng AOP trong dự án mall**
> để **ghi log truy cập API một cách tập trung**,
> bằng cách **tạo một aspect tại tầng Controller**.

🧠 **Head First mindset**
Nếu bạn:

* ghi log trong từng controller ❌
* copy–paste log ở mọi API ❌

👉 Bạn đang **tự tạo địa ngục cho chính mình** 😵

AOP sinh ra để:

> **làm một lần – áp dụng cho tất cả**

---

## 1️⃣ AOP là gì? (hiểu đúng trước khi code)

> **AOP (Aspect Oriented Programming)**
> = Lập trình hướng *cắt ngang*

🧠 Head First giải thích đơn giản:

* OOP → chia theo **đối tượng**
* AOP → chia theo **hành vi dùng chung**

Ví dụ hành vi dùng chung:

* log
* transaction
* security
* performance tracking

👉 Những thứ này:

* **không thuộc business**
* nhưng **xuất hiện ở khắp nơi**

---

## 2️⃣ Các thuật ngữ AOP (đọc chậm)

### 🔔 Advice (Thông báo)

> **Advice = việc bạn muốn làm**

Ví dụ:

* ghi log
* đo thời gian
* bắt exception

Các loại advice (rất hay dùng):

| Loại           | Khi nào chạy                  |
| -------------- | ----------------------------- |
| Before         | Trước khi method chạy         |
| After          | Sau khi method chạy           |
| AfterReturning | Chạy khi method thành công    |
| AfterThrowing  | Chạy khi method ném exception |
| Around         | Bao trọn method (trước + sau) |

🧠 Trong bài này → dùng **Around** vì cần:

* đo thời gian
* lấy request
* lấy response

---

### 🔗 JoinPoint (Điểm nối)

> **JoinPoint = thời điểm AOP can thiệp**

Ví dụ:

* Khi API `/brand/list` được gọi
* → đó chính là một JoinPoint

---

### 🎯 Pointcut (Điểm cắt)

> **Pointcut = chọn chỗ nào sẽ áp dụng AOP**

Ví dụ:

```java
execution(public * com.macro.mall.tiny.controller.*.*(..))
```

👉 Nghĩa là:

* tất cả method
* public
* trong package controller

---

### 🧩 Aspect (Mảnh ghép hoàn chỉnh)

> **Aspect = Advice + Pointcut**

👉 Nói cách khác:

> *“Khi nào”* + *“Làm gì”*

---

### 🧵 Weaving (Dệt)

> **Weaving = quá trình gắn aspect vào code**

Spring sẽ:

* tạo proxy
* wrap method gốc
* gọi advice trước / sau method

👉 Bạn **không cần tự làm**.

---

## 3️⃣ Annotation AOP trong Spring (nhớ mặt chữ)

| Annotation        | Ý nghĩa                  |
| ----------------- | ------------------------ |
| `@Aspect`         | Đánh dấu class là aspect |
| `@Pointcut`       | Định nghĩa phạm vi       |
| `@Before`         | Chạy trước               |
| `@After`          | Chạy sau                 |
| `@AfterReturning` | Sau khi thành công       |
| `@AfterThrowing`  | Khi exception            |
| `@Around`         | Bao trọn method          |

---

## 4️⃣ Tạo class chứa thông tin log – `WebLog`

🧠 **Tư duy Head First**
Trước khi log → hãy hỏi:

> “Mình muốn log NHỮNG GÌ?”

Câu trả lời:

* API nào
* gọi lúc nào
* mất bao lâu
* param gì
* trả về cái gì

👉 Gom tất cả vào **1 object duy nhất**

```java
public class WebLog {
    private String description;
    private String username;
    private Long startTime;
    private Integer spendTime;
    private String basePath;
    private String uri;
    private String url;
    private String method;
    private String ip;
    private Object parameter;
    private Object result;
}
```

🧠 Lợi ích:

* log JSON
* dễ gửi ELK
* dễ mở rộng sau này

---

## 5️⃣ Viết Aspect ghi log – `WebLogAspect`

### 🎯 Định nghĩa phạm vi áp dụng

```java
@Pointcut("execution(public * com.macro.mall.tiny.controller.*.*(..))")
public void webLog() {}
```

👉 **Tất cả API controller đều bị “soi”** 😄

---

### 🧠 Vì sao dùng `@Around`?

```java
@Around("webLog()")
public Object doAround(ProceedingJoinPoint joinPoint) throws Throwable
```

Vì:

* cần gọi `joinPoint.proceed()` để chạy method thật
* cần đo thời gian trước & sau
* cần lấy response

---

### ⏱️ Đo thời gian xử lý

```java
long startTime = System.currentTimeMillis();
Object result = joinPoint.proceed();
long endTime = System.currentTimeMillis();
```

👉 `spendTime = end - start`

---

### 🌐 Lấy thông tin HTTP Request

```java
ServletRequestAttributes attributes =
 (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();

HttpServletRequest request = attributes.getRequest();
```

👉 Spring đã giữ request trong **ThreadLocal**

---

### 🏷️ Lấy mô tả API từ Swagger

```java
if (method.isAnnotationPresent(ApiOperation.class)) {
    ApiOperation apiOperation = method.getAnnotation(ApiOperation.class);
    webLog.setDescription(apiOperation.value());
}
```

🧠 **Cực hay**:

* Không cần viết description riêng
* Tận dụng Swagger

---

### 📦 Lấy request parameter (điểm khó nhất)

```java
private Object getParameter(Method method, Object[] args)
```

Logic:

* `@RequestBody` → lấy object
* `@RequestParam` → map key–value
* nhiều param → list
* không có → null

👉 Tránh log:

* `HttpServletRequest`
* `HttpServletResponse`

---

### 🖨️ In log dạng JSON

```java
LOGGER.info("{}", JSONUtil.parse(webLog));
```

👉 Log đẹp
👉 Log có cấu trúc
👉 Dễ đưa vào ELK

---

## 6️⃣ Test thực tế

Chạy project → mở Swagger:

```
http://localhost:8080/swagger-ui.html
```

![](../images/refer_screen_107.png)

Console log:

```json
{
  "description": "分页查询品牌列表",
  "method": "GET",
  "uri": "/brand/list",
  "spendTime": 101,
  "parameter": [
    {"pageNum":1},
    {"pageSize":1}
  ],
  "result": {...}
}
```

🧠 **Đây là log chuẩn production**:

* đủ thông tin
* không spam
* không phụ thuộc business

---

## 🧠 Tổng kết Head First (rất quan trọng)

> Nếu bạn nhớ 5 điều này, bạn đã hiểu AOP đúng cách:

1️⃣ AOP = xử lý hành vi dùng chung
2️⃣ Log **không thuộc business**
3️⃣ Around advice mạnh nhất
4️⃣ Aspect giúp code sạch hơn
5️⃣ Log JSON = tương lai (ELK, observability)

---

## Mã nguồn tham khảo

🔗 [https://github.com/macrozheng/mall-learning/tree/master/mall-tiny-aop](https://github.com/macrozheng/mall-learning/tree/master/mall-tiny-aop)

---

## 公众号

![公众号图片](http://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/banner/qrcode_for_macrozheng_258.jpg)
