# Work Status

## Project Overview

| Field | Value |
|---|---|
| Repository | `mind-temporal-eval-baselines` |
| Dataset | MIND-small |
| Scope | PHUC-01 đến PHUC-05 |
| Project Stage | PHUC-01–03 Verified |
| Last Updated | 2026-10-19 |
| Current Focus | PHUC-03 đã nghiệm thu B3/B4 trên MIND-small; chờ người dùng review/xác nhận |

Nguồn phạm vi: [Project Plan](project-plan.md). Ràng buộc nghiên cứu: [Proposal](SNA-Proposal-v2.md), mục 3.2, 4.2–4.3, 5 và Appendix A.2/E1. Quy trình làm việc: [AGENTS.md](../AGENTS.md).

Placeholder không phải bằng chứng triển khai. Acceptance criteria dưới đây diễn giải các deliverable đã giao; các chi tiết còn mở phải được chốt trong kế hoạch được duyệt trước khi code. Không mở rộng sang mô hình graph hoặc kiểm định thống kê thuộc phần Future.

## Ticket Overview

| Ticket | Objective | Status | Last Updated |
|---|---|---|---|
| PHUC-01 | Evaluator & Metrics | Verified | 2026-10-19 |
| PHUC-02 | Random & Popularity | Verified | 2026-10-19 |
| PHUC-03 | Category & Semantic PLM | Verified | 2026-10-19 |
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

Các mục dưới giữ nguyên vấn đề ban đầu; quyết định và bằng chứng xử lý được ghi tiếp bên dưới và trong Evidence & Decision Log.

| ID | Vấn đề / quyết định cần xác nhận | Liên quan |
|---|---|---|
| OD-01 | Ban đầu raw/cache chỉ có `.gitkeep`. **Resolved, 2026-10-19:** người dùng cung cấp train/dev và sau đó tạo cache MiniLM thật; Codex đã xác minh checksum/model/version và chạy B1–B4 trên MIND-small. Chi tiết cache trong validation log PHUC-03. | 01–05 |
| OD-02 | Chốt schema, định nghĩa metric theo scorer MIND, MRR nhiều positive, tie, nhãn suy biến và chính sách thiếu dữ liệu. | 01, 05 |
| OD-03 | **Resolved cho PHUC-02, 2026-10-19:** cutoff đầu ngày; nguồn click impression train; không đếm History; cửa sổ `[d-24h,d)`. Kiểm thử biên, nguồn và audit thật Passed. Không suy rộng quyết định này thành protocol E1 hoặc Ghist; History vẫn không có timestamp từng click. | 02–04 |
| OD-04 | Chốt ngưỡng history từ train và cách đánh giá cold/warm/category trong impression chứa nhiều nhóm; không đồng nhất cold news với thời gian xuất bản. | 01, 05 |
| OD-05 | **Resolved cho PHUC-03, 2026-10-19:** người dùng duyệt subcategory frequency/category fallback; History của chính impression, toàn bộ ID; frozen MiniLM title + abstract, revision cố định, 256 word pieces, mean/cosine, fallback 0; xem config và validation log. Không suy ra History có trước đầu ngày. | 03 |
| OD-06 | Chốt biến thể B2 và nguồn click cho Random Split; tách split khỏi chính sách static popularity. Ghi hạn chế so sánh khi test set thay đổi và xử lý chênh lệch tương đối có mẫu số 0. | 04 |
| OD-07 | Xác nhận deadline/ngân sách: Plan ghi 19/10, 6 giờ; Proposal có lịch Phúc đến 27/10. Chốt phạm vi nhiều seed; Plan đặt tổng hợp nhiều seed/kiểm định trong Future. | 05, toàn phạm vi |
| OD-08 | **Resolved cho nghiệm thu PHUC-01, 2026-10-19.** Ban đầu pip trong phiên Codex bị chặn socket (WinError 10013), cache không có wheels. Người dùng đã cài requirements qua VS Code và cung cấp output pytest: 44 passed, 42 subtests passed, không Failed/Skipped. Không suy ra quyền mạng của sandbox Codex đã thay đổi. | 01 |
| OD-09 | **Resolved cho PHUC-03, 2026-10-19:** ban đầu pip/network probe WinError 10013, cache CLI thiếu torch. Người dùng cài dependencies/tạo cache qua VS Code; Codex kiểm tra cache, encoder thật offline và chạy B4 thành công. Không suy ra quyền mạng sandbox đã đổi; không phải lỗi Python 3.13. | 03 |

