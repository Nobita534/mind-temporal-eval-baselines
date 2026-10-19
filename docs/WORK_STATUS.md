# Work Status

## Project Overview

| Field | Value |
|---|---|
| Repository | `mind-temporal-eval-baselines` |
| Dataset | MIND-small |
| Scope | PHUC-01 đến PHUC-05 |
| Project Stage | Initial Scaffolding |
| Last Updated | 2026-10-19 |
| Current Focus | Chưa bắt đầu implementation |

Nguồn phạm vi: [Project Plan](project-plan.md). Ràng buộc nghiên cứu: [Proposal](SNA-Proposal-v2.md), mục 3.2, 4.2–4.3, 5 và Appendix A.2/E1. Quy trình làm việc: [AGENTS.md](../AGENTS.md).

Placeholder không phải bằng chứng triển khai. Acceptance criteria dưới đây diễn giải các deliverable đã giao; các chi tiết còn mở phải được chốt trong kế hoạch được duyệt trước khi code. Không mở rộng sang mô hình graph hoặc kiểm định thống kê thuộc phần Future.

## Ticket Overview

| Ticket | Objective | Status | Last Updated |
|---|---|---|---|
| PHUC-01 | Evaluator & Metrics | Not Started | 2026-10-19 |
| PHUC-02 | Random & Popularity | Not Started | 2026-10-19 |
| PHUC-03 | Category & Semantic PLM | Not Started | 2026-10-19 |
| PHUC-04 | E1 Temporal Leakage | Not Started | 2026-10-19 |
| PHUC-05 | Testing & Reproducibility | Not Started | 2026-10-19 |

## Ticket Status Definitions

| Status | Điều kiện |
|---|---|
| Not Started | Chưa triển khai; placeholder không làm thay đổi trạng thái. |
| In Progress | Đang triển khai theo kế hoạch đã được duyệt. |
| Blocked | Không thể tiếp tục công việc do vấn đề chưa giải quyết; ghi nguyên nhân và điều kiện gỡ chặn. |
| Implemented | Đã viết code nhưng chưa hoàn tất nghiệm thu. |
| Verified | Đạt toàn bộ acceptance criteria bắt buộc, có bằng chứng kiểm chứng. |

Có open decision khi chưa bắt đầu không tự động chuyển ticket sang Blocked. Test Passed đơn lẻ không đủ để đánh dấu Verified. Khi bằng chứng không còn hợp lệ sau thay đổi, đánh giá lại trạng thái và ghi lý do.

## Shared Blockers & Open Decisions

Các mục sau đều đang mở; chưa có quyết định triển khai được phê duyệt trong tracker này.

| ID | Vấn đề / quyết định cần xác nhận | Liên quan |
|---|---|---|
| OD-01 | `data/raw/` và `data/cache/` hiện chỉ có `.gitkeep`; cần nguồn dữ liệu/model hợp lệ trước chạy end-to-end. Fixture không thay thế kết quả MIND. | 01–05 |
| OD-02 | Chốt schema, định nghĩa metric theo scorer MIND, MRR nhiều positive, tie, nhãn suy biến và chính sách thiếu dữ liệu. | 01, 05 |
| OD-03 | Chốt cutoff theo impression hay ngày; nguồn click, cách tránh đếm lặp History và biên cửa sổ 24 giờ. History không có timestamp từng click; không tự giả định toàn bộ đều trước 09/11. | 02–04 |
| OD-04 | Chốt ngưỡng history từ train và cách đánh giá cold/warm/category trong impression chứa nhiều nhóm; không đồng nhất cold news với thời gian xuất bản. | 01, 05 |
| OD-05 | Chốt kết hợp category/subcategory, fallback thiếu dữ liệu; PLM revision, trường text, giới hạn history và mapping cache. | 03 |
| OD-06 | Chốt biến thể B2 và nguồn click cho Random Split; tách split khỏi chính sách static popularity. Ghi hạn chế so sánh khi test set thay đổi và xử lý chênh lệch tương đối có mẫu số 0. | 04 |
| OD-07 | Xác nhận deadline/ngân sách: Plan ghi 19/10, 6 giờ; Proposal có lịch Phúc đến 27/10. Chốt phạm vi nhiều seed; Plan đặt tổng hợp nhiều seed/kiểm định trong Future. | 05, toàn phạm vi |

