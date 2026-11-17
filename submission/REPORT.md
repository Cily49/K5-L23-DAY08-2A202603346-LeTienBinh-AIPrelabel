# Báo cáo Day 08 — 2A202603346 - Lê Tiến Bình

> Một file duy nhất cho cả bài. Mọi con số kèm **mẫu số** và nguồn. Phần chữ của báo cáo được soạn với sự hỗ trợ của Claude (xem mục 8).

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | bicycle 0.2, motorbike 0.1; person / car / bus / truck dùng mặc định 0.3 | r01 / r02: bắt đủ người, xe máy gần camera; nâng 0.25 → 0.5 làm mất 215 → 139 box, chủ yếu `person` thật (img_007, img_015), nên không dùng một ngưỡng cao chung |
| three_wheeler | YOLOE-26s-seg, prompt `three_wheeler` | 0.15 | r04: tìm đúng tuk-tuk chính ở img_002 / img_032 (conf 0.93 / 0.92) mà YOLO26 gọi là car / truck |

- Lệnh merge đã dùng:
  `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "bicycle=0.2,motorbike=0.1,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **SAM 2.1 base (`sam2.1_b.pt`)**: bản khuyến nghị, cân bằng chất lượng / tốc độ; chạy trên Apple MPS ~2.4 s/ảnh (r03), RAM 24 GB đủ. Không dùng SAM 3 vì không cần quyền truy cập gated cho bài chính.
- **Lỗi không sửa được bằng ngưỡng**: tuk-tuk / xích lô bị YOLO26 gọi là `car` / `truck` (img_002, img_011, img_032; r01, r02). Đây là lỗi **tên lớp** của detector COCO (không có lớp xe ba bánh), không phải lỗi điểm. Cách sửa: dùng model open-vocabulary YOLOE cho riêng `three_wheeler` và cho YOLOE thắng khi trùng box (merge, IoU 0.6).
- **Điểm cao mà sai / thấp mà đúng**: img_003 có hai box `three_wheeler` 0.90 và 0.80 trên cùng một xe ba gác (một trong hai là box trùng; sau duyệt chỉ còn 1 box). Ngược lại `three_wheeler` 0.38 ở img_035 (xích lô ở xa) là đúng.
- **Giới hạn**: ngưỡng chọn và chấm trên **cùng 4 ảnh gold** (54 vật, không tính ignore_region) nên precision / recall của máy trên gold là **lạc quan**; bus (1 vật) và truck (0 vật) quá ít nên giữ mặc định 0.3; person và car không ngưỡng nào ≤ 0.5 đạt recall 90% (`qc/threshold_sweep.md`).

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 568 | 90.9% | 625 box máy |
| edited | 12 | 1.9% | 625 box máy |
| deleted | 44 | 7.0% | 625 box máy |
| added | 37 | 6.0% | 620 đối tượng sau duyệt |

(Đã loại 21 đối tượng quanh vùng bẫy: lỗi nhân tạo không phản ánh máy. Trong deleted + added có 10 ca **đổi lớp**, nên số vật máy sót thật ≈ 37 − 10 = 27.)

- **Lớp tệ nhất: `three_wheeler`**: chỉ 9 accepted / 43 box máy (9 accepted, 9 edited, 25 deleted), thêm 5 box sót. Nguyên nhân: YOLOE sinh nhiều box lồng nhau cho cùng một xe (img_003 có 8 box `three_wheeler` trước khi duyệt, img_008 / img_010 có 4). Lớp **`car`** bị sót nhiều nhất trong các lớp xe: 11 added, so với 54 box máy được giữ (0 deleted), tức máy gần như không thừa car nhưng sót car.
- **So với gold**: precision **67.6%** (48/71 box máy, KTC 95% 56–77%), recall **88.9%** (48/54 vật gold, KTC 95% 78–95%). Máy thừa box nhiều hơn là sót.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy **7/12 = 58.3%** (KTC 95%: 32–81%), dưới ngưỡng 80%. Theo loại: missing 3/3, extra 2/3, wrong_class 1/3, loose 1/3. Lọt: img_029 và img_022 (wrong_class), img_005 và img_033 (loose), img_025 (extra).
- Lọt chủ yếu ở **bước 3 (sai lớp) và bước 4 (box lỏng)** của checklist. Tôi duyệt 24 ảnh trong 20 phút (0.83 phút/ảnh, khuyến nghị ~3 phút/ảnh): đủ thời gian để quét tìm vật bị sót (bước 1, bắt 3/3) nhưng không phóng to kiểm lớp và viền từng box. img_005 và img_029 nằm trong nhóm 6 ảnh dày xe máy, nơi box lỏng / sai lớp dễ chìm giữa hàng chục box đúng.
- **Accepted 90.9% mà bắt bẫy 58.3%**: con số accepted cao không chứng minh máy tốt, một phần là do tôi bấm giữ những box trông hợp lý (thiên kiến tự động hoá). 5/12 lỗi cài sẵn được "accept"; nếu tỉ lệ đó đúng cho lỗi thật của máy thì trong 568 box accepted vẫn còn lỗi chưa được sửa. QC (mục 7) xác nhận điều này: 3/5 ảnh mẫu còn lỗi.

## 4. Máy có đáng dùng không?

- Thời gian (`time_log.csv`): gán tay **30/4 = 7.5 phút/ảnh** vs duyệt **20/24 = 0.83 phút/ảnh** (nhanh hơn ~9 lần). Thiên lệch: gold gán trước nên lúc duyệt đã quen cảnh; và tốc độ duyệt nhanh một phần vì duyệt **chưa kỹ** (bắt bẫy 58%), nên mức tiết kiệm thật nhỏ hơn con số 9 lần.
- **Kết luận: có, nhưng theo lớp.** Đáng dùng cho `person` (259/268 box máy accepted) và `motorbike` (232/241 accepted). **Không** nên tin cho `three_wheeler` (9/43 accepted, phần lớn box trùng) và cần kiểm kỹ `car` (11 vật sót). Kể cả khi duyệt lại ở tốc độ khuyến nghị 3 phút/ảnh, pre-label vẫn nhanh hơn gán tay (7.5 phút/ảnh).
- **Ba quy tắc giảm thiên kiến**:
  1. **Thời gian tối thiểu**: mỗi ảnh ≥ 2 phút (ảnh dày ≥ 4 phút), ghi vào `time_log`. Kiểm: `batch_review` ≥ 48 phút cho 24 ảnh.
  2. **Kiểm lớp có chủ đích**: với mọi box `car`, `truck`, `bus`, `three_wheeler`, phóng to và đếm bánh / nhìn mui trước khi giữ. Kiểm: lần sau bẫy `wrong_class` bắt 3/3.
  3. **Kiểm viền theo bộ lọc**: sau khi duyệt xong một ảnh, lọc box theo lớp, bấm `Tab` qua từng box và nhìn 4 cạnh, chưa kiểm hết thì chưa sang ảnh khác. Kiểm: bẫy `loose` bắt 3/3 và QC `sai_vi_tri_kich_thuoc` = 0 trên mẫu.

## 5. Mask SAM cho 3 vật (Part 5)

Tất cả dùng **SAM 2.1 base**, prompt bằng click (`scripts/sam2_points.py`).

| Ảnh | SAM | Số click (lượt) | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_003 (xe ba gác chở trái cây) | 2.1 b | 1 → **5** (3 dương + 2 âm) | Lượt 1 chỉ khoanh **đống trái cây** (6.1k px, 2% box xe). Lượt 2 phủ cả mui, thùng, bánh, hàng trong thùng; **trẻ em ngồi trên xe vẫn bị gộp** dù có click âm lên người | Click rơi vào hàng hoá thì SAM hiểu "vật" là hàng hoá; trẻ em ngồi sát thùng khó tách | `tuktuk_img003.jpg`, `tuktuk_img003_v2.jpg` |
| img_013 (tuk-tuk Thái) | 2.1 b | 3 → 6 → **5** | Lượt 1 (2 dương + 1 âm) phủ đủ xe nhưng **gồm người lái** và một phần người phía sau. Lượt 2 (thêm 3 click âm bên trong xe) **hỏng**: mask gần như biến mất. Lượt 3 (3 dương + 2 âm trên người lái) phủ đủ xe, vẫn dính một phần người lái | Thêm click âm bên trong vật làm mask sụp; người ngồi bên trong khung xe rất khó tách bằng click | `tuktuk_img013.jpg`, `_v2`, `_v3` |
| img_010 (xích lô) | 2.1 b | 2 → 4 → **6** (4 dương + 2 âm) | Lượt 1–2 đặt click trên lưng người đạp nên mask là **người đạp**, không phải xe. Lượt 3 (click trên mui, khung, 2 bánh trước, âm trên người) khoanh đúng xích lô và **loại được người đạp**; mask tách 3 polygon vì khung mảnh | Khung kim loại mảnh làm mask rời rạc; click sai vật thì đổi hẳn vật | `tuktuk_img010.jpg`, `_v2`, `cyclo_img010_v3.jpg` |

- **Đối chiếu guideline 4.2**: quy tắc 4.2.4 (hai vật chạm nhau, không chồng polygon) giúp quyết định người lái **không** thuộc mask xe, nhưng v1.0 không nói rõ ca người ngồi **bên trong** xe ba bánh (câu hỏi mở 4), nên đã thêm quy tắc 4.2.5 ở v1.1 (CAL-03). Quy tắc 4.1.5 về hàng nhô ra chỉ nói box, nên v1.1 áp dụng cả cho mask (CAL-04). Rút ra: **click phải đặt trên phần chỉ thuộc xe** (mui, bánh, khung), và mask đúng hình không cứu được nếu chọn sai vật.
- Interactor SAM trong CVAT: chưa thử.

## 6. Calibration (Part 6)

- **6 quyết định** (`qc/calibration_decisions.csv`): 2 `loi_may`, 3 `cau_hoi_mo`, 1 `bay_lot`. Quan trọng nhất: **CAL-02** (một vật một box), vì box trùng là lỗi lặp lớn nhất của máy (`three_wheeler` 25 deleted) và vẫn còn sót trong nhãn sau duyệt (QC: 3/5 lỗi là `thua`).
- Quy tắc mới trong `guideline_v1.1_calibrated.md`: mục 2 / 4.3 xe 3 bánh có mui là `three_wheeler` (CAL-01); 4.1.8 một vật một box (CAL-02); 4.2.5 mask xe không gồm người (CAL-03); chốt câu hỏi mở 3, 4, 6.
- **Nguyên nhân gốc**: CAL-01 và CAL-02 là **máy** (COCO thiếu lớp, YOLOE sinh box lồng); CAL-03 / 04 / 05 là **guideline mơ hồ**; CAL-06 là **tôi** (duyệt quá nhanh), nên không đổi guideline mà đổi quy trình (mục 4).

## 7. QC và quyết định (Part 6)

- Mẫu: **5 ảnh** (0 ảnh `can_review` vì tôi không tick box nào + 5 ngẫu nhiên, seed 42): img_003, img_006, img_014, img_031, img_035; 116 đối tượng. Người kiểm: Claude xem ảnh `work/qc/review/` và điền phiếu (**hạn chế**: không phải người kiểm độc lập thứ hai; xem mục 8).
- Lỗi theo **đối tượng**: **5/116 = 4.3%** · theo **ảnh**: **3/5 = 60%** (ngưỡng Release ≤ 2%).
- Top lỗi: **thừa** 3 (box trùng: img_006 #78 trong hàng xe máy đỗ, img_031 #537 xe đạp tách 2 box, img_035 #629 người có 2 box); **sai vị trí / kích thước** 2 (img_031: box xe đạp #534 thiếu bánh sau và giỏ, box người #535 cắt mất chân).
- **Giới hạn mẫu nhỏ**: 3/5 ảnh lỗi cho cận trên 95% (Wilson) là **88%**; với 5 ảnh, kể cả 0 lỗi cũng không chứng minh được "≤ 2%". Mẫu chỉ đủ để **bác bỏ** Release, không đủ để ước lượng chính xác tỉ lệ lỗi.
- Script kiểm được: đếm, mẫu số, khoảng tin cậy, box < 20 px (`validate_labels`), bẫy. Script **không** kiểm được: box có ôm sát không, vật mà người duyệt và máy **cùng bỏ sót**, người duyệt có thật sự theo checklist không (chỉ đo gián tiếp qua bẫy và thời gian).

**Quyết định: Rework**, ngày 2026-11-17. Lý do: Hold? Không. Guideline đã chốt các câu hỏi gây lỗi (v1.1) và dữ liệu không hỏng. Rework? **Có**: ảnh lỗi 3/5 = 60% > 2%; lỗi lặp có hệ thống (`thua` 3/5 lỗi); bắt bẫy 7/12 = 58% < 80% nên lô duyệt chưa đáng tin.
Việc cần làm tiếp: (1) duyệt lại cả 24 ảnh theo v1.1 với quy tắc ở mục 4 (≥ 2 phút/ảnh), ưu tiên xoá box trùng (4.1.8) và kiểm lớp `car` / `three_wheeler`; (2) sửa ngay các lỗi QC ở img_006, img_031, img_035 (export `labels/qc_fixed.json`); (3) QC lại với mẫu lớn hơn (≥ 10 ảnh) và một người kiểm khác.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài:** Claude Code (Claude Opus) — cài môi trường (dùng lại venv có sẵn, Python 3.15 + torch CUDA), chạy các script Part 2–6 (`prelabel`, `sam2_points`, `threshold_sweep`, `merge_prelabels`, `inject_traps`, `review_stats`, `build_release`, `qc_sample`, `qc_report`), tạo Project/Task CVAT, import / export nhãn qua CVAT, nháp nhận xét `model_log.csv` từ ảnh `viz/`, nháp CAL-02/CAL-03, điền số liệu vào REPORT và diễn đạt thành câu các ý mình trả lời (nguyên nhân bẫy lọt, quyết định Rework, 3 quy tắc, máy có đáng dùng). **Không** dùng AI để gán gold hay review batch (tự làm trong CVAT). Cách kiểm tra: so mọi con số với `review_stats.md`, `threshold_sweep.md`, `qc_report_auto.md`; mở ảnh `masks/` và `viz/` đối chiếu nhận xét.
- Gold có bị nhiễm không? Không: model chỉ chạy lên ảnh gold (`--split all`) sau khi đã export `gold_manual.json`.
- Máy: macOS, Apple Silicon, RAM 24 GB, GPU Apple MPS. CVAT chạy Docker **v2.82.1** (image `v2.81.0` không có bản arm64 nên dùng image `dev` sẵn có). Vướng mắc: máy chỉ có Python 3.14 nên dùng `uv` tạo venv 3.12; YOLOE lỗi thiếu `clip` (venv uv không có pip), phải cài tay `git+https://github.com/ultralytics/CLIP.git`; Task `batch-review` lúc đầu tạo ngoài Project nên import báo "Label 'motorbike' is not registered", đã chuyển Task vào Project.
- Dùng cache `data/cache/` thay vì tự chạy? Không (chỉ đọc cache YOLOE để tìm ứng viên xe ba bánh cho Part 5).
- Lỗi / bug script đã gặp: không sửa script. YOLOE lần đầu tự cài CLIP (34 s); cảnh báo `torch.jit.load` trên Python 3.14+ không ảnh hưởng kết quả.
- Một điều làm khác lần sau: theo đủ 4 bước checklist và dành ~2–3 phút/ảnh thay vì 0.83 phút/ảnh.

