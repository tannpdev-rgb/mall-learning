学习不走弯路，[关注公众号](#公众号) 回复「学习路线」，获取mall项目专属学习路线！

---

# mall tích hợp RabbitMQ để triển khai **Delayed Message (消息延迟)**

> Bài viết này hướng dẫn cách **mall tích hợp RabbitMQ để xử lý message delay**,
> ví dụ điển hình: **tự động hủy đơn hàng khi quá thời gian thanh toán**.

---

## 🧠 Tư duy Head First – Vấn đề thật sự là gì?

👉 Khi người dùng **đặt hàng nhưng không thanh toán**, hệ thống phải:

* Không hủy ngay ❌
* Không chờ user bấm nút ❌
* **Tự động xử lý sau X phút** ✅

💡 Vấn đề cốt lõi:

> ❓ Làm sao “hẹn giờ” một hành động trong hệ thống phân tán?

Câu trả lời của mall:
👉 **RabbitMQ + Dead Letter Queue (TTL)**

---

## Giới thiệu framework sử dụng

---

## 🐰 RabbitMQ là gì?

> RabbitMQ là một **message broker mã nguồn mở**, được sử dụng rộng rãi.
>
> Nó nhẹ, dễ triển khai, hỗ trợ nhiều protocol và cực kỳ phù hợp cho:
>
> * hệ thống phân tán
> * xử lý bất đồng bộ
> * retry / delay / event-driven

---

## Cài đặt RabbitMQ (Windows)

### 1️⃣ Cài Erlang (RabbitMQ chạy trên Erlang)

Link tải:
[http://erlang.org/download/otp_win64_21.3.exe](http://erlang.org/download/otp_win64_21.3.exe)

![](../images/arch_screen_53.png)

---

### 2️⃣ Cài RabbitMQ

Link tải:
[https://dl.bintray.com/rabbitmq/all/rabbitmq-server/3.7.14/rabbitmq-server-3.7.14.exe](https://dl.bintray.com/rabbitmq/all/rabbitmq-server/3.7.14/rabbitmq-server-3.7.14.exe)

![](../images/arch_screen_54.png)

---

### 3️⃣ Vào thư mục `sbin` của RabbitMQ

![](../images/arch_screen_55.png)

---

### 4️⃣ Bật giao diện quản lý

```bash
rabbitmq-plugins enable rabbitmq_management
```

![](../images/arch_screen_56.png)

---

### 5️⃣ Truy cập giao diện quản lý

[http://localhost:15672/](http://localhost:15672/)

![](../images/arch_screen_57.png)

---

### 6️⃣ Đăng nhập mặc định

* username: `guest`
* password: `guest`

---

### 7️⃣ Tạo user mới: `mall / mall` (admin)

![](../images/arch_screen_58.png)

---

### 8️⃣ Tạo Virtual Host `/mall`

![](../images/arch_screen_59.png)

---

### 9️⃣ Gán quyền cho user mall

![](../images/arch_screen_60.png)

![](../images/arch_screen_61.png)

✅ **Hoàn tất cấu hình RabbitMQ**

---

## 🧩 Mô hình message của RabbitMQ

![](../images/arch_screen_52.png)

| Ký hiệu | Tên           | Tiếng Anh     | Mô tả                     |
| ------- | ------------- | ------------- | ------------------------- |
| P       | Producer      | Người gửi     | Gửi message               |
| X       | Exchange      | Bộ định tuyến | Quyết định message đi đâu |
| Q       | Queue         | Hàng đợi      | Lưu message               |
| C       | Consumer      | Người nhận    | Xử lý message             |
| type    | Loại exchange | direct        | match theo routing key    |

🧠 **Head First ghi nhớ**:

> Producer **KHÔNG gửi thẳng** vào Queue
> 👉 Luôn gửi qua **Exchange**

---

## Lombok là gì?

> Lombok giúp bạn **khỏi viết getter / setter / constructor**.

📌 Chỉ cần:

* cài plugin Lombok trong IDEA
* thêm dependency

![](../images/arch_screen_48.png)

---

## 🎯 Bối cảnh nghiệp vụ (Business Scenario)

### Vấn đề: Đơn hàng quá hạn thanh toán

Luồng thực tế:

1. User đặt hàng
2. Hệ thống:

   * khóa tồn kho
   * áp dụng coupon
   * trừ điểm
3. Tạo orderId
4. Nếu **sau 60 phút chưa thanh toán**:

   * hủy đơn
   * trả tồn kho
   * trả coupon
   * trả điểm

❓ Làm sao biết “60 phút sau” để xử lý?

👉 **Delayed Message**

---

## 🚀 Tích hợp RabbitMQ để xử lý Delayed Message

---

## 1️⃣ Thêm dependency

```xml
<!-- RabbitMQ -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>

<!-- Lombok -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
```

---

## 2️⃣ Cấu hình RabbitMQ

```yml
rabbitmq:
  host: localhost
  port: 5672
  virtual-host: /mall
  username: mall
  password: mall
  publisher-confirms: true
```

---

## 3️⃣ Enum định nghĩa Queue (rất hay!)

### 🧠 Head First:

> **Tên exchange, queue, routing key**
> 👉 không viết hard-code
> 👉 gom vào enum

```java
public enum QueueEnum {
    QUEUE_ORDER_CANCEL(
        "mall.order.direct",
        "mall.order.cancel",
        "mall.order.cancel"
    ),
    QUEUE_TTL_ORDER_CANCEL(
        "mall.order.direct.ttl",
        "mall.order.cancel.ttl",
        "mall.order.cancel.ttl"
    );
}
```

📌 Có 2 queue:

* **TTL queue**: chờ hết hạn
* **Real queue**: xử lý hủy đơn

---

## 4️⃣ Cấu hình Exchange + Queue + Binding

👉 Điểm mấu chốt:

```java
.withArgument("x-dead-letter-exchange", "mall.order.direct")
.withArgument("x-dead-letter-routing-key", "mall.order.cancel")
```

🧠 **Giải thích Head First**:

* Message vào `TTL queue`
* Hết thời gian → trở thành **dead letter**
* Tự động chuyển sang queue xử lý thật

📦 Bạn **KHÔNG cần cron job**

---

### Trên giao diện RabbitMQ sẽ thấy

![](../images/arch_screen_62.png)
![](../images/arch_screen_63.png)
![](../images/arch_screen_64.png)
![](../images/arch_screen_65.png)

---

## 5️⃣ Producer – Gửi message delay

```java
message.getMessageProperties()
       .setExpiration(String.valueOf(delayTimes));
```

🧠 Ví dụ:

* delayTimes = `30_000`
* message chờ 30 giây
* sau đó được chuyển sang queue hủy đơn

---

## 6️⃣ Consumer – Nhận message & hủy đơn

```java
@RabbitListener(queues = "mall.order.cancel")
public void handle(Long orderId) {
    portalOrderService.cancelOrder(orderId);
}
```

📌 Consumer **không quan tâm delay**
👉 chỉ xử lý khi message tới

---

## 7️⃣ Service – Nối business với MQ

```java
sendDelayMessageCancelOrder(orderId);
```

🧠 Head First flow:

```
generateOrder()
   ↓
send delay message
   ↓
RabbitMQ chờ
   ↓
cancelOrder()
```

---

## 8️⃣ Test API

⏱ Delay được set: **30 giây**

![](../images/arch_screen_49.png)
![](../images/arch_screen_50.png)
![](../images/arch_screen_51.png)

👉 Sau 30s → log hủy đơn xuất hiện

---

## 🧠 RECAP – Tổng kết Head First

### 🎯 Bạn vừa học được gì?

✅ RabbitMQ **không chỉ để async**, mà còn dùng delay
✅ Delay Message = **TTL + Dead Letter Queue**
✅ Không cần cron job
✅ Không block thread
✅ Rất phù hợp hệ thống ecommerce

---

### 🎯 Khi nào nên dùng cách này?

| Tình huống                 | Có nên dùng |
| -------------------------- | ----------- |
| Hủy đơn quá hạn            | ✅           |
| Retry thanh toán           | ✅           |
| Gửi mail sau X phút        | ✅           |
| Task cực chính xác theo ms | ❌           |

---

### 🎯 Câu thần chú cần nhớ

> **RabbitMQ không có delay thật**
> 👉 Delay là do **Queue giữ message**
> 👉 Hết hạn → đẩy sang queue khác
