
## 📚 Học tập không đi đường vòng

👉 **[Theo dõi公众号](#公众号)** và **trả lời “学习路线”** để nhận **lộ trình học RIÊNG cho dự án mall**!

---

# 🔐 Dự án mall: Spring Security + JWT – Xác thực & phân quyền (Phần 2)

> **Phần 1**: bạn đã dựng xong “khung an ninh” (SecurityConfig, JWT Filter, UserDetails…).
>
> **Phần 2 này** sẽ làm 3 việc cực kỳ quan trọng:
>
> 1. Viết **Controller + Service** cho đăng nhập / đăng ký
> 2. **Chạy toàn bộ luồng login → token → phân quyền thật**
> 3. Nâng cấp **Swagger** để test API có bảo mật

💡 Head First nói thẳng:

> *Security mà không chạy được end-to-end thì mới chỉ là lý thuyết.*

---

## 🧩 1. Triển khai chức năng đăng nhập & đăng ký

---

## 👤 UmsAdminController – Controller cho user backend

> Controller này cung cấp 3 API:
>
> * Đăng ký user
> * Đăng nhập (trả JWT)
> * Lấy danh sách quyền của user

### 🎯 UmsAdminController

```java
@Controller
@Api(tags = "UmsAdminController", description = "Quản lý người dùng backend")
@RequestMapping("/admin")
public class UmsAdminController {
```

---

### 📝 API đăng ký

```java
@ApiOperation("Đăng ký người dùng")
@PostMapping("/register")
@ResponseBody
public CommonResult<UmsAdmin> register(@RequestBody UmsAdmin umsAdminParam) {
    UmsAdmin umsAdmin = adminService.register(umsAdminParam);
    if (umsAdmin == null) {
        return CommonResult.failed();
    }
    return CommonResult.success(umsAdmin);
}
```

💡 Head First hiểu nhanh:

> *Register = tạo user + mã hóa mật khẩu + lưu DB*

---

### 🔑 API đăng nhập – trả JWT

```java
@ApiOperation("Đăng nhập và trả JWT")
@PostMapping("/login")
@ResponseBody
public CommonResult login(@RequestBody UmsAdminLoginParam param) {
    String token = adminService.login(param.getUsername(), param.getPassword());
    if (token == null) {
        return CommonResult.validateFailed("Sai username hoặc password");
    }
    Map<String, String> tokenMap = new HashMap<>();
    tokenMap.put("token", token);
    tokenMap.put("tokenHead", tokenHead);
    return CommonResult.success(tokenMap);
}
```

👉 Sau login, frontend sẽ nhận được:

```json
{
  "token": "xxx.yyy.zzz",
  "tokenHead": "Bearer"
}
```

💡 Head First nhớ:

> *JWT được sinh ra ở đây – không phải trong filter.*

---

### 🔐 API lấy danh sách quyền

```java
@ApiOperation("Lấy toàn bộ quyền của user")
@GetMapping("/permission/{adminId}")
@ResponseBody
public CommonResult<List<UmsPermission>> getPermissionList(@PathVariable Long adminId) {
    return CommonResult.success(adminService.getPermissionList(adminId));
}
```

---

## 🧠 2. UmsAdminService – nơi xử lý logic thật sự

### 📄 Interface UmsAdminService

```java
public interface UmsAdminService {
    UmsAdmin getAdminByUsername(String username);
    UmsAdmin register(UmsAdmin umsAdminParam);
    String login(String username, String password);
    List<UmsPermission> getPermissionList(Long adminId);
}
```

💡 Head First note:

> *Controller chỉ điều phối – Service mới là não.*

---

## ⚙️ 3. UmsAdminServiceImpl – trái tim của login

---

### 📝 Logic đăng ký (register)

Luồng xử lý:

1. Copy dữ liệu
2. Set `createTime`, `status`
3. Kiểm tra username trùng
4. **Mã hóa password bằng BCrypt**
5. Lưu DB

```java
String encodePassword = passwordEncoder.encode(umsAdmin.getPassword());
umsAdmin.setPassword(encodePassword);
```

💡 Head First nhớ:

> *Không bao giờ lưu password dạng plain text.*

---

### 🔑 Logic đăng nhập (login)

Đây là đoạn **quan trọng nhất** 👇

```java
UserDetails userDetails =
    userDetailsService.loadUserByUsername(username);

if (!passwordEncoder.matches(password, userDetails.getPassword())) {
    throw new BadCredentialsException("Sai mật khẩu");
}

UsernamePasswordAuthenticationToken authentication =
    new UsernamePasswordAuthenticationToken(
        userDetails, null, userDetails.getAuthorities());

SecurityContextHolder.getContext().setAuthentication(authentication);

String token = jwtTokenUtil.generateToken(userDetails);
```

👉 Những gì xảy ra:

1. Load user + quyền
2. So sánh mật khẩu
3. Đưa user vào **SecurityContext**
4. Sinh **JWT**

💡 Head First kết luận:

> *Login thành công = có SecurityContext + có JWT.*

---

## 🧾 4. Nâng cấp Swagger để gửi JWT tự động

> Mục tiêu:
>
> 👉 Swagger **tự gắn Authorization header**
> 👉 Test API bảo mật **không cần Postman**

---

### 🛠️ Cấu hình Swagger Security

```java
.securitySchemes(securitySchemes())
.securityContexts(securityContexts());
```

---

### 🔑 Khai báo Authorization header

```java
new ApiKey("Authorization", "Authorization", "header");
```

👉 Swagger UI sẽ hiện nút **Authorize**

---

### 🔒 Chỉ áp dụng cho API cần login

```java
result.add(getContextByPath("/brand/.*"));
```

💡 Head First note:

> *Không phải API nào cũng cần token.*

---

## 🧩 5. Gắn quyền cho API bằng @PreAuthorize

> Bây giờ ta **khóa API theo permission thật**

### 🎯 Quy ước quyền

* `pms:brand:read`
* `pms:brand:create`
* `pms:brand:update`
* `pms:brand:delete`

---

### 🧪 Ví dụ

```java
@PreAuthorize("hasAuthority('pms:brand:read')")
public CommonResult<List<PmsBrand>> getBrandList() {
    return CommonResult.success(brandService.listAllBrand());
}
```

💡 Head First nhớ:

> *Permission là String – nhưng quyền lực là thật.*

---

## 🔄 6. Demo toàn bộ luồng xác thực & phân quyền

### 🌐 Truy cập Swagger UI

```
http://localhost:8080/swagger-ui.html
```

![Image](https://www.javainuse.com/static/boot-77-3-min.jpg)

![Image](https://keepgrowing.in/wp-content/uploads/2021/07/secured-swagger-ui.png)

---

### ❌ Chưa login → bị chặn

![Image](https://i.sstatic.net/eRwbE.png)

![Image](https://i.sstatic.net/kiptg.png)

---

### 🔑 Login bằng user test

![Image](https://futurestud.io/blog/content/images/2018/07/futureflix-api-swagger-docs-with-jwt-5.png)

![Image](https://miro.medium.com/v2/resize%3Afit%3A1400/0%2AD2KiuzlBHdiD2Op1.jpg)

---

### 🔐 Dán token vào Authorize

![Image](https://mattfrear.com/wp-content/uploads/2018/07/authbutton.jpg)

![Image](https://i.sstatic.net/iNBYM.png)

---

### ❌ User không có quyền → Forbidden

![Image](https://i.sstatic.net/tFPEt.png)

![Image](https://i2.wp.com/springframework.guru/wp-content/uploads/2017/02/swagger-ui_with_endpoint_documentation.png?ssl=1)

---

### ✅ Login bằng admin (có quyền)

![Image](https://ppolyzos.com/wp-content/uploads/2017/10/jwt-support-authorize-bearer.png)

![Image](https://www.freecodespot.com/app/uploads/2021/02/Authorize-button-1024x909.jpg)

---

## 📦 Source code dự án

🔗 GitHub:
[https://github.com/macrozheng/mall-learning/tree/master/mall-tiny-04](https://github.com/macrozheng/mall-learning/tree/master/mall-tiny-04)

---

## 📢 公众号

![Image](https://opengraph.githubassets.com/0e4358626612706b3d9867e82818afa40c744572ddb56dcd795566d96379e1ae/macrozheng/mall)

![Image](https://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/banner/qrcode_for_macrozheng_258.jpg)

👉 Theo dõi để:

* Hiểu Spring Security **từ config → filter → annotation**
* Làm backend **đúng chuẩn enterprise**
* Không đi đường vòng ❌
