# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 625; đối tượng sau duyệt: 620; ngưỡng accepted: IoU >= 0.9
- Đã **loại 21 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 568 | 568/625 = 90.9% | 625 box máy |
| edited (sửa) | 12 | 12/625 = 1.9% | 625 box máy |
| deleted (xoá) | 44 | 44/625 = 7.0% | 625 box máy |
| added (vẽ thêm) | 37 | 37/620 = 6.0% | 620 đối tượng sau duyệt |

Trong số deleted + added, **10 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 27.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 259 | 1 | 8 | 11 |
| bicycle | 8 | 0 | 2 | 1 |
| motorbike | 232 | 2 | 7 | 3 |
| car | 54 | 0 | 0 | 11 |
| bus | 2 | 0 | 1 | 3 |
| truck | 4 | 0 | 1 | 3 |
| three_wheeler | 9 | 9 | 25 | 5 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 54 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 48 | 23 | 6 | 67.6% (KTC 95%: 56–77%) | 88.9% (KTC 95%: 78–95%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 7/12 = 58.3% (KTC 95%: 32–81%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - wrong_class: 1/3
  - loose: 1/3
  - extra: 2/3
  - missing: 3/3
  - lọt: img_029.jpg (wrong_class), img_005.jpg (loose), img_033.jpg (loose), img_022.jpg (wrong_class), img_025.jpg (extra)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
