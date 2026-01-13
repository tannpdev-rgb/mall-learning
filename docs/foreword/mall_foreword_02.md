# 🧩 Những kiến thức cần có để học tốt dự án mall

> **mall KHÔNG phải dự án cho người mới hoàn toàn**.
> Nó là một **dự án tổng hợp**, gom rất nhiều công nghệ backend phổ biến hiện nay.

👉 Vì vậy:

* Tutorial **KHÔNG dạy lại từ đầu từng công nghệ**
* Mà **chỉ dạy phần liên quan trực tiếp đến mall**

Nếu bạn thấy **một số công nghệ còn mơ hồ**, hãy dùng danh sách dưới đây để **bù nền** trước hoặc học song song.

---

## 🧠 Tư duy đúng trước khi học mall

❌ Sai lầm thường gặp:

* “Cứ clone project về rồi đọc code”
* “Không hiểu thì search sau”

✅ Cách đúng:

> **Biết trước mình cần những mảnh ghép nào**, rồi ghép chúng lại bằng mall.

---

## 🛠️ Tài liệu khuyến nghị theo từng mảng

### 💻 IntelliJ IDEA (IDE bạn sẽ dùng hằng ngày)

📘 **IntelliJ IDEA Tutorial**
👉 [https://github.com/judasn/IntelliJ-IDEA-Tutorial](https://github.com/judasn/IntelliJ-IDEA-Tutorial)

**Vì sao cần?**

* mall code nhiều
* refactor liên tục
* debug rất thường xuyên

👉 Đây là **tutorial IDEA đầy đủ nhất**, học xong bạn sẽ:

* code nhanh hơn
* debug ít đau đầu hơn
* dùng IDE như “vũ khí”, không phải “gánh nặng”

---

### 🌱 Spring (nền móng của mọi thứ)

📘 **Spring in Action – Spring实战（第4版）**
👉 [https://book.douban.com/subject/26767354/](https://book.douban.com/subject/26767354/)

**Vì sao nên đọc?**

* Hiểu **IOC, DI, AOP**
* Hiểu **Spring tư duy như thế nào**
* Không còn mù mờ khi thấy @Component, @Bean, @Autowired

👉 **Khuyên đọc toàn bộ**, không cần quá sâu, nhưng phải **có cái nhìn tổng thể**.

---

### 🚀 Spring Boot (xương sống của mall)

📘 **Spring Boot in Action – Spring Boot实战**
👉 [https://book.douban.com/subject/26857423/](https://book.douban.com/subject/26857423/)

**Nhận xét thật:**

* Sách mỏng (~200 trang)
* Đọc nhanh
* Phần Groovy, Grails → **có thể bỏ qua**

👉 Đọc xong bạn sẽ:

* Hiểu vì sao Spring Boot “auto-config”
* Hiểu cấu trúc project mall nhanh hơn rất nhiều

---

### 🗄️ MyBatis (làm việc với Database)

📘 **MyBatis 从入门到精通**
👉 [https://book.douban.com/subject/27074809/](https://book.douban.com/subject/27074809/)

**Vì sao nên học MyBatis trước mall?**

* mall dùng MyBatis **rất nhiều**
* Mapper, XML, Generator xuất hiện liên tục

👉 Đây là sách:

* Viết bởi **tác giả PageHelper**
* Có thể dùng làm **sách tra cứu lâu dài**

---

### 🐬 MySQL (developer cần biết bao nhiêu là đủ?)

📘 **深入浅出 MySQL**
👉 [https://book.douban.com/subject/25817684/](https://book.douban.com/subject/25817684/)

👉 Là dev, bạn **KHÔNG cần** đọc hết.

📌 Chỉ cần:

* Phần **Cơ bản**
* Phần **Phát triển**
* Phần **Tối ưu**

👉 Đủ để:

* đọc schema mall
* hiểu index
* không viết SQL “tự sát hiệu năng”

---

### 🐧 Linux (để deploy được project)

📘 **循序渐进 Linux（第2版）**
👉 [https://book.douban.com/subject/26758194/](https://book.douban.com/subject/26758194/)

👉 Chỉ cần đọc:

* Phần cơ bản
* Phần dựng server

Bạn sẽ dùng Linux khi:

* deploy mall
* chạy Docker
* cấu hình server

---

### 🔍 Elasticsearch (tìm kiếm sản phẩm)

📘 **Elasticsearch 权威指南 (Official Guide)**
👉 [https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html](https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html)

⚠️ Lưu ý:

* Version cũ (2.x)
* Nhưng **tư duy vẫn đúng**

📘 **Elasticsearch 技术解析与实战**
👉 [https://book.douban.com/subject/26967826/](https://book.douban.com/subject/26967826/)

👉 Nếu bạn:

* Thấy guide cũ quá
* Muốn ví dụ thực tế hơn

---

### 🍃 MongoDB (NoSQL trong mall)

📘 **MongoDB 实战（第二版）**
👉 [https://book.douban.com/subject/27061123/](https://book.douban.com/subject/27061123/)

👉 Sách này:

* Viết bởi người từng tham gia phát triển MongoDB driver
* Rất thực tế

Trong mall, MongoDB dùng cho:

* lịch sử xem sản phẩm
* dữ liệu không cần join phức tạp

---

### 🐳 Docker (deploy mall)

📘 **Spring Cloud 与 Docker 微服务架构实战**
👉 [https://book.douban.com/subject/27028228/](https://book.douban.com/subject/27028228/)

👉 Chỉ cần đọc:

* Phần **Docker**
* Không cần đọc Spring Cloud

Docker giúp bạn:

* chạy mall nhanh
* deploy dễ
* giống môi trường production

---

## 🎯 Tổng kết (rất quan trọng)

> Nếu bạn:

* Đã đọc **một phần** các tài liệu trên
  hoặc
* Đã có **nền tảng tương đương**

👉 **Việc học mall sẽ rất mượt**.

❗ mall không khó vì code
❗ mall khó vì **thiếu nền tảng tổng hợp**

---

## 📌 Gợi ý lộ trình học mall (ngắn gọn)

1️⃣ Có nền Spring + Spring Boot
2️⃣ Biết MyBatis + MySQL cơ bản
3️⃣ Biết Redis, ES, MQ ở mức khái niệm
4️⃣ Vào mall → học theo **kiến trúc + nghiệp vụ**

---

## 📢 公众号

Học không đi đường vòng, theo dõi公众号 **macrozheng**
👉 Trả lời **「学习路线」** để nhận **lộ trình học mall chi tiết**

![公众号图片](http://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/banner/qrcode_for_macrozheng_258.jpg)
