# Trả lời phản biện của anh Dương: số đo thật (THEIA E3, 2026-11-14)

Quy ước: [đo] = có output script, [suy] = suy từ số đo, chưa kiểm trực tiếp. Mọi số dưới đây lấy từ output đã chạy, không phải ước lượng.

## 1. Phễu có chọn lọc không (node-level)

Nguồn: `eval_123.py` trên `log.txt` (169.650 process), `layer_123_candidate_result.csv` (cột `subject_uuid`), GT96.

| | Toàn dataset (GT96) | Sau cutoff (GT49) |
|---|---|---|
| Dân số process | 169.650 | 63.225 |
| Phễu giữ | 77.013 = 45,40% | 34.358 = 54,34% |
| GT lọt phễu | 20/96 = 0,2083 | 14/49 = 0,2857 |
| Ngẫu nhiên cùng số lượng, kỳ vọng | 43,6 (ngưỡng 95% = 52) | 26,6 |
| Lift (recall / retention) | 0,46× | 0,53× |
| P(ngẫu nhiên ≤ quan sát) | 4,6e-7 | 2,3e-4 |

Kết luận [đo]: ở mức node, seed của phễu **không giàu GT hơn ngẫu nhiên, mà thấp hơn** (có ý nghĩa thống kê). Không được viết "phễu chọn lọc" hay "phễu chứa GT hơn ngẫu nhiên". Recall story (95/96 và 49/49 ở L1–L3; 83/96 và 39/49 sau L4) đến từ BFS dựng story và L4, không phải từ chọn seed.

### 1b. Độ phủ của story L1–L3 (`eval_story_gt.py` trên `layer_123_story_result.jsonl`) [đo]

| | Toàn bộ story | Chỉ story có seed ≥ cutoff (luật anh Dương) |
|---|---|---|
| Số story / story TP | 7.156 / 8 | 2.038 / 3 |
| Story precision | 0,0011 | 0,0015 |
| Member process phân biệt | 110.336 = 65,0% toàn dataset | 36.863 (trong đó 33.172 sau cutoff = 52,5% process sau cutoff) |
| Recall GT96 / GT49 | 95/96 = 0,9896 / 49/49 | — / 49/49 = 1,0000 |
| GT kỳ vọng nếu phủ ngẫu nhiên cùng tỷ lệ | 62,4 (GT96) | 25,7 (GT49) |
| Lift (trần) | 1,52× (1,54×) | 1,90× (1,90×) |
| P(ngẫu nhiên ≥ quan sát) | 6,0e-17 | 1,85e-14 |

Đọc: L1–L3 giữ một nửa process sau cutoff mà không mất GT nào (đạt mức trần của lift), nhưng không phải bộ lọc mạnh; precision node của tập member ≈ 49/33.172 = 0,15%. P-value so với *tập con ngẫu nhiên cùng kích thước*, không phải *seed ngẫu nhiên + BFS* (member BFS là thành phần liên thông, GT tụ gần nhau). Không trích dòng lift 2,35× của góc nhìn thứ hai (trộn quần thể).

### 1c. Baseline "seed ngẫu nhiên + BFS" (cùng pipeline, cùng cờ; 5/5 lần bốc) [đo]

**Trạng thái kiểm lại (2026-11-15):** chạy lại với binary `0dcf0620` (có hash): lần bốc 1 ra đúng số cũ (29.505 story, 76,0% phủ, 57/96, 24/49, precision 0,0008). Lần bốc 2–5 trong bảng dưới là đợt chạy trước (15/11 lúc 01:29, trước khi `parse_detect_wl` được build lúc 08:28): script gọi `parse_detect` của thời điểm đó, file đã bị ghi đè nên không có hash; seed cố định theo `rng-seed`.

Mỗi lần: 77.013 seed ngẫu nhiên từ 169.643 node_id, không ONNX, `EDR_DISABLE_TABLE_D_COTOUCH=1`, có `EDR_L4_EXPORT_FILE`. Chấm bằng `eval_story_gt.py` trên toàn bộ story.

| | Story | Process phủ | Recall GT96 | Recall GT49 | Story precision |
|---|---|---|---|---|---|
| Thật (phễu) | 7.156 | 65,0% | **95/96** | **49/49** | 0,0011 |
| Ngẫu nhiên 1 | 29.505 | 76,0% | 57/96 | 24/49 | 0,0008 |
| Ngẫu nhiên 2 | 29.439 | 76,5% | 90/96 | 48/49 | 0,0006 |
| Ngẫu nhiên 3 | 29.410 | 76,0% | 44/96 | 14/49 | 0,0006 |
| Ngẫu nhiên 4 | 29.611 | 76,2% | 44/96 | 23/49 | 0,0009 |
| Ngẫu nhiên 5 | 29.738 | 76,0% | 76/96 | 34/49 | 0,0011 |

Trung bình ngẫu nhiên GT96 = 62,2/96 (64,8%, độ lệch chuẩn 20,3, khoảng 44–90); GT49 = 28,6/49 (58,4%, độ lệch chuẩn 13,0, khoảng 14–48). Không lần nào đạt 95/96 hay 49/49. Ngẫu nhiên phủ nhiều process hơn thật (76% so với 65%) nhưng recall thấp hơn ở 4/5 lần (lần 2 đạt 90/96). Chưa tính p-value (n=5).

Truy vết bằng `trace_random_hits.py` (log `trace_random_hits.log`, `trace_real_hits.log`):
- Recall GT49 do **một story lớn** quyết định. Thật: seed `106634` cho 47 member, 44 GT49. Ngẫu nhiên lần 2: seed `106682` cho 47 member, 44 GT49 (đạt 90/96). Lần 1, 3, 4 không có story đó (lần 5 chưa truy vết), GT49 trúng chỉ 14–24, gom từ nhiều story nhỏ (phần lớn là seed chính là GT, story 2 member).
- Cùng seed `106634` ra 47 member ở lần chạy thật nhưng chỉ 4–5 member ở lần ngẫu nhiên 3 và 4. Nguyên nhân [đọc `story_reconstruct.c` ~dòng 1960–2060]: BFS có luật ancestor-filter qua tư cách seed (`is_seed_node_id`): ancestor chỉ mở rộng sang child nếu child cũng là seed, và parent là seed thì được mở rộng đầy đủ. Hình dạng story vì vậy **phụ thuộc cả tập seed**, không chỉ seed gốc.
- Node `106682` (uuid `6D3588B4…`) có trong `layer_123_candidate_result.csv` (grep = 1 dòng) [đo], tức phễu giữ nó.
- Phễu thật: 8 story chứa GT, cả 8 seed là GT; 1 GT không vào story nào (`D618C22D…`).

Cách viết an toàn: "Với seed ngẫu nhiên cùng số lượng và cùng pipeline, recall GT96 dao động 44–90 (trung bình 62,2) trên 5 lần bốc, vì phụ thuộc việc các seed nằm trên đường tấn công có lọt mẫu hay không; phễu giữ các seed đó một cách xác định (95/96)." Không viết "phễu chọn lọc theo node" (lift node 0,46×). Không viết "luôn thắng": thật chỉ có 1 lần chạy, ngẫu nhiên có 1/5 lần đạt 90/96.

Chưa giải thích: số seed chính là GT trong mỗi lần bốc đếm được 16–26, trong khi kỳ vọng khoảng 43,6 (96 × 77.013/169.643). Chưa kiểm.


### 1d. Ablation: bỏ whitelist ingestion (`should_log_*`) [đo, 2026-11-15]

Cùng dữ liệu, cùng cờ, chỉ đổi nội dung `should_log_memory_event` / `should_log_transfer_event` (và bỏ early-drop mmap/mprotect trước `resource_pool_intern`).

| | Có whitelist (`0dcf0620`) | Bỏ whitelist (ablation, `6b0b56d9`, `rep_abl`) |
|---|---|---|
| Bản ghi candidate (`csv.DictReader`; `wc -l` ra 85.313 và 86.346) | 85.292 | 86.325 (+1,2%) |
| Process phân biệt qua phễu | 77.013 | 77.013 (tập uuid giống hệt, lệch 0) |
| Story / story chứa GT | 7.156 / 8 | 7.156 / 8 |
| Member process | 110.336 (65,0%) | 110.336 (65,0%) |
| Recall GT96 / GT49 | 95/96 / 49/49 | 95/96 / 49/49 |
| GT lọt phễu / lift node | 20/96 / 0,46x | 20/96 / 0,46x |

Kết luận: bỏ whitelist chỉ thêm khoảng 1,2% dòng candidate của cùng tập process; recall và story không đổi. Whitelist là bộ lọc volume, không quyết định kết quả.

## 2. [SYS] chấm trên đúng GT của Velox (51)

GT 51 của Velox = 51 subject trong `node_test_after_cutoff.csv` (patch của người dùng; 66 dòng = 51 subject + 9 file + 6 netflow). Cửa sổ tấn công `attack_to_time_window` bắt đầu 12:35:00 US/Eastern = 16:35:00Z (trùng cutoff, lệch 1 s).

- 51 = 49 GT có `start` ≥ cutoff + 2 process `/home/admin/profile` (`FB1F9E30`, `FC1F9E30`, cùng start 2018-04-10 18:56:39Z) [đo].
- `FC1F9E30` (node_id 4674) còn event sau cutoff (max_ts 2018-04-12 16:44:35Z) [đo]. `FB1F9E30` không có event sau cutoff trong `log.txt` [đo]; vì sao nằm trong danh sách GT: chưa giải thích.
- Velox flag cả hai process này: không.

| | TP / 51 | Recall | Wilson 95% |
|---|---|---|---|
| [SYS] (story seed sau cutoff) | 39 | 0,7647 | 0,632–0,860 |
| [SYS] (mọi story, gồm trước cutoff) | 41 | 0,8039 | — |
| Velox epoch 11 (3 run trong 5 run `results_summary.csv`) | 11 | 0,2157 | — |
| Velox epoch 11 (2 run trong 5 run `results_summary.csv`) | 36 | 0,7059 | — |

Fisher hai phía, 39/51 vs 36/51: p = 0,654 (không khác biệt). 39/51 vs 11/51: p = 4,2e-8. Số 36 và 11 là kết quả của các run Velox khác nhau (seed), nên không nên đem trung bình để kiểm định. (Cộng thêm run `47a79761` cũng 11 TP, tổng 6 run: 4 run 11 TP, 2 run 36 TP; xem bảng 6 run ở mục 3.)

## 3. So sánh alert Velox và [SYS] (process-level)

- Velox (hash `47a79761`, epoch 11): 1.732 alert, TP 11, FP 1.721, precision 0,0064 [đo]. 5 run `results_summary.csv`: flag 1.854–2.276, TP 11–36.
- [SYS] sau cutoff: 47 process được flag, 39 TP, 8 FP, precision 0,8298 (Wilson 0,699–0,911); recall 39/49 = 0,7959 [đo].
- Universe Velox: 59.330 node `subject` (patch `only_subject_nodes=True`); ngưỡng vẫn tính từ val loss của mọi loại node, chỉ bước chấm điểm lọc theo process [đo, đọc patch].
- ADP (PIDSMaker): diện tích dưới đường "% attack bị phát hiện theo precision"; attack coi là phát hiện khi **một node bất kỳ** của nó được xếp trên [đo, đọc code]. Run này có 1 attack id nên ADP ≈ precision tại hạng của node GT cao nhất; node GT cao nhất hạng 5–6/59.330 ở epoch 3 và 5 [đo].

Bổ sung nhau [đo]: trong 11 TP của Velox, chỉ 3 nằm trong story [SYS]; 8 cái còn lại (7 firefox + 1 mail) đều có trong `fn.txt` (danh sách FN của [SYS]) [đo: grep = 8/8]. Process `gtcache` (C2) Velox gần như không bắt (1/11 TP). Gộp lý thuyết: 39 + 8 = 47/51 = 0,9216 [suy; không phải một hệ thống].

Đặt phễu trước Velox, phần TP [đo, run hash 47a79761]: cả **11/11 TP của Velox đều nằm trong tập candidate của phễu** (`layer_123_candidate_result.csv`), nên phễu không làm mất TP nào của Velox. 15/51 GT của Velox nằm trong phễu (đúng với recall export 0,2941). 8 TP của Velox nằm ngoài story L4 là candidate của phễu mà L4 không flag, tức là FN của L4, không phải FN của phễu; Velox bắt được các điểm này.

Phần FP [đo, run hash 47a79761 epoch 11, ánh xạ index_id→UUID từ `subject_node_table` restore lại từ `theia_e3_cut.dump`, 278.363 subject, 1732/1732 alert có UUID]:

| | alert | TP | FP | precision | recall (GT51) |
|---|---|---|---|---|---|
| Velox thuần | 1.732 | 11 | 1.721 | 0,0064 | 0,2157 |
| Velox ∩ phễu | 198 | 11 | 187 | 0,0556 | 0,2157 |
| Velox ∩ story L4 (5 story) | 3 | 3 | 0 | 1,0000 | 0,0588 |

Đọc: chỉ 198/1.732 alert (11,4%) nằm trong phễu, trong khi phễu giữ 45,4% process toàn dataset (54,3% sau cutoff). FP giảm 1.721 → 187 (−89,1%), precision tăng 8,7×, không mất TP nào. Phễu loại FP của Velox mạnh hơn nhiều so với chia đều (≈780–930).

Giới hạn bắt buộc nêu: (a) FP tính theo GT51; process tấn công không nằm trong GT vẫn bị tính FP. (b) Velox ∩ phễu vẫn kém xa [SYS] về precision (0,83): phễu chỉ là bộ lọc, không thay L4.

**Sáu run Velox (epoch 11), đặt phễu trước, mỗi hash đo riêng [đo]** (`run_velox5_x_funnel.sh`, `velox5_x_funnel.log`; ánh xạ 5/5 file đạt 100% UUID):

| Hash | Velox thuần (alert / TP / FP / prec) | Velox ∩ phễu (alert / TP / FP / prec) | FP giảm | Velox ∩ story L4 (alert / TP / FP) |
|---|---|---|---|---|
| 47a79761 | 1.732 / 11 / 1.721 / 0,0064 | 198 / 11 / 187 / 0,0556 | 89,1% | 3 / 3 / 0 |
| a9b4c3d0 | 2.276 / 36 / 2.240 / 0,0158 | 560 / **13** / 547 / 0,0232 | 75,6% | 28 / 26 / 2 |
| 0a78fe94 | 1.872 / 36 / 1.836 / 0,0192 | 307 / **2** / 305 / 0,0065 | 83,4% | 36 / 35 / 1 |
| 7d4ec390 | 2.064 / 11 / 2.053 / 0,0053 | 363 / 11 / 352 / 0,0303 | 82,9% | 3 / 3 / 0 |
| 78d38503 | 1.854 / 11 / 1.843 / 0,0059 | 266 / 11 / 255 / 0,0414 | 86,2% | 3 / 3 / 0 |
| fd46053a | 2.132 / 11 / 2.121 / 0,0052 | 425 / 11 / 414 / 0,0259 | 80,5% | 4 / 3 / 1 |

Đọc [đo]:
- FP giảm 75,6–89,1% ở cả 6 run. Ở 4 run có 11 TP, không mất TP nào và precision tăng 5,0–8,7 lần (8,7× / 5,7× / 7,0× / 5,0×).
- **Ở hai run có 36 TP, phễu làm mất TP:** 36 → 13 (a9b4c3d0) và 36 → 2 (0a78fe94). Recall GT51 của Velox tụt 0,7059 → 0,2549 và 0,0392; với 0a78fe94 precision còn giảm (0,0192 → 0,0065) vì TP mất nhiều hơn FP loại. Phễu chỉ chứa 15/51 GT của Velox nên điều này được dự đoán trước. Không được viết "phễu trước Velox không mất TP".
- Mức "attack được phát hiện" (ADP, chỉ cần ≥1 node GT) vẫn giữ ở cả 6 run (còn ≥2 TP), nhưng ở 0a78fe94 chỉ còn 2.
- Velox ∩ story L4: precision 0,93–0,97 ở hai run 36 TP (recall 0,51 và 0,69). Story L4 là đầu ra của [SYS], nên đây là Velox xác nhận lại member story, không phải kết quả độc lập.

Giải thích vì sao recall mức node giữ ở run 11 TP và tụt ở run 36 TP (lời giải thích của tác giả về cơ chế; chưa phân loại từng TP trong tài liệu này):
- Phễu chỉ giữ process **tự chạm** một bit HEAVY (hoặc ≥2 bit BASE), chủ yếu là các process thực hiện execve và connect ra ngoài. 11 TP mà mọi run Velox đều bắt là loại này, nên phễu giữ lại đủ 11/11 (mục trên).
- 25 TP còn lại ở các run 36 TP là process con hoặc process chỉ có send/recv qua IP: bản thân không chạm bit nào của phễu, nên không có trong tập seed. Với các process này phễu làm mất TP ở mức node (36 → 13 và 36 → 2).
- Chúng vẫn được thu lại ở bước dựng story: BFS từ các seed (cha/ancestor và liên kết cùng resource) gom các process con và process dùng chung IP vào cùng story. Số đo ủng hộ cơ chế này [đo]: ở run `0a78fe94`, phễu giữ 2/36 TP nhưng story L4 chứa **35/36** TP; ở run `a9b4c3d0`, 13/36 so với 26/36 (cột "Velox ∩ story L4" trong bảng 6 run). Lưu ý: story L4 là đầu ra của [SYS] nên con số 35/36 cho thấy BFS gom được các process đó, không phải kết quả độc lập của Velox.
- Hệ quả cho cách đọc: recall mức node của phễu không phải thước đo của [SYS]; thước đo là recall story (95/96, 49/49 trước L4; 83/96, 39/49 sau L4).

Cách viết an toàn: "Đặt phễu trước Velox giảm FP 75–89% trên cả 6 run. Phễu giữ các process tự chạm ranh giới (execve, connect ra ngoài), nên recall mức node giữ nguyên ở các run 11 TP; ở các run 36 TP, các TP còn lại là process con hoặc process chỉ gửi/nhận qua IP, không nằm trong tập seed của phễu và bị mất ở mức node, nhưng được thu lại khi dựng story từ các seed đó (story L4 chứa 35/36 và 26/36 TP của hai run này)."

## 3b. L4 theo seed huấn luyện (Dương điểm 2) [đo, 2026-11-16]

Cùng binary, cùng cờ, cùng GT (GT96, GT49), chỉ đổi model. Binary là `~/theia_e3/parse_detect_wl` (`0dcf0620`): các script chạy (`run_l4_seed.sh` cho s1–s2, `run_l4_all.sh`, `run_l4_s3.sh`, `run_l4_45.sh` cho s4–s5) đều có dòng `sha256sum "$BIN" | grep -q 0dcf0620… || { echo "SAI binary"; exit 1; }`, tức chạy sai binary thì script thoát chứ không chạy [đọc script]. Log của từng lần chạy không ghi hash; năm lần chạy kết thúc 16/11 từ 07:46 đến 16:00 [đo: `ls -l` các `run_log_l4_s*.txt`]. Model: s1–s5 (HF `MinhHieu1234567890/SYSLAYER4MODELS1S5`, `s{N}/model_seed{N}.zip`). Mỗi model chạy full pipeline C (cấu hình L4-gated: `EDR_STORY_LITE_ONLY=1`, không `EDR_STORY_ALL_OCCURRENCES`; seed story = L4 `pred==1` và `score>=0,5`), chấm bằng `eval_story_gt.py` trên toàn bộ story. Số tuyệt đối từng seed:

| Seed | Story | Story TP | Story FP | Precision story | Member | Recall GT96 | Recall GT49 | Story sau cutoff (TP/tổng) |
|---|---|---|---|---|---|---|---|---|
| s1 | 4 | 4 | 0 | 1,0000 | 120 | 83/96 | 39/49 | 1/1 (seed 106687) |
| s2 | 7 | 4 | 3 | 0,5714 | 131 | 83/96 | 39/49 | 1/1 (106687) |
| s3 | 4 | 4 | 0 | 1,0000 | 118 | 82/96 | 38/49 | 1/1 (seed 106754) |
| s4 | 6 | 5 | 1 | 0,8333 | 128 | 86/96 | 42/49 | 2/3 (106687, 107385) |
| s5 | 5 | 4 | 1 | 0,8000 | 125 | 83/96 | 39/49 | 1/2 (106687) |

Trung bình ± độ lệch chuẩn mẫu (n = 5), phương sai mẫu trong ngoặc:

| Chỉ số | Trung bình ± SD (phương sai) | Khoảng |
|---|---|---|
| Story | 5,20 ± 1,30 (1,70) | 4–7 |
| Story TP | 4,20 ± 0,45 (0,20) | 4–5 |
| Story FP | 1,00 ± 1,22 (1,50) | 0–3 |
| Precision story | 0,841 ± 0,177 (0,0313) | 0,571–1,000 |
| Member process | 124,4 ± 5,4 (29,3) | 118–131 |
| GT96 trúng | 83,4 ± 1,5 (2,3); recall 0,869 ± 0,016 | 82–86 |
| GT49 trúng | 39,4 ± 1,5 (2,3); recall 0,804 ± 0,031 | 38–42 |

Model gốc (huấn luyện trước đó, không ghi seed, không tính vào trung bình): 83/96 và 39/49, 4 story TP, 1 story FP sau cutoff.

Đọc [đo]:
- Ba story TP trước cutoff (seed 2701, 2732, 4615; 12 + 24 + 8 = 44 GT96) có mặt ở cả 5 seed. Story sau cutoff có mặt ở cả 5 seed: seed 106687 ở s1/s2/s4/s5, seed 106754 ở s3 (cùng cụm tấn công; 40 member / 38 GT49 thay vì 42 / 39).
- Phần recall lệch chỉ do hai chỗ: s3 thiếu 1 GT sau cutoff, s4 có thêm story TP thứ 5 (seed 107385, 3 GT49).
- Story FP là phần biến thiên chính (0–3). Đều nhỏ (2–6 member, 0 GT). s2: 3 story FP trước cutoff (seed 2686, 3057, 3686, bảng C); s4 và s5 cùng một FP (seed 135882, 2018-04-13 00:57Z, 6 member); s1 và s3: không có FP.
- Precision story dao động 0,571–1,000 vì mẫu số nhỏ (4–7 story): thêm 1 story FP làm precision đổi 0,1–0,2. Trong paper báo số story tuyệt đối (TP/FP) kèm số này.

Giới hạn: n = 5 seed, một host; phần sau cutoff chỉ có một cụm tấn công nên không phải kiểm định held-out có ý nghĩa thống kê; đây là phương sai của seed huấn luyện, không phải của dữ liệu.

Cách viết an toàn: "Qua 5 seed huấn luyện, recall GT96 = 83,4 ± 1,5 (82–86), GT49 = 39,4 ± 1,5 (38–42); cả 3 story TP trước cutoff và story sau cutoff có mặt ở mọi seed; số story FP từ 0 đến 3."

