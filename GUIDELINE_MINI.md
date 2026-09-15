# HƯỚNG DẪN GÁN NHÃN THEO DÕI ĐỐI TƯỢNG (MINI ANNOTATION GUIDELINES)

Tài liệu này quy chuẩn hóa quy trình gán nhãn (bounding box và ID) cho xe cộ trong chuỗi video nhằm tối đa hóa độ chính xác và tính nhất quán với mô hình AI.

---

## 1. Nguyên tắc vẽ Bounding Box (BBox Tightness)
- **Bám sát biên thực tế:** Hộp nhận diện phải ôm sát viền ngoài cùng của lốp xe, gương chiếu hậu và các chi tiết nhô ra của vỏ xe.
- **Không vẽ dư thừa:** Tránh kéo rộng hộp ra khoảng không hậu cảnh hoặc bóng đổ của xe dưới lòng đường (gây tăng lỗi FP hoặc sai số LocA).
- **Nhất quán kích thước:** Đảm bảo bbox phủ kín vật thể đồng đều giữa các khung hình liên tiếp khi xe không đổi hướng đột ngột.

## 2. Quy chuẩn gán nhãn Xe Bus lớn và Xe Toa dài
- **Bao phủ toàn diện:** Bắt buộc vẽ 1 bbox lớn duy nhất bao trùm từ đầu xe đến hết đuôi xe bus. Không được chia cắt xe bus thành nhiều hộp nhỏ hoặc bỏ quên phần toa phía sau.
- **Khi xe bus bị che khuất một phần:** Tiếp tục duy trì kích thước hộp ước lượng bao trọn xe cho đến khi xe ra khỏi vùng chồng lấn.

## 3. Quản lý che khuất (Occlusion) và Điểm mù
- **Ngưỡng gán nhãn:** Chỉ thực hiện gán nhãn khi vật thể xuất hiện tối thiểu **30% diện tích** có thể nhận dạng được bằng mắt thường.
- **Mất dấu tạm thời (Dưới 5 frames):** Nếu xe bị che khuất bởi cột đèn, biển báo hoặc xe bus lớn khác rồi lộ diện ngay sau đó, bắt buộc phải **giữ nguyên ID cũ**.
- **Mất dấu kéo dài (Trên 5 frames):** Nếu xe đi khuất hẳn vào điểm mù quá lâu và không thể xác định chắc chắn diện mạo cũ bằng mắt thường, hãy gán một **ID mới hoàn toàn** khi xe xuất hiện trở lại.

## 4. Quản lý ID Nhất quán (Identity Management)
- Mỗi xe là một thực thể duy nhất xuyên suốt video. Tuyệt đối không tự ý đổi ID khi xe di chuyển bình thường không bị che khuất.
- Kiểm tra chéo định kỳ các phân cảnh giao cắt (overlap) giữa 2 xe để đảm bảo ID không bị đổi chéo cho nhau.
