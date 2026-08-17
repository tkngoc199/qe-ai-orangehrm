
---

## 1. Workflow Overview | Tổng quan luồng đăng nhập

The OrangeHRM login system follows a standard secure authentication architecture (MVC pattern in modern releases such as OrangeHRM 5.x). It authenticates credentials against the database, establishes an encrypted session context, and enforces Role-Based Access Control (RBAC) to route users to their designated dashboard.

> **Luồng tổng quát:** Hệ thống sử dụng kiến trúc xác thực bảo mật dựa trên vai trò (RBAC). Hệ thống nhận dữ liệu từ giao diện, kiểm tra tính hợp lệ, xác thực mật khẩu mã hóa trong CSDL, khởi tạo Session/Token và chuyển hướng người dùng đến trang làm việc tương ứng.

---

## 2. Step-by-Step Login Flow | Tiến trình từng bước

```markdown
![OrangeHRM Standard Login Interface](image_agent_tag_8005989583506580181)

---

## 1. Workflow Overview | Tổng quan luồng đăng nhập

The OrangeHRM login system follows a standard secure authentication architecture (MVC pattern in modern releases such as OrangeHRM 5.x). It authenticates credentials against the database, establishes an encrypted session context, and enforces Role-Based Access Control (RBAC) to route users to their designated dashboard.

> **Luồng tổng quát:** Hệ thống sử dụng kiến trúc xác thực bảo mật dựa trên vai trò (RBAC). Hệ thống nhận dữ liệu từ giao diện, kiểm tra tính hợp lệ, xác thực mật khẩu mã hóa trong CSDL, khởi tạo Session/Token và chuyển hướng người dùng đến trang làm việc tương ứng.

---

## 2. Step-by-Step Login Flow | Tiến trình từng bước


```

[User Access Login Page] ──► [Input Credentials] ──► [Client Validation]
│
[Redirect to Dashboard] ◄── [Session/Token Issued] ◄── [Server Validation]

