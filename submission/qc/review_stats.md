# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 348; đối tượng sau duyệt: 426; ngưỡng accepted: IoU >= 0.9
- Đã **loại 26 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 191 | 191/348 = 54.9% | 348 box máy |
| edited (sửa) | 94 | 94/348 = 27.0% | 348 box máy |
| deleted (xoá) | 63 | 63/348 = 18.1% | 348 box máy |
| added (vẽ thêm) | 141 | 141/426 = 33.1% | 426 đối tượng sau duyệt |

Trong số deleted + added, **12 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 129.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 120 | 46 | 10 | 58 |
| bicycle | 3 | 0 | 2 | 8 |
| motorbike | 38 | 38 | 8 | 42 |
| car | 20 | 4 | 6 | 20 |
| bus | 2 | 0 | 0 | 5 |
| truck | 1 | 0 | 1 | 3 |
| three_wheeler | 7 | 6 | 36 | 5 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 55 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 36 | 6 | 19 | 85.7% (KTC 95%: 72–93%) | 65.5% (KTC 95%: 52–77%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 7/12 = 58.3% (KTC 95%: 32–81%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - extra: 2/3
  - loose: 1/3
  - missing: 2/3
  - wrong_class: 2/3
  - lọt: img_003.jpg (loose), img_034.jpg (extra), img_018.jpg (wrong_class), img_021.jpg (missing), img_035.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
