# Project Plan - Social Network Analysis

## Thông tin cơ bản

- **Dự án:** B.5 - Building an Intelligent Personalized Content Feed
- **Môn học:** Mạng Xã Hội (IS353.R11)
- **Nhóm:** 04
- **Thành viên:** Trần Hồng Phúc - 23521222
- **Dữ liệu:** MIND-small
- **Phạm vi được phân công:** Bộ đánh giá; baseline B1-B4; thí nghiệm E1 với Popularity
- **Deadline:** Cuối ngày 19/10/2026
- **Ngân sách làm việc:** 6 giờ theo chế độ Heavy Productivity

## Mục tiêu dự án

Xây dựng hệ thống news feed cá nhân hóa trên MIND-small dưới dạng temporal link prediction trên đồ thị dị thể. Dự án tập trung so sánh có kiểm soát các cấu hình đồ thị trên cùng backbone và protocol temporal, đo ảnh hưởng của data leakage, đồng thời phân tích nhóm người dùng và loại tin được hưởng lợi từ tín hiệu đồ thị.

Các câu hỏi nghiên cứu chính:

1. Tín hiệu đồ thị có cải thiện mô hình content-only hay không?
2. Dạng đồ thị nào hiệu quả nhất?
3. Nhóm người dùng và loại tin nào hưởng lợi nhiều nhất?
4. Random split hoặc dữ liệu tương lai làm metric bị thổi phồng bao nhiêu?

## Nguyên tắc thực hiện

- Metric được tính theo từng impression rồi mới lấy trung bình.
- Mọi thống kê, popularity và đặc trưng phải chỉ dùng dữ liệu có trước thời điểm dự đoán.
- Ngưỡng phân nhóm phải được xác định từ train, không dùng validation/test.
- Mỗi kết quả phải có config và seed để tái lập.
- Không tự điền hoặc ước đoán số liệu khi dữ liệu hay dependency chưa sẵn sàng; phải ghi rõ blocker.

## Tickets

### PHUC-01 - Bộ đánh giá và metric

**Ưu tiên:** P0

**Tasks**

- Chuẩn hóa schema input/output theo impression.
- Cài AUC, MRR, nDCG@5 và nDCG@10.
- Tính metric theo từng impression rồi lấy trung bình.
- Hỗ trợ phân tích theo độ dài history, cold/warm news và category.
- Viết test cho tie score, nhãn bất thường và dữ liệu thiếu.

**Output**

- Evaluator dùng chung cho mọi mô hình.
- Bộ test metric.
- Bảng metric tổng thể và theo nhóm.

### PHUC-02 - Baseline B1 và B2

**Ưu tiên:** P0

**Tasks**

- Cài B1 Random với seed cố định.
- Cài B2 theo tổng click trước thời điểm dự đoán.
- Cài B2 theo số click trong 24 giờ trước thời điểm dự đoán.
- Xử lý tin chưa từng được click và tie score.
- Kiểm thử để bảo đảm không sử dụng click tương lai.
- Chạy B1-B2 qua evaluator.

**Output**

- Baseline B1 và hai biến thể B2 chạy end-to-end.
- Bảng metric B1-B2.
- Test chống temporal leakage.

### PHUC-03 - Baseline B3 và B4

**Ưu tiên:** P0

**Tasks**

- Tạo hồ sơ category/subcategory từ lịch sử user cho B3.
- Chấm điểm tin ứng viên theo độ phù hợp category/subcategory.
- Tạo hoặc nạp embedding PLM của tin cho B4.
- Lấy trung bình embedding lịch sử làm vector user.
- Chấm điểm semantic match bằng cosine similarity.
- Xử lý user mới, history rỗng và metadata thiếu.
- Chạy B3-B4 qua evaluator.

**Output**

- Baseline B3 và B4 chạy end-to-end.
- Embedding cache kèm model/version.
- Bảng metric B3-B4.
- Thống kê các mẫu thiếu dữ liệu.

### PHUC-04 - E1 với Popularity

**Ưu tiên:** P0

**Tasks**

- Chạy B2 với Global Temporal Split đúng thời gian.
- Chạy B2 với Random Split 80/10/10 trên dữ liệu 09-15/11.
- Chạy B2 với Global Temporal Split nhưng popularity dùng dữ liệu tĩnh toàn kỳ 09-15/11.
- Giữ nguyên cấu hình và evaluator giữa các protocol.
- Tính chênh lệch tuyệt đối và tương đối của metric.
- Kiểm tra kích thước từng split và đánh dấu rõ kết quả có leakage.

**Output**

- Bảng AUC, MRR, nDCG@5 và nDCG@10 cho ba protocol.
- Bảng mức thổi phồng metric.
- Kết luận sơ bộ về ảnh hưởng của leakage lên Popularity.

### PHUC-05 - Kiểm thử, tái lập và bàn giao

**Ưu tiên:** P0

**Tasks**

- Chạy smoke test toàn pipeline.
- Ghi seed, cấu hình, phiên bản thư viện và model embedding.
- Chuẩn hóa định dạng kết quả.
- Viết hướng dẫn để B5, B6 và THG-Rec dùng lại evaluator.
- Ghi rõ phần hoàn thành, blocker và kết quả chưa chính thức.

**Output**

- README hoặc hướng dẫn chạy.
- Config và seed tái lập.
- Bảng kết quả tổng hợp B1-B4 và E1-Popularity.
- Evaluator sẵn sàng bàn giao cho cả nhóm.

## Nội dung Notes / Checklist cho Heavy Productivity

### How can I best achieve it

Hoàn thành phần việc được giao bằng cách xây dựng một evaluator thống nhất và có kiểm thử trước, sau đó triển khai B1-B4 trên cùng giao diện, kiểm soát nghiêm ngặt dữ liệu theo thời gian và dùng B2 để đo mức thổi phồng metric do leakage. Mọi kết quả phải tái lập được và có thể bàn giao để các thành viên khác tích hợp B5, B6 và THG-Rec.

### Output

- Evaluator cho AUC, MRR, nDCG@5 và nDCG@10.
- Metric tổng thể và theo nhóm user/news/category.
- Baseline B1-B4 chạy end-to-end.
- Kết quả E1-Popularity trên ba protocol và bảng mức thổi phồng metric.
- Test chống temporal leakage.
- Config, seed, embedding cache và hướng dẫn chạy tái lập.

### Future

- Tích hợp B5, B6 và THG-Rec vào evaluator chung.
- Chạy E1 đầy đủ với B6, G1 và THG-Rec.
- Tổng hợp nhiều seed, paired t-test và hiệu chỉnh Holm.
- Hỗ trợ E2, E3, E5 và phần phân tích SNA/báo cáo cuối kỳ.