Khi giải quyết: giữ ID, ghi ngày, quyết định, xác nhận của người dùng và ảnh hưởng; không xóa lịch sử quan trọng.

**2026-10-19 — phê duyệt PHUC-02:** người dùng đã cung cấp `data/raw/train/` và `data/raw/dev/`, đồng ý kế hoạch và yêu cầu thực hiện. OD-01 được gỡ cho dữ liệu B1/B2 (PLM/cache cho PHUC-03 chưa được xác nhận). OD-03 chốt cho PHUC-02: cutoff đầu ngày; B2 tích lũy dùng click impression trước cutoff, B2-24h dùng `[cutoff - 24h, cutoff)`, không đếm History. Validation 14/11 dùng click 09–13/11; test/dev 15/11 dùng click 09–14/11. Cold/warm lấy count tích lũy của cùng nguồn quan sát, không phải toàn bộ Ghist. Giữ seed mặc định 0; chưa triển khai E1 hoặc tổng hợp nhiều seed.

**2026-10-19 — kế hoạch PHUC-01 được người dùng phê duyệt ("ok"):** OD-02/OD-04 chốt MRR nhiều positive theo MIND, stable tie theo thứ tự candidate, metric không xác định trả null và đếm riêng; schema strict. History rỗng tách riêng; Q25/Q75 học từ history dương của train. Cold/warm là cohort impression cold_only/warm_only/mixed/unknown; category là cohort có chứa category, cho phép chồng lấp, luôn giữ đầy đủ candidate. Metadata có cutoff không muộn hơn impression, chỉ chứa click trước cutoff. Cho phép thêm/cài NumPy, scikit-learn, PyYAML, pytest trong `.venv`. OD-03 về nguồn History và cách dựng B2 vẫn mở; không được suy ra từ hợp đồng metadata của evaluator.

## Ticket Details

### PHUC-01 — Evaluator & Metrics

#### A. Objective

Xây evaluator độc lập mô hình, chấm điểm theo impression và phân tích theo nhóm.

#### B. Task Checklist

- [x] Chuẩn hóa schema input/output, định danh impression và ánh xạ candidate–label–score.
- [x] Cài AUC, MRR, nDCG@5 và nDCG@10 theo định nghĩa đã chốt.
- [x] Tính metric từng impression rồi lấy trung bình.
- [x] Hỗ trợ nhóm history length, cold/warm news và category.
- [x] Viết test tie score, nhãn bất thường và dữ liệu thiếu.

Checklist trên ghi nhận phần đã triển khai; nghiệm thu dựa trên evidence ở mục D. Lượt pytest do người dùng chạy trong VS Code đã hoàn tất cả hai kiểm tra dependency trước đây bị skip.

#### C. Acceptance Criteria

- AC1: Evaluator nhận cùng schema cho các mô hình, giữ đầy đủ candidate; fixture có đáp án biết trước kiểm chứng bốn metric và aggregation.
- AC2: Chính sách edge cases được ghi rõ và test Passed; báo số mẫu bị loại nếu có.
- AC3: Có bảng overall/group-wise, ngưỡng chỉ học từ train và định nghĩa nhóm được duyệt; lưu bằng chứng có thể truy lại từng impression.

#### D. Implementation Evidence

