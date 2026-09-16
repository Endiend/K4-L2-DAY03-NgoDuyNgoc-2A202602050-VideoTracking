# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: `Ngô Duy Ngọc`
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT / khác: `CVAT` |
| Thời gian gán `clip_02` (warm-up) | `40` phút |
| Thời gian gán `clip_01` | `180` phút |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `≈ 80` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. `vehicle 7` xuất hiện phía sau đầu xe bus(vehicle 5) khá khó để nhận dạng và track theo khi 2 xe gần như có cùng tốc độ. 
Cách xử lý: phóng to hình ảnh, phán đoán vị trí xe và locked vehicle 5 để không track nhầm
2. `vehicle 8` xuất hiện phía sau đuôi xe bus(vehicle 5) độ khó track và cách xử lý tương tự vehicle 7
3. `vehicle 5` dễ theo dõi nhưng phần bánh xe(rất nhỏ) bị hàng rào che mất nên e không biết nên đê occluded không
Cách xử lý: xem cách áp dụng occluded: bất cứ khi nào có một vật thể khác che khuất một phần vật thể chính (dù chỉ là phần nhỏ)=> vẫn để occluded dù chỉ phần nhỏ bánh xe bị che                

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: `...`
- Lượt 2: `...`
- Lượt 3: `...`

