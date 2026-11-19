# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603225 - Ngô Xuân Tùng

> **Bản bổ sung quy tắc gán nhãn v1.1 do học viên Ngô Xuân Tùng thiết lập** (chỉ viết phần thay đổi so với v1.0). Tôi chốt các câu hỏi mở ở mục 6 của v1.0 và bổ sung các quy tắc mới rút ra từ lỗi thực nghiệm của máy và các bẫy bị lọt. Mỗi ca khó đều có ảnh ví dụ (`img_0xx`) và khớp với một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định của tôi | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện và xe máy điện: xếp vào `bicycle` hay `motorbike`? | Tôi quy định: Xe đạp điện có bàn đạp cơ học trợ lực xếp vào `bicycle`; xe máy điện kiểu dáng xe tay ga không bàn đạp xếp vào `motorbike` | `img_024.jpg` | CAL-02 |
| 3 | Hàng hóa nhô cồng kềnh ra khỏi xe: box xe có gồm hàng không? | Tôi quy định: Hàng hóa nằm trong lòng thùng xe được tính vào box; hàng hóa nhô ra > 20% chiều dài xe thì không tính vào box phương tiện | `img_015.jpg` | CAL-04 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của tôi | Quy tắc mới tôi bổ sung (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_013.jpg` | Xe tuk-tuk buồng lái hở có hành khách | YOLO26 bỏ sót hoặc gọi là car; YOLOE sinh box trùng | Tôi gán nhãn `three_wheeler`, xóa box trùng lặp | Mục 2 & 4.1: Mọi xe ba bánh có mui hở quy về three_wheeler, phân biệt rõ với car/truck |
| `img_003.jpg` | Box xe ba bánh bị lệch mép viền (bẫy loose) | Gợi ý box lỏng bao cả phần bóng đổ mặt đường | Tôi nắn khít 4 mép hộp ôm sát tiếp tuyến thân xe | Mục 4.1: Tiêu chuẩn viền khít (sai lệch <= 3px), bắt buộc zoom >= 150% rà soát |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Tôi chốt phân loại xe đạp điện/máy điện; chuẩn hóa nhận diện xe ba bánh mui hở; siết quy tắc chống viền lỏng | Khắc phục lỗi lặp của máy và các bẫy lọt trong quá trình tôi thực nghiệm (CAL-01, CAL-02, CAL-03) |