- Files: `src/data_loader/schema.py`; `src/evaluation/{metrics,evaluator,group_analyzer}.py`; `tests/test_metrics_edge_cases.py`; `configs/eval_config.yaml`; `requirements.txt`; hướng dẫn API trong `README.md`.
- Lịch sử kiểm tra của Codex: `.venv\Scripts\python.exe -m unittest discover -s tests -p test_metrics_edge_cases.py -v` — 42 Passed, 0 Failed, 2 Skipped; khi đó reference/YAML Not Run và pytest chưa khởi động được (`No module named pytest`).
- **Evidence mới do người dùng cung cấp ngày 2026-10-19:** chạy `.\.venv\Scripts\python.exe -m pytest tests/test_metrics_edge_cases.py -v` trong terminal VS Code tại repository. Output nguyên văn kết luận: `44 passed, 42 subtests passed in 26.09s`; không Failed/Skipped. Môi trường hiển thị: Windows, Python 3.13.5, pytest 9.1.1, pluggy 1.7.0, interpreter `.venv\Scripts\python.exe`. Đây là output người dùng gửi trong hội thoại, không phải lượt Codex tự chạy lại; chưa có file transcript riêng. 42 subtests là các trường hợp con, không cộng thành 86 test độc lập.
- `test_reference_numpy_and_sklearn` và `test_yaml_matches_approved_config`: **Passed** trong output mới, gỡ phần kiểm chứng còn thiếu. Phiên bản NumPy/scikit-learn/PyYAML không có trong output nên chưa ghi số phiên bản cụ thể.
- Artifact sẵn có: [validation log lịch sử](../outputs/logs/phuc01_validation.md), [report JSON](../outputs/tables/phuc01_synthetic_2026-10-19/report.json), [overall](../outputs/tables/phuc01_synthetic_2026-10-19/overall.csv), [groups](../outputs/tables/phuc01_synthetic_2026-10-19/groups.csv), [per-impression](../outputs/tables/phuc01_synthetic_2026-10-19/per_impression.csv). Các file này giữ nguyên bằng chứng và môi trường của lượt chạy cũ; lượt pytest mới được ghi tại đây. Bảng metric vẫn là **synthetic**, không phải kết quả MIND.
- **AC1 Passed:** fixture tính tay, oracle AUC, reference NumPy/scikit-learn và aggregation. **AC2 Passed:** schema/edge cases và toàn bộ suite pytest. **AC3 Passed:** cohort giữ nguyên candidate, ngưỡng train-only, bảng/per-impression artifact sẵn có và YAML config check. Đủ điều kiện Verified cho phạm vi evaluator được duyệt; không đồng nghĩa đã đánh giá trên MIND.

#### E. Blockers & Open Decisions

OD-02/OD-04 đã chốt và triển khai; OD-08 đã gỡ cho nghiệm thu dựa trên output người dùng. Không còn blocker cho phạm vi PHUC-01 đã duyệt. PHUC-02 đã cung cấp dữ liệu MIND và kiểm chứng nguồn click upstream cho B1/B2; evaluator vẫn yêu cầu producer khác tự bảo đảm hợp đồng cutoff.

#### F. Next Actions

Evaluator đã được PHUC-02 dùng nguyên trạng trên validation/test MIND-small; 44 regression tests và 42 subtests Passed trong lượt kiểm tra tích hợp. Tiếp tục dùng giao diện này cho các ticket sau; không thay protocol nếu chưa được duyệt.

#### G. Last Updated

2026-10-19 — PHUC-01 Verified, đối chiếu output pytest do người dùng cung cấp và artifact synthetic sẵn có.

### PHUC-02 — Random & Popularity

#### A. Objective

Cung cấp B1 và hai biến thể B2 với dữ liệu quá khứ hợp lệ.

#### B. Task Checklist

- [x] Cài B1 Random với seed cố định.
- [x] Cài B2 tổng click trước cutoff đã duyệt.
- [x] Cài B2 số click trong cửa sổ 24 giờ trước cutoff.
- [x] Xử lý tin chưa có click và tie score theo chính sách đã chốt.
- [x] Kiểm thử không dùng click tại/sau cutoff và các biên cửa sổ.
- [x] Chạy B1 và cả hai B2 qua evaluator.

#### C. Acceptance Criteria

- AC1: B1 tái lập với cùng seed/input; hai B2 khớp số đếm trên fixture có timestamp biết trước.
- AC2: Test chống leakage Passed, bao gồm click đúng cutoff; không gán timestamp giả cho History.
- AC3: Có kết quả end-to-end và bảng bốn metric cho B1, B2 tích lũy, B2 24 giờ, gắn với split/config/seed và chính sách cold/tie.

#### D. Implementation Evidence