## 3c. Ablation theo ranh giới RG1–RG5 (Dương điểm 6) [đo, 2026-11-17]

Định nghĩa: RG1 data→code, RG2 process→process, RG3 privilege, RG4 isolation, RG5 network external. L1 cho qua khi có ≥1 bit HEAVY hoặc ≥2 bit BASE khác nhau; mỗi bit HEAVY thuộc một ranh giới.

**(a) Ablation chạy thật** (cấu hình L1–L3, cờ `EDR_L1_RG_DISABLE`, 6 lần chạy mỗi binary: base + tắt RG1..RG5). Có hai binary, kết quả khác nhau ở RG1 và RG5, nên báo cả hai.

- **Binary cũ `81438c44…`** (`parse_detect`, file sửa 16/11 23:44, trước các lần chạy từ 17/11 01:13; `time.txt` của `rep_rg_base`, `rep_rg_off1`, `rep_rg_off5` ghi lệnh `~/theia_e3/parse_detect`; thư mục `rep_rg_*`). `objdump` không thấy hai điểm vá trong `process_state_update.c` (`mark_fresh_trigger_bits` dùng `layer1_is_heavy_bit_active`, `layer1_and_layer2` gọi `layer1_stats_event`), nên cờ tắt RG chỉ đổi `layer1_enough`, không chặn đường L2.
- **Binary vá `58e0e654…`** (`parse_detect_rg`, build 17/11 17:13, có đủ hai điểm vá; thư mục `rep_rgp_off1..5`): cờ tắt RG cũng chặn đường L2 khi bit heavy mới thuộc ranh giới bị tắt.

[đo 19/11/2026] đếm bằng `csv.DictReader` (`audit_binary_numbers.sh`), chấm bằng `eval_story_gt.py` (`rep_rgp_compare.txt`). Cột "base" lấy từ `rep_rg_base` (binary cũ, không chạy lại).

| | Candidate process (cũ → vá) | Bản ghi candidate (cũ → vá) | Story (cũ → vá) | Story TP | GT96 | GT49 |
|---|---|---|---|---|---|---|
| Nền (base) | 77.013 | 85.292 | 7.156 | 8 | 95/96 | 49/49 |
| Gỡ RG1 | 52.092 → 10.069 | 60.763 → 14.586 | 5.341 → 2.595 | 8 → 7 | 95/96 → 88/96 | 49/49 → 42/49 |
| Gỡ RG2 | giống nền (cả hai) | giống nền | giống nền | 8 | 95/96 | 49/49 |
| Gỡ RG3 | giống nền (cả hai) | giống nền | giống nền | 8 | 95/96 | 49/49 |
| Gỡ RG4 | giống nền (cả hai) | giống nền | giống nền | 8 | 95/96 | 49/49 |
| Gỡ RG5 | 77.013 → 67.420 | 85.292 → 75.178 | 7.156 → 5.006 | 8 → 6 | 95/96 → 75/96 | 49/49 |

Đọc [đo]:
- Với binary cũ, chỉ RG1 làm đổi số candidate (52.092) và không làm mất GT nào; RG2–RG5 không đổi. Điều này không có nghĩa RG5 dư thừa: binary cũ không chặn đường L2 theo RG.
- Với binary vá (cờ tắt RG chặn đường L2): gỡ RG1 còn 10.069 candidate, mất 7 GT96 và 7 GT49 (88/96, 42/49), còn 7 story TP. Gỡ RG5 còn 67.420 candidate, mất 20 GT96 (75/96), GT49 giữ nguyên 49/49, còn 6 story TP. RG2, RG3, RG4 không đổi (THEIA E3 không có hành vi chạm các ranh giới này, xem (b)).
RSS khoảng 1,47–1,49 GB (các lần chạy bản cũ); đo lại 2026-11-20 trên 6r: 1,55 GB (xem dòng "Số lệch" ở mục 4b). Bài ghi khoảng 1,2–1,6 GB tùy cấu hình.

- Hai cột "cũ" và "vá" cho RG1 và RG5 khác nhau vì binary khác nhau, không phải vì dữ liệu khác nhau; số của binary vá mới là số cho thấy tác động thật của từng ranh giới lên đường tới L2.

**(b) Thống kê L1 theo process, một lần chạy** (binary `58e0e654…`, build từ source hiện tại + whitelist; seed rỗng, chỉ xuất `rg_stats.csv`, 121.436 process, 90 phút 1 lần chạy):

| | process có bit RG | chỉ RG này | chỉ RG này và không qua base | GT có bit | GT chỉ RG này |
|---|---|---|---|---|---|
| RG1 | 70.328 | 69.350 | 63.550 | 18 | 10 |
| RG2 | 0 | 0 | 0 | 0 | 0 |
| RG3 | 0 | 0 | 0 | 0 | 0 |
| RG4 | 0 | 0 | 0 | 0 | 0 |
| RG5 | 10.564 | 9.586 | 3 | 10 | 2 |

Cả hai RG cùng có: 978 process (GT: 8). Process qua L1: 103.207 / 121.436. GT trong thống kê: 96/96 (118 UUID đọc từ hai file `node_*.csv`, 96 có trong log); GT tự qua L1: 85/96.

Đọc [đo]:
- RG2, RG3, RG4 không bật ở bất kỳ process nào của THEIA E3. Ablation "không đổi" ở ba ranh giới này vì dataset không chạm chúng, không phải vì chúng dư thừa. Không viết "mỗi ranh giới đều cần thiết" hay "gỡ ra không ảnh hưởng"; viết "THEIA E3 không có hành vi chạm RG2–RG4, ablation không đánh giá được".
- Chỉ-RG5-và-không-qua-base: 3 process toàn dataset và 2 process GT; RG1 chiếm gần hết. Đây là thống kê bit, không phải tác động lên đường L2: ở (a), gỡ RG5 bằng binary vá làm mất 20 GT96 (75/96).
- 65/85 GT pass L1 mà không có bit RG nào (15 nhờ base-bit của chính nó, 50 nhờ cha/cây con). **Đây là cờ L1, không phải leo thang**: cả 65 đều nằm NGOÀI `export.csv`. Trong `layer1_and_layer2()`, L2 chỉ chạy khi `mark_fresh_trigger_bits()` thấy bit heavy mới (dòng `if (!layer1_is_heavy_bit(b)) continue;`), nên process chỉ có base-bit không bao giờ tới L2/L4 bằng đường tự leo thang.
- Đối chiếu UUID giữa `rep_rg_stats/rg_stats.csv` (cột `rg` khác rỗng) và `rep_rg_base/export.csv` [đo 20/11/2026]: 79.914 process có bit RG; 77.013 candidate; giao = 77.013; **RG nhưng không candidate = 2.901**; candidate không có bit RG = 0. Tức mọi candidate đều đã vượt ít nhất một ranh giới (chiều "candidate thì đã vượt ranh giới" đúng), nhưng không phải mọi process vượt ranh giới đều thành candidate. Không viết "candidate khi và chỉ khi vượt RG1–RG5"; viết "chỉ process đã vượt ít nhất một ranh giới mới thành candidate". Phân loại 2.901 process đó [đo, `rg_stats.csv`]: cả 2.901 đều `pass1` (qua L1, bị loại ở L2), đều có bit RG1 (2.452 chỉ RG1, 449 RG1 và RG5, 0 chỉ RG5), đều `cross1`; không có process chỉ-RG5 nào bị loại. Chiếm khoảng 4% trong 70.328 process mang RG1. Số GT trong 2.901 process đó = 0: cả 20 GT có bit RG (18 RG1 ∪ 10 RG5, giao 8) đều là candidate. Vậy chỗ rò này không làm mất GT nào trên THEIA E3. Chưa đo: vì sao L2 không leo thang 2.901 process đó (cần verdict L2, hiện không có file ghi lại; phải chạy lại với `EDR_L2_DUMP_FILE`).
- Đo từ `rep_rg_base/export.csv` (`gt_where_from.py`): 20/96 GT tự leo thang, cả 20 đều có bit RG. 76 còn lại (65 pass L1 qua base + 11 không pass L1) vào story nhờ BFS/L3.
- 20 GT tự leo thang chia theo IV: iv1 = 5, iv13 = 10, chỉ bảng C (không IV) = 5; theo bảng (một uuid có thể ở nhiều bảng): A 4, B 1, C 9, D 10. IV13 là cầu heuristic (không phải bất biến kernel), nên không thể nói IV1 chiếm phần lớn.
- Đo IV trên cùng `rep_rg_base/export.csv` (`iv_measure.py`): 85.292 dòng, 77.013 subject phân biệt, 20 GT, precision subject 0,0003. Trong output của script chỉ iv1 và iv13 xuất hiện.

| Nhóm | subject | GT | precision subject |
|---|---|---|---|
| bảng A | 29 | 4 | 0,1379 |
| bảng B | 1 | 1 | 1,0000 |
| bảng C | 36.374 | 9 | 0,0002 |
| bảng D | 41.632 | 10 | 0,0002 |
| iv1 | 30 | 5 | 0,1667 |
| iv13 (= chỉ iv13) | 41.632 | 10 | 0,0002 |
| không IV nào (C thuần) | 35.351 | 5 | 0,0001 |
| có IV, không iv13 | 30 | 5 | 0,1667 |
| tất cả trừ iv13 | 35.381 | 10 | 0,0003 |

- Đọc: iv1 chọn 30 subject, 5 là GT. iv13 chọn 41.632 subject, 10 là GT. Bỏ iv13 khỏi export thì còn 10/20 GT tự leo thang trong export (số này là mức export, chưa đo ảnh hưởng lên story/recall cuối; chưa chạy).
- 20/96 là số GT tự leo thang (có mặt trong `export.csv` của `rep_rg_base`), UUID lấy từ hai file `node_*.csv` (`gt_where_from.py`, `iv_measure.py`). Số này không phải recall story (95/96).

Ghi chú của tác giả (không đo trong tài liệu này): RG2–RG4 không bật là do giới hạn của bộ dữ liệu THEIA E3; trên dữ liệu thực tế các ranh giới này sẽ xuất hiện nhiều hơn. Đường ≥2 base-bit khi bật làm FP rất cao, nên thiết kế chuyển trọng tâm sang đường heavy-bit. Lưu ý: binary dùng cho các số trong mục 3c vẫn còn đường ≥2 base-bit trong `layer1_enough`, nhưng đường này chỉ gắn cờ L1 và không dẫn tới leo thang (xem trên); bài cần nêu rõ điều đó.

[suy, chưa đo] Đường ≥2 base-bit nằm trong cửa sổ N=256 sự kiện (bit sống ≤ ~2N), nên có thể nhạy với padding, còn đường heavy-bit đơn điệu. Điều này chưa được kiểm bằng mimicry (mục 3d).

## 3d. Mimicry (Dương điểm 6) [đo, 2026-11-18 và 2026-11-20]

**Replay heavy (chèn đúng cửa sổ thời gian của seed).** `mimicry_replay.py`, `KIND=heavy N=1000000`: chèn lại sự kiện heavy (EXECUTE/CONNECT) của seed ngay sau sự kiện đầu tiên của chính seed, trong cửa sổ thời gian của seed, 1.000.000 lần mỗi seed (4 seed, 4.000.000 sự kiện, ~6% tổng 63,85 triệu record). Binary `81438c44…` (`parse_detect` trong `~/theia_e3_test_mimicry`), model s1, `EDR_STORY_LITE_ONLY=1`. Chạy 7h27m, RSS đỉnh 4,59 GB, exit 0. (Kết quả lấy từ lần chạy pipeline bắt đầu 17:20:28Z; lần bắt đầu 12:58:13Z bị ghi đè, xem mục 6.7.)

Story sau chèn so với baseline s1 (lần 2):

| Story (seed) | Baseline | Sau chèn |
|---|---|---|
| 2701 (`4E18B22A`) | 13 member / 12 GT | 13 / 12 |
| 2732 (`6818A92B`) | 58 / 24 | 19 / 18 |
| 4673 (`FB1F9E30`) | không có | 39 / 6 |
| 4615 (`C21F0F30`) | 9 / 8 | 9 / 8 |
| 106687 (`7235B0B4`) | 42 / 39 | 42 / 39 |
| Tổng | 4 story / 4 TP / 120 member / 83/96 / 39/49 | 5 story / 5 TP / 120 member / 83/96 / 39/49, precision story 1,0 |

Đọc [đo]:
- Recall GT96/GT49 và số member phân biệt không đổi (83/96, 39/49, 120). Không mất story TP nào, không có story FP mới.
- Story của seed 2732 bị tách làm hai: 58 member/24 GT thành 19/18 (seed 2732) và 39/6 (seed 4673), 19+39=58 và 18+6=24 (tổng giữ nguyên). Đọc từ `story_reconstruct.c`: ở baseline, BFS của seed 2732 chạm được seed 4673 nên 4673 nằm trong story 2732 và bị cache chéo seed (`g_node_covered`) bỏ qua, không có story riêng; sau chèn, BFS của 2732 không còn chạm 4673 (còn 19 member), cache không phủ 4673 nên 4673 chạy BFS riêng và ra story 39 member. So log baseline (`rep_l4_s1/run_log_l4_s1.txt`) với log replay (`rep_mimr_heavy_N1000000/run_log.txt`) [đo]: cả hai đều có đúng hai dòng `[network-cotouch]` cho seed 2732 (`+17 node`) và seed 4673 (`+38 node`), không có dòng `[PERF][hub-guard]` nào cho hai seed này, không có `[resource-cotouch]`. Vậy đường network co-touch và hub-guard cây cha–con không đổi giữa hai lần chạy; phần đổi nằm ở đường còn lại của BFS (cạnh `causal_graph_collect_component`, nơi nút siêu-hub bị bỏ qua mà không ghi log, hoặc cây cha–con). Chưa đo cạnh nào mất. Ghi nhận khác: log ghi `disk co 749879 event` cho 2732, `750664` cho 106687; `member_hard_cap=200000` là cap của L4, không phải của BFS (`MAX_MEMBERS=100000`).
- L4 trên hai seed bị spam thật (2732 và 106687): `SCORED`, score 1,000000, `pred=1`, agg_total_events lên 129.878 đến ~200.100 (2732) và ~200.000–200.245 (106687). Baseline: 2732 điểm 1,0 ở 5 và 27 sự kiện; 106687 điểm 0,982 ở 8 sự kiện. Spam không làm tụt điểm L4 trên hai seed này.

**Replay benign (sự kiện không-heavy).** `mimicry_replay.py`, `KIND=benign N=250000`: chèn lại sự kiện không-heavy của seed (MMAP, MPROTECT, OPEN, READ...) ngay sau sự kiện đầu tiên của chính seed, trong cửa sổ thời gian của seed, 250.000 lần mỗi seed (4 seed, tổng 1.000.000 sự kiện; heavy là 1.000.000 mỗi seed, tổng 4.000.000, nên hai lần không cùng cỡ). Lý do giảm N: lần đầu đặt N=1.000.000 mỗi seed, máy bị BSOD giữa chừng, chạy lại thì quá lâu nên bỏ (không có số). Binary `81438c44…` (`parse_detect` trong `~/theia_e3_test_mimicry`, sha256 đo 20/11/2026), model s1. Chạy 2h11m49s, RSS đỉnh 4,54 GB, exit 0.

| | Baseline s1 | Replay benign (250.000 mỗi seed) |
|---|---|---|
| Story / story TP / story FP | 4 / 4 / 0 | 6 / 4 / 2 |
| Member process phân biệt | 120 | 140 |
| Recall GT96 / GT49 | 83/96 / 39/49 | 83/96 / 39/49 |
| Precision story (toàn bộ) | 1,0000 | 0,6667 (4/6) |
| Story có seed sau cutoff (TP / tổng) | 1/1 | 1/2 (precision 0,5) |

Bốn story TP giữ nguyên: 2701 (13 member / 12 GT), 2732 (58 / 24, **không tách** như lần heavy), 4615 (9 / 8), 106687 (42 / 39). Hai story FP mới, đều không phải seed bị chèn, đều bảng D, 0 GT:
- seed 66518 (`3316F77F`, 2018-04-11 18:47:28Z, trước cutoff), 6 member, cmdline `du -sh /bin /boot /cdrom ...`.
- seed 106562 (`F534ECB3`, 2018-04-12 16:41:25Z, sau cutoff), 15 member, cmdline `/bin/bash -c /usr/bin/firefox`.

Số sự kiện đã vào component (`agg_total_events` trong `export.csv`; `events_truncated=0` ở mọi dòng, tức chưa chạm cap 200.000):

| Seed | agg_total_events (các dòng khác 1 và 5) |
|---|---|
| 2701 (`4E18B22A`) | 78.557 và 111.045 |
| 2732 (`6818A92B`) | 46.018 và 95.283 |
| 4615 (`C21F0F30`) | 11.571, 70.916 và 111.371 |
| 106687 (`7235B0B4`) | 2.377 và 6.305 |

Đọc [đo]:
- Spam không-heavy vào component của cả 4 seed (khác heavy, nơi chỉ 2/4 seed phình). Ba seed vượt xa cap đọc 8.192 sự kiện của L4; 106687 chỉ 2.377–6.305 (dưới 8.192) dù chèn 250.000 bản (chưa giải thích vì sao ít).
- L4 vẫn flag cả 4 seed (cả 4 story TP còn nguyên) và recall GT không đổi (83/96, 39/49).
- Precision story giảm 1,0000 → 0,6667 vì có 2 story FP mới (4/6 story; Wilson 95% = 0,30–0,90, mẫu số quá nhỏ nên báo số tuyệt đối 2 FP, không báo tỷ lệ như số ổn định). Đây là tác động có thật của lần chạy này, không được viết "mimicry không ảnh hưởng".
- Nguyên nhân 2 story FP [đo, so `export.csv.infer_status` baseline s1 với lần benign]: điểm L4 của hai UUID này tăng từ ~0 lên ~1 khi số sự kiện vào component tăng:

| UUID (seed) | Baseline s1 (agg_total_events → điểm) | Replay benign |
|---|---|---|
| `3316F77F` (66518) | 1 → 0,000000 (pred 0) | 1 → 0,000000; **18.797 → 0,999702 (pred 1)** |
| `F534ECB3` (106562) | 531 / 1 / 10 → 0,001169 / 0,001169 / 0,000002 (pred 0) | 1.787 đến 15.726 → 0,001169 (pred 0); **17.160 đến 118.527 → 1,000000 (pred 1)** |

  Trong cùng một lần chạy và cùng binary, `F534ECB3` cho đúng điểm baseline (0,001169) khi component còn ≤ 15.726 sự kiện và 1,000000 khi ≥ 17.160 sự kiện. Vì vậy điểm đổi do số sự kiện trong component (baseline chỉ có 1–531 sự kiện), không do khác binary (`81438c44` so với `0dcf0620`). Chưa đo: sự kiện chèn đi vào component của hai UUID này qua cạnh nào (chưa dump thành viên component). Hai UUID này không phải seed bị chèn.
- Ý nghĩa: chèn sự kiện không-heavy vào cửa sổ của seed tấn công không làm L4 bỏ lỡ seed (recall giữ nguyên), nhưng làm component của process lân cận phình, và L4 chấm các process đó ~1,0, sinh 2 story FP.

