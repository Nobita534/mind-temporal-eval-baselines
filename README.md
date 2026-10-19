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

Đã triển khai API schema, metric, evaluator và phân nhóm. Baseline, loader TSV,
temporal split, CLI scripts và statistical tests vẫn là placeholder. Các kết quả
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
