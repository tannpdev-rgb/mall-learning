## 📚 Học tập không đi đường vòng

👉 **[Theo dõi公众号](#公众号)** và **trả lời “学习路线”** để nhận **lộ trình học RIÊNG cho dự án mall**!

---

# ⏰ Dự án mall: Tích hợp Spring Task để triển khai **tác vụ định kỳ**

> Bài viết này sẽ **dẫn bạn từng bước** tích hợp **Spring Task** vào dự án **mall**,
> thông qua một ví dụ **rất đời thực**:
>
> 👉 **Hủy đơn hàng quá hạn và hoàn lại tồn kho theo lịch định kỳ**

💡 Head First nói thẳng:

> *Việc gì lặp đi lặp lại theo thời gian → giao cho Scheduler làm!* 😄

---

## 🧩 1. Giới thiệu framework sử dụng

### ⏱️ Spring Task là gì?

> **Spring Task** là công cụ **lập lịch (schedule)** nhẹ, gọn, do chính Spring cung cấp.

So với Quartz:

* ✅ Cấu hình **đơn giản hơn**
* ✅ **Không cần thêm dependency**
* ✅ Đủ dùng cho **đa số bài toán business**

👉 Kết luận nhanh:

> *Không cần workflow phức tạp → chọn Spring Task*

---

### 🧠 Cron Expression là gì?

> **Cron expression** là một chuỗi ký tự dùng để **chỉ định thời điểm chạy task**.

Trong Spring Task, Cron giúp bạn nói với hệ thống:

> *“Hãy chạy việc này đúng lúc tao cần”* 😎

---

## 🧾 2. Cú pháp Cron Expression

### 📐 Cấu trúc tổng quát

```
Seconds Minutes Hours DayOfMonth Month DayOfWeek
```

![Image](https://substackcdn.com/image/fetch/%24s_%21JIXk%21%2Cf_auto%2Cq_auto%3Agood%2Cfl_progressive%3Asteep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F494650c6-0993-4b60-b69f-f1cd05c464c6_1600x995.png)

![Image](https://ppolyzos.com/wp-content/uploads/2020/05/cron-expressions-notes.jpg)

---

### 📊 Ý nghĩa từng trường trong Cron

| Trường     | Ký tự cho phép | Giá trị          |
| ---------- | -------------- | ---------------- |
| Seconds    | , - * /        | 0–59             |
| Minutes    | , - * /        | 0–59             |
| Hours      | , - * /        | 0–23             |
| DayOfMonth | , - * / ? L W  | 1–31             |
| Month      | , - * /        | 1–12             |
| DayOfWeek  | , - * / ? L #  | 1–7 hoặc SUN–SAT |

---

### ✨ Các ký tự đặc biệt trong Cron (rất hay dùng)

| Ký tự | Ý nghĩa             | Ví dụ                           |
| ----- | ------------------- | ------------------------------- |
| `,`   | Liệt kê             | `5,10` → phút 5 & 10            |
| `-`   | Khoảng              | `5-10` → từ phút 5 đến 10       |
| `*`   | Bất kỳ              | `*` → mỗi phút                  |
| `/`   | Chu kỳ              | `5/10` → từ phút 5, mỗi 10 phút |
| `?`   | Không xác định      | dùng cho Day                    |
| `#`   | Thứ mấy trong tháng | `1#3` → Chủ nhật thứ 3          |
| `L`   | Cuối cùng           | `5L` → Thứ 5 cuối               |
| `W`   | Ngày làm việc       | `5W` → ngày làm việc gần nhất   |

💡 Head First nhớ:

> *Cron nhìn rối, nhưng dùng quen là cực kỳ “đã”*

---

## 🧠 3. Bài toán nghiệp vụ (rất thực tế)

Hãy tưởng tượng hệ thống bán hàng của bạn 👇

1. User đặt hàng
2. Hệ thống tạo đơn + **khóa tồn kho**
3. Nếu **60 phút không thanh toán** → hủy đơn
4. **Hoàn lại tồn kho**
5. Việc kiểm tra này **phải chạy định kỳ**

👉 Không thể chờ user gọi API
👉 Không thể làm thủ công
👉 **Scheduler sinh ra cho chuyện này**

---

## 🔌 4. Tích hợp Spring Task

💡 Tin vui:

> *Spring Task có sẵn trong Spring Boot → KHÔNG cần thêm dependency*

---

### ⚙️ Bước 1: Bật Spring Task

> Chỉ cần **1 annotation** 🎯

```java
@Configuration
@EnableScheduling
public class SpringTaskConfig {
}
```

👉 Từ giờ, project của bạn **có khả năng chạy task định kỳ**

---

### 🧱 Bước 2: Tạo task xử lý đơn hàng quá hạn

> Ta tạo một component chuyên làm việc này.

```java
@Component
public class OrderTimeOutCancelTask {
    private Logger LOGGER = LoggerFactory.getLogger(OrderTimeOutCancelTask.class);

    /**
     * Cron: mỗi 10 phút chạy 1 lần
     * Quét các đơn quá hạn chưa thanh toán → hủy + hoàn kho
     */
    @Scheduled(cron = "0 0/10 * ? * ?")
    private void cancelTimeOutOrder() {
        // TODO: gọi service hủy đơn & hoàn kho
        LOGGER.info("Hủy đơn quá hạn và giải phóng tồn kho");
    }
}
```

![Image](https://www.callicoder.com/static/f8fdf1db471f36281a695fe939878ed9/3c051/spring-boot-task-scheduler-example-directory-structure.png)

![Image](https://javatechonline.com/wp-content/uploads/2023/01/CronExpression-2.jpg)

---

### 🧠 Phân tích Cron trong ví dụ

```
0 0/10 * ? * ?
```

| Phần   | Ý nghĩa             |
| ------ | ------------------- |
| `0`    | giây = 0            |
| `0/10` | mỗi 10 phút         |
| `*`    | mọi giờ             |
| `?`    | không quan tâm ngày |
| `*`    | mọi tháng           |
| `?`    | không quan tâm thứ  |

👉 Kết luận:

> **Cứ mỗi 10 phút → chạy task**

---

## 💡 Head First – Những điều cần nhớ

* Spring Task **rất hợp cho job đơn giản**
* Cron là **linh hồn của Scheduler**
* Task nên:

  * Nhẹ
  * Nhanh
  * Idempotent (chạy lại không lỗi)

👉 Job nặng → tách service
👉 Job phức tạp → cân nhắc Quartz / Message Queue

---

## 📦 Source code dự án

🔗 GitHub:
[https://github.com/macrozheng/mall-learning/tree/master/mall-tiny-05](https://github.com/macrozheng/mall-learning/tree/master/mall-tiny-05)

---

## 📢 公众号

![Image](https://opengraph.githubassets.com/0e4358626612706b3d9867e82818afa40c744572ddb56dcd795566d96379e1ae/macrozheng/mall)

![Image](https://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/banner/qrcode_for_macrozheng_258.jpg)

👉 Theo dõi để:

* Hiểu Scheduler trong dự án thực tế
* Kết hợp **Spring Task + Business**
* Không đi đường vòng ❌
