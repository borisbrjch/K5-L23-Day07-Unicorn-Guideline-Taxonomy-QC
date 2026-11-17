# Thông tin nhóm

- Tên nhóm: Unicorn
- Mã thành viên (ba chữ số): 143, 180
- Người điều phối: 143
- Người review bản cuối: 143
- Commit chốt bài (điền ở kênh nộp sau khi commit): f4d8971

## Kết quả tóm tắt (điền cuối bài)

- Lượt gán hoàn tất / được giao: 80/80 lượt nộp; 143 gán tay 40/40, 180 pre-label bằng AI rồi chỉnh tay 40/40 (lệch RULES.md: pre-label thuộc Day 08)
- IAA trước calibration (kèm mẫu số): IoU TB 0.836, cùng lớp 62.2% trên 283 cặp box ghép (143|180) — chưa đạt, chỉ tham khảo vì nhãn 180 xuất phát từ pre-label AI
- Số ảnh phải calibration: 40 / 40
- Vòng QC và số ảnh đã duyệt / cả lô: Vòng 1, 40/40 ảnh
- Tỉ lệ lỗi: 0/40 ảnh lỗi (0.0%), 0/481 annotation lỗi (0.0%), còn 8 box can_review
- **Quyết định** (Release / Hold / Rework) và lý do một dòng: Hold — nhãn cuối còn 8 box can_review và lượt 180 dùng AI pre-labeling; chi tiết trong submission/qc/QC_REPORT.md §5