```

1. **HTTP GET Request / Access Page (`/web/index.php/auth/login`)**:
   * **EN:** The user opens the login URL. The backend generates an HTTP session and embeds a unique **CSRF token** into the form.
   * **VI:** Người dùng truy cập URL đăng nhập. Server tạo một phiên làm việc HTTP và nhúng một mã **CSRF Token** duy nhất vào Form.

2. **Credential Input & Submission / Nhập dữ liệu & Gửi Form**:
   * **EN:** The user inputs their `Username` and `Password` and clicks "Login".
   * **VI:** Người dùng nhập `Username` (Tên đăng nhập) và `Password` (Mật khẩu) rồi nhấn "Login".

3. **Client-Side Validation / Kiểm tra tại Frontend**:
   * **EN:** JavaScript checks for empty inputs or illegal whitespace characters before initiating the POST request.
   * **VI:** JavaScript tại trình duyệt kiểm tra xem các trường có bị bỏ trống hoặc chứa khoảng trắng không hợp lệ trước khi gửi request.

4. **Server-Side Authentication / Xác thực tại Backend**:
   * **EN:** The server verifies the CSRF token, queries the DB for the username, verifies the password hash, and checks if the account is active.
   * **VI:** Máy chủ đối soát CSRF token, tìm kiếm tài khoản trong CSDL, giải mã/đối chiếu chuỗi băm mật khẩu và kiểm tra trạng thái hoạt động của tài khoản.

5. **Session Initialization & Routing / Cấp Session & Chuyển hướng**:
   * **EN:** Upon successful authentication, a secure session cookie or Bearer JWT token is generated. The user is redirected based on their assigned system role.
   * **VI:** Khi xác thực thành công, hệ thống tạo Session Cookie (hoặc JWT Bearer Token) và chuyển hướng người dùng đến bảng điều khiển theo phân quyền.

---

## 3. Validation Rules | Quy tắc kiểm tra dữ liệu

| Field / Component | Validation Rule (Quy tắc) | Level | Description (Mô tả) |
|---|---|---|---|
| **Username** | Required / Non-empty | Client & Server | Cannot be null, empty, or contain trailing spaces. |
| **Password** | Required / Non-empty | Client & Server | Must be provided. Case-sensitive matching required. |
| **CSRF Token** | Anti-tamper token match | Server | Must match the active session token generated on form load. |
| **Account Status** | `Status = Enabled` | Server | User status in DB must be active (`1` / Enabled). |
| **Input Sanitization** | SQLi / XSS Prevention | Server | Strip HTML tags and execute parameterized queries (PDO / Doctrine). |

---

## 4. Error Messages | Thông báo lỗi hệ thống

To prevent credential enumeration attacks, generic error messages are presented for authentication failures.

| Trigger Condition (Điều kiện) | Displayed Error Message | Thông báo Tiếng Việt |
|---|---|---|
| Empty Username or Password | `Required` | *Trường này là bắt buộc* |
| Wrong Username or Password | `Invalid credentials` | *Thông tin đăng nhập không hợp lệ* |
| Account Status = Disabled | `Invalid credentials` *(or "Account Disabled")* | *Thông tin không hợp lệ (hoặc Tài khoản đã bị vô hiệu hóa)* |
| Invalid / Expired CSRF Token | `CSRF token validation failed` / Page Reload | *Lỗi xác thực Token / Trình duyệt tự tải lại trang* |
| Max Login Attempts Exceeded | `Account temporarily locked. Try again later.` | *Tài khoản bị tạm khóa do nhập sai nhiều lần.* |

---

## 5. Session Behavior | Quản lý phiên làm việc

* **Session Cookie & Token Creation**:
  * **EN:** On login, OrangeHRM creates a session cookie (`PHPSESSID` or `orangehrm` cookie) configured with flags: `HttpOnly` (prevents XSS cookie theft), `SameSite=Lax/Strict`, and `Secure` (when running over HTTPS).
  * **VI:** Sau khi đăng nhập thành công, hệ thống tạo Cookie phiên làm việc được đính kèm các cờ bảo mật: `HttpOnly`, `SameSite`, và `Secure` (khi chạy HTTPS).
* **Idle Timeout (Tự động hết hạn)**:
  * **EN:** Sessions automatically expire after a predefined period of inactivity (default usually 15–30 minutes, configurable in `php.ini` or system settings).
  * **VI:** Phiên làm việc tự động hết hạn nếu người dùng không thao tác trong một khoảng thời gian nhất định (thường từ 15–30 phút).
* **Logout Behavior (Đăng xuất)**:
  * **EN:** Clicking Logout triggers server-side session destruction (`session_destroy()`), revokes auth tokens, clears browser cookies, and sets HTTP cache headers (`Cache-Control: no-store`) to block the back button from rendering cached pages.
  * **VI:** Đăng xuất sẽ hủy hoàn toàn Session trên Server, xóa Cookie ở Client và xóa Cache trình duyệt để ngăn người dùng nhấn nút "Back" quay lại trang trước.

---

## 6. User Roles & Redirection | Phân quyền & Chuyển hướng

OrangeHRM dynamically adjusts user interface capabilities and home dashboards depending on the assigned role:

| User Role (Vai trò) | Landing Page (Trang chuyển hướng) | Access Scope (Phạm vi truy cập) |
|---|---|---|
| **System Admin** | `/web/index.php/admin/viewSystemUsers` | Full control over user accounts, roles, system configuration, and logs. |
| **ESS (Employee Self Service)** | `/web/index.php/dashboard/index` or `/pim/viewMyDetails` | Limited to personal profile, leave applications, timesheets, and company announcements. |
| **Supervisor** | `/web/index.php/dashboard/index` | Access to manage subordinate details, approve leave requests, and review performance. |
| **HR Admin / Regional Admin** | `/web/index.php/pim/viewEmployeeList` | Administrative access restricted to designated business units, locations, or sub-departments. |

---

## 7. Security Considerations | Các khía cạnh bảo mật

* **Password Hashing (Mã hóa mật khẩu)**: Passwords are stored in the database using strong, salted password hashing algorithms (e.g., **BCrypt** or **Argon2**), ensuring plaintext passwords are never readable.
* **Anti-Brute Force Protection (Chống tấn công dò mật khẩu)**: Implements rate-limiting or temporary lockout policies after consecutive failed login attempts.
* **User Enumeration Prevention (Chống dò tên người dùng)**: Uses standard error messages (`Invalid credentials`) whether the username doesn't exist or the password is wrong.
* **CSRF Protection (Bảo mật chống giả mạo Yêu cầu)**: Every login form submission requires a cryptographically secure token tied to the current browser session.
* **Transport Layer Security (Mã hóa đường truyền)**: Enforcement of HTTPS protocol prevents eavesdropping or Man-in-the-Middle (MitM) credential sniffing.

```
