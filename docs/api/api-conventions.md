# API Conventions — Kindergarten Management System

## 0. Document Information

| Mục | Nội dung |
| :---- | :---- |
| Phạm vi | Quy ước viết REST API cho backend ASP.NET Core của dự án |
| Nguồn | Suy ra từ `AuthController` (module đã chạy thật) + `architecture.md` §7–§9 + `DECISIONS.md` D18, D20, D35, D38 |
| Bắt buộc với | Mọi Controller mới. Đọc hết trước khi viết dòng đầu tiên |
| Cập nhật lần cuối | 12/09/2026 |

> Tài liệu này **mô tả cái đang chạy**, không phải đề xuất. Mọi ví dụ đều lấy từ code thật
> trong `backend/`. Muốn làm khác → theo quy trình D36, đừng tự quyết.

---

## 1. Nguyên tắc nền

1. **RESTful** (D18): tài nguyên là danh từ, hành động là HTTP method.
2. **Một Controller cho một nhóm chức năng**, khớp module trong `functional-requirements.md`
   (Auth, Attendance, Pickup, Health, Incident, Media, Tuition, Menu, Notification,
   User/Class/Teacher, Dashboard) — `architecture.md` §7.
3. **Mọi response đúng một khuôn `ApiResponse<T>`** (D38), kể cả lỗi.
4. **Không có endpoint nào không có phân quyền** — `architecture.md` §13. Public duy nhất là
   `POST /api/auth/login`.
5. **DTO tách khỏi Entity.** Không bao giờ trả thẳng Entity ra ngoài: chỉ lộ đúng thứ vai trò
   đó được xem (VD: phụ huynh xem hóa đơn không được thấy dữ liệu trẻ khác) — `architecture.md` §7.
6. **Đơn giản trước** (D35): không tạo abstraction chỉ để trông chuyên nghiệp.

---

## 2. Đặt tên route

### 2.1 Quy tắc

| Hạng mục | Quy tắc | Ví dụ đúng | Ví dụ sai |
| :---- | :---- | :---- | :---- |
| Tiền tố | luôn là `api/` | `api/attendances` | `attendances` |
| Tên tài nguyên | **danh từ số nhiều**, chữ thường | `api/children` | `api/Child`, `api/getChild` |
| Nhiều từ | ngăn bằng gạch nối | `api/registered-pickup-persons` | `api/registeredPickupPersons` |
| Định danh | tham số route `{id}` | `api/children/{id}` | `api/children?id=5` |
| Tài nguyên con | lồng 1 cấp, tối đa | `api/children/{id}/growth-measurements` | lồng 3–4 cấp |
| Hành động không CRUD | động từ đặt **cuối** | `POST api/attendances/{id}/check-out` | `POST api/checkOutAttendance` |
| Lọc / phân trang / sắp xếp | query string | `?classId=..&date=..&page=1` | nhét vào path |

### 2.2 Bảng route chuẩn cho một tài nguyên

Lấy `attendances` làm ví dụ — áp dụng tương tự cho mọi module:

| Method | Route | Ý nghĩa | Status thành công |
| :---- | :---- | :---- | :---- |
| `GET` | `api/attendances?classId=..&date=..` | Danh sách, có lọc | 200 |
| `GET` | `api/attendances/{id}` | Chi tiết 1 bản ghi | 200 |
| `POST` | `api/attendances` | Tạo mới | 201 |
| `PUT` | `api/attendances/{id}` | Cập nhật toàn bộ | 200 |
| `DELETE` | `api/attendances/{id}` | Xóa | 200 |
| `POST` | `api/attendances/{id}/check-out` | Hành động nghiệp vụ riêng | 200 |

> **Lưu ý về xóa:** phần lớn dữ liệu trong dự án **không được xóa cứng** (D37 — lịch sử điểm
> danh/sức khỏe/sự cố là bằng chứng nghiệp vụ). Với tài khoản người dùng, "xóa" nghĩa là
> `users.is_active = false` (D20), không phải `DELETE` bản ghi. Hỏi Lead trước khi viết
> endpoint `DELETE` cho bất kỳ bảng nào.

---

## 3. Khuôn response — `ApiResponse<T>` (D38)

### 3.1 Cấu trúc

```jsonc
{
  "success": true,      // bool  — thành công hay không
  "data": { },          // T?    — dữ liệu trả về, null khi lỗi
  "message": null,      // string? — thông báo cho người dùng cuối
  "errors": null        // string[]? — danh sách lỗi validate, null nếu không có
}
```

