**Mini Proposal \- Version 2**

**A. Information**

* **Project / Mã và Tên đề tài:** B.5 \- Building an Intelligent Personalized Content Feed

* **Course / Môn học:** Mạng Xã Hội \- Mã lớp: IS353.R11

* **Duration / Thời gian thực hiện:** 9/2026 \- 12/2026

* **Group / Nhóm:** 04

* **Members / Thành viên:**

  1. Nguyễn Hữu Nhiệm \- 23521119 \- 23521119@gm.uit.edu.vn

  2. Trần Hồng Phúc \- 23521222 \- 23521222@gm.uit.edu.vn

  3. Lê Ngô Minh Hoàng \- 23520523 \- 23520523@gm.uit.edu.vn

  4. Điền Quốc Huy \- 23520600 \- 23520600@gm.uit.edu.vn

* **Supervisor / Giảng viên hướng dẫn:** Trần Hưng Nghiệp, nghiepth@uit.edu.vn

* **Phiên bản:** v2 (14/10/2026).

***Tóm tắt thay đổi so với v1***

| Nhận xét của giảng viên | Thay đổi trong v2 | Mục |
| :---- | :---- | :---- |
| Data MIND (news, WikiData entities, click, có timestamp) phù hợp | Giữ MIND-small. Bổ sung thống kê, chỉ rõ trường nào có/không có timestamp, kế hoạch EDA theo bucket thời gian | 3, A.2 |
| Thử 1 heterogeneous graph chứa mọi loại node/cạnh; so sánh thực nghiệm xem dạng graph nào tốt nhất | Định nghĩa đồ thị dị thể 5 loại node, 6 quan hệ (+1 tùy chọn) và 4 đồ thị đồng nhất; RQ2 và thí nghiệm E3 so sánh 6 cấu hình đồ thị trên cùng backbone | 1, 3.4, 6, E3 |
| Mô hình hóa thành link prediction; chia train/test theo timestamp để tránh leakage; survey cách chia temporal; EDA theo bucket thời gian | Phát biểu bài toán thành temporal link prediction; survey 5 cách chia dữ liệu; chọn global temporal split \+ đồ thị snapshot theo ngày; E1 đo mức thổi phồng do leakage, E6 đánh giá cuốn chiếu theo thời gian | 1, 4, E0, E1, E6 |
| NRMS và Graph-NRMS đã cũ: survey kĩ, xem có ai làm chưa; định nghĩa rõ kiến trúc và điểm mới | Survey thêm các công trình 2020-2026 (GERL, GNewsRec, DIGAT, GLORY, PromptHG, PNR-LLM, ...). Kết luận: Graph-NRMS của v1 về bản chất đã có (GERL), nên chuyển thành baseline; thêm GLORY làm baseline đồ thị mạnh; định nghĩa kiến trúc THG-Rec và điểm mới cụ thể | 2, 5, 6 |

**B. Content**

**1\. Problem Formulation**

**Project description.** Đề tài xây dựng news feed cá nhân hóa trên dữ liệu MIND. Ở mỗi lượt hiển thị (impression), hệ thống xếp hạng danh sách tin ứng viên sao cho tin user sẽ click nằm trên cùng. Nhóm phát biểu bài toán thành *temporal link prediction* trên một đồ thị dị thể gồm user, tin, thực thể và chủ đề, được dựng chỉ từ dữ liệu trước thời điểm dự đoán. Trọng tâm của v2 gồm ba việc: (i) so sánh có kiểm soát các dạng đồ thị (đồng nhất và dị thể) khi gắn vào cùng một bộ mã hóa nội dung; (ii) đánh giá theo thời gian để tránh data leakage; (iii) phân tích đồ thị giúp nhóm user và nhóm tin nào.

**Định nghĩa hình thức.** Mỗi impression là bộ (u, t, Hu, C), trong đó Hu là danh sách tin user u đã click trước thời điểm t, C \= {c1, …, cm} là các tin được hiển thị với nhãn yc ∈ {0, 1}. Gọi G\<t là đồ thị dị thể chỉ chứa các cạnh quan sát được trước t. Cần học hàm điểm s(u, c | G\<t, Hu) sao cho tin có yc \= 1 xếp trên tin có yc \= 0\. Đây là dự đoán cạnh (u, clicked, c) tại thời điểm t. Tập ứng viên là các tin thực sự được hiển thị, nên mẫu âm là mẫu âm thật (hard negatives do hệ thống gốc chọn) chứ không phải mẫu âm ngẫu nhiên.

**Assumptions.** Click là phản hồi dương ngầm; tin được hiển thị mà không click là mẫu âm (có nhiễu, vì user có thể không nhìn thấy tin). Các tin trong History xảy ra trước impression chứa nó, dù không có timestamp từng click. Metadata của tin (title, category, entity) có ngay khi tin được đăng, nên cạnh nội dung của tin mới được phép thêm vào đồ thị tại thời điểm t; cạnh click thì không.

**Constraints.** GPU giới hạn (cỡ Colab/Kaggle), khoảng 2 tháng. Thí nghiệm chính chạy trên MIND-small. **Scope:** xếp hạng offline trong từng impression; tiếng Anh; dùng title, abstract, category, subcategory, entity. **Out of scope:** A/B test online, serving real-time, nội dung body, dữ liệu tiếng Việt, fine-tune end-to-end mô hình ngôn ngữ lớn.

**Existing approaches & gap.** Mô hình content-based (NRMS, NAML, LSTUR) mã hóa từng user độc lập từ lịch sử click. Các mô hình graph-enhanced (GERL, DIGAT, GLORY) bổ sung tín hiệu cộng tác hoặc toàn cục, nhưng mỗi công trình dùng một thiết kế đồ thị cố định: đồ thị hai phía user-tin (GERL), đồ thị theo từng tin ứng viên và từng user (DIGAT), hai đồ thị đồng nhất tách rời tin-tin và thực thể-thực thể (GLORY). Gần đây PromptHG dùng đồ thị dị thể tin-thực thể. Khoảng trống: (1) theo khảo sát sơ bộ của nhóm, chưa có so sánh có kiểm soát giữa các dạng đồ thị trên cùng backbone và cùng protocol temporal trên MIND; (2) mô hình đồ thị dễ bị leakage nếu đồ thị được dựng từ toàn bộ dữ liệu, và việc chia dữ liệu không theo trục thời gian chung có thể làm thay đổi thứ hạng giữa các mô hình \[15, 16\]; (3) còn ít phân tích theo nhóm user và nhóm tin.

**Research questions & hypotheses.**

1. **RQ1 (graph vs content):** Dưới protocol temporal nghiêm ngặt, thêm tín hiệu đồ thị vào bộ mã hóa nội dung mạnh nhất có cải thiện xếp hạng không? **H1:** THG-Rec đạt nDCG@10 và AUC cao hơn mô hình content-only tốt nhất, khác biệt có ý nghĩa thống kê (p \< 0.05, 5 seed).

2. **RQ2 (dạng đồ thị):** Dạng đồ thị nào tốt nhất: các đồ thị đồng nhất (user-tin, tin-tin, tin-thực thể, user-user) hay đồ thị dị thể đầy đủ? **H2:** Đồ thị dị thể tốt hơn hoặc ngang đồ thị đồng nhất tốt nhất. Cạnh tin-thực thể và tin-subcategory đóng góp chủ yếu cho tin mới; cạnh click và co-click đóng góp cho tin đã có tương tác.

3. **RQ3 (ai được lợi):** Lợi ích có khác nhau theo độ dài lịch sử user và độ mới của tin không? **H3:** Lợi ích lớn nhất ở user có lịch sử ngắn và tin ứng viên chưa từng được click trước t.

4. **RQ4 (leakage):** Chia ngẫu nhiên, hoặc dựng đồ thị tĩnh từ toàn bộ dữ liệu, thổi phồng kết quả bao nhiêu? **H4:** Metric bị thổi phồng, và mô hình đồ thị bị thổi phồng nhiều hơn mô hình chỉ dùng nội dung, vì cạnh click của tương lai đi thẳng vào đồ thị.

**Objectives & potential contributions.**

* Pipeline tái lập được: EDA theo thời gian, dựng đồ thị snapshot, huấn luyện, đánh giá (code \+ config \+ seed).

* So sánh có kiểm soát 6 cấu hình đồ thị (G0-G5) trên cùng backbone. Đây là đóng góp chính, dạng nghiên cứu thực nghiệm.

* Mô hình THG-Rec: GNN dị thể kết hợp với bộ mã hóa nội dung qua cổng học được, dùng được cho user và tin mới (inductive).

* Đo ảnh hưởng của leakage lên mô hình đồ thị; phân tích mạng (SNA) các đồ thị suy ra: phân phối bậc, thành phần liên thông, cộng đồng.

**2\. Related Work**

| Nhóm | Công trình | Ý tưởng chính | Hạn chế / liên hệ với đề tài |
| :---- | :---- | :---- | :---- |
| Content-based | DKN \[5\]; NRMS \[2\]; NAML \[3\]; LSTUR \[4\] | CNN/attention mã hóa title (và entity, category); user \= attention trên lịch sử click | Mỗi user mã hóa độc lập, không có tín hiệu cộng tác. NRMS là baseline content chuẩn |
|  | PLM-NR \[21\] | Thay GloVe bằng mô hình ngôn ngữ huấn luyện trước làm news encoder | Mạnh hơn rõ nhưng tốn tài nguyên; nhóm dùng biến thể PLM đóng băng |
|  | Iana et al. \[14\] | User encoder đơn giản (late fusion: tích vô hướng với các tin đã click) và loss contrastive thường đủ cạnh tranh | Cảnh báo: cải thiện có thể đến từ text encoder chứ không từ kiến trúc → cần baseline late fusion |
| Graph-based | LightGCN \[6\] | Lan truyền embedding ID trên đồ thị user-item | Thuần ID, kém với tin mới (chiếm tỉ lệ lớn trong news) |
|  | GERL \[7\] | Kết hợp nội dung tin với hàng xóm bậc cao trên đồ thị user-tin | Về bản chất trùng với ý tưởng Graph-NRMS của v1 |
|  | GNewsRec \[8\] | Đồ thị dị thể user-tin-chủ đề tiềm ẩn; sở thích dài hạn (GNN) \+ ngắn hạn | Không dùng entity; chủ đề là biến ẩn |
|  | User-as-Graph \[9\] | Mỗi user là một đồ thị hành vi dị thể, pooling thành vector user | Đồ thị riêng từng user, không khai thác quan hệ giữa các user |
|  | DIGAT \[10\] | Đồ thị ngữ nghĩa cho từng tin ứng viên (truy xuất tin tương tự bằng PLM) \+ đồ thị tin-chủ đề cho lịch sử user; tương tác hai đồ thị | Đồ thị cục bộ theo từng mẫu, không có đồ thị toàn cục user-tin |
|  | GLORY \[11\] | Đồ thị toàn cục tin-tin (đọc liên tiếp) và thực thể-thực thể, mã hóa bằng GGNN, hợp nhất với biểu diễn cục bộ; code công khai | Hai đồ thị đồng nhất tách rời. Là baseline đồ thị mạnh nhất có thể tái lập |
|  | PromptHG \[12\] | LLM sinh entity từ title, nối tin qua entity chung; đồ thị dị thể tin-thực thể \+ chuỗi click; graph encoder nhẹ | Hướng gần nhất với đề tài (ECIR 2026); cần đọc bản đầy đủ để tránh trùng điểm mới |
| LLM | PNR-LLM \[13\] | LLM làm giàu title và sinh entity liên quan, cải thiện nhiều mô hình nền trên MIND | Chi phí gọi LLM; nhóm chỉ xem là phần mở rộng tùy chọn (E7) |
| Đánh giá | Ji et al. \[15\]; Gusak et al. \[16\] | Không tôn trọng trục thời gian chung gây leakage và làm đổi thứ hạng mô hình; khuyến nghị global temporal split | Cơ sở cho protocol ở mục 4 |

**Kết luận survey.** Ý tưởng “NRMS \+ GNN trên đồ thị user-tin” (Graph-NRMS ở v1) đã có từ GERL (2020) và đã được DIGAT, GLORY mở rộng. Vì vậy v2 không xem Graph-NRMS là đóng góp: nó trở thành cấu hình G1 trong thí nghiệm so sánh. GLORY được thêm làm baseline đồ thị mạnh, và điểm mới được định vị lại ở mục 6.3.

**3\. Data**

***3.1 MIND-small***

MIND là log ẩn danh của Microsoft News, thu thập từ 12/10 đến 22/11/2019 \[1, 20\]. MIND-small gồm một tập con user được lấy mẫu ngẫu nhiên, chỉ công bố train và dev (không có test có nhãn). Giấy phép: Microsoft Research License Terms, chỉ dùng cho nghiên cứu. MIND-large chỉ dùng nếu còn tài nguyên.

|  | Train | Dev (dùng làm Test) | Tổng MIND-small |
| :---- | :---- | :---- | :---- |
| Thời gian impression | 09/11 \- 14/11/2019 | 15/11/2019 | 09/11 \- 15/11/2019 |
| \# Impression | 156.965 | 73.152 | 230.117 |
| \# Click | 236.344 | 111.383 | 347.727 |
| \# User | 50.000 | 50.000 | 94.057 (user ID khác nhau) |
| \# Tin (news.tsv) | 51.282 | 42.416 | 65.238 |

*Bảng 1\. Thống kê MIND-small. Cột “Tổng” và khoảng thời gian lấy từ \[20\]; số liệu từng split là số liệu thường được báo cáo cho MIND-small (cộng lại khớp với cột Tổng). Nhóm sẽ xác nhận lại toàn bộ bằng script EDA (A.2).*

***3.2 Các trường dữ liệu và thông tin thời gian***

| File | Trường | Timestamp? | Dùng cho |
| :---- | :---- | :---- | :---- |
| behaviors.tsv | impression\_id, user\_id, time, history, impressions (tin-1/0) | Chỉ có thời điểm impression (đến giây). History là danh sách tin đã click, không có timestamp từng click | Nhãn, cạnh user-tin, chia dữ liệu theo thời gian |
| news.tsv | category, subcategory, title, abstract, url, title/abstract entities (WikidataId, confidence, vị trí) | Không có thời điểm đăng tin | Text encoder, cạnh tin-thực thể, tin-subcategory |
| entity\_embedding.vec, relation\_embedding.vec | Embedding TransE 100 chiều của thực thể và quan hệ WikiData | \- | Feature cho node thực thể. MIND không phát hành bộ ba KG, nên cạnh thực thể-thực thể lấy từ đồng xuất hiện |

Hệ quả cho thiết kế: (1) đơn vị thời gian là thời điểm impression; (2) cạnh click từ History được xem là xảy ra trước impression chứa nó; (3) không có thời điểm đăng tin, nên “tin mới” được định nghĩa là tin chưa có click nào trước t.

***3.3 EDA theo bucket thời gian***

Theo góp ý, trước khi chọn ranh giới chia dữ liệu nhóm thống kê theo từng ngày (7 bucket, 09-15/11) và theo 6 giờ: số impression, số click, CTR, số user, % user lần đầu xuất hiện, số tin ứng viên, % tin ứng viên chưa từng được click trước bucket. Ngoài ra: phân vị độ dài lịch sử (để chia nhóm user), và kiểm tra History của một user có thay đổi giữa các impression hay không. Script đã viết sẵn (*mind\_temporal\_eda.py*); kết quả sẽ điền vào Bảng A.3. Mục đích: chọn ngày validation, chọn ngưỡng nhóm user, ước lượng mức cold-start của tin và user.

***3.4 Đồ thị suy ra từ MIND***

| Quan hệ | Node | Cách dựng cạnh | Quy tắc thời gian |
| :---- | :---- | :---- | :---- |
| R1 clicked | User \- Tin | User đã click tin (History \+ click trong impression train) | History: mọi snapshot; click impression: chỉ ngày \< d |
| R2 mentions | Tin \- Thực thể | Entity trong title/abstract; trọng số \= confidence | Có từ lúc đăng tin → dùng cho mọi tin, kể cả tin ứng viên ở test |
| R3 in / sub\_of | Tin \- Subcategory \- Category | Từ metadata | Như R2 |
| R4 co-click | Tin \- Tin | Hai tin được cùng user click liên tiếp; trọng số \= số lần; giữ top-10 láng giềng | Chỉ từ dữ liệu trước t |
| R5 co-occur | Thực thể \- Thực thể | Hai entity cùng xuất hiện trong một tin; trọng số \= số lần | Như R2 |
| R6 co-click (tùy chọn) | User \- User | Jaccard tập tin đã click ≥ τ, giữ top-10 | Chỉ từ dữ liệu trước t |

