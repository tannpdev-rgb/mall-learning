Học tập **không đi đường vòng** 🧭
👉 [Theo dõi公众号](#公众号) và **trả lời “学习路线”** để nhận **lộ trình học riêng cho dự án mall**!

---

# Triển khai tự động mall trên Linux (dựa trên Jenkins)

> Bài viết này trình bày **cách mall triển khai tự động bằng Jenkins**,
> áp dụng cho **dự án Spring Boot đa module**,
> theo phong pháp **CI/CD thực tế trong doanh nghiệp**.

🧠 **Head First mindset**
Jenkins **không phải** chỉ để “build cho vui” ❌
Jenkins là **robot thay bạn deploy** ✅

---

## Bức tranh tổng thể: Jenkins đang làm việc gì?

Hãy tưởng tượng luồng sau 👇

```
Dev push code
     ↓
Jenkins pull code
     ↓
Build dependency modules
     ↓
Build service module
     ↓
SSH sang server
     ↓
Stop container cũ
     ↓
Run container mới
```

👉 Mỗi bước = **1 việc cụ thể**
👉 Jenkins chỉ làm đúng những gì bạn dạy nó

---

## 1️⃣ Jenkins – kiến thức nền

> Phần kiến thức cơ bản về Jenkins có thể xem tại:
> [使用Jenkins一键打包部署SpringBoot应用，就是这么6！]

🧠 Ở đây **KHÔNG lặp lại kiến thức nhập môn**,
chỉ tập trung vào **cách áp dụng Jenkins cho mall**.

---

## 2️⃣ Chuẩn bị script triển khai (rất quan trọng)

> Jenkins **không tự deploy được** nếu không có script.

---

### 📁 Thư mục script

* Toàn bộ script nằm trong:

```text
mall/document/sh
```

👉 Mỗi service có **1 script riêng**:

* `mall-admin.sh`
* `mall-portal.sh`
* `mall-search.sh`

🧠 **Tư duy chuẩn**:

> “Jenkins gọi script, script làm việc thật”

---

### ⚠️ Lỗi rất hay gặp: sai format dòng

Trước khi upload script:

👉 **BẮT BUỘC đổi line separator sang `LF`**

Nếu không:

* Jenkins SSH sang Linux
* Script **chạy không được**
* Lỗi rất khó hiểu 😵

![](../images/mall_deploy_jenkins_01.png)

---

### Upload script lên server

* Upload toàn bộ script lên:

```text
/mydata/sh
```

![](../images/mall_deploy_jenkins_02.png)

---

### Cấp quyền thực thi

```bash
chmod +x ./mall-*
```

🧠 Linux không có quyền execute → **script = file thường**

![](../images/mall_deploy_jenkins_03.png)

---

## 3️⃣ Tạo Jenkins Job cho mall (multi-module)

> Vì `mall` là **dự án đa module**,
> nên **KHÔNG thể build 1 module đơn lẻ ngay**.

🧠 **Nguyên tắc sống còn**:

> 👉 **Build dependency trước, service sau**

---

## 4️⃣ Tạo job cho `mall-admin` (giải thích chi tiết)

### Bước 1: Tạo job

* Chọn:

```text
构建一个自由风格的软件项目
```

* Đặt tên: `mall-admin`
* Cấu hình Git repository

![](../images/mall_deploy_jenkins_04.png)

---

### Bước 2: Build các module phụ thuộc

```bash
clean install -pl mall-common,mall-mbg,mall-security -am
```

🧠 **Head First giải thích câu lệnh này**:

| Tham số         | Ý nghĩa                       |
| --------------- | ----------------------------- |
| `-pl`           | chỉ build module chỉ định     |
| `-am`           | build cả dependency của chúng |
| `clean install` | build & cài vào local repo    |

👉 Nếu **bỏ bước này** →
`mall-admin` sẽ **build FAIL**

![](../images/mall_deploy_jenkins_05.png)

---

### Bước 3: Build riêng module mall-admin

* Chỉ định đúng `pom.xml` của mall-admin

![](../images/mall_deploy_jenkins_06.png)

🧠 **Tư duy chuẩn**:

> “Dependency build 1 lần – service build riêng”

---

### Bước 4: SSH sang server để deploy

* Thêm **SSH Execute task**
* Chạy script:

```text
/mydata/sh/mall-admin.sh
```

![](../images/mall_deploy_jenkins_07.png)

🧠 Script này thường làm:

1. Stop container cũ
2. Xóa container cũ
3. Run container mới

---

### Bước 5: Lưu job

👉 `mall-admin` job hoàn tất 🎉

---

## 5️⃣ Tạo job cho `mall-portal`

> `mall-portal` **giống 90% mall-admin**

🧠 **Đừng làm lại từ đầu** – hãy copy job.

---

### Copy từ mall-admin

![](../images/mall_deploy_jenkins_08.png)

---

### Sửa pom.xml

```text
${WORKSPACE}/mall-portal/pom.xml
```

![](../images/mall_deploy_jenkins_09.png)

---

### Sửa script SSH

```text
/mydata/sh/mall-portal.sh
```

![](../images/mall_deploy_jenkins_10.png)

---

### Lưu job

👉 `mall-portal` xong ✅

---

## 6️⃣ mall-search

👉 Làm **y hệt mall-admin & mall-portal**
👉 Chỉ khác:

* pom.xml
* script deploy

---

## 7️⃣ Hoàn tất các job Jenkins

![](../images/mall_deploy_jenkins_11.png)

🧠 **Tại thời điểm này**:

* Push code
* Bấm Build
* Jenkins tự deploy

👉 **Không SSH tay**
👉 **Không gõ docker run thủ công**

---

## 8️⃣ Vì sao cách này “chuẩn doanh nghiệp”?

🧠 Head First tổng kết:

1️⃣ Multi-module → build có thứ tự
2️⃣ Script tách riêng → dễ sửa, dễ debug
3️⃣ Jenkins chỉ orchestration → không ôm logic
4️⃣ SSH deploy → phù hợp server on-premise
5️⃣ Copy job → tiết kiệm 70% thời gian

---

## 9️⃣ Dự án tham khảo

🔗 [https://github.com/macrozheng/mall](https://github.com/macrozheng/mall)

---

## 公众号

![公众号图片](http://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/banner/qrcode_for_macrozheng_258.jpg)
