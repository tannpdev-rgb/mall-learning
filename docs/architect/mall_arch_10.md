## 📚 Học tập không đi đường vòng

👉 **[Theo dõi公众号](#公众号)** và **trả lời “学习路线”** để nhận **lộ trình học RIÊNG cho dự án mall**!

---

# ☁️ Dự án mall: Tích hợp OSS để upload file (chuẩn production)

> Bài viết này sẽ **dẫn bạn từng bước** tích hợp **Alibaba Cloud OSS** vào dự án mall
> với cách làm **đúng chuẩn hệ thống lớn**:
>
> 👉 **Server ký – Frontend upload trực tiếp lên OSS**

💡 Head First nói thẳng:

> *File upload KHÔNG nên đi qua server backend.*

---

## 🧩 1. OSS là gì?

> **OSS (Object Storage Service)** là dịch vụ lưu trữ đối tượng của Alibaba Cloud,
> dùng để lưu:
>
> * Ảnh sản phẩm
> * Video
> * Log
> * File dung lượng lớn

👉 Đặc điểm:

* Rẻ
* Nhanh
* Bền
* Scale vô hạn

💡 Head First nhớ:

> *Database lưu dữ liệu – OSS lưu file*

---

## 🧠 2. Các khái niệm OSS bắt buộc phải nhớ

| Khái niệm | Hiểu đơn giản           |
| --------- | ----------------------- |
| Endpoint  | Domain truy cập OSS     |
| Bucket    | Thư mục gốc (container) |
| Object    | File                    |
| AccessKey | Chìa khóa truy cập      |

👉 Tất cả file **bắt buộc nằm trong một Bucket**

---

## ⚙️ 3. Cấu hình OSS trên Alibaba Cloud

### 🔓 Bước 1: Mở dịch vụ OSS

* Đăng nhập Alibaba Cloud
* Products → Object Storage OSS
* Click **Mở dịch vụ**

---

### 🪣 Bước 2: Tạo Bucket

![Image](https://yqintl.alicdn.com/dc99ab3b3df93b522092df2d0ebbcccb7cf0252a.png)

![Image](https://yqintl.alicdn.com/5b39f4ab6fc601268964075d61708ff5a29a92c8.png)

* Chọn khu vực
* Quyền truy cập: **Public Read**

![Image](https://yqintl.alicdn.com/2be3ec83d58adf7d3d14bcb4e89dbd734ce111b5.png)

![Image](https://yqintl.alicdn.com/a389cfae9b0609ff8cf9630dfd4612d338e32b33.png)

---

### 🌍 Bước 3: Cấu hình CORS (rất quan trọng)

> ❗ Nếu không cấu hình CORS → frontend upload sẽ **BỊ CHẶN**

![Image](https://help-static-aliyun-doc.aliyuncs.com/assets/img/en-US/8104597571/p1007027.png)

![Image](https://docs.cloudreve.org/assets/oss-cors.CycyeU05.png)

👉 Cho phép:

* Origin: `*`
* Method: `POST, GET`
* Header: `*`

💡 Head First:

> *Frontend upload trực tiếp = bắt buộc CORS*

---

## 🔁 4. Vì sao dùng “Server ký – Frontend upload”?

### ❌ Cách NGU (không dùng)

```
Frontend → Backend → OSS
```

* Backend tốn băng thông
* Backend dễ chết khi upload file lớn

---

### ✅ Cách ĐÚNG (production)

![Image](https://docs.aws.amazon.com/images/solutions/latest/data-transfer-hub/images/guidance-arch.png)

![Image](https://media.licdn.com/dms/image/v2/D4D12AQEkQEmek4G_hA/article-cover_image-shrink_720_1280/B4DZlPZ8ziJEAI-/0/1757973827782?e=2147483647\&t=RG8wSzied7VSkfmQw7mfofciVPGbQcnJ2eotctMiimM\&v=beta)

```
Frontend → Backend (xin chữ ký)
Frontend → OSS (upload)
OSS → Backend (callback)
```

💡 Head First chốt:

> *Backend chỉ ký – không upload hộ*

---

## 📦 5. Thêm dependency OSS

```xml
<dependency>
  <groupId>com.aliyun.oss</groupId>
  <artifactId>aliyun-sdk-oss</artifactId>
  <version>2.5.0</version>
</dependency>
```

---

## ⚙️ 6. Cấu hình OSS trong `application.yml`

```yml
aliyun:
  oss:
    endpoint: oss-cn-shenzhen.aliyuncs.com
    accessKeyId: xxx
    accessKeySecret: xxx
    bucketName: macro-oss
    policy:
      expire: 300
    maxSize: 10
    callback: http://localhost:8080/aliyun/oss/callback
    dir:
      prefix: mall/images/
```

💡 Head First:

> *callback phải là PUBLIC URL*

---

## 🧱 7. Tạo OSSClient

```java
@Bean
public OSSClient ossClient() {
  return new OSSClient(endpoint, accessKeyId, accessKeySecret);
}
```

👉 OSSClient = cổng giao tiếp với OSS

---

## 📄 8. DTO cho upload OSS

### 🧾 OssPolicyResult – trả cho frontend

```java
accessKeyId
policy
signature
dir
host
callback
```

👉 Frontend **KHÔNG CẦN biết secret**

---

### 🔁 OssCallbackParam – OSS gọi ngược về server

```java
callbackUrl
callbackBody
callbackBodyType
```

---

### 📦 OssCallbackResult – kết quả upload

```java
filename
size
mimeType
width
height
```

---

## 🧠 9. OssService – logic ký upload

### Sinh chữ ký upload

```java
PolicyConditions policyConds = new PolicyConditions();
policyConds.addConditionItem(
  PolicyConditions.COND_CONTENT_LENGTH_RANGE,
  0, maxSize
);
```

👉 Giới hạn size file
👉 Giới hạn thư mục upload

💡 Head First:

> *Chữ ký = luật chơi upload*

---

### Xử lý callback từ OSS

```java
String filename = request.getParameter("filename");
```

👉 OSS trả info file về server
👉 Server trả lại frontend

---

## 🌐 10. OssController – API upload

### API lấy chữ ký

```
GET /aliyun/oss/policy
```

---

### API callback

```
POST /aliyun/oss/callback
```

---

## 🧪 11. Test upload thực tế

### Test API ký upload

![Image](https://swagger.io/getmedia/7bd69649-e725-4342-bda0-b68b7b00bc4f/SwaggerHub-UI-Example?height=366\&width=800)

![Image](https://i.sstatic.net/5kOcg.png)

---

### Frontend upload file

![Image](https://payloadcms.com/images/docs/uploads-overview.jpg)

![Image](https://img.alicdn.com/imgextra/i4/O1CN01huK2US1yegdIQXVmW_%21%216000000006604-0-tps-1920-1080.jpg)

Luồng request:

1. Frontend → backend xin policy
2. Frontend → OSS upload file
3. OSS → backend callback

🎉 Backend **KHÔNG hề upload file**

---

## 📦 Source code dự án

🔗 GitHub:
[https://github.com/macrozheng/mall-learning/tree/master/mall-tiny-09](https://github.com/macrozheng/mall-learning/tree/master/mall-tiny-09)

---

## 📢 公众号

![Image](https://opengraph.githubassets.com/0e4358626612706b3d9867e82818afa40c744572ddb56dcd795566d96379e1ae/macrozheng/mall)

![Image](https://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/banner/qrcode_for_macrozheng_258.jpg)

👉 Theo dõi để:

* Hiểu **OSS chuẩn production**
* Upload file **không nghẽn backend**
* Không đi đường vòng ❌
