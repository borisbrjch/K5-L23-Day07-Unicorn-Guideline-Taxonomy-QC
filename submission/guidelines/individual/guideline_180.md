# Guideline gán nhãn Bounding Box 2D — v0.1 — Thành viên 180

- Phiên bản: v0.1 (Bản cá nhân khởi tạo từ v0 và khảo sát 40 ảnh)
- Ngày: 2026-11-16
- Người viết: Thành viên 180
- Loại nhãn: 2D Bounding Box (Rectangle)
- Công cụ: CVAT, xuất chuẩn COCO 1.0

---

## 1. Mục tiêu

Nhãn bounding box 2D được sử dụng để huấn luyện và đánh giá mô hình phát hiện vật thể (Object Detection: YOLO, Faster R-CNN, DETR) phục vụ hệ thống **xe tự lái (Autonomous Driving)** và **hệ thống giám sát giao thông thông minh (ITS)** trong môi trường đô thị phức tạp.

### Phân tích rủi ro an toàn:
- **Bỏ sót người hoặc phương tiện thực tế (False Negative):** Là lỗi nghiêm trọng nhất. Bỏ sót người đi bộ, người đi xe máy có thể dẫn đến va chạm gây tai nạn nguy hiểm đến tính mạng.
- **Gán nhầm vật vô tri thành người / phương tiện (False Positive):** Gán nhầm tượng đá, tượng đồng, ma-nơ-canh, hình in 2D trên biển quảng cáo hoặc ảnh phản chiếu qua gương sẽ khiến xe tự hành nhận diện nhầm là chướng ngại vật thực sự, gây ra hiện tượng phanh gấp ngoài ý muốn (phantom braking), tiềm ẩn nguy cơ xe phía sau đâm dồn toa.
- **Kéo box sai kích thước (Bounding Box Jitter / Loose Box):** Bao gồm cả khối hàng hóa rời cồng kềnh vào box thân xe hoặc vẽ box quá lỏng sẽ làm sai lệch mô-đun ước lượng khoảng cách không gian 3D (3D depth estimation), dẫn đến tính toán sai khoảng cách an toàn khi phanh.

---

## 2. Lớp và thuộc tính

### Danh sách lớp (`classes`)

Mỗi đối tượng hợp lệ chỉ thuộc đúng một lớp:

| Lớp | Định nghĩa chuẩn | Ví dụ trong 40 ảnh | Phản ví dụ / Dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con từ 4 đến 9 chỗ chở người, xe sedan, hatchback, SUV, CUV, MPV du lịch. | Xe ô tô sedan, SUV di chuyển trên đường phố (`img_001.jpg`, `img_014.jpg`, `img_040.jpg`), ô tô con chở trên sàn xe tải (`img_002.jpg`). | Xe bán tải chở hàng hở thùng (`truck`), xe van thùng kín cỡ lớn (`van`), xe mô hình đồ chơi trẻ em (`img_038.jpg` — KHÔNG gán). |
| `truck` | Xe tải chở hàng mọi trọng tải, xe đầu kéo, xe chuyên dụng có cabin và thùng hàng/sàn tải tách rời. | Xe tải chở ô tô 2 tầng sàn (`img_002.jpg`), xe tải nhỏ giao hàng thùng hở (`img_007.jpg`, `img_012.jpg`, `img_036.jpg`). | Xe van liền thân kín (`van`), tượng xe bán tải bằng đá (`img_027.jpg` — KHÔNG gán). |
| `van` | Xe ô tô thùng kín liền khối (thân vỏ liền khối từ cabin ra khoang hàng/khoang khách phía sau). | Xe giao hàng thùng kín, xe bán tải gắn nắp thùng cao kín liền thân như Master 75 (`img_037.jpg`). | Xe tải có cabin tách biệt với thùng chở hàng rời (`truck`), ô tô con chở khách 4-5 chỗ (`car`). |
| `motorcycle` | Phương tiện giao thông hai bánh chạy bằng động cơ (xe máy số, xe tay ga, mô tô phân khối lớn). | Xe máy số, xe tay ga lưu thông trên đường (`img_005.jpg`, `img_008.jpg`, `img_026.jpg`), xe máy chở hàng cồng kềnh (`img_029.jpg`). | Xe đạp, xích lô chạy bằng sức người (`img_006.jpg`, `img_023.jpg`, `img_039.jpg` — KHÔNG gán vào lớp này), xe ba bánh tuk-tuk (`img_004.jpg`, `img_035.jpg` — ca biên đề xuất họp). |
| `person` | Người thật còn sống (người đi bộ, người đứng, người ngồi, người điều khiển hoặc ngồi sau phương tiện). | Người đứng bên đường (`img_001.jpg`), người bán hoa quả (`img_003.jpg`, `img_010.jpg`), người điều khiển xe máy (`img_005.jpg`, `img_026.jpg`). | Ma-nơ-canh thời trang (`img_033.jpg` — KHÔNG gán), tượng đồng/đá (`img_011.jpg`, `img_017.jpg` — KHÔNG gán), chân dung in trên pano quảng cáo (`img_018.jpg` — KHÔNG gán). |
| `ignore_region` | Vùng bỏ qua (không phải đối tượng đơn lẻ). Dùng để bao phủ đám đông người/xe quá dày đặc hoặc cụm vật nhỏ li ti. | Cụm người và phương tiện xa tít tắp ở đường chân trời có chiều cao $< 20\text{ px}$ (`img_025.jpg`, `img_040.jpg`). | Không dùng để che giấu lỗi bỏ sót vật thể rõ ràng ở tiền cảnh. |

