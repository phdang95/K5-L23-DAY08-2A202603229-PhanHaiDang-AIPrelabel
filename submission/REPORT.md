# Báo cáo Day 08 — 2A202603229 - Phan Hải Đăng

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào).

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | `yolo26-m` | `0.3` (mặc định) | `r01`, `r02`, `r05`: `person`/`motorbike`/`car` không đạt recall 90% ở ngưỡng ≤ 0.5 (tối đa 62%/78%/80%); `bicycle` (3 vật), `bus` (0), `truck` (0) có `< 5` vật trong gold nên giữ mặc định `0.3` theo `threshold_sweep.md`. |
| three_wheeler | `yoloe-26s-seg` (prompt: `tuk-tuk`, `cyclo`, `cargo tricycle`) | `0.15` | `r03`, `r06`: đạt recall `100%` và precision `78%` trên `7` vật gold (`threshold_sweep.md`). |

- Lệnh `merge_prelabels.py` đã dùng:
  `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM đã chọn:** `sam2.1_b.pt` (SAM 2.1 base) vì cân bằng tốt giữa chất lượng biên mask và tốc độ chạy trên Colab, không cần xin quyền gated repo như SAM 3 (`facebook/sam3`).
- **Lỗi không sửa được bằng đổi ngưỡng:** YOLO26 là detector closed-set 80 lớp COCO nên gọi nhầm tuk-tuk / xe ba gác thành `truck` (`img_011`) hoặc `car` (`img_013`). Cách sửa: dùng model open-vocabulary **YOLOE** cho lớp `three_wheeler` rồi gộp ưu tiên thắng `car`/`truck` khi IoU ≥ 0.6.
- **Box điểm cao mà sai / điểm thấp mà đúng:** Trên `img_011`, YOLO26 gán nhầm tuk-tuk mép trái thành `truck 0.77` (điểm cao nhưng sai lớp); trên `img_002`, YOLOE bắt đúng xe ba bánh bên trái với `three_wheeler 0.40` (điểm thấp nhưng đúng vật) nhưng cũng sinh box giả trên đuôi xe bán tải trắng (`0.37`).
- **Giới hạn:** Ngưỡng `0.15` cho `three_wheeler` được chọn và chấm trên cùng **4 ảnh gold** (`53` đối tượng đánh giá / `62` đối tượng tổng gồm ignore), nên precision (`61.0%`) và recall (`67.9%`) trên gold mang tính **lạc quan (in-sample)** và khoảng tin cậy rộng.

## 2. AI error profile (từ `submission/qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 436 | 436/521 = 83.7% | 521 box máy |
| edited | 25 | 25/521 = 4.8% | 521 box máy |
| deleted | 60 | 60/521 = 11.5% | 521 box máy |
| added | 72 | 72/533 = 13.5% | 533 đối tượng sau duyệt |

*(Trong `deleted + added` có **11 ca đổi lớp**, số vật máy sót thật ≈ `72 - 11 = 61` vật).*
- **Lớp máy tệ nhất:** `three_wheeler` (`accepted` chỉ `9`, `deleted = 25`, `edited = 8`, `added = 6`) do ngưỡng `0.15` của YOLOE kéo theo nhiều box giả trên bạt, kệ hàng, hộp điện (`img_005`, `img_014`) và nhận nhầm `truck`/`bus` thành `three_wheeler` (`img_003`, `img_013`, `img_018`).
- **So với gold:** Precision của máy `36/(36+23) = 61.0%` (KTC 95%: `48–72%`), Recall `36/53 = 67.9%` (KTC 95%: `55–79%`, mẫu số `53` đối tượng gold).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- **Bắt bẫy:** `10/12 = 83.3%` (KTC 95%: `55–95%`, đạt ngưỡng ≥ 80%): `extra 3/3`, `missing 3/3`, `wrong_class 2/3`, `loose 2/3`.
- **2 bẫy lọt theo script (`img_013.jpg` loose, `img_034.jpg` wrong_class):**
  - `img_034.jpg` (`wrong_class`): Thực tế đã sửa đúng xe tải nhỏ mui bạt 4 bánh từ `car` sang `truck` (IoU `1.0`), nhưng nhãn gốc của máy trước khi cài bẫy là `three_wheeler` nên `review_stats.py` không tính khớp.
  - `img_013.jpg` (`loose`): Box `person` ngồi bàn phía sau tuk-tuk bị khuất chân nên khi thu gọn phần chân làm IoU với box gốc còn `0.459 < 0.5`.