- Files: `src/baselines/{base,b1_random,b2_popularity}.py`, `src/data_loader/{mind_reader,temporal_split}.py`, `scripts/run_baselines.py`, `configs/baselines_config.yaml`, `tests/test_temporal_leakage.py`, hướng dẫn trong `README.md`. Không sửa logic evaluator PHUC-01.
- Tests: `.venv\Scripts\python.exe -m pytest tests/test_temporal_leakage.py tests/test_metrics_edge_cases.py -q` — **82 Passed, 42 subtests Passed, 0 Failed/Skipped** (38 PHUC-02 + 44 PHUC-01). Một cảnh báo quyền ghi pytest cache, không ảnh hưởng kết quả.
- Run thật seed 0 hoàn tất cả 3 baseline × 2 split: train 126.695, validation 30.270, train cuối 156.965, test/dev 73.152 impression. Không loại impression do metric không xác định; metadata category đủ cho mọi candidate.
- Evidence: [validation log](../outputs/logs/phuc02_validation.md), [bảng metrics](../outputs/tables/phuc02_mindsmall_seed0_2026-10-19/metrics.csv), [manifest](../outputs/tables/phuc02_mindsmall_seed0_2026-10-19/manifest.json). Bảng group theo split/model ở cùng thư mục run; prediction/metadata đầy đủ nằm trong `outputs/predictions/phuc02_mindsmall_seed0_2026-10-19/` (Git-ignored).
- AC1/AC2 **Passed**: fixtures + audit Counter độc lập trên 80 impression/3.446 candidate thực khớp score, count tích lũy và cutoff. AC3 **Passed**: đủ metric và provenance của sáu lượt chạy; 11 code hashes khớp manifest. **Verified** cho protocol đã duyệt, không phải popularity có toàn bộ Ghist hay kết quả E1.

#### E. Blockers & Open Decisions

Không còn blocker nghiệm thu PHUC-02. OD-01 đã gỡ cho dữ liệu train/dev; OD-03 đã chốt và kiểm chứng. Giới hạn nghiên cứu: loại History khỏi popularity, snapshot ngày, seed 0; chưa có tổng hợp nhiều seed hoặc kiểm định ý nghĩa thống kê. Cảnh báo pytest cache là không chặn; có thể dùng `-p no:cacheprovider` khi chạy lại test.

#### F. Next Actions

Người dùng đã duyệt PHUC-03; B2/GTS được tái sử dụng cho metadata cold/warm. B2/GTS sẵn sàng cho E1 sau khi duyệt protocol của PHUC-04; không tự chạy thêm test-set configurations hoặc commit/push.

#### G. Last Updated

2026-10-19 — PHUC-02 Verified sau kiểm thử, chạy thật train/dev và audit artifact.

### PHUC-03 — Category & Semantic PLM

#### A. Objective

Cung cấp B3 dựa trên hồ sơ chủ đề và B4 semantic matching bằng PLM đóng băng.

#### B. Task Checklist

- [x] Tạo hồ sơ category/subcategory từ History của chính impression theo giả định đã duyệt.
- [x] Chấm điểm candidate theo độ phù hợp category/subcategory.
- [x] Tạo/nạp embedding PLM thật: 65.238 news × 384 chiều; kiểm tra model/revision, checksum, mapping và encoder parity.
- [x] Lấy trung bình embedding history làm vector user; kiểm chứng synthetic và audit cosine trên dữ liệu thật.
- [x] Tính cosine similarity giữa user và candidate.
- [x] Xử lý user mới, history rỗng và metadata thiếu.
- [x] Chạy B3–B4 qua evaluator trên MIND-small, đủ validation/test và diagnostics.

#### C. Acceptance Criteria

- AC1: B3/B4 khớp ví dụ kiểm chứng đơn giản; fallback và zero-norm handling được xác nhận, không tạo score không hợp lệ.
- AC2: Có embedding cache với model/revision, preprocessing và mapping news ID; history không dùng hành vi tương lai.
- AC3: Có bảng bốn metric end-to-end cho B3–B4 và thống kê mẫu thiếu dữ liệu, gắn config/seed và cache đã dùng.

#### D. Implementation Evidence

