# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Ngô Duy Ngọc`
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm (nếu có): `...`

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che dưới  (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | `Xe vẫn là cùng một đối tượng` |
| Xe bị che lâu hơn ngưỡng trên | `Kết thúc track cũ; khi xuất hiện lại thì tạo ID mới` | `Khoảng mất dấu quá lâu nên không tiếp tục ID cũ` |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | `Xe đã rời khỏi video, khi quay lại được xem là một track mới` |
| Hai xe cắt nhau / chồng lên nhau | `Giữ nguyên ID của từng xe` | `Không đổi ID chỉ vì hai xe che/chồng lên nhau` |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: `...` |
| Xe đang đỗ, không di chuyển | `Giữ bbox ổn định, không thay đổi tùy ý` |
| Keyframe đặt dày ở đâu | `Đặt keyframe dày tại các đoạn xe đổi hướng, bị che, hoặc bbox có nguy cơ trôi` |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01, frame 55–78, track ID 5`
- Tình huống: `xác định xe xuất hiện và bắt đầu track từ frame 55, nhưng khi chấm với gold, công cụ báo đây là "bbox treo" — track tham chiếu (gold) chỉ thật sự bắt đầu từ khoảng frame 79. Nói cách khác, hai bên bất đồng về thời điểm được tính là "xe đã xuất hiện đủ rõ để bắt đầu track"`
- Quyết định: `quyết định giữ nguyên theo phán đoán của mình`
- Lý do: `gold do máy tracking sai`

### Ca 2
- Clip / frame / ID: `clip_01, frame 81, 93, 96, 97, track ID 5`
- Tình huống: `Cùng track 5 nói trên, ở các frame này bbox chỉ đạt IoU 0.57–0.60 so với gold (dưới ngưỡng khớp 0.5 rất sát)`
- Quyết định: `quyết định giữ nguyên theo phán đoán của mình`
- Lý do: `gold do máy tracking sai`

### Ca 3
- Clip / frame / ID: `clip_01, frame 1–15 và frame 148–162, track ID 3`
- Tình huống: `Công cụ kiểm tra định dạng (check_mot_labels.py) cảnh báo track 3 gần như đứng yên hoàn toàn trong hai đoạn này`
- Quyết định: `quyết định giữ nguyên theo phán đoán của mình`
- Lý do: `xe đỗ ở yên 1 vị trí không thay đổi, cam cũng là cố định nên vùng track không đổi chỉ khi có người hoặc vật che thì thêm occluded`

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- `Mục 3 "Xe vừa xuất hiện, còn rất nhỏ/rất mờ" cần một ngưỡng cụ thể (ví dụ kích thước pixel tối thiểu hoặc mốc thị giác rõ ràng), vì thiếu ngưỡng này đã gây ra 7 lỗi "bbox treo" thật trong bài (track ID 4, 5, 6, 7, 8 — mỗi track lệch 3–24 frame ở điểm bắt đầu/kết thúc so với gold).`
- `Mục 3 "Keyframe đặt dày ở đâu" cần nói rõ nên thêm keyframe khi xe đổi hướng/tốc độ đột ngột, vì thiếu quy tắc này đã gây ra 4 lỗi "bbox trôi" thật (track ID 5, frame 81/93/96/97, IoU chỉ còn 0.57–0.60).`
