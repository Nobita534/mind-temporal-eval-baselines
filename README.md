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