> **Xử lý đối tượng ngoài taxonomy hiện tại:**
> - Xe đạp, xích lô chạy bằng sức người (`img_006.jpg`, `img_023.jpg`): Gán box `person` cho người đạp; không gán box cho khung xe đạp/xích lô (hoặc nhóm xem xét bổ sung lớp trong Pha 2).
> - Xe tuk-tuk / auto rickshaw 3 bánh (`img_004.jpg`, `img_035.jpg`): Tạm thời đánh dấu `can_review` hoặc xếp vào `motorcycle` và đưa ra họp nhóm ở Pha 2 để bổ sung lớp `tuk_tuk`.
> - Xe buýt (`img_009.jpg`, `img_032.jpg`, `img_040.jpg`): Tạm thời xếp vào `truck` hoặc đánh dấu `can_review` để thống nhất bổ sung lớp `bus`.

---

### Danh mục thuộc tính (`attributes`) với ngưỡng định lượng

| Thuộc tính | Kiểu | Các giá trị | Định nghĩa & Ngưỡng định lượng cụ thể |
|---|---|---|---|
| `occluded_level` | select | `0`, `1`, `2`, `3` | **Mức độ che khuất của vật thể bởi vật khác nằm phía trước:**<br>- `0`: Hoàn toàn không bị che khuất (0% diện tích bị che, toàn bộ đường bao nhìn rõ).<br>- `1`: Bị che khuất nhẹ (từ 1% đến 30% diện tích nhìn thấy bị che lấp).<br>- `2`: Bị che khuất trung bình đến nhiều (từ 31% đến 70% diện tích bị che lấp).<br>- `3`: Bị che khuất nặng (trên 70% diện tích bị che, chỉ nhìn thấy một phần nhỏ như bánh xe hoặc đầu xe nhưng vẫn nhận diện được). |
| `truncated` | select | `no`, `yes` | **Vật thể bị cắt bởi mép ảnh:**<br>- `no`: Vật thể nằm trọn vẹn trong khung hình, không chạm mép cắt của ảnh.<br>- `yes`: Một phần cơ thể hoặc thân xe bị cắt cụt bởi mép trên, dưới, trái hoặc phải của ảnh. |
| `can_review` | checkbox | `true` / `false` | **Đánh dấu cần xem lại:**<br>- `false` (mặc định): Tự tin về nhãn và thuộc tính theo quy tắc.<br>- `true`: Tick chọn khi gặp tình huống biên khó phân xử, cần đưa ra phiên calibration của nhóm. |

---

## 3. Quy tắc vẽ box chi tiết

- **`R-01` — Box ôm sát phần nhìn thấy (Tight Fit Box):** Bounding box phải ôm khít các điểm cực biên (x_min, y_min, x_max, y_max) của **phần vật thể thực sự nhìn thấy** trên ảnh. Không cố vẽ phỏng đoán phần cơ thể hoặc thân xe bị che khuất hoàn toàn phía sau vật cản hoặc nằm ngoài viền ảnh.
- **`R-02` — Người đi xe máy (Tách 2 box riêng biệt):**
  - Mọi trường hợp người điều khiển hoặc người ngồi sau xe máy đều phải tách thành 2 box riêng biệt:
    1. Một box `person` bao quanh người từ đỉnh mũ bảo hiểm xuống đến hết chân/tay người.
    2. Một box `motorcycle` bao quanh toàn bộ khung xe, bánh xe và gương xe.
  - Vùng giao nhau giữa thân người và yên xe để chồng lấn tự nhiên, không cố tình cắt bớt.