Giới hạn bắt buộc nêu (replay heavy):
- **Chỉ 2/4 seed bị spam thật.** Với 2701 (`4E18B22A`) và 4615 (`C21F0F30`), số sự kiện chấm điểm vẫn nhỏ (1 và 31; 1 và 21; số sự kiện gốc theo Pass A: 25 và 22), `events_truncated=0`, dù manifest ghi chèn 1.000.000; 2732 lên 129.878–200.114 và 106687 lên 200.000–200.245 (`events_truncated=1`). Phép đo chứng minh CONNECT lặp không làm component phình. Cả 4 seed đều có 1 sự kiện CONNECT; chỉ 2732 và 106687 có thêm 1 sự kiện EXECUTE (tập heavy để nhân bản: 2 so với 1). Giải thích phù hợp (đọc từ `process_state_update.c`, chưa kiểm bằng chạy thực): pipeline có RLE cho network (`net_event_same()`: cùng syscall, behavior, src, dst, port, address family; giống hệt liên tiếp thì chỉ `count++`, không thêm vào `event_history`), nên CONNECT giống hệt liên tiếp bị gộp, còn EXECUTE lặp (trong các đoạn code đã đọc không thấy RLE cho exec) làm history phình. `export.csv` không có cột `count` nên chưa chứng minh trực tiếp cơ chế RLE; chưa rõ `agg_total_events` đếm sau RLE hay cách khác. Chưa giải thích: 1.000.000 bản sao nhưng chỉ ~750.000 sự kiện trên đĩa cho 2732 và 106687. Chưa kiểm: kẻ tấn công xen kẽ các đích/port khác nhau để RLE không gộp được; chưa xác nhận kỹ exec không có RLE trên toàn file. `log.txt` là bảng tóm tắt theo process, không dùng để tra sự kiện. Không được viết "4 seed đều chịu 1M sự kiện".
- Chỉ kiểm tra spam heavy (đường heavy-bit). Đường ≥2 base-bit chỉ gắn cờ L1 (không leo thang, mục 3c) nên không ảnh hưởng điểm leo thang. Spam sự kiện không phải heavy (MMAP/MPROTECT/OPEN/READ): đã chạy replay đúng thời điểm (`KIND=benign`, N=250.000 mỗi seed, xem "Replay benign" ở trên); N=1.000.000 mỗi seed chưa chạy được (BSOD, rồi quá lâu).
- Một model (s1), một lần chạy mỗi loại (heavy, benign), một host. Không có bằng chứng thống kê. Heavy (4M) và benign (1M) khác cỡ, không so trực tiếp.
- Số sự kiện L4 thực sự đọc: cap 8.192 sự kiện sớm nhất theo thiết kế; chưa đo tỷ lệ spam trong 8.192 sự kiện đó.
- Căn cứ thiết kế (Goyal et al., NDSS'23): kẻ tấn công chỉ thêm được sự kiện, không bớt được.

Cách viết an toàn: "Chèn 1.000.000 sự kiện EXECUTE/CONNECT lặp lại vào cửa sổ của từng seed (2 trong 4 seed nhận được spam), L4 vẫn chấm điểm 1,0 cho cả hai seed và tập GT được phủ không đổi (83/96, 39/49); một story tách thành hai nhưng tổng member/GT không đổi. Với 250.000 sự kiện không-heavy mỗi seed (tổng 1.000.000), cả 4 seed phình (2.377 đến 111.371 sự kiện), L4 vẫn flag cả 4 seed và recall không đổi (83/96, 39/49), nhưng xuất hiện 2 story FP mới ở hai process lân cận (precision story 1,0 → 0,667): component của chúng phình từ ≤ 531 lên 18.797–118.527 sự kiện và điểm L4 tăng từ ~0 lên ~1; đường sự kiện chèn đi vào component đó chưa đo."

## 4. Các điều phải ghi trong paper

1. 77.013 là số process phân biệt. Số dòng candidate là 85.292 bản ghi CSV (`csv.DictReader`); `wc -l` cho 85.313 vì 6 bản ghi có field chứa xuống dòng (21 dòng vật lý dư), cùng một file `export.csv` [đo, `audit_binary_numbers.sh`]. Cùng tập 77.013 process (đã so tập uuid: chung 77.013, lệch 0).
2. Không dùng chữ "phễu chọn lọc"; ghi lift 0,46× và nói recall đến từ BFS + L4.
3. Mẫu số: Velox 51, [SYS] 49 (giải thích như mục 2).
4. Velox: train gồm 2018-04-10 (ngày có Firefox backdoor), val 04-09, test 04-12/13, 04-11 không dùng; ngưỡng từ val loss mọi loại node.
5. Luận điểm thiết kế phễu: phễu là điều kiện cần, không phải bộ xếp hạng. Seed của phễu chủ yếu là connect/execve (hành vi attacker bắt buộc phải làm nhưng benign cũng làm rất nhiều), nên ở mức node phễu không giàu GT hơn ngẫu nhiên (lift 0,46×) và precision node thấp là điều dự kiến. Phễu bảo toàn (không bỏ sót vì chạm ranh giới kernel là bắt buộc), L4 mới là bước cô đọng. Không dùng lift node của phễu để đánh giá chọn lọc.
6. Đơn vị đếm so với Orthrus/Velox: họ tính "phát hiện cuộc tấn công" khi bắt được ít nhất một node (ADP: attack detected nếu có node GT bị flag). [SYS] chấm từng node trong GT (49 sau cutoff), khắt khe hơn. Phải nêu rõ để không so nhầm số.
7. Độ phủ story: 65,0% process (sau cutoff 52,5%), GT trúng 95/96 so với kỳ vọng 62,4 nếu GT nằm ngẫu nhiên (lift 1,52× so với trần 1,54×). P-value so với tập con ngẫu nhiên cùng kích thước, chưa so với seed ngẫu nhiên + BFS.
8. Giới hạn: baseline "seed ngẫu nhiên + BFS" đã chạy 5 lần bốc (mục 1c); Velox ∩ phễu đã đo 6 run (mục 3).
9. ADP của run này không so được với 0,72 của bài gốc (1 attack id; ADP phụ thuộc hạng node GT đầu tiên), và không coi là lỗi tái lập. Không so với 0,72 vì ngày train và ngày test khác nhau: siêu tham số (lr 0,001, hid 320, 12 epoch, emb 256, dropout 0,3) được tinh chỉnh theo ngày cho kết quả tốt nhất rồi áp sang ngày test khác nên điểm lệch; PIDSMaker cũng nêu điều này.
10. Velox: hyperparameter giữ nguyên bản gốc THEIA_E3 (lr 0,001, hid 320, 12 epoch, emb 256, dropout 0,3), cùng cửa sổ test với [SYS] (04-12/13). Chỉ khác split train (do dữ liệu cục bộ) và `only_subject_nodes=True`.
11. Giới hạn về phạm vi dữ liệu (ba bộ DARPA TC: THEIA E3, THEIA E5, TRACE E3; không có CADETS): phễu L1–L3 và Group A dùng đường dẫn và ngữ nghĩa syscall của Linux (`/etc/shadow`, `LD_PRELOAD`, `/proc/mem`, `/dev/kmem`, `DIR_MODPROBE_D`...), do đó cả ba bộ đều là Linux. CADETS E3/E5 chạy FreeBSD, không có các đường dẫn này và tên syscall khác, nên không chạy được mà không viết lại Group A và bảng syscall cho FreeBSD. Đó là công việc thiết kế, không phải chạy thêm dữ liệu, nên để lại làm hướng mở rộng. Hệ quả: kết luận chỉ áp dụng cho host Linux. Ba bộ dùng ba host/collector khác nhau (THEIA, TRACE) và hai đợt thu (E3, E5), nhưng không đủ để khẳng định tổng quát hóa sang HĐH khác.

## 4b. Limitations đề xuất

- **L4 theo seed huấn luyện (Dương điểm 2):** đã huấn luyện 5 seed (s1–s5), mỗi seed khoảng 2,5–3 giờ trên 2×T4 (60 epoch = 50 + 10 HNM; không phải 2,7 ngày) và chạy lại full pipeline C với từng model; số liệu ở mục 3b. Giới hạn: n = 5 seed trên một host và một bộ dữ liệu train; phương sai chỉ phản ánh seed huấn luyện, không gồm biến thiên của dữ liệu train. Chưa kiểm: nhãn mâu thuẫn ở các khoá (syscall, path) trong dữ liệu train, và việc relabel IP C2 đã áp vào `chunks-v3` hay chưa. Kết quả chuyển giao giữa host (LOSO) thấp hơn trong cùng host; báo đầy đủ, không loại số xấu.
- **Mimicry và ablation theo ranh giới (Dương điểm 6):** phễu trước Velox (mục 3), ablation whitelist (mục 1d), ablation RG1–RG5 (mục 3c) và mimicry replay trên THEIA E3 thật (mục 3d) đã đo. Giới hạn: RG2–RG4 không bật trên THEIA E3 nên không đánh giá được; mimicry replay có hai loại: heavy (EXECUTE/CONNECT lặp, 4M sự kiện; 2/4 seed bị spam thật, theo code `process_state_update.c` CONNECT giống hệt liên tiếp bị RLE gộp, chưa kiểm bằng chạy thực) và không-heavy (1M sự kiện; 4/4 seed phình, recall không đổi nhưng thêm 2 story FP ở process lân cận do component phình, điểm L4 từ ~0 lên ~1; đường chèn vào component chưa đo); một model, một lần chạy mỗi loại, hai loại khác cỡ. Đường ≥2 base-bit chưa được kiểm padding riêng (chưa đo bit nào bị bật bởi sự kiện chèn). Bộ nhớ: peak RSS 1,47–1,49 GB (lần RG), 4,59 GB (heavy, component phình tới 200.000 sự kiện, 7h27m) và 4,54 GB (benign, 2h11m).
- **Velox theo split gốc (Dương điểm 3):** không có phần cứng để chạy lại Velox với đúng split của bài gốc, nên không kiểm được ADP 0,72. Hyperparameter giữ đúng bản THEIA_E3; khác ở split train và `only_subject_nodes=True`. ADP của các run ở đây không so được với 0,72.
- **Orthrus (Dương điểm 3):** không chạy Orthrus vì tốn khoảng 135 GB RSS (theo docx v13); không có so sánh định lượng. Nếu trích số công bố của Orthrus thì ghi rõ khác split, khác GT, khác đơn vị đếm (attack detected so với từng node).
- **Chỉ một tấn công có số đầy đủ (Dương điểm 1):** số story L1–L3 + L4 kèm GT và hash binary chỉ có đầy đủ ở THEIA E3, nơi tập test chỉ có tấn công Browser Extension. THEIA E5 (cross-host, train trên E3+TRACE, test E5, ngưỡng 0,5 lấy từ E3): story L1–L3 recall 63/64 (0,9844); L4 precision 0,3891, recall 0,7031 (45/64), F1 0,5009 [đo; log đầy đủ các lần chạy THEIA E5 đã có]. Mức nâng precision story do L4: THEIA E3 từ 0,0011 (L1–L3, mục 1b) lên 0,9667 (29 TP / 1 FP, cùng host); THEIA E5 cross-host từ 0,0001 (L1–L3, [đo]) lên 0,3891. TRACE E3 chưa chạy cả pipeline. Không chạy thêm được: không có GPU để huấn luyện lại và ổ đĩa không đủ chứa TRACE E3. Đây là giới hạn tài nguyên; câu được phép viết là "có số đo đầy đủ trên một tấn công THEIA E3, và kết quả cross-host trên E5 ở mức precision 0,39, recall 0,70", không viết "tổng quát hóa".
- **Tái lập và tài nguyên (Dương điểm 7):**
  - Model chính không ghi seed; chỉ năm model phụ s1–s5 có seed (mục 3b). 
  - TRACE E3 chạy bằng cùng binary với THEIA E3, nên hash binary THEIA E3 ở mục 4c áp dụng cho TRACE E3; không có binary riêng.
  - Bộ nhớ: đỉnh RSS khi chạy ngoại tuyến cả dataset THEIA E3 `ta1-theia-e3-official-6r.bin` (169.650 process, ~63,85 triệu record) nằm trong khoảng **1,2–1,6 GB tùy cấu hình**: lần đo 2026-11-20 với cấu hình đầy đủ (`EDR_STORY_ALL_OCCURRENCES` + export + `CG_VERIFY`, không model) là 1.550.232 KB (1,59 GB thập phân, 1,48 GiB); các lần RG (mục 3c) 1,47–1,49 GB. Mimicry (mục 3d) cao hơn vì component phình: 4,59 GB (heavy) và 4,54 GB (benign). Ngưỡng 20 MB mỗi host là ngưỡng của agent chạy runtime trên một host; hai con số đo hai thứ khác nhau (batch trên cả dataset so với runtime theo host), không so trực tiếp được. State sau khi chạy: 685,9 MB (`processes.bin`) + 31,8 MB (pool) ≈ 718 MB cho cả dataset, không đạt 20 MB; binary không ghi `hostId` nên không đo được state theo từng host. Bài không tuyên bố đạt 20 MB và bỏ câu "to meet these requirements"; chỉ nêu state nhỏ hơn khoảng 14× so với CDM gốc. Chi phí runtime chưa đo.
- **CADETS E3/E5 (Dương điểm 4):** xem mục 4, số 11.
- **Số lệch (Dương điểm 5):**
  - 27+22+44=93 so với 95: phép cộng đến từ bản docx cũ đã bị xóa; bản hiện tại không còn. 95/96 là tổng GT trúng trong 8 story: 44+24+12+8+3+2+1+1 (`trace_real_hits.log`). 27 và 22 là GT của TRACE E3, không cộng được với THEIA E3.
  - 15,3 GB (Bảng 9, THEIA E3 L1–L3) là cột "Dữ liệu đọc và ghi", không phải kích thước file; chênh với input gồm ghi (`processes.bin` 0,69 GB, `resource_pool.bin` 0,03 GB, các file export) và đọc lại; chỉ cần chú thích dưới bảng. Số "10,6 GB" không có trong v13.
  - **Kích thước dữ liệu và state (Table 3, §3.6), tất cả đo trên bộ THEIA E3 `ta1-theia-e3-official-6r.bin` (file chính + `.1`–`.12`), đo 2026-11-20:**
    - Input `.bin`: 9.894.818.787 byte = **9,89 GB**. Bài ghi 10,99 GB (10.989.583.931 byte) = 9,89 GB + file `.zip` 1.094.765.144 byte cùng thư mục, tức cộng nhầm file nén; sửa thành 9,89 GB.
    - JSON CDM của cùng bộ 6r (`ta1-theia-e3-official-6r.json.tar.gz`, 13 file, cộng cột kích thước của `tar -tzvf`): 47.995.994.407 byte = **48,00 GB** (44,70 GiB). Bài ghi 33 GB là sai. Cả hai (`.bin` 9,89 GB và JSON 48,00 GB) ghi vào bài.
    - State, binary chính `parse_detect_wl` `0dcf0620` (có whitelist): `processes.bin` 685.861.150 byte (685,9 MB), `resource_pool.bin` 31.782.546 byte (31,8 MB), spill 131.287.287 byte (131,3 MB). Tỉ lệ so với 9,89 GB: **14,4×** (chỉ processes), 13,8× (+pool), 11,7× (+spill); so với JSON 48,00 GB: khoảng 70× (processes). Bài ghi 675+5 MB và 16× là số của bộ `1r` (export-only, chạy 2026-10-09), không dùng cho 6r; sửa thành 686+32 MB (đoạn mở đầu "khoảng 680 MB" thành khoảng 720 MB) và ≈14×.
    - Không whitelist (`parse_detect_abl` `6b0b56d9`, `rep_abl`; trùng `rand_run_1..5`): `processes.bin` 1.138,5 MB, pool 31,8 MB, spill 131,3 MB; tỉ lệ 8,7× / 8,5× / 7,6×. Không dùng cho Table 3 vì số chính của bài chạy bản whitelist; có thể nêu như cột so sánh.
    - `parse_detect` `81438c44` KHÔNG phải bản bỏ whitelist: `nm`/`objdump` cho `should_log_memory_event` 72 lệnh, `should_log_transfer_event` 57 lệnh (0x1a530/0x1bb40), giống hệt `parse_detect_wl`; `parse_detect_abl` chỉ 12 và 5 lệnh. Chạy song song `measure_wl.sh`/`measure_nowl.sh` trên 6r cho kết quả y hệt từng byte (20.526.652 events written, peak RSS 1.550.232 / 1.550.300 KB, 1:53:21 / 1:53:08; thời gian bị kéo dài vì chạy song song).
    - Phân rã state theo miền sự kiện và theo syscall: xem "Phụ lục: chi phí state theo miền sự kiện". Raw log: mục 6.8.
    - Thời gian chạy trong bài: 6.400 s (L1–L3) và 5.900 s (L1–L4). Lần đo 2026-11-20 của L1–L3 (`measure_wl.sh`, cấu hình `ALL_OCCURRENCES`) là 1:53 (khoảng 6.800 s) nhưng chạy song song hai bản nên bị kéo dài, không dùng để kết luận. Giải thích của tác giả cho việc L1–L4 nhanh hơn L1–L3 (đọc từ cấu hình script, chưa đo): cấu hình L1–L3 bật `EDR_STORY_ALL_OCCURRENCES=1` nên dựng story cho toàn bộ 77.013 seed (cả TP lẫn FP); cấu hình L1–L4 (`run_l4_seed.sh`) không bật cờ này, seed = các process L4 `pred==1`, `EDR_STORY_LITE_ONLY=1`, nên chỉ dựng story cho candidate nghi ngờ. Đề nghị ghi bài theo giờ (khoảng 1,8 giờ và 1,6 giờ) kèm câu giải thích này. Đang chạy lại từng cấu hình riêng, lần lượt không song song (`run_timing_solo.sh` gọi `measure_wl.sh` rồi `measure_l4.sh`, model s1, binary `0dcf0620`); kết quả chưa có.
- **Nguồn gốc binary (Dương điểm 5):** code đóng băng ở commit `f07baf0aa5dca39890755623e262cb9560852a64` (nhánh `update-firefox-csv`); hash binary nằm trong `BINARIES_SHA256.txt` (commit `8cfc190`). Các số THEIA E3 trong tài liệu này được đo lại ngày 2026-11-15 bằng binary build từ source có whitelist ingestion (`should_log_memory_event` / `should_log_transfer_event`) với sha256 `0dcf062091d54610a9beccd0fa07f4e34ff952423a1835bd1913500aedd8a5b2`. Lần chạy này tái tạo đúng số cũ: 77.013 process phân biệt (tập uuid trùng hoàn toàn bản cũ), 7.156 story, 8 story TP, recall GT96 95/96, GT49 49/49, member 110.336, lift 0,46×; số bản ghi candidate 85.292 (đếm bằng `csv.DictReader`; `wc -l` ra 85.313 do 6 bản ghi có field xuống dòng). Binary `6b0b56d9…` (`parse_detect_abl`, file sửa 15/11 12:44, do `run_abl.sh` copy từ `$SRC/build/parse_detect` rồi chạy `rep_abl`) chỉ dùng cho cột bỏ whitelist (mục 1d), không dùng cho số nào khác; trước khi đo lại, source E3 ở trạng thái ablation (bỏ gate) lẫn với bản whitelist nên các lần chạy không đồng nhất, đã bị loại. THEIA E5 dùng binary build riêng (`93c66184…`, build 13/09), chỉ có một bản build này. TRACE E3: ngoài phạm vi các số liệu trong tài liệu này (chưa có số cuối); TRACE E3 và THEIA E3 cùng nguồn code và cùng binary. Model chính không ghi seed (chỉ s1–s5 của năm model phụ có seed). Lưu ý khi đọc code: gate whitelist ở ingestion trái NT-3/NT-4 (project đã yêu cầu gỡ ngày 2026-09-30); số liệu trong bài đo ở bản có whitelist; đây là bộ lọc volume, không phải logic phán quyết của L1–L3.

- **Tái tạo đầu-cuối (Dương điểm 5):** ngày 2026-11-15 các số THEIA E3 được chạy lại bằng binary có hash `0dcf0620` và khớp số cũ (mục 4b phía trên, mục 1d ablation); log lần chạy này được giữ (`run_wl_all.log`, `run_abl.log`). Lần bốc ngẫu nhiên 1 khớp từng số; lần 2–5 là bài đo riêng (đợt chạy trước, seed cố định theo `rng-seed`). Các lần chạy trước bằng binary khác không còn log.

## 4c. Hash binary dùng cho các số liệu (sha256) [đo]

| Binary | sha256 | Dùng cho |
|---|---|---|
| `~/theia_e3/parse_detect_wl` (có whitelist, nguồn của số THEIA E3 trong bài) | `0dcf062091d54610a9beccd0fa07f4e34ff952423a1835bd1913500aedd8a5b2` | Mục 1–3 (số chính); mục 3b: xem ghi chú đầu mục |
| `build_wl/parse_detect` = `~/theia_e3_test_mimicry/parse_detect` (source hiện tại + 3 file whitelist + patch RG) | `81438c442e5a783a8e2011b9170e3dbcfca9e2a52fc5017723bd7678b00ed321` | Mục 3c(a) (ablation RG 6 lần chạy) và mục 3d (mimicry replay, 17–18/11/2026) |
| `build_wl/parse_detect` build lại ngày 17/11, copy thành `~/theia_e3/parse_detect_rg` | `58e0e654432f6579400ee9af1285aaa776c746f2177ed27bf54f06e4b56e93ee` | Mục 3c(b) (thống kê RG một lần chạy) và ablation RG chạy lại `rep_rgp_off1..5` (mục 3c(a)) |
| `theia_trace_e3/build/parse_detect` (source ablation, không whitelist; sha in ra lúc `make abl`) | `190db76dfa7abfccc7ef22ef8c9bdd0ab5440402041c5da419451bf9384fff7a` | Bản build khác, ở thời điểm khác; không dùng cho số nào trong tài liệu |
| `~/theia_e3/parse_detect_abl` (file sửa 15/11 12:44; `run_abl.sh` dòng 5 copy vào, dòng 12 chạy `rep_abl`) | `6b0b56d90aca1be70cd3288f3e42e63421d82d11e72f01204308dbcf5e5630d2` | Cột bỏ whitelist của mục 1d (`rep_abl`) [đo: `run_abl.sh`]; không dùng cho số chính |
| `~/theia_e3/parse_detect` (cùng hash với bản mimicry) | `81438c442e5a783a8e2011b9170e3dbcfca9e2a52fc5017723bd7678b00ed321` | Giống `theia_e3_test_mimicry/parse_detect` (đo 18/11/2026) |
| Binary THEIA E5 `93c66184…` (build 13/09) | tiền tố 8 ký tự | THEIA E5, ngoài phạm vi tài liệu này |

Mã nguồn đóng băng: commit `f07baf0aa5dca39890755623e262cb9560852a64` (nhánh `update-firefox-csv`); danh sách hash trong repo: `BINARIES_SHA256.txt` (commit `8cfc190`). Hash khác nhau giữa các dòng trên là bình thường vì source khác nhau (thêm patch RG); tính tương đương của đường chính được chứng minh bằng hồi quy với `rep_wl` (file story 58.786 dòng, export 85.292 bản ghi, giống hệt; đo bằng `audit_binary_numbers.sh`).

## 5. File kèm theo

`eval_123.py`, `count_proc_cutoff.py`, `log_ts_by_uuid.py`, `velox_extract.py`, `map_velox_gt.py`, `inspect_velox_art.py`, `velox_gt51_uuids.txt`, `velox_gt66_nodes.csv`.

## 6. Log gốc của các lần chạy

Mục 6.1–6.5 là nội dung nguyên văn của file log/output gốc, không chỉnh sửa; mục 6.6–6.7 là output nguyên văn các lần dán trong phiên 17–18/11/2026 kèm vài ghi chú của người ghi (đánh dấu "Ghi chú" hoặc tiêu đề đoạn, không nằm trong khối mã). Không đính kèm: `trace_real_hits.log` (có trên đĩa, đầu file: 7156 story, 51630 stub, GT96 95/96, GT49 49/49) và log các lần chạy lại E3/E5/TRACE E3 bằng binary đóng băng (bị ghi đè).

### 6.1 Baseline seed ngẫu nhiên + BFS (5 lần bốc) — script và output đầy đủ

Chạy trong ~/theia_e3, 15/11/2026 01:29 (file đính kèm của phiên làm việc). Bản này chạy đủ 5 lần bốc.

```
enovo@DESKTOP-9J5D6AQ:~/theia_e3$ #!/usr/bin/env bash
# Baseline "seed NGAU NHIEN + BFS" cho THEIA E3. Chay trong ~/theia_e3.
# BUOC 0 (kiem script): cham story that, phai ra ~95/96 va ~7156 story:
#   python3 eval_story_gt.py --stories layer_123_story_result.jsonl --log log.txt \
#     --gt node_Browser_Extension_Drakon_Dropper.csv node_Firefox_Backdoor_Drakon_In_Memory.csv --label seed_that
# BUOC 1: sinh 5 file seed (da co sau lenh nay)
# BUOC 2: moi file seed = 1 lan chay pipeline day du (BFS chay inline cuoi run, phu thuoc tap seed -> KHONG gop duoc).
#         Thay PIPE_CMD bang DUNG dong lenh ban da chay ra layer_123_story_result.jsonl (binary + danh sach .bin).
set -euo pipefail
cd ~/theia_e3
G="node_Browser_Extension_Drakon_Dropper.csv node_Firefox_Backdoor_Drakon_In_Memory.csv"

python3 gen_random_seeds.py --log log.txt --n 77013 --pop all   --draws 5 --out-prefix seeds_rand_all
python3 gen_random_seeds.py --log log.txt --n 34358 --pop after --draws 5 --out-prefix seeds_rand_after

# CAU HINH (doc tu source avro_main.c / story_reconstruct.c, khong doan):
#  - SEEDS_FILE thang ALL_OCCURRENCES/autoseed (if/else-if, avro_main.c:936) -> seed = DUNG file ngau nhien.
#  - KHONG set EDR_L4_SEQ_ONNX/VOCAB -> auto-story seed = 0 (avro_main.c:1083), khong tron them seed L4.
#  - PHAI set EDR_L4_EXPORT_FILE: story_reconstruct_run(l4_export_path) doc CSV de build resource_index/touched_resource
#    (story_reconstruct.c:2519); thieu CSV thi mat nhanh join A/D => khong cung dieu kien voi lan chay that.
#  - EDR_DISABLE_TABLE_D_COTOUCH=1 (D tat) va file co-touch mac dinh TAT (khong set EDR_ENABLE_FILE_COTOUCH);
#    network co-touch luon BAT (live). XAC NHAN tu chat 'layer 1 2 3 thuan': lan chay ra all_occ_theia_e3_v4_noD_1.jsonl dung EDR_DISABLE_TABLE_D_COTOUCH=1 + EDR_STORY_ALL_OCCURRENCES=1 + EDR_L4_EXPORT_FILE, khong ONNX.
#  - Moi lan chay 1 thu muc rieng (rand_run_k): pipeline ghi processes.bin/resource_pool.bin vao cwd, khong de de len ban that.
ROOT=$PWD
ls parse_detect ta1-theia-e3-official-6r.bin ta1-theia-e3-official-6r.bin.12 >/dev/null || { echo 'THIEU parse_detect hoac file .bin trong ~/theia_e3'; exit 1; }
BINS="$ROOT/ta1-theia-e3-official-6r.bin $(for i in $(seq 1 12); do printf '%s ' "$ROOT/ta1-theia-e3-official-6r.bin.$i"; done)"
PIPE_CMD='mkdir -p rand_run_$k && cd rand_run_$k && EDR_L4_EXPORT_FILE=export_rand_$k.csv CG_VERIFY=1 EDR_DISABLE_TABLE_D_COTOUCH=1 $ROOT/parse_detect $BINS > run_log_rand_all_$k.txt'
for k in ${DRAWS:-1 2 3 4 5}; do
  S=seeds_rand_all_$k.txt
  export EDR_STORY_SEEDS_FILE=$PWD/$S         # uu tien hon autoseed/ALL_OCCURRENCES (xem avro_main.c)
  export EDR_STORY_ALL_OCCURRENCES=1          # chi de BO phan in "out" nang (khong chon seed, vi SEEDS_FILE thang)
  export EDR_STORY_LITE_FILE=$PWD/story_rand_all_$k.jsonl
  unset EDR_STORY_OUTPUT_FILE EDR_STORY_AUTOSEED_TOPK EDR_STORY_AUTOSEED_TOPK_C || true
  echo "### draw $k: seeds=$S"
  ( eval "$PIPE_CMD" ) 2>&1 | tail -5
  tail -3 rand_run_$k/run_log_rand_all_$k.txt || true
  python3 eval_story_gt.py --stories story_rand_all_$k.jsonl --log log.txt --gt $G --label rand_all_$k | tee -a baseline_random.log
done
[INFO] dan so (all) = 169643 node_id phan biet; n = 77013
[OUT] seeds_rand_all_1.txt (77013 seed)
[OUT] seeds_rand_all_2.txt (77013 seed)
[OUT] seeds_rand_all_3.txt (77013 seed)
[OUT] seeds_rand_all_4.txt (77013 seed)
[OUT] seeds_rand_all_5.txt (77013 seed)
[INFO] dan so (after) = 63223 node_id phan biet; n = 34358
[OUT] seeds_rand_after_1.txt (34358 seed)
[OUT] seeds_rand_after_2.txt (34358 seed)
[OUT] seeds_rand_after_3.txt (34358 seed)
[OUT] seeds_rand_after_4.txt (34358 seed)
[OUT] seeds_rand_after_5.txt (34358 seed)
### draw 1: seeds=seeds_rand_all_1.txt
[INFO][cache] tong ket: 47508 seed SKIP nho cache (da covered) / 29505 seed tu BFS day du (77013 tong, ty le skip 61.7%)
[L4_SCORE_CACHE] TONG KET: 512 slot, hit=0 miss=81979 (ty le hit=0.0%), evict=81467 (so lan phai da 1 component con "song" ra khoi cache vi het slot - cao = can them slot qua EDR_L4_SCORE_CACHE_SLOTS).
[execve_id_source] tu Event.properties (duong CU) = 70890
[execve_id_source] tu Subject.cmdLine (fallback MOI) = 168643
[execve_id_source] fallback chiem 70.40% tong so execve_id duoc dien (168643/239533)
NETWORK       : 13225082
PRIVESC       : 0
unique paths  : 400930
[rand_all_1] story that = 29505 | stub covered_by = 47508 | GT96 = 96 (GT49 sau cutoff = 49)
-- TOAN BO story --
  story: 29505 | story chua GT (TP): 23 | story precision = 0.0008
  member process phan biet: 128889
  recall GT96  : 57/96 = 0.5938  Wilson [0.494,0.687]
  recall GT49  : 24/49 = 0.4898  Wilson [0.356,0.625]
  PHU SONG: member = 128889 / 169650 process = 76.0% | GT96 ky vong neu phu ngau nhien cung ty le = 72.9 | lift = 0.78x (tran = 1.32x) | P(ngau nhien >= 57) = 1
  PHU SONG sau cutoff: member sau cutoff = 50732 / 63225 = 80.2% | GT49 ky vong = 39.3 | P(ngau nhien >= 24) = 1
-- chi story co seed start >= cutoff --
  story: 9213 | story chua GT (TP): 15 | story precision = 0.0016
  member process phan biet: 57018
  recall GT96  : 49/96 = 0.5104  Wilson [0.412,0.608]
  recall GT49  : 24/49 = 0.4898  Wilson [0.356,0.625]
  PHU SONG: member = 57018 / 169650 process = 33.6% | GT96 ky vong neu phu ngau nhien cung ty le = 32.3 | lift = 1.52x (tran = 2.98x) | P(ngau nhien >= 49) = 0.000317
  PHU SONG sau cutoff: member sau cutoff = 48994 / 63225 = 77.5% | GT49 ky vong = 38.0 | P(ngau nhien >= 24) = 1
### draw 2: seeds=seeds_rand_all_2.txt
[INFO][cache] tong ket: 47574 seed SKIP nho cache (da covered) / 29439 seed tu BFS day du (77013 tong, ty le skip 61.8%)
[L4_SCORE_CACHE] TONG KET: 512 slot, hit=0 miss=81979 (ty le hit=0.0%), evict=81467 (so lan phai da 1 component con "song" ra khoi cache vi het slot - cao = can them slot qua EDR_L4_SCORE_CACHE_SLOTS).
[execve_id_source] tu Event.properties (duong CU) = 70890
[execve_id_source] tu Subject.cmdLine (fallback MOI) = 168643
[execve_id_source] fallback chiem 70.40% tong so execve_id duoc dien (168643/239533)
NETWORK       : 13225082
PRIVESC       : 0
unique paths  : 400930
[rand_all_2] story that = 29439 | stub covered_by = 47574 | GT96 = 96 (GT49 sau cutoff = 49)
-- TOAN BO story --
  story: 29439 | story chua GT (TP): 18 | story precision = 0.0006
  member process phan biet: 129767
  recall GT96  : 90/96 = 0.9375  Wilson [0.870,0.971]
  recall GT49  : 48/49 = 0.9796  Wilson [0.893,0.996]
  PHU SONG: member = 129767 / 169650 process = 76.5% | GT96 ky vong neu phu ngau nhien cung ty le = 73.4 | lift = 1.23x (tran = 1.31x) | P(ngau nhien >= 90) = 6.57e-06
  PHU SONG sau cutoff: member sau cutoff = 51309 / 63225 = 81.2% | GT49 ky vong = 39.8 | P(ngau nhien >= 48) = 0.000444
-- chi story co seed start >= cutoff --
  story: 9001 | story chua GT (TP): 5 | story precision = 0.0006
  member process phan biet: 57736
  recall GT96  : 48/96 = 0.5000  Wilson [0.402,0.598]
  recall GT49  : 48/49 = 0.9796  Wilson [0.893,0.996]
  PHU SONG: member = 57736 / 169650 process = 34.0% | GT96 ky vong neu phu ngau nhien cung ty le = 32.7 | lift = 1.47x (tran = 2.94x) | P(ngau nhien >= 48) = 0.000904
  PHU SONG sau cutoff: member sau cutoff = 49357 / 63225 = 78.1% | GT49 ky vong = 38.3 | P(ngau nhien >= 48) = 7.91e-05
### draw 3: seeds=seeds_rand_all_3.txt
[INFO][cache] tong ket: 47603 seed SKIP nho cache (da covered) / 29410 seed tu BFS day du (77013 tong, ty le skip 61.8%)
[L4_SCORE_CACHE] TONG KET: 512 slot, hit=0 miss=81979 (ty le hit=0.0%), evict=81467 (so lan phai da 1 component con "song" ra khoi cache vi het slot - cao = can them slot qua EDR_L4_SCORE_CACHE_SLOTS).
[execve_id_source] tu Event.properties (duong CU) = 70890
[execve_id_source] tu Subject.cmdLine (fallback MOI) = 168643
[execve_id_source] fallback chiem 70.40% tong so execve_id duoc dien (168643/239533)
NETWORK       : 13225082
PRIVESC       : 0
unique paths  : 400930
[rand_all_3] story that = 29410 | stub covered_by = 47603 | GT96 = 96 (GT49 sau cutoff = 49)
-- TOAN BO story --
  story: 29410 | story chua GT (TP): 18 | story precision = 0.0006
  member process phan biet: 128972
  recall GT96  : 44/96 = 0.4583  Wilson [0.362,0.558]
  recall GT49  : 14/49 = 0.2857  Wilson [0.178,0.424]
  PHU SONG: member = 128972 / 169650 process = 76.0% | GT96 ky vong neu phu ngau nhien cung ty le = 73.0 | lift = 0.60x (tran = 1.32x) | P(ngau nhien >= 44) = 1
  PHU SONG sau cutoff: member sau cutoff = 50984 / 63225 = 80.6% | GT49 ky vong = 39.5 | P(ngau nhien >= 14) = 1
-- chi story co seed start >= cutoff --
  story: 9061 | story chua GT (TP): 11 | story precision = 0.0012
  member process phan biet: 56796
  recall GT96  : 14/96 = 0.1458  Wilson [0.089,0.230]
  recall GT49  : 14/49 = 0.2857  Wilson [0.178,0.424]
  PHU SONG: member = 56796 / 169650 process = 33.5% | GT96 ky vong neu phu ngau nhien cung ty le = 32.1 | lift = 0.44x (tran = 2.99x) | P(ngau nhien >= 14) = 1
  PHU SONG sau cutoff: member sau cutoff = 49219 / 63225 = 77.8% | GT49 ky vong = 38.1 | P(ngau nhien >= 14) = 1
### draw 4: seeds=seeds_rand_all_4.txt
[INFO][cache] tong ket: 47402 seed SKIP nho cache (da covered) / 29611 seed tu BFS day du (77013 tong, ty le skip 61.6%)
[L4_SCORE_CACHE] TONG KET: 512 slot, hit=0 miss=81979 (ty le hit=0.0%), evict=81467 (so lan phai da 1 component con "song" ra khoi cache vi het slot - cao = can them slot qua EDR_L4_SCORE_CACHE_SLOTS).
[execve_id_source] tu Event.properties (duong CU) = 70890
[execve_id_source] tu Subject.cmdLine (fallback MOI) = 168643
[execve_id_source] fallback chiem 70.40% tong so execve_id duoc dien (168643/239533)
NETWORK       : 13225082
PRIVESC       : 0
unique paths  : 400930
[rand_all_4] story that = 29611 | stub covered_by = 47402 | GT96 = 96 (GT49 sau cutoff = 49)
-- TOAN BO story --
  story: 29611 | story chua GT (TP): 28 | story precision = 0.0009
  member process phan biet: 129196
  recall GT96  : 44/96 = 0.4583  Wilson [0.362,0.558]
  recall GT49  : 23/49 = 0.4694  Wilson [0.337,0.606]
  PHU SONG: member = 129196 / 169650 process = 76.2% | GT96 ky vong neu phu ngau nhien cung ty le = 73.1 | lift = 0.60x (tran = 1.31x) | P(ngau nhien >= 44) = 1
  PHU SONG sau cutoff: member sau cutoff = 50698 / 63225 = 80.2% | GT49 ky vong = 39.3 | P(ngau nhien >= 23) = 1
-- chi story co seed start >= cutoff --
  story: 9259 | story chua GT (TP): 17 | story precision = 0.0018
  member process phan biet: 55007
  recall GT96  : 23/96 = 0.2396  Wilson [0.165,0.334]
  recall GT49  : 23/49 = 0.4694  Wilson [0.337,0.606]
  PHU SONG: member = 55007 / 169650 process = 32.4% | GT96 ky vong neu phu ngau nhien cung ty le = 31.1 | lift = 0.74x (tran = 3.08x) | P(ngau nhien >= 23) = 0.973
  PHU SONG sau cutoff: member sau cutoff = 48818 / 63225 = 77.2% | GT49 ky vong = 37.8 | P(ngau nhien >= 23) = 1
### draw 5: seeds=seeds_rand_all_5.txt
[INFO][cache] tong ket: 47275 seed SKIP nho cache (da covered) / 29738 seed tu BFS day du (77013 tong, ty le skip 61.4%)
[L4_SCORE_CACHE] TONG KET: 512 slot, hit=0 miss=81979 (ty le hit=0.0%), evict=81467 (so lan phai da 1 component con "song" ra khoi cache vi het slot - cao = can them slot qua EDR_L4_SCORE_CACHE_SLOTS).
[execve_id_source] tu Event.properties (duong CU) = 70890
[execve_id_source] tu Subject.cmdLine (fallback MOI) = 168643
[execve_id_source] fallback chiem 70.40% tong so execve_id duoc dien (168643/239533)
NETWORK       : 13225082
PRIVESC       : 0
unique paths  : 400930
[rand_all_5] story that = 29738 | stub covered_by = 47275 | GT96 = 96 (GT49 sau cutoff = 49)
-- TOAN BO story --
  story: 29738 | story chua GT (TP): 32 | story precision = 0.0011
  member process phan biet: 128956
  recall GT96  : 76/96 = 0.7917  Wilson [0.700,0.861]
  recall GT49  : 34/49 = 0.6939  Wilson [0.555,0.805]
  PHU SONG: member = 128956 / 169650 process = 76.0% | GT96 ky vong neu phu ngau nhien cung ty le = 73.0 | lift = 1.04x (tran = 1.32x) | P(ngau nhien >= 76) = 0.277
  PHU SONG sau cutoff: member sau cutoff = 50932 / 63225 = 80.6% | GT49 ky vong = 39.5 | P(ngau nhien >= 34) = 0.98
-- chi story co seed start >= cutoff --
  story: 9257 | story chua GT (TP): 22 | story precision = 0.0024
  member process phan biet: 53807
  recall GT96  : 34/96 = 0.3542  Wilson [0.266,0.454]
  recall GT49  : 34/49 = 0.6939  Wilson [0.555,0.805]
  PHU SONG: member = 53807 / 169650 process = 31.7% | GT96 ky vong neu phu ngau nhien cung ty le = 30.4 | lift = 1.12x (tran = 3.15x) | P(ngau nhien >= 34) = 0.249
  PHU SONG sau cutoff: member sau cutoff = 48522 / 63225 = 76.7% | GT49 ky vong = 37.6 | P(ngau nhien >= 34) = 0.914
enovo@DESKTOP-9J5D6AQ:~/theia_e3$
```

### 6.2 Truy vết GT trong story của các lần bốc ngẫu nhiên (`trace_random_hits.py`)

Output của lệnh chạy 15/11/2026 01:15.

```
enovo@DESKTOP-9J5D6AQ:~/theia_e3$ python3 trace_random_hits.py --log log.txt \
  --gt node_Browser_Extension_Drakon_Dropper.csv node_Firefox_Backdoor_Drakon_In_Memory.csv \
  --stories story_rand_all_1.jsonl story_rand_all_2.jsonl story_rand_all_3.jsonl story_rand_all_4.jsonl \
  --top 10 | tee trace_random_hits.log
GT96=96 GT49=49

=== story_rand_all_1.jsonl: 29505 story that, 47508 stub ===
GT96 trung 57/96; GT49 trung 24/49
story chua GT: 23. Top 10 theo so GT:
  nGT96 nGT49 seed_node_id seed_la_GT seed_start_ms member_count seed_uuid
     25     0      106838 False     1523552487126           60 07369EB8020000000000000000000020
      9     9      106755 False     1523551818878           11 B63501B6020000000000000000000020
      7     7      106676 True      1523551394323            8 673559B4020000000000000000000020
      3     3      107385 True      1523553424209            3 283847BC020000000000000000000020
      2     2      106521 True      1523551202271            5 CC3499B3020000000000000000000020
      1     1      107249 True      1523552967126            2 A1377EBA020000000000000000000020
      1     1      107207 True      1523552826599            2 7737F1B9020000000000000000000020
      1     1      106914 True      1523552571822            2 5336F2B8020000000000000000000020
      1     1      106832 True      1523552469374            2 01368CB8020000000000000000000020
      1     1      106818 True      1523552287815            2 F335D6B7020000000000000000000020
seed (story that + stub) chinh la GT: 19
  node_id=106767 uuid=C2357BB6020000000000000000000020
  node_id=106676 uuid=673559B4020000000000000000000020
  node_id=4659 uuid=EE1F3230000000000000000000000020
  node_id=106818 uuid=F335D6B7020000000000000000000020
  node_id=106521 uuid=CC3499B3020000000000000000000020
  node_id=106832 uuid=01368CB8020000000000000000000020
  node_id=106762 uuid=BD353EB6020000000000000000000020
  node_id=107207 uuid=7737F1B9020000000000000000000020
  node_id=106914 uuid=5336F2B8020000000000000000000020
  node_id=2707 uuid=5418F32A000000000000000000000020
GT khong nam trong story nao: 39; vi du: 033690B8020000000000000000000020, 043694B8020000000000000000000020, 1920A530000000000000000000000020, 3536B9B8020000000000000000000020, 3F36C5B8020000000000000000000020

=== story_rand_all_2.jsonl: 29439 story that, 47574 stub ===
GT96 trung 90/96; GT49 trung 48/49
story chua GT: 18. Top 10 theo so GT:
  nGT96 nGT49 seed_node_id seed_la_GT seed_start_ms member_count seed_uuid
     44    44      106682 True      1523551441785           47 6D3588B4020000000000000000000020
     25     0        2759 True      1523385589397           60 8318AC2C000000000000000000000020
      8     0        4615 True      1523386456639            9 C21F0F30000000000000000000000020
      2     2      107425 True      1523553859639            2 4D38FABD020000000000000000000020
      2     2      106518 False     1523551202249            5 C93499B3020000000000000000000020
      1     1      107207 True      1523552826599            2 7737F1B9020000000000000000000020
      1     1      106766 True      1523551939312            2 C1357AB6020000000000000000000020
      1     0        4672 True      1523386599790            2 FA1F9E30000000000000000000000020
      1     0        4668 True      1523386592476            2 F71F9730000000000000000000000020
      1     0        4548 True      1523386402855            5 7F1FD92F000000000000000000000020
seed (story that + stub) chinh la GT: 16
  node_id=107207 uuid=7737F1B9020000000000000000000020
  node_id=4672 uuid=FA1F9E30000000000000000000000020
  node_id=106766 uuid=C1357AB6020000000000000000000020
  node_id=106682 uuid=6D3588B4020000000000000000000020
  node_id=2759 uuid=8318AC2C000000000000000000000020
  node_id=2707 uuid=5418F32A000000000000000000000020
  node_id=107425 uuid=4D38FABD020000000000000000000020
  node_id=4548 uuid=7F1FD92F000000000000000000000020
  node_id=4668 uuid=F71F9730000000000000000000000020
  node_id=2708 uuid=5518F72A000000000000000000000020
GT khong nam trong story nao: 6; vi du: 293847BC020000000000000000000020, 4E18B22A000000000000000000000020, 4F18B22A000000000000000000000020, 65189F2B000000000000000000000020, 6618A32B000000000000000000000020

=== story_rand_all_3.jsonl: 29410 story that, 47603 stub ===
GT96 trung 44/96; GT49 trung 14/49
story chua GT: 18. Top 10 theo so GT:
  nGT96 nGT49 seed_node_id seed_la_GT seed_start_ms member_count seed_uuid
     17     0        2837 True      1523385798632           52 D0187D2D000000000000000000000020
      8     0        4615 True      1523386456639            9 C21F0F30000000000000000000000020
      5     5      106755 False     1523551818878            7 B63501B6020000000000000000000020
      3     3      106634 True      1523551353293            4 3D3530B4020000000000000000000020
      1     1      107290 True      1523553026719            2 CA37B9BA020000000000000000000020
      1     1      107247 True      1523552947293            2 9F376ABA020000000000000000000020
      1     1      106894 True      1523552526157            2 3F36C5B8020000000000000000000020
      1     1      106884 True      1523552514240            2 3536B9B8020000000000000000000020
      1     1      106835 True      1523552477786            2 043694B8020000000000000000000020
      1     1      106834 True      1523552473580            2 033690B8020000000000000000000020
seed (story that + stub) chinh la GT: 16
  node_id=2837 uuid=D0187D2D000000000000000000000020
  node_id=106816 uuid=F135CDB7020000000000000000000020
  node_id=106817 uuid=F235D6B7020000000000000000000020
  node_id=106834 uuid=033690B8020000000000000000000020
  node_id=106634 uuid=3D3530B4020000000000000000000020
  node_id=106894 uuid=3F36C5B8020000000000000000000020
  node_id=2729 uuid=6618A32B000000000000000000000020
  node_id=2735 uuid=6B18B82B000000000000000000000020
  node_id=4615 uuid=C21F0F30000000000000000000000020
  node_id=106773 uuid=C835BAB6020000000000000000000020
GT khong nam trong story nao: 52; vi du: 01368CB8020000000000000000000020, 06187E2A000000000000000000000020, 283847BC020000000000000000000020, 293847BC020000000000000000000020, 4036C6B8020000000000000000000020

=== story_rand_all_4.jsonl: 29611 story that, 47402 stub ===
GT96 trung 44/96; GT49 trung 23/49
story chua GT: 28. Top 10 theo so GT:
  nGT96 nGT49 seed_node_id seed_la_GT seed_start_ms member_count seed_uuid
     12     0       58036 False     1523468604968           47 BD72F470010000000000000000000020
     11     0        2755 True      1523385581986           14 7F18A52C000000000000000000000020
      4     4      106634 True      1523551353293            5 3D3530B4020000000000000000000020
      3     3      107385 True      1523553424209            3 283847BC020000000000000000000020
      2     2      106521 True      1523551202271            4 CC3499B3020000000000000000000020
      1     1      107290 True      1523553026719            2 CA37B9BA020000000000000000000020
      1     1      107249 True      1523552967126            2 A1377EBA020000000000000000000020
      1     1      107232 True      1523552910630            2 903745BA020000000000000000000020
      1     1      107208 True      1523552826899            2 7837F1B9020000000000000000000020
      1     1      106918 True      1523552613082            2 56361CB9020000000000000000000020
seed (story that + stub) chinh la GT: 26
  node_id=2755 uuid=7F18A52C000000000000000000000020
  node_id=106634 uuid=3D3530B4020000000000000000000020
  node_id=4656 uuid=EB1F2430000000000000000000000020
  node_id=2730 uuid=6718A92B000000000000000000000020
  node_id=4668 uuid=F71F9730000000000000000000000020
  node_id=4659 uuid=EE1F3230000000000000000000000020
  node_id=4703 uuid=1920A530000000000000000000000020
  node_id=106771 uuid=C6359FB6020000000000000000000020
  node_id=106834 uuid=033690B8020000000000000000000020
  node_id=106766 uuid=C1357AB6020000000000000000000020
GT khong nam trong story nao: 52; vi du: 01368CB8020000000000000000000020, 043694B8020000000000000000000020, 29202531000000000000000000000020, 3536B9B8020000000000000000000020, 3F36C5B8020000000000000000000020
enovo@DESKTOP-9J5D6AQ:~/theia_e3$
```

### 6.3 Velox × phễu, 5 hash (`run_velox5_x_funnel.sh`)

Output `bash run_velox5_x_funnel.sh 2>&1 | tee velox5_x_funnel.log`, 15/11/2026 01:50.

```
(hfenv) enovo@DESKTOP-9J5D6AQ:~/theia_e3$ bash run_velox5_x_funnel.sh 2>&1 | tee velox5_x_funnel.log
map: 278363 dong | GT51: khop 51, lech 0, thieu 0
=> DUNG DUOC
[INFO] anh xa: 278363 index_id -> uuid
[OUT] /home/enovo/velox_out5/0a78fe94_alert_nodes_ep11.uuid.txt: 1872/1872 alert co UUID
[OUT] /home/enovo/velox_out5/78d38503_alert_nodes_ep11.uuid.txt: 1854/1854 alert co UUID
[OUT] /home/enovo/velox_out5/7d4ec390_alert_nodes_ep11.uuid.txt: 2064/2064 alert co UUID
[OUT] /home/enovo/velox_out5/a9b4c3d0_alert_nodes_ep11.uuid.txt: 2276/2276 alert co UUID
[OUT] /home/enovo/velox_out5/fd46053a_alert_nodes_ep11.uuid.txt: 2132/2132 alert co UUID
######## a9b4c3d0
(2) [SYS] CHAM TREN DUNG GT CUA VELOX
[INFO] VELOX-GT: 51 uuid (regex-toan-file)
  Velox GT ∩ log = 51
  ∩ GT49 (sau cutoff) = 49 | Velox có, GT96 khong có = 0 | GT49 có, Velox khong có = 0
  Velox GT theo start: truoc cutoff = 2 | sau cutoff = 49
(3) VELOX ALERT ∩ PHEU / STORY MEMBER
  TP duoc tinh theo: Velox GT51 (n=51)
[INFO] ALERT /home/enovo/velox_out5/a9b4c3d0_alert_nodes_ep11.uuid.txt: 2276 uuid (regex-toan-file)
  TB Velox           : alert 2276 | TP 36.0 | FP 2240 | precision 0.0158 | recall 0.7059
  TB Velox ∩ phễu    : alert 560 | TP 13.0 | FP 547 | precision 0.0232 | recall 0.2549
  TB Velox ∩ story   : alert 28 | TP 26.0 | FP 2 | precision 0.9286 | recall 0.5098
######## 0a78fe94
(2) [SYS] CHAM TREN DUNG GT CUA VELOX
[INFO] VELOX-GT: 51 uuid (regex-toan-file)
  Velox GT ∩ log = 51
  ∩ GT49 (sau cutoff) = 49 | Velox có, GT96 khong có = 0 | GT49 có, Velox khong có = 0
  Velox GT theo start: truoc cutoff = 2 | sau cutoff = 49
(3) VELOX ALERT ∩ PHEU / STORY MEMBER
  TP duoc tinh theo: Velox GT51 (n=51)
[INFO] ALERT /home/enovo/velox_out5/0a78fe94_alert_nodes_ep11.uuid.txt: 1872 uuid (regex-toan-file)
  TB Velox           : alert 1872 | TP 36.0 | FP 1836 | precision 0.0192 | recall 0.7059
  TB Velox ∩ phễu    : alert 307 | TP 2.0 | FP 305 | precision 0.0065 | recall 0.0392
  TB Velox ∩ story   : alert 36 | TP 35.0 | FP 1 | precision 0.9722 | recall 0.6863
######## 7d4ec390
(2) [SYS] CHAM TREN DUNG GT CUA VELOX
[INFO] VELOX-GT: 51 uuid (regex-toan-file)
  Velox GT ∩ log = 51
  ∩ GT49 (sau cutoff) = 49 | Velox có, GT96 khong có = 0 | GT49 có, Velox khong có = 0
  Velox GT theo start: truoc cutoff = 2 | sau cutoff = 49
(3) VELOX ALERT ∩ PHEU / STORY MEMBER
  TP duoc tinh theo: Velox GT51 (n=51)
[INFO] ALERT /home/enovo/velox_out5/7d4ec390_alert_nodes_ep11.uuid.txt: 2064 uuid (regex-toan-file)
  TB Velox           : alert 2064 | TP 11.0 | FP 2053 | precision 0.0053 | recall 0.2157
  TB Velox ∩ phễu    : alert 363 | TP 11.0 | FP 352 | precision 0.0303 | recall 0.2157
  TB Velox ∩ story   : alert 3 | TP 3.0 | FP 0 | precision 1.0000 | recall 0.0588
######## 78d38503
(2) [SYS] CHAM TREN DUNG GT CUA VELOX
[INFO] VELOX-GT: 51 uuid (regex-toan-file)
  Velox GT ∩ log = 51
  ∩ GT49 (sau cutoff) = 49 | Velox có, GT96 khong có = 0 | GT49 có, Velox khong có = 0
  Velox GT theo start: truoc cutoff = 2 | sau cutoff = 49
(3) VELOX ALERT ∩ PHEU / STORY MEMBER
  TP duoc tinh theo: Velox GT51 (n=51)
[INFO] ALERT /home/enovo/velox_out5/78d38503_alert_nodes_ep11.uuid.txt: 1854 uuid (regex-toan-file)
  TB Velox           : alert 1854 | TP 11.0 | FP 1843 | precision 0.0059 | recall 0.2157
  TB Velox ∩ phễu    : alert 266 | TP 11.0 | FP 255 | precision 0.0414 | recall 0.2157
  TB Velox ∩ story   : alert 3 | TP 3.0 | FP 0 | precision 1.0000 | recall 0.0588
######## fd46053a
(2) [SYS] CHAM TREN DUNG GT CUA VELOX
[INFO] VELOX-GT: 51 uuid (regex-toan-file)
  Velox GT ∩ log = 51
  ∩ GT49 (sau cutoff) = 49 | Velox có, GT96 khong có = 0 | GT49 có, Velox khong có = 0
  Velox GT theo start: truoc cutoff = 2 | sau cutoff = 49
(3) VELOX ALERT ∩ PHEU / STORY MEMBER
  TP duoc tinh theo: Velox GT51 (n=51)
[INFO] ALERT /home/enovo/velox_out5/fd46053a_alert_nodes_ep11.uuid.txt: 2132 uuid (regex-toan-file)
  TB Velox           : alert 2132 | TP 11.0 | FP 2121 | precision 0.0052 | recall 0.2157
  TB Velox ∩ phễu    : alert 425 | TP 11.0 | FP 414 | precision 0.0259 | recall 0.2157
  TB Velox ∩ story   : alert 4 | TP 3.0 | FP 1 | precision 0.7500 | recall 0.0588
(hfenv) enovo@DESKTOP-9J5D6AQ:~/theia_e3$
```

### 6.4 Kéo kết quả Velox từ HuggingFace (`velox_pull_runs.sh`)

Script và output, 15/11/2026 01:46.

```
(hfenv) enovo@DESKTOP-9J5D6AQ:~/theia_e3$ #!/usr/bin/env bash
# Chay trong WSL: source ~/hfenv/bin/activate truoc. Stream tung tar.gz tu HF, CHI giai nen scores_model_epoch_11.pkl
# (khong luu tar.gz xuong dia). Roi velox_extract.py -> alert_nodes_ep11.txt theo tung run.
# Ket qua: ~/velox_runs/<RUN>/... va ~/velox_out_all/<RUN>__<hash8>_alert_nodes_ep11.txt
set -uo pipefail
R=MinhHieu1234567890/pidsmaker-results
RUNS="${RUNS:-theia_e3_cut_same_host_20261112_1053_A_velox theia_e3_cut_same_host_20261112_1053_A_velox_s1 theia_e3_cut_same_host_20261112_1053_A_velox_s2 theia_e3_cut_same_host_20261112_1053_A_velox_s3 theia_e3_cut_same_host_20261112_1053_A_velox_s4}"
TOKEN=$(python3 -c "from huggingface_hub import get_token;print(get_token() or '')")
AUTH=(); [ -n "$TOKEN" ] && AUTH=(-H "Authorization: Bearer $TOKEN")
mkdir -p ~/velox_runs ~/velox_out_all
for RUN in $RUNS; do
  D=~/velox_runs/$RUN; mkdir -p "$D"
  if find "$D" -name 'scores_model_epoch_11.pkl' | grep -q .; then echo "[skip] $RUN da co"; else
    echo "[stream] $RUN ..."
    curl -sL "${AUTH[@]}" "https://huggingface.co/datasets/$R/resolve/main/$RUN/$RUN.tar.gz" \
      | tar -xzf - -C "$D" --wildcards '*precision_recall_dir/scores_model_epoch_11.pkl' 2>/dev/null
    find "$D" -name 'scores_model_epoch_11.pkl' | grep -q . || { echo "[LOI] $RUN: khong thay scores_model_epoch_11.pkl (tar.gz thieu/token/duong dan)"; continue; }
  fi
  ROOT=$(find "$D" -type d -path '*evaluation/evaluation' | head -1)
  [ -n "$ROOT" ] || { echo "[LOI] $RUN: khong thay thu muc evaluation/evaluation"; continue; }
  echo "=== $RUN  (root=$ROOT)"
  OUT=~/velox_out_tmp_$$; rm -rf "$OUT"
  python3 ~/theia_e3/velox_extract.py "$ROOT" --epoch 11 --out "$OUT" 2>&1 | tail -n +1
  for f in "$OUT"/*_alert_nodes_ep11.txt; do [ -f "$f" ] && cp "$f" ~/velox_out_all/${RUN#theia_e3_cut_same_host_20261112_}__$(basename "$f"); done
  rm -rf "$OUT"
done
echo; ls -l ~/velox_out_all/*alert_nodes_ep11.txt
[stream] theia_e3_cut_same_host_20261112_1053_A_velox ...
=== theia_e3_cut_same_host_20261112_1053_A_velox  (root=/home/enovo/velox_runs/theia_e3_cut_same_host_20261112_1053_A_velox/evaluation/evaluation)
hash       epoch   nodes   GT  alert   TP     FP     prec  recall  bestGTrank
a9b4c3d0      11   59330   66   2276   36   2240  0.01582  0.5455        1495
[stream] theia_e3_cut_same_host_20261112_1053_A_velox_s1 ...
=== theia_e3_cut_same_host_20261112_1053_A_velox_s1  (root=/home/enovo/velox_runs/theia_e3_cut_same_host_20261112_1053_A_velox_s1/evaluation/evaluation)
hash       epoch   nodes   GT  alert   TP     FP     prec  recall  bestGTrank
0a78fe94      11   59330   66   1872   36   1836  0.01923  0.5455          54
a9b4c3d0      11   59330   66   2276   36   2240  0.01582  0.5455        1495
[stream] theia_e3_cut_same_host_20261112_1053_A_velox_s2 ...
=== theia_e3_cut_same_host_20261112_1053_A_velox_s2  (root=/home/enovo/velox_runs/theia_e3_cut_same_host_20261112_1053_A_velox_s2/evaluation/evaluation)
hash       epoch   nodes   GT  alert   TP     FP     prec  recall  bestGTrank
0a78fe94      11   59330   66   1872   36   1836  0.01923  0.5455          54
7d4ec390      11   59330   66   2064   11   2053  0.00533  0.1667        1437
a9b4c3d0      11   59330   66   2276   36   2240  0.01582  0.5455        1495
[stream] theia_e3_cut_same_host_20261112_1053_A_velox_s3 ...
=== theia_e3_cut_same_host_20261112_1053_A_velox_s3  (root=/home/enovo/velox_runs/theia_e3_cut_same_host_20261112_1053_A_velox_s3/evaluation/evaluation)
hash       epoch   nodes   GT  alert   TP     FP     prec  recall  bestGTrank
0a78fe94      11   59330   66   1872   36   1836  0.01923  0.5455          54
78d38503      11   59330   66   1854   11   1843  0.00593  0.1667         706
7d4ec390      11   59330   66   2064   11   2053  0.00533  0.1667        1437
a9b4c3d0      11   59330   66   2276   36   2240  0.01582  0.5455        1495
[stream] theia_e3_cut_same_host_20261112_1053_A_velox_s4 ...
=== theia_e3_cut_same_host_20261112_1053_A_velox_s4  (root=/home/enovo/velox_runs/theia_e3_cut_same_host_20261112_1053_A_velox_s4/evaluation/evaluation)
hash       epoch   nodes   GT  alert   TP     FP     prec  recall  bestGTrank
0a78fe94      11   59330   66   1872   36   1836  0.01923  0.5455          54
78d38503      11   59330   66   1854   11   1843  0.00593  0.1667         706
7d4ec390      11   59330   66   2064   11   2053  0.00533  0.1667        1437
a9b4c3d0      11   59330   66   2276   36   2240  0.01582  0.5455        1495
fd46053a      11   59330   66   2132   11   2121  0.00516  0.1667        1461

-rw-r--r-- 1 enovo enovo 15931 Thg 11 14 17:27 /home/enovo/velox_out_all/1053_A_velox__a9b4c3d0_alert_nodes_ep11.txt
-rw-r--r-- 1 enovo enovo 13103 Thg 11 14 17:35 /home/enovo/velox_out_all/1053_A_velox_s1__0a78fe94_alert_nodes_ep11.txt
-rw-r--r-- 1 enovo enovo 15931 Thg 11 14 17:35 /home/enovo/velox_out_all/1053_A_velox_s1__a9b4c3d0_alert_nodes_ep11.txt
-rw-r--r-- 1 enovo enovo 13103 Thg 11 14 17:43 /home/enovo/velox_out_all/1053_A_velox_s2__0a78fe94_alert_nodes_ep11.txt
-rw-r--r-- 1 enovo enovo 14447 Thg 11 14 17:43 /home/enovo/velox_out_all/1053_A_velox_s2__7d4ec390_alert_nodes_ep11.txt
-rw-r--r-- 1 enovo enovo 15931 Thg 11 14 17:43 /home/enovo/velox_out_all/1053_A_velox_s2__a9b4c3d0_alert_nodes_ep11.txt
-rw-r--r-- 1 enovo enovo 13103 Thg 11 14 17:51 /home/enovo/velox_out_all/1053_A_velox_s3__0a78fe94_alert_nodes_ep11.txt
-rw-r--r-- 1 enovo enovo 12977 Thg 11 14 17:51 /home/enovo/velox_out_all/1053_A_velox_s3__78d38503_alert_nodes_ep11.txt
-rw-r--r-- 1 enovo enovo 14447 Thg 11 14 17:51 /home/enovo/velox_out_all/1053_A_velox_s3__7d4ec390_alert_nodes_ep11.txt
-rw-r--r-- 1 enovo enovo 15931 Thg 11 14 17:51 /home/enovo/velox_out_all/1053_A_velox_s3__a9b4c3d0_alert_nodes_ep11.txt
-rw-r--r-- 1 enovo enovo 13103 Thg 11 14 18:00 /home/enovo/velox_out_all/1053_A_velox_s4__0a78fe94_alert_nodes_ep11.txt
-rw-r--r-- 1 enovo enovo 12977 Thg 11 14 18:00 /home/enovo/velox_out_all/1053_A_velox_s4__78d38503_alert_nodes_ep11.txt
-rw-r--r-- 1 enovo enovo 14447 Thg 11 14 18:00 /home/enovo/velox_out_all/1053_A_velox_s4__7d4ec390_alert_nodes_ep11.txt
-rw-r--r-- 1 enovo enovo 15931 Thg 11 14 18:00 /home/enovo/velox_out_all/1053_A_velox_s4__a9b4c3d0_alert_nodes_ep11.txt
-rw-r--r-- 1 enovo enovo 14923 Thg 11 14 18:00 /home/enovo/velox_out_all/1053_A_velox_s4__fd46053a_alert_nodes_ep11.txt
(hfenv) enovo@DESKTOP-9J5D6AQ:~/theia_e3$ cd ~/theia_e3
```

### 6.5 Đánh giá Velox gốc, hash 47a79761 (`precision_recall_dir`)

Output đánh giá của run Velox 47a79761…, 14/11/2026 15:43.

```
======================================================================
/home/enovo/pidsmaker-results/A_THEIA_E3_CUT_same_host/artifacts/evaluation/evaluation/47a79761d351455271b3e629d0b243cfe035231cf35c8f52c065d74991411339/THEIA_E3_CUT/precision_recall_dir/scores_model_epoch_11.pkl (1.0 MB)
======================================================================
[OK] mo bang torch
root: dict len=5
  'pred_scores': list len=59330
    [0]: float
      repr=0.9051531553268432
    [1]: float
      repr=3.290122185717337e-05
    [2]: float
      repr=0.9744672179222108
    [3]: float
      repr=0.9128693342208862
    [4]: float
      repr=0.9051531553268432
    [5]: float
      repr=8.354979515075684
  'y_preds': list len=59330
    [0]: int
      repr=0
    [1]: int
      repr=0
    [2]: int
      repr=0
    [3]: int
      repr=0
    [4]: int
      repr=0
    [5]: int
      repr=0
  'y_truth': list len=59330
    [0]: int
      repr=0
    [1]: int
      repr=0
    [2]: int
      repr=0
    [3]: int
      repr=0
    [4]: int
      repr=0
    [5]: int
      repr=0
  'nodes': list len=59330
    [0]: int
      repr=186100
    [1]: int
      repr=186101
    [2]: int
      repr=186102
    [3]: int
      repr=186104
    [4]: int
      repr=186105
    [5]: int
      repr=186106
  'node2attacks': dict len=51
    216833: set len=1
      repr={0}
    216194: set len=1
      repr={0}
    216451: set len=1
      repr={0}
    216322: set len=1
      repr={0}
    216450: set len=1
      repr={0}
    216323: set len=1
      repr={0}
    ... (45 khoa nua)

======================================================================
/home/enovo/pidsmaker-results/A_THEIA_E3_CUT_same_host/artifacts/evaluation/evaluation/47a79761d351455271b3e629d0b243cfe035231cf35c8f52c065d74991411339/THEIA_E3_CUT/precision_recall_dir/result_model_epoch_11.pth (5.0 MB)
======================================================================
[OK] mo bang torch
root: defaultdict len=59330
  186100: dict len=5
    'score': float
      repr=0.9051531553268432
    'tw_with_max_loss': int64 shape=() dtype=int64
      repr=np.int64(368)
    'y_true': int
      repr=0
    'is_seen': int
      repr=0
    'y_hat': int
      repr=0
  186101: dict len=5
    'score': float
      repr=3.290122185717337e-05
    'tw_with_max_loss': int64 shape=() dtype=int64
      repr=np.int64(352)
    'y_true': int
      repr=0
    'is_seen': int
      repr=0
    'y_hat': int
      repr=0
  186102: dict len=5
    'score': float
      repr=0.9744672179222108
    'tw_with_max_loss': int64 shape=() dtype=int64
      repr=np.int64(347)
    'y_true': int
      repr=0
    'is_seen': int
      repr=0
    'y_hat': int
      repr=0
  186104: dict len=5
    'score': float
      repr=0.9128693342208862
    'tw_with_max_loss': int64 shape=() dtype=int64
      repr=np.int64(358)
    'y_true': int
      repr=0
    'is_seen': int
      repr=0
    'y_hat': int
      repr=0
  186105: dict len=5
    'score': float
      repr=0.9051531553268432
    'tw_with_max_loss': int64 shape=() dtype=int64
      repr=np.int64(368)
    'y_true': int
      repr=0
    'is_seen': int
      repr=0
    'y_hat': int
      repr=0
  186106: dict len=5
    'score': float
      repr=8.354979515075684
    'tw_with_max_loss': int64 shape=() dtype=int64
      repr=np.int64(358)
    'y_true': int
      repr=0
    'is_seen': int
      repr=0
    'y_hat': int
      repr=0
  ... (59324 khoa nua)

======================================================================
/home/enovo/pidsmaker-results/A_THEIA_E3_CUT_same_host/artifacts/evaluation/evaluation/47a79761d351455271b3e629d0b243cfe035231cf35c8f52c065d74991411339/THEIA_E3_CUT/precision_recall_dir/stats_model_epoch_11.pth (0.0 MB)
======================================================================
[OK] mo bang torch
root: dict len=37
  'precision': float64 shape=() dtype=float64
    repr=np.float64(0.00635)
  'recall': float64 shape=() dtype=float64
    repr=np.float64(0.21569)
  'fpr': float64 shape=() dtype=float64
    repr=np.float64(0.0290322)
  'fscore': float64 shape=() dtype=float64
    repr=np.float64(0.01234)
  'ap': float64 shape=() dtype=float64
    repr=np.float64(0.00347)
  'accuracy': float64 shape=() dtype=float64
    repr=np.float64(0.97032)
  ... (31 khoa nua)
enovo@DESKTOP-9J5D6AQ:~/theia_e3$
```

### 6.6 Thống kê RG1–RG5 một lần chạy (`run_rg_once.sh`, `rg_summarize.py`), 17/11/2026

Khởi chạy 10:29:55Z (giờ WSL), kết thúc pipeline 12:00:21Z, hết 12:01:59Z. Bước Python tóm tắt trong script lỗi `int('')` (cột `rg` rỗng khi không có RG); không ảnh hưởng `rg_stats.csv`. Tóm tắt lại bằng `rg_summarize.py` từ file có sẵn. Output nguyên văn, theo thứ tự các lần dán trong phiên.

**(1) Khởi chạy (sau 120 giây, `head -15 run_rg_once.log`):**

```
enovo@DESKTOP-9J5D6AQ:~/theia_e3$ cp /mnt/c/Users/Lenovo/Downloads/run_rg_once.sh ~/theia_e3/
cd ~/theia_e3 && nohup bash run_rg_once.sh > run_rg_once.log 2>&1 &
sleep 120; head -15 run_rg_once.log
[1] 4369
nohup: ignoring input
patching file process_state_update.c
gcc -O2 -std=c11 -D_POSIX_C_SOURCE=200809L -I. -I/opt/onnxruntime/include -MMD -MP -c process_state_update.c -o build_wl/process_state_update.o
gcc build_wl/avro_main.o build_wl/avro_container.o build_wl/avro_datum_decode.o build_wl/avro_datum_dispatch.o build_wl/avro_datum_skip.o build_wl/avro_event_decode.o build_wl/avro_table_bridge.o build_wl/avro_fill_exec.o build_wl/avro_fill_file.o build_wl/avro_fill_memory.o build_wl/avro_fill_network.o build_wl/avro_fill_privesc.o build_wl/avro_fill_process.o build_wl/avro_fill_transfer.o build_wl/cdm18_chunk_reader.o build_wl/cdm18_exec_parser.o build_wl/cdm18_file_parser_fast.o build_wl/cdm18_memory_parser.o build_wl/cdm18_network_parser.o build_wl/cdm18_privesc_parser.o build_wl/cdm18_process_parser.o build_wl/cdm18_transfer_parser.o build_wl/domain_exec_handler.o build_wl/domain_file_handler.o build_wl/domain_memory_handler.o build_wl/domain_network_handler.o build_wl/domain_privesc_handler.o build_wl/domain_process_handler.o build_wl/domain_transfer_handler.o build_wl/resource_pool.o build_wl/process_state.o build_wl/process_state_update.o build_wl/process_table.o build_wl/causal_graph.o build_wl/detect_layer1.o build_wl/detect_layer2.o build_wl/event_classify.o build_wl/event_decode_readable.o build_wl/layer1_dump.o build_wl/layer2_dump.o build_wl/layer2_priority.o build_wl/layer2_scan_common.o build_wl/layer2_tier1_cheap.o build_wl/layer2_tier2_mid.o build_wl/layer2_tier3_disk.o build_wl/layer3.o build_wl/kv_evict.o build_wl/l4_export.o build_wl/l4_feature_extract.o build_wl/l4_seq_feature.o build_wl/l4_seq_classifier.o build_wl/l4_logreg.o build_wl/l4_charcnn.o build_wl/l4_infer.o build_wl/story_reconstruct.o build_wl/syscall_name.o -o build_wl/parse_detect -L/opt/onnxruntime/lib -Wl,-rpath,/opt/onnxruntime/lib -lonnxruntime -lm
58e0e654432f6579400ee9af1285aaa776c746f2177ed27bf54f06e4b56e93ee  build_wl/parse_detect
### RUN stats-only Thứ ba, 17 Tháng 11 năm 2026 10:29:55 UTC
enovo@DESKTOP-9J5D6AQ:~/theia_e3$
```

**(2) Log sau khi xong (`tail -f run_rg_once.log`; `ls -la rep_rg_stats/`):**

```
### RUN stats-only Thứ ba, 17 Tháng 11 năm 2026 10:29:55 UTC
### XONG Thứ ba, 17 Tháng 11 năm 2026 12:00:21 UTC
Traceback (most recent call last):
  File "<stdin>", line 19, in <module>
ValueError: invalid literal for int() with base 10: ''
######## XONG Thứ ba, 17 Tháng 11 năm 2026 12:01:59 UTC
```

```
enovo@DESKTOP-9J5D6AQ:~/theia_e3$ cd ~/theia_e3/rep_rg_stats
ls -la
head -3 rg_stats.csv
wc -l rg_stats.csv
awk -F, 'NR>1{for(i=1;i<=NF;i++) if($i=="") e[i]++} END{for(i in e) print "cot",i,"rong",e[i]}' rg_stats.csv
grep RGSTAT ~/theia_e3/run_rg_once.log | head
total 835852
drwxr-xr-x  2 enovo enovo      4096 Thg 11 17 12:00 .
drwxrwxrwx 34 enovo enovo     20480 Thg 11 17 12:48 ..
-rw-r--r--  1 enovo enovo         0 Thg 11 17 10:29 empty_seeds.txt
-rw-r--r--  1 enovo enovo  15513745 Thg 11 17 11:59 mem.spill
-rw-r--r--  1 enovo enovo  52064640 Thg 11 17 11:59 net.spill
-rw-r--r--  1 enovo enovo  62776726 Thg 11 17 11:59 path.spill
-rw-r--r--  1 enovo enovo 685860369 Thg 11 17 11:59 processes.bin
-rw-r--r--  1 enovo enovo  31782546 Thg 11 17 11:59 resource_pool.bin
-rw-r--r--  1 enovo enovo   6917604 Thg 11 17 12:00 rg_stats.csv
-rw-r--r--  1 enovo enovo      3366 Thg 11 17 12:00 run_log.txt
-rw-r--r--  1 enovo enovo    932176 Thg 11 17 12:00 subj.spill
-rw-r--r--  1 enovo enovo      1457 Thg 11 17 12:00 time.txt
node_id,subject_uuid,events,pass,rg_mask,rg,via_base,cross
1,01000000-0000-0000-0000-000000000020,648,1,0,,1,1
4,D1050F00-0000-0000-0000-000000000020,9144,1,0,,1,1
121437 rg_stats.csv
cot 6 rong 41522
enovo@DESKTOP-9J5D6AQ:~/theia_e3/rep_rg_stats$
```

**(3) Phân bố cột (`cut | sort | uniq -c`) và dòng `RGSTAT`:**

```
enovo@DESKTOP-9J5D6AQ:~/theia_e3/rep_rg_stats$ cd ~/theia_e3/rep_rg_stats
cut -d, -f5 rg_stats.csv | sort | uniq -c | sort -rn | head -20
cut -d, -f6 rg_stats.csv | sort | uniq -c | sort -rn | head -20
cut -d, -f4,7,8 rg_stats.csv | sort | uniq -c
grep -i RGSTAT run_log.txt time.txt | head
  69350 1
  41522 0
   9586 16
    978 17
      1 rg_mask
  69350 RG1
  41522
   9586 RG5
    978 RG1|RG5
      1 rg
  18229 0,0,0
  55152 1,0,0
   8686 1,0,1
   7402 1,1,0
  31967 1,1,1
      1 pass,via_base,cross
run_log.txt:RGSTAT RG1 any=70328 sole=63550
run_log.txt:RGSTAT RG2 any=0 sole=0
run_log.txt:RGSTAT RG3 any=0 sole=0
run_log.txt:RGSTAT RG4 any=0 sole=0
run_log.txt:RGSTAT RG5 any=10564 sole=3
enovo@DESKTOP-9J5D6AQ:~/theia_e3/rep_rg_stats$
```

**(4) Tóm tắt GT (`rg_summarize.py`):**

```
enovo@DESKTOP-9J5D6AQ:~/theia_e3/rep_rg_stats$ cp /mnt/c/Users/Lenovo/Downloads/rg_summarize.py ~/theia_e3/
python3 -I ~/theia_e3/rg_summarize.py
GT file: /home/enovo/theia_e3/node_Browser_Extension_Drakon_Dropper.csv
GT file: /home/enovo/theia_e3/node_Firefox_Backdoor_Drakon_In_Memory.csv
GT uuids: 118
rows 121436 pass 103207 GT_in_stats 96 GT_pass 85
RG1 any 70328 only 69350 only_notbase 63550 | GT any 18 only 10 only_notbase 10
RG2 any 0 only 0 only_notbase 0 | GT any 0 only 0 only_notbase 0
RG3 any 0 only 0 only_notbase 0 | GT any 0 only 0 only_notbase 0
RG4 any 0 only 0 only_notbase 0 | GT any 0 only 0 only_notbase 0
RG5 any 10564 only 9586 only_notbase 3 | GT any 10 only 2 only_notbase 2
GT pass without any RG (base-bit path only): 65
enovo@DESKTOP-9J5D6AQ:~/theia_e3/rep_rg_stats$
```

Không có trong phiên làm việc này (nằm ở phần trước, không còn nguyên văn): output ablation RG1–RG5 chạy thật (bảng ở mục 3c(a) lấy từ bản tóm tắt). Cần dán lại file log gốc nếu muốn đính kèm nguyên văn.

### 6.7 Mimicry replay (`mimicry_replay.py`, `run_mimicry_replay.sh`), 17–18/11/2026

**Ghi chú về hai lần khởi động.** Lần dán đầu (sau 60 giây) ghi `SINH FILE ... 10:29:37 UTC`, sau sinh file là `RUN pipeline ... 12:58:13 UTC`. Log cuối cùng (`run_mimr.log` lúc chạy xong) ghi `SINH FILE` và `RUN pipeline` cùng `17:20:28 UTC`, không có các dòng sinh file (đã dùng lại file đã sinh), và `time.txt` cho wall 7:27:05, kết thúc 00:49:04Z 18/11 (khớp bắt đầu ~17:22). Tức `run_mimr.log` và thư mục `rep_mimr_heavy_N1000000` thuộc **lần chạy pipeline bắt đầu 17:20:28Z**; lần chạy bắt đầu 12:58:13Z bị ghi đè (lý do chưa rõ; chưa hỏi). Kết quả ở mục 3d lấy từ lần 17:20:28Z.

**(1) Khởi chạy lần đầu và sinh file (đầu `run_mimr.log`, lần 10:29:37Z → 12:58:13Z):**

```
(hfenv) enovo@DESKTOP-9J5D6AQ:~/theia_e3_test_mimicry$ cp /mnt/c/Users/Lenovo/Downloads/{mimicry_replay.py,run_mimicry_replay.sh} ~/theia_e3_test_mimicry/
cd ~/theia_e3_test_mimicry && KIND=heavy N=1000000 bash run_mimicry_replay.sh > run_mimr.log 2>&1 &
sleep 60; head -12 run_mimr.log
[1] 4343
81438c442e5a783a8e2011b9170e3dbcfca9e2a52fc5017723bd7678b00ed321  /home/enovo/theia_e3_test_mimicry/parse_detect
######## SINH FILE (heavy, N=1000000)  Thứ ba, 17 Tháng 11 năm 2026 10:29:37 UTC
(hfenv) enovo@DESKTOP-9J5D6AQ:~/theia_e3_test_mimicry$ top
```

```
81438c442e5a783a8e2011b9170e3dbcfca9e2a52fc5017723bd7678b00ed321  /home/enovo/theia_e3_test_mimicry/parse_detect
######## SINH FILE (heavy, N=1000000)  Thứ ba, 17 Tháng 11 năm 2026 10:29:37 UTC
  [A] 2M rec 135s (ta1-theia-e3-official-6r.bin)
  [A] 4M rec 279s (ta1-theia-e3-official-6r.bin)
  [A] 6M rec 416s (ta1-theia-e3-official-6r.bin.1)
  [A] 8M rec 556s (ta1-theia-e3-official-6r.bin.1)
  [A] 10M rec 698s (ta1-theia-e3-official-6r.bin.1)
  [A] 12M rec 838s (ta1-theia-e3-official-6r.bin.2)
  [A] 14M rec 998s (ta1-theia-e3-official-6r.bin.2)
  [A] 16M rec 1173s (ta1-theia-e3-official-6r.bin.3)
  [A] 18M rec 1370s (ta1-theia-e3-official-6r.bin.3)
  [A] 20M rec 1560s (ta1-theia-e3-official-6r.bin.3)
  [A] 22M rec 1713s (ta1-theia-e3-official-6r.bin.4)
  [A] 24M rec 1860s (ta1-theia-e3-official-6r.bin.4)
  [A] 26M rec 2022s (ta1-theia-e3-official-6r.bin.5)
  [A] 28M rec 2228s (ta1-theia-e3-official-6r.bin.5)
  [A] 30M rec 2451s (ta1-theia-e3-official-6r.bin.5)
  [A] 32M rec 2608s (ta1-theia-e3-official-6r.bin.6)
  [A] 34M rec 2768s (ta1-theia-e3-official-6r.bin.6)
  [A] 36M rec 2933s (ta1-theia-e3-official-6r.bin.7)
  [A] 38M rec 3213s (ta1-theia-e3-official-6r.bin.7)
  [A] 40M rec 3518s (ta1-theia-e3-official-6r.bin.7)
  [A] 42M rec 3732s (ta1-theia-e3-official-6r.bin.8)
  [A] 44M rec 3961s (ta1-theia-e3-official-6r.bin.8)
  [A] 46M rec 4179s (ta1-theia-e3-official-6r.bin.9)
  [A] 48M rec 4394s (ta1-theia-e3-official-6r.bin.9)
  [A] 50M rec 4613s (ta1-theia-e3-official-6r.bin.9)
  [A] 52M rec 4815s (ta1-theia-e3-official-6r.bin.10)
  [A] 54M rec 5031s (ta1-theia-e3-official-6r.bin.10)
  [A] 56M rec 5246s (ta1-theia-e3-official-6r.bin.11)
  [A] 58M rec 5592s (ta1-theia-e3-official-6r.bin.11)
  [A] 60M rec 5990s (ta1-theia-e3-official-6r.bin.11)
  [A] 62M rec 6165s (ta1-theia-e3-official-6r.bin.12)
  [A] xong 63852494 record 6295s
seed 4e18b22a  events=25 host_file=ta1-theia-e3-official-6r.bin ts=[1523385082992535880..1523385345189136549]  pool(heavy)=1  types={'EVENT_CONNECT': 1, 'EVENT_OPEN': 1, 'EVENT_READ': 1, 'EVENT_MMAP': 1, 'EVENT_SENDTO': 4, 'EVENT_RECVFROM': 2, 'EVENT_CLONE': 11, 'EVENT_MPROTECT': 4}
seed 6818a92b  events=64 host_file=ta1-theia-e3-official-6r.bin ts=[1523385330168523991..1523385867843495196]  pool(heavy)=2  types={'EVENT_EXECUTE': 1, 'EVENT_OPEN': 4, 'EVENT_MMAP': 7, 'EVENT_READ': 3, 'EVENT_MPROTECT': 24, 'EVENT_CONNECT': 1, 'EVENT_SENDTO': 4, 'EVENT_RECVFROM': 2, 'EVENT_CLONE': 18}
seed c21f0f30  events=22 host_file=ta1-theia-e3-official-6r.bin ts=[1523386456645767918..1523386606400895234]  pool(heavy)=1  types={'EVENT_CONNECT': 1, 'EVENT_OPEN': 1, 'EVENT_READ': 1, 'EVENT_MMAP': 1, 'EVENT_SENDTO': 4, 'EVENT_RECVFROM': 2, 'EVENT_CLONE': 7, 'EVENT_MPROTECT': 5}
seed 7235b0b4  events=569 host_file=ta1-theia-e3-official-6r.bin.8 ts=[1523551481543364366..1523551481621262419]  pool(heavy)=2  types={'EVENT_EXECUTE': 1, 'EVENT_OPEN': 6, 'EVENT_MMAP': 344, 'EVENT_READ': 6, 'EVENT_MPROTECT': 10, 'EVENT_CONNECT': 1, 'EVENT_SENDTO': 1, 'EVENT_RECVFROM': 200}
[B] ta1-theia-e3-official-6r.bin -> /home/enovo/theia_e3_test_mimicry/mimr_heavy_N1000000/ta1-theia-e3-official-6r.bin  chen={'4e18b22a': 1000000, '6818a92b': 1000000, 'c21f0f30': 1000000}  1403s
[B] ta1-theia-e3-official-6r.bin.8 -> /home/enovo/theia_e3_test_mimicry/mimr_heavy_N1000000/ta1-theia-e3-official-6r.bin.8  chen={'7235b0b4': 1000000}  1192s
manifest: /home/enovo/theia_e3_test_mimicry/mimr_heavy_N1000000/replay_manifest.json
thay: ta1-theia-e3-official-6r.bin -> /home/enovo/theia_e3_test_mimicry/mimr_heavy_N1000000/ta1-theia-e3-official-6r.bin
thay: ta1-theia-e3-official-6r.bin.8 -> /home/enovo/theia_e3_test_mimicry/mimr_heavy_N1000000/ta1-theia-e3-official-6r.bin.8
######## RUN pipeline  Thứ ba, 17 Tháng 11 năm 2026 12:58:13 UTC
```

**(2) Giữa lúc chạy (18/11 06:06 giờ VN):**

```
enovo@DESKTOP-9J5D6AQ:~/theia_e3_test_mimicry/rep_mimr_heavy_N1000000$ ls
export.csv  export.csv.infer_status  export.csv.infer_status.pending_spillover  mem.spill  net.spill  path.spill  processes.bin  run_log.txt  subj.spill  time.txt
[1]-  Done                    cd ~/theia_e3 && nohup bash run_rg_once.sh > run_rg_once.log 2>&1  (wd: /mnt/c/Users/Lenovo)
(wd now: ~/theia_e3_test_mimicry/rep_mimr_heavy_N1000000)
```

```
(wd now: ~/theia_e3_test_mimicry/rep_mimr_heavy_N1000000)
enovo@DESKTOP-9J5D6AQ:~/theia_e3_test_mimicry/rep_mimr_heavy_N1000000$ cd ~/theia_e3_test_mimicry
jobs; pgrep -a parse_detect | cut -c1-80
cat rep_mimr_heavy_N1000000/time.txt
tail -25 run_mimr.log
ls -la rep_mimr_heavy_N1000000/ | head -20
tail -5 rep_mimr_heavy_N1000000/run_log.txt
dmesg 2>/dev/null | grep -i -E "killed process|out of memory" | tail -3
[2]+  Running                 cd ~/theia_e3_test_mimicry && KIND=heavy N=1000000 bash run_mimicry_replay.sh > run_mimr.log 2>&1 &  (wd: /mnt/c/Users/Lenovo)
1120 /home/enovo/theia_e3_test_mimicry/parse_detect /home/enovo/theia_e3_test_mi
81438c442e5a783a8e2011b9170e3dbcfca9e2a52fc5017723bd7678b00ed321  /home/enovo/theia_e3_test_mimicry/parse_detect
######## SINH FILE (heavy, N=1000000)  Thứ ba, 17 Tháng 11 năm 2026 17:20:28 UTC
thay: ta1-theia-e3-official-6r.bin -> /home/enovo/theia_e3_test_mimicry/mimr_heavy_N1000000/ta1-theia-e3-official-6r.bin
thay: ta1-theia-e3-official-6r.bin.8 -> /home/enovo/theia_e3_test_mimicry/mimr_heavy_N1000000/ta1-theia-e3-official-6r.bin.8
######## RUN pipeline  Thứ ba, 17 Tháng 11 năm 2026 17:20:28 UTC
total 797788
drwxr-xr-x 2 enovo enovo      4096 Thg 11 17 17:20 .
drwxr-xr-x 7 enovo enovo     20480 Thg 11 17 17:56 ..
-rw-r--r-- 1 enovo enovo  10985472 Thg 11 17 23:05 export.csv
-rw-r--r-- 1 enovo enovo   5530558 Thg 11 17 23:04 export.csv.infer_status
-rw-r--r-- 1 enovo enovo   1845696 Thg 11 17 23:04 export.csv.infer_status.pending_spillover
-rw-r--r-- 1 enovo enovo  13774848 Thg 11 17 23:05 mem.spill
-rw-r--r-- 1 enovo enovo  42844160 Thg 11 17 23:05 net.spill
-rw-r--r-- 1 enovo enovo  57274368 Thg 11 17 23:05 path.spill
-rw-r--r-- 1 enovo enovo 683835661 Thg 11 17 23:05 processes.bin
-rw-r--r-- 1 enovo enovo     59636 Thg 11 17 23:05 run_log.txt
-rw-r--r-- 1 enovo enovo    712704 Thg 11 17 23:00 subj.spill
-rw-r--r-- 1 enovo enovo         0 Thg 11 17 17:20 time.txt
[WARN][L4_COLLECT] seed=106687 node_id=106687: disk co 750664 event, vuot member_hard_cap=200000 - CAT xuong 200000 (xem events_truncated o CSV).
[WARN][L4_COLLECT] seed=2732 node_id=2732: disk co 749879 event, vuot member_hard_cap=200000 - CAT xuong 200000 (xem events_truncated o CSV).
[11/13] /home/enovo/theia_e3_test_mimicry/ta1-theia-e3-official-6r.bin.10 xong - total=59000000 subj=141420 written=21945300 no_state=0 exits=0
[WARN][L4_COLLECT] seed=106687 node_id=106687: disk co 750664 event, vuot member_hard_cap=200000 - CAT xuong 200000 (xem events_truncated o CSV).
[WARN][L4_COLLECT] seed=2732 node_id=2732: disk co 749879 event, vuot member_hard_cap=200000 - CAT xuong 200000 (xem events_truncated o CSV).
enovo@DESKTOP-9J5D6AQ:~/theia_e3_test_mimicry$
```

```
(11:39 giờ VN)
enovo@DESKTOP-9J5D6AQ:~/theia_e3_test_mimicry$ tail -5 rep_mimr_heavy_N1000000/run_log.txt
[WARN][L4_COLLECT] seed=2732 node_id=2732: disk co 749879 event, vuot member_hard_cap=200000 - CAT xuong 200000 (xem events_truncated o CSV).
[WARN][L4_COLLECT] seed=106687 node_id=106687: disk co 750664 event, vuot member_hard_cap=200000 - CAT xuong 200000 (xem events_truncated o CSV).
[WARN][L4_COLLECT] seed=2732 node_id=2732: disk co 749879 event, vuot member_hard_cap=200000 - CAT xuong 200000 (xem events_truncated o CSV).
[WARN][L4_COLLECT] seed=106687 node_id=106687: disk co 750664 event, vuot member_hard_cap=200000 - CAT xuong 200000 (xem events_truncated o CSV).
[WARN][L4_COLLECT] seed=2732 node_id=2732: disk co 749879 event, vuot member_hard_cap=200000 - CAT xuong 200000 (xem events_truncated o CSV).
enovo@DESKTOP-9J5D6AQ:~/theia_e3_test_mimicry$ cd rep_mimr_*
enovo@DESKTOP-9J5D6AQ:~/theia_e3_test_mimicry/rep_mimr_heavy_N1000000$ ls
export.csv  export.csv.infer_status  export.csv.infer_status.pending_spillover  mem.spill  net.spill  path.spill  processes.bin  run_log.txt  subj.spill  time.txt
enovo@DESKTOP-9J5D6AQ:~/theia_e3_test_mimicry/rep_mimr_heavy_N1000000$ nano export.csv
enovo@DESKTOP-9J5D6AQ:~/theia_e3_test_mimicry/rep_mimr_heavy_N1000000$ nano export.csv
[2]+  Done                    cd ~/theia_e3_test_mimicry && KIND=heavy N=1000000 bash run_mimicry_replay.sh > run_mimr.log 2>&1 &  (wd: /mnt/c/Users/Lenovo)
(wd now: ~/theia_e3_test_mimicry/rep_mimr_heavy_N1000000)
```

**(3) Kết quả cuối (`cat time.txt`; `tail -60 run_mimr.log`), nguyên văn:**

```
enovo@DESKTOP-9J5D6AQ:~/theia_e3_test_mimicry/rep_mimr_heavy_N1000000$ cd ~/theia_e3_test_mimicry
cat rep_mimr_heavy_N1000000/time.txt
tail -60 run_mimr.log
        Command being timed: "/home/enovo/theia_e3_test_mimicry/parse_detect /home/enovo/theia_e3_test_mimicry/mimr_heavy_N1000000/ta1-theia-e3-official-6r.bin /home/enovo/theia_e3_test_mimicry/ta1-theia-e3-official-6r.bin.1 /home/enovo/theia_e3_test_mimicry/ta1-theia-e3-official-6r.bin.2 /home/enovo/theia_e3_test_mimicry/ta1-theia-e3-official-6r.bin.3 /home/enovo/theia_e3_test_mimicry/ta1-theia-e3-official-6r.bin.4 /home/enovo/theia_e3_test_mimicry/ta1-theia-e3-official-6r.bin.5 /home/enovo/theia_e3_test_mimicry/ta1-theia-e3-official-6r.bin.6 /home/enovo/theia_e3_test_mimicry/ta1-theia-e3-official-6r.bin.7 /home/enovo/theia_e3_test_mimicry/mimr_heavy_N1000000/ta1-theia-e3-official-6r.bin.8 /home/enovo/theia_e3_test_mimicry/ta1-theia-e3-official-6r.bin.9 /home/enovo/theia_e3_test_mimicry/ta1-theia-e3-official-6r.bin.10 /home/enovo/theia_e3_test_mimicry/ta1-theia-e3-official-6r.bin.11 /home/enovo/theia_e3_test_mimicry/ta1-theia-e3-official-6r.bin.12"
        User time (seconds): 90030.90
        System time (seconds): 1512.55
        Percent of CPU this job got: 341%
        Elapsed (wall clock) time (h:mm:ss or m:ss): 7:27:05
        Average shared text size (kbytes): 0
        Average unshared data size (kbytes): 0
        Average stack size (kbytes): 0
        Average total size (kbytes): 0
        Maximum resident set size (kbytes): 4592036
        Average resident set size (kbytes): 0
        Major (requiring I/O) page faults: 180
        Minor (reclaiming a frame) page faults: 86844016
        Voluntary context switches: 34420
        Involuntary context switches: 121339
        Swaps: 0
        File system inputs: 20324056
        File system outputs: 1907056
        Socket messages sent: 0
        Socket messages received: 0
        Signals delivered: 0
        Page size (bytes): 4096
        Exit status: 0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,200000,SCORED,1.000000,1,13,0
7235B0B4-0200-0000-0000-000000000020,106687,C,106687,1,1,200000,SCORED,1.000000,1,13,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,200000,SCORED,1.000000,1,13,0
7235B0B4-0200-0000-0000-000000000020,106687,C,106687,1,1,200000,SCORED,1.000000,1,13,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,200000,SCORED,1.000000,1,13,0
7235B0B4-0200-0000-0000-000000000020,106687,C,106687,1,1,200000,SCORED,1.000000,1,13,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,200000,SCORED,1.000000,1,13,0
7235B0B4-0200-0000-0000-000000000020,106687,C,106687,1,1,200000,SCORED,1.000000,1,13,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,200000,SCORED,1.000000,1,13,0
7235B0B4-0200-0000-0000-000000000020,106687,C,106687,1,1,200000,SCORED,1.000000,1,13,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,200000,SCORED,1.000000,1,13,0
7235B0B4-0200-0000-0000-000000000020,106687,C,106687,1,1,200000,SCORED,1.000000,1,13,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,200000,SCORED,1.000000,1,13,0
7235B0B4-0200-0000-0000-000000000020,106687,C,106687,1,1,200000,SCORED,1.000000,1,13,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,200000,SCORED,1.000000,1,13,0
7235B0B4-0200-0000-0000-000000000020,106687,C,106687,1,1,200000,SCORED,1.000000,1,13,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,200000,SCORED,1.000000,1,13,0
7235B0B4-0200-0000-0000-000000000020,106687,C,106687,1,1,200000,SCORED,1.000000,1,13,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,200000,SCORED,1.000000,1,13,0
7235B0B4-0200-0000-0000-000000000020,106687,C,106687,1,1,200000,SCORED,1.000000,1,13,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,200000,SCORED,1.000000,1,13,0
7235B0B4-0200-0000-0000-000000000020,106687,C,106687,1,1,200000,SCORED,1.000000,1,13,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,200000,SCORED,1.000000,1,13,0
7235B0B4-0200-0000-0000-000000000020,106687,C,106687,1,1,200000,SCORED,1.000000,1,13,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,200000,SCORED,1.000000,1,13,0
7235B0B4-0200-0000-0000-000000000020,106687,C,106687,1,1,200000,SCORED,1.000000,1,13,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,200000,SCORED,1.000000,1,13,0
7235B0B4-0200-0000-0000-000000000020,106687,C,106687,1,1,200000,SCORED,1.000000,1,13,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,200000,SCORED,1.000000,1,13,0
7235B0B4-0200-0000-0000-000000000020,106687,C,106687,1,1,200000,SCORED,1.000000,1,13,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,200000,SCORED,1.000000,1,13,0
7235B0B4-0200-0000-0000-000000000020,106687,C,106687,1,1,200000,SCORED,1.000000,1,13,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,200000,SCORED,1.000000,1,13,0
7235B0B4-0200-0000-0000-000000000020,106687,C,106687,1,1,200000,SCORED,1.000000,1,13,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,200000,SCORED,1.000000,1,13,0
7235B0B4-0200-0000-0000-000000000020,106687,C,106687,1,1,200000,SCORED,1.000000,1,13,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,200000,SCORED,1.000000,1,13,0
7235B0B4-0200-0000-0000-000000000020,106687,C,106687,1,1,200000,SCORED,1.000000,1,13,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,200000,SCORED,1.000000,1,13,0
7235B0B4-0200-0000-0000-000000000020,106687,C,106687,1,1,200000,SCORED,1.000000,1,13,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,200000,SCORED,1.000000,1,13,0
7235B0B4-0200-0000-0000-000000000020,106687,C,106687,1,1,200000,SCORED,1.000000,1,13,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,200000,SCORED,1.000000,1,13,0
7235B0B4-0200-0000-0000-000000000020,106687,C,106687,1,1,200000,SCORED,1.000000,1,13,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,200000,SCORED,1.000000,1,13,0
7235B0B4-0200-0000-0000-000000000020,106687,C,106687,1,1,200000,SCORED,1.000000,1,13,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,200000,SCORED,1.000000,1,13,0
7235B0B4-0200-0000-0000-000000000020,106687,C,106687,1,1,200000,SCORED,1.000000,1,13,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,200000,SCORED,1.000000,1,13,0
7235B0B4-0200-0000-0000-000000000020,106687,C,106687,1,1,200000,SCORED,1.000000,1,13,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,200000,SCORED,1.000000,1,13,0
7235B0B4-0200-0000-0000-000000000020,106687,C,106687,1,1,200000,SCORED,1.000000,1,13,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,200000,SCORED,1.000000,1,13,0
--- story baseline
 seed_node           start(UTC) pre/post members   GT  TP/FP
      2701  2018-04-10 18:31:22      pre      13   12  TP   4E18B22A
      2732  2018-04-10 18:35:30      pre      58   24  TP   6818A92B
      4615  2018-04-10 18:54:16      pre       9    8  TP   C21F0F30
    106687  2018-04-12 16:44:41     post      42   39  TP   7235B0B4
######## XONG Thứ tư, 18 Tháng 11 năm 2026 00:49:04 UTC
enovo@DESKTOP-9J5D6AQ:~/theia_e3_test_mimicry$
```

**(4) Phần đầu của kết quả (`sed -n 1,45p run_mimr.log`) và `replay_manifest.json`:**

```
enovo@DESKTOP-9J5D6AQ:~/theia_e3_test_mimicry$ cd ~/theia_e3_test_mimicry
sed -n 1,45p run_mimr.log
python3 -c "import json;d=json.load(open('mimr_heavy_N1000000/replay_manifest.json'));print(json.dumps(d,indent=1)[:1500])"
81438c442e5a783a8e2011b9170e3dbcfca9e2a52fc5017723bd7678b00ed321  /home/enovo/theia_e3_test_mimicry/parse_detect
######## SINH FILE (heavy, N=1000000)  Thứ ba, 17 Tháng 11 năm 2026 17:20:28 UTC
thay: ta1-theia-e3-official-6r.bin -> /home/enovo/theia_e3_test_mimicry/mimr_heavy_N1000000/ta1-theia-e3-official-6r.bin
thay: ta1-theia-e3-official-6r.bin.8 -> /home/enovo/theia_e3_test_mimicry/mimr_heavy_N1000000/ta1-theia-e3-official-6r.bin.8
######## RUN pipeline  Thứ ba, 17 Tháng 11 năm 2026 17:20:28 UTC
[mimr_heavy_N1000000] story that = 5 | stub covered_by = 3 | GT96 = 96 (GT49 sau cutoff = 49)
-- TOAN BO story --
  story: 5 | story chua GT (TP): 5 | story precision = 1.0000
  member process phan biet: 120
  recall GT96  : 83/96 = 0.8646  Wilson [0.782,0.919]
  recall GT49  : 39/49 = 0.7959  Wilson [0.664,0.885]
  PHU SONG: member = 120 / 169650 process = 0.1% | GT96 ky vong neu phu ngau nhien cung ty le = 0.1 | lift = 1222.30x (tran = 1413.75x) | P(ngau nhien >= 83) = 1.77e-263
  PHU SONG sau cutoff: member sau cutoff = 58 / 63225 = 0.1% | GT49 ky vong = 0.0 | P(ngau nhien >= 39) = 9.33e-117
-- chi story co seed start >= cutoff --
  story: 1 | story chua GT (TP): 1 | story precision = 1.0000
  member process phan biet: 42
  recall GT96  : 39/96 = 0.4062  Wilson [0.313,0.506]
  recall GT49  : 39/49 = 0.7959  Wilson [0.664,0.885]
  PHU SONG: member = 42 / 169650 process = 0.0% | GT96 ky vong neu phu ngau nhien cung ty le = 0.0 | lift = 1640.96x (tran = 4039.29x) | P(ngau nhien >= 39) = 3.14e-127
  PHU SONG sau cutoff: member sau cutoff = 40 / 63225 = 0.1% | GT49 ky vong = 0.0 | P(ngau nhien >= 39) = 3.95e-130
 seed_node           start(UTC) pre/post members   GT  TP/FP
      2701  2018-04-10 18:31:22      pre      13   12  TP   4E18B22A
      2732  2018-04-10 18:35:30      pre      19   18  TP   6818A92B
      4615  2018-04-10 18:54:16      pre       9    8  TP   C21F0F30
      4673  2018-04-10 18:56:39      pre      39    6  TP   FB1F9E30
    106687  2018-04-12 16:44:41     post      42   39  TP   7235B0B4
######## DIEM L4 4 SEED: baseline vs sau khi chen (cot: node_id,table,..,agg_total_events,status,score,verdict)
--- baseline
4E18B22A-0000-0000-0000-000000000020,2701,C,2701,1,1,1,SCORED,1.000000,1,1,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,1,1,1,SCORED,0.000001,0,1,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,5,SCORED,1.000000,1,1,0
C21F0F30-0000-0000-0000-000000000020,4615,C,4615,1,1,1,SCORED,1.000000,1,1,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,27,SCORED,1.000000,1,1,0
7235B0B4-0200-0000-0000-000000000020,106687,D,132623,9,1,1,SCORED,0.000000,0,1,0
7235B0B4-0200-0000-0000-000000000020,106687,C,106687,1,1,8,SCORED,0.982396,1,1,0
--- sau chen
4E18B22A-0000-0000-0000-000000000020,2701,C,2701,1,1,1,SCORED,1.000000,1,1,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,1,1,1,SCORED,0.000001,0,1,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,9,SCORED,1.000000,1,1,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,8099,SCORED,1.000000,1,1,0
4E18B22A-0000-0000-0000-000000000020,2701,C,2701,1,1,31,SCORED,1.000000,1,1,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,129878,SCORED,1.000000,1,8,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,200022,SCORED,1.000000,1,13,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,200114,SCORED,1.000000,1,13,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,200016,SCORED,1.000000,1,13,0
{
 "kind": "heavy",
 "n": 1000000,
 "rng": 1,
 "replaced": {
  "/home/enovo/theia_e3_test_mimicry/ta1-theia-e3-official-6r.bin": "/home/enovo/theia_e3_test_mimicry/mimr_heavy_N1000000/ta1-theia-e3-official-6r.bin",
  "/home/enovo/theia_e3_test_mimicry/ta1-theia-e3-official-6r.bin.8": "/home/enovo/theia_e3_test_mimicry/mimr_heavy_N1000000/ta1-theia-e3-official-6r.bin.8"
 },
 "injected": {
  "ta1-theia-e3-official-6r.bin": {
   "4e18b22a": 1000000,
   "6818a92b": 1000000,
   "c21f0f30": 1000000
  },
  "ta1-theia-e3-official-6r.bin.8": {
   "7235b0b4": 1000000
  }
 }
}
enovo@DESKTOP-9J5D6AQ:~/theia_e3_test_mimicry$
```

**(5) `infer_status` sau chèn theo seed (`sort -u`; baseline: lệnh dùng sai đường dẫn `rep_l4_s1/export.csv.infer_status`; file thật tên `rep_l4_s1/export_l4_s1.csv.infer_status`, nên không in gì):**

```
enovo@DESKTOP-9J5D6AQ:~/theia_e3_test_mimicry$ for u in 4E18B22A 6818A92B C21F0F30 7235B0B4; do echo "== $u"; grep "^$u" rep_mimr_heavy_N1000000/export.csv.infer_status | sort -u; done
echo "== baseline"; for u in 4E18B22A 6818A92B C21F0F30 7235B0B4; do grep "^$u" ~/theia_e3/rep_l4_s1/export.csv.infer_status 2>/dev/null | sort -u; done
== 4E18B22A
4E18B22A-0000-0000-0000-000000000020,2701,C,2701,1,1,1,SCORED,1.000000,1,1,0
4E18B22A-0000-0000-0000-000000000020,2701,C,2701,1,1,31,SCORED,1.000000,1,1,0
== 6818A92B
6818A92B-0000-0000-0000-000000000020,2732,A,6590,1,1,1,SCORED,0.000001,0,1,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,129878,SCORED,1.000000,1,8,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,200000,SCORED,1.000000,1,13,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,200016,SCORED,1.000000,1,13,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,200022,SCORED,1.000000,1,13,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,200055,SCORED,1.000000,1,13,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,200103,SCORED,1.000000,1,13,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,200114,SCORED,1.000000,1,13,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,8099,SCORED,1.000000,1,1,0
6818A92B-0000-0000-0000-000000000020,2732,A,6590,2,1,9,SCORED,1.000000,1,1,0
== C21F0F30
C21F0F30-0000-0000-0000-000000000020,4615,C,4615,1,1,1,SCORED,1.000000,1,1,0
C21F0F30-0000-0000-0000-000000000020,4615,C,4615,1,1,21,SCORED,1.000000,1,1,0
== 7235B0B4
7235B0B4-0200-0000-0000-000000000020,106687,C,106687,1,1,200000,SCORED,1.000000,1,13,0
7235B0B4-0200-0000-0000-000000000020,106687,C,106687,1,1,200072,SCORED,1.000000,1,13,0
7235B0B4-0200-0000-0000-000000000020,106687,C,106687,1,1,200076,SCORED,1.000000,1,13,0
7235B0B4-0200-0000-0000-000000000020,106687,C,106687,1,1,200181,SCORED,1.000000,1,13,0
7235B0B4-0200-0000-0000-000000000020,106687,C,106687,1,1,200239,SCORED,1.000000,1,13,0
7235B0B4-0200-0000-0000-000000000020,106687,C,106687,1,1,200245,SCORED,1.000000,1,13,0
7235B0B4-0200-0000-0000-000000000020,106687,D,132623,9,1,1,SCORED,0.000000,0,1,0
== baseline
enovo@DESKTOP-9J5D6AQ:~/theia_e3_test_mimicry$
```

**(6) `export.csv` (header) và `agg_total_events` theo seed:**

```
enovo@DESKTOP-9J5D6AQ:~/theia_e3_test_mimicry/rep_mimr_heavy_N1000000$ cat export.csv | head -1
subject_uuid,node_id,table,priority_key,priority_count,member_count,agg_total_events,member_truncated,events_truncated,causal_graph_evict_total,iv1,iv2,iv3,iv4,iv5,iv6,iv7,iv9,iv11,iv12,iv13,n_iv_distinct,critical_iv_exist,cross_process_flag,external_ip_exists,cmdline
```

```
enovo@DESKTOP-9J5D6AQ:~/theia_e3_test_mimicry$ cd ~/theia_e3_test_mimicry
for f in rep_mimr_heavy_N1000000/export.csv ~/theia_e3/rep_l4_s1/export.csv; do
  echo "=== $f"
  awk -F, 'NR==1{next} tolower($1) ~ /^(4e18b22a|6818a92b|c21f0f30|7235b0b4)/ {print $1,$2,"agg_total_events="$7,"events_truncated="$9}' "$f" | sort -u
done
=== rep_mimr_heavy_N1000000/export.csv
4E18B22A-0000-0000-0000-000000000020 2701 agg_total_events=1 events_truncated=0
4E18B22A-0000-0000-0000-000000000020 2701 agg_total_events=31 events_truncated=0
6818A92B-0000-0000-0000-000000000020 2732 agg_total_events=129878 events_truncated=0
6818A92B-0000-0000-0000-000000000020 2732 agg_total_events=1 events_truncated=0
6818A92B-0000-0000-0000-000000000020 2732 agg_total_events=200000 events_truncated=1
6818A92B-0000-0000-0000-000000000020 2732 agg_total_events=200016 events_truncated=1
6818A92B-0000-0000-0000-000000000020 2732 agg_total_events=200022 events_truncated=1
6818A92B-0000-0000-0000-000000000020 2732 agg_total_events=200055 events_truncated=1
6818A92B-0000-0000-0000-000000000020 2732 agg_total_events=200103 events_truncated=1
6818A92B-0000-0000-0000-000000000020 2732 agg_total_events=200114 events_truncated=1
6818A92B-0000-0000-0000-000000000020 2732 agg_total_events=8099 events_truncated=0
6818A92B-0000-0000-0000-000000000020 2732 agg_total_events=9 events_truncated=0
7235B0B4-0200-0000-0000-000000000020 106687 agg_total_events=1 events_truncated=0
7235B0B4-0200-0000-0000-000000000020 106687 agg_total_events=200000 events_truncated=1
7235B0B4-0200-0000-0000-000000000020 106687 agg_total_events=200072 events_truncated=1
7235B0B4-0200-0000-0000-000000000020 106687 agg_total_events=200076 events_truncated=1
7235B0B4-0200-0000-0000-000000000020 106687 agg_total_events=200181 events_truncated=1
7235B0B4-0200-0000-0000-000000000020 106687 agg_total_events=200239 events_truncated=1
7235B0B4-0200-0000-0000-000000000020 106687 agg_total_events=200245 events_truncated=1
C21F0F30-0000-0000-0000-000000000020 4615 agg_total_events=1 events_truncated=0
C21F0F30-0000-0000-0000-000000000020 4615 agg_total_events=21 events_truncated=0
=== /home/enovo/theia_e3/rep_l4_s1/export.csv
awk: fatal: cannot open file `/home/enovo/theia_e3/rep_l4_s1/export.csv' for reading: No such file or directory
enovo@DESKTOP-9J5D6AQ:~/theia_e3_test_mimicry$
```

**(7) Tra `log.txt` (UUID viết thường; `log.txt` là bảng tóm tắt theo process, không liệt kê sự kiện):**

```
enovo@DESKTOP-9J5D6AQ:~/theia_e3_test_mimicry$ cd ~/theia_e3
grep -c "4E18B22A" log.txt; grep "4E18B22A" log.txt | grep -i -E "connect|execve|execute" | head -5
grep -c "C21F0F30" log.txt; grep "C21F0F30" log.txt | grep -i -E "connect|execve|execute" | head -5
0
0
```

```
enovo@DESKTOP-9J5D6AQ:~/theia_e3$ cd ~/theia_e3
for u in 4e18b22a c21f0f30 6818a92b 7235b0b4; do
  echo "== $u: tong dong = $(grep -ic "$u" log.txt)"
  grep -i "$u" log.txt | grep -i -E "connect|execve|execute" | head -3
done
head -3 log.txt
== 4e18b22a: tong dong = 2
== c21f0f30: tong dong = 2
== 6818a92b: tong dong = 2
== 7235b0b4: tong dong = 2
======================================================================
EDR PROCESS EVENT LOG
processes: 169651  shown: 169651
enovo@DESKTOP-9J5D6AQ:~/theia_e3$
```

**(8) Nguồn lập luận về RLE (trích code, không phải output chạy):** `process_state_update.c`, hàm `net_event_same()` so `syscall_id`, `behavior`, `src_id`, `dst_id`, `src_port`, `dst_port`, `address_family`; nếu giống hệt `pending_net` thì `pending_net.count++` và `return 0` (không đẩy vào `event_history`). Tìm thấy RLE cho network, file (`file_event_same()`) và transfer (`transfer_event_same()`); không thấy RLE cho exec trong các đoạn code đã đọc.

**(9) Replay benign N=250.000 mỗi seed, 19–20/11/2026 (nguyên văn output lần dán sau khi chạy xong; lần N=1.000.000 trước đó bị BSOD, tiến trình mồ côi bị `kill`, không có số):**

```
enovo@DESKTOP-9J5D6AQ:~/theia_e3_test_mimicry/rep_mimr_benign_N250000$ cd ~/theia_e3_test_mimicry/rep_mimr_benign_N250000; cat eval.txt list.txt; grep -E "Elapsed|Maximum resident|Exit status" time.txt; for u in 4e18b22a 6818a92b c21f0f30 7235b0b4; do echo "== $u"; awk -F, -v u=$u 'NR>1 && tolower($1) ~ "^"u {print $2" agg_total_events="$7" events_truncated="$9}' export.csv | sort -u; done; sha256sum ../parse_detect | cut -c1-16
[mimr_benign_N250000] story that = 6 | stub covered_by = 4 | GT96 = 96 (GT49 sau cutoff = 49)
-- TOAN BO story --
  story: 6 | story chua GT (TP): 4 | story precision = 0.6667
  member process phan biet: 140
  recall GT96  : 83/96 = 0.8646  Wilson [0.782,0.919]
  recall GT49  : 39/49 = 0.7959  Wilson [0.664,0.885]
  PHU SONG: member = 140 / 169650 process = 0.1% | GT96 ky vong neu phu ngau nhien cung ty le = 0.1 | lift = 1047.69x (tran = 1211.79x) | P(ngau nhien >= 83) = 1.21e-254
  PHU SONG sau cutoff: member sau cutoff = 72 / 63225 = 0.1% | GT49 ky vong = 0.1 | P(ngau nhien >= 39) = 3.4e-111
-- chi story co seed start >= cutoff --
  story: 2 | story chua GT (TP): 1 | story precision = 0.5000
  member process phan biet: 56
  recall GT96  : 39/96 = 0.4062  Wilson [0.313,0.506]
  recall GT49  : 39/49 = 0.7959  Wilson [0.664,0.885]
  PHU SONG: member = 56 / 169650 process = 0.0% | GT96 ky vong neu phu ngau nhien cung ty le = 0.0 | lift = 1230.72x (tran = 3029.46x) | P(ngau nhien >= 39) = 2.67e-117
  PHU SONG sau cutoff: member sau cutoff = 54 / 63225 = 0.1% | GT49 ky vong = 0.0 | P(ngau nhien >= 39) = 8.53e-119
 seed_node           start(UTC) pre/post members   GT  TP/FP
      2701  2018-04-10 18:31:22      pre      13   12  TP   4E18B22A
      2732  2018-04-10 18:35:30      pre      58   24  TP   6818A92B
      4615  2018-04-10 18:54:16      pre       9    8  TP   C21F0F30
     66518  2018-04-11 18:47:28      pre       6    0  FP   3316F77F
       3316F77F-0100-0000-0000-000000000020,66518,D,7060,19,1,1,0,0,0,0,0,0,0,0,0,0,0,0,0,1,1,0,0,0,"du -sh /bin /boot /cdrom /cpu0 /data /debug /dev /etc /home /initrd.img /lib /lib64 /lost+found /media /mnt /opt /proc /root /run /sbin
    106562  2018-04-12 16:41:25     post      15    0  FP   F534ECB3
       F534ECB3-0200-0000-0000-000000000020,106562,D,1193,541,1,1,0,0,0,0,0,0,0,0,0,0,0,0,0,1,1,0,0,0,"/bin/bash -c /usr/bin/firefox"
    106687  2018-04-12 16:44:41     post      42   39  TP   7235B0B4
        Elapsed (wall clock) time (h:mm:ss or m:ss): 2:11:49
        Maximum resident set size (kbytes): 4535288
        Exit status: 0
== 4e18b22a
2701 agg_total_events=111045 events_truncated=0
2701 agg_total_events=1 events_truncated=0
2701 agg_total_events=78557 events_truncated=0
== 6818a92b
2732 agg_total_events=1 events_truncated=0
2732 agg_total_events=46018 events_truncated=0
2732 agg_total_events=5 events_truncated=0
2732 agg_total_events=95283 events_truncated=0
== c21f0f30
4615 agg_total_events=111371 events_truncated=0
4615 agg_total_events=11571 events_truncated=0
4615 agg_total_events=1 events_truncated=0
4615 agg_total_events=70916 events_truncated=0
== 7235b0b4
106687 agg_total_events=1 events_truncated=0
106687 agg_total_events=2377 events_truncated=0
106687 agg_total_events=6305 events_truncated=0
81438c442e5a783a
```

**(10) Điểm L4 của hai UUID FP, baseline so với benign (`grep` trên `export_l4_s1.csv.infer_status` và `rep_mimr_benign_N250000/export.csv.infer_status`; trích các dòng, bản dán đầy đủ có 192 dòng, cột: uuid, node_id, bảng, priority_key, priority_count, member_count, agg_total_events, status, score, pred, ..., verdict):**

```
--- baseline (rep_l4_s1)
3316F77F-0100-0000-0000-000000000020,66518,D,7060,19,1,1,SCORED,0.000000,0,1,0
F534ECB3-0200-0000-0000-000000000020,106562,C,106562,1,1,531,SCORED,0.001169,0,1,0
F534ECB3-0200-0000-0000-000000000020,106562,D,1193,541,1,1,SCORED,0.000002,0,1,0
F534ECB3-0200-0000-0000-000000000020,106562,D,1194,894,1,10,SCORED,0.000002,0,1,0
--- benign N=250000 (trích)
3316F77F-0100-0000-0000-000000000020,66518,D,7060,19,1,18797,SCORED,0.999702,1,2,1
3316F77F-0100-0000-0000-000000000020,66518,D,7060,19,1,1,SCORED,0.000000,0,1,0
F534ECB3-0200-0000-0000-000000000020,106562,C,106562,1,1,15726,SCORED,0.001169,0,1,0
F534ECB3-0200-0000-0000-000000000020,106562,C,106562,1,1,17160,SCORED,1.000000,1,2,1
F534ECB3-0200-0000-0000-000000000020,106562,C,106562,1,1,118527,SCORED,1.000000,1,8,1
F534ECB3-0200-0000-0000-000000000020,106562,D,1194,956,1,17160,SCORED,1.000000,1,2,1
F534ECB3-0200-0000-0000-000000000020,106562,D,1194,1407,1,118527,SCORED,1.000000,1,8,1
```


### Phụ lục: chi phí state theo miền sự kiện và theo syscall [đo, 2026-11-20, `domain_cost.py`]

Đọc thẳng `processes.bin` của bộ 6r (không chạy lại pipeline): mỗi sự kiện 64 byte, byte 0 là tag miền, byte 2–3 là `syscall_id`. Layout block tự dò: meta FLUSH 12 B, SUMMARY counter 256 B + node_id 4 B. Hai file: `mem_wl/processes.bin` (`parse_detect_wl`, có whitelist, 685,9 MB) và `rep_abl/processes.bin` (`parse_detect_abl`, bỏ whitelist, 1.138,5 MB).

| Miền | Có whitelist: bản ghi / MB / % file | Bỏ whitelist: bản ghi / MB / % file |
|---|---|---|
| file | 4.045.932 / 258,9 / 37,8% | 4.515.713 / 289,0 / 25,4% |
| network | 2.919.214 / 186,8 / 27,2% | 3.050.129 / 195,2 / 17,1% |
| transfer | 1.911.636 / 122,3 / 17,8% | 7.074.686 / 452,8 / 39,8% |
| memory | 381.115 / 24,4 / 3,6% | 1.682.785 / 107,7 / 9,5% |
| process | 158.894 / 10,2 / 1,5% | 158.894 / 10,2 / 0,9% |
| exec | 70.890 / 4,5 / 0,7% | 70.890 / 4,5 / 0,4% |
| privesc | 0 | 0 |
| Không phải event (HEADER + meta FLUSH + SUMMARY) | 78,6 MB (11,5%) | 79,1 MB (7,0%) |

Có whitelist: 170.444 header (process), 9.487.681 bản ghi event, trung bình 55,7 bản ghi và 4,0 KB state mỗi process. RLE gộp network từ 13.225.082 sự kiện gốc xuống 2.919.214 bản ghi (4,5×; `recvfrom` 10.162.920 gốc xuống 667.262 bản ghi), file 1,18×. Chênh bản bỏ whitelist so với có whitelist (+452,6 MB): transfer +330,5 (`read` 297.891 lên 4.808.697), memory +83,3, file +30,1, network +8,4. Lưu ý memory không so được từng dòng: bản whitelist giữ `mmap` (381.115), bản ablation bỏ `mmap`/`mprotect` theo thiết kế nhưng ghi `munmap` (1.675.331) và `shmat` (7.454).

Theo syscall (có whitelist): file `open` 78,3% / `openat` 15,9% / `unlink` 5,7%; network `recvmsg` 26,8% / `sendmsg` 26,4% / `recvfrom` 22,9% / `sendto` 12,1% / `connect` 11,9%; transfer `write` 84,4% / `read` 15,6%; memory 100% `mmap`; process 100% `clone`; exec 100% `execve`. privesc = 0 trên THEIA E3.

### 6.8 Raw log: kích thước dữ liệu, state, RSS, binary, chi phí theo miền (2026-11-20)

Các lệnh chạy trong `~/theia_e3` (WSL), output dán nguyên văn. Ghi chú: `parse_detect` `81438c44` có whitelist (xem nm/objdump bên dưới); bản bỏ whitelist là `parse_detect_abl` `6b0b56d9`; `measure_nowl.sh` thực chất chạy lại bản whitelist lần hai.

**(1) Kích thước input bộ 6r (không tính `.zip`) và JSON** — `find ... -printf '%s'` cộng 13 file `ta1-theia-e3-official-6r.bin*`:
```
tong 6r: 9894818787 byte = 9.89 GB
-rwxrwxrwx 1 enovo enovo 1094765144 Thg 8 10 11:22 ta1-theia-e3-official-6r.bin.zip
```
9.894.818.787 + 1.094.765.144 = 10.989.583.931 byte (= "10,99 GB" trong bài, cộng nhầm file .zip). JSON, `tar -tzvf ta1-theia-e3-official-6r.json.tar.gz | awk '{s+=$3; n++}'`:
```
13 file, 47995994407 byte = 48.00 GB (thap phan), 44.70 GiB
```

**(2) Kích thước state bộ 6r** — `ls -l` các file trong `~/theia_e3` (mốc 2026-10-07, binary chính) và `wl_rand_run_1` (`parse_detect_wl`, 2026-11-15):
```
15.5 MB  2026-10-07 12:16  mem.spill
52.1 MB  2026-10-07 12:16  net.spill
62.8 MB  2026-10-07 12:16  path.spill
685.9 MB  2026-10-07 12:16  processes.bin
31.8 MB  2026-10-07 12:16  resource_pool.bin
0.9 MB  2026-10-07 12:16  subj.spill
== wl_rand_run_1
685.9 MB  2026-11-15 11:59  wl_rand_run_1/processes.bin
31.8 MB  2026-11-15 11:59  wl_rand_run_1/resource_pool.bin
== wl_rand_run_2   (lần chạy dở, bỏ qua)
56.5 MB  2026-11-15 12:12  wl_rand_run_2/processes.bin
```
Bản bỏ whitelist (`parse_detect_abl`, `rep_abl`, 2026-11-15 14:09) và `rand_run_1..5` (2026-11-14, binary cũ đã bị ghi đè):
```
15.5 MB  2026-11-15 14:09  rep_abl/mem.spill
52.1 MB  2026-11-15 14:09  rep_abl/net.spill
62.8 MB  2026-11-15 14:09  rep_abl/path.spill
1138.5 MB  2026-11-15 14:09  rep_abl/processes.bin
31.8 MB  2026-11-15 14:09  rep_abl/resource_pool.bin
0.9 MB  2026-11-15 14:09  rep_abl/subj.spill
(rand_run_1..5: processes.bin 1138.5 MB, resource_pool.bin 31.8 MB, spill 15.5/52.1/62.8/0.9 MB, cả 5 giống nhau)
```
Bộ `1r` (`ta1-theia-e3-official-1r.bin`, chạy 2026-10-09 11:05–11:11, EXPORT-ONLY không model, `parse_detect` trong thư mục đó sửa lại 2026-11-15 nên đã bị ghi đè): `processes.bin` 643,8 MiB = 675,1 MB, `resource_pool.bin` 4,8 MiB = 5,0 MB, tổng input 6,72 GiB, `events written: 15486093`. Đây là nguồn của "675 MB + 5 MB" trong bài.

**(3) Hash và nội dung binary** — `nm` + `objdump` hàm `should_log_*`:
```
=== parse_detect_wl 0dcf0620
000000000001a530 T should_log_memory_event
000000000001bb40 T should_log_transfer_event
  should_log_memory_event: 72 lenh
  should_log_transfer_event: 57 lenh
=== parse_detect 81438c44
000000000001a530 T should_log_memory_event
000000000001bb40 T should_log_transfer_event
  should_log_memory_event: 72 lenh
  should_log_transfer_event: 57 lenh
=== parse_detect_abl 6b0b56d9
000000000001a540 T should_log_memory_event
000000000001baa0 T should_log_transfer_event
  should_log_memory_event: 12 lenh
  should_log_transfer_event: 5 lenh
```

**(4) Đo RSS, thời gian, state trên bộ 6r (chạy song song, mỗi bản một lần)** — `mem_wl/summary.txt` và `mem_nowl/summary.txt` (+ `time.txt`):
```
=========== wl
label=wl bin=/home/enovo/theia_e3/parse_detect_wl sha=0dcf0620 start=2026-11-20T11:05:34+00:00
end=2026-11-20T12:58:57+00:00 exit=0
input_bytes=9894818787
elapsed=1:53:21
max_rss_kb(time -v)=1550232  = 1.59 GB (thap phan)
vmhwm_peak_kb(sampler)=1550232
processes.bin=685.9 MB
resource_pool.bin=31.8 MB
spill=131.3 MB
ratio processes=14.43x | +pool=13.79x | +spill=11.66x
[13/13] .../ta1-theia-e3-official-6r.bin.12 xong - total=63852494 subj=169644 written=20526652 no_state=0 exits=0 events written: 20526652
        Elapsed (wall clock) time (h:mm:ss or m:ss): 1:53:21
        Maximum resident set size (kbytes): 1550232
        Exit status: 0
=========== nowl   (thực chất chạy parse_detect 81438c44 = bản whitelist)
label=nowl bin=/home/enovo/theia_e3/parse_detect sha=81438c44 start=2026-11-20T11:05:34+00:00
end=2026-11-20T12:58:44+00:00 exit=0
elapsed=1:53:08
max_rss_kb(time -v)=1550300  = 1.59 GB (thap phan)
vmhwm_peak_kb(sampler)=1550300
processes.bin=685.9 MB
resource_pool.bin=31.8 MB
spill=131.3 MB
ratio processes=14.43x | +pool=13.79x | +spill=11.66x
Exit status: 0
```
`samples.csv` dòng cuối: `6792,685596,1550232,685861150,31782546,131287287` (wl), `6778,685616,1550300,685861150,31782546,131287287` (nowl). Hai bản chạy cùng lúc nên thời gian bị kéo dài. Peak RSS 1,59 GB (1,48 GiB) khi bật `ALL_OCCURRENCES` + export + `CG_VERIFY`; cấu hình nhẹ hơn cho khoảng thấp hơn (bài ghi 1,2 GB; thống nhất ghi khoảng 1,2–1,6 GB).

**(5) Kích thước tĩnh** — `sizeof` (source `theia_trace_e3`):
```
process_state_t                1184 B
event_union_t                    64 B
struct event_file                64 B
struct event_process             64 B
struct event_exec                64 B
struct event_memory              64 B
struct event_network             64 B
struct event_privesc             64 B
struct event_transfer            64 B
EVENT_HISTORY_SIZE=256  PROCESS_EVIDENCE_CAP=128  SYSCALL_INDEX_MAX=128
```
`event_history` (256 × 64 = 16.384 B) và `evidence` (128 × 64 = 8.192 B) cấp phát lười (`calloc` trong `process_state_ensure_history/evidence`), nên chi phí một process nằm trong khoảng 1.184 B đến khoảng 25,8 KB.

**(6) Chi phí theo miền và theo syscall** — `python3 domain_cost.py mem_wl rep_abl` (`domain_cost.txt`), nguyên văn (tên syscall: 0 read, 1 write, 2 open, 9 mmap, 11 munmap, 30 shmat, 42 connect, 44 sendto, 45 recvfrom, 46 sendmsg, 47 recvmsg, 56 clone, 59 execve, 87 unlink, 90 chmod, 92 chown, 93 fchown, 257 openat):
```
====================================================================================
mem_wl/processes.bin
====================================================================================
   [layout tu do] meta FLUSH=12 B (sau byte type), counter SUMMARY=256 B, node_id SUMMARY=4 B (doc duoc 300,000 block dau, 105410 SUMMARY)
file=685.9 MB | header=170,444 | segment FLUSH=149,982 | ban ghi event=9,487,681 (607.2 MB = 88.5% file)
phan KHONG phai event (HEADER + meta FLUSH + SUMMARY) = 78.6 MB (11.5% file)
mien            ban ghi       MB  % file  % event   su kien goc    RLE
file          4,045,932    258.9   37.8%    42.6%     4,779,035  1.18x
network       2,919,214    186.8   27.2%    30.8%    13,225,082  4.53x
transfer      1,911,636    122.3   17.8%    20.1%     1,911,636  1.00x
memory          381,115     24.4    3.6%     4.0%       381,115  1.00x
process         158,894     10.2    1.5%     1.7%       158,894  1.00x
exec             70,890      4.5    0.7%     0.7%        70,890  1.00x
privesc               0      0.0    0.0%     0.0%             0  0.00x

  [file] 4,045,932 ban ghi = 258.9 MB  -> theo syscall:
     sc_2              (id   2)    3,167,434    202.7 MB   78.3% mien     3,683,071 goc
     sc_257            (id 257)      642,724     41.1 MB   15.9% mien       644,575 goc
     sc_87             (id  87)      230,880     14.8 MB    5.7% mien       231,370 goc
     sc_90             (id  90)        3,206      0.2 MB    0.1% mien       218,331 goc
     sc_92             (id  92)          844      0.1 MB    0.0% mien           844 goc
     sc_93             (id  93)          844      0.1 MB    0.0% mien           844 goc

  [network] 2,919,214 ban ghi = 186.8 MB  -> theo syscall:
     sc_47             (id  47)      781,449     50.0 MB   26.8% mien     1,224,165 goc
     sc_46             (id  46)      771,098     49.4 MB   26.4% mien       975,350 goc
     sc_45             (id  45)      667,262     42.7 MB   22.9% mien    10,162,920 goc
     sc_44             (id  44)      353,085     22.6 MB   12.1% mien       495,465 goc
     sc_42             (id  42)      346,320     22.2 MB   11.9% mien       367,182 goc

  [transfer] 1,911,636 ban ghi = 122.3 MB  -> theo syscall:
     sc_1              (id   1)    1,613,745    103.3 MB   84.4% mien     1,613,745 goc
     sc_0              (id   0)      297,891     19.1 MB   15.6% mien       297,891 goc

  [memory] 381,115 ban ghi = 24.4 MB  -> theo syscall:
     sc_9              (id   9)      381,115     24.4 MB  100.0% mien       381,115 goc

  [process] 158,894 ban ghi = 10.2 MB  -> theo syscall:
     sc_56             (id  56)      158,894     10.2 MB  100.0% mien       158,894 goc

  [exec] 70,890 ban ghi = 4.5 MB  -> theo syscall:
     sc_59             (id  59)       70,890      4.5 MB  100.0% mien        70,890 goc

-- spill:
   mem.spill          15.5 MB
   net.spill          52.1 MB
   path.spill         62.8 MB
   subj.spill          0.9 MB
-- resource_pool.bin (31.8 MB), chia xap xi theo loai chuoi (pool dung chung moi mien):
   path (/...)                399,143 chuoi     30.7 MB
   cmdline (co dau cach)        1,552 chuoi      1.1 MB
   ip                             198 chuoi      0.0 MB
   khac                            37 chuoi      0.0 MB
====================================================================================
rep_abl/processes.bin
====================================================================================
   [layout tu do] meta FLUSH=12 B (sau byte type), counter SUMMARY=256 B, node_id SUMMARY=4 B (doc duoc 300,000 block dau, 99392 SUMMARY)
file=1138.5 MB | header=170,763 | segment FLUSH=177,320 | ban ghi event=16,553,097 (1059.4 MB = 93.0% file)
phan KHONG phai event (HEADER + meta FLUSH + SUMMARY) = 79.1 MB (7.0% file)
mien            ban ghi       MB  % file  % event   su kien goc    RLE
transfer      7,074,686    452.8   39.8%    42.7%     7,074,686  1.00x
file          4,515,713    289.0   25.4%    27.3%     4,779,035  1.06x
network       3,050,129    195.2   17.1%    18.4%    13,225,082  4.34x
memory        1,682,785    107.7    9.5%    10.2%     1,682,785  1.00x
process         158,894     10.2    0.9%     1.0%       158,894  1.00x
exec             70,890      4.5    0.4%     0.4%        70,890  1.00x
privesc               0      0.0    0.0%     0.0%             0  0.00x

  [transfer] 7,074,686 ban ghi = 452.8 MB  -> theo syscall:
     sc_0              (id   0)    4,808,697    307.8 MB   68.0% mien     4,808,697 goc
     sc_1              (id   1)    2,265,989    145.0 MB   32.0% mien     2,265,989 goc

  [file] 4,515,713 ban ghi = 289.0 MB  -> theo syscall:
     sc_2              (id   2)    3,637,202    232.8 MB   80.5% mien     3,683,071 goc
     sc_257            (id 257)      642,736     41.1 MB   14.2% mien       644,575 goc
     sc_87             (id  87)      230,880     14.8 MB    5.1% mien       231,370 goc
     sc_90             (id  90)        3,207      0.2 MB    0.1% mien       218,331 goc
     sc_92             (id  92)          844      0.1 MB    0.0% mien           844 goc
     sc_93             (id  93)          844      0.1 MB    0.0% mien           844 goc

  [network] 3,050,129 ban ghi = 195.2 MB  -> theo syscall:
     sc_47             (id  47)      782,208     50.1 MB   25.6% mien     1,224,165 goc
     sc_46             (id  46)      771,141     49.4 MB   25.3% mien     975,350 goc
     sc_45             (id  45)      677,406     43.4 MB   22.2% mien    10,162,920 goc
     sc_44             (id  44)      473,054     30.3 MB   15.5% mien       495,465 goc
     sc_42             (id  42)      346,320     22.2 MB   11.4% mien       367,182 goc

  [memory] 1,682,785 ban ghi = 107.7 MB  -> theo syscall:
     sc_11             (id  11)    1,675,331    107.2 MB   99.6% mien     1,675,331 goc
     sc_30             (id  30)        7,454      0.5 MB    0.4% mien         7,454 goc

  [process] 158,894 ban ghi = 10.2 MB  -> theo syscall:
     sc_56             (id  56)      158,894     10.2 MB  100.0% mien       158,894 goc

  [exec] 70,890 ban ghi = 4.5 MB  -> theo syscall:
     sc_59             (id  59)       70,890      4.5 MB  100.0% mien        70,890 goc

-- spill:
   mem.spill          15.5 MB
   net.spill          52.1 MB
   path.spill         62.8 MB
   subj.spill          0.9 MB
-- resource_pool.bin (31.8 MB), chia xap xi theo loai chuoi (pool dung chung moi mien):
   path (/...)                399,143 chuoi     30.7 MB
   cmdline (co dau cach)        1,552 chuoi      1.1 MB
   ip                             198 chuoi      0.0 MB
   khac                            37 chuoi      0.0 MB
```
