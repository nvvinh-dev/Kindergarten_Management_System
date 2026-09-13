# DECISIONS.md — Project Decisions

> File này ghi lại các quyết định quan trọng của dự án.
> Mỗi quyết định cần có lý do rõ ràng.
> Không tự ý thay đổi quyết định đã thống nhất.
> Nếu cần thay đổi, phải ghi lại quyết định mới và lý do thay đổi.

---

# 1. PROJECT

## D1 — Project Type

**Decision:** Xây dựng hệ thống quản lý nhà trẻ trên nền tảng web.

**Reason:**
- Phù hợp phạm vi Đồ án 1.
- Có nhiều nhóm người dùng.
- Có thể triển khai trên trình duyệt.
- Phù hợp với kiến trúc Client — API — Database.

**Status:** Accepted

---

# 2. TARGET USERS

## D2 — Age Group

**Decision:** Hệ thống phục vụ nhà trẻ dành cho trẻ từ 3–6 tuổi.

**Reason:**
Đây là phạm vi được xác định trong yêu cầu của dự án.

**Status:** Accepted

---

# 3. USER ROLES

## D3 — Main Roles

Hệ thống có 5 nhóm vai trò chính:

1. Admin / Ban giám hiệu
2. Giáo viên
3. Kế toán / Văn phòng
4. Y tế
5. Phụ huynh

**Decision:** Không tự ý tạo thêm role nếu chưa có yêu cầu.

**Status:** Accepted

---

# 4. BACKEND TECHNOLOGY

## D4 — Backend Framework

**Decision:** Sử dụng ASP.NET Core Web API.

**Status:** Accepted

---

## D5 — ORM

**Decision:** Sử dụng Entity Framework Core.

**Status:** Accepted

---

## D6 — Authentication / Identity

**Decision:** Không dùng ASP.NET Identity đầy đủ (không dùng UserManager/
RoleManager/cookie authentication). Chỉ dùng riêng
`Microsoft.AspNetCore.Identity`'s `PasswordHasher<T>` (package
`Microsoft.Extensions.Identity.Core`) để hash mật khẩu.

**Reason:** Backend tự phát hành và verify JWT (xem D20 — quyết định đầy
đủ về authentication strategy). Dùng nguyên bộ ASP.NET Identity + cookie
sẽ kéo theo rắc rối CORS/SameSite không cần thiết cho kiến trúc frontend
và backend tách domain.

**Status:** Accepted

---

## D7 — Validation

**Decision:** Sử dụng FluentValidation cho validation ở backend.

**Status:** Accepted

---

## D8 — Logging

**Decision:** Sử dụng Serilog cho logging.

**Status:** Accepted

---

## D9 — API Documentation

**Decision:** Sử dụng Swagger / OpenAPI (Swashbuckle.AspNetCore, có
Swagger UI thật tại /swagger, chỉ bật ở môi trường Development).

**Ghi chú kỹ thuật:** Dự án dùng net10.0 — package Microsoft.OpenApi
đi kèm là bản 2.x, đã đổi cấu trúc namespace so với các hướng dẫn cũ
trên mạng (bỏ hẳn namespace con `.Models`, ví dụ
`Microsoft.OpenApi.Models.OpenApiInfo` → `Microsoft.OpenApi.OpenApiInfo`).
Nếu thành viên nào tra cứu tài liệu Swashbuckle cũ và gặp lỗi biên
dịch tương tự, đây là nguyên nhân — sửa namespace theo đúng bản 2.x.

Do một số API của Microsoft.OpenApi 2.x với Swashbuckle chưa hoàn
toàn ổn định (bản .NET còn mới), Swagger UI hiện CHƯA tự động đính
JWT token vào mọi request khi bấm "Try it out" (thiếu cấu hình
AddSecurityRequirement — bị lược bỏ do lỗi biên dịch chưa xác định
được API đúng ở thời điểm này). Muốn test endpoint cần đăng nhập qua
Swagger, phải tự thêm header Authorization thủ công, hoặc dùng
PowerShell/Postman như đã làm xuyên suốt quá trình build Auth.

**Status:** Accepted

---

# 5. FRONTEND TECHNOLOGY

## D10 — Frontend Framework

**Decision:** Sử dụng React / Next.js với TypeScript.

**Status:** Accepted

---

## D11 — Styling

**Decision:** Sử dụng Tailwind CSS.

**Status:** Accepted

---

## D12 — Form Management

**Decision:** Sử dụng React Hook Form.

**Status:** Accepted

---

## D13 — Server State

**Decision:** Sử dụng TanStack Query để quản lý server state và API data.