- **`R-03` — Xe chuyên dụng chở ô tô (Xe chở xe - `img_002.jpg`):**
  - Vẽ 1 box `truck` bao toàn bộ xe tải chuyên dụng (gồm cả đầu kéo và giàn chở hàng).
  - Vẽ thêm từng box `car` riêng biệt cho **từng chiếc ô tô con** được chở trên các tầng sàn (nếu ô tô đó nhận diện rõ hình dạng ô tô). Thuộc tính `occluded_level` của các xe con được đánh giá dựa trên mức độ bị các thanh giàn của xe tải che khuất.
- **`R-04` — Xe chở hàng cồng kềnh / quá khổ (`img_029.jpg`, `img_035.jpg`):**
  - Box phương tiện (`motorcycle`, `truck`, `tuk_tuk`) **chỉ bao quanh phần thân vỏ, khung gầm, bánh xe và kết cấu cơ khí cố định** của phương tiện.
  - Các khối hàng hóa rời, cồng kềnh chất trên nóc xe (như khối bao tải cao ngất ngưởng trên `img_035.jpg`) hoặc giỏ hàng quá khổ buộc phía sau xe máy (`img_029.jpg`) **KHÔNG được gộp vào box xe**.
  - Nếu khối hàng che khuất một phần người điều khiển hoặc xe, tăng chỉ số `occluded_level` tương ứng.
- **`R-05` — Không gán tượng, ma-nơ-canh, mô hình đồ chơi (Vật thể nhân tạo vô tri):**
  - **Tượng người / Tượng thú:** Tuyệt đối KHÔNG gán `person` cho tượng đồng Wellington (`img_017.jpg`) hay nhân vật hóa trang tượng (`img_011.jpg`).
  - **Tượng xe điêu khắc:** Tuyệt đối KHÔNG gán `truck` hay `car` cho tượng đài xe bán tải tạc bằng đá hoa cương (`img_027.jpg`).
  - **Ma-nơ-canh:** Tuyệt đối KHÔNG gán `person` cho ma-nơ-canh bằng nhựa/thạch cao trong tiệm thời trang (`img_033.jpg`).
  - **Xe đồ chơi / Búp bê trong tủ kính:** Tuyệt đối KHÔNG gán cho các mô hình đồ chơi thu nhỏ trong tủ trưng bày (`img_038.jpg`).
- **`R-06` — Không gán hình ảnh phản chiếu qua gương chiếu hậu (`img_020.jpg`):**
  - Hình ảnh xe cộ, người đi đường phản chiếu bên trong mặt gương chiếu hậu hoặc phản quang qua kính tòa nhà KHÔNG được gán nhãn, vì đó là ảo ảnh quang học, không phải chướng ngại vật vật lý tại vị trí không gian đó.
- **`R-07` — Không gán tranh vẽ, ảnh in 2D trên pano quảng cáo (`img_018.jpg`):**
  - Chân dung người hoặc hình ảnh phương tiện in 2D trên biển hiệu quảng cáo, áp phích tranh cử KHÔNG được gán nhãn `person` hay `car`.
- **`R-08` — Phương tiện không động cơ (Xe đạp, Xích lô):**
  - Xe đạp và xích lô không thuộc 5 lớp phương tiện cơ giới của taxonomy v0. Gán box `person` cho người điều khiển/người đạp xe. Bỏ qua khung xe xích lô/xe đạp (hoặc bao phủ bằng `ignore_region` nếu chắn lối đi phức tạp).
- **`R-09` — Ngưỡng vật thể nhỏ và Đám đông xa (`min_box_height_px: 20`):**
  - Các vật thể đơn lẻ có chiều cao $< 20\text{ px}$ (chiếu theo quy tắc kỹ thuật trong taxonomy) không gán box đơn lẻ.
  - Với các cụm người/xe đông đúc ở phía chân trời xa, sử dụng box `ignore_region` bao quanh cả cụm để mô hình không bị phạt khi không nhận diện từng đối tượng nhỏ.
