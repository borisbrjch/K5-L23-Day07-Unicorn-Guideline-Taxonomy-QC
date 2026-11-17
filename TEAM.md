# Thông tin nhóm

- Tên nhóm: Unicorn
- Mã thành viên (ba chữ số): 143, 180
- Người điều phối: 143
- Người review bản cuối: 143, 180
- Commit chốt bài (điền ở kênh nộp sau khi commit): e8565c5

## Kết quả tóm tắt (điền cuối bài)

- Lượt gán hoàn tất / được giao: 80/80 lượt nộp (40/40 ảnh × 2 người: 143, 180)
- IAA trước calibration (kèm mẫu số): IoU TB 0.862, cùng lớp 96.1% trên 283 cặp box ghép (143|180) — đạt ngưỡng của lab (IoU ≥ 0.85, lớp ≥ 95%)
- Số ảnh phải calibration: 40 / 40
- Vòng QC và số ảnh đã duyệt / cả lô: Vòng 1, 40/40 ảnh
- Tỉ lệ lỗi: 0/40 ảnh lỗi (0.0%), 0/481 annotation lỗi (0.0%), 0 box can_review
- **Quyết định** (Release / Hold / Rework) và lý do một dòng: Release — đạt chuẩn IAA (IoU 0.862, lớp 96.1%), hoàn tất calibration 411 ca theo guideline v1.1, tắt hết can_review và 0% lỗi vòng 1; chi tiết trong submission/qc/QC_REPORT.md §5
