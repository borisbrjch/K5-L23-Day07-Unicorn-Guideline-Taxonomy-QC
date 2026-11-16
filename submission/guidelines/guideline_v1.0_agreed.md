# Guideline gán nhãn — v1.0 (bản nhóm thống nhất)

> Nhãn bounding box 2D cho bài toán phát hiện **người và phương tiện** trên ảnh đường phố.
> Bản này là nguồn sự thật duy nhất cho mọi lượt gán; `submission/taxonomy.json` phải khớp **1-1** với §2.

| Hạng mục | Giá trị |
| --- | --- |
| Nhóm | Unicorn |
| Phiên bản | `v1.0` |
| Trạng thái | 🔒 Đã chốt — khoá cho lượt gán độc lập (Pha 3) |
| Loại nhãn | Bounding box (hộp chữ nhật) trên ảnh đường phố |
| Công cụ | CVAT, xuất **COCO 1.0** |
| Taxonomy đi kèm | [`submission/taxonomy.json`](../taxonomy.json) |
| Cấu hình CVAT sinh tự động | `cvat/labels_generated.json` |

**Mục lục**

- [1. Mục tiêu và phạm vi](#1-mục-tiêu-và-phạm-vi)
- [2. Lớp và thuộc tính](#2-lớp-và-thuộc-tính)
- [3. Quy tắc vẽ box](#3-quy-tắc-vẽ-box)
- [4. Ca khó](#4-ca-khó)
- [5. QC và phiên bản](#5-qc-và-phiên-bản)
- [6. Câu hỏi mở](#6-câu-hỏi-mở)
- [7. Lịch sử thay đổi](#7-lịch-sử-thay-đổi)

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
| `motorcycle` | Phương tiện hai bánh chạy bằng động cơ: xe máy số, xe tay ga, mô tô phân khối lớn. | Xe máy số, xe tay ga lưu thông trên đường, xe máy chở hàng cồng kềnh. | Xe đạp, xích lô chạy bằng sức người. |
| `bicycle` | Phương tiện thô sơ hai bánh chạy bằng sức người. | Xe đạp. | Xe máy, xe gắn máy. |
| `street_vendor` | Gánh hàng rong hoặc xe đạp chở sạp hàng rong. | Xe đạp chở sạp hàng; người gánh sạp hàng. | Người đi bộ thuần túy. |
| `dynamic_person` | Người đang di chuyển. | Người đi bộ đang tham gia giao thông: trên vỉa hè, lòng đường, vạch sang đường. | Người đang ngồi hoặc đứng yên. |
| `static_person` | Người không tham gia giao thông. | Người đang ngồi, không ở phần đường dành cho xe cộ. | Người gánh hàng rong; người đang đi. |
| `moto_rider` | Người điều khiển phương tiện gắn máy. | Người đang điều khiển xe máy. | Người đi bộ. |
| `bicycle_rider` | Người điều khiển xe đạp / xe thô sơ. | Người đang điều khiển xe đạp. | Người đi bộ. |
| `ignore_region` | Vùng bỏ qua: không gán từng vật bên trong; bỏ sót trong vùng này **không tính là lỗi**. | Đám đông dày đặc không tách nổi box; cụm vật cao `< 20 px`; vùng đã xét nhưng chủ ý không gán (tượng, hình in, ảnh phản chiếu) khi cần ghi nhận có chủ ý. | Một người / một phương tiện thật đứng riêng lẻ. |

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
| **R-02** | **Không gán lớp thật cho vật không thật:** xe mô hình, xe đồ chơi, người mô hình, tượng, ma-nơ-canh, hình in trên biển quảng cáo, ảnh phản chiếu qua gương. Nếu vùng đó cần được ghi nhận là "đã xét", vẽ `ignore_region`. |
| **R-03** | **Box ôm sát phần nhìn thấy** của vật. Không gộp hàng hóa rời, thùng hàng tháo rời vào box thân xe; không vẽ box lỏng hơn vật. |
| **R-04** | Mỗi vật **đúng một lớp** và **đầy đủ thuộc tính**. Đọc §2.1 trước khi chọn lớp; không tự tạo lớp mới. |
| **R-05** | Vật chồng lấn / bị che: gán `occluded_level` theo §2.2. Vật bị cắt ở mép ảnh: `truncated=yes`. |
| **R-06** | Vật có **chiều cao `< 20 px`** không gán box riêng. Nếu cả cụm nhỏ như vậy nằm gọn trong một vùng → một box `ignore_region`. |
| **R-07** | **Không gán ego vehicle:** nắp capo, gương chiếu hậu, phần thân xe chở camera xuất hiện ở viền khung hình không được gán nhãn phương tiện tham gia giao thông. |
| **R-08** | Người **đang điều khiển** xe máy / xe đạp gán lớp `moto_rider` / `bicycle_rider` — không gán thêm `dynamic_person` cho cùng một người. Người ngồi trên xe nhưng **đứng yên** gán `static_person` (xem ca `img_003`). |

---

## 4. Ca khó

| Ảnh | Tình huống | Mã quy tắc | Quyết định | Lý do |
| --- | --- | --- | --- | --- |
| `img_003` | Có người ngồi trên xe máy nhưng không di chuyển, vừa đỗ trên vỉa hè, vừa đỗ dưới lòng đường. | R-08 | Gán `static_person` | Vật thể này đang không di chuyển và cũng sẽ không bất ngờ di chuyển. |
| `img_003` | Hai người bán hàng rong, một người ở trên vỉa hè, một người ở dưới lòng đường. | R-04 | Gán 2 lớp: `dynamic_person` cho người và `street_vendor` cho gánh hàng người đó sử dụng. | Phân biệt phương tiện đặc biệt với người tham gia giao thông. |

> Chưa chốt được: tick `can_review` trên box hoặc ghi vấn đề cấp ảnh. Mọi ca mới phát sinh khi gán phải được
> bổ sung vào bảng này kèm mã quy tắc, trước khi tính IAA.

---

## 5. QC và phiên bản

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

## 6. Câu hỏi mở

Các điểm dưới đây **cần nhóm chốt** trước lượt gán độc lập; hiện đã xử lý theo hướng ghi trong ngoặc.

| # | Vấn đề | Hướng xử lý hiện tại |
| --- | --- | --- |
| Q1 | `occluded_level` bản nháp để trống khoảng 30–40% và 70–80%. | Đã làm liền mạch: `2` = 31–70%, `3` = ≥ 71%. |
| Q2 | Bảng lớp nói `ignore_region` dùng cho "xe đồ chơi, biển quảng cáo", nhưng R-02 nói không gán. | Đã thống nhất: vật giả đơn lẻ **không vẽ gì**; `ignore_region` chỉ khi cần ghi nhận vùng đã xét hoặc đám đông / cụm vật `< 20 px`. |
| Q3 | Xe bus 16 chỗ bị loại khỏi `car`, `van` nhưng **không có lớp nào nhận**. | **Đã chốt:** bổ sung lớp `bus` (xe buýt, xe khách, xe từ 16 chỗ trở lên) — xem §2.1. |
| Q4 | Người ngồi **sau** xe máy đang chạy (không điều khiển) chưa được mô tả. | Chưa chốt — đề xuất `dynamic_person`. |
| Q5 | Xe ba bánh / xích lô có động cơ. | Chưa chốt — đề xuất `truck` nếu có thùng hàng, ngược lại `ignore_region`. |

---

## 7. Lịch sử thay đổi

| Phiên bản | Ngày | Thay đổi |
| --- | --- | --- |
| `v0` | — | Bản khởi đầu do giảng viên cung cấp (5 lớp + `ignore_region`). |
| `v1.0` | 2026-11-16 | Nhóm thống nhất 11 lớp + `ignore_region` (thêm `bus`); chuẩn hoá tên lớp sang `snake_case`; thêm mã quy tắc R-01…R-08; thêm mục rủi ro an toàn, checklist export và câu hỏi mở. |
