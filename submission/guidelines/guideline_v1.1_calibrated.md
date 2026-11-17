# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603346 - Lê Tiến Bình

> Chỉ ghi phần thay đổi so với v1.0. Mỗi quy tắc khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 3 | Hàng nhô ra: box / mask xe có gồm hàng không? | Hàng **nằm trong thùng hoặc giỏ gắn cố định** thuộc xe (box và mask). Hàng nhô ra ngoài thân xe > 20% chiều dài xe thì **không** tính vào xe (giữ mục 4.1.5, áp dụng cả cho mask). | img_003 (trái cây trong thùng ba gác), img_031 (giỏ trên xe đạp) | CAL-04 |
| 4 | Người ngồi trong / trên xe ba bánh có thuộc mask xe không? | **Không.** Người lái, khách, trẻ em trên xe là vật riêng: box `person` riêng nếu nhìn thấy ≥ 20%. Mask / polygon của xe **loại phần người** (mục 4.2.4: vùng chạm thuộc vật phía trước). | img_013 (tuk-tuk có người lái), img_010 (xích lô có người đạp), img_003 (trẻ em ngồi trên ba gác) | CAL-03 |
| 6 | Box máy chồng nhau nhiều: gộp hay giữ? | Mỗi vật **đúng một box**. Hàng xe đỗ sát nhau ở xa mà box cao < 20 px thì vẽ **một** `ignore_region` (mục 4.1.6). | img_006 (hàng xe máy đỗ), img_035 (một người hai box) | CAL-05 |

Câu **chưa chốt**: 1 (xe đạp điện), 2 (khối xe máy dày gần camera), 5 (kính xe), 7 (ngưỡng conf tối thiểu). Lý do: trong 24 + 4 ảnh chưa gặp ca đủ rõ để làm ví dụ, hoặc (câu 7) conf chỉ là điểm của máy, không phải quy tắc chất lượng.

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_002, img_011, img_032, img_013 | Tuk-tuk / xích lô có mui | YOLO26-m gọi là `car` / `truck`; nâng ngưỡng không sửa được | Gán `three_wheeler` | **Mục 2 + 4.3:** xe có mui hoặc thùng hàng mà đếm được 3 bánh là `three_wheeler`, không bao giờ là `car` / `truck` (CAL-01) |
| img_003, img_008, img_010, img_017 | Một xe ba bánh có 2–5 box lồng nhau | YOLOE-26s (prompt `three_wheeler`) sinh box cho mui, thân, cả cụm | Giữ 1 box ôm mui + thân + bánh | **Mục 4.1, quy tắc 8 (mới):** một vật một box; box nằm > 60% trong box khác cùng lớp là box trùng, xoá (CAL-02) |
| img_013, img_010 | Mask SAM của xe gộp cả người lái | SAM 2.1 (click) | Thêm click âm lên người; mask xe không gồm người | **Mục 4.2, quy tắc 5 (mới):** mask xe không gồm người ngồi trên / trong xe (CAL-03) |
| img_022, img_029, img_005, img_033 | Bẫy sai lớp / box lỏng bị lọt | (bẫy cài sẵn) | Lỗi người duyệt, guideline không mơ hồ | **Không đổi guideline**; siết quy trình duyệt (CAL-06) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu hỏi mở 3, 4, 6; thêm quy tắc 4.1.8 (một vật một box) và 4.2.5 (mask xe không gồm người); làm rõ mục 2 / 4.3 cho `three_wheeler` có mui | CAL-01 … CAL-05 (CAL-06 không đổi guideline) |
