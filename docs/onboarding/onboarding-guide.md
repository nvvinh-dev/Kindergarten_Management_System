# Hướng dẫn Onboarding — Backend & Frontend Dev

**Phiên bản 2 — đã audit lại theo code thật trong repo (12/09/2026).**

Đọc hết tài liệu này trước khi code dòng đầu tiên. Mọi thứ mô tả từ mục 2 trở đi đều đã chạy
thật trên máy Lead (build sạch 0 warning, login trả token thật), không phải kế hoạch.

> **Quy ước đọc tài liệu này**
> - ✅ = đã có sẵn trong repo, dùng ngay.
> - ⚠️ = có sẵn nhưng có bẫy, đọc kỹ ghi chú.
> - ⛔ = **chưa có**, đừng đi tìm — mục 8 ghi rõ ai làm / khi nào.

---

## 0. Trước khi bạn bắt đầu — checklist của Lead

Nếu bất kỳ dòng nào dưới đây chưa được tick, **báo Lead trước khi setup**, vì làm tiếp sẽ
lãng phí thời gian:

- [ ] Toàn bộ thư mục `backend/` đã được commit và push lên `origin/develop`.
- [ ] `DECISIONS.md` bản mới nhất (có **D37**, **D38**, có ghi chú Swagger ở **D9**) đã được push.
- [ ] `CLAUDE.md`, `PROGRESS.md`, `TASKS.md` trên remote đã được dọn hoặc xác nhận còn hiệu lực.
- [x] ~~CORS đã bật trong `Program.cs`~~ — **đã xong**, xem 4.1.
- [x] ~~Rate limit `/api/auth/login`~~ — **đã xong**, xem 5.6.
- [x] ~~Viết `docs/api/api-conventions.md`~~ — **đã xong**.
- [x] ~~Dọn rác `dotnet new`~~ — **đã xong**, xem 2.1.
- [ ] Dự án `frontend/` đã được khởi tạo và push (xem 4.0).
- [ ] Connection string + JWT Key đã gửi riêng cho từng thành viên qua chat cá nhân
      (**không nằm trong file này, không commit vào repo**).

---

## 1. Yêu cầu máy — cài trước khi clone

| Công cụ | Phiên bản | Cần cho | Ghi chú |
| :---- | :---- | :---- | :---- |
| **.NET SDK 10** | 10.0.4xx trở lên | Backend | Dự án target `net10.0`. SDK 8/9 sẽ **build fail** ngay. Kiểm tra bằng `dotnet --list-sdks` |
| dotnet-ef | 10.x | Backend | `dotnet tool install --global dotnet-ef` (đã có rồi thì `dotnet tool update --global dotnet-ef`) |
| IDE | VS 2022 17.14+ / VS 2026 / Rider 2025.2+ / VS Code | Backend | Solution dùng định dạng mới `NhaTreManagement.slnx`. IDE cũ **không mở được** — dùng CLI `dotnet build` hoặc mở từng `.csproj` |
| Node.js | 20 LTS trở lên | Frontend | |
| Git | bất kỳ | Cả hai | |
| Postman | bản mới | Cả hai | D30 — công cụ test API chính thức của nhóm |

Kiểm tra nhanh:

```bash
dotnet --list-sdks
```

Phải thấy một dòng bắt đầu bằng `10.`. Không có thì cài .NET 10 SDK rồi mới đi tiếp.

---

## 2. Những gì đã có sẵn — bạn KHÔNG cần tự làm lại

