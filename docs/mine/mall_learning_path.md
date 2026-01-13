# 🚀 Lộ trình học dự án thực chiến Mall – Gom trọn công nghệ chủ lưu!

> Có rất nhiều bạn từng hỏi mình:
> 👉 *“Dự án mall nên học như thế nào?”*
> 👉 *“Học theo thứ tự nào thì không bị loạn?”*

Thông thường, mình sẽ gửi cho họ **mục lục của “mall 学习教程”** và bảo:
➡️ *Cứ học lần lượt theo thứ tự đó là được.*

Vì sao?
👉 Vì mình luôn tin rằng **học thông qua dự án thực tế** là cách tốt nhất để nắm vững công nghệ.

Mall là một **dự án tổng hợp**:

* Vừa học được **công nghệ chủ lưu**
* Vừa có **kinh nghiệm dự án thực chiến**

Nhưng…
⚠️ Nếu học **không có lộ trình rõ ràng**, bạn sẽ:

* Bị ngợp
* Học lan man
* Không biết mình đang học để làm gì

👉 Vì vậy, mình đã tổng hợp lại **một lộ trình học mall tối ưu hơn**, giúp bạn **ít đi đường vòng nhất có thể**.

---

## 📚 推荐资料 – Tài liệu nên chuẩn bị trước

Mall sử dụng **rất nhiều công nghệ hiện đại**.
Nếu bạn là **người mới học Java**, lời khuyên là:

> ❗ Đừng lao vào mall ngay
> 👉 Hãy bù nền tảng trước

Bạn có thể tham khảo phần giới thiệu chi tiết tại:
[mall学习所需知识点](../foreword/mall_foreword_01.md)

![学习资料](../images/mall_learning_path_01.svg)

> 🎯 Mục tiêu của bước này:
>
> * Đọc code **không sợ**
> * Biết Spring Boot + MyBatis cơ bản
> * Hiểu Redis / ES / MQ ở mức khái niệm

---

## ⚙️ 学习后端技术栈 – Học toàn bộ backend tech stack của Mall

Nếu bạn đã có **nền tảng Java tương đối ổn**, thì:

> 👉 Cứ học thẳng những công nghệ **mall đang dùng**

### ❓ Vì sao phải học công nghệ trước, chưa học nghiệp vụ vội?

Head First nói thế này 👇

> **Nghiệp vụ thì dự án nào cũng khác**
> **Nhưng công nghệ thì gần như giống nhau**

👉 Khi học open-source project:

* Quan trọng nhất: **công nghệ**
* Nghiệp vụ: **xếp sau**

Trước tiên, hãy xem **mall dùng những công nghệ nào**:

![学习后端技术栈](../images/mall_learning_path_02.svg)

Tin tốt là 🎉
👉 Trong **《mall学习教程》参考篇**, gần như **mọi công nghệ đều đã có bài hướng dẫn riêng**.

📌 Gặp công nghệ chưa biết?
➡️ Cứ quay lại **参考篇** mà học.

---

### 🧱 Công nghệ dựng khung dự án

* [Spring Boot入门教程](../reference/springboot_start.md)
* [Spring Boot整合MyBatis，并使用MyBatis Generator生成代码](../reference/mybatis_generator_start.md)
* [Spring Boot整合Swagger使用教程](../reference/swagger_starter.md)
* [Lombok使用教程](../reference/lombok_start.md)
* [Hutool使用教程](../reference/hutool_start.md)

👉 Đây là **xương sống** của toàn bộ dự án.

---

### 🗄️ Công nghệ lưu trữ dữ liệu

* [常用MySQL命令整理](../reference/mysql.md)
* [Spring Boot整合Redis使用教程](../reference/spring_data_redis.md)
* [Elasticsearch入门教程](../reference/elasticsearch_start.md)
* [MongoDB入门教程](../reference/mongodb_start.md)
* [MinIO入门教程](../reference/minio.md)

👉 Không cần học thuộc API
👉 Chỉ cần:

* Biết dùng
* Biết khi nào nên dùng

---

### 🚢 Công nghệ vận hành & triển khai

* [在虚拟机中安装使用Linux的教程](../reference/linux_install.md)
* [常用Linux命令整理](../reference/linux_command.md)
* [常用Docker命令整理](../reference/docker_command.md)
* [使用Maven插件为Spring Boot应用构建Docker镜像](../reference/docker_maven.md)
* [使用Docker Compose部署SpringBoot应用](../reference/docker_compose.md)
* [Nginx使用教程](../reference/nginx.md)
* [Nginx支持HTTPS](../reference/nginx_https_start.md)
* [使用Jenkins自动化部署Spring Boot应用](../reference/jenkins.md)
* [使用Jenkins自动化部署Vue前端应用](../reference/jenkins_vue.md)

👉 Đây là **ranh giới giữa coder và developer thực chiến**.

---

### 📡 Công nghệ khác được dùng trong Mall

* [RabbitMQ使用教程](../reference/rabbitmq_start.md)
* [ELK日志收集系统搭建教程](../reference/mall_elk_advance.md)
* [Kibana设置密码保护教程](../reference/elk_security.md)

---

## 🏗️ 搭建项目骨架 – Tự tay dựng “bộ xương” dự án

Head First nói thế này 👇

> **Bạn chỉ thực sự “biết” một công nghệ khi bạn tự tay dựng nó**

Trong thực tế:

* Trước khi code tính năng
* Ta luôn **dựng project skeleton**

Khi bạn:

* Tự dựng được skeleton
* Dùng nó để viết vài chức năng

