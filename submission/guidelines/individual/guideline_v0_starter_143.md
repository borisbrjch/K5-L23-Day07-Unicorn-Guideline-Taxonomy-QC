# Guideline gán nhãn — v0 (bản khởi đầu)

- Phiên bản: v0
- Loại nhãn: bounding box (hộp chữ nhật) trên ảnh đường phố
- Công cụ: CVAT, xuất COCO 1.0

---

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình phát hiện **người và phương tiện** trên ảnh đường phố
(ví dụ cho camera giao thông hoặc xe tự lái).

## 2. Lớp và thuộc tính

### Lớp (mỗi vật thuộc đúng một lớp)

| Lớp | Định nghĩa | Positive Example | Negative Example
|---|---|---|---|
| `car` | Ô tô con. | Bao gồm xe 4 chỗ, suv, hatchback. | Xe chở hàng, xe bus 16 chỗ |
| `truck` | Xe tải. |  Xe chở hàng, hoặc vừa chở hàng vừa chở người | Xe con, xe bus, xe ba bánh |
| `van` | Xe thùng kín cỡ nhỏ. | Xe thùng kín cỡ 6 - 9 chỗ | Xe bus 16 chỗ, xe bán tải, xe con |
| `motorcycle` | Xe gắn máy, hai bánh | Xe máy, mô tô, side car | Xe đạp, xe 3 bánh |
| `bycicle` | Phương tiện thô sơ hai bánh | Xe đạp. | Xe moto gắn máy |
| `hàng rong` | Gánh hàng rong hoặc xe đạp hàng rong | Xe đạp chở sạp hàng hoặc người gánh sạp hàng | Người đi bộ |
| `dynamic person` | Người đang đi. | Người đi bộ đang tham gia giao thông (xuất hiện ở vỉa hè, lòng đường, cross walk). | Người đang ngồi hoặc đứng yên |
| `static person` | Người không tham gia giao thông. | Người đang ngồi, không xuất hiện ở phần đường dành cho xe cộ | Gánh hàng rong, người đang đi |
| `moto rider` | Người điều khiển phương tiện xe gắn máy. | Người điều kiển xe máy | Người đi bộ |
| `bycicle rider` | Người điều khiển xe đạp, xe thô sơ. | Người điều khiển xe đạp | Người đi bộ |
| `ignore_region` | Vùng không cần gán nhãn. | Xe đồ chơi, mô hình đồ chơi, biển quảng cáo,... | Người, phương tiện thật |

### Thuộc tính

| Thuộc tính | Giá trị | Ý nghĩa |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | Mức bị che: 0 là thấy rõ, 1 là bị che ít (từ 1 - 30%), 2 là bị che nhiều (từ 40 - 70%), 3 là không rõ (>=80%). |
| `truncated` | `no`, `yes` | Vật bị cắt ở mép ảnh. |
| `can_review` | tick / không | Tick khi bạn không chắc, để người khác xem lại. |

## 3. Quy tắc vẽ box

1. Gán box cho tất cả các class đã được định nghĩa trong guideline.
2. Không gắn nhãn cho những vật false positive, ví dụ như xe mô hình, xe đồ chơi, người mô hình, hình ảnh quảng cáo. Chỉ gắn nhãn cho vật thể là người, phương tiện thật.
3. Box ôm sát vật.
4. Chọn đúng lớp và thuộc tính.
5. Với object overlapping thì gắn cờ occlusion / truncation theo định nghĩa.
6. Vật thể nhỏ có chiều cao < 20px không cần gắn nhãn.


## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| img_003 | Có người ngồi trên xe máy nhưng không di chuyển, vừa đỗ trên vỉa hè, vừa đỗ dưới lòng đường. | 1 | Gắn nhãn 'static person' | Vật thể này đang không di chuyển và cũng sẽ không bất ngờ di chuyển|
|
| img_003 | Hai người bán hàng rong, một người ở trên vỉa hè, một người ở dưới lòng đường | 1 | Gán nhãn thành 2 class 'dynamic person' cho người và 'hàng rong' cho gánh hàng mà người đó sử dụng | Phân biệt phương tiện đặc biệt và người tham gia giao thông. |

Nếu chưa chốt, đánh dấu `can_review` trên box hoặc ghi vấn đề cấp ảnh. Dùng mã quy tắc trong nhật ký QC.

## 5. QC và phiên bản

- Mọi thay đổi làm đổi nhãn đều tăng phiên bản guideline.
- Ca mơ hồ: tick `can_review`.
