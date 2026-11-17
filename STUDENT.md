# Thông tin học viên

- Họ và tên: Lê Tiến Bình
- MSHV: 2A202603346
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: Cily49
- Repo bài nộp (URL): https://github.com/Cily49/K5-L2-Day08-LeTienBinh-2A202603346-AI-Assisted-Labeling
- Commit chốt bài (hash): <<>>
- Máy: macOS (Apple Silicon, RAM 24 GB), GPU Apple MPS
- CVAT (Docker): v2.82.1 (image dev; v2.81.0 không có bản arm64)

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 568 / 12 / 44 = 90.9% / 1.9% / 7.0% (mẫu số: 625 box máy); added: 37 = 6.0% (mẫu số: 620 đối tượng)
- So với gold: precision / recall của **máy** 67.6% (48/71) / 88.9% (48/54)
- Bắt bẫy: 7/12 = 58.3%
- Thời gian: gán tay 7.5 phút/ảnh (30/4); duyệt có pre-label 0.83 phút/ảnh (20/24)
- Lỗi QC: 60% theo ảnh (3/5), 4.3% theo đối tượng (5/116)
- **Quyết định:** Rework — lý do một dòng: ảnh lỗi 3/5 > 2%, lỗi box trùng lặp lại, bắt bẫy 58% < 80%