👉 Bạn đã **thực sự làm chủ công nghệ đó**

《mall学习教程》架构篇 chính là:

> 👉 Một bộ hướng dẫn **từng bước dựng skeleton của mall**

![搭建项目骨架](../images/mall_learning_path_03.svg)

### 📌 Thứ tự dựng khung (rất quan trọng!)

* [mall整合SpringBoot+MyBatis搭建基本骨架](../architect/mall_arch_01.md)
* [mall整合Swagger-UI实现在线API文档](../architect/mall_arch_02.md)
* [mall整合Redis实现缓存功能](../architect/mall_arch_03.md)
* [mall整合SpringSecurity和JWT实现认证和授权（一）](../architect/mall_arch_04.md)
* [mall整合SpringSecurity和JWT实现认证和授权（二）](../architect/mall_arch_05.md)
* [mall整合SpringTask实现定时任务](../architect/mall_arch_06.md)
* [mall整合Elasticsearch实现商品搜索](../architect/mall_arch_07.md)
* [mall整合Mongodb实现文档操作](../architect/mall_arch_08.md)
* [mall整合RabbitMQ实现延迟消息](../architect/mall_arch_09.md)
* [mall整合OSS实现文件上传](../architect/mall_arch_10.md)

👉 Học xong phần này:

* Bạn **không còn sợ project lớn**
* Bạn **biết cách bắt đầu một backend project chuẩn**

---

## 🚀 项目部署 – Deploy dự án cho chạy thật

Head First nhắc lại 👇

> ❌ Code chưa chạy = chưa học xong

Sau khi học xong kiến trúc, bạn có thể **chạy mall bằng nhiều cách**:

![项目部署](../images/mall_learning_path_04.svg)

### Backend

* [mall在Windows环境下的部署](../deploy/mall_deploy_windows.md)
* [mall在Linux环境下的部署（基于Docker容器）](../deploy/mall_deploy_docker.md)
* [mall在Linux环境下的部署（基于Docker Compose）](../deploy/mall_deploy_docker_compose.md)
* [mall在Linux环境下的自动化部署（基于Jenkins）](../deploy/mall_deploy_jenkins.md)

### Frontend

* [mall前端项目的安装与部署](../deploy/mall_deploy_web.md)

---

## 🛒 学习电商业务 – Hiểu nghiệp vụ thương mại điện tử

Khi dự án đã chạy:

> ❌ Đừng vội đọc code
> ✅ Hãy **dùng chức năng trước**

Cách học đúng:

1. Bấm chức năng
2. Hiểu chức năng
3. So với bảng DB

![电商业务](../images/mall_learning_path_05.svg)

👉 Nếu bạn:

* Nhìn bảng DB
* Đoán được chức năng

➡️ Bạn đã **hiểu nghiệp vụ thật**

---

## 🔍 解析技术要点 – Đọc source code đúng cách

Sau khi hiểu nghiệp vụ, lúc này mới:

> 👉 Đọc source code

Cách đọc hiệu quả:

```
Dùng chức năng
→ Bắt API
→ Controller
→ Service
→ Mapper
```

Thứ tự gợi ý:

```
权限 → 商品 → 订单 → 营销
```

![技术要点](../images/mall_learning_path_06.svg)

---

## 🎨 学习前端技术栈 – Nếu bạn muốn làm fullstack

Mall admin frontend:

* Vue
* Element-UI

![前端技术栈](../images/mall_learning_path_07.svg)

👉 Không cần học hết frontend
👉 Chỉ cần:

* Hiểu Vue cơ bản
* Đọc module **quyền**

---

## ☁️ 进阶微服务 – Nâng cấp lên Microservices

Java backend hiện đại **không thể không biết microservice**.

Mall có bản:

* **mall-swarm**
* Dựa trên **Spring Cloud & Alibaba**

![进阶微服务](../images/mall_learning_path_08.svg)

👉 Phần này dành cho:

* Người muốn lên **senior**
* Người muốn học hệ thống phân tán

---

## 🧰 开发工具使用 – Công cụ = sức mạnh

![开发工具](../images/mall_learning_path_09.svg)

* IDEA
* Git
* Navicat
* Postman
* Arthas
* Redis Desktop
* DataGrip

👉 Tool tốt = tăng x2 hiệu suất

---

## 🌱 扩展学习 – Mở rộng kiến thức

![扩展学习](../images/mall_learning_path_10.svg)

* MySQL nâng cao
* MyBatis nâng cao
* Log system
* Docker nâng cao
* Redis cluster
* MQ nâng cao
* Job scheduler

---

## 🏁 总结 – Tổng kết

《mall学习教程》 đã có **130+ bài viết gốc**.

👉 Nó không chỉ là:

* Tutorial cho 1 dự án

👉 Mà là:

* **Giáo trình Java backend thực chiến**

Học xong mall:

* Bạn không sợ dự án lớn
* Bạn biết bắt đầu từ đâu
* Bạn đủ tự tin viết dự án riêng

---

## 📦 项目地址

* mall:[https://github.com/macrozheng/mall](https://github.com/macrozheng/mall)
* mall-admin-web:[https://github.com/macrozheng/mall-admin-web](https://github.com/macrozheng/mall-admin-web)
* mall-learning:[https://github.com/macrozheng/mall-learning](https://github.com/macrozheng/mall-learning)
* mall-swarm:[https://github.com/macrozheng/mall-swarm](https://github.com/macrozheng/mall-swarm)

---

## 🧠 完整思维导图

![mall学习路线](../images/mall_learning_path_11.svg)