Định nghĩa tại `NhaTre.Application/common/ApiResponse.cs`, namespace `NhaTre.Application.Common`.

### 3.2 Ba cách dựng

```csharp
ApiResponse<T>.Ok(data)                    // success = true
ApiResponse<T>.Fail("Thông báo lỗi đơn")   // success = false, chỉ có message
ApiResponse<T>.Fail(errors)                // success = false, message mặc định + mảng errors
```

### 3.3 Những gì bạn KHÔNG phải làm

`Program.cs` đã xử lý tập trung các trường hợp sau — **không viết lại trong Controller**:

| Tình huống | Cơ chế lo giùm |
| :---- | :---- |
| 401 thiếu/sai/hết hạn token | `JwtBearerEvents.OnChallenge` |
| 403 sai vai trò | `JwtBearerEvents.OnForbidden` |
| 400 JSON sai kiểu / thiếu field | `ApiBehaviorOptions.InvalidModelStateResponseFactory` |
| 404 sai route, 405 sai method, 415 sai Content-Type | `SuppressMapClientErrors` + `UseStatusCodePages` |
| 429 gọi login quá nhiều | `AddRateLimiter` + `OnRejected` |
| 500 lỗi ngoài dự đoán | `GlobalExceptionHandler` |

Đừng bắt `Exception` chung chung trong Controller/Service để "cho chắc" — nuốt mất lỗi khiến
Serilog không ghi được gì, debug sẽ rất khổ (`architecture.md` §8).

---

## 4. Bảng status code

| Tình huống | Status | Cách trả |
| :---- | :---- | :---- |
| Lấy / cập nhật / xóa thành công | 200 | `Ok(ApiResponse<T>.Ok(data))` |
| Tạo mới thành công | 201 | `CreatedAtAction(nameof(GetById), new { id }, ApiResponse<T>.Ok(data))` |
| Validate thất bại | 400 | `BadRequest(ApiResponse<T>.Fail(errors))` |
| Sai thông tin đăng nhập | 401 | `Unauthorized(ApiResponse<T>.Fail("..."))` |
| Đúng vai trò nhưng không được đụng bản ghi này | 403 | `Forbid()` hoặc `StatusCode(403, ApiResponse<T>.Fail("..."))` |
| Không tìm thấy bản ghi | 404 | `NotFound(ApiResponse<T>.Fail("Không tìm thấy ..."))` |
| Vi phạm quy tắc nghiệp vụ | 409 | `Conflict(ApiResponse<T>.Fail("..."))` |
| Lỗi hệ thống | 500 | **không tự trả** — để `GlobalExceptionHandler` |

### 4.1 Khi nào 400, khi nào 409?

Ranh giới hay gây tranh cãi giữa 2 người viết, nên chốt cứng:

- **400** = dữ liệu gửi lên *tự nó* đã sai, chưa cần biết trong DB có gì.
  VD: thiếu `childId`, ngày sai định dạng, chiều cao là số âm.
- **409** = dữ liệu hợp lệ nhưng *xung đột với trạng thái hiện tại* trong DB.
  VD: điểm danh lần 2 trong cùng ngày cho cùng một bé; tạo lớp trùng tên; thanh toán một hóa
  đơn đã ở trạng thái `paid`.

Quy tắc nghiệp vụ bị vi phạm (BR-*) gần như luôn là **409** — `architecture.md` §8.

### 4.2 403 vs 404 khi truy cập dữ liệu người khác

Phụ huynh A gọi `GET api/children/{id}` với id của con nhà B:

- Trả **404** với message "Không tìm thấy trẻ." — **không** trả 403.
- Lý do: trả 403 tức là gián tiếp xác nhận "bản ghi này có tồn tại", làm lộ thông tin.

---

## 5. Viết message lỗi

| Quy tắc | Đúng | Sai |
| :---- | :---- | :---- |
| Tiếng Việt có dấu, đủ nghĩa cho người dùng cuối | `Không tìm thấy lớp học.` | `Class not found`, `Err_404` |
| Không lộ tên bảng / cột / class / namespace | `Dữ liệu gửi lên không hợp lệ.` | `FK violation on attendances.child_id` |
| Không lộ nội dung exception | `Đã xảy ra lỗi hệ thống...` | `NullReferenceException at line 42` |
| Nói được người dùng phải làm gì | `Bé này đã được điểm danh hôm nay rồi.` | `Duplicate record` |

