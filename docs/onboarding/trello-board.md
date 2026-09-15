# Cấu trúc Trello Board — Hệ thống Quản lý Nhà trẻ

Bản đã audit và đã chốt hết câu hỏi treo, dùng để dựng board thật. Mọi mã FR/AC đều đối chiếu
với `docs/requirements/functional-requirements.md` và `docs/requirements/acceptance-criteria.md`.

**Cập nhật 15/09/2026:** 12 câu hỏi còn mở đã được Lead chốt (xem §6). Không còn thẻ nào bị
chặn bởi quyết định — mọi thẻ chỉ còn phụ thuộc vào thứ tự làm việc giữa các thành viên.

---

## 1. Cột (List)

| Cột | Dùng khi nào |
| :---- | :---- |
| **Backlog** | Thẻ đã tạo, chưa ai bắt đầu |
| **Chờ quyết định** | Thẻ bị chặn bởi một câu hỏi chưa chốt. **Hiện đang trống** — giữ cột lại để dùng khi phát sinh câu hỏi mới |
| **Đang làm** | Mỗi người **tối đa 2 thẻ** cùng lúc, để tránh làm dở dang nhiều việc |
| **Chờ Review (PR)** | Đã mở Pull Request vào `develop`, chờ Lead review |
| **Done** | PR đã merge vào `develop` |

---

## 2. Nhãn (Label)

**Theo người:** Đức 🔵 · Giang 🟢 · Tiền 🟡 · Trang 🟠 · Tiên 🟣 · Lead ⚫

**Theo tầng** (nhãn thứ hai, để lọc nhanh): `BE` · `FE` · `Test` · `Docs`

Một thẻ luôn có **đúng 2 nhãn**: một người + một tầng. Lọc theo `BE` là thấy toàn bộ việc
backend bất kể của ai — tiện khi Lead review tiến độ theo tầng.

---

## 3. Mẫu thẻ

### 3.1 Thẻ Backend

```
Tiêu đề: [FR-XXX-YY] Mô tả ngắn bằng tiếng Việt

Mô tả:
- Tiêu chí nghiệm thu: AC-XXX-YY (chép Cho trước/Khi/Thì từ acceptance-criteria.md)
- Quy tắc nghiệp vụ: BR-XXX-YY
- Endpoint dự kiến: METHOD /api/...
- Vai trò được phép: [Authorize(Roles = Roles.Xxx)]
- Ràng buộc/Quyết định liên quan: Dxx (nếu có)

Checklist:
[ ] DTO request/response (dùng record, không dùng Entity làm DTO)
[ ] Validator FluentValidation, message tiếng Việt
[ ] Interface + implement Repository (chỉ Repository gọi DbContext)
[ ] Interface + implement Service
[ ] Controller, gắn [Authorize(Roles = ...)] bằng hằng số
[ ] Đăng ký DI đủ 3 dòng: Repository + Service + Validator
[ ] Kiểm tra phân quyền theo dữ liệu nếu endpoint đụng dữ liệu 1 trẻ/lớp cụ thể
[ ] Test Postman: 1 case đúng + 1 case validate lỗi + 1 case sai vai trò
[ ] Response đúng khuôn ApiResponse<T>, status code đúng bảng api-conventions §4
[ ] dotnet build sạch 0 warning
```

### 3.2 Thẻ Frontend

```
Tiêu đề: [FE][FR-XXX-YY] Mô tả ngắn bằng tiếng Việt

Mô tả:
- Màn hình: đường dẫn route, ví dụ (teacher)/attendance
- API dùng: METHOD /api/... (phải Done trước)
- Vai trò xem được màn hình này

Checklist:
[ ] Gọi API qua apiClient dùng chung, không tự tạo axios.create()
[ ] Đọc lỗi bằng toApiError(), hiển thị message tiếng Việt từ backend
[ ] Dùng React Hook Form cho mọi form (D12)
[ ] Dùng TanStack Query cho đọc/ghi dữ liệu (D13)
[ ] Có đủ 3 trạng thái: đang tải / lỗi / rỗng (AC-NFR-04)
[ ] Responsive trên cả desktop và điện thoại (AC-NFR-03)
[ ] npm run build sạch
```

