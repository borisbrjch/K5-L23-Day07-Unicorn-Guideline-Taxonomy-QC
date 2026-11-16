# Guideline gán nhãn — v0.1 — Thành viên 143

- Phiên bản: v0.1 (sửa từ bản khởi đầu `v0` của giảng viên)
- Ngày: 2026-11-16
- Người viết: Thành viên 143
- Loại nhãn: bounding box (hộp chữ nhật) trên ảnh đường phố
- Công cụ: CVAT, xuất COCO 1.0

---

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình phát hiện **người và phương tiện** trên ảnh đường phố
(ví dụ cho camera giao thông hoặc xe tự lái).

## 2. Lớp và thuộc tính

> Tên lớp viết `snake_case` ASCII (chỉ `a-z`, `0-9`, `_`) để dán được thẳng vào
> `submission/taxonomy.json` — schema kiểm bằng `^[a-z][a-z0-9_]*$`.

### Lớp (mỗi vật thuộc đúng một lớp)

| Lớp | Định nghĩa | Positive Example | Negative Example |
|---|---|---|---|
| `car` | Ô tô con. | Bao gồm xe 4 chỗ, SUV, hatchback. | Xe chở hàng, xe bus 16 chỗ |
| `truck` | Xe tải. | Xe chở hàng, hoặc vừa chở hàng vừa chở người | Xe con, xe bus, xe ba bánh |
| `van` | Xe thùng kín cỡ nhỏ. | Xe thùng kín cỡ 6 - 9 chỗ | Xe bus 16 chỗ, xe bán tải, xe con |
| `motorcycle` | Xe gắn máy, hai bánh | Xe máy, mô tô | Xe đạp; xe ba bánh (`tuk_tuk`, `xe_3_banh`); mô tô gắn thùng phụ (`side_car`) |
| `bicycle` | Phương tiện thô sơ hai bánh | Xe đạp. | Xe mô tô gắn máy |
| `tuk_tuk` | Xe ba bánh **chở khách**: có mui xe che khoang khách và ghế ngồi cho khách (tuk tuk, auto rickshaw). | Tuk tuk có mui che ghế khách, khách ngồi đối diện hoặc phía sau tài xế (`img_004`, `img_016`) | Xe ba bánh chở hàng, sau cabin là sàn/thùng hàng (`xe_3_banh`); xích lô đạp; xe máy |
| `xe_3_banh` | Xe ba bánh **chở hàng**: xe ba gác máy, xe lam; sau cabin tài xế là sàn tải hoặc thùng hàng thay cho khoang khách. | Xe ba bánh có sàn tải/thùng hàng chở bao tải (`img_035`) | Tuk tuk chở khách có mui che khoang khách (`tuk_tuk`); xích lô đạp ba bánh; xe máy |
| `side_car` | Mô tô hai bánh gắn **thùng phụ bên hông** cố định vào khung xe (sidecar). Cả mô tô và thùng phụ là **một** phương tiện, gán **một** box bao trùm cả hai. | Mô tô cổ gắn thùng phụ bên hông (`img_022`) | Mô tô không thùng phụ (`motorcycle`); xe ba bánh đóng khung liền (`tuk_tuk`, `xe_3_banh`) |
| `street_vendor` | Gánh hàng rong hoặc xe đạp chở sạp hàng rong | Xe đạp chở sạp hàng hoặc người gánh sạp hàng | Người đi bộ |
| `dynamic_person` | Người đang đi lại, có khả năng di chuyển bất ngờ. | Người đi bộ đang tham gia giao thông (xuất hiện ở vỉa hè, lòng đường, cross walk). | Người đang ngồi hoặc đứng yên |
| `static_person` | Người **không có khả năng di chuyển đột ngột**: đang ngồi hoặc đứng yên, kể cả khi ở lòng đường (ví dụ người ngồi trên xe máy đỗ). | Người đang ngồi, người đứng chờ, người ngồi trên xe máy đỗ (`img_003`) | Gánh hàng rong, người đang đi |
| `moto_rider` | Người điều khiển phương tiện xe gắn máy. | Người điều khiển xe máy | Người đi bộ |
| `bicycle_rider` | Người điều khiển xe đạp, xe thô sơ. | Người điều khiển xe đạp | Người đi bộ |
| `ignore_region` | Vùng không cần gán từng vật: đám đông dày đặc không tách nổi box, cụm vật cao `< 20 px`. Bỏ sót trong vùng này không tính là lỗi. | Đám đông phía chân trời, cụm xe nhỏ li ti | Người, phương tiện thật đứng riêng lẻ |