| Thành phần | Vị trí | Trạng thái |
| :---- | :---- | :---- |
| 4 project (Domain / Application / Infrastructure / API) | `backend/` | ✅ Đã dựng, build sạch 0 warning |
| 20 Entity class + `AppDbContext` + Fluent API config | `NhaTre.Domain/Entities/`, `NhaTre.Infrastructure/Persistence/` | ✅ Map đúng theo `database-design.md` |
| Database thật trên Supabase | — | ⚠️ Đã migrate + seed roles + 1 tài khoản Admin. **Cả nhóm dùng chung 1 database** — đọc kỹ mục 6 |
| Đăng nhập (JWT) | `NhaTre.API/Controllers/AuthController.cs` | ✅ Chạy thật — dùng làm **mẫu chuẩn** cho mọi Controller khác |
| `ApiResponse<T>` | `NhaTre.Application/common/ApiResponse.cs` (namespace `NhaTre.Application.Common`) | ✅ Mọi endpoint **do nhóm viết** phải trả đúng khuôn này — xem 5.4 |
| Validate (FluentValidation) | `NhaTre.Application/Validators/` | ✅ Gọi tường minh trong Controller, xem mẫu `LoginRequestValidator` |
| Global Exception Handler | `NhaTre.API/Middleware/GlobalExceptionHandler.cs` | ✅ Tự bắt lỗi ngoài dự đoán → 500 + `ApiResponse`. Không cần try-catch phòng thủ |
| Kiểm tra tài khoản còn hiệu lực mỗi request | `NhaTre.API/Middleware/ActiveUserMiddleware.cs` | ✅ Chạy tự động, trả 401 đúng khuôn `ApiResponse` |
| Khuôn response thống nhất cho **mọi** mã lỗi (401/403/404/405/415/500) | `Program.cs` | ✅ Đã gom về `ApiResponse<T>` — xem D38 và mục 4.4 |
| Serilog | cấu hình trong `Program.cs` | ✅ Ghi log ra `NhaTre.API/Logs/`, giữ 14 ngày, đã gitignore |
| Swagger UI | http://localhost:5015/swagger | ⚠️ Chạy được nhưng **chưa tự đính token** — xem D9 và mục 3.5 |
| CORS | `Program.cs` + `appsettings.json` | ✅ Đã bật cho `http://localhost:3000`, đã test thật — xem 4.1 |
| Rate limit `/api/auth/login` | `Program.cs` + `AuthController` | ✅ Đã bật: 5 lần thử / 1 phút / 1 IP, vượt ngưỡng trả 429 — xem 5.6 |
| Quy ước API (route / status code / DTO / phân quyền) | `docs/api/api-conventions.md` | ✅ Đã có — **đọc kỹ trước khi viết Controller mới** |

**Đừng tự tạo `DbContext` mới, đừng tự nghĩ ra format response khác, đừng tự chọn thư viện
validate khác.** Những thứ này đã chốt trong `DECISIONS.md`; làm khác sẽ phải sửa lại sau.

### 2.1 Repo đã được dọn sạch code mẫu

Rác do template `dotnet new` sinh ra (`/weatherforecast`, 3 file `Class1.cs`, file `.http` mẫu)
**đã bị xóa**. Mọi thứ còn lại trong `backend/` đều là code thật của dự án — cứ yên tâm dùng
làm mẫu. `NhaTre.API.http` nay chứa sẵn 2 request thật (login + me), mở bằng VS/VS Code là
bấm gửi được ngay.

---

## 3. Setup môi trường lần đầu — mỗi người tự làm 1 lần

### 3.1 Clone repo

```bash
git clone <url repo nhóm>
```

```bash
git checkout develop
```

```bash
git pull
```

Sau bước này, `ls backend` phải thấy 4 thư mục `NhaTre.*`. **Nếu `backend` trống hoặc không
tồn tại → dừng lại, báo Lead** (mục 0).

### 3.2 Nhận secrets từ Lead

Lead gửi riêng cho bạn 2 giá trị qua chat cá nhân:

1. Connection string Supabase (copy nguyên chuỗi, **không tự ghép từng phần**).
2. `Jwt:Key` (chuỗi ≥ 32 byte).

> **Không dán 2 giá trị này vào bất kỳ file nào trong repo, không gửi lên group public,
> không commit.** Cả nhóm dùng chung 1 database — lộ chuỗi này là lộ toàn bộ dữ liệu.

### 3.3 Lưu vào user-secrets của riêng máy bạn

`UserSecretsId` đã nằm sẵn trong `NhaTre.API.csproj`, nên chỉ cần set giá trị:

```bash
cd backend && dotnet user-secrets set "ConnectionStrings:DefaultConnection" "<chuỗi Lead gửi>" --project NhaTre.API
```

```bash
cd backend && dotnet user-secrets set "Jwt:Key" "<chuỗi Lead gửi>" --project NhaTre.API
```

Kiểm tra lại:

```bash
cd backend && dotnet user-secrets list --project NhaTre.API
```

Phải thấy đúng 2 key. user-secrets nằm ngoài thư mục dự án nên không bao giờ bị commit nhầm.

### 3.4 Build và chạy

```bash
cd backend && dotnet build
```

```bash
cd backend && dotnet run --project NhaTre.API
```

`dotnet run` mặc định chạy profile `http` → **http://localhost:5015**. Mở
http://localhost:5015/swagger, thấy `AuthController` liệt kê sẵn là setup đúng.

### 3.5 Test đăng nhập

Tài khoản Admin bootstrap (chỉ để kiểm tra setup — Lead sẽ đổi mật khẩu ngay khi có chức năng
quản lý user, đừng dùng làm tài khoản demo cuối cùng):

PowerShell:

```bash
Invoke-RestMethod -Uri "http://localhost:5015/api/auth/login" -Method Post -ContentType "application/json" -Body '{"email":"admin@nhatre.local","password":"Admin@123456"}'
```

Trả về `success: true` kèm `token` là bạn đã sẵn sàng code.

**Test endpoint cần đăng nhập:** Swagger UI hiện chưa tự gắn token (D9). Dùng Postman
(khuyến nghị — D30) và tự thêm header `Authorization: Bearer <token>`.

### 3.6 Lỗi hay gặp ngày đầu

| Triệu chứng | Nguyên nhân | Cách sửa |
| :---- | :---- | :---- |
| Lỗi `NETSDK1045 ... net10.0` khi build | Máy chỉ có SDK 8/9 | Cài .NET 10 SDK (mục 1) |
| `Thiếu cấu hình Jwt:Key trong user-secrets` lúc chạy | Chưa set secret, hoặc set nhầm project | Làm lại 3.3, nhớ `--project NhaTre.API` |
| Không mở được `NhaTreManagement.slnx` | IDE cũ chưa hỗ trợ `.slnx` | Nâng IDE, hoặc dùng `dotnet build` + mở từng `.csproj` |
| Timeout khi kết nối DB | Sai connection string, hoặc mạng chặn cổng 5432 | Đối chiếu lại chuỗi Lead gửi; thử mạng khác / 4G |
| Gọi API trả 401 dù token vừa lấy | Header thiếu chữ `Bearer `, hoặc tài khoản bị khóa | Xem lại header; đọc `message` trong body 401 |
| Login trả 429 dù mật khẩu đúng | Đã thử quá 5 lần trong 1 phút từ cùng 1 IP (tính chung cả nhóm nếu dùng chung Wi-Fi) | Đợi 1 phút — xem 5.6 |
| Frontend gọi API bị chặn, console báo CORS | Next.js đang chạy ở cổng khác 3000 (cổng 3000 bận nên tự nhảy 3001) | Giải phóng cổng 3000, hoặc báo Lead thêm origin vào `Cors:AllowedOrigins` — xem 4.1 |

---

## 4. Dành cho Frontend Dev

### 4.0 ⛔ Dự án frontend chưa tồn tại

Thư mục `frontend/` hiện **trống**. Trước khi 2 bạn Frontend bắt đầu, **một người duy nhất**
(Lead chỉ định) khởi tạo và push, người còn lại `git pull` về dùng — **không ai tự chạy
`create-next-app` riêng**, vì 2 bản scaffold khác nhau sẽ xung đột rất nặng.

Stack đã chốt, không thay thế:

| Hạng mục | Công nghệ | Quyết định |
| :---- | :---- | :---- |
| Framework | Next.js + TypeScript | D10 |
| Styling | Tailwind CSS | D11 |
| Form | React Hook Form | D12 |
| Server state | TanStack Query | D13 |
| HTTP client | Axios | D14 |
| Auth | React Context tự viết (**không dùng NextAuth**) | D20 |

Base URL backend đặt trong `.env.local` (đã gitignore):

```bash
NEXT_PUBLIC_API_BASE_URL=http://localhost:5015
```

> Backend có bật `UseHttpsRedirection`, nhưng khi chạy profile `http` mặc định thì không ảnh
> hưởng. Luôn gọi `http://localhost:5015`; đừng dùng cổng https 7137 trừ khi Lead báo đổi.

### 4.1 CORS — ✅ đã bật, bạn không cần làm gì

Backend đã cấu hình CORS theo đúng D20: **chỉ mở cho origin cụ thể** (không `AllowAnyOrigin`),
cho phép header `Authorization`, tự xử lý preflight `OPTIONS`, không dùng `AllowCredentials`
(vì token đi trong header chứ không phải cookie).

Danh sách origin nằm trong `backend/NhaTre.API/appsettings.json`:

```json
"Cors": {
  "AllowedOrigins": [ "http://localhost:3000" ]
}
```

Nên **Next.js phải chạy đúng cổng 3000**. Nếu cổng 3000 bận và Next tự nhảy sang 3001, trình
duyệt sẽ chặn request — lúc đó hoặc giải phóng cổng 3000, hoặc báo Lead thêm origin mới vào
`appsettings.json` (chỉ sửa file cấu hình, không phải sửa code).

Đã test thật, cả 3 trường hợp đều đúng:

| Request | Kết quả |
| :---- | :---- |
| Preflight `OPTIONS` từ `http://localhost:3000` | `204` + `Access-Control-Allow-Origin: http://localhost:3000`, `Allow-Headers: content-type,authorization` |
| `POST /api/auth/login` từ `http://localhost:3000` | `200` + header CORS đầy đủ, trả token bình thường |
| Request từ origin lạ | **không** có header `Access-Control-Allow-Origin` → trình duyệt chặn, đúng như mong muốn |

