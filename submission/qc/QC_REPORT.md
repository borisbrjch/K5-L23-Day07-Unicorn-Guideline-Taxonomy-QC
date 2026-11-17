# Báo cáo QC — Unicorn

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Đặc tả và QC nhãn bounding box 2D người và phương tiện trên 40 ảnh đường phố (lô thực hành). Bộ dữ liệu đã hoàn tất hiệu chuẩn calibration và nghiệm thu QC vòng 1 đạt chuẩn Release, sẵn sàng bàn giao cho kỹ sư huấn luyện mô hình phát hiện vật thể (Object Detection: YOLO / Faster R-CNN) và làm tập dữ liệu đánh giá chuẩn.
- Nguồn và phép sử dụng dữ liệu: 40 ảnh Wikimedia Commons, giấy phép và tác giả ghi trong `data/ATTRIBUTION.csv` (CC BY / CC BY-SA, giữ ghi công).
- Manifest và checksum nhãn cuối: `batch_manifest.json` (SHA256: `83af106f9fed034d18ad6d419dd2d40e36a9aa42cc8f53cce782a592a817c2a8`), `labels_calibrated.json` (SHA256: `c13e37b5db58c3c6bb3395ee13cd6c3495143771b1e912aef37203dc37ca925d`), `labels_calibrated.csv` (SHA256: `39f1097b222d60d63b76696a5a23ee22973223783ff57b18c5d08fe50edc55d8`). Khởi nguồn từ bản COCO hiệu chuẩn `labels_calibrated_d47fd78e5b9fb00e.json` (SHA256: `0a4d51d84da9b4f1745711e92fcd650a5677c788aa4b9dbf08dd1dd0cdf3fa21`).
- Guideline và taxonomy khi gán / khi bàn giao: Gán độc lập theo `guideline_v1.0_agreed.md` + `taxonomy.json` (15 lớp, `rules` giữ nguyên). Sau IAA nâng lên `guideline_v1.1_calibrated.md`, `guideline_change.json` kind `semantic`; taxonomy không đổi.

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 80/80 lượt (40 ảnh × 2 người: 143, 180) theo `assignment.csv`.
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 283 cặp ghép (IoU ≥ 0.5); IoU TB 0.862; cùng lớp 96.1%; cùng `occluded_level` 65.0%; cùng `truncated` 95.8% — mẫu số 283 cặp. ĐẠT NGƯỠNG IAA của lab (IoU ≥ 0.85, lớp ≥ 95%).
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: Chưa ghép 67 (143) / 133 (180); loại do ignore 0; bất đồng vùng ignore 5 ca.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: `img_035` (cả hai không gán `xe_3_banh` như ca khó v1.0; v1.1 chốt lại `tuk_tuk`, R-12); vật không thật `img_017`, `img_020`, `img_027`, `img_033`, `img_038` (R-02). Kiểm bằng crop trong `viz_disagreement.py` và đọc lại guideline §4.

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: 411/411 ca trong `calibration_decisions.csv` có quyết định (R-01: 200 ca `bo_sot_thua`, R-03: 81 ca `iou_thap`, R-11: 105 ca `sai_thuoc_tinh`, R-08/R-10/R-12: 11 ca `sai_lop`, R-02: 5 ca `bat_dong_ignore`, 9 ca `co_can_review`); người chốt 143, 180. Cột `kiem_lai` đã được hai thành viên rà soát và xác nhận đồng thuận.
- Tác động v1.1 lên nhãn và ảnh: Semantic, phạm vi cả lô 40/40 ảnh (330 ca phạm vi `lo`, 81 ca phạm vi `anh`). Thay đổi chính: R-02 bỏ `ignore_region` cho vật không thật; R-08 người lái xe tạm dừng trong dòng xe = `moto_rider` (`img_021`); R-10 người đứng chờ = `static_person` (`img_032`); thêm R-11 (occluded_level), R-12 (tuk_tuk, xe bán tải, xích lô). Taxonomy không đổi nên không cần mapping lớp.
- Tham chiếu bằng chứng khi Rework: Đạt chất lượng ngay vòng 1, không phát sinh vòng Rework.

## 4. Kết quả duyệt

- Vòng QC: 1
- Người review và ngày: 143, 2026-11-17 (duyệt toàn bộ lô 40 ảnh)
- Đã duyệt / cả lô: 40/40 ảnh
- Ảnh lỗi / ảnh đã duyệt: 0/40 = 0.0%
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: 0/481 = 0.0%
- Vật bỏ sót, ảnh và vùng: 0 ca bỏ sót trên nhãn cuối (đối chiếu phiếu review vòng 1). Trước calibration: 200 ca `bo_sot_thua` trên 40 ảnh.
- Ca khó / phần còn lại, lỗi lặp và ví dụ: Lỗi lặp trước calibration: `moto_rider`↔`static_person` 3 ca (ở `img_021`), `static_person`↔`dynamic_person` 4 ca (ở `img_032`), `truck`↔`van` 1 ca (ở `img_016`), `tuk_tuk`↔`side_car` 1 ca (ở `img_035`). Toàn bộ đã được chuẩn hóa dứt điểm trên nhãn cuối.
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Máy chặn được: box cao < 20 px (`validate_labels.py` cảnh báo `img_021` #161, #163 của 143). Cần người đọc ảnh: người ngồi xe máy đang chờ đèn hay xe đã đỗ hẳn (`img_021`, R-08).

## 5. Quyết định bàn giao

- Quyết định: Release
- Lý do, người nhận và phạm vi sử dụng: Lô thực hành đạt đầy đủ các tiêu chuẩn chất lượng của lab: (1) Chỉ số đồng thuận IAA đạt yêu cầu với IoU TB 0.862 và đồng thuận lớp 96.1%; (2) Toàn bộ 411 ca bất đồng đã được calibration và xử lý dứt điểm theo guideline v1.1; (3) Tất cả các cờ `can_review` đã được xử lý và tắt trên nhãn cuối; (4) Vòng 1 QC duyệt toàn bộ 40/40 ảnh đạt 0% ảnh lỗi (0/481 annotation lỗi). Bàn giao chính thức cho kỹ sư huấn luyện mô hình Object Detection và lưu trữ làm ground truth chuẩn.
- Nếu còn việc: Không còn việc tồn đọng kỹ thuật. Bộ nhãn và hồ sơ bàn giao hoàn chỉnh.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: Ví dụ ca `img_021` — `calibration_candidates.csv` (3 ca `sai_lop` moto_rider↔static_person) → `calibration_decisions.csv` (R-08, phạm vi `lo`) → `guideline_v1.1_calibrated.md` §3 R-08 và §4 ca `img_021` → nhãn gốc `submission/labels/143.json`, `180.json` → nhãn cuối `submission/qc/labels_calibrated.json` (người ngồi xe máy đã được chuẩn hóa lớp `moto_rider`).
