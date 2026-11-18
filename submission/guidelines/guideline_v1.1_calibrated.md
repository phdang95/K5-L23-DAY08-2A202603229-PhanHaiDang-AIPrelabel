# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603229 - Phan Hải Đăng

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 2 & 6 | Khối xe dày đặc (câu #2) & box chồng/nhiễu ảnh đêm (câu #6) | Cụm ≥ 5 vật sát nhau hoặc cao `< 20px` gom thành 1 `ignore_region`; xóa toàn bộ box ma sinh trên vệt sáng đèn đêm. | `img_018`, `img_024` | `CAL-02` |
| 3 | Hàng hóa nhô ra khỏi xe (xe chở chuối, thùng hàng) | Giỏ/thùng gắn cố định nằm gọn khung xe thì gộp vào box xe; hàng rời nhô ra `> 20%` chiều dài xe không kéo box xe theo hàng. | `img_009`, `img_010` | `CAL-03` |
| 1, 4, 5, 7 | Các câu mở còn lại (#1 xe đạp điện, #4 ngưỡng 20% che khuất, #5 kính xe trong mask, #7 ngưỡng `conf` tự chấp nhận) | **Chưa chốt cứng trong v1.1** (cần thêm mẫu thử / kiểm tra `conf` không thay thế việc nhìn ảnh theo checklist). | — | — |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_003`, `img_013`, `img_018` | Phân biệt `three_wheeler` với `truck` / `bus` / `car` | YOLO26 gọi tuk-tuk là `truck`/`car`; YOLOE gọi nhầm `bus`/`truck` là `three_wheeler` (`deleted=25`). | Đếm bánh và cấu trúc khung mui: 3 bánh → `three_wheeler`; ≥ 4 bánh có buồng lái/thùng hoặc xe khách → `truck`/`bus`. | Bổ sung mục 2 (Định nghĩa lớp `three_wheeler`, `truck`, `bus`). |
| `img_034`, `img_013` | Xe tải nhỏ mui bạt 4 bánh & tách người khỏi mask SAM | Máy gán xe tải nhỏ 4 bánh thành `three_wheeler`/`car` (`img_034`); SAM 2.1 tô dính người lái qua khung hở (`img_013`). | Luôn giữ `truck` cho xe tải nhỏ 4 bánh; khi tạo mask `three_wheeler` phải dùng nhiều click âm tách `person`. | Bổ sung mục 2 (`truck`) và mục 4.2 (Mask). |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt quy tắc `ignore_region` cụm dày/ảnh đêm, hàng nhô `> 20%`, phân biệt `three_wheeler` vs `truck`/`bus`, và tách người khỏi mask SAM. | Lỗi lặp `three_wheeler` (`CAL-01`), câu hỏi mở #2/#3/#6 (`CAL-02`, `CAL-03`), bẫy `img_034` & mask `img_013` (`CAL-04`). |
