# Guideline gán nhãn — v1.1 (sau calibration)

> Nhãn bounding box 2D cho bài toán phát hiện **người và phương tiện** trên ảnh đường phố.
> Bản này là nguồn sự thật duy nhất cho mọi lượt gán; `submission/taxonomy.json` phải khớp **1-1** với §2.

| Hạng mục | Giá trị |
| --- | --- |
| Nhóm | Unicorn |
| Phiên bản | `v1.1` |
| Trạng thái | Calibration sau IAA vòng 1 (Pha 5) — dùng để sửa nhãn cuối |
| Loại nhãn | Bounding box (hộp chữ nhật) trên ảnh đường phố |
| Công cụ | CVAT, xuất **COCO 1.0** |
| Taxonomy đi kèm | [`submission/taxonomy.json`](../taxonomy.json) |
| Cấu hình CVAT sinh tự động | `cvat/labels_generated.json` |


---

## 1. Mục tiêu và phạm vi

Nhãn bounding box 2D dùng để **huấn luyện và đánh giá** mô hình phát hiện vật thể người và phương tiện
trên ảnh đường phố (camera giao thông hoặc xe tự lái).

### 1.1. Phân tích rủi ro an toàn

| Lỗi | Hậu quả thực tế | Mức ưu tiên |
| --- | --- | --- |
| **Bỏ sót người / phương tiện thật** (false negative) | Lỗi nghiêm trọng nhất. Bỏ sót người đi bộ, người đi xe máy có thể dẫn tới va chạm gây tai nạn nguy hiểm đến tính mạng. | 🔴 Cao nhất |
| **Gán nhầm vật vô tri thành người / phương tiện** (false positive) | Tượng đá, tượng đồng, ma-nơ-canh, hình in 2D trên biển quảng cáo, ảnh phản chiếu qua gương bị nhận là chướng ngại vật thật → phanh gấp ngoài ý muốn (*phantom braking*), xe phía sau đâm dồn toa. | 🟠 Cao |
| **Box sai kích thước** (box lỏng / gộp hàng hóa rời) | Bao cả khối hàng cồng kềnh vào thân xe hoặc vẽ box quá lỏng làm lệch ước lượng khoảng cách 3D (*3D depth estimation*) → tính sai khoảng cách an toàn khi phanh. | 🟠 Cao |

### 1.2. Ngoài phạm vi

