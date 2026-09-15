# BÁO CÁO PHÂN TÍCH THEO DÕI ĐỐI TƯỢNG (VIDEO TRACKING REPORT)

## 1. Bảng tổng hợp kết quả (Metrics Summary Table)

| Cấu hình đối chiếu | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **YOLO26n + ReID vs. Nhãn của bạn** | 0.682 | 0.622 | 0.753 | 0.834 | 0.866 | 0.733 | 0.810 | 88 | 78 | 2 |

---

## 2. Trả lời các câu hỏi phân tích hệ thống

### Câu 1: So sánh MOTA và IDF1 & Cách thức phạt lỗi của MOTA
- **Kết quả thực tế:** Điểm **IDF1 (0.866)** cao hơn đáng kể so với **MOTA (0.733)**.
- **Giải thích ý nghĩa:** Điểm IDF1 cao cho thấy tính nhất quán định danh (identity preservation) xuyên suốt của cả bạn và mô hình ReID khá tốt, chỉ xảy ra 2 lỗi nhảy ID (IDSW = 2) trong toàn bộ quá trình.
- **Tại sao MOTA không phạt nặng lỗi ID Switch?**
  - MOTA tính toán dựa trên tổng số lỗi cục bộ ở từng khung hình độc lập:
    $$\text{MOTA} = 1 - \frac{\text{FP} + \text{FN} + \text{IDSW}}{\text{GT}}$$
  - Trong công thức này, một lỗi ID Switch chỉ bị phạt trọng số tương đương với 1 điểm lỗi phát hiện sai (FP) hoặc bỏ sót (FN). Nếu một xe di chuyển qua 100 frames và bị nhảy ID đúng một lần ở giữa chặng, MOTA chỉ phạt đúng 1 điểm lỗi (`IDSW = 1`), trong khi thực chất từ nửa sau của clip, mã định danh của vật thể đã hoàn toàn bị sai lệch. Ngược lại, **IDF1** đánh giá dựa trên tỷ lệ khớp chuỗi ID dài nhất (F-score trên toàn chuỗi), giúp phản ánh độ ổn định định danh nhạy bén và chính xác hơn nhiều đối với các hệ thống Tracking.

### Câu 2: Sự khác biệt giữa ByteTrack control và BoT-SORT + ReID treatment
- **Đánh giá định tính:**
  - **ByteTrack (Control):** Chỉ sử dụng thông tin chuyển động (Kalman Filter) và độ giao nhau hình học IoU để liên kết các hộp nhận diện. Khi các xe di chuyển đan xen hoặc bị che khuất mạnh (occlusion), ByteTrack dễ bị mất dấu hoặc gán nhãn ID mới cho xe cũ khi nó xuất hiện trở lại.
  - **BoT-SORT + ReID (Treatment):** Kết hợp thêm vector đặc trưng ngoại quan (Appearance Embeddings) từ mô hình ReID. Khi xảy ra chồng lấp giữa hai xe hoặc khi xe đi khuất sau xe bus và xuất hiện lại, mô hình ReID sẽ so sánh độ tương đồng đặc trưng màu sắc/kiểu dáng để duy trì chính xác ID cũ. Điều này giúp tối ưu hóa điểm **AssA** và **IDF1**, đồng thời hạn chế tối thiểu lỗi **IDSW**.

### Câu 3: Phân tích chỉ số DetA, FP, FN và nguồn gốc lỗi chính
- **Phân tích số liệu:**
  - **DetA (Detection Accuracy) = 0.622** thấp hơn rõ rệt so với **AssA (Association Accuracy) = 0.753**.
  - Hệ thống ghi nhận **88 lỗi FP** (Hộp nhận diện ảo) và **78 lỗi FN** (Bỏ sót xe thực tế).
- **Kết luận nguồn gốc lỗi:** 
  - Lỗi lớn nhất hiện tại thuộc về **Detector (YOLO26n)** chứ không phải do thuật toán liên kết Association. Mô hình YOLO bản Nano có dung lượng nhỏ nên dễ bỏ sót các phương tiện ở quá xa hoặc bị che khuất một phần (gây ra FN), hoặc nhận diện sai các yếu tố hậu cảnh tĩnh như biển báo, dải phân cách thành phương tiện giao thông (gây ra FP).

### Câu 4: Đối chiếu trực quan - Điểm đúng/sai giữa Nhãn gán tay và ReID
- **Chỗ Bạn Đúng và ReID Sai (Frame 104 - 106):**
  - *Mô tả hiện tượng:* Ở khu vực phía xa bên dưới cột biển báo lớn, xuất hiện chiếc xe con màu trắng đỗ ven đường. Mô hình ReID bị nhiễu do vật cản hậu cảnh, sinh ra các hộp nhận diện đè chéo, phân mảnh liên tục thành nhiều ID (như T24, T26). Trong khi đó, nhãn gán tay của bạn (B#) giữ được sự sạch sẽ, bao phủ chính xác và giữ nguyên ID đồng nhất cho phương tiện này.
- **Chỗ ReID Đúng và Bạn cần xem lại (Frame 111):**
  - *Mô tả hiện tượng:* Đối với chiếc xe bus lớn hai toa màu vàng-xanh di chuyển ở trung tâm khung hình. Mô hình ReID treatment (màu đỏ T#) bao quát rất chuẩn xác rìa khung vỏ xe bus. Nhãn gán bằng tay của bạn (màu xanh B#) ở một số phân đoạn bị lệch biên tương đối rộng hoặc hẹp hơn thực tế, làm giảm nhẹ chỉ số Location Accuracy (LocA = 0.834).

### Câu 5: Đề xuất cải tiến tài liệu hướng dẫn gán nhãn (`GUIDELINE_MINI.md`)
Nếu phải thực hiện gán nhãn thêm 10 video nữa, các quy tắc sau cần được bổ sung để nâng cao tính nhất quán:
1. **Quy định bao phủ xe Bus lớn:** Thống nhất rõ ràng việc vẽ bounding box phải bao trọn toàn bộ thân xe bus dài/xe bus hai toa từ đầu đến đuôi ở mọi góc máy, tránh gán thiếu phần đuôi xe.
2. **Ngưỡng gán nhãn vật thể bị che khuất (Occlusion Gate):** Quy định cụ thể chỉ gán nhãn khi vật thể lộ diện từ 30% diện tích trở lên. Nếu vật thể bị xe khác che khuất hoàn toàn quá 5 frames liên tiếp, bắt buộc phải ngắt track cũ và gán ID mới khi xuất hiện lại nếu không thể nhận dạng rõ ràng bằng mắt thường.
3. **Chuẩn hóa điểm biên (BBox Tightness):** Yêu cầu bounding box phải bám sát vào phần mép ngoài cùng của lốp xe và gương chiếu hậu để đảm bảo tính đồng nhất cao nhất với các nhãn sinh ra bởi mô hình AI.