**Các cấu hình đồ thị để so sánh (E3):** G0 không đồ thị (content-only); G1 user-tin (R1, tương ứng Graph-NRMS/GERL); G2 tin-tin (R4, tương ứng đồ thị tin của GLORY); G3 tin-thực thể (R2 \+ R5); G4 user-user (R6); **G5 đồ thị dị thể đầy đủ** (R1-R5, R6 tùy chọn). Mỗi đồ thị đều được phân tích SNA (phân phối bậc, mật độ, thành phần liên thông, Louvain modularity, NMI giữa cộng đồng và category).

**4\. Evaluation**

**Thực tế vs thí nghiệm.** Ngoài đời, feed tốt làm tăng CTR, thời gian đọc và tỉ lệ quay lại; các chỉ số này chỉ đo được bằng A/B test online. Offline, nhóm dùng proxy: tin user thực sự click có nằm ở đầu danh sách hay không. nDCG@10 tương ứng với phần đầu feed mà user thực sự nhìn thấy; AUC đo khả năng phân biệt click và không click.

***4.1 Survey các cách chia train/test theo thời gian***

| Cách chia | Mô tả | Leakage | Dùng trong đề tài |
| :---- | :---- | :---- | :---- |
| Random split | Chia ngẫu nhiên impression/tương tác | Cao: train chứa hành vi xảy ra sau test | Chỉ dùng trong E1 để đo mức thổi phồng |
| Leave-last-one-out | Tương tác cuối của mỗi user làm test; phổ biến nhất (77,3% bài sequential rec 2022-2024 theo \[16\]) | Vi phạm trục thời gian chung: test của user A có thể sớm hơn train của user B \[15, 16\] | Không dùng |
| Temporal theo từng user | Mỗi user: x% tương tác đầu làm train, phần sau làm test | Giảm leakage nhưng vẫn không theo trục thời gian chung | Không dùng |
| Global temporal split (GTS) | Một mốc thời gian T chung: train \< T ≤ test; validation cũng theo thời gian, retrain trên train \+ val trước khi test \[16\] | Không có leakage theo thời gian | **Protocol chính**; split chính thức của MIND đã là GTS |
| Rolling / expanding window | Nhiều mốc T liên tiếp, mỗi mốc huấn luyện trên dữ liệu trước T và test trên ngày kế tiếp | Không có leakage; tốn gấp k lần chi phí | Đánh giá độ ổn định theo thời gian (E6) |

***4.2 Protocol của nhóm***

| Giai đoạn | Impression dùng | Đồ thị dùng | Mục đích |
| :---- | :---- | :---- | :---- |
| Train (chọn tham số) | 09/11 \- 13/11 | G\<d cho impression ngày d (bản cơ bản: Ghist, xem 6.2) | Huấn luyện |
| Validation | 14/11 | G\<14/11 | Chọn hyperparameter, số epoch, early stopping |
| Train cuối | 09/11 \- 14/11 | G\<d (bản cơ bản: Ghist) | Huấn luyện lại với tham số đã chọn |
| Test | 15/11 (dev chính thức) | G\<15/11 \= History \+ click 09-14/11 | Báo cáo kết quả cuối, chỉ chạy 1 lần cho mỗi cấu hình |

Quy tắc chống leakage: (i) mọi thứ học từ dữ liệu (đếm popularity, cạnh đồ thị, chuẩn hóa, ngưỡng nhóm user) chỉ dùng dữ liệu trước mốc thời gian; (ii) đồ thị snapshot theo ngày, không dùng click cùng ngày; (iii) tin ứng viên mới ở test chỉ được nối vào đồ thị qua cạnh nội dung (R2, R3), không có cạnh click; (iv) mẫu âm lấy từ chính impression đó. Việc dựng snapshot theo ngày (thay vì từng giây) là lựa chọn bảo thủ để cài đặt đơn giản.

***4.3 Metrics***

Dùng bộ metric chính thức của MIND, tính cho từng impression rồi lấy trung bình: **AUC**, **MRR**, **nDCG@5**, **nDCG@10**. Đánh giá thêm theo nhóm: độ dài lịch sử user (ngưỡng theo phân vị từ EDA), tin ứng viên đã/chưa từng được click, category. Tùy chọn: độ đa dạng category trong top-10. Mỗi cấu hình chạy 5 seed (ablation E4 và E6 chạy 3 seed để tiết kiệm GPU), báo mean ± std; so sánh với baseline mạnh nhất bằng paired t-test trên metric từng impression, hiệu chỉnh Holm khi so nhiều cặp.

**5\. Baselines**

| Loại | ID | Baseline | Vai trò | Ưu tiên |
| :---- | :---- | :---- | :---- | :---- |
| Dummy | B1 | Random | Mức sàn | Bắt buộc |
|  | B2 | Popularity theo thời gian: số click của tin trong G\<t (toàn bộ và 24 giờ gần nhất) | Kiểm tra popularity bias | Bắt buộc |
| Simple | B3 | Category match: độ tương đồng giữa subcategory của tin và phân phối subcategory trong lịch sử user | Content đơn giản, không học | Bắt buộc |
|  | B4 | Semantic match: cosine giữa embedding PLM của tin ứng viên và trung bình embedding lịch sử | Biến thể “semantic matching”, không huấn luyện | Bắt buộc |
| Graph-only | B5 | LightGCN \[6\] trên đồ thị user-tin (tin mới không có embedding → dùng điểm B2) | Tín hiệu cộng tác thuần ID | Bắt buộc |
| Content SOTA | B6 | NRMS-GloVe \[2\] (cài đặt tham chiếu: Microsoft Recommenders) | Baseline content chuẩn | Bắt buộc |
|  | B7 | NRMS-PLM: embedding PLM đóng băng (all-MiniLM-L6-v2 \[22\]) thay cho bộ mã hóa GloVe | Text encoder hiện đại hơn | Nên có |
|  | B8 | Late fusion \[14\]: user \= trung bình embedding tin đã click, chấm điểm bằng tích vô hướng | Kiểm tra lợi ích của user encoder phức tạp | Nên có |
| Graph SOTA | B9 | GLORY \[11\] (code chính thức, dựng lại đồ thị theo quy tắc thời gian của nhóm) | Baseline đồ thị mạnh nhất | Có thì tốt |

Công bằng khi so sánh: cùng split và snapshot, cùng text encoder khi có thể, cùng ngân sách tinh chỉnh (learning rate và tối đa 1-2 tham số chính), cùng 5 seed.

**6\. Proposed Method: THG-Rec**

**Ý tưởng.** THG-Rec (Temporal Heterogeneous Graph Recommender) giữ bộ mã hóa nội dung làm xương sống và thêm một GNN dị thể chạy trên G\<t. Lý do kỳ vọng hiệu quả: (1) nội dung giúp xử lý tin mới; (2) trên đồ thị dị thể, tin mới nối được với các tin đã có click qua thực thể và subcategory chung, nên tín hiệu cộng tác lan được tới tin mới, điều mà LightGCN hay đồ thị tin-tin không làm được; (3) cổng hợp nhất cho phép mô hình quay về dùng nội dung khi đồ thị thưa.

![][image1]

*Hình 1\. Kiến trúc THG-Rec: (a) đồ thị dị thể snapshot; (b) bộ mã hóa nội dung, GNN dị thể và các cổng hợp nhất; (c) chấm điểm và hàm loss.*

***6.1 Kiến trúc chi tiết (d \= 256\)***

1. **Text encoder fT:** biến thể GloVe giống news encoder của NRMS (multi-head self-attention 16 head × 16 chiều \+ additive attention trên title) hoặc biến thể PLM đóng băng (embedding all-MiniLM-L6-v2 của “title. abstract”, tính trước, qua Linear 384 → 256). Bản chính dùng biến thể nào tốt hơn ở B6/B7, để so sánh công bằng.

2. **Feature của node xv:** tin \= \[zT(n); trung bình TransE các entity\] qua Linear; thực thể \= TransE 100 chiều qua Linear (entity thiếu embedding dùng vector học được); subcategory/category \= embedding học được; user \= trung bình zT của 50 tin gần nhất trong lịch sử (không dùng user ID, nên dùng được cho user mới).

3. **GNN dị thể theo quan hệ** (kiểu R-GCN \[17\], 2 lớp, lấy mẫu láng giềng kiểu GraphSAGE \[19\] với fanout 10/5 cho mỗi quan hệ):

hv(l+1) \= LayerNorm( hv(l) \+ ReLU( W0(l) hv(l) \+ Σr∈R (1/|N(v, r)|) Σw∈N(v, r) Wr(l) hw(l) ) )

Đầu ra lớp cuối là zG(n) cho tin và zG(u) cho user. HGT \[18\] là biến thể so sánh.

4. **Cổng hợp nhất tin:** hn \= zT(n) \+ gn ⊙ zG(n), với gn \= σ(Wg\[zT(n); zG(n)\] \+ bg).

5. **User encoder:** uloc \= MHSA \+ additive attention trên {hn : n ∈ Hu} (như NRMS); u \= uloc \+ gu ⊙ zG(u), gu là cổng tương tự.

6. **Chấm điểm và loss:** s(u, c) \= uThc. Với mỗi tin được click, lấy K \= 4 tin không click trong cùng impression; loss softmax cross-entropy: L \= −log( es+ / (es+ \+ Σk=1..K  es−,k) ). BPR và contrastive (InfoNCE với in-batch negatives) là các phương án so sánh (A5).

***6.2 Huấn luyện***

Mini-batch theo impression; lấy mẫu đồ thị con quanh các node user và tin trong batch (PyTorch Geometric HeteroData \+ NeighborLoader). Adam, early stopping theo nDCG@10 trên validation 14/11. Hyperparameter cụ thể ở A.2. Có hai mức cài đặt đồ thị, đều không gây leakage:

* **Bản cơ bản (bắt buộc):** chỉ 3 đồ thị cố định. Train dùng Ghist, dựng từ cột History (mọi click trong đó xảy ra trước 09/11); validation dùng Ghist \+ click 09-13/11; test dùng Ghist \+ click 09-14/11. Cách này đơn giản vì không phải đổi đồ thị theo từng batch.

* **Bản đầy đủ (nên có):** snapshot theo ngày G\<d; batch thuộc ngày d dùng G\<d. Tận dụng được click trong tuần train nhưng phức tạp hơn khi cài đặt.

***6.3 Điểm mới so với công trình gần nhất***

|  | Đồ thị dùng | Phạm vi đồ thị | Biểu diễn user |
| :---- | :---- | :---- | :---- |
| GERL \[7\] | User, tin (hai phía) | Toàn cục | Embedding user \+ láng giềng bậc cao trên đồ thị user-tin |
| DIGAT \[10\] | Tin, chủ đề | Cục bộ theo từng tin ứng viên và từng user | Đồ thị tin-chủ đề từ lịch sử user |
| GLORY \[11\] | 2 đồ thị đồng nhất tách rời: tin-tin và thực thể-thực thể | Toàn cục | Lịch sử click \+ embedding toàn cục của tin |
| PromptHG \[12\] | Tin, thực thể do LLM sinh (+ chuỗi click) | Toàn cục, dị thể | Từ chuỗi click (chi tiết chưa công bố đầy đủ) |
| **THG-Rec** | **1 đồ thị dị thể: user, tin, thực thể, subcategory, category; 6 quan hệ** | **Toàn cục, snapshot theo thời gian** | **Inductive (không user ID): attention lịch sử \+ node user trên đồ thị, hợp nhất qua cổng** |

Cụ thể, điểm mới gồm: **(N1)** gom mọi loại node và cạnh của MIND vào một đồ thị dị thể duy nhất (theo góp ý), có node user inductive và cổng quay về nội dung; **(N2)** so sánh có kiểm soát G0-G5 trên cùng backbone và cùng protocol để trả lời “dạng đồ thị nào tốt nhất”; **(N3)** đồ thị snapshot theo thời gian và đo riêng mức leakage của mô hình đồ thị; **(N4, tùy chọn)** thêm node chủ đề/ý định do LLM trích xuất. Nhóm không đặt mục tiêu vượt SOTA tuyệt đối (ví dụ các mô hình fine-tune PLM lớn); đóng góp chính là phân tích có kiểm soát. Vì PromptHG có hướng gần nhất, nhóm sẽ đọc bản đầy đủ khi được công bố và điều chỉnh điểm mới nếu trùng.

**7\. Expected Results**

| Phát hiện dự kiến | Thí nghiệm kiểm chứng |
| :---- | :---- |
| THG-Rec \> content-only tốt nhất (B6-B8) trên nDCG@10/AUC; ngang hoặc hơn GLORY nếu tái lập được (RQ1) | E2: so sánh chính, 5 seed, paired t-test |
| G5 (dị thể) ≥ đồ thị đồng nhất tốt nhất; G1 (Graph-NRMS) cải thiện ít so với G0 (RQ2) | E3: G0-G5 cùng backbone; E4: bỏ từng quan hệ |
| Lợi ích lớn nhất ở user lịch sử ngắn và tin chưa từng được click (RQ3) | E5: đánh giá theo nhóm user và nhóm tin |
| Random split / đồ thị tĩnh thổi phồng metric, mô hình đồ thị bị nhiều hơn (RQ4) | E1: so sánh 3 protocol |
| Thứ hạng giữa các mô hình ổn định qua các ngày | E6: expanding window 4 mốc |

**Phạm vi theo mức ưu tiên.** Thực nghiệm được chia ba mức (Bảng A.1). Riêng mức **Bắt buộc** đã đủ trả lời RQ1-RQ4 và đáp ứng các nhận xét của giảng viên. Mức **Nên có** và **Có thì tốt** chỉ làm khi mức trước đã xong và còn thời gian/GPU, nên nếu bị cắt cũng không ảnh hưởng đến kết luận chính.

**Backup plan nếu kết quả âm.**

* Đồ thị không giúp tổng thể: báo cáo như một kết quả âm có giá trị, kèm E5 để chỉ ra nhóm user/tin mà đồ thị có ích hoặc có hại.

* Đồ thị dị thể quá nặng: bỏ R6, giảm fanout, hoặc tính trước zG một lần cho mỗi snapshot rồi dùng như feature cố định.

* Không tái lập được GLORY trong thời gian cho phép: báo cáo G2 (đồ thị tin-tin kiểu GLORY trên backbone của nhóm) thay thế, và ghi rõ hạn chế.

* Không tái lập được NRMS: dùng cài đặt của Microsoft Recommenders.

**References**

\[1\] F. Wu, Y. Qiao, J.-H. Chen, C. Wu, T. Qi, J. Lian, D. Liu, X. Xie, J. Gao, W. Wu, M. Zhou. MIND: A Large-scale Dataset for News Recommendation. ACL 2020\. https\://msnews.github.io/

\[2\] C. Wu, F. Wu, S. Ge, T. Qi, Y. Huang, X. Xie. Neural News Recommendation with Multi-Head Self-Attention (NRMS). EMNLP-IJCNLP 2019\.

\[3\] C. Wu, F. Wu, M. An, J. Huang, Y. Huang, X. Xie. Neural News Recommendation with Attentive Multi-View Learning (NAML). IJCAI 2019\.

\[4\] M. An, F. Wu, C. Wu, K. Zhang, Z. Liu, X. Xie. Neural News Recommendation with Long- and Short-term User Representations (LSTUR). ACL 2019\.

\[5\] H. Wang, F. Zhang, X. Xie, M. Guo. DKN: Deep Knowledge-Aware Network for News Recommendation. WWW 2018\.

\[6\] X. He, K. Deng, X. Wang, Y. Li, Y. Zhang, M. Wang. LightGCN: Simplifying and Powering Graph Convolution Network for Recommendation. SIGIR 2020\.

\[7\] S. Ge, C. Wu, F. Wu, T. Qi, Y. Huang. Graph Enhanced Representation Learning for News Recommendation (GERL). WWW 2020\.

\[8\] L. Hu, C. Li, C. Shi, C. Yang, C. Shao. Graph Neural News Recommendation with Long-term and Short-term Interest Modeling (GNewsRec). Information Processing & Management 57(2), 2020\.

\[9\] C. Wu, F. Wu, Y. Huang, X. Xie. User-as-Graph: User Modeling with Heterogeneous Graph Pooling for News Recommendation. IJCAI 2021\.

\[10\] Z. Mao, J. Li, H. Wang, X. Zeng, K.-F. Wong. DIGAT: Modeling News Recommendation with Dual-Graph Interaction. Findings of EMNLP 2022\. arXiv:2210.05196

\[11\] B. Yang, D. Liu, T. Suzumura, R. Dong, I. Li. Going Beyond Local: Global Graph-Enhanced Personalized News Recommendations (GLORY). RecSys 2023\. arXiv:2307.06576; code: github.com/tinyrolls/GLORY

\[12\] D. Kieu, D. C. Zhang, M.-D. Nguyen, Q. Wu, M. Xu, D. D. Le. PromptHG: Prompt-Enhanced Heterogeneous Graph for Personalized News Recommendation. ECIR 2026\.

\[13\] H.-D. Kieu, Q. Wu, D. C. Zhang, M. Xu, M. D. Nguyen, D. D. Le. Enhancing News Recommendation with Hierarchical LLM Prompting (PNR-LLM). arXiv:2504.20452, 2025\.

