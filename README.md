# MIND Temporal Evaluation & Baselines (`mind-temporal-eval-baselines`)

Repository này cung cấp **Bộ khung Đánh giá Chuẩn hóa theo Thời gian (Temporal Evaluation Suite)** và các **Mô hình Mốc sàn (Baselines B1–B4)** trên tập dữ liệu **MIND-small**, thuộc khuôn khổ đồ án môn Mạng Xã Hội (IS353.R11) — **Đề tài B.5: Building an Intelligent Personalized Content Feed (Nhóm 04)**.

**Bàn giao seed 0:** PHUC-01–05 đã Verified. Bắt đầu tại
[hướng dẫn bàn giao](docs/HANDOFF.md), [bảng kết quả B1–B4/E1](outputs/tables/phuc05_handoff_seed0_2026-10-20/results.md)
và [validation log cuối](outputs/logs/phuc05_validation.md). Nhiều seed, mô hình
graph và kiểm định thống kê vẫn thuộc Future.

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
PHUC-03 được duyệt (seed 0, PLM đóng băng). E1 có runner/config riêng ở mục PHUC-04;
statistical tests vẫn là placeholder. Các kết quả
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

## PHUC-04: E1 — ảnh hưởng của temporal leakage lên Popularity

**Trạng thái:** Verified ngày 2026-10-20 cho protocol seed 0 đã duyệt. Ba protocol
đã chạy thật; 8/8 metric GTS sạch khớp chính xác PHUC-02. Bằng chứng AC1–AC3 nằm
trong [validation log](outputs/logs/phuc04_validation.md).

Chạy **B2 cumulative, seed 0** với cùng evaluator/config và đầy đủ candidate mỗi
impression. Kết quả theo từng impression rồi macro-average AUC/MRR/nDCG@5/nDCG@10;
stable tie theo thứ tự candidate, tin chưa có click nhận score 0. Không tune trên
validation/test. Ngày trong bảng là tháng 11/2019.

**Ý nghĩa timestamp:** mỗi positive label đóng góp một sự kiện mang **timestamp của
impression**, không phải thời gian click thực tế (MIND không cung cấp trường đó).
Đây là proxy thời gian cho bộ đếm. Không đưa `History` vào count hoặc gán timestamp
cho từng mục History.

| Protocol | Split | Nguồn count validation | Nguồn count test | Leakage |
|---|---|---|---|---|
| `gts_clean` | Train 09–13, validation 14, test 15 | Train 09–13, `< 14/11 00:00` | Train+validation 09–14, `< 15/11 00:00` | Không có theo proxy đã duyệt |
| `random` | 80/10/10 trên tất cả impression 09–15 | Random-train, không lọc timestamp | Random-train+validation, không lọc timestamp | Nguồn train có thể chứa sự kiện tương lai; không chứa chính impression evaluation |
| `gts_static` | Giống GTS sạch | Toàn bộ 09–15 | Toàn bộ 09–15 | Cố ý dùng cả nhãn evaluation và sự kiện tương lai |

Random split sắp xếp theo digest SHA-256 của UTF-8 compact JSON
`[seed, source, impression_id]` (`ensure_ascii=False`), phá tie bằng
`(source, impression_id)`. Train nhận `floor(8N/10)`, validation nhận `floor(N/10)`,
test nhận phần còn lại. Khóa source trong CLI cố định là `MINDsmall_train` và
`MINDsmall_dev`; ID trùng giữa hai source vẫn là hai impression khác nhau.
Thứ tự đọc dòng/file không ảnh hưởng membership hoặc thứ tự kết quả split.
Không xáo candidate, không tách một impression ra nhiều tập.

Ví dụ: một candidate chỉ có positive label trong impression test ngày 15/11 có
score GTS sạch bằng 0; nhánh Static đếm positive đó. Random có thể dùng positive
ngày 15 để chấm một impression ngày 09 nếu hai impression rơi vào train/test tương ứng.

### Chạy và nghiệm thu