- **`R-10` — Không gán xe chủ (Ego Vehicle):**
  - Bộ phận của xe chở camera (nắp capo, gương chiếu hậu mép dưới khung hình) xuất hiện ở viền ảnh KHÔNG được gán nhãn phương tiện tham gia giao thông.

---

## 4. Bảng phân tích các ca khó trong 40 ảnh

Bảng tổng hợp các tình huống biên thực tế phát hiện trong quá trình khảo sát 40 ảnh:

| Tên ảnh | Tình huống quan sát | Rule ID liên quan | Quyết định ban đầu (v0.1) | Lý do chi tiết |
|---|---|---|---|---|
| `img_002.jpg` | Xe tải chuyên dụng chở 8 ô tô con trên 2 tầng sàn. | `R-01`, `R-03` | Gán 1 box `truck` cho xe tải + Gán 8 box `car` cho 8 xe con trên sàn (đánh dấu `occluded_level=1` hoặc `2`). | Xe tải là phương tiện di chuyển chính; các xe con trên sàn có đầy đủ đặc trưng hình học của ô tô 2D, mô hình cần học nhận diện chúng để tránh nhầm lẫn. |
| `img_004.jpg` | Hai xe tuk-tuk 3 bánh đỗ cạnh lề đường. | `R-01`, đề xuất lớp mới | Tạm thời gán `motorcycle`, tick `can_review=true`. | v0 chưa có lớp `tuk_tuk`. Xe có 3 bánh chạy động cơ, gần với mô tô nhưng kích thước lớn; cần đưa ra Pha 2 họp nhóm bổ sung lớp `tuk_tuk`. |
| `img_006.jpg` | Đoàn xích lô du lịch chở khách ở Huế. | `R-02`, `R-08` | Gán box `person` cho người đạp xích lô và du khách ngồi trên xe. Không gán box phương tiện cho xích lô. | Xích lô chạy bằng sức người, không có động cơ, không thuộc lớp `motorcycle` hay `car`. |
| `img_011.jpg` | Nhân vật Predator hóa trang / tượng tại lễ hội mô tô. | `R-05` | KHÔNG gán nhãn `person` cho hình tượng Predator. Gán box `motorcycle` cho các mô tô thật xung quanh. | Đây là trang phục hóa trang/tượng vô tri giác, không phải người thật tham gia giao thông. |
| `img_012.jpg` | Xe bán tải chở đầy thùng hàng và bao tải hoa quả ở thùng sau. | `R-01`, `R-04` | Gán 1 box `truck`. Bounding box bao trọn cabin và thành thùng xe tải. | Xe bán tải có khoang chở hàng hở phía sau xếp vào `truck`. Hàng hóa nằm trong lòng thùng xe nên box xe bao trọn kết cấu thành thùng. |
| `img_017.jpg` | Tượng công tước Wellington cưỡi ngựa bằng đồng. | `R-05` | KHÔNG gán `person` cho tượng người cưỡi ngựa (ảnh 0 annotation). | Tượng điêu khắc bằng kim loại cố định, không phải đối tượng di chuyển tham gia giao thông. |
| `img_018.jpg` | Biển quảng cáo pano tấm lớn in chân dung ứng viên tranh cử. | `R-07` | KHÔNG gán `person` cho khuôn mặt trên biển quảng cáo (ảnh 0 annotation). | Hình ảnh in phẳng 2D trên pano ngoài trời, mô hình tự lái phải bỏ qua để tránh phanh nhầm. |
| `img_020.jpg` | Xe ô tô và cảnh vật phản chiếu qua gương chiếu hậu ô tô. | `R-06`, `R-10` | KHÔNG gán box cho xe trong gương (ảnh 0 annotation). | Hình ảnh phản chiếu không phải chướng ngại vật không gian phía trước. Không gán gương xe chủ theo `R-10`. |
| `img_022.jpg` | Mô tô cổ có gắn thùng phụ bên hông (sidecar 3 bánh). | `R-01`, `R-02` | Gán 1 box `motorcycle` bao trùm cả mô tô và thùng phụ sidecar + Gán từng box `person` cho người ngồi trên xe. | Thùng phụ gắn cố định vào kết cấu khung xe mô tô tạo thành một khối di chuyển thống nhất. |
| `img_027.jpg` | Đài tưởng niệm xe bán tải tạc hoàn toàn bằng đá hoa cương. | `R-05` | KHÔNG gán `truck` hay `car` cho khối tượng đá (ảnh 0 annotation). | Đây là công trình điêu khắc đá cố định trong nghĩa trang, không có động cơ hay khả năng lưu thông. |
| `img_029.jpg` | Xe máy chở khối hàng nguy hiểm cồng kềnh khổng lồ phía sau. | `R-02`, `R-04` | Gán 1 box `person` cho người lái; gán 1 box `motorcycle` chỉ ôm khung xe máy. KHÔNG kéo box bao trọn khối hàng khổng lồ. | Box phương tiện chỉ xác định thân xe để ước lượng kích thước gầm và trục bánh xe. Khối hàng cồng kềnh đánh dấu che khuất `occluded_level=2`. |
| `img_033.jpg` | Ma-nơ-canh mặc trang phục trưng bày sau cửa kính phố Buchanan. | `R-05` | KHÔNG gán `person` cho ma-nơ-canh (ảnh 0 annotation). | Ma-nơ-canh bằng nhựa/thạch cao vô tri giác, xe tự hành không được nhận nhầm là người đi bộ. |
| `img_035.jpg` | Xe auto rickshaw / tuk-tuk chở khối bao tải chất cao ngất ngưởng trên nóc. | `R-04`, đề xuất lớp mới | Gán box phương tiện ôm sát thân xe tuk-tuk (loại trừ khối bao tải phía trên). Tick `can_review=true`. | Ca điển hình của xung đột ranh giới: bao tải quá khổ chất rời trên nóc không thuộc thân xe. Cần chốt thống nhất tại Pha 2. |
| `img_037.jpg` | Xe bán tải gắn nắp thùng cao kín Master 75 và xe buýt phía sau. | `R-01`, đề xuất lớp mới | Gán xe bán tải nắp kín là `van` (hoặc `truck` tùy thảo luận Pha 2). Gán xe buýt là `bus` (đề xuất bổ sung lớp). Tick `can_review=true`. | Nắp thùng cao biến thùng bán tải thành khoang kín liền khối giống van. Xe buýt cần có lớp `bus` riêng biệt. |
| `img_038.jpg` | Tủ kính cửa hàng bày bộ sưu tập búp bê, đồ chơi và mô hình ô tô thu nhỏ. | `R-05` | KHÔNG gán `car` hay `person` cho bất kỳ mô hình đồ chơi nào (ảnh 0 annotation). | Đồ chơi trẻ em thu nhỏ nằm trong tủ trưng bày, không phải đối tượng thực trên đường phố. |