- Files: `src/baselines/{base,b3_category,b4_semantic_plm}.py`, `src/data_loader/mind_reader.py`, `scripts/{cache_plm_embeddings,run_baselines}.py`, `configs/content_baselines_config.yaml`, `tests/test_content_baselines.py`, `requirements.txt`, hướng dẫn README. Không sửa evaluator/B1/B2 scoring.
- Tests: `.venv\Scripts\python.exe -m pytest tests/test_content_baselines.py tests/test_temporal_leakage.py tests/test_metrics_edge_cases.py -q -p no:cacheprovider` — **123 Passed, 42 subtests Passed, 0 Failed/Skipped** (41 mới + 82 regression). Cache encoder mock không phải PLM thật.
- B3 thật seed 0: 30.270 validation + 73.152 test impressions; tất cả metric hợp lệ. [Metrics](../outputs/tables/phuc03_b3_mindsmall_seed0_2026-10-19/metrics.csv), [manifest](../outputs/tables/phuc03_b3_mindsmall_seed0_2026-10-19/manifest.json), [validation log](../outputs/logs/phuc03_validation.md). Prediction chi tiết ở `outputs/predictions/phuc03_b3_mindsmall_seed0_2026-10-19/` (Git-ignored).
- Audit độc lập từ TSV: **Passed**, 100 impressions/4.016 candidate scores và thứ tự khớp; 14 code hashes khớp manifest. Lệnh audit đầu sai tên cột `news_ids`; sửa thành `candidate_ids`, không sửa artifact. `git diff --check` Passed.
- Lịch sử: dependency install/cache CLI **Failed** vì môi trường; B4 ban đầu **Not Run**, ticket ở Implemented với AC2/AC3 Partial. Đã gỡ OD-09 bằng cache/dependencies người dùng cung cấp; không xóa lịch sử lỗi.
- **Tiếp tục 2026-10-19:** cache `data/cache/phuc03_minilm_2026-10-19/` Passed toàn bộ checksum/content/mapping/code/version, 65.238 × 384 float32. MiniLM revision `1110a243fdf4706b3f48f1d95db1a4f5529b4d41`; torch 2.14.1, sentence-transformers 5.7.0, transformers 5.20.1. Encoder thật offline trên 8 tin khớp cache (max abs error 7,45e-8).
- Chỉ B4 chạy mới, exit 0, validation 30.270 + test 73.152 impressions; tất cả metric hợp lệ. [B4 metrics](../outputs/tables/phuc03_b4_mindsmall_seed0_2026-10-19/metrics.csv), [B4 manifest](../outputs/tables/phuc03_b4_mindsmall_seed0_2026-10-19/manifest.json); predictions ở `outputs/predictions/phuc03_b4_mindsmall_seed0_2026-10-19/` (Git-ignored). B3 không chạy lại.
- Regression trong môi trường mới: **123 Passed + 42 subtests Passed, 3.24s, 0 Failed/Skipped**. Audit B4 100 impressions/4.016 scores Passed (cosine độc lập sai số tối đa 5,28e-8); đảo nhãn và chấm context tương lai không đổi score hiện tại; count metadata đúng nguồn train và cutoff. 14 run code hashes/input/cache hash khớp. Chi tiết và cảnh báo API không chặn trong validation log.
- **AC1–AC3 Passed:** công thức/fallback, cache PLM thật + provenance/isolation, đủ bảng B3/B4 + diagnostics. **Verified** theo protocol seed 0 đã duyệt; không khẳng định timestamp History đã được chứng minh.

#### E. Blockers & Open Decisions

Không còn blocker nghiệm thu PHUC-03; OD-01/OD-05/OD-09 đã giải quyết trong phạm vi này. History không có timestamp từng click: chỉ giả định trước impression, không khẳng định trước đầu ngày hay trước 09/11. Seed 0, chưa có multi-seed/significance; không thay protocol PHUC-02. Cảnh báo đổi tên API lấy chiều embedding không chặn, chưa cần sửa source.

#### F. Next Actions

Chờ người dùng review/xác nhận trước kế hoạch PHUC-04. Giữ cache và artifact làm bằng chứng; không chạy thêm test configurations để tuning, không tự commit/push.

#### G. Last Updated

2026-10-19 — PHUC-03 Verified sau kiểm tra cache/encoder thật, B4 end-to-end, regression và audit temporal.

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
- Tests/experiment runs: **Not Run** — pipeline E1 chưa triển khai; evaluator PHUC-01 và B2 PHUC-02 đã Verified.
- Outputs/logs: chưa có; chưa có bằng chứng đạt AC1–AC3.

#### E. Blockers & Open Decisions

Dữ liệu và B2 đã có từ PHUC-02. OD-03/OD-06 cho E1 vẫn cần xác nhận nguồn click từng protocol và biến thể B2; không tự coi protocol PHUC-02 là quyết định cho các nhánh leaky.

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

