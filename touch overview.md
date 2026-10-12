MỤC LỤC
CHƯƠNG 1.	PHÁT BIỂU BÀI TOÁN	3
CHƯƠNG 2.	DỮ LIỆU CẦN CÓ	4
CHƯƠNG 3.	3. XỬ LÝ DỮ LIỆU	5
CHƯƠNG 4.	4. CÁCH TRIỂN KHAI HỆ THỐNG	6
CHƯƠNG 5.	5. CÁCH ĐÁNH GIÁ	7
CHƯƠNG 6.	6. ĐIỂM MỚI SO VỚI CHẤM CÔNG THƯỜNG	8

 CHƯƠNG 1.	PHÁT BIỂU BÀI TOÁN
Nhóm xây hệ thống chấm công bằng khuôn mặt: nhân viên đứng trước camera, hệ thống tự nhận ra là ai và ghi nhận giờ vào/ra, thay cho vân tay hay thẻ từ.
đây không phải bài toán huấn luyện model nhận diện từ đầu. Nhóm không train mạng nơ-ron để học khuôn mặt. Thay vào đó dùng model đã huấn luyện sẵn (InsightFace / ArcFace) để biến mỗi khuôn mặt thành một vector đặc trưng, rồi giải bài toán so khớp vector. Đây gọi là cách tiếp cận embedding + truy hồi (retrieval), khác với phân loại (classification).
-Phân biệt này quan trọng vì nó quyết định "dữ liệu để làm gì":
-Không cần hàng nghìn ảnh mỗi người để train.
-Chỉ cần vài ảnh mỗi nhân viên để đăng ký (enroll).
Thêm người mới chỉ cần thêm vector, không phải train lại. Đây là ưu điểm lớn nên nhấn mạnh.
CHƯƠNG 2.	DỮ LIỆU CẦN CÓ
Chia làm 3 nhóm, mỗi nhóm phục vụ một mục đích khác nhau:
-Dữ liệu đăng ký (enrollment) - ảnh của chính nhân viên trong hệ thống.
-	Mỗi người vài ảnh (khuyến nghị 3-10 ảnh), chụp ở các góc, điều kiện ánh sáng khác nhau.
-	Đây là "danh bạ khuôn mặt" của hệ thống. Không dùng để train, chỉ để làm chuẩn so khớp.
-Dữ liệu kiểm thử có nhãn (test set) - để đo độ chính xác và chọn ngưỡng.

-	Gồm ảnh của người đã đăng ký (để kiểm tra có nhận đúng không) và ảnh của người lạ chưa đăng ký (để kiểm tra có từ chối đúng không).
-	Phần "người lạ" này rất quan trọng, vì nó mới đo được tỷ lệ nhận nhầm - đúng trọng tâm đề tài.
- Dữ liệu vận hành (log) - sinh ra khi hệ thống chạy.

-	Mỗi lần nhận diện ghi lại: thời điểm, điểm tương đồng, kết quả chấp nhận/từ chối.
-	Đây là nguyên liệu cho phần giám sát - điểm mới của nhóm.
-	Về nguồn dữ liệu để demo/thử nghiệm: có thể dùng dữ liệu tự thu thập trong nhóm, hoặc các bộ dữ liệu khuôn mặt công khai (ví dụ LFW) để kiểm thử. Nên nói rõ với giảng viên về vấn đề đạo đức và quyền riêng tư: khuôn mặt là dữ liệu sinh trắc nhạy cảm, cần có sự đồng ý khi thu thập, đây là điểm cộng khi trình bày.

CHƯƠNG 3.	3. XỬ LÝ DỮ LIỆU
-Luồng xử lý một ảnh khuôn mặt gồm các bước chuẩn:
-Phát hiện khuôn mặt (detection): tìm vị trí mặt trong ảnh, bỏ phần nền.
-Căn chỉnh (alignment): xoay/co giãn mặt về tư thế chuẩn dựa trên các điểm mốc (mắt, mũi, miệng). Bước này giúp vector ổn định hơn.
-Chuẩn hóa (normalization): đưa về cùng kích thước, chuẩn hóa độ sáng/màu.
-Trích đặc trưng (embedding): đưa khuôn mặt đã căn chỉnh qua model, ra một vector số (thường 512 chiều).
-Chuẩn hóa vector: đưa vector về cùng độ dài để so sánh bằng độ tương đồng cosine.
-InsightFace lo gần hết các bước 1-4. Phần nhóm cần quan tâm nhiều là chất lượng ảnh đầu vào: ảnh mờ, thiếu sáng, mặt bị che → vector kém → nhận sai nhiều. Nên có bước lọc chất lượng (bỏ ảnh không phát hiện được mặt, mặt quá nhỏ, quá nghiêng).

CHƯƠNG 4.	CÁCH TRIỂN KHAI HỆ THỐNG
Hệ thống gồm 3 luồng, nên trình bày tách bạch:

-Luồng 1 - Đăng ký: chạy một lần cho mỗi nhân viên. Ảnh → trích vector → lưu vào vector database kèm mã nhân viên. Vector DB (như FAISS, Milvus, Qdrant) là CSDL chuyên tìm vector gần nhau nhanh, kể cả khi có rất nhiều người.
-Luồng 2 - Nhận diện / chấm công: chạy mỗi khi có người đến. Ảnh từ camera → trích vector → vector DB tìm người giống nhất → so điểm tương đồng với một ngưỡng:
-Trên ngưỡng → chấm công cho người đó.
-Dưới ngưỡng → coi là người lạ, từ chối.
-Mỗi lượt đều được ghi log.
-Luồng 3 - Giám sát (điểm mới): chạy định kỳ trên log. Tính các chỉ số chất lượng theo thời gian, khi vượt ngưỡng cảnh báo thì tự động báo cho quản trị viên (email, Slack, dashboard báo đỏ).
Sơ đồ pipeline mình vẽ ở lượt trước mô tả đúng ba luồng này, bạn có thể đưa thẳng vào slide.

CHƯƠNG 5.	CÁCH ĐÁNH GIÁ 
Vì không train model, "độ chính xác" ở đây đo bằng chất lượng so khớp. Hai chỉ số cốt lõi:
-FAR (False Acceptance Rate) - tỷ lệ nhận nhầm: hệ thống chấp nhận sai người. Nguy hiểm nhất (gian lận chấm công hộ).
-FRR (False Rejection Rate) - tỷ lệ từ chối nhầm: đúng người nhưng không nhận ra. Gây phiền.
-Hai chỉ số này đánh đổi lẫn nhau qua ngưỡng: ngưỡng cao thì ít nhận nhầm nhưng hay từ chối nhầm, và ngược lại. Nhóm sẽ thử nhiều ngưỡng trên tập kiểm thử có nhãn, vẽ đường cong, và chọn ngưỡng cân bằng (thường quanh điểm EER - Equal Error Rate, nơi FAR và FRR bằng nhau).