> Vật vô tri (tượng, ma-nơ-canh, xe đồ chơi, hình in trên biển quảng cáo, ảnh phản chiếu)
> **không gán lớp nào** và **không** dùng `ignore_region` để che — xem quy tắc 2 ở §3.

### Thuộc tính

| Thuộc tính | Giá trị | Ý nghĩa |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | Mức bị vật khác che khuất phần nhìn thấy của vật: `0` = thấy rõ, không bị che (0%); `1` = bị che ít (1–30% diện tích nhìn thấy); `2` = bị che nhiều (31–70%); `3` = bị che nặng, chỉ còn nhận ra được lớp (≥ 71%). |
| `truncated` | `no`, `yes` | Vật bị cắt ở mép ảnh. |
| `can_review` | tick / không | Tick khi bạn không chắc, để người khác xem lại. |

> Bốn mức của `occluded_level` phủ kín 0–100% và không chồng lấn, để hai người gán
> cùng một box ra cùng một mức.

## 3. Quy tắc vẽ box

1. Gán box cho tất cả các class đã được định nghĩa trong guideline.
2. Không gắn nhãn cho những vật false positive, ví dụ như xe mô hình, xe đồ chơi, người mô hình, tượng, ma-nơ-canh, hình ảnh quảng cáo, ảnh phản chiếu qua gương. Chỉ gắn nhãn cho vật thể là người, phương tiện thật.
3. Box ôm sát vật.
4. Chọn đúng lớp và thuộc tính.
5. Với object overlapping thì gắn `occluded_level` theo §2; vật bị cắt ở mép ảnh thì `truncated=yes`.
6. Vật thể nhỏ có chiều cao < 20px không cần gắn nhãn (gộp cả cụm vào `ignore_region` nếu cần).
7. **Phân biệt phương tiện ba bánh:** căn cứ vào **công năng thiết kế của thân xe**, không căn cứ vào việc xe đang chở gì tại thời điểm chụp.
   Có mui che khoang khách + ghế ngồi cho khách → `tuk_tuk`. Sau cabin tài xế là sàn tải / thùng hàng → `xe_3_banh`.
   Mô tô hai bánh gắn thùng phụ bên hông cố định vào khung → `side_car`, gán **một** box bao trùm cả mô tô và thùng phụ.
   Xe chở khách có chất thêm hàng trên nóc vẫn là `tuk_tuk`. Không đọc được thân xe → tick `can_review`.
8. **Hàng hóa cồng kềnh:** box phương tiện chỉ ôm thân vỏ, khung, bánh và kết cấu cơ khí cố định.
   Không gộp bao tải, thùng hàng rời chất trên nóc hoặc buộc sau xe vào box.
   Nếu khối hàng che mất một phần thân xe → tăng `occluded_level` tương ứng.
9. **Người đi xe máy — 2 box riêng:** mỗi người điều khiển hoặc ngồi sau xe máy gán 1 box
   `moto_rider`; xe máy gán 1 box `motorcycle`. Hai box chồng lấn tự nhiên ở vùng yên xe.
   Không gán thêm `dynamic_person` cho cùng một người.