OD-01/OD-09 đã gỡ; OD-07 còn mở. PHUC-01–03 đã Verified, PHUC-04 chưa triển khai. Cần duyệt phạm vi tái chạy và tiêu chí so sánh phù hợp tài nguyên.

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
| 2026-10-19 | PHUC-01 approval | Người dùng trả lời "ok" cho kế hoạch schema/metrics/cohorts, tests và thêm/cài bốn dependencies. | Not Started → In Progress; OD-02/OD-04 chốt theo quyết định đã ghi; không mở rộng PHUC-02–05. |
| 2026-10-19 | PHUC-01 environment | Tạo `.venv` thành công; pip install Failed do WinError 10013, cache 0 wheels/0 HTTP files. | OD-08 mở. Core dùng thư viện chuẩn; suite tương thích pytest/unittest, YAML và reference parity cần dependencies. |
| 2026-10-19 | PHUC-01 verification | Lượt đầu 41 Passed/2 Skipped; sau chuẩn hóa scalar và test mới: 42 Passed/2 Skipped, 0 Failed. Synthetic CSV/JSON xuất thành công và khớp code hiện tại; chi tiết trong validation log. | In Progress → Implemented, chưa Verified: pytest, reference parity và YAML checks Not Run. PHUC-02–05 giữ Not Started; không commit/push. |
| 2026-10-19 | PHUC-01 user verification | Người dùng báo đã cài requirements qua VS Code và gửi output `.\.venv\Scripts\python.exe -m pytest tests/test_metrics_edge_cases.py -v`: `44 passed, 42 subtests passed in 26.09s`; Python 3.13.5, pytest 9.1.1, pluggy 1.7.0. Reference NumPy/scikit-learn và YAML đều Passed. Không phải lượt Codex tự chạy lại. | Implemented → Verified; AC1–AC3 đạt với evidence mới và artifact synthetic sẵn có. OD-08 resolved cho nghiệm thu; giữ nguyên lịch sử lỗi. PHUC-02–05 vẫn Not Started; kế hoạch PHUC-02 chờ duyệt, chưa code. |
| 2026-10-19 | PHUC-02 approval/data | Người dùng cung cấp train/dev và đồng ý kế hoạch. GTS đối chiếu đúng Proposal; duyệt cutoff đầu ngày và chỉ đếm click có timestamp, loại History. | Not Started → In Progress; OD-01 gỡ cho dữ liệu B1/B2, OD-03 chốt riêng PHUC-02. |
| 2026-10-19 | PHUC-02 implementation/verification | 82 tests + 42 subtests Passed; runner thật exit 0, 6 model/split runs completed, seed 0. Audit độc lập 80 impression/3.446 candidate khớp; code hashes và n_valid khớp manifest. | In Progress → Verified, AC1–AC3 đạt. Lưu metrics/manifest/group/prediction và validation log; warning pytest cache không chặn. Không triển khai PHUC-03–05, không commit/push. |
| 2026-10-19 | PHUC-03 approval | Người dùng duyệt kế hoạch bằng "oke", gồm thêm/cài sentence-transformers/PyTorch và tải model. History của chính impression, toàn bộ history; B3 subcategory với category fallback; B4 frozen MiniLM trên title + abstract; seed 0. | Not Started → In Progress. History chỉ được giả định có trước impression, không khẳng định trước đầu ngày; không thay B2/evaluator. Chưa có bằng chứng nghiệm thu PHUC-03. |
| 2026-10-19 | PHUC-03 environment | Hai pip attempts thất bại; PyPI probe WinError 10013; cache CLI thiếu torch. | OD-09 Open, không có cache PLM thật; không đổi quyền mạng, không giả lập kết quả B4 thật. |
| 2026-10-19 | PHUC-03 verification | 123 tests + 42 subtests Passed; B3 MIND-small validation/test completed; audit 100 impressions/4.016 scores Passed sau sửa tên cột của lệnh audit; 14 code hashes khớp. Xem validation log và manifest. | In Progress → Implemented; AC1 Passed, AC2/AC3 Partial. Chờ dependencies/cache/B4 thật; chưa Verified, không chuyển PHUC-04, không commit/push. |
| 2026-10-19 | PHUC-03 real-cache/B4 acceptance | Người dùng cài dependencies/tạo cache; Codex xác minh 65.238 embeddings, pinned revision/version/checksum và encoder parity 8 tin. Chỉ B4 chạy validation/test thành công; 123 tests + 42 subtests Passed; audit cosine/label isolation/cutoff trên 100 impressions/4.016 candidates Passed. | Implemented → Verified, AC1–AC3 Passed theo protocol được duyệt. OD-01/OD-09 resolved; giữ hạn chế History/seed 0 và lịch sử lỗi. Không rerun B3, không sửa source/config, không chuyển PHUC-04 hoặc commit/push. |