Kiểm chéo với: `...`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `...`. Số lỗi bạn ấy tìm được trong bản của bạn: `...`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`...`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `28b2ef2890dc76acbbd888b93a2b723eb99133b76a16a23dccfbf7ebb4a27aff` |
| Thời điểm khóa | `17:30` |
| Số row / frame / track trước khi mở reference | `639 bbox / 190 frame / 8 track` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
|  |  | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.782 | 0.761 | 0.806 | 0.872 | 0.942 | 0.878 | 0.859 | 68 | 2 | 0 |
| Sau rework |0.782	|0.761|	0.806	|0.872|	0.942|	0.878	|0.859	|68	|2|	0|

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): có 
Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox treo (dư trước khi xe vào khung) | 55–78 | gold ID 5 |  bấm outside đúng frame xe thật sự xuất hiện |
| Đánh dấu track |55-79 | gold ID 5 | tự động track của gold phát hiện xe muộn tới frame 79 mới phát hiện dẫn tới 2 bản pre-gold và sau rework giống nhau |
| Bbox trôi (IoU chỉ 0.57–0.60 so với gold) | 81, 93, 96, 97 | gold ID 5 |  thêm keyframe quanh các frame này |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13` |
| weights / hai tracker | `yolo26n.pt` |
| conf / IoU / imgsz / classes | `0.25 / 0.7 / 960 / [2,5,7]` |
| device | `0` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.782 | 0.761 | 0.806 | 0.872 | 0.942 | 0.878 | 0.859 | 68 | 2 | 0|
| ByteTrack control vs gold | 0.709  | 0.649 |  0.776 |  0.846  | 0.875 |  0.749 |  0.823  |    88   |   54  |  2 |
| BoT-SORT + ReID vs gold | 0.763 |  0.711 |  0.820  | 0.872  | 0.900 |  0.792  | 0.860  |   91   |  26   |    2 |
| ReID vs bạn | 0.763  | 0.711 |  0.820 |  0.872 |  0.900  | 0.792  | 0.860     | 91  |    26   |    2 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

`IDF1 (0.942) cao hơn MOTA (0.878). Ở đây IDSW = 0 nên khoảng cách này chủ yếu đến từ 68 bbox FP (chính là các bbox "treo" ở đoạn xe vào/ra khung được liệt kê ở mục 3): MOTA = 1 − (FP+FN+IDSW)/GT = 1 − (68+2+0)/573 ≈ 0.878, khớp với số notebook in ra. Lý do tổng quát MOTA không phạt nặng lỗi ID: công thức MOTA đếm mỗi lần ID switch là một lỗi duy nhất tại thời điểm nó xảy ra, bất kể sau đó ID sai đó tồn tại bao nhiêu frame tiếp theo; trong khi IDF1 đo theo khớp danh tính (identity matching) trên toàn bộ track, nên một lần đổi ID kéo theo rất nhiều frame "sai danh tính" phía sau sẽ bị IDF1 phạt nặng hơn nhiều so với MOTA.`

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

`IDF1 tăng từ 0.875 → 0.900, AssA tăng từ 0.776 → 0.820, nhưng IDSW giữ nguyên = 2 ở cả hai (chỉ đổi track bị ảnh hưởng: ByteTrack lệch ở track gold 4/5, frame 59 & 94; ReID lệch ở track gold 5/6, frame 87 & 113) và số lần TÁCH TRACK cũng giữ nguyên = 3 ở cả hai run. Vậy phần cải thiện IDF1/AssA không đến từ việc giảm số lần đổi ID, mà chủ yếu đến từ việc bbox khít hơn: số lần "BBOX LỆCH" giảm mạnh từ 30 xuống còn 9, và số track bị mất dấu một đoạn ("MODEL BẮT THIẾU ĐOẠN") giảm từ 3 track xuống còn 1 track. Vì ByteTrack và BoT-SORT+ReID là hai thuật toán tracker khác nhau hoàn toàn (không chỉ khác ở có/không có ReID), số liệu này chỉ cho thấy treatment tổng thể tốt hơn, không tách được riêng phần đóng góp của đặc trưng ReID khỏi các khác biệt khác trong thuật toán association của BoT-SORT.`

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

`DetA tăng từ 0.649 → 0.711, FN giảm mạnh từ 54 → 26 (ít bỏ sót vật thể hơn hẳn), nhưng FP lại tăng nhẹ từ 88 → 91. Nguồn gốc phần FP tăng thêm này một phần đến từ chính phát hiện ở câu 4 bên dưới: treatment liên tục "nhìn nhầm" một ki-ốt/cửa hàng sáng đèn ở nền thành vật thể theo dõi được (track kéo dài 43 frame) — đây là lỗi phía detector/appearance (bị đánh lừa bởi ánh sáng), không phải lỗi association. Trong khi đó AssA vẫn còn cách khá xa LocA (0.820 so với 0.872) và vẫn còn 2 IDSW + 3 lần tách track — cho thấy lỗi còn lại là cả hai phía: vẫn còn lỗi detector rõ ràng (FP do nhận nhầm nền) lẫn lỗi association chưa giải quyết hết (track vẫn bị chia sau khi bị che).`

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

`Ở các frame 104–111, phần "BBOX THỪA của model" liệt kê ID 7 (frame 16–116, 43 frame) không khớp với bất kỳ track nào trong nhãn của bạn. Xem trực tiếp ảnh frame 104/105/106/111 do notebook vẽ ra: mô hình BoT-SORT+ReID liên tục vẽ box đỏ quanh một ki-ốt/cửa hàng sáng đèn ở dải phân cách giữa đường (vật thể tĩnh, không phải xe) suốt nhiều frame liền — trong khi bạn (người gán nhãn) đã đúng khi không gán nhãn cho nó. Nhiều khả năng ánh sáng rực của ki-ốt bị nhầm với đặc trưng ngoại hình (appearance feature) của một phương tiện, khiến ReID "khóa" nhầm và duy trì track giả suốt 43 frame.`

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

`Ở mục 3, khi so nhãn của bạn với gold, track 5 của bạn bị lệch còn IoU chỉ 0.57–0.60 tại các frame 81, 93, 96, 97. Đáng chú ý, trong kết quả "BBOX LỆCH" của ReID vs gold, track gold 5 không xuất hiện trong danh sách lệch (chỉ còn track 1, 6, 7 bị lệch) — tức là ở đúng đoạn frame 81–97 này, box của model bám sát gold tốt hơn box bạn tự vẽ. Đây là bằng chứng đáng để bạn quay lại CVAT, mở đúng track 5 ở các frame 81/93/96/97 và kiểm tra lại xem có nên thêm keyframe để bbox khít hơn hay không.`

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

`có thể thêm quy tắc rõ ràng hơn về việc bấm "outside" ngay khi xe rời khung để tránh bbox treo, và quy định thêm keyframe dày hơn quanh các đoạn bị che/khuất hình để giảm bbox trôi`

## 7. Tệp đã nộp

- [ ] `annotations/clip_01/gt.txt`
- [ ] `annotations/clip_02/gt.txt`
- [ ] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [ ] `GUIDELINE_MINI.md` đã điền
- [ ] `outputs/eval_vs_gold.json`
- [ ] `outputs/model_bytetrack_clip_01.txt`
- [ ] `outputs/model_reid_clip_01.txt`
- [ ] `outputs/model_run_config.json`
- [ ] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md`
- [ ] `reports/REPORT.md` (file này)
