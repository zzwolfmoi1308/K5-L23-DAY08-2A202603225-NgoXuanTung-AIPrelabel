# Thông tin học viên

- Họ và tên: Ngô Xuân Tùng
- MSHV: 2A202603225
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: zzwolfmoi1308
- Repo bài nộp (URL): https://github.com/zzwolfmoi1308/K5-L23-DAY08-2A202603225-NgoXuanTung-AIPrelabel
- Commit chốt bài (hash): d9d322e
- Máy: Windows 11, không GPU (CPU)
- CVAT (Docker): v2.74.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 191 / 94 / 63 (mẫu số: 348 box máy); added: 141 (mẫu số: 426 đối tượng)
- So với gold: precision / recall của **máy** Precision 85.7% (36/42) / Recall 65.5% (36/55)
- Bắt bẫy: 7/12 = 58.3%
- Thời gian: gán tay 5.5 phút/ảnh; duyệt có pre-label 3.13 phút/ảnh
- Lỗi QC: 40.0% theo ảnh (2/5), 2.5% theo đối tượng (2/81)
- **Quyết định:** Rework — lý do một dòng: Tỉ lệ ảnh lỗi QC 40.0% > 2% và bắt bẫy 58.3% < 80% do lọt bẫy viền lỏng, cần rà soát lại trước khi phát hành chính thức