### 4.2 Luồng đăng nhập

```text
POST http://localhost:5015/api/auth/login
Body: { "email": "...", "password": "..." }
```

Response (`ApiResponse<LoginResponse>`):

```json
{
  "success": true,
  "data": {
    "token": "...",
    "expiresAtUtc": "2026-09-12T20:15:00Z",
    "userId": "guid",
    "fullName": "...",
    "role": "Admin"
  },
  "message": null,
  "errors": null
}
```

`role` là 1 trong 5 giá trị cố định: `Admin` | `Teacher` | `Accountant` | `Medical` | `Parent`
— trùng hằng số `NhaTre.Domain.Constants.Roles` phía backend, **không bao giờ là tiếng Việt**.
Dùng `role` để quyết định hiển thị route/menu nào.

Lưu token: React Context (in-memory) là nguồn chính, kèm 1 bản trong `localStorage` chỉ để
khôi phục phiên khi reload trang (D20 — rủi ro XSS đã được chấp nhận có chủ đích cho quy mô đồ án).

### 4.3 Gọi các API khác — luôn đính token

Cấu hình **1 Axios instance dùng chung**, interceptor tự gắn header cho mọi request:

```ts
api.interceptors.request.use((config) => {
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});
```

Endpoint kiểm tra token còn sống: `GET /api/auth/me` → trả `{ userId, role }` trong `data`.

### 4.4 Xử lý response — chỉ có MỘT khuôn duy nhất

**Mọi** response từ backend, không trừ mã lỗi nào, đều đúng khuôn này (D38):

```ts
{ success: boolean, data: T | null, message: string | null, errors: string[] | null }
```

Nghĩa là bạn chỉ cần viết **một** hàm đọc lỗi dùng chung, không phải đoán kiểu response:

```ts
const msg = res?.data?.message ?? 'Đã xảy ra lỗi, vui lòng thử lại.';
const chiTiet = res?.data?.errors ?? [];   // luôn là string[] hoặc null
```

Cách phân biệt 2 kiểu lỗi để hiển thị:

- `success: false` + `errors` có giá trị → hiển thị **danh sách** lỗi validate dưới từng ô nhập
  (VD: `["Email không đúng định dạng.", "Mật khẩu không được để trống."]`).
- `success: false` + chỉ có `message` → hiển thị **1 thông báo** đơn (VD: sai mật khẩu).

Bảng dưới đây là kết quả **test thật bằng curl**, không phải mô tả lý thuyết — cứ tin và code theo:

| Tình huống | HTTP | `message` trả về |
| :---- | :---- | :---- |
| Thiếu token / token sai | 401 | `Chưa đăng nhập hoặc token không hợp lệ.` |
| **Token hết hạn** | 401 | `Phiên đăng nhập đã hết hạn, vui lòng đăng nhập lại.` |
| Tài khoản bị khóa / bị đổi vai trò | 401 | `Tài khoản đã bị vô hiệu hóa.` / `Vai trò tài khoản đã thay đổi...` |
| Sai vai trò | 403 | `Bạn không có quyền thực hiện hành động này.` |
| Sai email/mật khẩu | 401 | `Email hoặc mật khẩu không đúng.` |
| Validate thất bại | 400 | `Dữ liệu không hợp lệ.` + mảng `errors` |
| JSON sai kiểu / thiếu field | 400 | `Dữ liệu không hợp lệ.` + `errors` dạng `Trường 'email' bị thiếu hoặc sai kiểu dữ liệu.` |
| Gọi sai đường dẫn | 404 | `Không tìm thấy đường dẫn yêu cầu.` |
| Sai HTTP method | 405 | `Phương thức HTTP không được hỗ trợ cho đường dẫn này.` |
| Quên `Content-Type: application/json` | 415 | `Định dạng dữ liệu không được hỗ trợ...` |
| Thử đăng nhập quá 5 lần / phút | 429 | `Bạn đã thử đăng nhập quá nhiều lần. Vui lòng đợi 1 phút rồi thử lại.` |
| Lỗi hệ thống | 500 | `Đã xảy ra lỗi hệ thống. Vui lòng thử lại sau...` |

Xử lý trong Axios response interceptor:

- **401 → xóa token khỏi Context + localStorage → redirect `/login`** (D20). Nếu `message` là
  câu "Phiên đăng nhập đã hết hạn", hiện luôn câu đó cho user trước khi chuyển trang.