**Status:** Accepted

---

## D14 — HTTP Client

**Decision:** Sử dụng Axios để giao tiếp với backend API.

**Status:** Accepted

---

# 6. DATABASE

## D15 — Database

**Decision:** Sử dụng PostgreSQL.

**Status:** Accepted

---

## D16 — Database Hosting

**Decision:** Sử dụng Supabase PostgreSQL.

**Important:**
Frontend không được truy cập trực tiếp database.

Kiến trúc:

```text
Frontend
   ↓
ASP.NET Core Web API
   ↓
Entity Framework Core
   ↓
PostgreSQL / Supabase
Status: Accepted

7. API ARCHITECTURE
D17 — Communication

Decision: Frontend giao tiếp với backend thông qua API.

Frontend không được:

Truy cập trực tiếp database.
Thực hiện query trực tiếp tới PostgreSQL.
Bỏ qua backend để truy cập dữ liệu.

Status: Accepted

D18 — API Style

Decision: Sử dụng RESTful API.

Status: Accepted

8. DATABASE DESIGN
D19 — Database First or Code First

Decision: Chưa triển khai migration cho đến khi database design được phân tích và review.

Workflow:

Requirements
    ↓
Business Rules
    ↓
Entities
    ↓
Relationships
    ↓
ERD
    ↓
Database Review
    ↓
EF Core Implementation
    ↓
Migration

Status: Accepted

9. AUTHENTICATION
## D20 — Authentication Strategy

**Decision:** Backend (ASP.NET Core) là nơi duy nhất phát hành và xác
thực JWT (Bearer token). Frontend gọi thẳng backend qua Axios (không
dùng BFF proxy).

**Đề xuất bởi:** Claude Sonnet — **Audit bởi:** Claude Opus — **Quyết
định cuối:** [Lead]

### Chi tiết quyết định
- KHÔNG dùng NextAuth (khác với tech stack liệt kê ban đầu trong đề
  tài — lý do: NextAuth giải quyết bài toán multi-OAuth-provider mà dự
  án không có; team tự viết AuthProvider bằng React Context, ước lượng
  ~80 dòng, đơn giản hơn và tránh rủi ro breaking-change giữa NextAuth
  v4/Auth.js v5).
- `login_identifier` = email. Chuẩn hóa lowercase ở tầng Application
  (Service, trước khi lưu và trước khi so sánh lúc login) — không dùng
  `citext`, tránh phụ thuộc extension PostgreSQL không cần thiết.
- `credential_reference` = password hash, dùng
  `PasswordHasher<T>` (xem D6), `CompatibilityMode = IdentityV3` (ghi
  tường minh để đồng nhất hash giữa máy các thành viên).
- `IPasswordHasher` và `ITokenService` là interface định nghĩa ở
  Application layer, implement ở Infrastructure layer (đúng dependency
  rule đã chốt trong `architecture.md` §2.2).

### Cấu hình JWT
- Thuật toán: HS256, khóa ký ≥ 32 byte, lưu trong biến môi trường /
  user-secrets (không hard-code, không commit).
- Claims: `sub` (user_id), `role`, `iss`, `aud`, `iat`, `jti`, `exp`.
  KHÔNG đưa `full_name` vào token (JWT chỉ được ký, không mã hóa —
  payload đọc được bởi bất kỳ ai cầm token).
- Role claim: dùng hằng số C# (`Roles.Teacher`, `Roles.Parent`, ...)
  ánh xạ từ `role_id` — KHÔNG dùng thẳng `roles.name` tiếng Việt làm
  claim.
- `TokenValidationParameters` khai báo tường minh: `RoleClaimType =
  "role"`, `NameClaimType = "sub"`, `ValidateIssuer`/`Audience = true`,
  `ValidAlgorithms` giới hạn `{ HS256 }`, `ClockSkew` rút xuống 1 phút.
- **`options.MapInboundClaims = false` là bắt buộc** trong cấu hình
  `AddJwtBearer` (đặt trước `TokenValidationParameters`). Nếu thiếu,
  .NET tự đổi tên claim `sub`/`role` thành URI dài theo cơ chế cũ
  (`ClaimTypes.NameIdentifier`...), khiến mọi `User.FindFirst("sub")`/
  middleware kiểm tra claim đều thất bại âm thầm với lỗi 401 khó
  chẩn đoán. Đây là lỗi thực tế đã gặp khi build Auth — bất kỳ ai
  thêm claim mới vào JWT cần nhớ quy tắc này.
- Thời hạn token: 10 giờ — khớp đúng giờ hoạt động thực tế của nhà trẻ (7h–17h). Giáo viên/nhân viên đăng nhập lại mỗi đầu ca, không cần thời hạn dư ra.
- KHÔNG làm refresh token (trade-off có chủ đích, phù hợp quy mô đồ
  án). Hệ quả: "đăng xuất" chỉ xóa token ở phía client — token cũ vẫn
  hợp lệ tới khi hết hạn nếu đã bị sao chép ra ngoài.

### Kiểm tra is_active / role_id mỗi request
- KHÔNG chỉ tin dữ liệu đóng băng trong token suốt 12h. Middleware/
  filter query nhẹ theo PK (`SELECT is_active, role_id FROM users
  WHERE id = @sub`) trên mỗi request — đảm bảo Admin khóa tài khoản
  hoặc đổi role có hiệu lực ngay.

### Lưu trữ token ở Frontend
- React Context (in-memory) là nguồn chính; đồng thời lưu 1 bản vào
  `localStorage` chỉ để khôi phục phiên khi reload trang.
- Rủi ro đã biết: token có thể bị đọc nếu xảy ra XSS (đánh đổi có chủ
  đích cho quy mô đồ án, không phải giải pháp an toàn tuyệt đối).
- Axios response interceptor: gặp 401 → xóa token khỏi Context và
  `localStorage` → redirect về trang login.

### CORS
- Bắt buộc cấu hình `AddCors` với origin cụ thể của frontend (không
  `AllowAnyOrigin`), cho phép header `Authorization`, xử lý preflight
  `OPTIONS`. Không cần `AllowCredentials` (không dùng cookie).

### Bổ sung bảo mật
- Rate limiting cho `/api/auth/login` bằng `AddRateLimiter` (.NET 7+).

### Quy trình mật khẩu (đã chốt)
- Mật khẩu ban đầu: Admin tự nhập trực tiếp khi tạo tài khoản mới
  (endpoint tạo User nhận thêm trường password thô, hash ngay trong
  Application Service bằng PasswordHasher<T> trước khi lưu — không
  bao giờ lưu password thô).
- Không bắt buộc đổi mật khẩu lần đăng nhập đầu — không thêm cột
  must_change_password, giữ nguyên schema users đã chốt.
- Không làm chức năng quên mật khẩu tự động (không gửi email/SMS —
  dự án chưa có provider nào cho việc này, tránh thêm dependency mới
  ngoài phạm vi D35). Admin tự đặt lại mật khẩu thủ công cho user khi
  có yêu cầu, qua chức năng quản lý user sẵn có của Admin.
- Giờ hoạt động thực tế nhà trẻ: 7h–17h (~10 tiếng) — đã dùng để xác
  nhận thời hạn token ở trên.

**Rule:** Không tự ý triển khai vượt quá phạm vi quyết định này trước
khi Open Question còn lại được chốt.

**Status:** Accepted

10. PAYMENT
D21 — Online Payment Provider

Decision: Chưa lựa chọn nhà cung cấp thanh toán online.

Các lựa chọn có thể được đánh giá sau:

VNPay
MoMo
PayOS
Stripe
Provider khác nếu phù hợp

Rule:

Không tự ý tích hợp payment provider trước khi nhóm quyết định.

Status: Pending

11. MEDIA STORAGE
D22 — Photo / Video Storage

Decision: Chưa lựa chọn storage provider cuối cùng.

Các lựa chọn cần đánh giá:

Supabase Storage
Cloudinary
Amazon S3
Provider khác nếu phù hợp

Rule:

Không lưu file media tùy tiện trong source code repository.

Status: Pending

12. ATTENDANCE
D23 — Attendance Status

Decision: Chưa chốt danh sách trạng thái điểm danh.

Cần xác định:

Có mặt
Vắng
Đi trễ
Có phép
Các trạng thái khác nếu thực sự cần

Rule:

Không tự ý tạo thêm attendance status nếu chưa được thống nhất.

Status: Pending

13. PICKUP PERSON
D24 — Registered Pickup People

Decision: Hệ thống phải hỗ trợ người được phụ huynh đăng ký để đón trẻ.

Status: Accepted

Details pending:

Thông tin bắt buộc
Quan hệ với trẻ
Số điện thoại
Giấy tờ xác minh nếu cần
Ảnh nếu cần
Thời hạn đăng ký nếu cần

Các chi tiết trên phải được xác định trước khi thiết kế database.

14. NOTIFICATIONS
D25 — Notification Mechanism

Decision: Hệ thống cần hỗ trợ thông báo cho người dùng theo yêu cầu nghiệp vụ.

Details pending:

In-app notification
Email
Push notification
Các hình thức khác nếu cần

Không tích hợp notification provider bên ngoài nếu chưa có yêu cầu.

Status: Pending

15. ARCHITECTURE
D26 — System Architecture

Decision: Hệ thống sử dụng kiến trúc phân tách:

┌─────────────────────┐
│      Frontend       │
│ React / Next.js     │
│ TypeScript          │
└──────────┬──────────┘
           │ HTTP / REST API
           ↓
┌─────────────────────┐
│       Backend       │
│ ASP.NET Core Web API│
└──────────┬──────────┘
           │ EF Core
           ↓
┌─────────────────────┐
│      Database       │
│ PostgreSQL / Supabase│
└─────────────────────┘

Status: Accepted

16. DEVELOPMENT WORKFLOW
D27 — Git Branching

Decision: Sử dụng:

main
  ↓
develop
  ↓
feature/*

Feature branch được sử dụng cho từng nhóm chức năng.

Các feature branch hiện có:

feature/auth
feature/student
feature/teacher
feature/class
feature/attendance
feature/health
feature/tuition
feature/notification

Status: Accepted

D28 — Main Branch

Decision: Không phát triển trực tiếp trên main.

main phải giữ trạng thái ổn định.

Status: Accepted

D29 — Develop Branch

Decision: develop là branch tích hợp chính cho các feature đã hoàn thành.

Status: Accepted

17. TESTING
D30 — API Testing

Decision: Sử dụng Postman để kiểm thử API trong quá trình phát triển.

Status: Accepted

D31 — User Scenario Testing

Decision: Kiểm thử dựa trên các kịch bản sử dụng thực tế của từng role.

Các nhóm scenario:

Admin
Giáo viên
Kế toán / Văn phòng
Y tế
Phụ huynh

Status: Accepted

18. DOCUMENTATION
D32 — UML / ERD

Decision: Sử dụng PlantUML cho các sơ đồ cần thiết.

Bao gồm:

Use Case Diagram
ERD
Architecture Diagram
Các sơ đồ UML khác nếu cần

Status: Accepted

19. PROJECT MANAGEMENT
D33 — Task Management

Decision: Sử dụng Trello để quản lý:

Backlog
Tasks
Sprint
Progress

Status: Accepted

20. AI DEVELOPMENT
D34 — AI Coding Assistant

Decision: Claude Code được sử dụng để hỗ trợ:

Phân tích
Lập kế hoạch
Viết code
Refactor
Debug
Testing
Review
Documentation

Rule:

Claude không được tự ý:

Thay đổi kiến trúc quan trọng.
Thêm công nghệ mới.
Thêm package không cần thiết.
Thêm chức năng ngoài scope.
Thay đổi business rule chưa được thống nhất.
21. CODE QUALITY
D35 — No Unnecessary Complexity

Decision: Ưu tiên code đơn giản, dễ đọc và phù hợp với quy mô Đồ án 1.

Không over-engineering.

Không tạo abstraction chỉ để làm code "trông chuyên nghiệp".

Status: Accepted

22. CHANGE MANAGEMENT
D36 — Changing Existing Decisions

Khi cần thay đổi một quyết định:

Xác định quyết định hiện tại.
Nêu lý do cần thay đổi.
Đánh giá ảnh hưởng.
Đưa ra phương án mới.
Thống nhất phương án.
Cập nhật DECISIONS.md.
Cập nhật các tài liệu liên quan.
Sau đó mới triển khai.

Status: Accepted

23. CURRENT PENDING DECISIONS

Các quyết định cần được hoàn thiện:

 Initial password / password reset flow (xem D20, Open Question còn lại)
 Xác nhận giờ hoạt động nhà trẻ (ảnh hưởng thời hạn token, xem D20)
 Parent / guardian relationship
 Pickup person data
 Attendance statuses
 Online payment provider
 Media storage provider
 Notification mechanism
 Final database entities
 Final database relationships
24. DECISION LOG
ID	Decision	Status
D1	Web-based system	Accepted
D2	Children aged 3–6	Accepted
D3	5 main roles	Accepted
D4	ASP.NET Core Web API	Accepted
D5	Entity Framework Core	Accepted
D6	PasswordHasher<T> only (no full Identity)	Accepted
D7	FluentValidation	Accepted
D8	Serilog	Accepted
D9	Swagger / OpenAPI	Accepted
D10	React / Next.js + TypeScript	Accepted
D11	Tailwind CSS	Accepted
D12	React Hook Form	Accepted
D13	TanStack Query	Accepted
D14	Axios	Accepted
D15	PostgreSQL	Accepted
D16	Supabase PostgreSQL	Accepted
D17	Frontend → API → Database	Accepted
D18	RESTful API	Accepted
D19	Database before migration	Accepted
D20	JWT (backend-issued), no NextAuth, no BFF	Accepted
D21	Payment provider	Pending
D22	Media storage	Pending
D23	Attendance statuses	Pending
D24	Registered pickup people	Accepted / Details pending
D25	Notification mechanism	Pending
D26	System architecture	Accepted
D27	Git branching	Accepted
D28	No direct development on main	Accepted
D29	develop as integration branch	Accepted
D30	Postman API testing	Accepted
D31	User scenario testing	Accepted
D32	PlantUML	Accepted
D33	Trello	Accepted
D34	Claude Code	Accepted
D35	Avoid over-engineering	Accepted
D36	Decision change process	Accepted
## D37 — Foreign Key Delete Behavior for "Recorded By" Relationships

**Decision:** Mọi khóa ngoại kiểu "người ghi nhận" (RecordedByTeacher,
RecordedByUser, UploadedByTeacher) dùng ON DELETE RESTRICT thay vì
CASCADE (mặc định của EF Core convention). Khóa ngoại User → Role
cũng đổi sang RESTRICT.

**Reason:** Cascade mặc định sẽ xóa sạch lịch sử điểm danh/sức khỏe/
sự cố/hoạt động nếu tài khoản giáo viên/y tế bị xóa cứng — đây là bằng
chứng nghiệp vụ không được phép mất. Admin nên vô hiệu hóa tài khoản
(users.is_active = false, xem D20) thay vì xóa cứng; Restrict ép buộc
đúng hành vi này ở tầng database.

**Status:** Accepted

---

## D38 — API Response Format

**Decision:** Mọi response đi ra khỏi backend — thành công, lỗi validate,
lỗi nghiệp vụ, 401, 403, 404, 405, 415 và 500 — đều dùng đúng một khuôn
`ApiResponse<T>`:

```json
{ "success": bool, "data": T|null, "message": string|null, "errors": string[]|null }
```

**Reason:** Frontend chỉ cần viết **một** hàm đọc lỗi dùng chung. Trước
quyết định này, backend trả về 4 khuôn khác nhau (ApiResponse từ
Controller, `{ message }` từ ActiveUserMiddleware, body rỗng từ JWT
middleware, ProblemDetails từ `[ApiController]`), buộc Frontend phải
đoán kiểu response trước khi đọc được thông báo lỗi.

**Đã áp dụng tại (đã test thật bằng curl, không phải suy đoán):**

| Nơi | Trường hợp | Cơ chế |
| :---- | :---- | :---- |
| `AuthController` | 200, 400 validate, 401 sai mật khẩu | gọi `ApiResponse<T>` tường minh |
| `GlobalExceptionHandler` | 500 lỗi ngoài dự đoán | `IExceptionHandler` |
| `ActiveUserMiddleware` | 401 do `is_active` / đổi `role_id` / token hỏng | `WriteUnauthorized` |
| `JwtBearerEvents.OnChallenge` | 401 thiếu token / token sai / **token hết hạn** | `Program.cs` |
| `JwtBearerEvents.OnForbidden` | 403 sai vai trò | `Program.cs` |
| `ApiBehaviorOptions.InvalidModelStateResponseFactory` | 400 do JSON sai kiểu / thiếu field | `Program.cs` |
| `UseStatusCodePages` + `SuppressMapClientErrors` | 404 sai route, 405 sai method, 415 sai Content-Type | `Program.cs` |

**Ràng buộc kèm theo:** `InvalidModelStateResponseFactory` **không**
được trả thẳng `ModelState.ErrorMessage` ra client — .NET nhét cả tên
class DTO và vị trí byte vào chuỗi đó, lộ cấu trúc nội bộ (vi phạm
NFR-SEC-03). Chỉ trả tên trường bị sai.

**Ghi chú cho người viết Controller mới:** không cần làm gì thêm — cứ
`return Ok(ApiResponse<T>.Ok(data))` / `BadRequest(ApiResponse<T>.Fail(...))`
theo mẫu `AuthController`. Toàn bộ phần 401/403/404/405/415/500 đã được
xử lý tập trung ở `Program.cs`, **không tự viết lại trong Controller**.

**Status:** Accepted

---

25. LAST UPDATED
Date: 2026-09-12
Status: Authentication Strategy Finalized — Ready for EF Core Implementations