---

## 4. Thẻ theo người

### 🔵 Đức — Backend Giáo viên + Y tế (**15 thẻ**)

| Thẻ | AC liên quan | Ghi chú |
| :---- | :---- | :---- |
| ~~[KT] Migration: thêm `mood` + `temperature`~~ → **kéo thẳng vào Done** | — | ✅ **Lead đã làm xong 15/09.** Migration `AddHealthMoodAndTemperature_RestrictPickupPerson` đã áp lên Supabase. Hằng số `HealthMoods` (4 giá trị) đã có trong `NhaTre.Domain.Constants` — Đức chỉ việc dùng, đừng gõ chuỗi tay |
| **[KT] Dựng `IFileStorageService` + hiện thực Supabase Storage** | — | **Làm TRƯỚC FR-INCIDENT-02 và FR-MEDIA-01.** Dùng chung cho cả hai, tránh viết trùng phần kết nối Supabase |
| [FR-ATT-01] Điểm danh vào lớp | AC-ATT-01, AC-ATT-02 | Trạng thái `Present`/`AbsentExcused`/`AbsentUnexcused` (D23). Unique (`child_id`,`attendance_date`) → điểm danh trùng trả **409** |
| [FR-ATT-02] API phụ huynh xem lịch sử điểm danh | AC-ATT-03, AC-ATT-04 | Lọc theo `child_guardians`; xem trẻ khác → trả **404** (không phải 403) |
| [FR-PICKUP-01] Điểm danh đón về | AC-PICKUP-01 | Unique theo `attendance_id` → mỗi bản điểm danh 1 lần đón |
| [FR-PICKUP-02] Đối chiếu người đón đã đăng ký | AC-PICKUP-02 | Giáo viên đối chiếu họ tên bằng mắt (D24). **Chốt:** người đón không khớp danh sách → **chặn**, không tạo bản ghi; giáo viên phải chọn "Phụ huynh trực tiếp đón" (khi đó `pickup_person_id` để rỗng) hoặc liên hệ phụ huynh. Không cần migration |
| [FR-PICKUP-03] API quản lý người đón (FE do Trang làm) | AC-PICKUP-03 | Chỉ lưu `full_name` (D24) |
| [FR-HEALTH-01] Ghi nhận sức khỏe nhanh | AC-HEALTH-01 | **Chốt trường (D39 mục 1):** `Mood` — dùng hằng số `HealthMoods.Happy/Normal/Tired/Fussy`, `TemperatureCelsius` (nullable, °C), `Notes`. **Schema đã sẵn sàng**, không phải migration nữa. Validator phải chặn giá trị `Mood` ngoài 4 giá trị — dùng `HealthMoods.IsValid()` |
| [FR-HEALTH-02] Theo dõi chiều cao | AC-HEALTH-02 | Check: ít nhất 1 trong 2 giá trị cao/nặng |
| [FR-HEALTH-03] Theo dõi cân nặng | AC-HEALTH-03 | Chung bảng `growth_measurements` với FR-HEALTH-02 |
| [FR-HEALTH-04] Xem lịch sử sức khỏe/sự cố | AC-HEALTH-04, AC-INCIDENT-04 | **Chốt:** gộp **cả 3 nguồn** theo thứ tự thời gian — `incidents` + `growth_measurements` + `quick_health_statuses`. Cả ba đã có `child_id`, chỉ là gộp truy vấn |
| [FR-INCIDENT-01] Ghi nhận sự cố | AC-INCIDENT-01 | |
| [FR-INCIDENT-02] Đính ảnh sự cố | AC-INCIDENT-02, AC-INCIDENT-03 | Supabase Storage, chỉ ảnh (D22). **Chốt giới hạn:** tối đa **5 ảnh/lần**, mỗi ảnh **≤ 5MB** — validate ở tầng Api trước khi đẩy xuống storage |
| [FR-MEDIA-01] Chia sẻ ảnh hoạt động theo lớp | AC-MEDIA-01 | Chỉ ảnh, không video (D22). Cùng giới hạn 5 ảnh / 5MB |
| [FR-MEDIA-02] API phụ huynh xem ảnh hoạt động | AC-MEDIA-02 | **Chốt:** giới hạn **chỉ con mình**, lọc qua `child_guardians` → `class_id`. Xem lớp khác → **404**, giống FR-ATT-02 |