- **403 → KHÔNG logout**, chỉ hiện "Bạn không có quyền truy cập chức năng này".
- **404/405/415** → thường là bug gọi sai API của chính mình, không phải lỗi người dùng; log ra
  console lúc dev.

### 4.5 Thời hạn token

Token sống **10 giờ** (khớp giờ hoạt động 7h–17h, D20). **Không có refresh token** — hết hạn
là phải đăng nhập lại. Nên đặt 1 timer dựa trên `expiresAtUtc` trả về lúc login để chủ động
logout đúng lúc, thay vì để user gặp 401 giữa chừng khi đang nhập dở form.

---

## 5. Dành cho Backend Dev — viết module mới đúng mẫu

### 5.1 Luôn theo đúng 4 lớp

```text
Controller (API)      → nhận request, validate, gọi Service, trả ApiResponse<T>
Service (Application) → business logic, gọi Repository interface
Repository interface  → định nghĩa ở Application (VD: IChildRepository)
Repository impl       → viết ở Infrastructure (VD: ChildRepository)
                        ĐÂY LÀ NƠI DUY NHẤT được gọi DbContext
```

Mở 4 file mẫu này ra đọc trước khi viết dòng đầu tiên, copy y hệt cấu trúc:

- `NhaTre.API/Controllers/AuthController.cs`
- `NhaTre.Application/Interfaces/IAuthService.cs` + `NhaTre.Application/Services/AuthService.cs`
- `NhaTre.Application/Interfaces/IAuthRepository.cs` + `NhaTre.Infrastructure/Persistence/Repositories/AuthRepository.cs`
- `NhaTre.Application/Validators/Auth/LoginRequestValidator.cs`

### 5.2 Checklist tạo 1 Controller mới (VD: `AttendanceController`)

1. DTO request/response trong `NhaTre.Application/DTOs/{Module}/` — dùng `record`, **không dùng Entity làm DTO**.
2. Validator trong `NhaTre.Application/Validators/{Module}/`, kế thừa `AbstractValidator<T>`, message **tiếng Việt**.
3. Interface Repository trong `NhaTre.Application/Interfaces/` — **chỉ khai báo đúng method cần dùng**, không làm generic repository cho mọi entity (D35).
4. Implement Repository trong `NhaTre.Infrastructure/Persistence/Repositories/`.
5. Interface Service + implement Service trong `NhaTre.Application/`.
6. Controller trong `NhaTre.API/Controllers/`, gắn `[Authorize(Roles = Roles.Xxx)]` (hằng số ở `NhaTre.Domain.Constants.Roles`, **không gõ chuỗi tay**).
7. Đăng ký DI trong `Program.cs` — **3 dòng, dòng thứ 3 hay bị quên**:
   ```csharp
   builder.Services.AddScoped<IAttendanceRepository, AttendanceRepository>();
   builder.Services.AddScoped<IAttendanceService, AttendanceService>();
   builder.Services.AddScoped<IValidator<CheckInRequest>, CheckInRequestValidator>();
   ```
   Thiếu dòng validator → app **crash lúc gọi endpoint**, không phải lúc build.
8. Build, test bằng Postman, đối chiếu response đúng khuôn `ApiResponse<T>`.

### 5.3 Quy ước route & status code

👉 **Bản đầy đủ nằm ở [`docs/api/api-conventions.md`](../api/api-conventions.md)** — có Controller
mẫu hoàn chỉnh, quy tắc đặt tên DTO, phân quyền theo dữ liệu, ranh giới 400 / 409 / 403 / 404,
và checklist trước khi mở PR. **Đọc file đó trước khi viết Controller đầu tiên.**

Dưới đây chỉ là bản rút gọn để tra nhanh:

| Hạng mục | Quy ước | Ví dụ |
| :---- | :---- | :---- |
| Route gốc | `[Route("api/{tài-nguyên-số-nhiều}")]`, chữ thường, gạch nối | `api/attendances`, `api/registered-pickup-persons` |
| Lấy danh sách | `GET api/attendances?classId=..&date=..` | |
| Lấy 1 bản ghi | `GET api/attendances/{id}` | |
| Tạo mới | `POST api/attendances` | |
| Cập nhật | `PUT api/attendances/{id}` | |
| Xóa | `DELETE api/attendances/{id}` | |
| Hành động không CRUD | động từ đặt cuối | `POST api/attendances/{id}/check-out` |