- Không gán nhãn vùng trời, mặt đường, vạch kẻ đường, biển báo.
- Không gán nhãn xe chở camera (**ego vehicle**) — xem [R-07](#3-quy-tắc-vẽ-box).
- Ảnh mờ / hỏng không đọc được: **không vẽ bừa**, ghi vấn đề cấp ảnh vào `image_issues.csv`.

---

## 2. Lớp và thuộc tính

### 2.1. Bảng lớp

| Lớp | Định nghĩa | ✅ Nên gán | ❌ Không gán |
| --- | --- | --- | --- |
| `car` | Ô tô con 4–9 chỗ chở người: sedan, hatchback, SUV. | Xe 4 chỗ, SUV, hatchback. | Xe chở hàng, xe bus 16 chỗ, xe bán tải chở thùng hàng, xe van thùng kín, xe mô hình. |
| `truck` | Xe tải chở hàng mọi trọng tải; xe bán tải (vừa chở hàng vừa chở người); xe đầu kéo; xe chuyên dụng có cabin và thùng hàng / sàn tải tách rời. | Xe tải chở ô tô, xe tải nhỏ giao hàng thùng hở. | Xe van liền thân kín. |
| `van` | Ô tô thùng kín liền khối. | Xe thùng kín cỡ 6–9 chỗ. | Xe bus 16 chỗ, xe bán tải, xe con. |
| `bus` | Xe chở nhiều người: xe buýt, xe khách, xe từ 16 chỗ trở lên. | Xe buýt nội thành, xe khách đường dài, xe 16 chỗ. | Xe van thùng kín 6–9 chỗ, xe con 4–9 chỗ, xe tải chở hàng. |
| `tuk_tuk` | Xe ba bánh có **khoang khách ngồi** với mui che, thiết kế để chở người (tuk tuk, xe ba gác chở khách). | Xe ba gác có khoang khách kín hoặc mui che; tuk tuk. | Xe ba bánh có sàn tải/thùng hàng; xe xích lô; xe máy; xe đạp. |
| `xe_3_banh` | Xe ba bánh có **sàn tải hoặc thùng hàng**, thiết kế để chở hàng. Phân loại dựa trên **thiết kế thân xe**, không phải hàng hóa hiện đang chở. | Xe ba bánh chở hàng, có sàn tải hoặc thùng hàng phía sau cabin (`img_035`). | Xe ba bánh có khoang khách ngồi (`tuk_tuk`). |
| `motorcycle` | Phương tiện hai bánh chạy bằng động cơ: xe máy số, xe tay ga, mô tô phân khối lớn. | Xe máy số, xe tay ga lưu thông trên đường, xe máy chở hàng cồng kềnh. | Xe đạp, xích lô chạy bằng sức người; mô tô gắn thùng phụ bên hông cố định (`side_car`). |
| `side_car` | Mô tô hai bánh gắn **thùng phụ bên hông** cố định vào khung (sidecar). Cả mô tô và thùng phụ là **một** phương tiện → gán **một** box bao trùm cả hai. | Mô tô cổ gắn thùng phụ bên hông (`img_022`). | Mô tô không thùng phụ (`motorcycle`); xe ba bánh đóng khung liền (`tuk_tuk`, `xe_3_banh`) |
| `bicycle` | Phương tiện thô sơ chạy bằng sức người: xe đạp, xích lô đạp. | Xe đạp; xích lô (`img_023`). | Xe máy, xe gắn máy; xe đạp chở sạp hàng rong (`street_vendor`). |
| `street_vendor` | Gánh hàng rong hoặc xe đạp chở sạp hàng rong. | Xe đạp chở sạp hàng; người gánh sạp hàng. | Người đi bộ thuần túy. |
| `dynamic_person` | Người đi bộ **đang bước đi** (thấy rõ tư thế bước, ảnh mờ do chuyển động). | Người đi bộ đang tham gia giao thông: trên vỉa hè, lòng đường, vạch sang đường; người gánh hàng rong đang đi. | Người đứng chờ, xếp hàng, đứng nói chuyện; người ngồi. |
| `static_person` | Người **không có khả năng di chuyển đột ngột**: đang ngồi hoặc đứng yên, kể cả khi ở lòng đường; người ngồi trên xe đã **đỗ hẳn** (chống chân, đỗ trên vỉa hè/lề). | Người đang ngồi; người đứng xếp hàng chờ xe buýt (`img_032`); người ngồi trên xe máy đỗ (`img_003`). | Người đang bước đi; người ngồi lái xe đang dừng trong dòng xe. |
| `moto_rider` | Người điều khiển hoặc ngồi sau phương tiện gắn máy **đang ở trong dòng xe**, kể cả khi tạm dừng vì kẹt xe / đèn đỏ. | Người đang điều khiển xe máy; người ngồi lái chờ đèn đỏ (`img_021`, `img_040`). | Người đi bộ; người ngồi trên xe máy đỗ hẳn (`static_person`). |
| `bicycle_rider` | Người điều khiển xe đạp / xe thô sơ. | Người đang điều khiển xe đạp. | Người đi bộ. |
| `ignore_region` | Vùng bỏ qua: không gán từng vật bên trong; bỏ sót trong vùng này **không tính là lỗi**. | Đám đông dày đặc không tách nổi box; cụm vật cao `< 20 px`. | Một người / một phương tiện thật đứng riêng lẻ; vật không thật (tượng, ma-nơ-canh, mô hình, hình in, ảnh phản chiếu) — xem R-02. |

> `ignore_region` không có thuộc tính — không cần điền `occluded_level`, `truncated`, `can_review`.

### 2.2. Bảng thuộc tính

| Thuộc tính | Kiểu | Giá trị | Mặc định | Ý nghĩa |
| --- | --- | --- | --- | --- |
| `occluded_level` | `select` | `0` `1` `2` `3` | `0` | Mức bị che: `0` = thấy rõ (0%); `1` = bị che ít (1–30%); `2` = bị che nhiều (31–70%); `3` = bị che nặng, chỉ còn nhận ra lớp (≥ 71%). |
| `truncated` | `select` | `no` `yes` | `no` | Vật bị cắt ở mép ảnh. |
| `can_review` | `checkbox` | tick / không tick | không tick | Tick khi bạn **không chắc** để người khác xem lại. |

> Thuộc tính bị **khoá** trong CVAT (`mutable: false`): phải chọn đúng ngay khi tạo box, không sửa được sau.


---

## 3. Quy tắc vẽ box

Mọi tranh luận về sau phải dẫn được **mã quy tắc** dưới đây.

| Mã | Quy tắc |
| --- | --- |
| **R-01** | Gán box cho **mọi** vật thuộc các lớp ở §2.1, trừ các ngoại lệ R-02, R-06, R-07. Bỏ sót người / phương tiện là lỗi nặng nhất. |
| **R-02** | **Không gán lớp thật cho vật không thật:** xe mô hình, xe đồ chơi, người mô hình, tượng, ma-nơ-canh, hình in trên biển quảng cáo, ảnh phản chiếu qua gương. **Không vẽ box nào** lên vật không thật, kể cả `ignore_region` (để mô hình vẫn bị phạt khi nhận nhầm vật giả — §1.1). |
| **R-03** | **Box ôm sát phần nhìn thấy** của vật. Không gộp hàng hóa rời, thùng hàng tháo rời vào box thân xe; không vẽ box lỏng hơn vật. |
| **R-04** | Mỗi vật **đúng một lớp** và **đầy đủ thuộc tính**. Đọc §2.1 trước khi chọn lớp; không tự tạo lớp mới. |
| **R-05** | Vật chồng lấn / bị che: gán `occluded_level` theo §2.2. Vật bị cắt ở mép ảnh: `truncated=yes`. |
| **R-06** | Vật có **chiều cao `< 20 px`** không gán box riêng. Nếu cả cụm nhỏ như vậy nằm gọn trong một vùng → một box `ignore_region`. |
| **R-07** | **Không gán ego vehicle:** nắp capo, gương chiếu hậu, phần thân xe chở camera xuất hiện ở viền khung hình không được gán nhãn phương tiện tham gia giao thông. |
| **R-08** | Người ngồi trên xe máy / xe đạp **trong dòng xe** gán `moto_rider` / `bicycle_rider`, **kể cả khi tạm dừng** (kẹt xe, chờ đèn đỏ) — không gán thêm `dynamic_person` cho cùng một người. Chỉ gán `static_person` khi xe đã **đỗ hẳn** (chống chân, đỗ trên vỉa hè / lề; ca `img_003`). Mỗi người cưỡi xe là **2 box**: box người (`moto_rider`) và box xe (`motorcycle`), kể cả khi vật nhỏ ở xa (`img_034`). |
| **R-09** | **Mô tô gắn thùng phụ (sidecar):** Cả mô tô và thùng phụ gắn cố định là **một** phương tiện → gán **một** box `side_car`. Không tách thành 2 box (`motorcycle` + phụ tiện riêng). Người điều khiển gán `moto_rider`, người ngồi trong thùng phụ gán `static_person`. |
| **R-10** | **Người đứng yên vs đang đi:** chỉ gán `dynamic_person` khi thấy rõ tư thế bước đi hoặc mờ chuyển động. Người đứng chờ, xếp hàng, đứng nói chuyện → `static_person` (`img_032`). |
| **R-11** | **Ngưỡng occluded_level:** ước theo **tỉ lệ diện tích vật bị che** so với toàn bộ vật (không phải so với box). Phân vân giữa hai mức → chọn mức **cao hơn**. `truncated=yes` chỉ khi vật bị **mép ảnh** cắt, không dùng cho vật bị vật khác che. |
| **R-12** | **Phương tiện đặc thù:** xe bán tải / xe songthaew (thùng sau có ghế băng) → `truck` (`img_008`, `img_012`, `img_028`); xe đạp chở sạp hàng → `street_vendor` (`img_010`); xích lô → `bicycle`, người đạp → `bicycle_rider` (`img_023`); xe ba bánh thân tuk tuk chở khách vẫn là `tuk_tuk` dù chất hàng trên nóc (`img_016`, `img_035`). |

---

## 4. Ca khó

| Ảnh | Tình huống | Mã quy tắc | Quyết định | Lý do |
| --- | --- | --- | --- | --- |
| `img_003` | Có người ngồi trên xe máy nhưng không di chuyển, vừa đỗ trên vỉa hè, vừa đỗ dưới lòng đường. | R-08 | Gán `static_person` | Vật thể này đang không di chuyển và cũng sẽ không bất ngờ di chuyển. |
| `img_003` | Hai người bán hàng rong, một người ở trên vỉa hè, một người ở dưới lòng đường. | R-04 | Gán 2 lớp: `dynamic_person` cho người và `street_vendor` cho gánh hàng người đó sử dụng. | Phân biệt phương tiện đặc biệt với người tham gia giao thông. |
| `img_022` | Mô tô cổ gắn thùng phụ bên hông (sidecar), có người ngồi trong thùng phụ. | R-09 | Gán **1** box `side_car` bao trùm cả mô tô và thùng phụ; người điều khiển gán `moto_rider`, người ngồi trong thùng phụ gán `static_person` | Thùng phụ gắn cố định vào khung mô tô nên là một phương tiện, không tách thành 2 box; cũng không phải xe ba bánh đóng khung liền |
| `img_035` | Xe ba bánh màu vàng chất đầy bao tải trên nóc và sau thân xe. | R-12, R-03, R-05 | **(sửa ở v1.1)** Gán `tuk_tuk` ôm sát thân xe, **không** gộp khối bao tải; `occluded_level=2` | Thân xe là tuk tuk chở khách (mui che khoang khách); hàng trên nóc không đổi công năng thiết kế. Cả 143 và 180 đều không gán `xe_3_banh` cho xe này. |
| `img_021` | Hàng chục người ngồi lái xe máy đang dừng chờ trong dòng xe kẹt. | R-08 | `moto_rider` + `motorcycle` cho từng xe | Xe đang trong làn, có thể chạy ngay; khác ca `img_003` xe đỗ hẳn. IAA v1.0: 17 ca lệch `moto_rider`↔`static_person`. |
| `img_032` | Nhóm người đứng xếp hàng ở trạm xe buýt trên vỉa hè. | R-10 | `static_person` | Đứng chờ, không bước đi. IAA v1.0: 18 ca lệch `dynamic_person`↔`static_person`. |
| `img_017`, `img_020`, `img_027`, `img_033`, `img_038` | Tượng, ảnh phản chiếu, ma-nơ-canh, xe mô hình. | R-02 | Không vẽ box nào | v1.0 cho phép `ignore_region` "khi cần ghi nhận", dẫn tới một người vẽ, một người không. |
| `img_016` | Tuk tuk bị gán `truck` / `van`. | R-12 | `tuk_tuk` | Có mui che khoang khách. |
| `img_022` | Sidecar bị gán `motorcycle` cho riêng thùng phụ. | R-09 | Một box `side_car` bao cả mô tô và thùng phụ | Theo R-09 v1.0. |

> Chưa chốt được: tick `can_review` trên box hoặc ghi vấn đề cấp ảnh. Mọi ca mới phát sinh khi gán phải được
> bổ sung vào bảng này kèm mã quy tắc, trước khi tính IAA.

---

## 5. QC và phiên bản

- **Cảnh báo thay đổi taxonomy:** việc bổ sung `xe_3_banh` và `side_car` làm thay đổi lớp và định nghĩa ranh giới nhãn. Trước khi áp dụng cho lượt gán chính thức, hãy tăng phiên bản guideline và `submission/taxonomy.json` đồng bộ, sau đó chạy `python scripts/make_cvat_labels.py`. Nếu đây là cập nhật sau khi đã khóa v1.0, ghi nhận thành phiên bản mới (ví dụ v1.1), không âm thầm thay đổi v1.0.

- Mọi thay đổi **làm đổi nhãn** (thêm / bớt lớp, đổi định nghĩa ranh giới, đổi giá trị thuộc tính) đều phải **tăng phiên bản** guideline và `taxonomy.json`, rồi chạy lại `python scripts/make_cvat_labels.py`.
- Sửa câu chữ không đổi cách gán → ghi `"kind": "editorial"` trong `submission/guideline_change.json`; đổi nghĩa → `"semantic"`.
- Ca mơ hồ: tick `can_review`.
- Trước khi export COCO 1.0, tự kiểm bằng checklist dưới.

### Checklist trước khi export

- [ ] Đã gán đủ vật thuộc §2.1 (R-01), kể cả vật bị che một phần.
- [ ] Không có box nào trên tượng, hình in, ảnh phản chiếu, mô hình (R-02).
- [ ] Mọi box ôm sát phần nhìn thấy, không gộp hàng hóa rời (R-03).
- [ ] Mọi box có đủ `occluded_level` và `truncated`; box mơ hồ có `can_review` (R-04, R-05).
- [ ] Không có box nào trên phần ego vehicle ở viền ảnh (R-07).
- [ ] Không có box riêng cho vật cao `< 20 px` (R-06).
- [ ] `python scripts/validate_labels.py submission/labels/<mã>.json` chạy sạch.

---

## 7. Lịch sử thay đổi

| Phiên bản | Ngày | Thay đổi |
| --- | --- | --- |
| `v0` | — | Bản khởi đầu do giảng viên cung cấp (5 lớp + `ignore_region`). |
| `v1.0` | 2026-11-16 | Nhóm thống nhất 14 lớp + `ignore_region` (thêm `tuk_tuk`, `xe_3_banh`, `side_car` so với v0); làm kín ngưỡng `occluded_level` (1–30 / 31–70 / ≥71); định nghĩa `static_person` theo khả năng di chuyển đột ngột; bổ sung R-08, R-09 và ca khó `img_003`, `img_022`, `img_035`. |
| `v1.1` | 2026-11-17 | **Semantic.** Sau IAA vòng 1 (143\|180: IoU TB 0.836, cùng lớp 62.2% trên 283 cặp — chưa đạt). Sửa R-02 (bỏ `ignore_region` cho vật không thật); sửa R-08 (người lái xe tạm dừng trong dòng xe = `moto_rider`; 2 box người + xe); thêm R-10 (đứng yên vs đi), R-11 (cách ước `occluded_level`, `truncated`), R-12 (phương tiện đặc thù); `bicycle` gồm xích lô; đổi ca `img_035` sang `tuk_tuk`; thêm ca `img_016`, `img_021`, `img_032`, vật không thật. Taxonomy không đổi (vẫn 15 lớp, `rules` giữ nguyên). Phạm vi sửa: cả lô 40 ảnh. |