### 🟢 Giang — Backend Kế toán/Văn phòng + Phụ huynh + Admin (**13 thẻ**)

| Thẻ | AC liên quan | Ghi chú |
| :---- | :---- | :---- |
| **[KT] Bổ sung `fullName` vào `GET /api/auth/me`** | — | **Làm sớm.** Hiện FE hiển thị tên rỗng sau khi reload — chặn phần header của mọi màn hình, cả Tiền lẫn Trang đều chờ |
| [FR-USER-01] Quản lý tài khoản | AC-USER-01 | Admin tự nhập mật khẩu khi tạo (D20). **Chốt:** chỉ **vô hiệu hóa** (`is_active = false`), **không xóa cứng** — nhất quán với D37 |
| [FR-USER-02] Quản lý vai trò/phân quyền | AC-USER-02 | **Chốt:** giao Giang. Phạm vi = **chỉ đổi vai trò của một user** trong 5 vai trò cố định; không tạo/sửa quyền chi tiết (khớp câu trả lời "role-only" của OQ #2) |
| [FR-CLASS-01] Quản lý lớp theo độ tuổi | AC-CLASS-01, AC-CLASS-02 | **Chốt:** 3 lớp cố định — **3-4 / 4-5 / 5-6 tuổi**. Điền `min_age_months`/`max_age_months` tương ứng (36-48, 48-60, 60-72) |
| [FR-STU-01] Hồ sơ nhập học trẻ | AC-STU-01 | **Chốt:** dùng đúng schema `Child` hiện có (`FullName`, `DateOfBirth`, `EnrollmentDate`, `ClassId`). Không thêm giấy tờ, không thêm cột trạng thái active/withdrawn. **Không cần migration** |
| [FR-TEACHER-01] Hồ sơ giáo viên | AC-TEACHER-01 | Quan hệ 1:1 với `users` |
| [FR-TUITION-01] Quản lý biểu phí | AC-TUITION-01 | **Chốt:** **1 mức phí chung toàn trường**, không phân theo lớp/tuổi/dịch vụ. `tuition_fees` giữ nguyên dạng danh mục phẳng |
| [FR-TUITION-02] Quản lý hóa đơn | AC-TUITION-02 | `status` chỉ `unpaid`/`paid` |
| [FR-TUITION-03] Báo cáo doanh thu | AC-TUITION-03 | Là nguồn dữ liệu cho Dashboard của Tiên — **ưu tiên làm sớm** |
| [FR-TUITION-05] API phụ huynh xem hóa đơn | AC-TUITION-05 | **Chốt:** giới hạn **chỉ con mình**, lọc qua `child_guardians`. Xem hóa đơn trẻ khác → **404** |
| [FR-TUITION-06] API thanh toán (mô phỏng) | AC-TUITION-06 | Luồng mô phỏng nội bộ, **UI phải ghi rõ "mô phỏng"** (D21). **Chốt:** luồng mô phỏng **không có đường thất bại** — bấm là thành công; chỉ chặn hóa đơn đã `paid` → **409** |
| [FR-MENU-01] Quản lý thực đơn tuần | AC-MENU-01 | Unique (`weekly_menu_id`,`day_of_week`). **Chốt:** chỉ Kế toán quản lý **và xem**; không làm màn hình xem cho phụ huynh (giữ đúng phạm vi CC-02) |
| [FR-NOTI-01] API thông báo | AC-NOTI-01 | Chỉ in-app, không cột `is_read` (D25). **Chốt 4 sự kiện tạo thông báo:** hóa đơn mới, sự cố sức khỏe, thông báo nghỉ học, có ảnh hoạt động mới |

### 🟡 Tiền — Frontend Giáo viên + Y tế (**7 thẻ**)

| Thẻ | Phụ thuộc |
| :---- | :---- |
| **[FE] Layout + menu điều hướng cho vai trò Giáo viên và Y tế** | **Không phụ thuộc ai — làm ngay** |
| **[FE] Chuyển trang đăng nhập sang React Hook Form** | **Không phụ thuộc ai — làm ngay.** Hiện là bản khung dùng `useState`, chưa đúng D12 |
| [FE][FR-ATT-01] Giao diện điểm danh vào lớp | Chờ Đức xong FR-ATT-01 |
| [FE][FR-PICKUP-01/02] Giao diện điểm danh đón về + đối chiếu người đón | Chờ Đức. Phải có nút "Phụ huynh trực tiếp đón" và chặn khi người đón không khớp danh sách |
| [FE][FR-HEALTH-01, FR-INCIDENT-01/02] Giao diện sức khỏe nhanh + ghi sự cố kèm ảnh | Chờ Đức. Form sức khỏe có 3 trường: tâm trạng (dropdown 4 giá trị), nhiệt độ (tùy chọn), ghi chú. Chặn upload quá 5 ảnh hoặc ảnh > 5MB ngay ở FE |
| [FE][FR-HEALTH-04] Giao diện xem lịch sử sức khỏe/sự cố | Chờ Đức. Hiển thị trộn 3 loại bản ghi theo thời gian, có nhãn phân biệt loại |
| [FE][FR-HEALTH-02/03] Giao diện Y tế theo dõi chiều cao/cân nặng | Chờ Đức |

### 🟠 Trang — Frontend Kế toán/Văn phòng + Phụ huynh (**10 thẻ**)

| Thẻ | Phụ thuộc |
| :---- | :---- |
| **[FE] Layout + menu điều hướng cho vai trò Kế toán và Phụ huynh** | **Không phụ thuộc ai — làm ngay** |
| **[FE] Bộ component dùng chung: bảng, form, phân trang, hộp thoại xác nhận** | **Không phụ thuộc ai — làm ngay.** Cả Tiền cũng dùng lại |
| [FE][FR-STU-01] Giao diện hồ sơ trẻ | Chờ Giang. Đúng 4 trường, không thêm |
| [FE][FR-TEACHER-01] Giao diện hồ sơ giáo viên | Chờ Giang |
| [FE][FR-TUITION-01/02] Giao diện học phí + hóa đơn | Chờ Giang. Biểu phí chỉ 1 mức chung |
| [FE][FR-MENU-01] Giao diện thực đơn tuần (chỉ Kế toán) | Chờ Giang |
| [FE][FR-ATT-02] Phụ huynh xem lịch sử điểm danh | Chờ **Đức** (FR-ATT-02) |
| [FE][FR-MEDIA-02] Phụ huynh xem ảnh hoạt động | Chờ **Đức** (FR-MEDIA-02) |
| [FE][FR-TUITION-05/06] Phụ huynh xem + thanh toán hóa đơn | Chờ Giang. **Phải hiện chữ "thanh toán mô phỏng"** (D21) |
| [FE][FR-PICKUP-03, FR-NOTI-01] Phụ huynh đăng ký người đón + xem thông báo | Chờ Đức (pickup) và Giang (thông báo) |

### 🟣 Tiên — Tester + Dashboard (**8 thẻ**)

| Thẻ | Ghi chú |
| :---- | :---- |
| **[Test] Dựng Postman Collection dùng chung** | **Làm ngay, không phụ thuộc ai.** Bắt đầu từ 2 request Auth đã chạy được |
| **[Test] Kiểm thử 5 tiêu chí phi chức năng** | **Làm được sớm.** AC-NFR-01 tới AC-NFR-05 — test được ngay trên phần Auth hiện có |
| [Test] Kịch bản người dùng — Giáo viên | Chạy song song khi Đức xong từng module (D31) |
| [Test] Kịch bản người dùng — Kế toán/Văn phòng | Chạy song song khi Giang xong từng module |
| [Test] Kịch bản người dùng — Y tế / Phụ huynh / Admin | Chạy song song. Nhớ test cả 3 luồng phụ huynh xem trẻ khác → phải ra **404** |
| [FR-DASH-01/02] Dashboard tổng quan + thống kê số trẻ | Chờ Giang xong FR-STU-01. **Chốt:** tổng số trẻ + tách theo 3 lớp |
| [FR-DASH-03] Thống kê điểm danh | Chờ Đức xong FR-ATT-01. **Chốt:** tỷ lệ có mặt **theo tháng hiện tại**, tách theo lớp. Nhớ lọc `status = 'Present'` (xem bẫy `check_in_time` ở D23) |
| [FR-DASH-04] Thống kê doanh thu + [FR-TUITION-04] Xuất Excel/PDF | Chờ Giang xong FR-TUITION-03. **Chốt:** tổng thu **tháng hiện tại** + số hóa đơn chưa thanh toán. Xuất file **chỉ cho báo cáo doanh thu**, không xuất hóa đơn |

### ⚫ Lead (**3 thẻ**)

| Thẻ | Ghi chú |
| :---- | :---- |
| [Done] FR-AUTH-01/02 Đăng nhập JWT + phân quyền theo vai trò | Đã xong — **tạo thẻ và kéo thẳng vào Done** để board phản ánh đủ 32 FR khi báo cáo |
| [Done] CORS + rate limit login | Đã xong |
| **[Docs] Ghi 12 quyết định mới ở §6 vào `DECISIONS.md`** | **Quan trọng.** Theo D36, quyết định phải nằm trong `DECISIONS.md` chứ không chỉ trong file Trello. Đề xuất gom thành **D39 — Chốt phạm vi chi tiết các module** |

---

## 5. Tổng số thẻ

| Người | Số thẻ | Bắt đầu được ngay | Chờ người khác |
| :---- | ----: | ----: | ----: |
| Đức 🔵 | 15 | 15 | 0 |
| Giang 🟢 | 13 | 13 | 0 |
| Tiền 🟡 | 7 | 2 | 5 |
| Trang 🟠 | 10 | 2 | 8 |
| Tiên 🟣 | 8 | 2 | 6 |
| Lead ⚫ | 3 | 3 | 0 |
| **Tổng** | **56** | **37** | **19** |

**Không còn thẻ nào bị chặn bởi câu hỏi chưa chốt.** 19 thẻ còn lại chỉ chờ theo thứ tự làm
việc bình thường giữa backend và frontend.

---

## 6. Các quyết định đã chốt (15/09/2026)

Trước đây board có 8 câu hỏi treo ở mục này, cộng thêm 4 câu phát sinh khi rà soát. Tất cả đã
được Lead chốt. Bảng dưới là nguồn tra cứu nhanh; **bản chính thức phải được ghi vào
`DECISIONS.md`** theo quy trình D36 — xem thẻ của Lead.

| # | Câu hỏi | Chốt | Ảnh hưởng |
| :---- | :---- | :---- | :---- |
| 1 | Trường "sức khỏe nhanh" | `mood` (4 giá trị) + `temperature` (nullable) + `note` | ⚠️ **Cần migration** |
| 2 | Trường bắt buộc hồ sơ trẻ | Đúng schema `Child` hiện có, không thêm gì | Không đổi schema |
| 3 | Ranh giới độ tuổi lớp | 3 lớp: 3-4 / 4-5 / 5-6 tuổi | Dữ liệu mẫu |
| 4 | Cấu trúc biểu phí | 1 mức chung toàn trường | Không đổi schema |
| 5 | Giới hạn ảnh upload | ≤ 5 ảnh/lần, mỗi ảnh ≤ 5MB | Validate tầng Api |
| 6 | Sự kiện tạo thông báo | 4 loại: hóa đơn mới, sự cố sức khỏe, nghỉ học, ảnh mới | Nghiệp vụ |
| 7 | Vô hiệu hóa hay xóa tài khoản | Chỉ `is_active = false`, không xóa cứng | Nghiệp vụ |
| 8 | Báo cáo nào xuất Excel/PDF | Chỉ báo cáo doanh thu | Thu hẹp phạm vi |
| 9 | Lịch sử Y tế gồm những gì | Cả 3 nguồn: sự cố + số đo + sức khỏe nhanh | Gộp truy vấn |
| 10 | Phạm vi phụ huynh xem ảnh/hóa đơn | Chỉ con mình, giống điểm danh → sai phạm vi trả **404** | Lọc `child_guardians` |
| 11 | Người đón không có trong danh sách | **Chặn**, bắt chọn "Phụ huynh trực tiếp đón" hoặc liên hệ phụ huynh | Không đổi schema |
| 12 | Phân tách thống kê dashboard | Tổng + theo lớp, khoảng thời gian = tháng hiện tại | Truy vấn |

Ba chốt nhỏ kèm theo, Lead quyết theo đề xuất — nói nếu muốn khác:

- **FR-USER-02** giao **Giang**; "quản lý quyền" = chỉ đổi vai trò của user trong 5 vai trò cố
  định, không tạo quyền chi tiết.
- **FR-MENU-01**: chỉ Kế toán quản lý và xem, **không** làm màn hình xem cho phụ huynh — đặc tả
  không nêu vai trò nào là người xem, thêm vào là mở rộng phạm vi (CC-02).
- **FR-TUITION-06**: luồng mô phỏng không có đường thất bại; chỉ chặn trường hợp hóa đơn đã
  `paid`. Rate limit login 5 lần/phút đã đủ, **không** làm thêm cơ chế khóa tài khoản.

---

## 7. Ba vấn đề về tiến độ cần chú ý

**1. Hai bạn Frontend chỉ có 2 thẻ khởi động mỗi người.** Tiền và Trang mỗi người có 2 thẻ làm
được ngay (layout theo vai trò, component dùng chung, chuyển login sang React Hook Form), phần
còn lại chờ backend. Nếu hết 2 thẻ đó mà backend chưa xong, cho họ làm trước giao diện tĩnh với
dữ liệu giả rồi nối API sau, đừng để ngồi không.

**2. Ba API phía phụ huynh suýt không ai làm.** FR-ATT-02, FR-MEDIA-02 (Đức) và FR-TUITION-05
(Giang) là thẻ bổ sung trong lần audit này — bản nháp chỉ có thẻ frontend "Chờ..." mà không có
thẻ backend tương ứng.

**3. Hai thẻ kỹ thuật phải làm trước.** `IFileStorageService` chặn FR-INCIDENT-02 và FR-MEDIA-01;
migration `quick_health_statuses` chặn FR-HEALTH-01. Cả hai đứng đầu bảng của Đức, đừng nhảy cóc.

---

## 8. Quy ước vận hành board

- **Giới hạn việc đang làm:** mỗi người tối đa **2 thẻ** ở cột "Đang làm".
- **Thẻ chỉ được kéo sang "Chờ Review"** khi đã đủ checklist Definition of Done ở
  `onboarding-guide.md` §7.1.
- **Thứ tự ưu tiên theo số người đang chờ:**
  1. `[KT] fullName vào /api/auth/me` (Giang) — Tiền và Trang đều chờ
  2. `[KT] IFileStorageService` + `[KT] migration sức khỏe` (Đức) — chặn 3 thẻ khác
  3. FR-ATT-01 (Đức) — Tiền và Tiên chờ
  4. FR-STU-01 (Giang) — Trang và Tiên chờ
  5. FR-TUITION-03 (Giang) — Tiên chờ
- **Migration trên DB dùng chung phải được Lead duyệt trước khi chạy** (guide §6). Hiện chỉ có
  đúng **một** migration được duyệt: thêm `mood` + `temperature`.
- **Thẻ bị chặn quá 3 ngày** → chuyển sang "Chờ quyết định" và báo Lead trong group.
- Mỗi thẻ khi làm xong ghi số PR vào phần bình luận để truy vết lúc báo cáo đồ án.