\[14\] A. Iana, G. Glavaš, H. Paulheim. Simplifying Content-Based Neural News Recommendation: On User Modeling and Training Objectives. SIGIR 2023\. arXiv:2304.03112

\[15\] Y. Ji, A. Sun, J. Zhang, C. Li. A Critical Study on Data Leakage in Recommender System Offline Evaluation. ACM TOIS, 2023\. doi:10.1145/3569930

\[16\] D. Gusak, A. Volodkevich, A. Klenitskiy, A. Vasilev, E. Frolov. Time to Split: Exploring Data Splitting Strategies for Offline Evaluation of Sequential Recommenders. RecSys 2025\. arXiv:2507.16289

\[17\] M. Schlichtkrull, T. N. Kipf, P. Bloem, R. van den Berg, I. Titov, M. Welling. Modeling Relational Data with Graph Convolutional Networks (R-GCN). ESWC 2018\.

\[18\] Z. Hu, Y. Dong, K. Wang, Y. Sun. Heterogeneous Graph Transformer. WWW 2020\.

\[19\] W. L. Hamilton, R. Ying, J. Leskovec. Inductive Representation Learning on Large Graphs (GraphSAGE). NeurIPS 2017\.

\[20\] D. Liu, D. Greene, I. Li, X. Jiang, R. Dong. Topic-Centric Explanations for News Recommendation. arXiv:2306.07506, 2023 (thống kê MIND-small, Table 1).

\[21\] C. Wu, F. Wu, T. Qi, Y. Huang. Empowering News Recommendation with Pre-trained Language Models (PLM-NR). SIGIR 2021\.

\[22\] N. Reimers, I. Gurevych. Sentence-BERT: Sentence Embeddings using Siamese BERT-Networks. EMNLP-IJCNLP 2019\.

**Appendix**

**A.1 Timeline**

| Thời gian | Công việc | Phụ trách |
| :---- | :---- | :---- |
| 14/10 \- 20/10 | Nộp v2; tải MIND-small, chạy EDA (E0), điền Bảng A.2, A.3; dựng quan hệ R1-R5 và 3 đồ thị của bản cơ bản | Nhiệm, Huy |
| 14/10 \- 27/10 | Bộ đánh giá (AUC, MRR, nDCG, theo nhóm); B1-B4; E1 với Popularity | Phúc |
| 21/10 \- 03/11 | B5, B6 (bắt buộc); B7, B8 (nên có) | Hoàng |
| 28/10 \- 10/11 | THG-Rec bản cơ bản; E3 (G0, G1, G2, G3, G5) | Huy, Nhiệm |
| 11/11 \- 17/11 | E1, E2, E5 (bắt buộc); E4 và snapshot theo ngày (nên có) | Cả nhóm |
| 18/11 \- 24/11 | Mức có thì tốt nếu còn thời gian: GLORY, G4/R6, HGT, E6, E7 | Thành viên đã xong phần của mình |
| 18/11 \- 30/11 | Phân tích SNA, error analysis, technical report, slides, code \+ README cho pre-defense (điều chỉnh theo lịch môn học) | Cả nhóm |

**A.2 Experimental plan**

***0\. Mức ưu tiên***

| Mức | Nội dung | Trả lời được |
| :---- | :---- | :---- |
| **Bắt buộc** | E0 (EDA \+ SNA); baseline B1-B6; THG-Rec bản cơ bản (R-GCN 1-2 lớp, quan hệ R1-R5, đồ thị Ghist như 6.2); E1 (leakage), E2 (so sánh chính, 5 seed), E3 với G0, G1, G2, G3, G5; E5 (theo nhóm) | RQ1-RQ4; đủ các nhận xét của giảng viên |
| **Nên có** | Baseline B7 (NRMS-PLM), B8 (late fusion); E4 ablation A1-A5 (3 seed); đồ thị snapshot theo ngày G\<d | Cải thiện đến từ đâu; độ chắc chắn của kết luận |
| **Có thì tốt** | B9 (tái lập GLORY); quan hệ R6 và đồ thị G4 (user-user); A6 (HGT); E6 (ổn định theo thời gian); E7 (node chủ đề/ý định do LLM trích xuất) | So với SOTA đồ thị; mở rộng theo biến thể đề tài |

*Bảng A.1. Ba mức ưu tiên của thực nghiệm. Làm xong mức trên mới chuyển sang mức dưới.*

***1\. Dataset***

Dataset: MIND-small (MINDsmall\_train.zip, MINDsmall\_dev.zip, msnews.github.io), thống kê tổng ở Bảng 1\. Thống kê theo các split của protocol:

| Split | Ngày | \# Impression | \# Click | \# User | Nguồn số liệu |
| :---- | :---- | :---- | :---- | :---- | :---- |
| Train (chọn tham số) | 09-13/11 | EDA | EDA | EDA | Script EDA |
| Validation | 14/11 | EDA | EDA | EDA | Script EDA |
| Train cuối | 09-14/11 | 156.965 | 236.344 | 50.000 | Bảng 1, xác nhận lại bằng EDA |
| Test | 15/11 | 73.152 | 111.383 | 50.000 | Bảng 1, xác nhận lại bằng EDA |

*Bảng A.2. Thống kê train/validation/test. Ô “EDA” sẽ được điền sau khi chạy script; nhóm không ước lượng trước.*

| Ngày | \# Imp. | \# Click | CTR | \# User | % user mới | \# Tin ứng viên | % tin ứng viên chưa từng được click |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| 09/11 … 14/11 (train) |  |  |  |  |  |  |  |
| 15/11 (test) |  |  |  |  |  |  |  |

*Bảng A.3. Mẫu bảng EDA theo bucket ngày (mỗi ngày một dòng), sinh tự động bởi mind\_temporal\_eda.py; bảng 6 giờ xuất ra file CSV.*

**Data pipeline:**

1. Đọc behaviors.tsv/news.tsv; parse thời gian; tách impressions thành (tin, nhãn).

2. EDA theo ngày/6 giờ (E0); chọn ngưỡng nhóm user theo phân vị độ dài lịch sử chỉ từ dữ liệu train.

3. Tiền xử lý text: tokenize title/abstract (GloVe 300d, title ≤ 30 token) hoặc tính trước embedding PLM; lọc entity theo confidence.

4. Dựng quan hệ R1-R5 (R6 thuộc mức có thì tốt); tạo 3 đồ thị của bản cơ bản: Ghist, Ghist \+ click 09-13/11, Ghist \+ click 09-14/11 (bản đầy đủ: snapshot G\<d cho từng ngày); lưu dạng HeteroData.

5. Tạo mẫu huấn luyện: mỗi tin được click \+ K \= 4 tin không click cùng impression; lưu config và seed.

***2\. Evaluation protocol***

Global temporal split như mục 4.2: chọn tham số trên 14/11, huấn luyện lại trên 09-14/11, test một lần trên 15/11. Metric: AUC, MRR, nDCG@5, nDCG@10 (script chính thức của MIND), cộng thêm metric theo nhóm. 5 seed {0, 1, 2, 3, 4}; paired t-test trên metric từng impression, hiệu chỉnh Holm.

***3\. Baselines***

B1-B9 như Bảng ở mục 5\.

***4\. Proposed method và cấu hình***

| Tham số | Giá trị dự kiến | Tham số | Giá trị dự kiến |
| :---- | :---- | :---- | :---- |
| Chiều ẩn d | 256 | Learning rate (tune) | {1e-4, 3e-4, 1e-3} |
| Độ dài lịch sử | 50 tin gần nhất | Batch | 64 impression |
| K mẫu âm / click | 4 (cùng impression) | Epoch | ≤ 5, early stop theo val nDCG@10 |
| Số lớp GNN (tune) | {1, 2} | Dropout | 0.2 |
| Fanout mỗi quan hệ | \[10, 5\] | Top-k láng giềng R4, R6 | 10 |
| Thư viện | PyTorch, PyTorch Geometric, sentence-transformers, Microsoft Recommenders, NetworkX | Tái lập | Cố định seed, ghi phiên bản thư viện, lưu config YAML và kết quả CSV |

***5\. Ablation***

| ID | Biến thể | Trả lời câu hỏi | Ưu tiên |
| :---- | :---- | :---- | :---- |
| A1 | Bỏ node user / zG(u) (u \= uloc) | Biểu diễn user trên đồ thị có cần không? | Nên có |
| A2 | Bỏ lần lượt từng quan hệ: R1, R2 \+ R5, R3, R4 (và thêm/bỏ R6) | Quan hệ nào đóng góp nhiều nhất (RQ2) | Nên có |
| A3 | Bỏ cổng (cộng thẳng zT \+ zG); chỉ dùng zG | Cổng có giúp khi đồ thị thưa không? | Nên có |
| A4 | Text encoder GloVe-NRMS vs PLM đóng băng | Cải thiện đến từ đồ thị hay từ text encoder? | Nên có |
| A5 | Loss: softmax CE vs BPR vs InfoNCE | Ảnh hưởng của ranking loss | Nên có |
| A6 | GNN kiểu R-GCN vs HGT; 1 vs 2 lớp | Độ nhạy theo kiến trúc GNN | Có thì tốt |

***6\. Experiment specifications***

| ID | Mục tiêu | Biến thay đổi | Giữ cố định | Kết quả cần báo cáo | Ưu tiên |
| :---- | :---- | :---- | :---- | :---- | :---- |
| E0 | EDA \+ SNA | Bucket thời gian; đồ thị G1-G5 | \- | Bảng A.2, A.3; phân phối bậc, thành phần liên thông, modularity, NMI với category | Bắt buộc |
| E1 | RQ4: leakage | Protocol: GTS \+ snapshot / random split 80-10-10 trên 09-15/11 / GTS \+ đồ thị tĩnh dựng từ cả 09-15/11 | Mô hình: B2, B6, G1, THG-Rec; tham số | Độ chênh metric so với GTS \+ snapshot, theo từng mô hình | Bắt buộc |
| E2 | RQ1: so sánh chính | THG-Rec vs B1-B6 (bắt buộc), B7-B8 (nên có), B9 (có thì tốt) | Protocol 4.2; 5 seed | Bảng AUC/MRR/nDCG mean ± std, p-value | Bắt buộc |
| E3 | RQ2: dạng đồ thị | G0, G1, G2, G3, G5 (G4 có thì tốt, vì cần R6) | Text encoder, GNN, loss, tham số | Bảng metric theo từng đồ thị | Bắt buộc |
| E4 | Ablation | A1-A5 (A6 có thì tốt) | Như E3; 3 seed | Bảng ablation | Nên có |
| E5 | RQ3: theo nhóm | Nhóm độ dài lịch sử; tin đã/chưa từng được click; category | Mô hình tốt nhất từ E2, E3 | Metric theo nhóm, khoảng tin cậy | Bắt buộc |
| E6 | Ổn định theo thời gian | Mốc test 12/11, 13/11, 14/11, 15/11 (train trên mọi ngày trước đó) | B2, mô hình content tốt nhất, THG-Rec (B9 nếu có); 3 seed | Đường metric theo ngày; thứ hạng mô hình | Có thì tốt |
| E7 | Biến thể LLM | Thêm node Topic/Intent do LLM mã nguồn mở cỡ nhỏ trích xuất từ title | Như E3 | G5 vs G5 \+ Topic | Có thì tốt |

