
## 📚 Học tập không đi đường vòng

👉 **[Theo dõi公众号](#公众号)** và **trả lời “学习路线”** để nhận **lộ trình học RIÊNG cho dự án mall**!

---

# 🧾 Dự án mall: Tích hợp Swagger-UI để tạo tài liệu API online

> Bài viết này sẽ **dẫn bạn từng bước** cách dự án **mall tích hợp Swagger-UI**
> để tạo ra một **bộ tài liệu API online đầy đủ, dễ đọc, dễ test**.

💡 Head First nói thẳng:

> *API mà không có tài liệu → sớm muộn cũng thành “đống bí ẩn”* 😅
> Swagger-UI sinh ra để giải quyết chuyện đó.

---

## 🧩 1. Giới thiệu framework sử dụng

### 🧾 Swagger-UI là gì?

> **Swagger-UI** là một bộ công cụ gồm **HTML + JavaScript + CSS**,
> cho phép **tự động sinh tài liệu API online dựa trên annotation** trong code Java.

👉 Nói cách khác:

* Bạn viết annotation
* Swagger đọc annotation
* Swagger **vẽ ra tài liệu API cho bạn**

🎯 Không cần viết Word, không cần Postman thủ công.

---

### 🏷️ Các annotation Swagger thường dùng

Đây là phần **rất quan trọng**, nhớ kỹ nhé 👇

* `@Api`
  👉 Gắn lên **Controller**
  👉 Sinh tài liệu cho cả nhóm API

* `@ApiOperation`
  👉 Gắn lên **method**
  👉 Mô tả từng API cụ thể làm gì

* `@ApiParam`
  👉 Gắn lên **tham số request**
  👉 Giải thích ý nghĩa từng parameter

* `@ApiModelProperty`
  👉 Gắn lên **field của entity**
  👉 Dùng khi entity là **request / response**

💡 Head First nhớ:

> *Annotation = lời giải thích cho Swagger đọc*

---

## 🔌 2. Tích hợp Swagger-UI vào dự án

### 📦 Bước 1: Thêm dependency

> Mở `pom.xml` và thêm Swagger-UI vào 👇

```xml
<!-- Swagger-UI: công cụ sinh tài liệu API -->
<dependency>
  <groupId>io.springfox</groupId>
  <artifactId>springfox-swagger2</artifactId>
  <version>2.7.0</version>
</dependency>
<dependency>
  <groupId>io.springfox</groupId>
  <artifactId>springfox-swagger-ui</artifactId>
  <version>2.7.0</version>
</dependency>
```

👉 Sau bước này:
✔ Project đã “biết” Swagger là ai
❌ Nhưng chưa hoạt động – cần cấu hình tiếp

---

### ⚙️ Bước 2: Thêm cấu hình Swagger-UI

> Tạo file cấu hình Java cho Swagger

⚠️ Swagger cho bạn **3 cách chọn phạm vi sinh tài liệu**:

1. Sinh API theo **package**
2. Sinh API theo **annotation ở class**
3. Sinh API theo **annotation ở method**

👉 Bạn chọn **1 trong 3**, không phải dùng hết.

---

#### 📄 File cấu hình Swagger

```java
@Configuration
@EnableSwagger2
public class Swagger2Config {

    @Bean
    public Docket createRestApi(){
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                // Sinh tài liệu cho controller trong package này
                .apis(RequestHandlerSelectors
                      .basePackage("com.macro.mall.tiny.controller"))

                // Hoặc: chỉ sinh Controller có @Api
                // .apis(RequestHandlerSelectors.withClassAnnotation(Api.class))

                // Hoặc: chỉ sinh method có @ApiOperation
                // .apis(RequestHandlerSelectors.withMethodAnnotation(ApiOperation.class))

                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("SwaggerUI demo")
                .description("mall-tiny")
                .contact("macro")
                .version("1.0")
                .build();
    }
}
```

💡 Head First note:

> *Swagger không tự đoán API của bạn – bạn phải nói rõ cho nó biết*

---

## 🧩 3. Thêm Swagger annotation cho Controller

> Bây giờ ta “dạy” Swagger hiểu API của mình.

👉 Chỉ cần **thêm annotation**, logic code **KHÔNG đổi**.

---

### 🧾 PmsBrandController (đã thêm Swagger)

```java
@Api(tags = "PmsBrandController", description = "Quản lý thương hiệu sản phẩm")
@Controller
@RequestMapping("/brand")
public class PmsBrandController {
```

---

### 🧠 Ví dụ annotation cho API

```java
@ApiOperation("Phân trang danh sách thương hiệu")
public CommonResult<CommonPage<PmsBrand>> listBrand(
    @RequestParam(defaultValue = "1")
    @ApiParam("Số trang") Integer pageNum,

    @RequestParam(defaultValue = "3")
    @ApiParam("Số phần tử mỗi trang") Integer pageSize
)
```

👉 Swagger sẽ tự hiểu:

* API này làm gì
* Parameter có ý nghĩa gì
* Hiển thị rõ ràng trên UI

---

## 🧠 4. Nâng cấp MyBatis Generator để sinh Swagger annotation

### ❓ Vấn đề

> MBG mặc định chỉ sinh **comment JavaDoc**,
> **KHÔNG sinh @ApiModelProperty** 😢

👉 Nếu entity nhiều → thêm tay annotation là **ác mộng**.

---

### 💡 Giải pháp

👉 **Custom CommentGenerator** cho MBG:

* Dùng comment trong DB
* Sinh thẳng `@ApiModelProperty`
* Tự động import annotation

🎯 Viết **1 lần**, dùng **mãi mãi**

---

### 🛠️ Custom CommentGenerator

```java
/**
 * Custom Comment Generator
 * Sinh @ApiModelProperty từ comment DB
 */
public class CommentGenerator extends DefaultCommentGenerator {

    private static final String API_MODEL_PROPERTY =
        "io.swagger.annotations.ApiModelProperty";

    @Override
    public void addFieldComment(Field field,
        IntrospectedTable table,
        IntrospectedColumn column) {

        String remarks = column.getRemarks();
        if (remarks != null && !"".equals(remarks)) {
            field.addJavaDocLine(
              "@ApiModelProperty(value = \"" + remarks + "\")"
            );
        }
    }

    @Override
    public void addJavaFileComment(CompilationUnit unit) {
        if (!unit.isJavaInterface()) {
            unit.addImportedType(
              new FullyQualifiedJavaType(API_MODEL_PROPERTY)
            );
        }
    }
}
```

💡 Head First nhớ:

> *Database comment → Swagger doc → API rõ ràng*

---

## ▶️ 5. Chạy lại MBG để sinh code mới

> Chạy `Generator.main()`

👉 Kết quả:

* Entity tự có `@ApiModelProperty`
* Không cần sửa tay
* Swagger đọc được luôn

![Image](https://avatars.githubusercontent.com/u/42258113?v=4)

![Image](https://i.sstatic.net/VN4Y4.png)

---

## ▶️ 6. Chạy project & xem kết quả

### 🌐 Truy cập Swagger-UI

📍 Địa chỉ:

```
http://localhost:8080/swagger-ui.html
```

![Image](https://i1.wp.com/springframework.guru/wp-content/uploads/2017/02/swagger-ui_with_default_endpoint_documentation.png?ssl=1)

![Image](https://media2.dev.to/dynamic/image/width%3D800%2Cheight%3D%2Cfit%3Dscale-down%2Cgravity%3Dauto%2Cformat%3Dauto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbvgpy7imwr12vw8p3sux.png)

---

### ✅ Request parameter có mô tả

![Image](https://user-images.githubusercontent.com/6017680/109132621-f1ec9b80-774b-11eb-84e5-ac7a32ef89bf.png)

![Image](https://user-images.githubusercontent.com/36691961/37033985-cfc77f5c-216d-11e8-941b-ed0b79db355c.PNG)

---

### ✅ Response trả về có mô tả

![Image](https://i.sstatic.net/5kOcg.png)

![Image](https://i.sstatic.net/fX65j.png)

---

### 🧪 Test API trực tiếp trên trình duyệt

![Image](https://i.sstatic.net/T9qfY.png)

![Image](https://user-images.githubusercontent.com/3322909/29047817-713d5826-7b82-11e7-8c80-6551a57ded2f.png)

👉 Không cần Postman
👉 Không cần viết tài liệu tay

---

## 📦 Source code dự án

🔗 GitHub:
[https://github.com/macrozheng/mall-learning/tree/master/mall-tiny-02](https://github.com/macrozheng/mall-learning/tree/master/mall-tiny-02)

---

## 📢 公众号

![Image](https://opengraph.githubassets.com/0e4358626612706b3d9867e82818afa40c744572ddb56dcd795566d96379e1ae/macrozheng/mall)

![Image](https://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/banner/qrcode_for_macrozheng_258.jpg)

👉 Theo dõi để:

* Có lộ trình học rõ ràng
* Học Spring Boot + Mall bài bản
* Không đi đường vòng ❌ 💙
