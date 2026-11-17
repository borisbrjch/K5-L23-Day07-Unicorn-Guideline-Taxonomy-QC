# Tóm tắt IAA (taxonomy v1.0)

- Ảnh được so sánh (có >=2 người gán): **40**; số người gán: 2
- Lượt gán đủ **80/80**.
- IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa.
- Mẫu số: 283 cặp box ghép (tổng mọi cặp người gán, ảnh chung).

| Cặp | Ảnh | Cặp box ghép | IoU TB | Cùng lớp | Cùng occluded_level | Cùng truncated | Chưa ghép (A/B) | Bỏ qua do ignore | Đạt IAA |
|---|---|---|---|---|---|---|---|---|---|
| **TỔNG** | 40 | 283 | 0.836 | 62.2% | 65.0% | 95.8% | 67/133 | 0 | CHƯA ĐẠT |
| 143|180 | 40 | 283 | 0.836 | 62.2% | 65.0% | 95.8% | 67/133 | 0 | CHƯA ĐẠT |

Ngưỡng IAA: IoU TB >= 0.85, đồng thuận lớp >= 95%.

Số ca bất thường: 543 ({'bo_sot_thua': 200, 'iou_thap': 117, 'sai_thuoc_tinh': 105, 'sai_lop': 107, 'co_can_review': 9, 'bat_dong_ignore': 5}), trên 40 ảnh cần calibration.
