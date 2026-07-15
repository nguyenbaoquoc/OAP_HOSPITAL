# OAP Hospital Management (Ứng dụng hp002)

## 📌 Tổng quan dự án (Overview)
**OAP Hospital Management (hp002)** là một dự án thực hành nhằm xây dựng hệ thống quản lý thông tin bệnh nhân trên nền tảng OAP (Oracle Application Platform). 
Dự án được tạo ra với mục tiêu chính là **luyện tập, nghiên cứu và sử dụng thành thạo các tính năng của OAP Web Platform**, từ việc cấu hình metadata, xử lý database đến việc giải quyết các bài toán thực tế trong quá trình phát triển ứng dụng Low-code/No-code.

## 🎯 Mục tiêu (Goals)
- Làm quen và thành thạo kiến trúc Page Metadata của OAP.
- Nắm vững cách tương tác giữa giao diện (Frontend) và cơ sở dữ liệu (Backend) thông qua PL/SQL và cấu trúc dữ liệu.
- Hiểu rõ cơ chế xử lý kiểu dữ liệu, các ràng buộc và cách giải quyết lỗi phát sinh (ví dụ: lỗi ép kiểu DML ngầm định của runtime với các cột `_ID`).
- Áp dụng các mô hình thiết kế chuẩn của hệ thống (như mô hình `rowActions` gọi Modal Form thay vì Inline-Editing cho các cấu trúc dữ liệu khóa chính là Text).

## 💡 Tiện ích & Lợi ích (Benefits)
- **Đối với người học:** Cung cấp một case-study thực tế để đối chiếu lý thuyết OAP vào thực hành, bao gồm cả việc debug, thay đổi luồng xử lý và refactor kiến trúc trang.
- **Đối với ứng dụng:** Cung cấp một giao diện quản lý hồ sơ bệnh nhân trực quan, cho phép thêm mới, chỉnh sửa, xóa và tìm kiếm bệnh nhân nhanh chóng thông qua hệ thống Grid và Form.

## 🗂 Bố cục ứng dụng (Layout & Pages)
Dự án bao gồm 2 trang (pages) chính được liên kết chặt chẽ với nhau:

### 1. H001 - Danh sách Bệnh nhân
- **Page Type:** `form`
- **Layout:** `grid` (Vùng duy nhất `list_region`)
- **Nội dung:** Hiển thị danh sách bệnh nhân dưới dạng bảng (`datagrid`).
- **Tính năng nổi bật:** 
  - Khóa tính năng sửa trực tiếp (inline edit) trên cột khóa chính `PATIENT_ID` để tránh lỗi ép kiểu của runtime OAP.
  - Tích hợp `rowActions` để mở form thêm mới hoặc xem chi tiết.
  - Lọc dữ liệu thông minh qua biến bind (`:p_search_name`, `:p_search_phone`, `:p_gender`).

### 2. H001_FORM - Chi tiết Bệnh nhân
- **Page Type:** `form`
- **Hình thức hiển thị:** Mở dạng Modal qua `rowActions` từ trang H001.
- **Tính năng nổi bật:**
  - Nhập liệu và chỉnh sửa thông tin chi tiết cho 1 bệnh nhân.
  - Cấu trúc lưu trữ linh hoạt bằng `saveQuery` dưới dạng một khối **PL/SQL ẩn danh** (Anonymous Block). Điều này cho phép khai báo cụ thể biến kiểu `VARCHAR2` cho `PATIENT_ID`, vượt qua rào cản tự động nhận diện kiểu số của hệ thống OAP.

---

## 🛠 Kiến trúc Page Metadata (OAP)
Trong OAP, giao diện được cấu hình qua file JSON metadata với phân cấp chuẩn như sau: `layout` → `pane` → `region` → `field`

### `layout`
Quy định cách chia bố cục tổng thể của trang.
- `type: "grid"` — lưới đơn giản, không chia pane/sidebar/tab.
- `type: "splitter"` — chia trang thành nhiều pane có thể kéo giãn/thu gọn. Có `orientation: "horizontal"` (trái-phải) hoặc `"vertical"` (trên-dưới).

### `pane`
Chỉ tồn tại khi `layout.type: "splitter"`. Mỗi pane là một khung con của trang.
- `id`: định danh pane
- `size`: độ rộng/cao ("260px" cố định hoặc "1fr" chiếm phần còn lại)
- `collapsible`, `resizable`: cho phép thu gọn / kéo giãn
- `regionIds`: danh sách region hiển thị bên trong pane đó (có thể xếp chồng nhiều region)

### `region`
Khung chứa nội dung, nằm bên trong 1 pane (hoặc trực tiếp trong `regions[]` nếu dùng `layout.type: "grid"`).
- `id`: phải khớp với regionIds khai ở pane
- `type`: `"card"` (khung có viền/tiêu đề) hoặc `"plain"` (không viền)
- `fields[]`: danh sách field bên trong region

### `field`
Đơn vị nhỏ nhất — 1 ô input, nút, bảng, biểu đồ... nằm trong `region.fields[]`.
- `name`: tên field (dùng làm key dữ liệu)
- `type`: loại field (text, number, date, select, datagrid...)
- `label`: nhãn hiển thị
- `layout.col` / `colSpan`: vị trí trên hệ lưới 12 cột bên trong region

---

## 🗄 Bộ dữ liệu (Database Schema)
Dữ liệu của hệ thống được lưu trữ tại schema `APPS` (Quyền truy cập cơ bản: SELECT, INSERT, UPDATE, DELETE).

### Bảng: DEMO_PATIENTS
Quản lý thông tin hồ sơ bệnh nhân. Khóa chính (PK): `PATIENT_ID`

| Cột | Kiểu | Bắt buộc | Default | Mô tả |
|---|---|---|---|---|
| `PATIENT_ID` | `VARCHAR2(20)` | có | — | Khóa chính, mã định danh bệnh nhân (VD: P001, P002) |
| `FIRST_NAME` | `VARCHAR2(20)` | không | — | Họ và tên đệm của bệnh nhân |
| `LAST_NAME` | `VARCHAR2(20)` | không | — | Tên chính của bệnh nhân |
| `GENDER` | `VARCHAR2(20)` | không | — | Giới tính (M: Nam, F: Nữ) |
| `DATE_OF_BIRTH` | `DATE` | không | — | Ngày sinh |
| `CONTACT_NUMBER` | `VARCHAR2(50)` | không | — | Số điện thoại liên hệ |
| `ADDRESS` | `VARCHAR2(20)` | không | — | Địa chỉ nơi ở |
| `REGISTRATION_DATE` | `DATE` | không | — | Ngày đăng ký hồ sơ bệnh án |
| `INSURANCE_PROVIDER`| `VARCHAR2(20)` | không | — | Đơn vị cung cấp bảo hiểm y tế |
| `INSURANCE_NUMBER` | `VARCHAR2(20)` | không | — | Mã số bảo hiểm y tế |
| `EMAIL` | `VARCHAR2(20)` | không | — | Địa chỉ email |
