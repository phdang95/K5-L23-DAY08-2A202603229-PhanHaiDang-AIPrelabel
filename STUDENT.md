# Thông tin học viên

- Họ và tên: Phan Hải Đăng
- MSHV: 2A202603229
- Lớp (L2 / L3 / khác): L23
- Tài khoản GitHub: phdang95
- Repo bài nộp (URL): https://github.com/phdang95/K5-L23-DAY08-2A202603229-PhanHaiDang-AIPrelabel
- Commit chốt bài (hash): (cập nhật ở commit cuối)
- Máy: Windows 11, không có GPU NVIDIA (chạy model trên Google Colab CPU)
- CVAT (Docker): v2.81.0

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 436/521 (83.7%) / 25/521 (4.8%) / 60/521 (11.5%) (mẫu số: 521 box máy); added: 72/533 (13.5%) (mẫu số: 533 đối tượng sau duyệt)
- So với gold: precision / recall của **máy** 36/(36+23) = 61.0% (KTC 95%: 48–72%) / 36/53 = 67.9% (KTC 95%: 55–79%)
- Bắt bẫy: 10/12 = 83.3% (KTC 95%: 55–95%)
- Thời gian: gán tay 25/4 = 6.25 phút/ảnh; duyệt có pre-label 191/24 = 7.96 phút/ảnh (tổng 3 lượt gồm sửa lại bẫy; lượt đầu 100/24 = 4.17 phút/ảnh)
- Lỗi QC: 0.0% theo ảnh (0/5 ảnh mẫu), 0.0% theo đối tượng (0/105 đối tượng mẫu)
- **Quyết định:** Release — lý do một dòng: ảnh lỗi 0/5 = 0.0% (≤ 2%), bắt bẫy đạt 10/12 = 83.3% (≥ 80%), guideline v1.1 đã calibration; lưu ý giới hạn mẫu nhỏ (cận trên 95% Wilson = 43%).
