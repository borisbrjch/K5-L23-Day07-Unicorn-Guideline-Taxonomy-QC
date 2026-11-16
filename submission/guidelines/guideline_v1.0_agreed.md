# Guideline gán nhãn — v0 (bản khởi đầu)

- Phiên bản: v1.0
- Loại nhãn: bounding box (hộp chữ nhật) trên ảnh đường phố
- Công cụ: CVAT, xuất COCO 1.0

---

## 1. Mục tiêu

Nhãn bounding box 2D được sử dụng để huấn luyện và đánh giá mô hình phát hiện vật thể **người và phương tiện** trên ảnh đường phố
(cho camera giao thông hoặc xe tự lái).

### Phân tích rủi ro an toàn:
- **Bỏ sót người hoặc phương tiện thực tế (False Negative):** Là lỗi nghiêm trọng nhất. Bỏ sót người đi bộ, người đi xe máy có thể dẫn đến va chạm gây tai nạn nguy hiểm đến tính mạng.
- **Gán nhầm vật vô tri thành người / phương tiện (False Positive):** Gán nhầm tượng đá, tượng đồng, ma-nơ-canh, hình in 2D trên biển quảng cáo hoặc ảnh phản chiếu qua gương sẽ khiến xe tự hành nhận diện nhầm là chướng ngại vật thực sự, gây ra hiện tượng phanh gấp ngoài ý muốn (phantom braking), tiềm ẩn nguy cơ xe phía sau đâm dồn toa.
- **Kéo box sai kích thước (Bounding Box Jitter / Loose Box):** Bao gồm cả khối hàng hóa rời cồng kềnh vào box thân xe hoặc vẽ box quá lỏng sẽ làm sai lệch mô-đun ước lượng khoảng cách không gian 3D (3D depth estimation), dẫn đến tính toán sai khoảng cách an toàn khi phanh.

## 2. Lớp và thuộc tính

### Lớp (mỗi vật thuộc đúng một lớp)

| Lớp | Định nghĩa | Positive Example | Negative Example |
|---|---|
| `car` | Ô tô con từ 4 đến 9 chỗ chở người, xe sedan, hatchback, SUV. | Bao gồm xe 4 chỗ, suv, hatchback. | Xe chở hàng, xe bus 16 chỗ, xe bán tải chở hàng chở thùng, xe van thùng kín, xe mô hình.  |
| `truck` | Xe tải chở hàng mọi trọng tải, xe bán tải (vừa chở hàng vừa chở người), xe đầu kéo, xe chuyên dụng có cabin và thùng hàng/sàn tải tách rời. | Xe tải chở ô tô, xe tải nhỏ giao hàng thùng hở | Xe van liền thân kín | 
| `van` |  Xe ô tô thùng kín liền khối | Xe thùng kín cỡ 6 - 9 chỗ |  Xe bus 16 chỗ, xe bán tải, xe con |
| `motorcycle` | Phương tiện giao thông hai bánh chạy bằng động cơ (xe máy số, xe tay ga, mô tô phân khối lớn). | Xe máy số, xe tay ga lưu thông trên đường, xe máy chở hàng cồng kềnh | Xe đạp, xích lô chạy bằng sức người |
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
7. Không gán ego vehicle: Bộ phận của xe chở camera (nắp capo, gương chiếu hậu mép dưới khung hình) xuất hiện ở viền ảnh KHÔNG được gán nhãn phương tiện tham gia giao thông.

## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| img_003 | Có người ngồi trên xe máy nhưng không di chuyển, vừa đỗ trên vỉa hè, vừa đỗ dưới lòng đường. | 1 | Gắn nhãn 'static person' | Vật thể này đang không di chuyển và cũng sẽ không bất ngờ di chuyển|
|
| img_003 | Hai người bán hàng rong, một người ở trên vỉa hè, một người ở dưới lòng đường | 1 | Gán nhãn thành 2 class 'dynamic person' cho người và 'hàng rong' cho gánh hàng mà người đó sử dụng | Phân biệt phương tiện đặc biệt và người tham gia giao thông. |


## 5. QC và phiên bản

- Mọi thay đổi làm đổi nhãn đều tăng phiên bản guideline.
- Ca mơ hồ: tick `can_review`.