---

## 5. Tiêu chuẩn QC và Lịch sử phiên bản

### Tiêu chuẩn QC áp dụng (tham chiếu `taxonomy.json`):
- Kích thước box tối thiểu: Chiều cao $\ge 20\text{ px}$.
- Ngưỡng IoU đồng thuận giữa 2 người gán độc lập:
  - Box kích thước lớn ($\ge 50\text{ px}$): $\text{IoU} \ge 0.85$.
  - Box kích thước nhỏ ($< 50\text{ px}$): $\text{IoU} \ge 0.70$.
- Tỉ lệ đồng thuận phân lớp: $\ge 95\%$.
- Tỉ lệ ảnh lỗi tối đa toàn lô để nghiệm thu (Release): $\le 2\%$.

### Lịch sử thay đổi:

| Phiên bản | Ngày | Người thực hiện | Tóm tắt nội dung thay đổi | Cơ sở / Lý do |
|---|---|---|---|---|
| `v0` | 2026-11-16 | Giảng viên | Bản sơ lược ban đầu, chỉ có định nghĩa định tính 5 lớp. | Tài liệu khởi tạo của lab. |
| `v0.1` | 2026-11-16 | Thành viên 180 | Bổ sung định lượng ngưỡng % thuộc tính, quy tắc phân tách người đi xe máy (`R-02`), xe chở xe (`R-03`), hàng cồng kềnh (`R-04`), loại bỏ tượng/ma-nơ-canh/gương/quảng cáo (`R-05`–`R-07`), phân tích chi tiết 15 ca khó trong 40 ảnh. | Khảo sát thực tế toàn bộ 40 ảnh và giải quyết các điểm mơ hồ của v0. |

