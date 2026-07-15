# Hệ thống Quản lý Bệnh nhân (Hospital Management) - Ứng dụng hp002

Tài liệu tham khảo cấu trúc ứng dụng hp002 - Phân hệ quản lý Bệnh nhân trên nền tảng OAP.

## Cấu trúc Page Metadata (OAP)
Phân cấp: `layout` → `pane` → `region` → `field`

### `layout`
Quy định cách chia bố cục tổng thể của trang.
- `type: "grid"` — lưới đơn giản, không chia pane/sidebar/tab.
- `type: "splitter"` — chia trang thành nhiều pane có thể kéo giãn/thu gọn. Có `orientation: "horizontal"` (trái-phải) hoặc `"vertical"` (trên-dưới).

### `pane`
Chỉ tồn tại khi `layout.type: "splitter"`. Mỗi pane là một khung con của trang.
- `id`: định danh pane
- `size`: độ rộng/cao ("260px" cố định hoặc "1fr" chiếm phần còn lại)
- `collapsible`, `resizable`: cho phép thu gọn / kéo giãn
- `regionIds`: danh sách region hiển thị bên trong pane đó (có thể nhiều region xếp chồng dọc)

### `region`
Khung chứa nội dung, nằm bên trong 1 pane (hoặc trực tiếp trong `regions[]` nếu dùng `layout.type: "grid"`).
- `id`: phải khớp với regionIds khai ở pane
- `type`: `"card"` (khung có viền/tiêu đề) hoặc `"plain"` (không viền)
- `fields[]`: danh sách field bên trong region

### `field`
Đơn vị nhỏ nhất — 1 ô input, nút, bảng, biểu đồ... nằm trong `region.fields[]`.
- `name`: tên field (dùng làm key dữ liệu)
- `type`: loại field — input (text, number, date...), selection (select, combobox, radio...), layout (datagrid, dx-toolbar, container...), advanced (chart, widget, dashboard, kanban...)
- `label`: nhãn hiển thị
- `layout.col` / `colSpan`: vị trí trên hệ lưới 12 cột bên trong region

### Ví dụ minh họa (Trang H001)
```json
{
  "layout": {
    "type": "grid",
    "regions": [
      { "id": "list_region", "type": "plain", "fields": [ /* field bảng bệnh nhân */ ] }
    ]
  }
}
```

---

## Database Schema
Schema: `APPS` (Quyền: SELECT, INSERT, UPDATE, DELETE)

### DEMO_PATIENTS
Quản lý thông tin bệnh nhân. PK: `PATIENT_ID`

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

---

## Kiến trúc Màn hình (Pages)

### 1. H001 - Danh sách Bệnh nhân
- **Page Type:** `form`
- **Layout:** `grid` (Vùng duy nhất `list_region`)
- **Field chính:** `tbl_patients` (type: `datagrid`) hiển thị danh sách dạng bảng.
- **Tính năng:**
  - Lưới read-only (tắt inline-editing).
  - Sử dụng `rowActions` để gọi chức năng thêm mới và xem chi tiết, tránh được việc runtime tự động casting các cột khóa chính (`PATIENT_ID`) sang định dạng NUMBER gây ra lỗi khi có chứa các ký tự dạng text (ví dụ: `P001`).
  - Lọc dữ liệu qua biến bind (`:p_search_name`, `:p_search_phone`, `:p_gender`).

### 2. H001_FORM - Chi tiết Bệnh nhân
- **Page Type:** `form`
- **Hình thức hiển thị:** Mở dạng Modal qua `rowActions` từ trang H001.
- **Tính năng:**
  - Nhập liệu và chỉnh sửa thông tin cho 1 bệnh nhân.
  - Sử dụng `saveQuery` dưới dạng một khối **PL/SQL ẩn danh** (Anonymous Block) để khai báo cụ thể biến kiểu `VARCHAR2` cho `PATIENT_ID` thay cho kiểu ngầm định của runtime.
  - Bao gồm các trường text, date (DD/MM/YYYY) và select (LOV tĩnh M/F).