| Tình huống | Status | Trả về |
| :---- | :---- | :---- |
| Thành công, có dữ liệu | 200 | `Ok(ApiResponse<T>.Ok(data))` |
| Tạo mới thành công | 201 | `CreatedAtAction(..., ApiResponse<T>.Ok(data))` |
| Validate thất bại | 400 | `BadRequest(ApiResponse<T>.Fail(errors))` |
| Vi phạm nghiệp vụ (VD: điểm danh 2 lần trong ngày) | 409 | `Conflict(ApiResponse<T>.Fail("..."))` |
| Chưa đăng nhập / token hỏng | 401 | để middleware tự xử lý, **không tự viết** |
| Sai vai trò | 403 | để `[Authorize(Roles = ...)]` tự xử lý |
| Không tìm thấy bản ghi | 404 | `NotFound(ApiResponse<T>.Fail("Không tìm thấy ..."))` |
| Lỗi ngoài dự đoán | 500 | **không bắt**, để `GlobalExceptionHandler` lo |

Message trả cho client viết **tiếng Việt, có dấu, đủ nghĩa cho người dùng cuối** — không lộ
tên bảng, tên cột, hay nội dung exception (NFR-SEC-03).

### 5.4 `ApiResponse<T>` — đã phủ toàn bộ, bạn không phải làm gì thêm

`ApiResponse<T>` bắt buộc cho mọi thứ đi ra từ Controller. Phần **401 / 403 / 404 / 405 / 415 /
500** đã được xử lý tập trung một lần trong `Program.cs` (D38) — **không tự viết lại trong
Controller**, không tự bắt `UnauthorizedAccessException`, không tự trả `ProblemDetails`.

Cụ thể, 4 cơ chế đang chạy sẵn cho bạn:

| Cơ chế trong `Program.cs` | Lo giùm bạn việc gì |
| :---- | :---- |
| `JwtBearerEvents.OnChallenge` / `OnForbidden` | 401 (kể cả token hết hạn) và 403 |
| `InvalidModelStateResponseFactory` | 400 khi JSON sai kiểu / thiếu field |
| `SuppressMapClientErrors` + `UseStatusCodePages` | 404 / 405 / 415 |
| `GlobalExceptionHandler` | 500 |

> ⚠️ Khi viết validator hoặc message lỗi, **không để lộ tên class/bảng/cột hay nội dung
> exception** ra ngoài (NFR-SEC-03). Đây là lỗi thật đã từng xảy ra: mặc định .NET nhét cả
> `NhaTre.Application.DTOs.Auth.LoginRequest` vào message lỗi parse JSON — đã chặn ở D38.

### 5.5 Lưu ý đã học được — đừng lặp lại

- Foreign key kiểu "người ghi nhận" (`recorded_by_...`) dùng `OnDelete(DeleteBehavior.Restrict)`,
  không dùng Cascade mặc định — xem D37.
- `options.MapInboundClaims = false` đã bật sẵn trong `Program.cs`. Nếu thiếu, .NET đổi tên
  claim `sub`/`role` thành URI dài và mọi `User.FindFirst("sub")` thất bại **âm thầm** với 401
  rất khó chẩn đoán. Thêm claim mới → phải sửa đồng bộ cả nơi phát hành (`TokenService`) lẫn
  nơi đọc (`Controller` / `ActiveUserMiddleware`).
- Mọi request đã đăng nhập đều tốn thêm 1 query nhẹ theo khóa chính (`ActiveUserMiddleware`
  kiểm tra `is_active` + `role_id`). Đó là chủ ý (D20) — đừng "tối ưu" bỏ đi.
- Chuẩn hóa email `Trim().ToLowerInvariant()` ở tầng Application trước khi lưu **và** trước khi
  so sánh — áp dụng cho mọi chỗ đụng tới `login_identifier` (D20).

### 5.6 Rate limit trên `/api/auth/login` (D20)

Đã bật sẵn: **5 lần thử / 1 phút / 1 địa chỉ IP**. Vượt ngưỡng, backend trả `429` kèm
`ApiResponse` với message *"Bạn đã thử đăng nhập quá nhiều lần. Vui lòng đợi 1 phút rồi thử lại."*

Ba điều cần biết:

- Chỉ áp cho **endpoint đăng nhập**, không áp toàn hệ thống — giáo viên điểm danh cả lớp sẽ bắn
  nhiều request hợp lệ liên tiếp, chặn hết là hỏng nghiệp vụ.
- Đếm **theo IP**, nên khi cả nhóm test chung một mạng Wi-Fi, 5 lượt thử đó là **dùng chung**.
  Đang demo mà gặp 429 bất ngờ thì thường là bạn ngồi cạnh vừa gõ sai mật khẩu.
- **Frontend phải xử lý 429**: hiện thẳng `message` trả về và tạm khóa nút đăng nhập, đừng để
  user bấm dồn thêm.