- **Ý nghĩa `accepted` cao (`83.7%`) kèm bắt bẫy `83.3%`:** Cho thấy giữ nguyên đa số box chuẩn của máy nhưng vẫn kiểm soát tốt các bẫy cài thêm; tuy nhiên ở lượt 1 chỉ bắt được `7/12` (`loose 0/3`) do chủ quan nhìn lướt các box lệch nhẹ, sau khi soi lại cạnh box mới lên `10/12`.

## 4. Máy có đáng dùng không?

- **Thời gian (từ `submission/time_log.csv`):**
  - Gán tay 4 ảnh gold: `25/4 = 6.25 phút/ảnh`.
  - Duyệt 24 ảnh batch có pre-label: lượt 1 rà toàn bộ 24 ảnh hết `100/24 = 4.17 phút/ảnh` (nhanh hơn gán tay ~33%); tổng cả 3 lượt gồm quay lại rà & sửa bẫy là `191/24 = 7.96 phút/ảnh`.
  - *Thiên lệch cần nêu:* Gán 4 ảnh gold trước giúp quen bối cảnh đường phố châu Á trước khi duyệt batch.
- **Kết luận:** **Đáng dùng** cho các lớp phổ biến (`person`, `motorbike`, `car` với `accepted` lần lượt `248`, `124`, `46` box), giúp tiết kiệm thời gian dựng khung ban đầu ở lượt duyệt chính; riêng `three_wheeler` cần siết lại lọc trùng/nhiễu và không nên tin ngưỡng `0.15` khi chưa kiểm.
- **Ba quy tắc giảm thiên kiến cá nhân:**
  1. Bấm ẩn toàn bộ nhãn máy trước khi xem mỗi ảnh (`100%` ảnh) và tự nhẩm số lượng vật trước khi bật lại box.
  2. Luôn rà riêng một lượt độ khít 4 cạnh box ở các ảnh đông xe (`img_013`, `img_024`) để tránh bỏ sót lỗi `loose` (`0/3` ở lượt 1).
  3. Nghỉ giải lao 5 phút sau mỗi 10 ảnh (theo `time_log.csv`) để tránh mệt mỏi làm lọt bẫy ở nửa cuối lô.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_013.jpg` | `sam2.1_b.pt` | 2 pos, 1 neg | Có dính người lái và người ngồi sau khung hở | 1 click âm trên lưng chưa tách hết đầu/chân tài xế và bánh xe máy phía trước | `submission/masks/tuktuk_img013.jpg` |
| `img_010.jpg` | `sam2.1_b.pt` | 1 pos, 1 neg | Tách được phần lớn người đạp phía sau | Đứt/sót một phần bánh xe mảnh và chi tiết nan hoa | `submission/masks/cyclo_img010.jpg` |
| `img_017.jpg` | `sam2.1_b.pt` | 1 pos, 1 neg | Dính nhẹ phần nền/vật che cạnh xe | Vật bị che khuất một phần nên biên mask bị răng cưa ở vùng giao nhau | `submission/masks/occluded_img017.jpg` |

- **Đối chiếu guideline mục 4.2:** Quy tắc tách `person` và xe (mục 4.1.4 & 4.2) rất cần thiết vì SAM mặc định gộp cả người lái vào xe khi dùng 1–2 click; quy tắc lỗ thủng `< 5%` giúp không phải đục các khe nhỏ trên khung xe.
- **Interactor SAM trong CVAT (`SAM 3 (Google Colab GPU)`):** Đã cài và kết nối qua Nuclio proxy (link notebook SAM 3: https://colab.research.google.com/drive/1LNpNfKtkZG_NwuGHZ6SwbMyh7SqF24hr#scrollTo=step3), nhưng **không sử dụng** cho bài nộp chính mà dùng `sam2.1_b.pt` qua `sam2_points.py` để đồng bộ định dạng ảnh mask đầu ra kèm tọa độ click.

## 6. Calibration (Part 6)

- **Số quyết định:** `4` dòng (`CAL-01` đến `CAL-04`) trong `submission/qc/calibration_decisions.csv`. Quan trọng nhất là **`CAL-01`**: phân biệt `three_wheeler` với `truck`/`bus` dựa vào số bánh và cấu trúc buồng lái để giảm lỗi `deleted=25` của YOLOE.
- **Quy tắc mới đã vào `submission/guidelines/guideline_v1.1_calibrated.md`:** Chốt cụm dày `< 20px` dùng `ignore_region` và xóa box ma trên vệt sáng đèn đêm (`CAL-02`), quy tắc hàng nhô `> 20%` không kéo dài box xe (`CAL-03`), và tách người lái khỏi mask SAM (`CAL-04`).
- **Nguyên nhân gốc:** Kết hợp cả **máy** (YOLOE ngưỡng thấp sinh box ma & nhầm `truck`/`bus`) và **guideline v1.0 còn câu mở** (#2 khối xe dày, #3 hàng nhô, #6 nhiễu ảnh đêm).

## 7. QC và quyết định (Part 6)

- **Mẫu:** `5/24` ảnh batch (`0` ca khó `can_review` + `5` ảnh ngẫu nhiên seed `42`: `img_003`, `img_006`, `img_014`, `img_031`, `img_035`), tổng `105` đối tượng. Người kiểm: Phan Hải Đăng (**hạn chế:** người kiểm đồng thời là người duyệt).
- **Lỗi theo đối tượng:** `0/105 = 0.0%` · **Lỗi theo ảnh:** `0/5 = 0.0%` (ngưỡng Release ≤ 2%, dữ liệu từ `submission/qc/qc_report_auto.md`).
- **Top lỗi:** Không ghi nhận lỗi trên 5 ảnh mẫu (`error_type = khong` cho cả 5 ảnh).
- **Giới hạn mẫu nhỏ:** `0` ảnh lỗi trong `5` ảnh mẫu có cận trên khoảng tin cậy 95% (Wilson) lên tới **`43%`**. Do đó mẫu nhỏ 5 ảnh **không thể chứng minh** tỉ lệ lỗi thật của toàn bộ 24 ảnh dưới 2%.
- **Script kiểm được / không kiểm được:** Script (`validate_labels.py`, `review_stats.py`, `qc_report.py`) kiểm được cấu trúc COCO, tên nhãn, kích thước `< 20px`, tỉ lệ bắt bẫy và tổng hợp phiếu QC; **không** tự kiểm được box có ôm sát biên thực tế hay người duyệt có sót cùng vật với máy không.

**Quyết định: Release**, ngày 2026-11-18. Lý do (theo cây Hold → Rework → Release): Guideline v1.1 đã được calibration (4 quyết định), không còn lỗi hệ thống, bắt bẫy đạt `10/12 = 83.3%` (≥ 80%), và kiểm mẫu QC đạt `0/5 = 0.0%` ảnh lỗi (≤ 2%).

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài:** Có sử dụng trợ lý AI để hỗ trợ kiểm tra lỗi môi trường Windows/Google Drive, giải nén file export từ CVAT/Colab, chạy các script đo kiểm (`threshold_sweep.py`, `review_stats.py`, `qc_sample.py`, `qc_report.py`), và rà soát đối chiếu số liệu từ `review_stats.md` vào báo cáo. Toàn bộ việc gán nhãn và review trong CVAT do học viên tự thực hiện bằng mắt.
- **Gold có bị nhiễm không:** **Không.** 4 ảnh gold được gán tay hoàn toàn trong CVAT trước khi chạy bất kỳ model pre-label nào ở Part 2.
- **Máy / GPU / CVAT:** Windows 11, không có GPU NVIDIA; chạy CVAT Docker `v2.81.0` trên máy (kèm Nuclio `YOLO26n`, `YOLOE26s`, và `SAM 3 Colab proxy` — có cài nhưng không dùng, link: https://colab.research.google.com/drive/1LNpNfKtkZG_NwuGHZ6SwbMyh7SqF24hr#scrollTo=step3) và chạy script `prelabel.py` / `sam2_points.py` trên Google Colab (CPU). Vướng mắc: Google Drive Desktop đôi lúc ngắt kết nối giữa ổ `G:` và `D:`.
- **Dùng cache `data/cache/`:** **Không.** Đã tự chạy toàn bộ `prelabel.py` (`pilot` và `all`) trên Google Colab và lưu `work/runs.csv`.
- **Lỗi / bug đã gặp:** Bẫy `img_034.jpg` (`wrong_class`) gốc máy là `three_wheeler` bị đổi thành `car`; khi sửa đúng theo guideline thành `truck` thì `review_stats.py` không tính bắt được bẫy.
- **Một điều làm khác lần sau:** Sao chép repo ra ổ đĩa cục bộ ngay từ đầu để tránh rớt đồng bộ Google Drive và kiểm tra kỹ 4 cạnh box ngay từ lượt review đầu tiên.