Đây là yêu cầu bắt buộc (NFR-SEC-03), không phải gợi ý thẩm mỹ. Lỗi thật đã từng xảy ra trong
dự án: .NET mặc định nhét cả `NhaTre.Application.DTOs.Auth.LoginRequest` vào message lỗi parse
JSON — đã bị chặn ở D38.

---

## 6. Validation

- Dùng **FluentValidation** (D7), không dùng DataAnnotations.
- Validator đặt tại `NhaTre.Application/Validators/{Module}/`, kế thừa `AbstractValidator<T>`.
- Gọi **tường minh** trong Controller (không dùng auto-validation pipeline) — xem mẫu mục 8.
- Mọi message trong validator viết bằng tiếng Việt.
- Validate **luôn chạy lại ở server**, bất kể frontend đã kiểm tra (NFR-SEC-02).

Chỉ validate cái thuộc về *hình dạng dữ liệu* (bắt buộc, độ dài, định dạng, khoảng giá trị).
Kiểm tra *trạng thái nghiệp vụ* (bé đã điểm danh chưa, lớp còn chỗ không) thuộc về **Service**,
không nhét vào validator.

Ví dụ có thật — `LoginRequestValidator` cố ý **không** kiểm tra độ dài mật khẩu, vì đây là
endpoint đăng nhập (so khớp với hash đã có), không phải đăng ký.

---

## 7. Phân quyền

- Mọi action đều phải có `[Authorize]`. Ngoại lệ duy nhất: `POST api/auth/login`.
- Dùng **hằng số**, không gõ chuỗi tay:

```csharp
[Authorize(Roles = Roles.Teacher)]                    // 1 vai trò
[Authorize(Roles = $"{Roles.Teacher},{Roles.Medical}")] // nhiều vai trò
```

Hằng số ở `NhaTre.Domain.Constants.Roles`: `Admin`, `Teacher`, `Accountant`, `Medical`, `Parent`.
**Không bao giờ** dùng `roles.name` tiếng Việt trong DB làm claim (D20).

Tra vai trò nào được làm gì tại `docs/business/user-roles.md` §7 (ma trận quyền).

### 7.1 Phân quyền theo dữ liệu (rất hay bị quên)

`[Authorize(Roles = Roles.Parent)]` chỉ chặn được "có phải phụ huynh không", **không** chặn
được "có phải phụ huynh của *bé này* không". Với mọi endpoint mà `Parent` hoặc `Teacher` truy
cập dữ liệu của một trẻ/lớp cụ thể, Service **phải** kiểm tra thêm quan hệ sở hữu, rồi trả 404
theo mục 4.2.

Lấy `userId` của người đang đăng nhập từ claim `sub`:

```csharp
var userId = User.FindFirst(JwtRegisteredClaimNames.Sub)?.Value;
```

---

## 8. Controller mẫu đầy đủ

Đây là khung chuẩn, copy rồi đổi tên. Giữ nguyên thứ tự và cách xử lý.

```csharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using FluentValidation;
using NhaTre.Application.Common;
using NhaTre.Application.DTOs.Attendance;
using NhaTre.Application.Interfaces;
using NhaTre.Domain.Constants;

namespace NhaTre.API.Controllers;

[ApiController]
[Route("api/attendances")]
[Authorize(Roles = Roles.Teacher)]   // mặc định cho cả Controller
public class AttendanceController : ControllerBase
{
    private readonly IAttendanceService _attendanceService;
    private readonly IValidator<CheckInRequest> _checkInValidator;

    public AttendanceController(
        IAttendanceService attendanceService,
        IValidator<CheckInRequest> checkInValidator)
    {
        _attendanceService = attendanceService;
        _checkInValidator = checkInValidator;
    }

    [HttpGet]
    public async Task<ActionResult<ApiResponse<IReadOnlyList<AttendanceResponse>>>> GetByClassAndDate(
        [FromQuery] Guid classId,
        [FromQuery] DateOnly date)
    {
        var result = await _attendanceService.GetByClassAndDateAsync(classId, date);
        return Ok(ApiResponse<IReadOnlyList<AttendanceResponse>>.Ok(result));
    }

    [HttpGet("{id:guid}")]
    public async Task<ActionResult<ApiResponse<AttendanceResponse>>> GetById(Guid id)
    {
        var result = await _attendanceService.GetByIdAsync(id);

        if (result is null)
            return NotFound(ApiResponse<AttendanceResponse>.Fail("Không tìm thấy bản ghi điểm danh."));

        return Ok(ApiResponse<AttendanceResponse>.Ok(result));
    }

    [HttpPost]
    public async Task<ActionResult<ApiResponse<AttendanceResponse>>> CheckIn(
        [FromBody] CheckInRequest request)
    {
        // 1. Validate hình dạng dữ liệu
        var validationResult = await _checkInValidator.ValidateAsync(request);
        if (!validationResult.IsValid)
        {
            var errors = validationResult.Errors.Select(e => e.ErrorMessage).ToList();
            return BadRequest(ApiResponse<AttendanceResponse>.Fail(errors));
        }

        // 2. Gọi Service — mọi business logic nằm ở đó, không nằm ở đây
        var result = await _attendanceService.CheckInAsync(request);

        // 3. Dịch kết quả nghiệp vụ sang status code
        if (result is null)
            return Conflict(ApiResponse<AttendanceResponse>.Fail("Bé này đã được điểm danh hôm nay rồi."));

        return CreatedAtAction(nameof(GetById), new { id = result.Id },
            ApiResponse<AttendanceResponse>.Ok(result));
    }
}
```

