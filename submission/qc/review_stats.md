# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 521; đối tượng sau duyệt: 533; ngưỡng accepted: IoU >= 0.9
- Đã **loại 22 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 436 | 436/521 = 83.7% | 521 box máy |
| edited (sửa) | 25 | 25/521 = 4.8% | 521 box máy |
| deleted (xoá) | 60 | 60/521 = 11.5% | 521 box máy |
| added (vẽ thêm) | 72 | 72/533 = 13.5% | 533 đối tượng sau duyệt |

Trong số deleted + added, **11 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 61.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 248 | 9 | 13 | 23 |
| bicycle | 6 | 0 | 2 | 5 |
| motorbike | 124 | 4 | 10 | 21 |
| car | 46 | 3 | 5 | 10 |
| bus | 2 | 1 | 0 | 5 |
| truck | 1 | 0 | 5 | 2 |
| three_wheeler | 9 | 8 | 25 | 6 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 53 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 36 | 23 | 17 | 61.0% (KTC 95%: 48–72%) | 67.9% (KTC 95%: 55–79%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 10/12 = 83.3% (KTC 95%: 55–95%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - extra: 3/3
  - missing: 3/3
  - wrong_class: 2/3
  - loose: 2/3
  - lọt: img_013.jpg (loose), img_034.jpg (wrong_class)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