[image1]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAlgAAAE+CAYAAABRI24GAABzfElEQVR4XuydBdgdNdbHYXH7WJxFd1tK2WWLtFjRUgqFeqlQp2gLdXehXupeatTd3d2Furu7K1Ly9aR7hkwyc+99753r//M8v2eSk0wmk8w7+b+ZuZkbBAwGg8FgMBjMU7tBd8BgMBgMBoPBQjO/Aqvv6Lmix7CZAACQIujecfb8Rf2WEpRNm9ZXTJrUAwAAYgK6J/kzV4HFN8kVG3aJMxeuiPOXfgMAgICgewbdO/g+EqzxzWz79lXiypVz4rffLgIAQFShexHdk/j+5GaOAotuiKs37zVumgAAkFLOXfxV3lOuXr2q32p8Gt24du1aa9zcAAAgVqB7lJvIMgQW3Qj1GyQAAIRKSmay6Ial38gAACBWcRJZEFgAgIgwfvYqMXr6Mv2W42gQWACAeMKvwBo7c4WYcO0mqN8YAQDACwKZxVq0aKxYunSCcQMDAIBYhe5ZixePs93LbAKLbn54oR0AEC4CEVj0nyBeaAcAxBN0z9JnsQyBpd8QAQDAKwIVWPrNCwAAYh0ILABA1IDAAgAkKhBYAICoAYEFAEhUILAAAFEDAgsAkKhAYAEAogYEFgAgUfFMYG3bsUfccMMNht8JX/koTU3fsWufz/z6vroPRA6972IVqmOlKlUNvy+CObennnra2o8YPnKMkSfZ8Vpg7d27Q7Y1xylcuPBnRj5OK1GimOEnatasbus7omrVykY+r6HjLF++2PADAOIPzwQW3RgWLFpmxRcvXSEqVKosipcoKRYtWW7kvfHGG40yOE0dyJwE1vFT50SBgp+JkaPHWb7JU2fIfLQl2L91+25R4vMvRPOWrWxlcB4a9IoUK275K1auIurVb2jLS9Rv2Eg0a/GjFJK077GTZ620KdNmymMcOnrSVj4f46uvvxVDh4+ylXfi9HnxzbelxYhRY41jderSXZT+row4c/6yY3kcnz13oRWvUbO2KFb8c6OtVVq2aiPqN/jB2n/XngNG2YUKF5HblavXicpVq4nvvi8rzipLdXBe+twJtdvaDZutNO476p+CnxWyyneifYfO4suvvrGdv1qPosVK2PIvXb5K9k2ZsuVtfnWfwkWKyv5W053ahepIAmvX3oNGeUSDHxrLfVQfn5veD27o1yxBbab7kh2vBRa1+5IlC2xxEliHD+8TxYsXteUlP6XrZRAssFq3bmmkzZkzQ0JhEmi//nrBlr5s2UJRsmQJudXLLFPmO6O8hg3riyZNfpBhOqYqsBYunCuKFSsiTp48YvnoeHz8Nm1+tPzt2rUW5cuXEbNnTzeOAQCIPJ4KLD2uMmnKdCNNL0NNmzZzjoSEiZr36af/aZTtdDzy/dSzt6Of8//rX6ks/8/9Bhh59x86ZpR91113ye2a9ZuMNCJduhcc/USZchVkmnpcRj9/JvMHWWx+NV+qVKkd9+E8Knoegs7ZKc3J1659J0c/weel+7ksHT3PgEFDHP3q/rp/0JDhjn6ChLBTmlrWHXfc4Zqm0r1HL0e/fk4q2bLnEO+8+57hBybhEFh6/M4777T1nZ4+duwooxwWWCRYFiyYI1H30XFLc/MfOrTX0U+wwNL9zz77rPSfO3fSOIaed9++ncY5AQAiiycCa96CJfKPWvcTNJvBf/Tse/6//3XNr98o9H3V+C9rNxhpelkffpjVMZ3C9AjHqVw1vn3nXrl95ZVXpZ8f+5DAooGcwqvWrDfKUPd3S6Pwuo1bZHjg4KHi9ttvl2GeMXLbh+OqwPL3yIvy3HPPPba4LrD0fQi9//S8bml79h92LZP89957r6Of9/n4k2yO+6e0Pk7tQn6eQVXzd/upl8+ynOrjhJ4vJfsmG14KLJq5onZWfdz2anzIkIG2+FtvvWmU5fSI0KnMevXqyPDq1StE7do1ZbhKlUq2sl5//TXX/WnLwunKlfMyrgosFnbqPiyw1Nk1ir/wQjrbcQEA0cUTgbV+01b5B676KP7a62+I8ROnWDcHTmORopfD+6lp+iNCCt98883i/gcesFDT9LLuvvtuW96x4ye55tWPQ/QfOFhu6ZEW+atVv34TJYH15ltvy7BaPtfHrTy3c/g0X36/++hpLLC2bNslBQPn4VkXFfLTY0w17iaw6JEmxZ9Nm9boPz2vvzS9Hsyjj/7Dys/CT92fhBGFSVDlyJlLhl966eUU1cetXSjMwkvNz8dxKks/ji+c8jn53KC8g4eOMPyJiJcCa8uW6/9wqT6KZ8v2sS1Oj+/UeI4c2Y2yfD0i5GuBwiTWKEyP7J577jnj+Hp+PU7bDh3a2tJUgUX/iKiQnwWWeoyLF89Ys+vEG2+8btQDABBZPBFYBP1Rc5hnZdQ0X3G9HDXNSWCpcXrXS03r2Lmba96+/Qfa0jis5j164oyYPmuuDOsD/+lzl6wwCaydu/fLMD3y43K+/qaU47HVuJ62ZNlKuSUBRP4vvvzayMdhes+L6kVhFlinzl60HYdn21R4f3p37D/PPy/DbgLr1ltvlXF6Z2jT1usvDev1oDC1lVsax/V6EHy++j5qWBVYqn/Zil8c9yEhvPfAEVuaW7tQ2ElgUdtSeOLkaUYahw8cPm6V6QbNRNapW9/m43JUPsmWXfr1d9Wc8iYqXgosgtpOj7NvwoQxMnz69DEZP3bsoIxv2rTOKIcFVq5cOaXIInr06GqUqQosFnicRkKItl27drIdl8K33XabUVbp0tf/sVEF1uOPP2bVqXLlCla5vA+zf/8uK6yWCQCIHp4KrMZNm9viOmpa6meeMcpQ9+O4LrDovSi3cm+66SbD55ZXDfvL26FjF8vXuk17uSXhQWn58hdw3E8vQ43TIO20D6G/G0SijvwsWgl6CZ62KXkHi17M5vSmzVvK7cDBw4y6EaqQfOyxx23p+rEIfnlbL0cNq+j7L1x8/ccR6v6qwFKFHM2qqfk4/MCDDzoeX0X1OwksX/uwmNb9bujlvPteJiOdfjih70e8n/kDw5eohENgtWzZzBbnl9mJBx54wEp78sknpE8vg3B6RPj0009ZZfJ+qsCieJ48uW37qPVw8r/99luW791335FbFlj0crvTfk4CS89HL/Xr5wQAiCyeCSxf72HpBJovVqD3iTjMNzA9T6zDL+UTGzZvC/ocYu38Y60+KaV2nXpG/ekXk3q+RMVrgbVo0TxDfLhB+fAoDQAQLjwTWImM+h6PPhjGC/TOl3oO6hIQKSHW2iDW6gNShtcCCwAAYgUILABA1IDAAgAkKhBYAICoAYEFAEhUILAAAFEDAgsAkKhAYAEAogYEFgAgUYmowOrU5a81qtygn+zz0gTJiPptv1hm+crVhi8YAvmuX6Cs37hVlP7ue8MPYhcILABAohI2gZUhQwbD99prrxk+J7786vpCm5HCqa7RIFv27LZ4wYKfGXkYqrNX9a5U2fycjC/WrPtryQd/VK1W3Qq71Vf103pfTh9Fps8HqefcsFFjK61Y8RK2RWQDpd+AQYYvEqh1Twm0vIbui3cgsAAAiYpnAmvTlh1y8Ctd+voMwpdffiVy5splGzzJp+9H6YUKFRYff/KJ5Xvr7bdt+2X9+GPx7nvvSSj+zjvviD59+xtl0cBM+33wvw8kb9yyXcbfffddKw8dq3DhIqLgZ4Us36uvvipeeeUVkSt3bqNMPq+OnbrI8yHfJ59ks9LHT5wst4WLFJX5GjdtJuPTZsySW/ItXfGLUW6t2nVkWoP/LTbZtl0HGW/WoqWM0zm///77cqvvS/loBonb6OtvvrXSaBCmOtGq5pReoWIlUb5CRZlWo2Yt6eO4Wh4dp137jnLbqnVb6Rs9doItDx8vy4cfiqxZr3/jkcUQpb3xxhu2cqksmfd/5/DNtXpSvjJly1l5ypWvYJXbum17W78wXB91ZpPy0pbamdJYlNPq7YuWLJe+VavXyzWluHwVmimkBV+37tgtdu45YJXB5W3fdf0bklR3WiVeLYN8eT/91BLA+rlXqlxF+vLkySvjdE1QnNqf96ctLZVBfrreeV+6tuhazJ4jh1HnN998U+677+ARIy1egcACACQqnggsGshocFZ9H374kdzSp1mmTp8lTp65IGbPXWDL86oyo8UDmDqQ8awE+35o1EQUKVrMyMeoA5VOv/7XZyt4IOTBj6ABjbZOApCEEG1JJGXJcl249ezVR267dO1u5M+T93q5JCz4Mz56XflTLE7wLFa9+g2MNLWsPj/3ExUrVRYnTp8X8xcuseWh1c9ZRGXLdr08Pkcn1Pqp4bfeesvw85bEwTffXv8skH5+TmzbudcSZXpZbj7m4JHrn6fh/ectWHztvM/J8KL/tTE/smzUuKno2r2HURb3P1Pi88/ltlTp78S4CddFsr/65M6dx/Cp4cyZP7A+P8SsXL3O9skegoQzbflvRIXLK1PmLxGqpyUSEFgAgETFE4Hl68b/Y6s2csvCSEX15ciR00gfOXqsFAtz5i+UcZpp4jT9mHqcfUdPnJblUPz7MmWN/PXqNzR8zKw5860wfSqFZjtI5On5Z86eJwdpmmFxGpjVGTSm4Q+NrDz0HlLZcuVd92fadehkhYeNGCW3ahuy4FP3LVrsenqLlq2M8hj1EeGYcROssFoOCTWaGdq976Dla9qshSWynGje4kcrrIoJLtepP15//XVbGWqf8zXCPhKz1H+MWg5BIlctS8WpndXwF/8T23SO7OP6suimzzZR2/HxaRaS06mszdt2yji1A5e9ZfsusXnrTmO2Tz+WU/87+eIdCCwAQKISFoHV7X8zCGqanofgGSx63DJzzjwZ5kczk6dOl1t1JkctQ323Ry2LIaFAs2fqfk6DqZOP4Tr5y89bfkRJYVUYjBo9zlauDu/D7xnpx2FIYNGgrqb7qo8apveU9PKY1es2Gj51X350RgJz0pTp0vf22+/ILYkOqre+r7q/W1gVXSRgaUuP49QyVCGin5/T40ROoxk2foHeqX56WWp4194DYu36TWLKtJmWgOTZM5otpG8jUvjwsVM+3+Mi0az7+BG6+g8FPbamrVNd/JUX70BgAQASFU8EFj2uoQGBB0OngcJpwBg+cowoXuJz0UT5SDS9P0V5+TGaU1kECw0VStfz06/KnOrA7+yoPn68p5dJ72ZxviPHT8swvffEj/NIHJCPfsVG72KRjx7h6WUx9C4N5efHd/T4jOL0vtfnJb+Qvt59+jm2GfkoD6fRbAiFa9SsbZx7zpzX3xkjSKiSj46tl0mDPaWxuGU+/7yk/GFCyx9bWz5qI8qripY3MmaUPmobdX96fMl1UuvWstX18tp37GS1JcV79elr25+hPB07d3Usi8JE4yZNZZze86KtOnvGs6gqfE40k8g+LoPeW6Mtve9Ej7+pfHpfi3z0z4BaDj3Ko3R6r4/i9DdAcc5Hs5cU5/cCnep+6OgJI83p3Ts6N3pnTffHMxBYAIBExROBFQo0cAX668KUsuKXtYYvWNTBLx7Qf5EIUk689Xk8AoEFAEhUoi6wAADJCwQWACBRgcACAATN1at/hsTpsxfUW46jnT9/Wvz551W/6Dc3AACIJhBYAICgYaG0des2kSpVKlGzZk25bdKkiSGmnIDAAgAkKhBYAICgIZE0c+Ys0bJlS0M8BYKTwMqfP784e/asDJNYcxJYZ8+escKUBwILABBrQGABAIKGRBIJHBZMmzdvkehCyg0ngUVGZS5ZskSGnQQWi6qLFy+IHj1+gsACAMQcEFgAgKDRBRaROnVqQ0jp0IKyvgQWGZVL5ktgPfPMM5ZPv7kBAEA0gcACAAQNiSRaVHfduvWWeFIF1++//yHSpUsndu3abRNY5GOBNWXKFPW2I43SyUaMGOEosIhChQrJBWAhsAAAsQgEFgAgaFgwde3aVTz77LOiefMWxmxVyZIlxaBBg22+kSNHWQJrz5496m1HVKpUyRZ2E1gVK1a0xfWbGwAARBMILABA0OhiyheLFi2ywmnTphXHjh0Xx06cVm85juYmsHT0mxsAAEQTCCwAQNDoIiql+HoHi61Vq5bi888/t2jbtq1NWJ07d1bs379fZMuWDSQBVapUNgayWIDqpdcVJC6BXIcQWACAqBHKSu50k/v000/17LAkMHo3j/pfvyaiAdWD6gNLPqP7j6/rEAILABA1ghVYNWpUF/Xq1dOzwpLIfvvtN1GzZg3j2ogkuA5h1P9u1yEEFgAgagQrsOi/RhjM1+xBJMB1CCNzuw4hsAAAUSNYgUW/TITB6J28kyePGNdHJDhx4giuQ5g0t+sQAgsAEDWCFVgDBgzQs8GS0Pr37y/Gjx9jXB+RYNy40bgOYdLcrkMILABA1AhWYI0cOVLPBktCo+tg2LDBxvURCei4uA5hZG7XIQQWACBqQGDBQjG3gS0SQGDB2NyuQwgsAEDUgMCChWJuA1skgMCCsbldhxBYAICoAYEFC8XcBrZIAIEVGfv999/F5cuXdXdMmdt1CIEFAIgaEFiwUMxtYIsE0RZYX331lbjhhhtsBLLg6YoVKxw/sB6LRud0/Phx3S3PUz93ao9omdt1CIEFAIga4RZYR44cEYcOHRJ33323aNiwoZ5sGd2g3Yx+gq3asWPHbPFYtuzZs+suV/PVBim1w4cPy/KuXLki4//4xz+sNPJTv5A1btxYbN682fKXLl3ayhdI3d0GtkgQbYHF1qBBA7FkyRIZrlatmmzHf/3rXzK+dOlSkTVrVhkeNGiQqFy5sk2U6PbAAw9I/3333Wf56PrX8y9cuNDwPfLIIzLesWNHy3fjjTdK37p16yzfiRMnjH0pzMdmo1krPZ9udN50/tE2t+sQAgsAEDXCLbDo8QIb36idbtwc3r17t5V28OBBK03N/9lnn9n208vp0aOH3NLg5mS8z+TJk6042y233CK3W7ZskduhQ4da+a9evSp9VP+NGzda++XIkUOGb7rppuuFKEYi5ddff5Xpb731luWnT3yQjwZAMv0cVR8bDdQ82Krp5cuXt/KwqfvpppahCyx1PwiswIwF1rZt20StWrWkb+/evaJFixYy/OSTT8ott63bDJb6GE4N8370t/HDDz/IsH6t0d/EmjVrZPiuu+6y/HTtkamiS7/GdJ9qbn42CCwAAHAhUgLr+eeftwacQG/w7NNnsFhguZXz559/GulOxukfffSR3LLwI1NnENg4jfJNmzbN8quzAydPnrTCZCRS6tSpI8PqwKeKPxZ1TuejhnkmhGz06NFW+PXXX7fCbL7OndqzZs2acnDUBdbZs2ctIQiBFZixwHrmmWdkGzI333yzlUftDzeBlT9/ftv+GzZskH71+v/ggw/kdv78+ZaPTC+f/sngvzfV6O9RPQbv53a9uPnZILBCpEix4nL7yquvGWnE4KEjDJ8KdZDu8wXdhHRfMOUkItwG4ydOMdL0PG7cd//9hi8Y6D8o3Qfij0gJLHpEtX//fhkmQUF/5wwZ38i7dOkiHn/8cesdD7KUCiwnH0F12bFjh5w9GjZsmEiTJo2VN3369OJvf/ubmDBhgm3ft99+W2TMmNFWH3V2YezYsbZzqVChgpVGpoqUUqVKye3MmTNt++htwGE9XRVYPPPlZmpZZEWLFhXnz5+XYW5PyqMLLLK0adPKLQRWYMYCq1OnTmL8+PF6snx0R7Oft912m4yvX79eXje6LViwQHdJcxJY6dKls3xkTz31lPWPRebMmcWlS5dsj9L37Nljhek6102/Xtjc/GxJL7AWLl4mPsr6ic2XLXsOcfbCFSMvMWLUWPFJtuxi87adMs4Ca+jwUVaelq3aiAqVKsuwKrA6delulLd9514r/EPjpqJQ4SK2/MdOnrXVjy5C8h88csJWzuclvzTKjkeoPctVqGTF69StLzp27mbkY/IXKChq16knw3Sx03bbjj1W+uFjp2R/7j90zJbn5JkLolefvkZ5lF+NU1svWLTMOgaT99N8onuPXlYeNT9t6Y+UwkdPnDGOkYh07vqTyJP3Uyu+YfM227Ws06RZC/FZocI2H/VzmbLlrThd+zlz5RZ7DxyRcd6qjB0/SfzYuq3h94pICSwyJxHE5pTGYRYmbCkVWKrxbBGZnpfjhQoVErly5XLMQ6b/ouqPP/6wxVVzElhk/OhGNadjqaYKrO+++84Kd+/e3Qqz0f7nzp2z4k4CiwZkml3TBRZZs2bNILACNBZYZLfffrv13hMJrnnz5ol8+fLJtNq1a4vevXvLsHq9qUa+e++915bmJLBeeOEFmUefJaPrm/ZXfU888YT45JNPLB+9C0n/INM9nI/jVBcyNz9bUgusEp9/IQUThVnoUOfTljpCz3/q7EU5cFCYbv601WewqMHVfVhgUUefOX/Zlnb81DkrvOKXtZao4zJoO3vuQpvPbQaLqFW7ruGLJ/S24/jufYeu/QfydMD5eQZr7YbN4v3MH8jwrj0HrDyLl64QL7+c3ihv38Gjho/LnD5rrmjfobPNd+joSSm2VV+3n66LrmSawaIbEf1tcPyXtRsssaX3EUF/Q/y3wOn0d6f+fWzdvlu8kfFNGd6z/7DcLlqy3CjLqXwviaTAyp07t9zWq1dPnhe9+E5bMt727dtXhhky+u9fjbPAopmAO++8U/qHDx9uK0cPs+3bt0/ccccdMk19tEbxGjVqWGE2enxI17p6fF1gkZ9eEKZ7qnq+ZG4Ci/a5//77xa233mqVqx5j+fLlMvzoo49aPlVgkZH/scceczxPMh5A1XLJ1AGb/E4Ci8IQWLB4MbfrMKwCi/5I1HiHjl3EidPnrTjNnvAfH/3H/c9//ssoIxCBpfuYqTNmW2E1z7elvjN8HPYlsDK9n9nwxQs0A8jilXE6f+4PCqd59lnH/CywnNqdfPpx9P3dfGofqzjlTSaBpbebGv++TDnLx376Z4PTK1WpauzjFCecBFbWj7MZPi8Jt8CCJba5DWyRAAIrcqaPCUQsmdt1GFGBdeDwcVG3XgMZbtO2g1i3cYstnR7h6WUEIrCc/Drq4xV6DKjvw2F6uVTfl6DZF3/ve8U6BT8rZIs7nb9buhpngZUjZy7jUS/lef6//5WPhvXyHnzoIcOnHsOtj9mnCjeaBdDzJCp6e9Bj23MXf5VheiSg5y9arIQVplkapzLo8eLpc5dsPn6kS4932QeBBYtlcxvYIgEEFozN7ToMq8Ai6Mb+8MOPWO/90MBIz2jV/7Kd8vOAoAssejxCaZyuih59ECF69v7Zlk7T1vTOEMepHlSfKdNnSR899iL/mvWbRMNGTax9n376n0bZ8QYJH34+T3ESLNyW+uNVYsy4ifKxA+fnrfqSO/n+/ve/G3neefc9MW7CZCsftSc/9lVR+4z7mGYeaYaK6vrAgw+KRx/9h/F4lx75UnjLtl1GmYkGiXs61/sfeMDyUZzayE0AcXrmD7LIOD0G5r5W85DoVfuO/h7U97TcyvcKCCxYKOY2sEUCCCwYm9t1GHaBFW3UAUXHV5rOI488avhA4CTTIz0QOOEUWPSyeN26dXW3NPplVawb3Z98Gb+zpZr6gnGs2Ysvvii3q1evdlwmIBhzG9giQUoFFi+2umrVKtv7d+XKlbPC5Cf69etn+eiFf30pDDXerl07ea28+eabrj94oHXT+J049Rd9Tqukk9GvXXXT6+Bkbtcs+SN5bXI75syZ0+bTTfdRnN4/JKP3CwM1t+sw4QWWL/iXbwCA6BBOgcXmdNOPB4Hlz+JVYHlpbgNbJEiJwCLhU716dRmeM2eO7asC/Ms8Mr5WaQV8XgqDPgFD646dOXNGxh988EGrn+mHG6dPn76+s4t17drVWqRWN1Vg6eJMXU6Bli/hXyCS6XnZnP7WaFkQ/pUjGy2E6hR2M/rFKX8VQDVeDFg3tR4cdqpbmTJlrF/E9unTR67fNWPGDBmvVKmSa7vp5nYdJrXAAgBEl5QIrGrVqlo3rkAHNjKnGysLLFqdumrVqvJmyvno5+1sFy9etMKUTgOdWh6tok6rW1N9OL1s2bJWupPROlitWrWyfT6GVtumXyfSf9D0c3oyPg4NZlmyZJGrYdOWB9SPP/7Y+jQKG71zR0si0PIH/BP6Xr16yXdL6ef6fEzah6CBU7WSJUvKZRcIbgeaCeTzY6Mwzaw89NBDctFSykvHu3DhgpVOdf3xxx9Fhw4dpM9pBovWAitRooT48ssvrc/nFCxYUHTr1k2u/+XUd6q5DWyRICUC65VXXrHCJLB+/vln69zcBBb3HwksXqCT87DAatq0qVwywZf5akMWWJSHFq/NkCGD7Ff2sXGYllsgQUKCyalcJx/NxtE6bvS3xNcd/XKUFpXlFeF5P/qVLS1USqjnS2t00Ur19Atg9tG5b9261fGYTnV3ykcCS02nfmGBRabPcLmZ23UIgQUAiBopEVjZsmWTkNAKdGCj9wPppq2b0wyWegOmG63qU/PTYEcCiYyWdWDLlCmTFfZlAwcOtMI0kJDRjxXY9AFBrRedPwssEjBsNFCQqXnpfVfd51Smav4WEGVzKpOMV2F3SncSWOq6YKlTp5Zbp33dzG1giwQpEVjqebDAIlFNMyS6wCJo+Qw2Fljko+uF/ilQZypp0Vrez8n09lTjLLDUT99wOolcflTJs1nqvuoaWGxOdSDBwjNYel3YSCypXyRQ02iFed3vVo7qY1jgO+Wjvxta7JYsT548hsBy2sfJ3K5DCCwAQMRZunyVGDRkmKhWr7m8OfmiceNq8ubFAosIZGCjT4e4GQsmEmBsTjftl156yUhTjT5qrBq93+KWl43XfSLjm7k6w8HmNJjQbBsLLPURIc1akal533nnHcPnNFA62csvvyxnkGg2jQZ03Zzaiow+SaT7OOwksNTHhrSWF5nTvm7mNrBFglAFFvt1gUVGM1jcHiyw1HS3R8FO7aV/N5DLJWOBpfr09udvDOppTuaUHojAIiHHfwuq/7fffhPffPONFWdzKyelPv0fEwgshX0Hr688Tau+j5swyYqD6EDtT/0weMhw9EcMEA/9kdIZrFWrlsuwv4GNPuWhviisGwss9T0T9WZKnxLhGSAymjl44403rDh9CoRMF1hk/B8zrWRevHhxLdVZYKnHVj8yTUYfheZV0MkXqMDiMA2w6qM7PZ9q/P05MmqbL774Qmzfvl3GWTyROR2HzElgcRsHKrB4ZoS+q+hWTza3gS0SpERgDR482HrXSBVYy5Ytc21LDqsCi40FFi3+eurUKRn+8MMPRYECBdRslqkzpGpYfURIRrNh9JiQjT57o9bpnnvusb5DSP2om1N/uQmsWbNmWY/g2N+yZUsrnY3S+DM8+uNzPezPR4v0MmQssNhUgbVz504xZMgQW7qbuV2HcSWwaAkF2vJ/sd9+W0ocP3lWrN+4RXTr3kPGyU95ChQsKHopSzQA76H2VWcVqP2pHwYOGmrrDwb9EV7isT9SIrBU/A1sdONU0a1WrVpWmEQBvSuk59Nv0vSOEL3jpK5Ern4Ohl44pvSePXvKONWRPlWiG79rRLZlyxYrTO8e0WOyTZs2ybhaH3r/igY/td7jxo2zwiwm6Z2sKlWqGL+AonecVIGonyvbokWLpNBRH8vQavLPPvusksu+vxqmRz1k1HYUpne02Jo3by63Bw4csM6RfWSqYKQXwukdHb0PdHMb2CJBSgQWGZ8LCWRVmJPgcgrT7A0Je7pGWGCwqd8NpO9X0jWpfvvPyerXry8qVqxo89Ex2Jo0aWJ8xJlMrRMZCavvv//eEu2qOfUXCUDqSzK9rBMnTkgRz7Z48WKZh2Gjl8+//fZbK+7WZv58etn0Xpdq1C8sWJ3Oxc3crsOYF1j0nzcNBp06dzXSAoEU/e69Bw0/CI7hI0aH3B+0P5Wjp4GUE+/9ES6BFaql5OYaCaPHneqHl2PdQmk/emGeX3Dv37+/nmwzt4EtEqRUYCWDhdLv8Wxu12HMCyyvWLFqjciZK5fhB4FDA3HxEn+tEh4KVA76IzQSoT9iVWDB4sPcBrZIAIEFY3O7DmNSYNFnQGjw4M+BeMXIUWPDUm6ig/6ILRKpPyCwYKGY28AWCSCwYGxu12HMCSz6GHQ4b/LhKjdRiUR/UPm6HziTaP0BgeXbeFVpJ/NqNXRf5rSiN1skju/P3Aa2SOC1wBozZozukua2mKZXRr98/fXXX3V3Qlm4r2O36zDmBFakiOQgEs9Eqp0idZx4J1LtFKnjJLvAmj17tm09IfWF30DeZ+FlJEIx9Tjqy8FFihSxwm5WoUIF3RVRcxvYIoEXAiuQPqZFOlNitGBtSox+lOH2yRzVAqmrLytVqpTuCovp9dTjThbqdex2HcaUwIrUTZ1Yu35TRI8Xj0SyfdAf/olk+0SqPxJFYKm/umOjX3bRzZ1xMjc/2W233WaFaakFLod+8cjma38yf8f3tbCouk8g4WiY28AWCUIVWHrfqFvVzwKLfvHHfvr1nZv56xMug35xSkYCi2Z4yKcvOMrQoqMcpuvy6aefFhMnTjTqTJ/xYaNP+7CflvpQy/Nl6q8FVVP397V8gl7++++/b4VpzTEuQ/1nQt8npeZ2HcaMwDp28mxEbugqdDxa5kH3A/RHrJGo/ZEoAouWTHBaE4tNXdpBNV83dvqIL5uazy3sy9zyufnJ3I7jFo6GuQ1skSBUgUUWSFuywAokL5mvNNV4fTISWOnTp5fhuXPnihYtWqjZLFPLJYHltBRC69atre8U6vUIdAaLxBgtd+Jmerm66enqel1ubajvk1Jzuw5jRmBFevAAvolWf0TruLFOtNol3MeNV4FFYkrH6btltKChurCjbvqNnRYM5UVFeTFKMn1mge3OO++0wk5GgxV/gsbJ9OOr5jYAuYWjYW4DWySIhsCi64PRja9Dysdh3egzTbToKb1zRd+BJCOBpb7nxfUoV66cDLPYUetHAks1uj73798vFxTlhTofffRRWx43gaX/HfE5OJmbXzU9j/olAqdV8/VwMOZ2HcaMwIoW4R5A4pVotUu0jhvrRKtdwn3ceBVYurndoN38bDR4rVq1yoqrAktdUdtNYPkr31/6pEmTjEVE2dyO4xaOhrkNbJEgGgIrEPOVz+l4JLBoxXaytWvXirp161p5yDifuq8qsGgR0F27dslw2rRpHb9OQEYL4AZi+n5sga4Bp++vHjcpBdbWHbtFg4aNDH8k+L5MWbFt5x7Dn8xQf5QpW9bwRwL0h0ki90eiCCwnowGBbtyMm9FK8pzH6aPPZIEILP19KloFPJDj06NIp3zqp4L69Okj0x5++GFbnvvvv98KR8PcBrZI4IXAuu+++6z21PuV4yywLl686NhPuvlKU8sgMURGAou+N6iXy/HHH39cxtu2bSvj/A6WapyXVphngcXf5ST4iwX6MVJivC+Rknew1LibwAr1Ona7DmNCYNGnPOjDr7o/EgwcPFSi+5MZ9Edskcj9kcgCywtz+jYbG30Xjo0/UeO10bfy3EwfyKJhbgNbJPBCYMHCb/RNy3Bfx27XYUwIrHkLFovmLVoa/khw4PAxfEpHI5r90ax5SzF/4WLDn8wkcn8kk8BSfwkYyn/ywZp+7EgfPxzmNrBFAgis6FosXctu12FMCKzTZy+KyVOnG34QHag/ovHZFIKOS8fX/clMIvdHMgksmPfmNrBFAggsGJvbdRgTAiuahPsl3nglWu0SrePGOtFql3AfFwILFoq5DWyRAAILxuZ2HcaMwPq2VGkxdXrkjkdMmTZDlCpd2vAD9Eeskaj9AYEFC8XcBrZIAIEFY3O7DmNGYBHh/m9ZJ9LHizci3T6RPl68Een2icTxghVYw4YN07PBktCGDh16bXAbZlwfkWDEiKG4DmHS3K7DmBJYAIDkIliBVbt2bT0bLAmtVq1aYsOGNcb1EQnouLgOYWRu12FcCix66faXNRvki/HDRoySP2GnLcU3bd0R1pdygQn6I7aIp/4IVmDR7BoMRteBfm1EElyHMDK36zBmBdaceQvlB1Sp4jrFChcU9auXF91aNxSDuv8ohvVqI7cUr1ahtEzX9yHKlisvy9WPBfzjqz8KFS4qKteoK5q37SI69hgguvQeIrcUL1uxqkzX90F/hIbP/ihSSFStXUX82LGF6NKnk/ipf1e5pXj5yuVkur5PtPojFIF1/PhxPSssyYyuA/3aiCS4DmHHjh1zvQ5jRmANHTbSdrNvUreyuLxrgfhj/xLP2LNigixXPQ4dV68LMPujRr1GYt3ec2LTocueMXfVdlku+sM/en/UblBTbDm2Uew8vdUzFm2cL8uNZH8EK7AwuMGo//VrIhpQPWDJab7EFRE1gdWtew/rJj5rVA9DDEUSOj7Xheql1zUZUPtj4JjphhiKJHR89Mdf/TFs4mBDDEUSOn64+iMUgUXMnj3DqhtIHipVqmBcC9GE6qPXESQ+dP/RrwWViAssegxBFaPHebrQiQWoXlS/cuUrGHVPRLg/6HGeLnRiAapXMvYHPc7ThU4sQPXysj9CFVjJDPWD7otFfv31gqzr/PlzjDQQ/8TLdRgNIiawcufOLTsi2rNVgcKzWp06dzXOJRHg/oj2bFWg8KxWovdHtGerAoVntULtDwis4Im3gY3qu337ZsMP4pt4uw4jSdgFVvbs2WUH6AImnqD603no5xaPcH/oAiaeSMT+0AVMPBFKf0BgBU88Dmzt2rWJy3oDd9Cf7oRNYK1as142/OYFowzBEo/QedD56OcZL3B/TF203hAs8QidRyL0x+yVMwzBEo/QeQTTHxBYwROvA9vRowdk3Y8dO2SkgfgjXq/DSBAWgbVz937Z6LpISQTkbMOe/cY5xzLcH7pISQTiuT90kZIIpLQ/ILCCJ94HNqr/xYtnDD+IL+L9Ogwnngus2XMXJKy4Yuj8Ir1eULBwf+jCJJGIx/7QhUkikZL+gMAKnkQY2OgcJkwYa/hB/JAI12G48FRgfV+mrChRtJAhSBIROs/jJ88abRBLUH8ULlrcECSJCJ1nPPRHkWJFDEGSiNB5BtIfEFjBkygDGy+1oftBfIC+c8dTgZXoM1c6dL56G8QSiT5zpRMP/aELkUQmkP6AwAqeRBrYLl8+l1Dnk0yg39zxTGAVL17cECBekOujd33Go0nfTs1E8RIljLaIBag/dAESCo1bdxNtug+w4pmz5jDyRJs2XXrHdH/oAiRYylUvI7Jk+8CiYq1yRh43brjhBsMXLjr81M5vf0BgBU+iDWxnzhyX57R//y4jDcQuiXYdeolnAitcs1c0IPiKR5tA/kuPBl7PXhX/uoxse46r4VgilvtDFyDB8uZ7GcWsVdMNf0po0LKeGDl9mOH3Gn/9AYEVPIk6sDVp0kiULl3K8IPYJFGvQy/wRGC1bddBTB7S1RAfXqALKo5XKV1U3HfvPaJlnbKW7+TGGeKxRx4UlUsVsXwHVk0UxfN/ItKmfkrMHuF9Hem827XvaGuPKdNmGG0UTvTjUX/8PGyCIT5CgQQWbW+++Wa5VQXWqxnfEXkKFhMvZnhN3P/AQ7b01996zwrPXL5Vbus1ay/uuPMu8V3FWp4LtZ+HTojJ/hg4pp8hPoLFTWBRW959z12iYIkC4oX06aQve95sFpyHts+/+Lx4K9Ob0r9y51KROev7tnL0soNlwOh+Rn+oQGAFTyIPbN9//11Cn18igX5yxxOBFa7ZK8JNYOl+3bd25iDRpVk1KbAevP/vRl4v4f/Sq1atJsP+/mv3Gj4mHZ/juvAIFRZYLIhUYaSGb7zxRiMfhx965FEjfziIxf7QhUcokMDidiX6jOgp/aowchNJ7NdnsNi/Zu8q8cY7rxv7hYKv9ofACp5EH9hWrVomz5Hez9LTQOyQ6NdhKMStwCJWTOkrZ7GefOwRK23uqO4W62YNlgKr2vfheT+M4QE1ltBFR6iwwCJU0UT8K/WzYsCYmRbkm7dmt/jyu0rincxZxYxlW8T4ub/Y9pk4b42M3/N/9xrHChW9LWIBXXSEgq8ZLKewUx5dYE1ZPNHnfqFA56/fMyCwQofaVfclIoULFxY9e/5k+EFskCzXYTDEvMC65ZabxdF1U624LrhUn1NapAQWtUMszZjooiNUVIH1XaXaNrGkhlXIv/HgJRm++57/E/94/EnHPLovVGKxP3TREQpeCKy2P7USvYb1sKVl+SSzeOChB4x9QsVX+0NgBU8yDWy5cuUUefLkMfwg+iTTdZhSPBFY4XwHi6BBgTmzZZbhG9jpB8e8x64Js3ALLKd3sKZOD6zdvEI/XjjfwWJUYTR/7R5buzvlofC81busMFO9QQvjWKHg9A6W3j7hRj9eON7BUtvw1ttulX4Kcx49zHE3v57mFXgHK3wk28DWuXOHpDvneAB94o4nAougRtbFRzLg67/zaBKOWax4IJb7QxcfsUY4BJa//oDACp5kHNjo+4XJeN6xDPrDHc8EFq13o4uPRCem18G6Vi9dfCQ6Mb0O1rV66eIjlsjzWW7DFypYByu8JPPARudesWIFww8iTzJfh/7wTGARyTaL5e+/82iTbLNY8dAfughJZALpDwis4En2ge3LL78QrVq1NPwgsiT7degLTwUWvkUYW+BbhLEFvkVoAoEVPBjYLop8+fKJHDlyGH4QOXAduuOpwCJmz12Q8DNZdH5z5i00zj0W4f7QBUkiEY/9oQuSRCIl/QGBFTwY2P6C2mLo0EGGH4QfXIfueC6wiJ179iesyJKD47Xz089Z5+rVPw0yZsxo2+oMHTrU8Pli48ZNcqsfW4f7QxcmiYBTf+jt5AVufRYoTv2hC5NEwKk/fAGBFTwY2OzkyJFdLF483/CD8ILr0J2wCCxi1Zr1suE3LxhliJR4ZNP8kfJ89PN0Qx9giVSpUsntuXPnjTSiV6/ehs8Xa9aslVv92E5wf0xdtN4QKfHI1IXrXPtDb6dg4f4i3PosUPQ6cn/MXjnDECnxyKyV0137wxcQWMGDgc2E2uSbb742/CB84Dp0J2wCi1m9bmNcC636NSrI+q9Zt8k4N1/QoEoD9KxZs8ULL7xgxdXtxYuXRKlSpcT06TNEjRo1bAKrVq1aVt4xY8baBvuXXnpJzJkzRy685zR4+4L7Y9ri+BRaVWrW89sf1CZ58uQV/fsPEOnSpRNbtmwRU6dOs9rw8uUrIm3atGLRokXiueeek75ChQrJcK9evWz9NGDAQDFjxkxb+1OYyk6dOrUVpz4ePnyEqFixouWbOHGiqFSpks8+4v6Ysyo+hVa12lX99ocvILCCBwObM+fPn0LbRBC0tTthF1hM7ty5ZUfMGtXDEDGxCNWT6tupS1fjXAKhQIEC1oCsDsxOW4YEFs2UsCA7evSYFFec/sMPjcSCBQuteLdu3X0O3r7g/hg4ZrohYmIRqmeg/UFtUqxYMaudMmXKJLetW7cx2j19+vRySwKLfU79w+EsWT60fB988IFrPr1v9TrqcH8MmzjYEDGxCNUz0P7wBQRW8GBg8w21z4kTRww/8BZch+5ETGAx5cpfnxHq1rqhIWpiAaoX1a/8tXrqdU8Jzz//vG2AdRp89UGYBNbAgYNEtWrVZXzatGlyBoweBRK7du0SvXv3sfIvXrw4oMHbF9wfzdp0NkRNLED1Sml/UJvQjCC3E4utrl27Wu3ObcqPWQMVWKqvSpWqhk8NkwDmuF5HN7g/WnZoboiaWIDqldL+8AUEVvBgYPOPHGu6dTH8wDtwHboTcYHFdOt+fYaImDkyurNadHyuC9VLr2swbNiwUTRq1MgabJ0G6ZIlS9rS+RHh9OnTxaVLl2WYHmVxOvt4myZNmhQN3r5Q+2Pg6GmG0IkkdPxQ+oPaxJfA+uabb8S+ffttbR+owJo8eYp8FOgvH/PFF1/KrV5Hf6j9MTTKs1p0/FD6wxcQWMGDgS0waHZ4ypSJhh94A65Dd6ImsHSGDr/+EjnTuE5lcXnXAkMMhcLu5eNluepx6Lh6XbyAB1gWQ+FEP7YX6P1Ro+4PYt3ec4YYCoU5K7fLcr3uD7193KB34HSfl1y58qsV1uuYUvT+qNWgpthybKMhhkJh0Yb5slyv+8MXEFjBQ/2j+4AzM2ZMRXuFCbSrOzEjsHTmzl8o8ufPb7vZM0ULFRT1qpeXj/MGdm8phvVsLbcUr1q+tEzX9yHo8QuVqx8rHOiDbTjRjx0OfPXHZ4WLiMrV68jHeR1+6i869x4stxQvU6GKTNf3CWd/6O0TC+h1DBVf/VGoyGeiaq0q8nFelz6dRPf+XeWW4uUqlZXp+j7h7A9fQGAFD/WZ7gPuXL58TrbZ6tUrjTQQPLgO3YlZgeWL02cvitVrN4jJU2eI4SNGi8FDhsstxTdv3SnT9X0ijT7AhhP92JFG74+evX+Oan/o7RML6HUMJ3p/xOLfBwOBFTwY2IKD2u3gwT2GHwQHrkN34lJggdiG/uB0HwBOQGAFDwa24GnYsD7azyPQju5AYAFPUR856WkA6EBgBQ8GttDYs2cH2tAD0IbuQGABT4HAAikBAit4MLB5A7XjlSvnDT8IDFyH7kBgAc+BuAKBAoEVPBjYvKNQoes//ND9wD9oN3cgsIDnQGCBQIHACh4MbN4yZsxI8emneQ0/8A2uQ3cgsIDnQGCBQIHACh4MbN7Trl0btGsKQXu5A4EFPAcCCwQKBFbwYGALD0eOHJBte+zYISMNmOA6dAcCC3gOBBYIFAis4MHAFl6qVasqqlSpZPiBHVyH7kBgAc+BwAKBAoEVPBjYwk/Jkp+jnf2A9nEHAgt4DgQWCBQIrODBwBYZZs+egbb2AdrGHQgs4DkQWCBQILCCBwNbZKH2HjFiqOFPdnAdugOBBTwHAgsECgRW8GBgizzZs2cXBQsWMPzJDK5DdyCwgOdAYIFAgcAKHgxs0aFJk0aidOlShj9ZwXXoDgQW8BwILBAoEFjBg4Etepw7d1K2/65d24y0ZAPXoTsQWMBzILBAoEBgBQ8GtuhDfVC3bh3Dn0zgOnQHAgt4DgQWCBQIrODBwBYbFC5cWPTs+ZPhTxZwHboDgQU8BwILBAoEVvBgYIsd1q37RfbHpUtnjbREB9ehOxBYwHMgsECgQGAFDwa22CMZ+yQZzzlQILCA50BggUCBwAoeDGyxCfULzWjp/kQF16E7EFjAcyCwQKBAYAUPBrbYhfqmYsUKhj8RwXXoDgQW8BwILBAoEFjBg4Ettjl16mhS9FEynGOwQGABz4HAAoECgRU8GNhin507t16/H54/ZaQlCrgO3YHAAp4DgQUCBQIreDCwxQ/UV0OHDjL8iQCuQ3cgsIDnQGCBQIHACh4MbPHFsmWL5bcMdX+8g+vQHQgs4DkQWCBQILCCBwNb/DFy5LCE67dEOx8vgcACnuNLYF29+qeYOHGS3Dqh5weJDQRW8GBgi0/ofaxE6rtEOhevgcACnuNPYO3du09uDx8+Ii5duixq164jzp07D4GVhEBgBQ8GtviG+q9kyc8Nf7yB69AdCCzgOf4EFs9gLV68WPz73/+W4VSpUkFgJSEQWMGDgS3+qVKlUtz3Y7zXP5xAYAHPSYnAOnLkqAwXKFAAAisJgcAKHgxsicGmTeviui/jue7hBgILeE5KBNbx4ycgsJIYCKzgoL8xZubMaUY6iC9mzJhqEyoLF84z8sQaXGdch+5AYAHPgcACgQKBFRzqwKangfjl00/zxlW/4jr0DQQW8Bx/AssXen6Q2EBgBceqVcsxsCUgLVs2t/q1YMECRnqswdchbfU0AIEFwgAEFggULwTWiBFDbf9Jg8SmSpXKxjUQC1C99LqCxCWQ6xACC3gOXXy6DwAnQhFYdJ19mje3+GP/EpBkDOvVRva/fk1EA6pHl95DxKZDl0GSkSfvpz6vQwgs4DkQWCBQghVYNWpUF3WrlTMGXpA8XN69UNSsWcO4NiIJXYeVqtcxBl6QPFD/u12HEFjAcyCwQKAEK7DoGtMHXJB8+Jo9iAR0fH3ABcmH23UIgQU8BwILBEqwAqtk8cLGYAsSlz9/u+TIoUMHxR9//C7+/PNqxKHjHjp0SFz5/aoj+iAMEpcixUqIkyePGPcpCCzgORBYIFCCFVj9uzQ3BmGQuLCg6tGti01gdezQXpw/f84QP0Tv3r0MXzDQVyZ0H0HHPXn6jCGsILCSj7Zd+4jx48cY9ykILOA5EFggUIIVWMN7tTEGYZC4qALru9KlxJaN62R81fJl4uzZs1LwTJ06RVSpUuVa/IxYsWK5KFeunBgyZIhMu3DhvPjqqy/Fxo0bZPzKlcvi8OFDokmTJmLEiOE24UT76gJr3bp1on379jY/HffUmbNSTC1f9Yto37GTaPhDIwisJKRLnyFi2LDBxn0KAgt4DgQWCBQILBAILLDkbNK1bbp06eS2U4f2UugcPHhALFy44H/C54zcqjNY77zzjty2aNFCzJgxXZw5c9qambp69Q/RqFEjGX711VdtIooFFm1JeA0cOMBRYFEe2h4+ehwCKwmBwAIRAwILBAoEFggE/RHhkEH9bQKLBM/zzz8vhc6vv14xBJYumEhgLVgw3xBRGTNmdMzP4bx581phVWA1atLUElZnL1yCwEoyILBAxIDAAoECgQUCIRCBpQuiHj1+snybN2+ywp988okhsC5evCBGjx5tK0cvj4DAAk5AYIGIAYEFAgUCCwSCP4E1adJEKYSIsWPHWMKIxRFtM2TIYMV1gaULKTc/BBZwAgILRAwILBAoiSaw5o7qLn6o+q0VL5E/mziwaqKRL5Yo/1VBwxdrsMDSOXf6pDGDFQwnThwX2bNnN/y+UAWWE/ogDBIXCCwQMSCwQKAkk8C64YYbJNW+L26ls0+N58+R2ebT8w7u0ljG06Z+SpzcOEP68n6Sycr3/lsZHMtV47/uWWj5VIFV7suC0vdexvQyvnJqPzGxf1vp+6Zo9D5LpAsrrwXWb7/9avjcoFksInfu3CJXrtzi0NFjhriCwEouILBAxIDAAoGSLAIr1VOPG3l1AaT73PI+/uhDcksCq3eburb0Inmzitkjurru63QcDk8d3EF0aVZNhidcE1UECazbbrvVVl4sQdeB08AWCei4NLDqgy1IPiCwQMSAwAKBkmgCa+G4nqJuxS+teKHcH4oja6fIMAkjEjP7V06Q8RtvvFHcdecdFuRzE1hOeUlgcXrmt19x3d9JTKm+MiXzWz71GOnTpbVmsPQyYwUILBALQGCBiAGBBQIl0QQWcbsy4+NL8PhK03HyOwms+//+f0Y+fwLrb3/7m9zSI8Dti0bZ9oXAcieaAuuNt9+XfchxNewV6/adFzffcosoUrKUkebGTTfdZPjCwbsffGz4ogkEFogYEFggUBJRYD2b6ikpWmjQy/hKOumjMD1qe/jB+0TPVrWljx7lUT7yOQkfFXpcR2lUNudxElhcxhP/eNjKt3h8LxkmWCx1aFxZxu+4/Tbx/ef5bPum+deTckuPDCGw3Im2wPokV34rHg6BVb5GQ8MXKzz/QnrDF00gsEDEgMACgZKIAgtEjmQWWLT94OOccqvPZk1dvNHy9Rw8wZZG20xZslnx3sMmizEzl4tc+YvYjpG7QFHx06BxErX81l37yW2LTn3E25k+FOPn/mLVh2ewGrXqKv7x+JOibtN28vE2H4ugYzkJwpKlKoj0r70pKtdubKV37D3sGkPFkAlzLR/V5+lUz8gtxWmWbfSMZSLHp4VEweJfG+VGAggsEDEgsECgQGCBUEh2gfXgw4+KjQcvWeKDtlXrNRNV6jSRQqVctfrS32/UdJH2Py9Y+6v533z3A7F27znjGOWrNzDyEyyweg2ZKP2rd5220lhgqfkff/Jpw+cksNzSX37lDUucsU+dwXryn6lElbpN5Tk7lRsJILBAxIDAAoECgQVCIdkFFqGKDzeBcfPNN1tp81bvEn+/7wFbeumKNeVMkOpzE1iNW3ez5VOFjZPAIhGo+5zq6ZSu+njGilAFFs166WVFGggsEDEgsECgRFpgffTe64YvHBTI8YHhY46um2r4dPg9LSZ7lrfkYMNxNRwtxvdrI0b1bCG5sH2uLY18G+YMMfYpmDOL4XOCzu++e+8xfHo+CKxrAuPVjJYQGTZ5gXj23/818lP6Sxlet8KTF66X4aWbD9vyqPuoAuuOO+60Zqo4X7X6zY19WWDRrNXwKQttaU4CSiXdSxlEs/Y9xYJ1e/3uw48ddX+3/qONciMBBBaIGBBYIFCiKbBofapLO+cbedbPHmL4UgK/4K76VAGSId1zxj6rpw+wxWkAUeN6eRzfOHeoUZbOtoUjDZ8/aCHS4+unGX7ioQfuk8tR6H4VFlgXd8wz0hh/9dLPmfjgnVdt8WQVWImOkwCLZSCwQNg5ffaiqFW7jhRYtKW4ngcAFa8EljoYOw3MDAus115+XhxaPUkM6txI3HrrLdJH6z4Vy/exDDevXca239mts+Wv6XT08gk6/vLJfUXHxlVknNbB2rt8nNye2zZHvPifNNbaWJS37BcFxPlrfq437Tuk6/XV2tUy9Xj/jg3FrqVjRO3yJY06ELTo6T133+m4P0GLlLasU9bw/+PhB2V99TZg3nn9JcOnQwKLyjmxYbrt2DyDRb7LuxaIeaO7yzi1B8N5neqs+yCwEodxs1fK/iXo5Xg9PZaBwAIRgcQVo6cBoBMtgeWU39d+gdKtRQ0xc3gXGVbLc5vBUvOQIKFtyYLZbWWSELn55ptsPqf665DA4vC6WYONdDea1Chthdv9UMmWtmf59V+U6ZBfzUcCa8qgDjJMy1HMH/OTDKsCSz+uzuvp/ytFpOrT94PAArEABBaICFWrVpPiirZ6GgA68SKw6HGXLiqc8uvpvGp7IAIr9dPXP6fTql55sXPJaKNct7iexqgCa8die3m+6NehgRVuWvM7I50XJvWF+g7W0ol95JpaFNbfwaK1wX7bu8jYn3A6L90HgQViAQgsEDEwewUCxSuBRd/8G92rpeMgrMIC697/u1tuT2+eaeWnz8Pw7NPkge2Nff1Bjxw/fj+jzcdl6x94dgqzwCIyvZnBsRynuJ7G+BNYbo8I/QmsYd2bigfuu9fwq/gTWJvnD5fbdM+llu97UXjTvGG2MpzOS/fFm8Ci9Zx0XzSgR3C0UKkOpd13/4NGfhXqA90Xq9ASFP/39/tEpz7Dbf5H/vGYkZfpMXi84fMHBBYICwcOHxNrN2wW02fOFsNHjhZDh40UPXv9LLcUnzt/oUynfPq+wHuOnjgjNm3dIWbNmS9GjBoj+2HQkGFyS/EFi5fJdMqn7xsNvBJYRN/29UXX5tXl6uN6GqO+WN24eimbmCCWTfpZVC/zlxhKCU7HZd+V3Qts6aum9Tf2UV+u14WEHlf3czouQe+NcdjpZf5j66aKw2smG3715XYSjXo607ZhRSnQiDNbZtnSTm6cYb3gTu+XcTrPzO1bMV6KTnr8qR5XPRf9nH/6sZbxa0UIrND4ukwVMWr6UpsvVZrnjHwqRb4obfhU/pX6WcOXEtbsPiP7PhQhR5/5UfenX01OWbRBhp/6V2ojv853lWobPl9AYIEUMXf+IvFDo8a2d6p08n2aT3z95TeiWuUaok3z9qJnl77i5+4DRL8eg+WW4s0b/SjTKR/l18tgsufIIY9Hx9XrAq4zeep08dVXXxttx+S41oafFy8pKpWvIn5s2kb07PyzvT+uxRs3aCbTKR/l18tgqlevIY+n18FrvBRYiYQuiPRlGhIdOld9mQb6cYCeL94EFi346U88cDrx6GNPGOmBoJbh61i6wMpTsJgVLv51GTFuziq/Zej4yrt+/wWrPLfvFqoryt/zf/ca6SyefNXLza+nUZjW/6ItLZzqlCcQILCAweixE0SePHlsA2vntt3EzrUHxMk9F6MGHb9uzfq2elE9qb76OSQay1euNkRT1/Y/iX2bjhrtFAnouHR8tU5fff21rKde92CAwAKhEG8Ci9Z54rDboyhdAOjpE+etMaDFQ/V8BH2uZsbyLYaf0QWWejwSWLxOFi1Equ7n9Emapu16SKgMDut5/J1bttwF5HbsrBWiQLGvXPMRGV5/S/QfPcPw6/uwEONHoLfffodrPqdwIEBgATFj1lzbQNnux07i6I4zxqAai1A9qb5q/el89HOMN85d/FXUqFnLOqeK5SqLE7svGOcfS2xeuUvWk+tM9afz0M8tECCwQCjoAuvixTPGtRIughFY6iPCdj0GGemEv4GeZpl0Gv7Y2chX4puytsVAnfAnsDhMn9+hmSOOf/5tOaMspzJ0/J1b4c9LyS0tNtq13ygZph9V6PkI/duJKnrZtEgqCyy1PLf66Pv7AwIrSRkzboI1ENasVkcc33XeGDDjEToPOh8+NzpP/dxjmVy5cll1nzNpoXF+8QTVn8+lTt16xrn6IpEF1qQB7QxfsuK2ZhjhtqBpILDAqlat6vXrr04t41oJF9ESWIFAHz9O+590hl9HF1ivv/WeFfYlsFbuOGGUxfiqcyDnxivAE/Qxa3qsqOdx25fp0GuI+GeqNFZcFVhudXALBwIEVhJRsuQX8mZTu3o9YzBMZHLmzCnPm85fb5NYgEXI8P5jjLonEnR+dJ45r4lIvQ10EllgPf7oQ4YvpTSo8rXhCwe0yCqtHr9ofC8jLVBoUDq1aaYM08vs3xbLI8NDuzWRL9Tr+fV9dV8g0HXAf1fRIKUCa+H6fVbY7dGd+mjN6TFbIPAjOrdHdcyIqYvE4o0HbT5+hDZ00nzLR+9i0UelKayKMCd8HY+YuXyr8RK6Ds1kZc9T0PAT05duDujciPI1GooPs+W2PULNnvczK+zW1jx7FigQWEnAl19+Jf/o505ZbAx6yQSdP7UDvRCut1GkoV/r8c1Yr2eis331Puvc9XZhIi2weNAngv0uIb2cy2F1aQUdVWDRYEKsnTnI8BXK/aGxL/PuGy8bPoY+V0P708vgbgKlSN6sVrhWuZJGOqHuu3XBCMfP23BdCadvDNLK+G6f7VHLV8PqEhBuq9H7I95msOKBdz/42PCp+FvGwRczlm0RUxdvlO+i+RJY4Ub9lqGO/r5ZIEBgJTDFS5SQN5epY2YZg1wyQ+1B7ULto7dZJChbrrw8fqI8lg0WOn9qB2oPvY1iTWCpQoLR86RUYDkJDKdynfAlsJzK1fEnsGj5CP2bgG5l+Upz8+tpbgIrWOLtHSydJ/+Zyob6WM5L9OMUKPqlkSdeoceB+vnpecJN3AmsNes2GT43flm7wfARb2TMaIvny1/Ahp4/pfgqq2Wr1obPa77/vozIny+/MaABExrgqb30NgwHuXPnTsoZq0CgdqH24baKNYHVpkEFAz1PKALrycceMXxO0IeoiTcy/NcK63loYVQOu5XnT2Cpa2/5+4yNm5/QP06t5o+kwIokXggskBjEncDSxZEvMmTIYPjcyti554CYOWee4Q8GX+WEW2DRQFWvVgNjAAPu1K3ZwOfjKi+g8gf3GW4cG/zFoGvtw/0QaYGVNvVTVthJYAUCrT7OYV+iw0lgOYkOX4Q6g0WLqfJioqogcyuH6Ny0mpGHBNTv+xYbfoZWxFeFp1quWz3Vda5G9mhulBkIEFggFoiowMqUKZPcugmfzl262eJ6vpWr14nXX39dzF+01Epfv3Grle+jj7LK7SuvvGKlU94Nm7bZyvEnsHLnziMW/O8YY8dPFEuXr7KOQVuamSpfoaJYuuIX6Xv1tddkPRo1birj0RJYNDgd3n7aGLiAf6jdwiWyqNxYX2IhVujctrtsr0gLLBIBNMjTp16CFVgElUEEMoOl5p8+tJPhy5f9fWNfxpfAOrJ2ityf3n/SRZIKH8dpBkvPc8fttxlpajrh9A4WQR91VvPxB6CzZnrDykPvaXG6OoPlq/6+gMACsUBEBRahiyanNF95WBw1aNhIChsSU2p+t7BTGSqqwJo2Y7atDEYvM+vHH1u+SpWrWv5oCCw8evIGr0UW+iU4qN38mZcCK5EoVSyv3Fb8prD4JPObRnos4UtA0Sd9SJzp/kCAwAKxQEQFFi06ePTEafGdyzsv6zZukULn7bffMdKYN954Q25HjRkvTpw+b6QfOX5afP6/n+MHK7CcBNKbb75plMkCi+E0p/2ZcAgsGoyO7zpnDFKBMLL/ONt/l3q6Sqkvvhe71h40/IHy8IMPG75Yg9rRK5FF5ezZcMQ4RqjcddfdVn/9/d77jPREYM+Gw6JTp07qbceweBBY6t+WLzHhNaunD7A9uotUPSJ1HH9AYIFYIKICiwVIvfoNxaGjJ410zsOrPzsJJH026ZtvSxmzSyRi9uw/LKZOnyV9NWrWtpWREoFFM2TZsmcXpUt/bzsGwQKLZtLo8eRPPXtbedR6qngtsDp26iIqlK1kDFCB4k9UqYQqsFJyrGhSvkxFo52DIWeOnEbZoUJtuHzOWsOfiPibxYoHgQWiAwQWiAUiKrBiBSeBFSm8FlihPoJyEz2qn8MksL4o9o34svi30nd42/X3vZ595jk5q1K9Qm1RvnRla5+K31eV2xnj5lu+vDnySzie7aOc4uMs2cWwn0dL35aVu0XqVGlEySJfyWNxHegdmddfySj36dCyq7U/Ueyzz0Wbph1Fr079jDoHi750wOmzF422d2PV6nXyg8h6mV7gdl7kp/75OEs22R7k47bm9p49caF4/rn/ynbV+7dKuRrihedfFE8+/pT0UZ/SjGPRgiXEKy+/ZuUlf75cBUTXttfXq2H/W6+/Y9QpVBrWbSxmz56t3npsBoEF3IDAArFAVAVWl24/2dDTw0X3Hr0MX6SYO3+R4QuFUAUWUbNSXUussM8pTAP4yAHjDf+DDzxolEmQECLRxfnUMkl8zZu8xChLzcOCaePyHSLrB5/4zKvH928+bktLKdSu9OMG2hK1atcx2t4N3kcv0wv0c1bp232wrb11VP+0MXMNH8ECy6n/dXp37u83T6j4msWCwAJuQGCBWCCqAguETo4cOYxBKRS++6qs3DoNsCSw1i3ZZvgzvZPZKMdpf9VHMy1jBk8Sk0bMsNDzTB095/p21GxrZsytPIJE24q568S9//d3mz8YWCSFgl6mF+jn7OT3lcdXexMssO668y6/5RF0vRCzJoTnu4nUjm7mJLCG9GhlDLYg+Rj8UysxcuQw4/qIBCNGDBWdejl/TxAkF516DnS8Dj0VWPRCuu7zGnqni96jUn1O70CFk0CPd+b8Zes9s1DxeiBv3rC13DoN2CSwUv0ztQzTKty33nqrkZdJm+bfcntwy0lHQbRjzX7xj0cfM/a77dbbrPAtt9xihZ3q43Rc8jn5Uwq1K7Vv1arVZDiWZrDUmb9jO6//uMGpfYh9m445+pn8eT4TQ/qMkuF33nwvRTNYnOYrPRRoaYuUCqxaVcoYgy1IPmpVLiM2bFhjXB+RgI5boUrKP6sCEo8KVWo4XodhE1g0UJHA4HiLlq3E8BGjrfiyFavFz/0GyPDhY6dEq9ZtxcLFy6z0xUtXiJ69f7bim7ftFM2atwxIYJ29cEX+ipGOSfH9h46Jhj80suWpU7e+Lf5DoyZixqy5VpyO0b5jJ0eBRMdbu2GzmDRlupHWf+Bg0bRZCxnuN2CQbBP61aSeL6WEOpDTezkkhmjWh8UVc+cdd4qls1aLr0qUkvGfuw0SB7acEK9leENULV/Tlpfeo3rowYesR3NDfx4l7rj9DjlI8v5Ek7otbPH0L74ij63OjFEeejw4vK/948f0/k+Jwl9YcbUchs6F6qj7U4p6jQXDvoNHxM61B4xyvYDamB7LvpjuZetzOxuW7ZD9Rf2itkuXNj1scXrnjfJV+K6K5du9/rBo1qCVDL+d8V3L//Yb74pHHn7UOL5Kq8btRfsWXQy/F+TJnUecPHlSvfXYzElg0d+DPtiC5IOuA/3aiCR0fH2wBcmH23UYFoHFguezQoXk9vS5S1aeqtVq2PIQvFxDhYqV5LZGzVri2MmzMszLJpAA4rg/gUXLOrz19ttGGgs+XgiVFjOl7aGjJ+R274EjYuz4STL86quvGvvrx+vUpZtNgHXr3sOWj0QhfexX3z8YDh45EbLIilWCmRkJZh8dnr0KlXjpF3UWLKULonrR3m74mr0icxNYR9ZNMwZckFy4DWyRgo6/eMMBY8AFycPiDftdr8OwCKz3M2cWWbNmFR9++JH1TUF9SYNKlatY+/2yZr3c0pILnJeWRqAlEXSBs2vvgYAEFq24rqfNmb9QLi7KZbPQIgHIdStarJj00crutH33vffktkzZchK9TF5tniCxRWm0OCrFvRRYRPYcOUSH1l2MASpeqVy2uij7bQXD7w/+9VwodGjVWban3sbBULRoUVGtUg3jGLHGz10Hiq8/L234/UGPF8P1weqqlaqLU6dOqbcdw5wEFg9uEFnJi9ugFmkwi5W8+BJXRFgE1nvvXRcuDH/ShmBxwjNZBH+smQVWlg8/NMqmX3rR9suvvg5IYG3YvM1II4FFW30BVM5D4q1wkaIyzDNZLLCc8hOqwFIZPHS42H/wqFHXUCld+jsxatBfv/ADKYd+MEDtqLdtKFSuUtXzHyIkOtRexb/4Rr3lOJqbwCJmz54hb3AguahUqYJxLUQTqo9eR5D40P1HvxZUPBVYk6fOkFt65CZnobJe/2YgQXESYLSYJ8WbNG1upfFsEwkS9vHCov36D5Lx8hUqyTit4M5CTC1bjZ88c0Fs2b5Lhvl4BL3XRdu16zfJfQoVKmzlpzi9V1WufAXpmzZjltwWKFDQVrZeJn+nkKDz55kw9lF5vfr0NcoIBerYiuWCX3Q0maHFWlu3aWe0qRfQY+xmjVoaxwQmdA03atwk6G8Rgovil1+Wixo1qhl+ACLF0KGDDB/4C08FVrTQBVYyUKZsWTlI6QMXcIZ/qVb2f495w8Vnn32GfvGD/M9v7gLZXhBYoUFtqfsAiBS4/nyTEAKrcdNmhi8ZoHfH6AIfMWCsMYiBvxjef4zYv3+/+OPqVXH16p8+KVGihOELlCxZrj/a3rhlu+yXRHpfzgvovTdqF2ofvoYhsEIDAxyIJrj+fJMQAivZoUec/ExYH9SSlX2bjv7VJtfaRxdDbmTLlt3wBUKqVKnkVu0XWi6Ejv/1F399CigZ+eqLr2U7UHvo1y4EVmjkzZvX8AEQCQoWLCD27Nlh+MFfQGAlEDygE5uW7zAGumSAPrfDbaAO6CR+6FFy7959RLt27WR86tRplkBiYUXb1KlTi+XLV8itLqSI06fPiCZNmogGDRqK999/X6xcuUoKrAEDBhp9QkycPE3Wp06NekZ9E5ns2bPL86bz19uEgcAKjcWL5xs+ACIBZq/8A4GVgNByEfny55d/APQhXX3gS0ToPOl86bydFoflWabffvvdEkpuAot97733niGuuBw97DSDpTN02EhL/KkrrycS44dPtc6R1m7T20AHAit0pkyZaPgACCcnTx4ROXPmNPzADgRWEkAr5POglz9fAbFk1i/GwBhPUP3pPPic1C8AuKEKJFoe4OzZc2LWrNmWL2PGjHKrCqw8efIY4koVU2o4EIGlcvzkWVGgYEHrHOZMXmScZzzQtX0P6xzofPTz9AcEVuhgJgFEGlxzgQGBlWTQZ4kqVqxkDYr0X8jimSuNgTOWGPzzCFlPrjPVn85DPzdfkPgpXry43GbOnNkSSP379xdbtmwRL730kiWw0qRJYwgplbVr10no0eBzzz0XlMDSqV6j5l9CJX8BMWX0TKMdYoHJ1+rF9SToc1D6uaQECKzQwWAHIg2uucCAwEpyzp6/bBvcmR/qNRHzpv71seFIQMej4+p1oU8QUT31uqcEXST549dfU74PoR83GGj1/6bNmhvt0KpZW7Fr3UGj3cLBirlr5fH0OlC99PqGAgRW6FC/6D4AwknXrp0MHzCBwAI+2bv/sPxIN61Urg+2XlClajVZvvph8HCgC6FAOXfuvJydUtHzqOjHDQfrNmwR3X7qIXLmymW0ZygUL1FClkvl68cMFxBYoTN69IhrA15nww9AOChcuLDhA85AYAEAogYEljeQQNZ9AIQDXGuBA4EFAIgaEFjegEEPRIKzZ0+IHDmyG37gDAQWACBqQGB5AwQWiAT0Y6NTp44afuAMBBYAIGpAYHnD4MEDxc8/9zL8AHgJhHzKgMACAEQNCCzvwOAHwkn//n1F+/ZtDT9wBwILABA1ILC8AwILhBNcXykHAgsAEDUgsLwDAyAIF5cuncX1FQQQWACAqAGB5R19+vQ0fAB4waeffioOHtxj+IFvILAAAFEDAstbdu/eZvgACBXMXgUHBBYAIGpAYHkLBkIQDho3/sHwAf9AYAEAogYElrdAYAGvGTp0kOEDgQGBBQCIGhBY3gKBBbwG11TwQGABAKIGBJa3lChRXBw6tM/wAxAsEFjBA4EFAIgaEFjeQj+np8+Z6H4AgqFgwQJiz54dhh8EBgQWACBqQGB5D804/PnnVUf0vCA+0PsxWFKlSmXbOqWp7N+/X26HDx9mpDnhVIY/9HNNJCCwAABRAwLLeyCwEg+9H4PFlwDS0/7443dLYL344otGfif0MgJBP9dEwq/AOnPhinFTBAAALwhUYF25cs64eQFn6tevJy5dumgb7LJmzZrwg1ki4yRerl79wwqrM1OHDx+SaS+88IL0PfPMM3J75MhhYwZLLZPD27dvk9vp06eLyZMnybAqsNR9/vOf/1hhTlu3bq2RTyV9+vRW+MKF88a5Jgp0z/IpsPqOnitWbNhl3BQBAMALAhFY06b1vXbTX2XcwIAz58+fEgcOHIDASiDcBAv5GD1PIL506dIZaefOnbXKzJIli/TpAqt27VqiVq1aRp3UeK5cuWxpTnkI/VwThe3bV8p7l2o2gXX2/EU8JgQAhIU1W/YFJLAuXDiLx4QphB7tqAMZBFZ84yRMMmTIYIgWX2LKyZc6dWrXtNOnT4lMmTLJsC6wOKwDgfUXdM+6ePGs7V5mE1hkEFgAgHAQiLhig8BKGSSwihcvLtKmTSu+/fZbCKw4h/puz57d8nEfC5Tdu3fJ/qW4Lo7U8IED+8Xzzz/vmK9bt65SZKlpJNyo3KJFi1oCS00/deqkDPOxVbEUiMD6/fffZL40adKILVs2G+eaKOiPB8kMgUUGkQUA8BK6p8xeukG/1fg0iKzAOXbsmLh8+ZIxuOn5QHyg92O44ZfbI4F+romAk7gicxRYdCOkG+L42auMGyUAAAQKPxZMqbgiW7NmtrxxLV06wbihATv0C7CDBw8mxWCWDOj9GE7++OMPzwRWixYtrNkvgmfEVPRzjWfo3kT3KLpXOZmjwGIbPX2ZvDkCAECwnL94Wb+1pMgWLBgtb2LAN7Rcg+4DwB85cmQXo0d3NvzAP3Rv8mU+BRYMBoPB4sNIYMFgKTVcN+EzCCwYDAZLAKtQoYLugsF82qBBg0THjh11N8wjg8CCwWCwBLDjx4/rLhjMp2H2KrwGgQWDwWAJYhUrVtRdMJirQWCF1yCwYDAYLEEMAyYsULty5Yo4evSo7oZ5aH4F1p9//ikObd0qNs6dK9bNmAEAACBGIYGl+wBwIlfOnIYP+Ie0EGki0kb+zFVgzezRQ7Jz+XJx8eRJcfnMGQAAADFMwYIFxbEDBww/ADokxnUfCAzSRKSNWCe5maPAoh12r1plFAoAACB2WblkiahWtarhB0BlcP/+4scWLQw/SDmkldxEliGwFg4ZIk7u22cUAgAAIPbBzATwB64RbyHNtOiadtLNEFikxPSdAQAAxAcYPIEvLpw6hWskDDjNYtkE1oqxY8XK8eONHQEAAMQHefPmFWeOHjX8ABAFCxQQOzdvNvwgNEg7rRg3TpVUdoFFCuzU/v3GjgAAAOKDhXPmiHp16hh+AAjMXoUHekyoz2IZAkvfCQAAQHyBQRQ4cfzAAZEzZ07DD7wBAgsAABIcCCzgBK6L8AKBBQAACU6OHDkMHwAQWOEFAgsAABKcmVOmGD4AOnfoYPiAd0BgAQBAEtClY0fDB5KXIoULGz7gLZ4JrF1btogbbrjB8Ov4y0PpKnp6KISjTEKt79wZM4x0PZ/uBwCAcIPHQUAF10P48UxgkXAo9/33hl/n8ccfF3/7298MP/Hu22/LcvZu3y7jGdKnN/LEGjfddJNNNK1ZvtzIw0BgAQCiBQZUwJw6fFjkyJ7d8ANv8VRg6XEVPU3fn3g2TRqZVqJoUXFB+6D0bbfdZitv99atjsfJ/b+fnHZs29bmb9Gkia0ud911l7Fvnly5ZNr9991npD3y8MNGfYntGzfK9M+LFTPSdNTjXzx1yjiGnk/1jxg82NEPAACBAIEFGFwLkcETgUUL2ekDPsUnjhlj5OU03aemOYkIPU4c3bfP5svxySdWnAXWlnXrHMtggbVo7lwjTQ1v+OUXGXYSWCsXL5ZpaZ55Rm779e5t7a/n1ctVwxy/5ZZbrLBab4LP7cCuXUa5AADgj4F9+4reAd7TQWIDgRUZPBFYc6ZPt4mFVUuWuIoMwlca07pFC5nv1ltvtfb5+osvbHnatWplK4vEEsdZYKn5VVHDAsspTQ1z3ElgOe2TLWtW47i+8julPfHEE1acoFkyNY++LwAABAINrBhckxfq+4b164t2rVsbacB7PBFYhD7gU7xvr15GPqe8TPqXXzby8ftaTqLi9JEj0nf+xAkZf/vNN608XgksfrToT2Cp8TczZjTy6vmd9r3zzjt97sPUq1XL8AEAgBs7Nm2yxBUEVvKiXgO4DsKPpwKreePGVvy5tGktcaCKgfQvveQqDv7z73/b9lHz8eM4ht/B4pkuht/dCkVgURkcX71smdzSrJJeX3U/ht/f0vPpx3Da181/9tgx0aBOHcOvlw8AAG7kyZMHA2uSkzt3blwDEcQzgTV/1qyABn3KwzNOscraFSuscN2aNWWdu3fqZOQDAAQGvac5ZMAAEGV4cNX9IDkoX64c+j8ERg4dmiL94pnAkoX5WS2YPizpL08sUKFsWdtMES3FoOcBAPiGZoJ5QG/evLkYPny4GDlyJAAAxCWDBw8W+fLlk/c02ur3PB1PBRYAADB58+ZVbycwGAyWUObvUSsEFgDAc+jGA4PBYIlsRYoUEYvnzTPufwwEFgDAcyCwYDBYMpivWSwILACA50BgwWCwZDAILABARMH7VzAYLBkMAgsAEFEgsGAwWDIYBBYAIKJAYMFgsGQwCCwAQESBwILBYMlgEFgAgIgCgQWDwZLBILAAABEFAgsGgyWDQWABACIKBBYMBksGi4rAuu222wz0PCo/Nmtm+HxB3zUcN3Kk4Y8EgXzUOlTuuusuwxcI/upWo2pVK5w5UyYjPRj+fu+98rjflyplpIHkBAILBoMlg0VFYDH+BnzgLf7aWxVYoXL22DHb8ejjvnoekJxAYMFgsGSwmBBYjzz8sMiWNasMnzp82EjnGax+vXuL22+/3ZZGDOrXz1auOoPF+cqXKSPyXbuxq75n06QRb2XMaPM1qFNHzJs5U4Y/ypJFbPjlF1t6Dq3BbrnlFluc8z380EPiwK5dMnz33XdbaX179bLlU0n/0kty+8QTT8jtT507S9zys09NS/vss7Y8WT/8UGxcvdrIp5dBdWSBRe3slu//27sSWCuKLBqXmGiiJqIDRmVxiQsyyCiboICKwrAGEQcEEQaMbCJBImRwQXRAUBYNgiMiomK+yIDElY8ssimb+YKKCwoqrqBjEIwbPX0Kb3vf7er33v/89/77/HOSk666tXX369d1uqr6Nka2RCz169PHbWtUr56YnyQtK6PAeumll6ypVBgzZow1lQl33nln0LhxY8ctW7a4/1lFYtu2bW4G4tprr7VJMUyfPt2acoqTwntwaVGa83nYYYdZ00FBt33EEUdkdU5zjdKcj/JEtWrVrKlSomAElrZD9CBN0rXAks69Tu3asfqEWmDNeOihWHrryy93WwgssbVq0cJtIbDEhvbvDm9oY26/PUXMTH3ggZQ8I9XIj86Xjc3HlUuXBlvffjsx/8knn5xyfqZNnRrs2rkzVo8tZ9vcs2tXyvlJJ7BE9IFzZs2KpVvadm3b6ViavGTlY2UTWLgeDxblUYcPuao3W3wcPkQCP/zwg0mJI58C6+eff7amREyePNmaSoXyEs8W33zzjTV5UdHXAJGMghNYMtqj07XAkrTT69SJ1SfUAmvWI4/E0n0CS0aPrMCyZZPSatWsmWK3AiPJ5uOqZcuCrSUl3vw9wqcaa5Owb22WzWPTx40d67Z7v/suElgyaqapz7cVYChr89u2bBzs37dv0PLSS6M41mvhyU3OJXlostAFFq7V6667LqhVq5aLH3XUUSlpgqFDh0ZhC+S78cYbo/yvvfZa8P3335tcmaHb02Ftq1+/vtvHd9991yY7IE+XLl285X247LLLrMlByst206ZNrt2rr746OOGEE6K0ww8/3I1qffDBB1FZAAKrX79+QatWrVweyd+zZ8/guOOOi/LBduSRRx64Z/z0U2QbPHhwcMwxx0T5BI/9MbOBeoDi4uKgRfjAjNmFjRs3OhsE4Omnn+46vA8//DAq+0jYP5xxxhkuP6CPEW35zplcFwDShTimhg0bBt+F90OgefPmQbNmzdyxHB/e2zR0vfa8rlq1Kmjbtq27n2OEEOjatWvQtGnT2KjZl19+6crp/QfR9uVhP1evXr2UtpHWqVOnWPs4t77fXfLhGsP5OO2004J77rknaNOmTZSG6xr/hT59+qTUe2vYn7Rr1y6qp3v37u53AvD/Qv5zzjknSoetUaNGLq5HsNAnXHHFFUH16tWjvCD2B+eqkFFwAqvhRRcFq5cvd6Mrkp5JYKWbIswksESUSFtaYN0/fnzQuWNHF7512DC3/fCdd9xWFuYvefllt20U/rF0PW2vusoJlX3hxYdj0mk2bCkCC6NSx4c3npL1692fF2n/DgURBM2yxYtT6sA0na9OjL6NHjkysU2xYSsCS2zVwpum5PMJLFz4mH5MEnYy0odp1cEDBkRpuPFNnjgxVka4cN68mI08dFjoAqtXr14p8ZtvvjkK47oWJAms5eH9a/Xq1dYcDBkyJCX+0UcfOdGhaaHb02GfzZe+cOHC4J3wnpUJ7733Xgp9dX311VdB586dXacO6DwdOnSI2WwdOL5vv/3Wmwb8K7z3AjoNHTsg5Xzw1SWQtHR59AiWL7+dqrMCS28BLQQEVmC9/vrrUbh27dpu66tLwhBKSfDlt2EII2uzI3efffZZShyQ/PI7aBvwn1AHQGBBTAl2797ttoMGDXLba665JkqT/4D9PWxcBJb+j9nzZMOFiAoVWBVNPYJV6MSFZG2adc87L2bLF0XAlZYQXh3bt0+xNQj/yDYfeWix0AUWgBEnuXnrDlbf0JME1tSpU71TVPfdd19K/JlnnnH1aVpk6kwypWP6av/+/dacFrvCh9u/hA+9FhhNOvHEE6O4r710+6MFpKTpPL17947ZRMykOwbbjm8fbB6NTALrqvBhWQMjYQJffjx8A3q0yQosoFu3bikjcr66BHJufPAdLyCjRdqu07GeT+PXX39NiQOSP0lgTQwflCGw5s6dG9lWrlzptgsWLHBbLUgtcJ39/vvvUbxBgwZuKwKrdevWURrOF5B0vIUICiyPvZCIPysuImvXRPo/b7ghZs81xQXD3XfcEUsjySQWusDas2ePu+n7OiUJo/MSgeW7yYvt6aefdtvbbrtNJ2eNY4891o0cYXQhXTs2rCH2WbNmmZTSwZ4PTIc98cQTLvzKK69EaXPmzHECFVNRGukE1pQpU9IKLNgwMoJRcwuco/Xr10dxLDPBNB1GUKQu/FY//vijC+/cuTPKC9SsWdP95oDdL8AKrKKiomD06NHBkiVLvPlFYGFqEHkQ9wkslPH9fnXr1o1Eh0yxZRJYn376aRTWdgh9jEh+/fXXsXQRWJhRgKj2Tb9K/kwCC8cKXH/99VGaCCxAptlFfA0cONBtIaBwrC+++KKLjxo1ym31FOH8+fPd1neudbgQUaUFFkmS+WehCyzc8PWUlKxvEdi1RVhL4sNbb70VhQ+mI7DtlQXZTBOWFbLOCZDjFMGSDXT5suKXX36JRAbgO2cQEbI+6mAhYi1bdOzY0ZrSAoJo+/bt1lxqQJzv27fPmr1A3rIAAmvt2rUZy2/YsCFtHGv6fMD/UdaiVTZQYOWQkyZMiNlIsqqz0AVWZcYpp5zinv6F+UZZhKSsvUoHjJah7ptuuskmFSwefvhht896VKcQgfVZ2M9p06bZpKwgAouIo0IEFn5MS5vHctSIETGbpUxZaVtSG4hjMX1Smrb9t6jI+6acj127dInCcLb52IwZsTyl5aXNm0f7lO7tSZKsDKTAIgiiKqBCBJbQCps1oeDRcXH4CQ4Kn1xseRALCXVc3uoT6jfhNHv16BGF7X74bDbuI5ykaoGVRPi50vEvt29PcbCqiXnxj7dujeIQbTYPSVYmUmARBFEVUBACS7yza5sVNEkCy+azAgvpWMSn813SrFksT6Z6Jb68uDj2bcRF8+cH8+bOTcn31/PPj5Uteuqp6Bt/vuO0bcITfKcOHVJsJFnZmWuBJb6T8o0LL7zQ+VjSC+RzAdTdI3xAxHbZsmVuUTTCQotx48ZZkwOmsIDzw3tVNkiqH4CrFoFvIXo2GDt2rNsmtXH22WdbU6mg6/W18fjjj0fhJ5988s+EBNg64L5Cr7sToH/D71RewDUmvrw2b95sUg/A7psGyuzdu9eaI+AFBaxpKyvStZ1LYAF9adb+5QMFIbD0zUFs9evVc28mvL1hg4uXVWAJ9bfwbBkb99kw/WjzCGvUqBGF27Vp47ZJAgv+vRA+79xzU9JsGMQxb9640YWnP/ig8yrfpHHjlDyW/+jWLfhbgwZRvEO7dimCECN6+MSNbks+IUSS+WC+BRbeosIaHr3wGY43BwwYEMWxTibTWhm8Ji5vdQHr1q1zN1AB/lMQCXhlXcIAtjjmmTNnuniTJk1SXomHDyl5qwqAmwcAnYW8XSXwdV4QWElA21deeWW0LyUlJW5RPjpoLPrGvixevNilyWJzOHv0vbIP+NoH8JYY2vjtt9+Ce++919mkTfuSgABvuMF5pixu1udL0Ldv3+h60QJL59HAJ4SSIPvuOwbUh2tC6hXh8v777zu/aMOGDdPZHXQ9eAtSBBbOqV7wjTdI9ZuIaANe2u3/APuOa9d3bHDxAQetqFcLLO3tHedSrh29b7Y+e/y4PmRtm86LNwTFhjc44QNNgGsT+wuhhutIQ+r4X/hfh1DD+cAWb5zCgaoA7jHuuuuuyP0CgLL67VO8Zbhjx46UOJyu4joD8PIA8mN/8JKDCMNXX33V7Z98YUDcSLRs2fJARX9A/59zgYIRWDZNKJ/EGRLegGyar2ySwNIcfsstaevw2WxcEyJGwnAMim3T8KKyZUsrsBY8+2zkaR3EWrB1q1al5NkR/tEgRLVNO0uFo9Qp998fxeWTOrYtkswXbcdS3rACC57GAbm52w5GexEX6FfMAVsGHal0upKm8/jC6BjgUkDb9Bte8io7PHBj4bBtE/DZILCwUFxooUewdHkRlHCMCsCNwgUXXBDLp5Fk941gIa9236ABv1ZnnnlmFAakY/adT0AElrVrZBJY6cr6RrDEl5VvBMruJwSW+J4Sm3beqfPjGsVxiwNRScNicbuP2Ac5R7hefCNYtozEred3QOdNKgeIawnYZMTrrLPOctujjz7abSHK8QCjIXV88skn0dcFdL3i60rb1qxZE7PJvsP7P0aFR4wYEaXZUWIILj2CBQ/9AAY/AAjc4cOHu3C64y9vFITAwlQYwg9OmhTZMK2Ht/Ak/sX27e5zNhjJ8dUBQtzABwq24kUcI2HdQ4Us+ay4Ql6kiTBKskl53xShpEMEirDa/cUX7gIZOnhwVNYnsOAJHel4+0cfi663xSWXBO3DzsGmi3d3Sy2whPCkroWWrYsk88V8CywLjCjg+scTNoBRIsTRISQBT9vII52w9msknxhBusAXRqco7hJsulDbfPDZ041gAVpgaV9U+IzLxRdf7EYYbJqvHSDJniSwBLacHg0RWIFlxRIElq1HIJ9/8Z1LAWzwqyV+lSx8AispL4D6tICBwJJRQBm106NumNYFfOfFZ0uK+wSW9coux+oD+hqBrVvHtcASSBhfIRCkE1jWpsPa75b40ZI0eLqHEJPrEw8O+F/r6Vb4BsP5f+6551xcBJZ8aUBDjyD69iVXqFCBVRHE9Ji1ZaJ8LicX3BlehBL+y0knxdKz4czwxijhTW+8Efw9vJjERQQ+O9O7Z0/XjjhWxbQjLizJgy1dSpD5Yr4FljiW1J0CsGXLFjfNpzEh/B9kwqJFi4Jbwgc1GX3K1ElKGJ2idEbpyowcOdJNm8kogYbtECDYMgksTMMItIjQfqOAgxFY2sN7NgJr6dKlwQsvvJBiswLLlhGx4huVEVhRpiH14QPVdoQSmDFjRhQWgeXLJ7D7qddgicAaP3682wI2v89mR2Z0msAnsGRkVIAymE6UERyb5gvbeDqBJVNtGFkrq8DStv79+8dsevrdQnvTB1DOtwZLXE9QYFVhTgyfMMeGN0FrL29m62qCJHPJXAss3DSFuOHK1xD09I0QwEiwjgO2Y7VlAClnPU0D6OglLlufwPr8889dGD6rsEUHK99cmxc+HM2ePduFNTBKL/uCdUyZFrlDkIhdCyy0o8tkEli6DTtlhvMMO8RtNgILwHSTbt8KLEwv6XQ9GuSrD8hGYAHw3v7888+r1D+PDyiNwAKwhs4nsACp99FHH43iOg3AmjeEJ/0xi2MhddSpU8crsPDBa73/soVDVCu+9Ggj0nU5/K4STyew8BCAMAQkrmENyZNJYOGbjbptnQZg1ErS8b/AWkWdX8IgjkMLLLGfeuqpLp4ksLCgP5egwCJJMq/MtcAiiMoG+UDyihUrooXquQQ+9VMeOBgHo/JR7IpCcXGxNZU7KLBIkswrKbAIgqgKoMAiSTKvpMAiCKIqgAKLJMm8kgKLIIiqAAoskiTzSgosgiCqAiiwSJLMKymwCIKoCqDAIkkyr6TAIgiiKoACiyTJvJICiyCIqgAKLJIk80oKLIIgqgIosEiSzCspsAiCqAqgwCJJMq9M940xgiCIQwUUWCRJ5pUUWARBVAVQYJEkmVdSYBEEcaijqKgoeGDChNj9T0iBRZJkuXPD2rVBSUmJvp0QBEEcUkg3egVSYJEkmRMOGjjQ3YDwlEcQBHEoYP/+/cGQIUMyiiuQAoskyZwSQ+i4GZEkSVZ2tm/fPlizYkXsPucjBRZJkiRJkmQ5kwKLJEmSJEmynEmBRZIkSZIkWc7MKLD27t4dK0SSJEmSJEn6Ce2UVmAtnz072LZuXawgSZIkSZIk6edHb77pNJRGisACOE1IkiRJkiSZPe3oFUCBRZIkSZIkeRDMSmABFFkkSZIkSZKZ6RNXgFdgASgAYl6RC99JkiRJkiQPLGiHNhKdlIREgSXAoi2phCRJkiRJsqrTLmj3IaPAIgiCIAiCIEoHCiyCIAiCIIhyBgUWQRAEQRBEOeP/jdqfgq+tOREAAAAASUVORK5CYII=>