```powershell
.\.venv\Scripts\python.exe -m pytest tests/test_e1_leakage.py tests/test_temporal_leakage.py tests/test_metrics_edge_cases.py tests/test_content_baselines.py -q -p no:cacheprovider
.\.venv\Scripts\python.exe -m scripts.run_e1_leakage --help
.\.venv\Scripts\python.exe -m scripts.run_e1_leakage --output-dir outputs/tables/my_e1_run --predictions-dir outputs/predictions/my_e1_run --splits-dir data/splits/my_e1_run
```

Ba thư mục output phải mới, riêng biệt, không lồng nhau; runner từ chối ghi đè
artifact cũ. Không cần dependency mới hoặc embedding cache. Các flags:

- `--train-behaviors`, `--dev-behaviors`, `--news PATH [PATH ...]`: mặc định dữ liệu
  MIND-small trong `data/raw/train/` và `data/raw/dev/`.
- `--config`, `--eval-config`: mặc định `configs/e1_protocols.yaml` và
  `configs/eval_config.yaml`; thay đổi protocol chưa được duyệt sẽ bị từ chối.
- `--reference-dir`: mặc định `outputs/tables/phuc02_mindsmall_seed0_2026-10-19`.
  Cần `manifest.json` và `metrics.csv` của lượt PHUC-02 completed, cùng input
  checksum, seed, data kind và evaluation config. Dữ liệu synthetic phải có
  reference PHUC-02 tương ứng và truyền `--data-kind synthetic`.
- `--output-dir`, `--predictions-dir`, `--splits-dir`: bắt buộc, lưu các loại
  artifact dưới đây. Seed 0 cố định trong protocol hiện được duyệt.

GTS sạch được đối chiếu **cả validation và test** với B2 cumulative PHUC-02:
bốn mean/split, `n_impressions`, `n_valid`, `n_undefined`. CSV được đọc ở độ chính
xác đầy đủ; tolerance tuyệt đối `1e-12`, tương đối 0, đồng thời ghi exact equality.
Nếu khác, manifest ghi `failed` với lý do và runner dừng trước hai nhánh leaky.
Sai input/config/seed/reference status bị từ chối trước khi tạo output.
Phải giải thích sai khác trước nghiệm thu, không thay reference để làm khớp kết quả.

### Artifact và giới hạn diễn giải

- Summary: `metrics.csv` (24 dòng: ba protocol × hai split × bốn metric),
  `inflation.csv` (16 dòng), `phuc02_comparison.csv` (8 dòng), `manifest.json`
  và `<protocol>/<split>/overall.csv`.
- Predictions: `<protocol>/<split>/{per_impression.csv,metadata.json}`; giữ đầy đủ
  ID/label/score/thứ tự candidate. Các file lớn được Git-ignore.
- Splits: `<protocol>/membership.csv`, khóa source+ID và timestamp. Manifest giữ
  checksum membership, split sizes/click counts/candidate counts/time bounds;
  cả Static và GTS lưu membership riêng nhưng có cùng checksum.
- Manifest ghi resolved config, seed, input/code/config/artifact hashes, phiên bản
  runtime, nguồn count và audit overlap/nguồn tương lai. Audit pair count đếm một
  source event nhiều lần nếu nó khả dụng cho nhiều evaluation impressions;
  không có nghĩa mọi event đó thuộc một candidate đang được chấm.

`absolute_delta = mean_protocol - mean_gts_clean`;
`relative_delta_pct = 100 * absolute_delta / mean_gts_clean`.
Giữ nguyên delta âm. Mẫu số 0 → relative delta null; metric thiếu → cả delta null,
có reason; JSON dùng null, CSV dùng ô trống. Không làm tròn trước khi so sánh.

E1 báo **metric tổng thể**. Không fit ngưỡng history cho cả ba nhánh, không truyền
`prior_click_count` hoặc cutoff metadata để tránh trình bày count leaky như lịch sử
hợp lệ. Evaluator PHUC-01 và source/config PHUC-02–03 giữ nguyên. Các nhãn cohort
phụ trong per-impression output không phải kết quả phân tích nhóm E1:
history không rỗng/cold-warm là `unknown`; category vẫn mô tả đủ candidate của impression.

