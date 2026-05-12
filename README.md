# BÀI TẬP VỀ NHÀ 03: THIẾT KẾ VÀ CÀI ĐẶT CSDL QUẢN LÝ CẦM ĐỒ

**Môn học:** Hệ quản trị CSDL  
**Giảng viên:** Đỗ Duy Cốp  
**Lớp:** 59KMT  

---

## 👤 Thông tin sinh viên
* **Họ và tên:** Mông Chí Thành
* **Mã sinh viên:** k235480106066


---

## 🎯 Mô tả bài toán
Dự án này là hệ thống CSDL nhằm quản lý các hợp đồng vay tiền thế chấp tài sản. Điểm đặc thù của hệ thống bao gồm:
* Cơ chế tính lãi linh hoạt: Lãi đơn (trước Deadline 1) và Lãi kép (từ sau Deadline 1).
* Quản lý trạng thái đa dạng của hợp đồng và danh mục tài sản thế chấp.
* Xử lý thanh lý đồ tự động khi hợp đồng quá hạn.

---

## 🚀 Quá trình thực hiện & Báo cáo kết quả

### Nhiệm vụ 1: Thiết kế Cơ sở dữ liệu

Quá trình phân tích bài toán để đưa ra mô hình thực thể liên kết (ERD) với 4 bảng chính: `Khách Hàng`, `Hợp đồng`, `Tài Sản` và `Log`. CSDL đã được thiết kế đảm bảo chuẩn hóa tối thiểu ở mức 3NF.

![Sơ đồ ERD]([Link_ảnh_chụp_sơ_đồ_ERD_của_bạn])
*Hình 1: Sơ đồ Thực thể Liên kết (ERD) thể hiện rõ thực thể, thuộc tính, khóa chính và khóa ngoại.*

### Nhiệm vụ 2: Cài đặt SQL & Xử lý sự kiện

#### Event 1: Đăng ký hợp đồng mới
Đã viết Stored Procedure để tiếp nhận thông tin khách hàng, lưu danh sách tài sản (kèm giá định giá), số tiền vay và thiết lập mốc Deadline 1, Deadline 2.

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/df519d6d-2ef2-4cc4-bbc2-89695802c6f0" />

*Hình 2: Thực thi Stored Procedure tạo hợp đồng và thêm tài sản thành công.*
<br><br><br>
Dữ liệu thử:
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/06b55f39-10ce-4df1-b553-6eee53d0d9d1" />

#### Event 2: Tính toán công nợ thời gian thực
Xây dựng các Function tính toán nợ:
* `fn_CalcMoneyTransaction`: Tính số tiền phải trả cho một giao dịch đến ngày TargetDate.
* `fn_CalcMoneyContract`: Tính tổng số tiền khách phải trả (Gốc + Lãi đơn + Lãi kép) đến ngày TargetDate.

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/96c51712-5363-4153-8a6c-deccc2f0d334" />

*Hình 3: Kết quả truy vấn hàm tính tổng tiền nợ có bao gồm xử lý lãi kép.*
<br><br><br>
Dữ liệu thử:
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/faf988d9-3586-4561-ac2a-34d6ee4cc944" />

#### Event 3: Xử lý trả nợ và hoàn trả tài sản
Cài đặt Stored Procedure xử lý dòng tiền khi khách thanh toán. Hệ thống tự động kiểm tra cờ `IsSold` để quyết định có thu tiền hay không. Nếu khách trả một phần, hệ thống cập nhật trạng thái "Đang trả góp", ghi nhận LOG, và gợi ý danh sách tài sản có thể trả lại (Điều kiện: Giá trị tài sản còn lại >= Dư nợ).

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/3010c167-0156-4366-a4ce-860f16420736" />
<br><br><br>
*Hình 4: Kiểm tra bảng Log ghi nhận thao tác thanh toán của khách.*
Dữ liệu thử:
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/c3a8f303-b19a-438d-89ad-05409093faf6" />

#### Event 4: Truy vấn danh sách nợ xấu
Truy vấn xuất danh sách khách hàng quá hạn Deadline 1 chưa thanh toán. View/Query hiển thị đầy đủ các cột yêu cầu: Tên KH, SĐT, Số tiền gốc, Số ngày quá hạn, Tổng tiền hiện tại và dự kiến sau 1 tháng.

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/7c3b0a0b-733a-418e-822e-7b1ee57759c6" />
<br><br><br>

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/c5e346e1-e906-4bea-a8d1-63b0cc5b6ce6" />

*Hình 5: Danh sách khách hàng nợ xấu (nợ khó đòi).*

#### Event 5: Quản lý thanh lý tài sản tự động
Cài đặt hệ thống Trigger để chuyển đổi trạng thái tự động:
* Đổi hợp đồng sang "Quá hạn (nợ xấu)" khi vượt Deadline 1.
* Đổi tài sản sang "Sẵn sàng thanh lý" khi vượt Deadline 2.
* Chuyển trạng thái tài sản thành "Đã bán thanh lý" khi hợp đồng "Đã thanh lý".

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/432dc2b1-91d5-4242-ac38-d08dde0a496f" />
<br><br><br>
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/fe185e17-1e4e-40d4-a4e3-82e60f7c1374" />

*Hình 6: Test hệ thống Trigger tự động đổi trạng thái khi đổi ngày.*

---

