# Hướng dẫn Cấu hình & Khắc phục lỗi kiểu dữ liệu trên OAP (Wiki)

Tài liệu này hướng dẫn chi tiết cách thiết lập các trang danh sách (Grid) và trang chi tiết (Form) trên hệ thống OAP. Đặc biệt, hướng dẫn này tập trung vào cách giải quyết lỗi phổ biến liên quan đến tự động ép kiểu dữ liệu của hệ thống, giúp bạn dễ dàng áp dụng pattern này cho các module khác.

---

## 🛑 Vấn đề thường gặp: Lỗi ép kiểu `_ID` (ORA-01722)

**Nguyên nhân:**
Mặc định, OAP runtime tự động nhận diện tất cả các cột có đuôi `_ID` (như `PATIENT_ID`, `PRODUCT_ID`) là kiểu số (`NUMBER`). Nếu khóa chính của bạn chứa ký tự chữ (VD: `P001`), khi lưu dữ liệu từ Grid inline-editing, OAP sẽ tự ép kiểu sang số và gây ra lỗi `ORA-01722: invalid number`.

**Giải pháp (Pattern chuẩn):**
Không sử dụng tính năng lưu trực tiếp (inline edit) trên bảng cho những bảng có khóa chính là Text. Thay vào đó, chúng ta chia làm 2 màn hình:
1. **Trang Grid (Danh sách):** Chỉ để xem và có các nút hành động (rowActions) mở form.
2. **Trang Form (Chi tiết):** Mở dưới dạng Modal, dùng khối PL/SQL tùy chỉnh (`saveQuery`) để ép lại kiểu dữ liệu thành chữ.

---

## 🛠 Hướng dẫn thiết lập Step-by-Step

### Bước 1: Tạo trang Form (VD: `H001_FORM`)
Trang này dùng để thêm mới hoặc sửa dữ liệu. Cấu hình quan trọng nhất nằm ở `dataSource`.

1. Khởi tạo file `metadata.json` cho trang Form.
2. Tại phần `dataSource`, bạn **bắt buộc** phải viết một khối PL/SQL ẩn danh trong thuộc tính `saveQuery` để ép kiểu thủ công.

**Ví dụ cấu hình dataSource cho Form:**
```json
"dataSource": {
  "type": "query",
  "tableName": "DEMO_PATIENTS",
  "keyField": "PATIENT_ID",
  "query": "SELECT PATIENT_ID, FIRST_NAME, LAST_NAME FROM DEMO_PATIENTS WHERE PATIENT_ID = :id",
  "saveQuery": "DECLARE v_patient_id VARCHAR2(20) := :PATIENT_ID; v_first_name VARCHAR2(20) := :FIRST_NAME; v_last_name VARCHAR2(20) := :LAST_NAME; BEGIN IF :id IS NULL THEN INSERT INTO DEMO_PATIENTS (PATIENT_ID, FIRST_NAME, LAST_NAME) VALUES (v_patient_id, v_first_name, v_last_name); ELSE UPDATE DEMO_PATIENTS SET FIRST_NAME = v_first_name, LAST_NAME = v_last_name WHERE PATIENT_ID = v_patient_id; END IF; COMMIT; END;"
}
```
*Lưu ý: Phải khai báo `v_patient_id VARCHAR2(20) := :PATIENT_ID;` để hệ thống hiểu đây là Text, không phải Number.*

### Bước 2: Thiết lập trang Grid (VD: `H001`)
Trang này hiển thị danh sách, tìm kiếm và phân trang.

1. Đặt `type` của component lưới là `"table"` (hoặc `"datagrid"`).
2. Xóa bỏ `saveMode: "auto"` trong `dataSource` nếu có.
3. Thiết lập thuộc tính `allowEditing: false` cho cột Khóa chính (VD: `PATIENT_ID`) để khóa việc chỉnh sửa trực tiếp trên lưới.
4. Thêm thuộc tính `rowActions` để gọi sang trang Form vừa tạo ở Bước 1.

**Ví dụ cấu hình rowActions trong Grid:**
```json
"rowActions": {
  "create": {
    "pageCode": "H001_FORM",
    "size": "xl",
    "title": "Thêm dữ liệu mới"
  },
  "detail": {
    "pageCode": "H001_FORM",
    "size": "xl",
    "title": "Chi tiết bản ghi"
  }
}
```

### Bước 3: Triển khai (Deploy)
Sử dụng các công cụ OAP Devkit (MCP) để đẩy cấu hình lên server:
1. Triển khai trang Form trước: Dùng `update_page` cho `H001_FORM`.
2. Triển khai trang Grid sau: Dùng `update_page` cho `H001`.

---

## 📝 Tóm tắt Checklist khi tạo một module mới
Mỗi khi bạn muốn tạo một module mới tương tự:
- [ ] Xác định khóa chính (PK) của bảng là Số hay Chữ.
- [ ] Nếu là Chữ và có đuôi `_ID` -> Bắt buộc dùng pattern `rowActions` + `saveQuery` (PL/SQL).
- [ ] Tạo metadata cho trang Form (`_FORM`). Viết câu lệnh `saveQuery` khai báo cụ thể kiểu `VARCHAR2`.
- [ ] Tạo metadata cho trang Grid. Chèn cấu hình `rowActions` trỏ về trang Form.
- [ ] Khóa thuộc tính `allowEditing: false` ở cột PK trong Grid.
- [ ] Đẩy (Deploy) Form lên server.
- [ ] Đẩy (Deploy) Grid lên server.

Chúc bạn thành công với việc mở rộng ứng dụng trên nền tảng OAP!
