# Thông tin nhóm

- Tên nhóm: Unicorn
- Mã thành viên (ba chữ số): 143, 180
- Người điều phối: 143
- Người review bản cuối: 143
- Commit chốt bài (điền ở kênh nộp sau khi commit):

## Kết quả tóm tắt (điền cuối bài)

- Lượt gán hoàn tất / được giao: 80/80 lượt nộp; 143 gán tay 40/40, 180 pre-label bằng AI rồi chỉnh tay 40/40 (lệch RULES.md: pre-label thuộc Day 08)
- IAA trước calibration (kèm mẫu số): IoU TB 0.836, cùng lớp 62.2% trên 283 cặp box ghép (143|180) — chưa đạt, chỉ tham khảo vì nhãn 180 xuất phát từ pre-label AI
- Số ảnh phải calibration: 40 / 40
- Vòng QC và số ảnh đã duyệt / cả lô: chưa chạy vòng QC, 0/40 (chưa có nhãn sau calibration)
- Tỉ lệ lỗi: chưa đo — chưa có vòng QC
- **Quyết định** (Release / Hold / Rework) và lý do một dòng: Hold — chưa có nhãn sau calibration và vòng QC; lượt 180 dùng pre-label AI; chi tiết và người xử lý trong `submission/qc/QC_REPORT.md` §5
