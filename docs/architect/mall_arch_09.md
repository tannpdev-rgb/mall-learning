## 📚 Học tập không đi đường vòng

👉 **[Theo dõi公众号](#公众号)** và **trả lời “学习路线”** để nhận **lộ trình học RIÊNG cho dự án mall**!

---

# ⏰ Dự án mall: Tích hợp RabbitMQ để xử lý **delay message (tin nhắn trễ)**

> Bài viết này sẽ **dẫn bạn từng bước** tích hợp **RabbitMQ vào dự án mall**
> để giải quyết một bài toán kinh điển trong e-commerce:
>
> 👉 **Đơn hàng quá hạn thanh toán thì tự động bị hủy**

💡 Head First nói thẳng:

> *Delay message = không phải chờ, không phải cron, không phải polling.*

---

## 🧩 1. RabbitMQ là gì?

> **RabbitMQ** là một **message queue (hàng đợi tin nhắn)** mã nguồn mở,
> được dùng để:
>
> * Tách producer & consumer
> * Xử lý bất đồng bộ
> * Chịu tải cao
> * Đảm bảo message không bị mất

👉 Trong mall:

> *Đặt hàng* ≠ *Hủy đơn*
> → Hai việc này **không nên chạy cùng lúc**

---

## ⚙️ 2. Cài đặt RabbitMQ (Windows)

### 🧱 Bước 1: Cài Erlang (bắt buộc)

🔗 Link tải:
[http://erlang.org/download/otp_win64_21.3.exe](http://erlang.org/download/otp_win64_21.3.exe)

![Image](https://www.rose-hulman.edu/class/csse/resources/Erlang/ErlPrompt.png)

![Image](https://www.tutorialspoint.com/erlang/images/select_components.jpg)

💡 Head First nhớ:

> *Không có Erlang → RabbitMQ không chạy được*

---

### 🐰 Bước 2: Cài RabbitMQ

🔗 Link tải (v3.7.14):
[https://dl.bintray.com/rabbitmq/all/rabbitmq-server/3.7.14/rabbitmq-server-3.7.14.exe](https://dl.bintray.com/rabbitmq/all/rabbitmq-server/3.7.14/rabbitmq-server-3.7.14.exe)

![Image](https://raw.github.com/mythz/rabbitmq-windows/master/img/rabbitmq-management-ui.png)

![Image](https://raw.github.com/mythz/rabbitmq-windows/master/img/rabbitmq-service.png)

---

### 🧰 Bước 3: Bật giao diện quản lý (Management Plugin)

Vào thư mục `sbin` → mở CMD → chạy:

```bash
rabbitmq-plugins enable rabbitmq_management
```

![Image](https://coderjony.com/img/blogs/how-to-enable-rabbitmq-management-plugin-in-windows/rabbitmq-user-interface-2.png)

![Image](https://static.thegeekstuff.com/wp-content/uploads/2013/10/rabbitmq-set-current-permission.png)

Truy cập:

```
http://localhost:15672
```

Tài khoản mặc định:

```
guest / guest
```

![Image](https://www.cloudamqp.com/img/blog/management-overview.png)

![Image](https://www.rabbitmq.com/assets/images/management-oauth-with-basic-auth-3711e59ce457ceb2900716d53e5cd731.png)

---

### 👤 Bước 4: Tạo user & virtual host

* User: `mall / mall`
* Role: **administrator**
* Virtual host: `/mall`
* Gán quyền cho user `mall`

![Image](https://www.cloudamqp.com/img/blog/vhost-rabbitmq-management.png)

![Image](https://www.tutlane.com/images/rabbitmq/rabbitmq_management_set_user_permissions.PNG)

👉 Đến đây: **RabbitMQ sẵn sàng chiến đấu** 💪

---

## 🧠 3. Mô hình message trong RabbitMQ

![Image](https://www.rabbitmq.com/assets/images/hello-world-example-routing-cbe9a872b37956a4072a5e13f9d76e7b.png)

![Image](https://www.cloudamqp.com/img/blog/exchanges-topic-fanout-direct.png)

| Ký hiệu | Tên      | Ý nghĩa                   |
| ------- | -------- | ------------------------- |
| P       | Producer | Gửi message               |
| X       | Exchange | Nhận & định tuyến message |
| Q       | Queue    | Lưu message               |
| C       | Consumer | Xử lý message             |

💡 Head First nhớ:

> *Producer không gửi thẳng vào Queue → phải qua Exchange*

---

## 🎯 4. Bài toán nghiệp vụ: Hủy đơn hàng quá hạn

### Luồng nghiệp vụ chuẩn e-commerce

1. Người dùng **đặt hàng**
2. Hệ thống:

   * Khóa tồn kho
   * Áp voucher
   * Tạo orderId
3. Nếu **60 phút không thanh toán**
4. 👉 **Tự động hủy đơn**

   * Trả tồn kho
   * Trả voucher
   * Hoàn điểm

💡 Câu hỏi lớn:

> *Ai sẽ nhớ để hủy đơn sau 60 phút?*

👉 **RabbitMQ Delay Message trả lời câu hỏi đó.**

---

## 🧱 5. Ý tưởng Delay Message với RabbitMQ

> RabbitMQ **không có delay queue “xịn” mặc định**,
> nên ta dùng:
>
> 👉 **TTL + Dead Letter Queue**

### Luồng tư duy Head First

```
Đặt hàng
  ↓
Gửi message vào queue TTL (có thời gian sống)
  ↓ (hết TTL)
Message tự động chuyển sang queue thật
  ↓
Consumer xử lý → HỦY ĐƠN
```

---

## 📦 6. Thêm dependency

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

## ⚙️ 7. Cấu hình RabbitMQ

```yml
spring:
  rabbitmq:
    host: localhost
    port: 5672
    virtual-host: /mall
    username: mall
    password: mall
    publisher-confirms: true
```

---

## 🧠 8. QueueEnum – gom toàn bộ cấu hình queue

> **Đừng hard-code tên queue** – rất dễ toang 😅

```java
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
```

💡 Head First:

> *Enum = cấu hình tập trung = dễ maintain*

---

## 🧱 9. Cấu hình Exchange & Queue

### Queue TTL (delay queue)

```java
.withArgument("x-dead-letter-exchange", "mall.order.direct")
.withArgument("x-dead-letter-routing-key", "mall.order.cancel")
```

👉 Ý nghĩa:

> *Hết hạn → chuyển message sang queue hủy đơn*

![Image](https://www.cloudamqp.com/img/blog/dead-letter-exchange.png)

![Image](https://miro.medium.com/1%2A__U3ZU5cIU3IFsrAF6T9SA.png)

---

## 📤 10. Producer – gửi delay message

```java
message.getMessageProperties()
       .setExpiration(String.valueOf(delayTimes));
```

👉 `delayTimes` = thời gian chờ (ms)

💡 Head First:

> *Delay nằm trên MESSAGE, không nằm trên queue*

---

## 📥 11. Consumer – nhận message hủy đơn

```java
@RabbitListener(queues = "mall.order.cancel")
public void handle(Long orderId) {
    portalOrderService.cancelOrder(orderId);
}
```

👉 Khi message tới đây:

> **Đơn hàng chắc chắn đã quá hạn**

---

## 🧠 12. Gắn delay message vào flow đặt hàng

```java
// Sau khi tạo đơn
sendDelayMessageCancelOrder(orderId);
```

```java
long delayTimes = 30 * 1000; // demo 30s
cancelOrderSender.sendMessage(orderId, delayTimes);
```

💡 Head First nhớ:

> *Đặt hàng xong là “quên nó đi” – RabbitMQ sẽ nhớ giúp bạn*

---

## 🧪 13. Test API

### Gọi API đặt hàng

![Image](https://i.sstatic.net/LjKwg.png)

![Image](https://i.sstatic.net/Y4m7m.png)

⏳ Sau 30 giây…

![Image](https://www.cloudamqp.com/img/blog/delay-message-exchange.png)

![Image](https://user-images.githubusercontent.com/442035/96842403-46ee6d00-144d-11eb-806c-93261c11ca54.png)

👉 Log xuất hiện:

```
receive delay message orderId=xxx
process cancelOrder
```

🎉 Thành công!

---

## 📦 Source code dự án

🔗 GitHub:
[https://github.com/macrozheng/mall-learning/tree/master/mall-tiny-08](https://github.com/macrozheng/mall-learning/tree/master/mall-tiny-08)

---

## 📢 公众号

![Image](https://opengraph.githubassets.com/0e4358626612706b3d9867e82818afa40c744572ddb56dcd795566d96379e1ae/macrozheng/mall)

![Image](https://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/banner/qrcode_for_macrozheng_258.jpg)

👉 Theo dõi để:

* Hiểu **RabbitMQ sâu hơn Kafka**
* Xây hệ thống **event-driven**
* Không đi đường vòng ❌
