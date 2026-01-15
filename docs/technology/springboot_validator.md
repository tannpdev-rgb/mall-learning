Học tập **không đi đường vòng** 🧭
👉 [Theo dõi公众号](#公众号) và **trả lời “学习路线”** để nhận **lộ trình học riêng cho dự án mall**!

---

# Hai cách xử lý validate trong Spring Boot – nhìn thì đơn giản, nhưng rất “thông minh”

> Khi viết API, **validate tham số** là việc xảy ra mỗi ngày:
>
> * tên có được để trống không?
> * số có âm không?
> * trạng thái có hợp lệ không?
>
> Bài này giới thiệu **2 cách xử lý validate** thường dùng trong dự án mall:
>
> 1️⃣ Hibernate Validator (validate bằng annotation)
> 2️⃣ Validate bằng **Global Exception + Assert**

🧠 **Head First mindset**
Đừng hỏi *“cách nào đúng?”*
👉 Hãy hỏi *“cách nào phù hợp với loại logic này?”*

---

## 1️⃣ Cách 1: Hibernate Validator (validate bằng annotation)

> Hibernate Validator là **framework validate mặc định** của Spring Boot
> → chỉ cần dùng Spring Boot là **có sẵn**.

### 🧠 Ý tưởng cốt lõi

> “Để dữ liệu **tự nói xem nó có hợp lệ không**”

Bạn **không cần if–else trong controller**,
chỉ cần **khai báo luật ngay trên DTO**.

---

## Các annotation thường dùng (nhìn mặt để nhớ)

| Annotation      | Ý nghĩa             |
| --------------- | ------------------- |
| `@NotNull`      | không được null     |
| `@NotEmpty`     | không rỗng          |
| `@NotBlank`     | không rỗng (String) |
| `@Min` / `@Max` | giá trị min / max   |
| `@Size`         | độ dài              |
| `@Pattern`      | regex               |
| `@Email`        | email hợp lệ        |

🧠 **Ghi nhớ Head First**
👉 Đây là **validate hình thức**, không phải business logic.

---

## Ví dụ: validate khi thêm Brand

### 1️⃣ Đặt luật ngay trong DTO

```java
public class PmsBrandParam {
    @NotEmpty(message = "名称不能为空")
    private String name;

    @Min(value = 0, message = "排序最小为0")
    private Integer sort;

    @FlagValidator(value = {"0","1"}, message = "显示状态不正确")
    private Integer showStatus;
}
```

🧠 Khi đọc class này, bạn **đã biết ngay luật validate**
→ rất dễ đọc, rất “self-document”.

---

### 2️⃣ Bật validate trong Controller

```java
public CommonResult create(
    @Validated @RequestBody PmsBrandParam pmsBrand,
    BindingResult result
)
```

👉 `@Validated` = bật validate
👉 `BindingResult` = nơi Spring bỏ lỗi vào

---

### 3️⃣ Dùng AOP để xử lý BindingResult (tránh lặp code)

🧠 Nếu mỗi API đều viết:

```java
if(result.hasErrors()) { ... }
```

👉 Code sẽ **rất bẩn** ❌

Giải pháp: **AOP**

```java
@Around("execution(public * com.macro.mall.controller.*.*(..))")
```

Logic:

1. Tìm `BindingResult`
2. Nếu có lỗi → return ngay
3. Nếu không → cho method chạy

👉 Controller **sạch như nước suối** 🌊

---

### 4️⃣ Kết quả

* Không truyền `name`
* → trả về: **“名称不能为空”**

![](../images/springboot_validator_01.png)

---

## ❗ Nhược điểm của cách này

🧠 Head First phân tích thẳng:

* ❌ Phải truyền `BindingResult` vào method
* ❌ Chỉ phù hợp validate **đơn giản**
* ❌ **Không làm được** logic kiểu:

  * “đã tồn tại trong DB chưa?”
  * “user này đã nhận coupon chưa?”

👉 Vì vậy cần **cách thứ 2**.

---

## 2️⃣ Cách 2: Validate bằng Global Exception + Assert

> Ý tưởng rất đơn giản:
>
> 👉 **Validate thất bại → ném exception**
> 👉 **Controller không cần biết chi tiết**

---

## 🧠 Tư duy thiết kế (rất quan trọng)

> Service **không nên trả CommonResult**

Service chỉ nên:

* chạy logic
* hoặc **fail**

👉 Việc “bọc response” là trách nhiệm của Controller / Global Handler.

---

## 1️⃣ Tạo exception riêng cho API

```java
public class ApiException extends RuntimeException {
    private IErrorCode errorCode;
}
```

🧠 Dùng RuntimeException để:

* không cần try–catch
* fail là fail ngay

---

## 2️⃣ Tạo lớp Asserts (rất hay)

```java
public class Asserts {
    public static void fail(String message) {
        throw new ApiException(message);
    }
}
```

🧠 Khi đọc code:

```java
Asserts.fail("优惠券不存在");
```

👉 Ý nghĩa **rõ hơn if–else rất nhiều**.

---

## 3️⃣ Global Exception Handler

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ApiException.class)
    public CommonResult handle(ApiException e) {
        return CommonResult.failed(e.getMessage());
    }
}
```

👉 Mọi `ApiException` → **tự động convert thành response**

---

## 4️⃣ So sánh trước & sau (rất đáng học)

### ❌ Trước (Service trả CommonResult)

```java
if(coupon == null){
    return CommonResult.failed("优惠券不存在");
}
```

👉 Service **dính UI logic** ❌

---

### ✅ Sau (Service chỉ lo logic)

```java
if(coupon == null){
    Asserts.fail("优惠券不存在");
}
```

👉 Service:

* sạch
* dễ test
* dễ reuse

Controller:

```java
memberCouponService.add(couponId);
return CommonResult.success(null,"领取成功");
```

---

### Test lỗi

Truyền couponId không tồn tại:

![](../images/springboot_validator_03.png)

👉 Response đúng, code gọn.

---

## ⚖️ So sánh nhanh 2 cách

| Tiêu chí       | Hibernate Validator | Global Exception |
| -------------- | ------------------- | ---------------- |
| Độ gọn         | ⭐⭐⭐⭐                | ⭐⭐⭐              |
| Dễ đọc         | ⭐⭐⭐⭐                | ⭐⭐⭐⭐             |
| Validate DB    | ❌                   | ✅                |
| Business logic | ❌                   | ✅                |
| Annotation     | ✅                   | ❌                |

---

## 🧠 Kết luận Head First (rất quan trọng)

> **Không có cách nào “ăn hết”** 🍱
> Cách đúng là **kết hợp cả hai**.

👉 Quy tắc dùng trong dự án mall:

* ✅ **Validate đơn giản** (null, range, format)
  → **Hibernate Validator**
* ✅ **Validate phức tạp** (DB, trạng thái, business rule)
  → **Global Exception + Asserts**

---

## Mã nguồn tham khảo

🔗 [https://github.com/macrozheng/mall](https://github.com/macrozheng/mall)

---

## 公众号

![公众号图片](http://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/banner/qrcode_for_macrozheng_258.jpg)
