Học tập **không đi đường vòng** 🧭
👉 [Theo dõi公众号](#公众号) và **trả lời “学习路线”** để nhận **lộ trình học riêng cho dự án mall**!

---

# Triển khai mall trên Linux (dựa trên Docker Container)

> Bài viết này trình bày **toàn bộ quá trình triển khai hệ thống mall trên Linux**,
> sử dụng **Docker** để chạy các middleware và **Spring Boot application**,
> môi trường nền là **CentOS 7.6**.

🧠 **Head First mindset**
Đây **không phải** là “chạy vài container cho vui” ❌
Đây là **một kiến trúc production-level thu nhỏ** ✅

---

## Bức tranh tổng thể trước khi bắt đầu

Hãy nhìn hệ thống mall như sau 👇

```
[Browser / App]
       |
     Nginx
       |
-------------------------
| admin | portal | search |
-------------------------
 |   |     |       |
MySQL Redis Mongo  ES
           |
        RabbitMQ
```

👉 Mỗi thành phần:

* chạy **trong container riêng**
* có **volume riêng**
* có thể **restart độc lập**

---

## 1️⃣ Cài đặt Docker Environment

### Vì sao phải dùng Docker?

🧠 Head First trả lời:

* Không phải cài MySQL / Redis trực tiếp trên OS
* Không lo xung đột version
* Dễ migrate sang server khác

---

### Cài các gói cần thiết

```bash
yum install -y yum-utils device-mapper-persistent-data lvm2
```

👉 Chuẩn bị môi trường cho Docker storage.

---

### Thêm Docker repo

```bash
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

👉 Để `yum` biết **cài Docker ở đâu**.

---

### Cài & khởi động Docker

```bash
yum install docker-ce
systemctl start docker
```

🧠 Nếu Docker chưa chạy → **mọi bước sau đều vô nghĩa**.

---

## 2️⃣ MySQL – “Trái tim dữ liệu”

### Kéo image MySQL 5.7

```bash
docker pull mysql:5.7
```

👉 Version 5.7 ổn định, tương thích tốt với mall.

---

### Chạy container MySQL

```bash
docker run -p 3306:3306 --name mysql \
-v /mydata/mysql/log:/var/log/mysql \
-v /mydata/mysql/data:/var/lib/mysql \
-v /mydata/mysql/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=root  \
-d mysql:5.7
```

### 🧠 Head First: tại sao phải mount volume?

| Thư mục | Ý nghĩa               |
| ------- | --------------------- |
| data    | dữ liệu DB (sống còn) |
| log     | log để debug          |
| conf    | config tùy chỉnh      |

👉 Xóa container **KHÔNG mất dữ liệu**.

---

### Import database mall

* Tạo DB
* Import `mall.sql`
* Tạo user `reader`

🧠 **Quan trọng**:
User `reader@%` → cho phép **container khác truy cập MySQL**

---

## 3️⃣ Redis – Cache & Session

```bash
docker pull redis:5
```

```bash
docker run -p 6379:6379 --name redis \
-v /mydata/redis/data:/data \
-d redis:5 redis-server --appendonly yes
```

🧠 Redis dùng cho:

* cache
* token
* verification code
* giảm tải DB

---

## 4️⃣ Nginx – Cổng vào hệ thống

🧠 Head First:
**User không bao giờ gọi thẳng Spring Boot**.

---

### Chạy Nginx & tách config ra host

👉 Mục tiêu:

* config dễ sửa
* reload không rebuild image

```bash
docker container cp nginx:/etc/nginx /mydata/nginx/
```

👉 Sau đó chạy lại với volume mount.

---

## 5️⃣ RabbitMQ – Message Queue

🧠 RabbitMQ dùng cho:

* async order
* stock unlock
* email / notification

---

### Chạy RabbitMQ + Management UI

```bash
docker pull rabbitmq:3.7.15
docker run -p 5672:5672 -p 15672:15672 --name rabbitmq \
-d rabbitmq:3.7.15
```

👉 `15672` = web quản lý
👉 `5672` = service queue

---

### Tạo user & vhost riêng cho mall

🧠 **Best practice**:

* Không dùng `guest`
* Mỗi project → 1 vhost

---

## 6️⃣ Elasticsearch – Tìm kiếm

```bash
docker pull elasticsearch:7.6.2
```

### Fix lỗi memory

```bash
sysctl -w vm.max_map_count=262144
```

👉 **Bước này rất hay quên** → ES không start.

---

### Chạy ES

```bash
docker run -p 9200:9200 -p 9300:9300 --name elasticsearch \
-e "discovery.type=single-node" \
-v /mydata/elasticsearch/data:/usr/share/elasticsearch/data \
-d elasticsearch:7.6.2
```

---

### Cài IK Analyzer (bắt buộc cho tiếng Trung)

👉 Nếu không có IK:

* search tiếng Trung = sai

---

## 7️⃣ Logstash + Kibana – Logging & Monitoring

🧠 Đây là **ELK Stack**:

* Logstash → thu log
* Elasticsearch → lưu log
* Kibana → xem log

👉 **Không phải trang trí**, dùng để:

* debug
* audit
* production issue

---

## 8️⃣ MongoDB – Dữ liệu linh hoạt

```bash
docker pull mongo:4.2.5
docker run -p 27017:27017 --name mongo \
-v /mydata/mongo/db:/data/db \
-d mongo:4.2.5
```

🧠 Mongo dùng cho:

* browse history
* member attention
* data không cần schema cứng

---

## 9️⃣ Tổng kết Middleware

👉 Tại thời điểm này, bạn đã có:

| Thành phần    | Trạng thái |
| ------------- | ---------- |
| MySQL         | running    |
| Redis         | running    |
| RabbitMQ      | running    |
| Elasticsearch | running    |
| Logstash      | running    |
| Kibana        | running    |
| MongoDB       | running    |
| Nginx         | running    |

🧠 **Đây là một hệ sinh thái hoàn chỉnh**.

---

## 🔟 Deploy Spring Boot Applications

### Build Docker Image bằng Maven

👉 `docker-maven-plugin`:

* build image
* push image
* không cần viết Dockerfile thủ công

🧠 **Rất enterprise**.

---

### Deploy từng service

| Service     | Port | Vai trò        |
| ----------- | ---- | -------------- |
| mall-admin  | 8080 | backend admin  |
| mall-search | 8081 | search service |
| mall-portal | 8085 | user API       |

---

### Ví dụ: mall-admin

```bash
docker run -p 8080:8080 --name mall-admin \
--link mysql:db \
--link redis:redis \
-d mall/mall-admin
```

🧠 `--link` giúp container:

* resolve hostname
* không cần IP cứng

---

## 1️⃣1️⃣ Test toàn hệ thống

👉 Swagger UI là **điểm kiểm tra cuối cùng**.

| Service | URL                |
| ------- | ------------------ |
| admin   | `/swagger-ui.html` |
| search  | `/swagger-ui.html` |
| portal  | `/swagger-ui.html` |

Nếu Swagger load được → **service OK**.

---

## 🧠 Tổng kết Head First (rất quan trọng)

> Nếu bạn hiểu được 5 điều này, bạn đã **vượt level junior**:

1️⃣ Docker = **môi trường chuẩn hóa**
2️⃣ Volume = **bảo vệ dữ liệu**
3️⃣ Middleware = **tách trách nhiệm**
4️⃣ Service = **deploy độc lập**
5️⃣ Swagger = **health check nhanh nhất**

---

## 公众号

![公众号图片](http://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/banner/qrcode_for_macrozheng_258.jpg)
