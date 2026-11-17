# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 5 ảnh (0 ca khó + 5 ngẫu nhiên), 116 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 5/116 = 4.3% | 116 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 3/5 = 60.0% | 5 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 568/625 = 90.9% / 12/625 = 1.9% / 44/625 = 7.0% | 625 box máy |
| Máy: added | 37/620 = 6.0% | 620 đối tượng sau duyệt |
| Bắt lỗi bẫy | 7/12 = 58.3% | 12 bẫy |

## Top 3 loại lỗi
| Loại lỗi | Số lỗi | Ảnh ví dụ |
|---|---|---|
| thua | 3 | img_006.jpg, img_031.jpg, img_035.jpg |
| sai_vi_tri_kich_thuoc | 2 | img_031.jpg |

**Lỗi hệ thống**: 'thua' lặp lại -> sửa quy tắc / recipe, không chỉ sửa từng ảnh.

## Giới hạn thống kê
- 3 ảnh lỗi trong 5 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **88%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**REWORK**

- ảnh có lỗi 60.0% > 2%
- lỗi hệ thống: 'thua' chiếm 3/5 lỗi
- bắt bẫy 58% < 80% -> lô duyệt chưa đáng tin

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