Muốn sửa ngưỡng → `Program.cs`, phần `AddRateLimiter` (`PermitLimit` / `Window`). Đổi thì báo
Lead, vì đây là con số đã chốt theo D20.

---

## 6. Database dùng chung — quy tắc bắt buộc

Cả nhóm **kết nối vào đúng 1 database Supabase duy nhất**. Không có DB riêng cho từng máy.
Hệ quả:

- ⛔ **Không tự tạo migration mới** (`dotnet ef migrations add ...`) khi chưa hỏi Lead. Schema
  20 bảng đã khớp `database-design.md`; nếu bạn thấy thiếu cột, đó là dấu hiệu cần bàn lại
  thiết kế (quy trình D36), không phải tự thêm.
- ⛔ **Tuyệt đối không** chạy `dotnet ef database drop`, không `database update` về migration
  cũ, không xóa bảng bằng tay trên Supabase Studio. Một lệnh sai làm mất dữ liệu của cả 5
  người còn lại.
- ⚠️ Dữ liệu test bạn tạo ra, người khác **cũng nhìn thấy**. Đặt tên dễ nhận biết
  (VD: `Bé Test Đức 01`) và tự dọn khi xong.
- Ứng dụng **không** tự chạy migration lúc khởi động; nó chỉ seed tài khoản Admin khi bảng
  `users` hoàn toàn trống. Chạy app lần đầu không làm hỏng gì.

---

## 7. Git workflow

- `develop` là nhánh tích hợp — **không code trực tiếp trên `develop`**.
- `main` chỉ nhận merge từ `develop` khi có bản ổn định, **do Lead quyết định thời điểm**.
  Không ai tự mở PR vào `main`.
- Mỗi module làm trên nhánh riêng. 8 nhánh đã có sẵn trên remote:
  `feature/auth`, `feature/student`, `feature/teacher`, `feature/class`, `feature/attendance`,
  `feature/health`, `feature/tuition`, `feature/notification`.
  Frontend **chưa có nhánh** — Lead tạo thêm `feature/fe-teacher-medical` và
  `feature/fe-accounting-parent` (hoặc tên tương đương) trước khi 2 bạn FE bắt đầu.

Vòng làm việc chuẩn cho mỗi module:

```bash
git checkout develop && git pull
```

```bash
git checkout feature/attendance && git merge develop
```

```bash
git push origin feature/attendance
```

Sau đó mở Pull Request vào `develop`, gán Lead review.

**Commit message:** `<loại>: <mô tả ngắn tiếng Việt>` — loại gồm `feat` / `fix` / `refactor` /
`docs` / `chore`. Ví dụ: `feat: thêm API điểm danh buổi sáng cho giáo viên`.

### 7.1 Definition of Done — PR chỉ được review khi đủ các mục này

- [ ] `dotnet build` sạch, **0 warning** (backend) / `npm run build` sạch (frontend).
- [ ] Đã tự test bằng Postman: 1 case thành công + 1 case validate lỗi + 1 case sai role.
- [ ] Response đúng khuôn `ApiResponse<T>` và đúng bảng status code ở 5.3.
- [ ] Đã đăng ký đủ DI (repository + service + validator).
- [ ] Không hard-code secret, không commit connection string / token.
- [ ] Không thêm package mới (D34 — muốn thêm phải hỏi Lead trước).
- [ ] Không sửa file của module người khác; đụng `Program.cs` thì chỉ thêm dòng DI của mình.
- [ ] Mô tả PR ghi rõ: làm gì, endpoint nào, mã FR tương ứng, đã test ra sao.

---

## 8. Phân công

### 8.1 Backend

- **Đức** — Giáo viên + Y tế: `attendances`, `pickups`, `registered_pickup_persons`,
  `quick_health_statuses`, `growth_measurements`, `incidents`, `incident_photos`, `activity_photos`
- **Giang** — Kế toán/Văn phòng + Phụ huynh + Admin (trừ dashboard): `children`, `classes`,
  `teachers`, `tuition_fees`, `invoices`, `payments`, `weekly_menus`, `menu_entries`,
  `notifications`, `users` (quản lý)

### 8.2 Frontend

- **Tiền** — Giao diện Giáo viên + Y tế
- **Trang** — Giao diện Kế toán/Văn phòng + Phụ huynh

### 8.3 Tester + Dashboard