Khi giải quyết: giữ ID, ghi ngày, quyết định, xác nhận của người dùng và ảnh hưởng; không xóa lịch sử quan trọng.

## Ticket Details

### PHUC-01 — Evaluator & Metrics

#### A. Objective

Xây evaluator độc lập mô hình, chấm điểm theo impression và phân tích theo nhóm.

#### B. Task Checklist

- [ ] Chuẩn hóa schema input/output, định danh impression và ánh xạ candidate–label–score.
- [ ] Cài AUC, MRR, nDCG@5 và nDCG@10 theo định nghĩa đã chốt.
- [ ] Tính metric từng impression rồi lấy trung bình.
- [ ] Hỗ trợ nhóm history length, cold/warm news và category.
- [ ] Viết test tie score, nhãn bất thường và dữ liệu thiếu.

#### C. Acceptance Criteria

- AC1: Evaluator nhận cùng schema cho các mô hình, giữ đầy đủ candidate; fixture có đáp án biết trước kiểm chứng bốn metric và aggregation.
- AC2: Chính sách edge cases được ghi rõ và test Passed; báo số mẫu bị loại nếu có.
- AC3: Có bảng overall/group-wise, ngưỡng chỉ học từ train và định nghĩa nhóm được duyệt; lưu bằng chứng có thể truy lại từng impression.

#### D. Implementation Evidence

- Files/modules đã triển khai: chưa có.
- Tests: **Not Run** — chưa có implementation/test logic.
- Outputs/logs: chưa có; chưa có bằng chứng đạt AC1–AC3.

#### E. Blockers & Open Decisions

OD-02, OD-04 cần chốt trước phần triển khai liên quan; OD-01 ảnh hưởng đánh giá trên MIND. Chưa đánh dấu Blocked vì chưa bắt đầu.

#### F. Next Actions

Trình kế hoạch schema/metric/group và fixture để duyệt. Module dự kiến: `src/data_loader/schema.py`, `src/evaluation/`, `tests/test_metrics_edge_cases.py`; cần giao diện loader nhưng không phụ thuộc baseline.

#### G. Last Updated

2026-10-19 — khởi tạo tracker, chưa triển khai.

### PHUC-02 — Random & Popularity

#### A. Objective

Cung cấp B1 và hai biến thể B2 với dữ liệu quá khứ hợp lệ.

#### B. Task Checklist

- [ ] Cài B1 Random với seed cố định.
- [ ] Cài B2 tổng click trước cutoff đã duyệt.
- [ ] Cài B2 số click trong cửa sổ 24 giờ trước cutoff.
- [ ] Xử lý tin chưa có click và tie score theo chính sách đã chốt.
- [ ] Kiểm thử không dùng click tại/sau cutoff và các biên cửa sổ.
- [ ] Chạy B1 và cả hai B2 qua evaluator.

#### C. Acceptance Criteria

- AC1: B1 tái lập với cùng seed/input; hai B2 khớp số đếm trên fixture có timestamp biết trước.
- AC2: Test chống leakage Passed, bao gồm click đúng cutoff; không gán timestamp giả cho History.
- AC3: Có kết quả end-to-end và bảng bốn metric cho B1, B2 tích lũy, B2 24 giờ, gắn với split/config/seed và chính sách cold/tie.

#### D. Implementation Evidence

- Files/modules đã triển khai: chưa có.
- Tests: **Not Run** — chưa có implementation/test logic.
- Outputs/logs: chưa có; chưa có bằng chứng đạt AC1–AC3.

#### E. Blockers & Open Decisions

OD-01, OD-03; cần xác định cách dùng History cho đếm tích lũy và cửa sổ 24 giờ.

#### F. Next Actions

Phụ thuộc schema/evaluator PHUC-01 và loader/split. Trình kế hoạch `b1_random.py`, `b2_popularity.py` cùng fixture temporal trước khi code.

#### G. Last Updated

2026-10-19 — khởi tạo tracker, chưa triển khai.

### PHUC-03 — Category & Semantic PLM

#### A. Objective

Cung cấp B3 dựa trên hồ sơ chủ đề và B4 semantic matching bằng PLM đóng băng.

#### B. Task Checklist

