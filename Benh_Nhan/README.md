# OAP_HOSPITAL - Tài liệu Hướng dẫn & Khắc phục Sự cố (Wiki)

Tài liệu này cung cấp hướng dẫn sử dụng, tóm tắt phương pháp cấu hình (Page Builder) và cách khắc phục các lỗi (bug) thường gặp trong quá trình phát triển ứng dụng quản lý bệnh nhân trên nền tảng OAP.

---

## 1. Hướng dẫn Sử dụng (End-User)
Hệ thống **Bệnh nhân** được chia làm 2 trang chính nằm trong cùng một thư mục Menu:

* **Danh sách Bệnh nhân (H001):** 
  * Cung cấp cái nhìn tổng quan với các Thẻ chỉ số thống kê (Tổng số bệnh nhân, Có BHYT, Nam, Nữ...).
  * Bộ lọc bên trái giúp tìm kiếm nhanh theo Tên, Số điện thoại và Giới tính. 
  * Bảng dữ liệu (Grid) hiển thị thông tin bệnh nhân.
* **Biểu mẫu Bệnh nhân (H001_FORM):** 
  * Là một trang biểu mẫu độc lập (Standalone Form) có tích hợp thanh công cụ (Toolbar) trên cùng.
  * Hỗ trợ Thêm mới hoặc Chỉnh sửa thông tin chi tiết của từng bệnh nhân. Sau khi Lưu, hệ thống sẽ tự động đồng bộ sang trang Danh sách.

---

## 2. Hướng dẫn Chỉnh sửa & Cấu hình (Dành cho Developer)

Khi làm việc với các file `metadata.json` của hệ thống OAP, cần lưu ý các quy tắc sau:

### 2.1. Cấu trúc Layout (Splitter)
- Nên sử dụng layout `type: "splitter"` để dễ dàng chia màn hình thành các phần (Panes) linh hoạt (Ví dụ: Trái cho Bộ lọc, Phải cho Dữ liệu).
- **Quy tắc quan trọng (Tránh lỗi Virtual DOM):** Khi có nhiều trang liên kết chặt chẽ với nhau (Ví dụ Form và List), tuyệt đối **không đặt trùng tên ID của Pane**. 
  - *Sai:* Đặt id là `"center"` cho cả trang List và Form.
  - *Đúng:* Đặt id là `"list_center"` cho H001 và `"form_center"` cho H001_FORM. Việc này ép trình duyệt vẽ lại giao diện hoàn chỉnh khi chuyển trang.

### 2.2. Vùng dữ liệu Datagrid
- Khi nâng cấp từ `type: "table"` cũ sang `type: "datagrid"` hiện đại, cần bọc (wrap) region chứa grid đó trong `type: "card"` (thay vì `plain`). Datagrid mặc định sẽ cố gắng giãn ra chiếm 100% không gian; nếu không có `card` bọc lại, nó sẽ che khuất (overlap) các thành phần bên trên (như thẻ KPI).

### 2.3. Cấu hình Widgets (Thẻ KPI Thống kê)
- Mỗi widget cần một nguồn dữ liệu (`dataSource`) riêng.
- Nếu muốn widget thay đổi khi người dùng tìm kiếm, phải khai báo đầy đủ `bindVariables` lấy từ các bộ lọc (`f_search_name`, v.v.) và ghép vào câu query của widget.
- Không dùng dữ liệu cứng (Ví dụ: `SELECT 11 FROM DUAL`), hãy dùng các phép thống kê thực tế như `COUNT`, `SUM` để trang có ý nghĩa.

---

## 3. Nhật ký Khắc phục Sự cố (Troubleshooting & Bug Fixes)

Dưới đây là các lỗi kinh điển đã gặp trong dự án và cách xử lý để rút kinh nghiệm:

### 🔴 Lỗi 1: Chạy lệnh Database báo `ORA-00942: table or view does not exist`
* **Mô tả:** Mở phần mềm DBeaver/SQL Developer để chạy lệnh `ALTER TABLE` mở rộng cột, nhưng hệ thống báo không tìm thấy bảng.
* **Nguyên nhân:** Lỗi nhầm lẫn môi trường. Bạn đang kết nối phần mềm SQL vào database trên máy cá nhân (`localhost`), trong khi mã nguồn OAP và bảng dữ liệu thực tế đang chạy trên Server từ xa (`oap-meta.ovigroup.vn` - host: `oapdb`).
* **Cách fix:** Gửi câu lệnh SQL cho Quản trị viên (Admin) quản lý Database của máy chủ từ xa để họ chạy giúp, hoặc xin cấp quyền truy cập VPN vào Server.

### 🔴 Lỗi 2: Cột/Vùng "Chỉ số thống kê" biến mất khi đổi sang Datagrid
* **Mô tả:** Sau khi cấu hình Grid nâng cao, vùng hiển thị số liệu phía trên bị mất tăm.
* **Nguyên nhân:** Grid hiện đại render theo cơ chế Flex/Absolute height. Khi nằm chung một pane (`plain`) với các thẻ thống kê, Grid sẽ kéo giãn và chèn đè lên chúng.
* **Cách fix:** Mở `metadata.json`, tìm ID của Region chứa Grid (vd: `list_region`), đổi `type` từ `"plain"` sang `"card"`.

### 🔴 Lỗi 3: Vùng thống kê biến mất khi bấm chuyển từ Biểu mẫu về lại Danh sách
* **Mô tả:** Chuyển trang qua lại thông qua menu làm rớt một số thành phần giao diện (đặc biệt là KPI Region).
* **Nguyên nhân:** Lỗi bộ nhớ ảo (Virtual DOM conflict). Trình duyệt thấy 2 trang đều có layout Pane tên là `"center"`, nó cố gắng tái sử dụng (reuse) Pane này để tải nhanh hơn nhưng lại ráp thiếu các Region con.
* **Cách fix:** Mở `metadata.json` của cả 2 trang, đổi tên các Pane sao cho không bị trùng lặp (ví dụ `list_center` và `form_center`). Trình duyệt sẽ buộc phải vẽ lại bố cục mới 100%.

### 🔴 Lỗi 4: Thẻ thống kê hiển thị số không thực tế (Ví dụ: luôn hiện 11)
* **Nguyên nhân:** Mượn code từ template mẫu nhưng quên sửa câu SQL, dẫn đến câu SQL là `SELECT 11 AS VALUE FROM DUAL`.
* **Cách fix:** Sửa câu SQL thành `SELECT COUNT(INSURANCE_NUMBER) ... FROM DEMO_PATIENTS`, cập nhật lại tiêu đề ở file `translations.json` thành tên có ý nghĩa (vd: "Có BHYT").
