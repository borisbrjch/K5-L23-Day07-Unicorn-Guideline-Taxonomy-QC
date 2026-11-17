# Báo cáo QC — Unicorn

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Đặc tả và QC nhãn bounding box 2D người và phương tiện trên 40 ảnh đường phố (lô thực hành). Lô nhỏ, không đại diện cho mọi bối cảnh giao thông; **chưa dùng được để huấn luyện / đánh giá** vì lô đang Hold (xem §5).
- Nguồn và phép sử dụng dữ liệu: 40 ảnh Wikimedia Commons, giấy phép và tác giả ghi trong `data/ATTRIBUTION.csv` (CC BY / CC BY-SA, giữ ghi công).
- Manifest và checksum nhãn cuối: Chưa có. `labels_calibrated.json/.csv` và `batch_manifest.json` chưa sinh vì chưa có COCO sau sửa `submission/qc/labels_calibrated_00ee3899b65462ea.json` (bước sửa trên CVAT chưa làm).
- Guideline và taxonomy khi gán / khi bàn giao: Gán độc lập theo `guideline_v1.0_agreed.md` + `taxonomy.json` (15 lớp, `rules` giữ nguyên). Sau IAA nâng lên `guideline_v1.1_calibrated.md`, `guideline_change.json` kind `semantic`; taxonomy không đổi.

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 80/80 lượt (40 ảnh × 2 người: 143, 180) theo `assignment.csv`. **Lưu ý:** theo 180, nhãn `submission/labels/180.json` được **pre-label bằng AI rồi 180 chỉnh lại bằng tay**; 143 gán tay hoàn toàn. `RULES.md` ghi "Day 07 dùng người thật quyết định và vẽ box. Pre-label bằng mô hình thuộc Day 08", nên lượt của 180 là **lệch quy trình** (box và lớp khởi đầu từ đầu ra mô hình), dù có người chỉnh lại.
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 283 cặp ghép (IoU ≥ 0.5); IoU TB 0.836; cùng lớp 62.2%; cùng `occluded_level` 64.0%; cùng `truncated` 95.8% — mẫu số 283 cặp. CHƯA ĐẠT ngưỡng (IoU ≥ 0.85, lớp ≥ 95%). Vì nhãn 180 xuất phát từ pre-label AI, các số này đo đồng thuận giữa 143 và "AI + 180 chỉnh tay", **không hoàn toàn là hai người gán độc lập** — dùng để tìm ca bất đồng, không dùng làm bằng chứng đạt IAA (xem `iaa_summary.md`).
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: Chưa ghép 67 (143) / 133 (180); loại do ignore 0; bất đồng vùng ignore 5 ca. Số box chưa ghép phía 180 lớn hơn gấp đôi 143, có thể do box pre-label của AI chưa được lọc hết khi chỉnh tay.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: `img_035` (cả hai không gán `xe_3_banh` như ca khó v1.0; v1.1 chốt lại `tuk_tuk`, R-12); vật không thật `img_017`, `img_020`, `img_027`, `img_033`, `img_038` (R-02). Kiểm bằng crop trong `viz_disagreement.py` và đọc lại guideline §4.

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: 546/546 ca trong `calibration_decisions.csv` có quyết định (R-01: 194, R-03: 117, R-11: 109, R-08: 63, R-10: 34, R-12: 10, R-04: 7, R-06: 6, R-02: 5, R-09: 1); người chốt 143 (180 không tham gia calibration). Cột `kiem_lai` là **việc kiểm lại đã giao, chưa thực hiện**.
- Tác động v1.1 lên nhãn và ảnh: Semantic, phạm vi cả lô 40/40 ảnh (405 ca phạm vi `lo`, 141 ca phạm vi `anh`). Thay đổi chính: R-02 bỏ `ignore_region` cho vật không thật; R-08 người lái xe tạm dừng trong dòng xe = `moto_rider` (`img_021`); R-10 người đứng chờ = `static_person` (`img_032`); thêm R-11, R-12. Taxonomy không đổi nên không cần mapping lớp.
- Tham chiếu bằng chứng khi Rework: Chưa có vòng QC nào. Gói sửa đã chuẩn bị (`prepare_calibration.py`); chưa có COCO sau sửa.

## 4. Kết quả duyệt

- Vòng QC: chưa chạy (chưa có nhãn cuối để lấy mẫu)
- Người review và ngày: 143, 2026-11-17 (rà soát hồ sơ, chưa duyệt mẫu nhãn)
- Đã duyệt / cả lô: 0/40 ảnh — chưa có nhãn cuối
- Ảnh lỗi / ảnh đã duyệt: Chưa có (0 ảnh đã duyệt)
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: Chưa có (0 annotation đã duyệt)
- Vật bỏ sót, ảnh và vùng: Chưa đo trên nhãn cuối. Trước calibration: 200 ca `bo_sot_thua` trên 40 ảnh.
- Ca khó / phần còn lại, lỗi lặp và ví dụ: Lỗi lặp trước calibration: `moto_rider`↔`static_person` 34 ca (17 ở `img_021`), `dynamic_person`↔`static_person` 33 ca (18 ở `img_032`), `dynamic_person`↔`moto_rider` 12 ca.
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Máy chặn được: box cao < 20 px (`validate_labels.py` cảnh báo `img_021` #161, #163 của 143). Cần người đọc ảnh: người ngồi xe máy đang chờ đèn hay xe đã đỗ hẳn (`img_021`, R-08).

## 5. Quyết định bàn giao

- Quyết định: Hold
- Lý do, người nhận và phạm vi sử dụng: Thiếu đầu vào — chưa có nhãn sau calibration (`labels_calibrated_00ee3899b65462ea.json`) và chưa có vòng QC; ngoài ra lượt gán của 180 dùng pre-label AI (lệch `RULES.md`) nên IAA vòng 1 chỉ tham khảo. Không bàn giao cho người dùng nhãn; hồ sơ chỉ nộp làm bằng chứng cho coach.
- Nếu còn việc: (1) Nếu coach yêu cầu: 180 gán lại 40 ảnh hoàn toàn bằng tay theo v1.0, thay `submission/labels/180.json`, chạy lại `qc_compare.py` — người xử lý 180; 180 khai rõ cách dùng AI trong `individual_notes/180.md`. (2) Sửa nhãn trên CVAT theo v1.1 (có cả 143 và 180 kiểm lại), export `labels_calibrated_<hash>.json`, chạy `build_final.py` — người xử lý 143, 180. (3) Chạy `qc_sample.py` / `qc_report.py` vòng 1 và `check_submission.py` — người xử lý 143. Bằng chứng cần kiểm lại: `iaa_summary.json` mới, `calibration_decisions.csv`, `batch_manifest.json`, `submission/qc/reviews/round-1/`.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: Ví dụ ca `img_021` — `calibration_candidates.csv` (17 ca `sai_lop` moto_rider↔static_person) → `calibration_decisions.csv` (R-08, phạm vi `lo`) → `guideline_v1.1_calibrated.md` §3 R-08 và §4 ca `img_021` → nhãn gốc `submission/labels/143.json`, `180.json`. Nhãn cuối chưa có nên chưa kiểm được tới `labels_calibrated.json`.