10. **Xe chở xe** (`img_002`): gán 1 box `truck` cho toàn bộ xe vận chuyển, và thêm 1 box `car`
    cho **từng xe con** nhận rõ hình dạng ô tô trên sàn. `occluded_level` của xe con tính theo
    mức bị thanh giàn che.

## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| img_003 | Có người ngồi trên xe máy nhưng không di chuyển, vừa đỗ trên vỉa hè, vừa đỗ dưới lòng đường. | 4 | Gắn nhãn `static_person` | Người này đang không di chuyển và không có khả năng di chuyển đột ngột |
| img_003 | Hai người bán hàng rong, một người ở trên vỉa hè, một người ở dưới lòng đường | 1, 4 | Gán nhãn thành 2 class `dynamic_person` cho người và `street_vendor` cho gánh hàng mà người đó sử dụng | Phân biệt phương tiện đặc biệt và người tham gia giao thông. |
| img_002 | Xe tải chuyên dụng chở 8 ô tô con trên 2 tầng sàn. | 10 | Gán 1 box `truck` cho xe vận chuyển + 8 box `car` cho 8 xe con; `occluded_level` của xe con theo mức bị thanh giàn che | Xe con trên sàn có đủ đặc trưng hình học của ô tô, cần cho mô hình học nhận diện |
| img_004 | Hai xe tuk tuk ba bánh đỗ cạnh vỉa hè, có mui che ghế khách. | 7 | Gán `tuk_tuk` cho từng xe; người ngồi trong khoang khách gán `static_person` | Có mui che khoang khách và ghế ngồi cho khách → xe chở khách |
| img_016 | Tuk tuk và xe máy cùng lưu thông, một xe van thùng kín phía sau. | 4, 7, 9 | Gán `tuk_tuk`, `motorcycle`, `van` cho từng phương tiện; người điều khiển xe máy gán `moto_rider` | Ba lớp phương tiện khác nhau, không gộp box |
| img_035 | Xe ba bánh màu vàng chất đầy bao tải trên nóc và sau thân xe. | 7, 8 | Gán `xe_3_banh` ôm sát thân xe, **không** gộp khối bao tải; `occluded_level=2` | Sau cabin tài xế là sàn tải chở hàng, không có khoang khách → xe chở hàng |
| img_022 | Mô tô cổ gắn thùng phụ bên hông (sidecar), có người ngồi trong thùng phụ. | 7, 9 | Gán **1** box `side_car` bao trùm cả mô tô và thùng phụ; người điều khiển gán `moto_rider`, người ngồi trong thùng phụ gán `static_person` | Thùng phụ gắn cố định vào khung mô tô nên là một phương tiện, không tách thành 2 box; cũng không phải xe ba bánh đóng khung liền |

Nếu chưa chốt, đánh dấu `can_review` trên box hoặc ghi vấn đề cấp ảnh. Dùng mã quy tắc trong nhật ký QC.

## 5. QC và phiên bản

- Mọi thay đổi làm đổi nhãn đều tăng phiên bản guideline.
- Ca mơ hồ: tick `can_review`.

| Phiên bản | Ngày | Thay đổi | Lý do |
|---|---|---|---|
| `v0` | — | Bản khởi đầu của giảng viên (6 lớp, thuộc tính `occluded_level` 0–3). | — |
| `v0.1` | 2026-11-16 | Đổi tên lớp sang `snake_case` ASCII (`bycicle`→`bicycle`, `hàng rong`→`street_vendor`, `dynamic person`→`dynamic_person`…); thêm `tuk_tuk`, `xe_3_banh` và `side_car`; làm kín ngưỡng `occluded_level` (1–30 / 31–70 / ≥71, hết khoảng hở so với v0); thêm quy tắc 7–10 và 5 ca khó mới. | Tên lớp cũ không dán được vào `taxonomy.json`; phải tách xe ba bánh chở khách, xe ba bánh chở hàng và mô tô gắn thùng phụ; v0 để hở 31–39% và 71–79% nên hai người gán ra hai mức khác nhau |
