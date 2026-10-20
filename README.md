# MIND Temporal Evaluation & Baselines (`mind-temporal-eval-baselines`)

Repository này cung cấp **Bộ khung Đánh giá Chuẩn hóa theo Thời gian (Temporal Evaluation Suite)** và các **Mô hình Mốc sàn (Baselines B1–B4)** trên tập dữ liệu **MIND-small**, thuộc khuôn khổ đồ án môn Mạng Xã Hội (IS353.R11) — **Đề tài B.5: Building an Intelligent Personalized Content Feed (Nhóm 04)**.

## 🎯 Mục tiêu chính
1. **Standardized Evaluator (`PHUC-01`):** Cung cấp công cụ chấm điểm thống nhất (AUC, MRR, nDCG@5, nDCG@10) tính theo từng lượt hiển thị (impression-level) và bóc tách kết quả theo nhóm người dùng (history length), độ mới của tin (cold/warm news) và chủ đề (category).
2. **Zero-Training Baselines (`PHUC-02`, `PHUC-03`):** Cài đặt 4 mô hình cơ sở tuân thủ nghiêm ngặt ràng buộc thời gian ($G_{<t}$):
   - **B1 (Random):** Xếp hạng ngẫu nhiên với seed cố định.
   - **B2 (Temporal Popularity):** Xếp hạng theo tổng lượt click trong quá khứ ($<t$) và trong cửa sổ 24 giờ gần nhất.
   - **B3 (Category Match):** Khớp phân phối chủ đề (`category`/`subcategory`) từ lịch sử người dùng.
   - **B4 (Semantic Match - PLM):** Khớp ngữ nghĩa văn bản sử dụng vector tiền huấn luyện `all-MiniLM-L6-v2`.
3. **Quantifying Temporal Data Leakage (`PHUC-04` - Thí nghiệm E1):** Đo lường mức độ thổi phồng kết quả (metric inflation) trên mô hình Popularity khi vi phạm ràng buộc thời gian (so sánh giữa *Global Temporal Split chuẩn*, *Static Popularity toàn kỳ* và *Random Split 80/10/10*).

---

## PHUC-01: sử dụng evaluator

Đã triển khai API schema, metric, evaluator và phân nhóm; B1/B2 và runner GTS được
mô tả ở mục PHUC-02 bên dưới. B3/B4 đã được nghiệm thu trên MIND-small theo protocol
PHUC-03 được duyệt (seed 0, PLM đóng băng). E1 và statistical tests vẫn là placeholder. Các kết quả
trong `outputs/tables/phuc01_synthetic_2026-10-19/` chỉ là **synthetic fixtures**,
không phải kết quả thực nghiệm MIND-small.

### Môi trường và kiểm thử

Chạy từ thư mục gốc repository (PowerShell):

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m pytest tests/test_metrics_edge_cases.py -v
```

Core API dùng thư viện chuẩn Python 3.10+; môi trường kiểm tra hiện tại là 3.13.5.
NumPy/scikit-learn dùng để đối chiếu metric tham chiếu, PyYAML để đọc config và
pytest để chạy suite. Nếu mạng chặn cài dependencies, suite cũng chạy được bằng:

```powershell
.\.venv\Scripts\python.exe -m unittest discover -s tests -p test_metrics_edge_cases.py -v
```

Các kiểm tra cần dependency sẽ **skip với lý do**, không được coi là Passed.
Mỗi `report.json` ghi phiên bản Python và các dependency thực tế; giá trị null nghĩa
là chưa cài. `requirements.txt` khai báo khoảng phiên bản tương thích, chưa phải lockfile.

### Ví dụ API — dữ liệu tổng hợp

```python
from datetime import datetime
from src.data_loader.schema import CandidatePrediction, ImpressionPrediction
from src.evaluation.evaluator import evaluate, write_tables

