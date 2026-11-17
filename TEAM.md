# Thông tin nhóm

- Tên nhóm: Unicorn
- Mã thành viên (ba chữ số): 143, 180
- Người điều phối: 143
- Người review bản cuối: 143
- Commit chốt bài (điền ở kênh nộp sau khi commit):

## Kết quả tóm tắt (điền cuối bài)

- Lượt gán hoàn tất / được giao: 80/80 lượt nộp; chỉ 40/80 (của 143) là gán thủ công hợp lệ — 180 dùng AI để gán, trái RULES.md
- IAA trước calibration (kèm mẫu số): IoU TB 0.836, cùng lớp 62.2% trên 283 cặp box ghép (143|180) — chưa đạt, chỉ tham khảo vì nhãn 180 do AI gán
- Số ảnh phải calibration: 40 / 40
- Vòng QC và số ảnh đã duyệt / cả lô: chưa chạy vòng QC, 0/40 (chưa có nhãn sau calibration)
- Tỉ lệ lỗi: chưa đo — chưa có vòng QC
- **Quyết định** (Release / Hold / Rework) và lý do một dòng: Hold — thiếu lượt gán thủ công của 180 và nhãn sau calibration; chi tiết và người xử lý trong `submission/qc/QC_REPORT.md` §5