Static và GTS sạch so sánh cùng impression. Random đổi thành phần và kích thước
tập test, nên delta của Random phản ánh cả split composition lẫn temporal leakage.
Kết quả một seed là sơ bộ; không suy ra ý nghĩa thống kê, click time thực, hiệu quả
online hoặc ảnh hưởng trên mô hình graph. Xem [WORK_STATUS](docs/WORK_STATUS.md)
và [validation log PHUC-04](outputs/logs/phuc04_validation.md) cho nghiệm thu thực tế.

## PHUC-05: kiểm thử, tái lập và bàn giao evaluator

Đã triển khai CLI nhận score CSV từ B5/B6/THG-Rec và công cụ tổng hợp 64 dòng metric.
Adapter đối chiếu đủ impression/candidate theo thứ tự raw, tự dựng labels và metadata
nhóm sạch; không xác minh được quá trình huấn luyện của mô hình bên ngoài.
Model/seed/config/training source/code revision được ghi riêng dưới dạng khai báo.

```powershell
.\.venv\Scripts\python.exe -m pytest tests/ -q -p no:cacheprovider
.\.venv\Scripts\python.exe -m scripts.evaluate_predictions --help
.\.venv\Scripts\python.exe -m scripts.build_handoff --output-dir outputs/tables/my_handoff_new
```

CSV tối thiểu gồm `source,impression_id,candidate_ids,scores`; hai cột cuối là JSON
arrays. Mỗi file chứa đầy đủ một split validation hoặc test. Cần model-provenance
JSON và hai thư mục output mới. Hợp đồng, examples, flags và lệnh đầy đủ nằm trong
[HANDOFF.md](docs/HANDOFF.md).

Nghiệm thu: **199 tests + 42 subtests Passed**; smoke hai lần có split/score/metric
giống hệt; 8.615 score mẫu thật của B1–B4 khớp chính xác artifact cũ; CLI chấm lại B2
trên đủ validation/test khớp 8 overall metric và 176 group rows PHUC-02.
Snapshot môi trường, model revision và source identities nằm trong
[handoff manifest](outputs/tables/phuc05_handoff_seed0_2026-10-20/manifest.json).
`environment.freeze.txt` là bản ghi môi trường, không phải lockfile đa nền tảng.
Không tạo benchmark mới từ smoke, không cộng/gộp metric giữa các protocol.

## 📂 Cấu trúc thư mục

```text
mind-temporal-eval-baselines/
├── docs/                           # Tài liệu thiết kế và kế hoạch thực hiện
│   ├── SNA-Proposal-v2.md          # Bản đề xuất nghiên cứu chi tiết (Mini Proposal v2)
│   ├── HANDOFF.md                 # Hợp đồng prediction, CLI, tái lập và giới hạn bàn giao
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
│   │   ├── temporal_split.py       # GTS sạch và cutoff ngày
│   │   ├── random_split.py         # E1 Random 80/10/10 theo hash của seed + key
│   │   └── prediction_reader.py    # Score CSV đối chiếu raw, giữ đủ candidate
│   ├── evaluation/                 # Bộ công cụ đánh giá dùng chung cho mọi mô hình
│   │   ├── metrics.py              # Hàm tính AUC, MRR, nDCG@5, nDCG@10 (xử lý tie scores)
│   │   ├── evaluator.py            # Pipeline tính metric theo impression và lấy trung bình
│   │   ├── group_analyzer.py       # Phân tích kết quả theo nhóm user, cold/warm news, category
│   │   ├── handoff.py              # Tổng hợp artifact/provenance B1–B4/E1
│   │   └── stats_test.py           # Placeholder Future: paired t-test và Holm
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
│   ├── build_handoff.py            # Tổng hợp 64 metric và snapshot môi trường
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
- [Chi tiết Đề xuất Nghiên cứu (Mini Proposal v2)](docs/SNA-Proposal-v2.md)
- [Kế hoạch Thực hiện Chi tiết (Project Plan)](docs/project-plan.md)