row = ImpressionPrediction(
    impression_id="demo-1", user_id="U1", source="synthetic", split="test",
    timestamp=datetime(2019, 11, 15, 12), history_length=4,
    candidates=(
        CandidatePrediction("N1", 1, 0.9, "sports", 0),
        CandidatePrediction("N2", 0, 0.5, "finance", 2),
        CandidatePrediction("N3", 1, 0.1, "sports", 0),
    ),
    metadata_cutoff=datetime(2019, 11, 15),
    metadata_source="synthetic_counts",
)
result = evaluate([row], run_metadata={"data_kind": "synthetic"})
print(result.per_impression[0]["mrr"])  # (1 + 1/3) / 2 = 2/3
write_tables(result, "outputs/tables/my_new_run")
```

`write_tables` chỉ tạo **thư mục run mới**, không ghi đè run cũ. Xuất
`per_impression.csv`, `overall.csv`, `groups.csv` và `report.json`. JSON giữ null;
CSV biểu diễn null bằng ô trống. Candidate arrays và nhóm trong CSV là chuỗi JSON.
Mỗi dòng tổng hợp có `metric`, `mean`, `n_impressions`, `n_valid`, `n_undefined`.
Report giữ nguyên candidate IDs, labels, scores, metadata, config và nguồn ngưỡng.

### Hợp đồng input và chính sách đánh giá

- Mỗi `ImpressionPrediction` chứa đủ candidate theo thứ tự gốc. Khóa là
  `(source, impression_id)`; candidate ID không trùng trong impression. Label là
  integer 0/1; score hữu hạn. Lỗi dữ liệu làm dừng đánh giá, không tự bỏ candidate.
- Đánh giá một split mỗi lần: `train`, `validation`, `test` hoặc `train_final`.
  Nguồn cung cấp chịu trách nhiệm ánh xạ score đúng news ID và giữ đủ candidate.
- AUC cho tie nửa điểm; ranking MRR/nDCG giữ thứ tự candidate khi score bằng nhau.
  MRR lấy trung bình reciprocal rank của **mọi positive**, không chỉ click đầu tiên.
  Công thức theo [Recommenders reference](https://recommenders-team.github.io/recommenders/_modules/recommenders/models/deeprec/deeprec_utils.html#mrr_score);
  quy tắc stable tie được xác định riêng, không giả định trùng thứ tự tie của NumPy.
- AUC một lớp trả null; không có positive thì MRR/nDCG trả null. Tổng hợp macro
  theo impression và mẫu số hợp lệ của từng metric; không gộp candidate toàn tập,
  không làm tròn trung gian. Các nhóm không xuất hiện không được tạo dòng giả.
- `prior_click_count=None` nghĩa là chưa biết; 0 nghĩa là không có click được quan
  sát trong nguồn lịch sử đã khai báo. Count cần `metadata_cutoff <= timestamp`
  và `metadata_source`. Producer phải bảo đảm mọi click đóng góp đều **< cutoff**.
  Evaluator chỉ kiểm tra mốc/provenance, không xác minh lịch sử sự kiện không được
  truyền vào. Không suy diễn timestamp History hoặc publication time.
- Timestamp dùng cùng hệ quy chiếu naive/aware; không tự gán timezone cho MIND.
  Độ dài history là độ dài gốc khả dụng tại impression, không phải độ dài sau padding.

### Phân nhóm và config

Truyền `train_impressions=...` cho `evaluate` để học Q25/Q75 từ history **dương**
của train, tính theo impression bằng linear quantile. Validation/test/train_final
không được dùng để fit. Train phải trước mọi impression đánh giá ngoài train và
không trùng khóa với evaluation. Ngưỡng chỉ dùng để phân tích, không sửa score.

- History: `empty` nếu 0; `short <= Q25`, `Q25 < medium <= Q75`, `long > Q75`.
  Thiếu history/không có ngưỡng thì `unknown`; nếu không có history dương ở train,
  các history dương khi đánh giá cũng `unknown`. Ngưỡng trùng có thể làm nhóm medium rỗng.
- Cold/warm: `cold_only`, `warm_only`, `mixed`; thiếu count của bất kỳ candidate nào
  thì `unknown`. Nhóm mô tả thành phần impression, không phải hiệu quả riêng từng tin.
- Category: mỗi impression tham gia các nhóm category hiện diện; nhóm chồng lấp.
  Nhãn thật có prefix `category:`, metadata thiếu vào `unknown`. Không cộng số
  impression các nhóm category để suy ra tổng. **Mọi nhóm giữ nguyên candidate set.**

Khi PyYAML sẵn sàng, dùng `load_eval_config("configs/eval_config.yaml")` rồi truyền
`config=...` vào `evaluate`. Nếu không truyền, API dùng chính sách mặc định tương đương.
Config thiếu/thừa key hoặc thay chính sách đã duyệt sẽ bị từ chối; thay protocol cần
phê duyệt và cập nhật implementation/tests. Ghi dataset/split identity, seed và code
version trong `run_metadata` khi tích hợp mô hình; evaluator không tự tạo thông tin này.

## PHUC-03: Category và Semantic PLM

**Trạng thái:** PHUC-03 Verified: B3/B4 đã chạy thật, cache và encoder PLM đã đối chiếu;
kiểm thử numerical/mock, regression và audit temporal Passed. Xem `docs/WORK_STATUS.md` và
`outputs/logs/phuc03_validation.md` để biết bằng chứng và blocker hiện tại.

### Protocol được duyệt

- History lấy từ **chính impression**, giữ toàn bộ ID và các lần lặp, không gom từ
  impression khác và không cập nhật bằng nhãn evaluation. Giả định có trước impression;
  không có timestamp để chứng minh có trước đầu ngày. Khác với snapshot ngày của B2.
- B3 = tần suất subcategory của candidate trên các mục history có subcategory hợp lệ.
  Chỉ fallback category nếu candidate thiếu subcategory hoặc history không có
  subcategory hợp lệ; subcategory hợp lệ nhưng chưa gặp có điểm 0. Ví dụ history
  `[football, football, travel]` cho candidate football điểm `2/3`.
- B4 = cosine(candidate, mean(history embeddings)). MiniLM đóng băng, news vectors
  chuẩn hóa L2; tích lũy trung bình float64, zero-norm user (`norm <= 1e-12`) trả 0.
  Bỏ mục history không có metadata/text; không thay bằng tin khác.
- Text = title và abstract đã strip hai đầu, nối bằng `. `; trường rỗng bị bỏ.
  Model `sentence-transformers/all-MiniLM-L6-v2`, revision
  `1110a243fdf4706b3f48f1d95db1a4f5529b4d41`, 384 chiều, tối đa 256 word pieces,
  CPU, batch 64, seed 0. Không fine-tune, không dùng ID embedding của entity làm news embedding.
- User rỗng/không có profile hoặc candidate thiếu nội dung: điểm 0. Điểm 0 này có
  thể cao hơn cosine âm; không pha popularity. Ties giữ thứ tự candidate gốc.
- Cache nội dung news train/dev độc lập từng tin, không fit corpus hay dùng nhãn.
  Thiếu cache cho tin có text, sai revision/content/checksum/mapping: báo lỗi.
  News ID trùng nhưng metadata mâu thuẫn cũng báo lỗi.
- GTS/metric/cold-warm giữ nguyên PHUC-01/02; cutoff metadata click không được hiểu
  là cutoff của History B3/B4. Ngưỡng history vẫn fit train 09–13/11, dùng độ dài
  history gốc trước khi bỏ mục thiếu nội dung. Test 15/11 không dùng để tuning.

### Chạy cục bộ

Cài dependencies trong terminal có quyền mạng. Không cần PLM để chạy B3 hoặc test
numerical/cache mock. Không tải model trong pytest.

```powershell
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m pytest tests/test_content_baselines.py tests/test_temporal_leakage.py tests/test_metrics_edge_cases.py -q -p no:cacheprovider
```

Tạo cache thật một lần, rồi chạy **chỉ B4** nếu B3 đã chạy:

```powershell
.\.venv\Scripts\python.exe -m scripts.cache_plm_embeddings --output-dir data/cache/phuc03_minilm_2026-10-19
.\.venv\Scripts\python.exe -m scripts.run_baselines --content-config configs/content_baselines_config.yaml --models B4_semantic_plm --embedding-cache data/cache/phuc03_minilm_2026-10-19 --output-dir outputs/tables/phuc03_b4_mindsmall_seed0_2026-10-19 --predictions-dir outputs/predictions/phuc03_b4_mindsmall_seed0_2026-10-19
```

`--local-files-only` ở cache CLI dùng khi model đã tải đủ đúng revision trong HF cache.
Không truyền `--models` sẽ chạy cả B3/B4; B4 luôn cần `--embedding-cache`.
Muốn chỉ B3: `--models B3_category`, không cần cache. Các output directory phải mới,
không lồng nhau; không ghi đè run cũ và không rerun test để chọn cấu hình tốt hơn.

Artifact: `metrics.csv`, `manifest.json`, `<split>/<model>/{overall,groups}.csv`;
chi tiết ở prediction directory. `content_diagnostics` trong manifest đếm history
items/candidate items/impressions thiếu dữ liệu và fallback (các bộ đếm có thể chồng
lấp). Cache gồm `embeddings.npy`, `news_ids.json`, `manifest.json` với revision,
preprocessing, checksum input/code/vector, phiên bản thư viện. Cache/predictions lớn
được Git-ignore. Bằng chứng encoder mock/synthetic **không phải** kết quả PLM thật.

## PHUC-02: Random và Temporal Popularity

### Protocol đã duyệt

| Phần | Impression đánh giá | Nguồn click cho B2 |
|---|---|---|
| Train | Không báo metric train | 09–13/11/2019; dùng để fit ngưỡng history |
| Validation | 14/11 trong train chính thức | Click impression 09–13/11 |
| Test | 15/11 từ dev chính thức | Click impression 09–14/11 (train cuối) |

Cutoff `d` là 00:00 đầu ngày của impression. **B2 tích lũy** đếm click quan sát được
trước `d`; **B2-24h** đếm trong `[d - 24 giờ, d)`, không phải 24 giờ trượt đến timestamp
của impression. Cả hai không dùng click cùng ngày, click test hoặc nhãn của impression
đang chấm điểm. History chỉ cung cấp độ dài, **không tham gia bộ đếm** vì thiếu timestamp.
Đây là lựa chọn đã được duyệt cho PHUC-02, khác popularity đầy đủ trên `Ghist + clicks`.

Ví dụ: impression lúc 15/11 10:00 dùng cutoff 15/11 00:00. B2-24h chỉ đếm click ngày
14/11. Tin có click ngày 13/11 nhưng không có click ngày 14/11 có điểm B2-24h bằng 0,
nhưng vẫn được gắn warm theo count tích lũy của nguồn quan sát đã khai báo.

**B1** tạo điểm pseudorandom từ SHA-256 của `[seed, source, impression_id, news_id]`,
lấy 53 bit cao chia cho `2**53`. Điểm không phụ thuộc thứ tự xử lý batch hoặc label;
cùng seed/ID tái lập giữa các process. Seed mặc định là 0. **B2** trả count dưới dạng
score, tin chưa có click nhận 0. Tie giữ thứ tự candidate gốc theo evaluator PHUC-01.

### Chạy từ thư mục gốc

```powershell
.\.venv\Scripts\python.exe -m scripts.run_baselines --help
.\.venv\Scripts\python.exe -m scripts.run_baselines --output-dir outputs/tables/my_phuc02_run --predictions-dir outputs/predictions/my_phuc02_run
.\.venv\Scripts\python.exe -m pytest tests/test_temporal_leakage.py tests/test_metrics_edge_cases.py -v -p no:cacheprovider
```

Đường dẫn mặc định: `data/raw/train/behaviors.tsv`, `data/raw/dev/behaviors.tsv` và
`news.tsv` ở cả hai thư mục. Có thể chỉ định `--train-behaviors`, `--dev-behaviors`,
`--news PATH [PATH ...]`, `--config`, `--eval-config`, `--seed`. Dữ liệu fixture phải
gắn `--data-kind synthetic`; mặc định là `MIND-small`. Không dùng embedding files.

Runner yêu cầu **hai thư mục run mới, riêng biệt**; từ chối ghi đè evidence. Chạy
validation rồi test với cùng cấu hình; không tự tune theo test, không tự chạy E1.
Kiểm thử dùng fixture TSV tổng hợp; lệnh chạy thật mới sinh metric trên MIND.
Nếu có lỗi giữa chừng, manifest ghi `failed`, giữ artifact đã sinh để kiểm tra.

### Luồng xử lý và artifact

`mind_reader` → GTS → index click từ training partitions → label-free scoring →
ghép label vào prediction schema → evaluator PHUC-01 → CSV/provenance.

- `src/data_loader/mind_reader.py`: parse TSV strict, giữ thứ tự candidate, đếm history
  length; lỗi cột/nhãn/ID trùng kèm tên file và số dòng. Không sửa dữ liệu gốc.
- `temporal_split.py`: kiểm tra đúng khoảng ngày và nguồn train/dev; tạo train cuối
  bằng train + validation, không thay đổi các partition ban đầu.
- `ClickIndex`: mỗi positive candidate trong một impression là một sự kiện; không
  đếm lặp impression nạp trùng. Click của cùng user-news ở impression khác vẫn tính.
  Timestamp được sắp theo news; binary search thực hiện biên `< d` và `[d-24h,d)`.
- `BaseRecommender.score` chỉ nhận ID và thời gian, không nhận label. Metadata
  cold/warm luôn dùng count tích lũy, kể cả prediction của B1/B2-24h.
- Ngưỡng history lấy từ 09–13/11, giữ nguyên khi đánh giá validation/test. Train rows
  đưa vào evaluator để fit nhóm có score trung tính 0, **không được chấm hoặc báo
  thành kết quả baseline train**. Candidate objects bất biến được chia sẻ để giảm RAM.
- `outputs/tables/<run>/metrics.csv`: bảng bốn metric cho ba model × hai split;
  `<split>/<model>/{overall,groups}.csv`: bảng chi tiết tổng hợp.
- `outputs/predictions/<run>/<split>/<model>/`: `per_impression.csv` và `metadata.json`
  chứa prediction đầy đủ/provenance. Đây là artifact lớn, đã được Git bỏ qua.
- `outputs/tables/<run>/manifest.json`: trạng thái, cấu hình resolved, seed, source
  cutoff, kích thước split, hash SHA-256 input/code, Git base revision và phiên bản
  runtime. Hash code nhận diện cả thay đổi chưa commit; metadata dùng để truy lại run.

Kết quả PHUC-02 seed 0 chỉ là baseline ở cấu hình đã duyệt; chưa phải kết quả nhiều
seed hoặc kết luận về leakage E1. Cold/warm ở đây là chưa/đã có click trong nguồn
impression quan sát, không phải chưa/đã có click trong toàn bộ lịch sử của MIND.

## 📂 Cấu trúc thư mục

```text
mind-temporal-eval-baselines/
├── docs/                           # Tài liệu thiết kế và kế hoạch thực hiện
│   ├── proposal-v2.md              # Bản đề xuất nghiên cứu chi tiết (Mini Proposal v2)
│   └── project-plan.md             # Kế hoạch triển khai các tickets (PHUC-01 -> PHUC-05)
│
├── configs/                        # Cấu hình tham số và random seed phục vụ tái lập kết quả
│   ├── eval_config.yaml            # Cấu hình metric, ngưỡng chia nhóm user/news từ tập train
│   ├── baselines_config.yaml       # Tham số cho B1-B4 (seeds, cửa sổ 24h, tên model PLM)
│   └── e1_protocols.yaml           # Cấu hình 3 giao thức chia dữ liệu cho thí nghiệm E1
│
├── data/                           # Thư mục dữ liệu (bỏ qua file nặng trong Git)
│   ├── raw/                        # Chứa dữ liệu gốc MINDsmall_train và MINDsmall_dev
│   ├── splits/                     # Lưu chỉ mục (indices) của GTS split và Random split
│   └── cache/                      # Lưu PLM embeddings (.npy) và bảng tra cứu popularity
│
├── src/                            # Mã nguồn cốt lõi (Core Library dùng chung cho cả nhóm)
│   ├── data_loader/                # Đọc dữ liệu, chuẩn hóa schema và chia tập dữ liệu
│   │   ├── schema.py               # Định nghĩa schema dự đoán chuẩn theo từng impression
│   │   └── temporal_split.py       # Logic chia dữ liệu GTS, Static và Random Split 80/10/10
│   ├── evaluation/                 # Bộ công cụ đánh giá dùng chung cho mọi mô hình
│   │   ├── metrics.py              # Hàm tính AUC, MRR, nDCG@5, nDCG@10 (xử lý tie scores)
│   │   ├── evaluator.py            # Pipeline tính metric theo impression và lấy trung bình
│   │   ├── group_analyzer.py       # Phân tích kết quả theo nhóm user, cold/warm news, category
│   │   └── stats_test.py           # Kiểm định thống kê paired t-test và hiệu chỉnh Holm
│   └── baselines/                  # Cài đặt các mô hình cơ sở B1 -> B4
│       ├── base.py                 # Giao diện chung (BaseRecommender) cho các baseline
│       ├── b1_random.py            # Baseline B1: Random scoring
│       ├── b2_popularity.py        # Baseline B2: All-time (<t) & 24h-window (<t) Popularity
│       ├── b3_category.py          # Baseline B3: User-News Category/Subcategory matching
│       └── b4_semantic_plm.py      # Baseline B4: PLM Cosine Similarity (all-MiniLM-L6-v2)
│
├── scripts/                        # Các kịch bản chạy thực nghiệm từ dòng lệnh (CLI)
│   ├── cache_plm_embeddings.py     # Trích xuất và lưu cache vector PLM cho toàn bộ bài báo
│   ├── run_baselines.py            # Chạy end-to-end các mô hình B1-B4 và xuất bảng điểm
│   ├── run_e1_leakage.py           # Chạy thí nghiệm E1 đo mức độ thổi phồng metric do leakage
│   └── evaluate_predictions.py     # Công cụ độc lập để chấm điểm cho B5, B6 và THG-Rec
│
├── tests/                          # Bộ kiểm thử tự động (chạy với pytest)
│   ├── test_metrics_edge_cases.py  # Kiểm thử metric với tie scores, nhãn suy biến, dữ liệu thiếu
│   ├── test_temporal_leakage.py    # Kiểm thử đảm bảo không rò rỉ dữ liệu tương lai (>= t)
│   └── test_smoke_pipeline.py      # Kiểm thử nhanh toàn bộ pipeline trên tập dữ liệu nhỏ
│
├── outputs/                        # Kết quả đầu ra phục vụ báo cáo
│   ├── predictions/                # File dự đoán chuẩn (.parquet/.csv) của các mô hình
│   ├── tables/                     # Bảng tổng hợp kết quả (overall, group-wise, E1 leakage)
│   └── logs/                       # Log quá trình chạy và thống kê dữ liệu khuyết
│
├── requirements.txt                # Các thư viện Python cần thiết
└── README.md                       # Giới thiệu dự án và hướng dẫn sử dụng
```

## 📄 Tài liệu tham khảo nội bộ
- [Chi tiết Đề xuất Nghiên cứu (Mini Proposal v2)](docs/proposal-v2.md)
- [Kế hoạch Thực hiện Chi tiết (Project Plan)](docs/project-plan.md)