- [ ] Tạo hồ sơ category/subcategory từ history hợp lệ của user.
- [ ] Chấm điểm candidate theo độ phù hợp category/subcategory.
- [ ] Tạo hoặc nạp embedding PLM của tin.
- [ ] Lấy trung bình embedding history làm vector user.
- [ ] Tính cosine similarity giữa user và candidate.
- [ ] Xử lý user mới, history rỗng và metadata thiếu.
- [ ] Chạy B3–B4 qua evaluator.

#### C. Acceptance Criteria

- AC1: B3/B4 khớp ví dụ kiểm chứng đơn giản; fallback và zero-norm handling được xác nhận, không tạo score không hợp lệ.
- AC2: Có embedding cache với model/revision, preprocessing và mapping news ID; history không dùng hành vi tương lai.
- AC3: Có bảng bốn metric end-to-end cho B3–B4 và thống kê mẫu thiếu dữ liệu, gắn config/seed và cache đã dùng.

#### D. Implementation Evidence

- Files/modules đã triển khai: chưa có.
- Tests: **Not Run** — chưa có implementation/test logic.
- Outputs/logs/cache embedding: chưa có; chưa có bằng chứng đạt AC1–AC3.

#### E. Blockers & Open Decisions

OD-01, OD-03, OD-05; chưa xác nhận dữ liệu/model sẵn sàng hoặc chính sách fallback.

#### F. Next Actions

Phụ thuộc PHUC-01, metadata/history chuẩn hóa và PLM/cache. Trình kế hoạch `b3_category.py`, `b4_semantic_plm.py`, `scripts/cache_plm_embeddings.py`; không tự thêm dependencies.

#### G. Last Updated

2026-10-19 — khởi tạo tracker, chưa triển khai.

### PHUC-04 — E1 Temporal Leakage

#### A. Objective

Đo chênh lệch metric của Popularity giữa protocol sạch và hai protocol cố ý có leakage.

#### B. Task Checklist

- [ ] Chạy B2 với Global Temporal Split đúng thời gian.
- [ ] Chạy B2 với Random Split 80/10/10 trên 09–15/11/2019.
- [ ] Chạy B2 với GTS nhưng popularity tĩnh dùng toàn kỳ 09–15/11/2019.
- [ ] Giữ cấu hình mô hình/evaluator chung, chỉ thay các yếu tố protocol đã định nghĩa.
- [ ] Tính chênh lệch tuyệt đối và tương đối so với GTS sạch.
- [ ] Kiểm tra kích thước từng split và gắn nhãn kết quả có leakage.

#### C. Acceptance Criteria

- AC1: Có bảng AUC/MRR/nDCG@5/nDCG@10 cho đủ ba protocol, kèm split sizes, seed/config và nguồn click; không tách candidate của cùng impression sang nhiều split.
- AC2: Có bảng chênh lệch tuyệt đối/tương đối, xử lý mẫu số 0 và giữ nguyên chênh lệch âm nếu xuất hiện; cache/output sạch và leaky được phân biệt.
- AC3: Kết luận sơ bộ dựa trên số liệu thực, nêu giới hạn do Random thay test set; không suy rộng sang mô hình graph.

#### D. Implementation Evidence

- Files/modules đã triển khai: chưa có.
- Tests/experiment runs: **Not Run** — chưa có B2/evaluator/pipeline.
- Outputs/logs: chưa có; chưa có bằng chứng đạt AC1–AC3.

#### E. Blockers & Open Decisions

OD-01, OD-03, OD-06; không mặc định nguồn click hoặc biến thể B2 của thí nghiệm.

#### F. Next Actions

Phụ thuộc PHUC-01 và PHUC-02. Trình ma trận protocol, nguồn dữ liệu và phép so sánh để duyệt trước khi triển khai `scripts/run_e1_leakage.py` / `configs/e1_protocols.yaml`.

#### G. Last Updated

2026-10-19 — khởi tạo tracker, chưa triển khai.

### PHUC-05 — Testing & Reproducibility

#### A. Objective

Kiểm chứng luồng chung, chuẩn hóa artifact và bàn giao evaluator có thể tái sử dụng.

#### B. Task Checklist

- [ ] Chạy smoke test toàn pipeline trên dữ liệu nhỏ.
- [ ] Ghi seed, resolved config, phiên bản thư viện và model embedding.
- [ ] Chuẩn hóa định dạng kết quả.
- [ ] Viết hướng dẫn B5, B6 và THG-Rec dùng lại evaluator.
- [ ] Ghi rõ phần hoàn thành, blocker và kết quả chưa chính thức.

