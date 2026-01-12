## 📚 Học tập không đi đường vòng

👉 **[Theo dõi公众号](#公众号)** và **trả lời “学习路线”** để nhận **lộ trình học RIÊNG cho dự án mall**!

---

# 🗄️ Toàn cảnh **cấu trúc database** của dự án mall

> **mall** là một hệ thống **thương mại điện tử hoàn chỉnh**.
> Backend của mall bao gồm các module lớn:
>
> * Quản lý sản phẩm
> * Quản lý đơn hàng
> * Quản lý marketing / khuyến mãi
> * Quản lý nội dung
> * Quản lý người dùng

💡 Head First nói trước cho bạn khỏi sốc:

> *Nếu chưa hiểu database → đừng vội đọc service, controller.*

---

## 🛒 1. Quản lý sản phẩm (Product Management)

### 📦 Cấu trúc bảng dữ liệu

![Image](https://svg.template.creately.com/jiy85go8)

![Image](https://databasesample.com/_next/image?q=75\&url=%2Fdatabase%2Fshopping-mall-database.png\&w=3840)

👉 Nhóm bảng này xoay quanh:

* Sản phẩm (product)
* Thương hiệu (brand)
* Danh mục (category)
* Thuộc tính & SKU

💡 Head First hiểu nhanh:

> *1 sản phẩm = nhiều SKU = nhiều thuộc tính*

---

### 🧠 Cấu trúc chức năng

![Image](https://www.conceptdraw.com/How-To-Guide/picture/Blockdiagrams-Portersfiveforcesmodel.png)

![Image](https://www.researchgate.net/publication/239917579/figure/fig2/AS%3A298603491938305%401448204028068/Activity-Diagram-of-Shopping-Mall-Automation-System.png)

Bao gồm:

* Quản lý SP
* Quản lý thương hiệu
* Quản lý danh mục
* Quản lý thuộc tính

👉 Đây là **module lớn nhất & phức tạp nhất** trong mall.

---

## 📦 2. Quản lý đơn hàng (Order Management)

### 🧾 Cấu trúc bảng dữ liệu

![Image](https://databasesample.com/_next/image?q=75\&url=%2Fdatabase%2Fshopping-mall-database.png\&w=3840)

![Image](https://svg.template.creately.com/jiy85go8)

Bao gồm:

* Đơn hàng
* Chi tiết đơn hàng
* Trạng thái đơn
* Thanh toán
* Giao hàng

💡 Head First nhớ:

> *Order = trung tâm của cả hệ thống*

---

### 🔄 Cấu trúc chức năng

![Image](https://add2cart.blog/wp-content/uploads/2021/05/image-7.png?w=602)

![Image](https://www.slideteam.net/media/catalog/product/cache/1280x720/r/e/retail_store_order_management_process_flow_chart_slide01.jpg)

Luồng chính:

```
Tạo đơn → Thanh toán → Giao hàng → Hoàn tất / Hủy
```

👉 Tất cả logic **RabbitMQ, Redis, Delay Message** đều xoay quanh module này.

---

## 🎯 3. Quản lý marketing / khuyến mãi

### 🧮 Cấu trúc bảng dữ liệu

![Image](https://svg.template.creately.com/hjrrpwcg)

![Image](https://databasesample.com/_next/image?q=75\&url=%2Fdatabase%2Fshopping-mall-database.png\&w=3840)

Bao gồm:

* Coupon
* Promotion
* Flash sale
* Chiến dịch marketing

💡 Head First:

> *Marketing = nhiều luật + nhiều ràng buộc*

---

### 🧠 Cấu trúc chức năng

![Image](https://s3-us-west-2.amazonaws.com/courses-images/wp-content/uploads/sites/2986/2018/04/05220403/RetailMix_update.png)

![Image](https://www.researchgate.net/publication/331452187/figure/fig1/AS%3A731727462924289%401551468830180/Hierarchical-structure-of-shopping-mall-performance-index-main-criteria.png)

👉 Module này:

* Ít CRUD
* Nhiều logic nghiệp vụ
* Hay dùng cache (Redis)

---

## 📰 4. Quản lý nội dung (CMS)

### 📄 Cấu trúc bảng dữ liệu

![Image](https://drawsql-media.s3-us-east-2.amazonaws.com/screenshots/59736/conversions/1609368747-3468-thumbnail.jpg)

![Image](https://media.geeksforgeeks.org/wp-content/uploads/20240305153157/cmsDBModel.webp)

Bao gồm:

* Banner
* Quảng cáo
* Trang nội dung
* Đề xuất sản phẩm

💡 Head First:

> *CMS = phục vụ frontend hiển thị*

---

### 🧠 Cấu trúc chức năng

![Image](https://support.intuiface.com/hc/article_attachments/5283319950620)

![Image](https://www.researchgate.net/publication/239917579/figure/fig2/AS%3A298603491938305%401448204028068/Activity-Diagram-of-Shopping-Mall-Automation-System.png)

👉 Dữ liệu:

* Ít thay đổi
* Đọc nhiều
* Rất hợp dùng cache

---

## 👤 5. Quản lý người dùng (User Management)

### 🧑‍💻 Cấu trúc bảng dữ liệu

![Image](https://svg.template.creately.com/hm75n5d81)

![Image](https://svg.template.creately.com/iwi0s7z2)

Bao gồm:

* Người dùng
* Vai trò (role)
* Quyền (permission)
* Quan hệ user–role–permission

💡 Head First:

> *Spring Security + JWT = xoay quanh nhóm bảng này*

---

### 🔐 Cấu trúc chức năng

![Image](https://svg.template.creately.com/jiy85go8)

![Image](https://svg.template.creately.com/if5gv7jv3)

👉 Phục vụ cho:

* Phân quyền backend
* Kiểm soát API
* Bảo mật hệ thống

---

## ⚠️ Lưu ý quan trọng

> ❗ **Không phải toàn bộ chức năng đều đã implement**

* Một số bảng: **chỉ thiết kế**
* Các module đã hoàn thiện tốt:

  * ✅ Sản phẩm
  * ✅ Đơn hàng
  * ✅ Marketing

💡 Head First:

> *Thiết kế DB trước – code sau – mở rộng sau nữa*

---

## 📚 Tài liệu thiết kế đi kèm

### 🧩 File thiết kế DB (PowerDesigner)

* Sản phẩm:
  [https://github.com/macrozheng/mall-learning/blob/master/document/pdm/mall_pms.pdm](https://github.com/macrozheng/mall-learning/blob/master/document/pdm/mall_pms.pdm)
* Đơn hàng:
  [https://github.com/macrozheng/mall-learning/blob/master/document/pdm/mall_oms.pdm](https://github.com/macrozheng/mall-learning/blob/master/document/pdm/mall_oms.pdm)
* Marketing:
  [https://github.com/macrozheng/mall-learning/blob/master/document/pdm/mall_sms.pdm](https://github.com/macrozheng/mall-learning/blob/master/document/pdm/mall_sms.pdm)
* Nội dung:
  [https://github.com/macrozheng/mall-learning/blob/master/document/pdm/mall_cms.pdm](https://github.com/macrozheng/mall-learning/blob/master/document/pdm/mall_cms.pdm)
* Người dùng:
  [https://github.com/macrozheng/mall-learning/blob/master/document/pdm/mall_ums.pdm](https://github.com/macrozheng/mall-learning/blob/master/document/pdm/mall_ums.pdm)

---

### 🧠 Sơ đồ tư duy chức năng (MindMaster)

* Product: `pms.emmx`
* Order: `oms.emmx`
* Marketing: `sms.emmx`
* CMS: `cms.emmx`
* User: `ums.emmx`

👉 Dùng để **nhìn toàn cảnh nghiệp vụ**, cực kỳ đáng giá.

---

## 🛠️ Công cụ được sử dụng

* **PowerDesigner** – thiết kế database
  [http://powerdesigner.de/](http://powerdesigner.de/)

* **MindMaster** – vẽ sơ đồ tư duy
  [http://www.edrawsoft.cn/mindmaster](http://www.edrawsoft.cn/mindmaster)

---

## 📢 公众号

![Image](https://opengraph.githubassets.com/0e4358626612706b3d9867e82818afa40c744572ddb56dcd795566d96379e1ae/macrozheng/mall)

![Image](https://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/banner/qrcode_for_macrozheng_258.jpg)

👉 Theo dõi để:

* Hiểu **thiết kế DB hệ thống lớn**
* Đọc code mall **không bị lạc**
* Không đi đường vòng ❌

👉 Cứ nói, mình sẽ **đi cùng bạn từng bước – đúng chất Head First Java** 💙