### 8.1 Controller được phép làm gì

| Được | Không được |
| :---- | :---- |
| Nhận request, gọi validator | Viết business logic |
| Gọi Service | Gọi `DbContext` / `Repository` trực tiếp |
| Dịch kết quả sang status code | Tự query LINQ trên Entity |
| Đọc claim của người đăng nhập | Tự hash mật khẩu, tự sinh token |
| Trả `ApiResponse<T>` | Trả Entity, trả object ẩn danh tùy hứng |

Controller dài quá ~15 dòng cho một action thường là dấu hiệu logic đã lọt sai lớp.

---

## 9. DTO

- Đặt tại `NhaTre.Application/DTOs/{Module}/`.
- Dùng `record` (bất biến), như `LoginRequest` / `LoginResponse`.
- Đặt tên theo hành động: `CheckInRequest`, `AttendanceResponse`, `CreateChildRequest`.
- `DateTime` trả ra ngoài luôn là **UTC**, tên trường kết thúc bằng `Utc`
  (VD: `expiresAtUtc`) để frontend biết đường chuyển múi giờ.
- Không tái dùng một DTO cho cả request lẫn response nếu hai bên cần trường khác nhau.
- **Không bao giờ** đưa `password_hash` / `credential_reference` vào bất kỳ response nào.

JSON trả ra dùng **camelCase** (mặc định của ASP.NET Core) — C# viết `ExpiresAtUtc`, frontend
nhận `expiresAtUtc`. Không cấu hình lại.

---

## 10. Checklist trước khi mở Pull Request

- [ ] Route đúng mục 2 (số nhiều, chữ thường, gạch nối).
- [ ] Mọi action có `[Authorize]` với hằng số `Roles.*`.
- [ ] Đã kiểm tra phân quyền theo dữ liệu nếu endpoint đụng dữ liệu của một trẻ/lớp cụ thể (7.1).
- [ ] Response đều là `ApiResponse<T>`, status code đúng bảng mục 4.
- [ ] Message tiếng Việt, không lộ cấu trúc nội bộ (mục 5).
- [ ] Không gọi `DbContext` ngoài Repository.
- [ ] Đã đăng ký DI đủ 3 thứ: Repository, Service, **Validator**.
- [ ] Đã test bằng Postman: 1 case thành công, 1 case validate lỗi, 1 case sai vai trò.
- [ ] `dotnet build` sạch, 0 warning.

---

## 11. Tài liệu liên quan

| File | Dùng để |
| :---- | :---- |
| `DECISIONS.md` | D18 (REST), D20 (Auth), D35 (không over-engineering), D37 (delete behavior), D38 (response format) |
| `docs/architecture/architecture.md` | §7 API boundary, §8 error handling, §13 security |
| `docs/business/user-roles.md` | Ma trận vai trò – quyền |
| `docs/business/business-rules.md` | Mã BR-* để biết khi nào trả 409 |
| `docs/requirements/functional-requirements.md` | Mã FR-* để đặt tên task và mô tả PR |
| `docs/onboarding/onboarding-guide.md` | Setup máy, git workflow, phân công |