#### C. Acceptance Criteria

- AC1: Smoke test Passed với lệnh, kết quả và môi trường được ghi lại; bàn giao đủ bằng chứng nghiệm thu PHUC-01–04.
- AC2: Có hướng dẫn chạy, config/seed, data/split identity và model revision; có bằng chứng tái chạy kiểm tra đã thống nhất, không chỉ lưu tham số.
- AC3: Có bảng tổng hợp B1–B4/E1 và hợp đồng prediction cho mô hình ngoài; công khai giới hạn, blocker và trạng thái kết quả. Không yêu cầu triển khai B5/B6/THG-Rec.

#### D. Implementation Evidence

- Files/modules/hướng dẫn chạy đã triển khai cho ticket: chưa có; tài liệu scaffolding không chứng minh workflow chạy được.
- Tests: **Not Run** — chưa có pipeline/test logic.
- Outputs/logs: chưa có; chưa có bằng chứng đạt AC1–AC3.

#### E. Blockers & Open Decisions

OD-01, OD-07; PHUC-01–04 chưa triển khai. Cần duyệt phạm vi tái chạy và tiêu chí so sánh phù hợp tài nguyên.

#### F. Next Actions

Phụ thuộc PHUC-01–04 cho nghiệm thu cuối. Chuẩn bị kế hoạch metadata/handoff từ đầu; chỉ triển khai hoặc chuyển ticket khi được xác nhận. Module dự kiến: smoke test, CLI chấm prediction, configs và hướng dẫn chạy.

#### G. Last Updated

2026-10-19 — khởi tạo tracker, chưa triển khai.

## Update Rules

1. Sau mỗi task được duyệt và thực hiện, cập nhật checklist tương ứng; chỉ tick khi có bằng chứng hoàn thành mục đó.
2. Đồng bộ status/date ở Ticket Overview, chi tiết ticket và Project Overview; không suy ra tiến độ từ số file.
3. Bổ sung evidence thực vào mục D: file/module, lệnh kiểm tra, Passed/Failed/Not Run, lý do chưa chạy, môi trường/config/seed và đường dẫn artifact tồn tại.
4. Đối chiếu từng AC, ghi phần còn thiếu; chỉ Verified khi tất cả tiêu chí bắt buộc đạt và có bằng chứng. Không đổi AC để hợp thức hóa kết quả nếu chưa được duyệt.
5. Cập nhật blockers/open decisions và Next Actions/dependencies; giữ lịch sử blocker quan trọng cùng cách xử lý.
6. Thêm dòng Evidence & Decision Log; ghi cả kiểm tra thất bại, không xóa để thay bằng lần chạy thành công.
7. Bàn giao ngắn gọn: chức năng, files, checks, AC, blockers/next actions và giải thích kỹ thuật cần thiết. Chờ người dùng xác nhận trước task/ticket tiếp theo; không tự commit/push.

## Evidence & Decision Log

Các đường dẫn bằng chứng chỉ được thêm khi artifact thực sự tồn tại. Việc tạo tracker không hoàn thành task implementation nào.

| Date | Scope | Event / Evidence | Result / Decision / Remaining |
|---|---|---|---|
| 2026-10-19 | Khởi tạo tracker | Đối chiếu Project Plan và kiểm tra scaffolding: file Python/YAML rỗng; thư mục raw/cache và output chỉ có `.gitkeep`. | PHUC-01–05: Not Started; mọi checklist chưa tick; kiểm thử ticket Not Run. OD-01–07 còn mở. |
| 2026-10-19 | Kiểm tra tài liệu | Kiểm tra cấu trúc: checklist 5/6/7/6/5 khớp 29 task của Plan; mỗi ticket đủ A–G, 5 trạng thái Not Started, 0 mục đã tick. Đối chiếu quy tắc nghiệm thu, evidence và xác nhận chuyển task với AGENTS.md. | Passed cho kiểm tra tài liệu; không phải bằng chứng implementation. `git diff --check`: exit 0; kiểm tra thêm `git diff --no-index --check` với `/dev/null` cho hai file untracked không báo lỗi whitespace (exit 1 do có khác biệt; có cảnh báo LF/CRLF). Tests/ML experiments: Not Run, ngoài phạm vi. |