- **Tiên** — Kiểm thử toàn hệ thống + Admin Dashboard:
  - **Admin Dashboard & thống kê** (FR-DASH-01 → FR-DASH-04): cả backend lẫn frontend.
  - **Xuất báo cáo Excel/PDF** (FR-TUITION-04).
  - **Kiểm thử theo kịch bản người dùng** (D31): chạy đủ 5 nhóm scenario — Admin, Giáo viên,
    Kế toán/Văn phòng, Y tế, Phụ huynh — đối chiếu với `docs/requirements/acceptance-criteria.md`.
  - Dựng và duy trì **Postman Collection dùng chung** cho cả nhóm (D30), để 4 bạn kia không
    phải mỗi người tự gõ lại request.

> ⚠️ Lưu ý tên: **Tiên** (Tester + Dashboard) khác với **Tiền** (Frontend Giáo viên + Y tế).
> Khi gán task trên Trello và ghi mô tả PR, kiểm tra kỹ dấu để không gán nhầm người.

Dashboard đọc dữ liệu tổng hợp từ **cả hai** phần backend của Đức và Giang, nên phần này chỉ
bắt đầu được khi các module nguồn đã có endpoint. Trong lúc chờ, ưu tiên làm kiểm thử và
Postman Collection trước.

### 8.4 ⛔ Phần chưa có người nhận

- **Quản lý roles / permissions** (FR-USER-02) — đề xuất gắn cho Giang vì đã phụ trách
  `users`, nhưng **Lead xác nhận lại** trước khi bắt đầu.
- **Khởi tạo dự án frontend** (mục 4.0) — Lead chỉ định 1 trong 2 bạn FE.

### 8.5 ⚠️ Module phụ thuộc quyết định CHƯA CHỐT — hỏi Lead trước khi code

Năm quyết định dưới đây vẫn `Pending` trong `DECISIONS.md` §23. Nếu module bạn được giao nằm
trong bảng này, **làm phần không phụ thuộc trước**, và hỏi Lead trước khi chốt schema hoặc
luồng nghiệp vụ:

| Quyết định chưa chốt | Ảnh hưởng tới | Của ai |
| :---- | :---- | :---- |
| D23 — Trạng thái điểm danh | `attendances` (lõi module) | Đức |
| D24 — Dữ liệu người đón trẻ | `pickups`, `registered_pickup_persons` | Đức |
| D22 — Nơi lưu ảnh / video | `incident_photos`, `activity_photos` | Đức |
| D21 — Nhà cung cấp thanh toán | `payments` (thanh toán online) | Giang |
| D25 — Cơ chế thông báo | `notifications` | Giang |

Phần lưu ảnh và thanh toán đã có sẵn ranh giới kiến trúc (`IFileStorageService`,
`IPaymentService` — `architecture.md` §10, §11): cứ viết nghiệp vụ dựa trên interface, phần
kết nối nhà cung cấp làm sau khi có quyết định.

---

## 9. Tài liệu bắt buộc đọc trước khi code

| File | Nội dung |
| :---- | :---- |
| `DECISIONS.md` | Mọi quyết định kỹ thuật đã chốt — đặc biệt **D20** (Auth), **D34/D35** (không tự thêm công nghệ, không over-engineering), **D36** (quy trình đổi quyết định), **D37** (delete behavior), **D38** (khuôn response) |
| `docs/api/api-conventions.md` | **Quy ước viết API — bắt buộc đọc trước khi viết Controller đầu tiên** |
| `docs/database/database-design.md` | Toàn bộ 20 bảng, traceability về Business Rules |
| `docs/architecture/architecture.md` | Kiến trúc 4 lớp, error handling, logging, ranh giới media/payment |
| `docs/business/business-rules.md` | Nghiệp vụ chi tiết từng module |
| `docs/business/user-roles.md` | Ma trận vai trò – quyền, dùng để quyết định `[Authorize(Roles = ...)]` |
| `docs/requirements/functional-requirements.md` | Mã FR của từng chức năng, dùng để đặt tên task và mô tả PR |

Quản lý task trên **Trello** (D33). Mọi PR nên ghi mã FR tương ứng.

---

## 10. Nguyên tắc chung

1. Không chắc chắn thì **hỏi Lead trước khi tự quyết định** thêm công nghệ / pattern / package mới (D34).
2. Ưu tiên code đơn giản, đúng quy mô đồ án. Không tạo abstraction chỉ để "trông chuyên nghiệp" (D35).
3. Muốn đổi một quyết định đã chốt → theo quy trình D36 (nêu lý do, đánh giá ảnh hưởng, thống
   nhất, cập nhật `DECISIONS.md`), **rồi mới code**.
4. Không bao giờ commit secret. Không log password / token (NFR-LOG-02).
5. Kẹt quá 30 phút ở cùng 1 lỗi → hỏi trong group, đừng ngồi im.
