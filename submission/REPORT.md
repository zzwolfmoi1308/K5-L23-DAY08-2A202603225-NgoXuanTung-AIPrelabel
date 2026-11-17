# Báo cáo Day 08 — 2A202603225 - Ngô Xuân Tùng

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26-n | car=0.45; person, motorbike, bicycle, bus, truck=0.30 (mặc định) | YOLO26-n bắt rất nhanh và chuẩn xác các lớp COCO cơ bản (r01_img002, r01_img007, r01_img015). car đạt recall 100% ở ngưỡng 0.45 trên gold; các lớp ít mẫu (<5 vật: bicycle 4, bus 1, truck 0) hoặc không có ngưỡng <=0.5 đạt 90% recall (person tối đa 58%, motorbike tối đa 89%) giữ mặc định 0.30 để tránh bỏ sót. |
| three_wheeler | YOLOE-26s, prompt: 'tuk-tuk', 'cyclo', 'cargo tricycle' | 0.15 | YOLO26 hoàn toàn không có lớp xe ba bánh nên bỏ sót hoặc gọi nhầm sang truck/motorbike/bicycle (r01_img002, r01_img007, r01_img032). YOLOE giải quyết được điểm mù này (r03_img002, r03_img011, r03_img032); chọn ngưỡng 0.15 để đạt recall 100% trên gold (7/7 vật). |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): <<dán sau khi chạy ở Part 4>>
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: SAM 2.1 tiny (`sam2.1_t.pt`) vì máy chạy môi trường Windows CPU (không GPU), bản tiny tối ưu tốc độ (~5.12s/ảnh so với hàng chục giây của base), tiết kiệm RAM nhưng vẫn sinh polygon chi tiết (64-96 đỉnh) bám sát viền người và xe (r04_img004, r04_img013).
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? Lỗi phân loại sai bản chất do danh mục đóng của COCO không có lớp xe ba bánh: ở r01_img002 xe tuk-tuk bị YOLO26 gán thành truck (0.37), ở r01_img032 xích lô bị gán thành bicycle. Nâng ngưỡng lên 0.5 (r02_img002, r02_img032) chỉ làm box biến mất thành lỗi bỏ sót chứ không đổi được tên lớp. Sửa bằng cách kết hợp model open-vocabulary (YOLOE) với text prompt riêng cho three_wheeler, xếp YOLOE lên trước YOLO26 khi merge.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): 
  - Điểm cao mà sai: Ở `img_002.jpg`, chiếc xe bán tải ở rìa bên phải bị YOLOE gán nhầm thành `three_wheeler` với điểm 0.37 (r03_img002); chiếc tuk-tuk bị YOLO26 tự tin gán thành `truck` với điểm 0.37 (r01_img002).
  - Điểm thấp mà đúng: Ở `img_011.jpg`, chiếc xe tuk-tuk bị che khuất ở mép trái được YOLOE phát hiện đúng `three_wheeler` dù điểm chỉ đạt 0.28 (r03_img011).
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: Tập gold chỉ có 4 ảnh với tổng số 61 đối tượng (mẫu nhỏ); các lớp như bicycle (4), bus (1), truck (0) quá ít để có ý nghĩa thống kê; chọn và đo trên cùng một tập dữ liệu dẫn đến in-sample bias, do đó độ chính xác thực tế khi áp dụng lên 24 ảnh batch sẽ thấp hơn trên gold.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | <<>> | <<>> | <<box máy>> |
| edited | <<>> | <<>> | <<>> |
| deleted | <<>> | <<>> | <<>> |
| added | <<>> | <<>> | <<đối tượng sau duyệt>> |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? <<>>
- So với gold: precision / recall của **máy** <<>> (mẫu số: <<đối tượng gold>>)

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: <<.../12>>; loại nào lọt: <<>>; vì sao (bạn đã bỏ bước nào của checklist)? <<>>
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? <<>>

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay <<... phút/ảnh>> vs duyệt <<... phút/ảnh>>. Thiên lệch (bạn gán gold trước nên đã quen cảnh): <<>>
- Kết luận: <<có / không / chỉ cho lớp nào>>, dựa trên <<số liệu>>
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. <<>>
  2. <<>>
  3. <<>>

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| <<img_0xx>> | <<>> | <<>> | <<>> | <<>> | <<>> |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? <<>>
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: <<>>

## 6. Calibration (Part 6)

- Số quyết định: <<>> (`calibration_decisions.csv`); quan trọng nhất (dẫn `CAL-xx`): <<>>
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: <<>>
- Nguyên nhân gốc: <<guideline mơ hồ / chưa nắm quy tắc / máy>>

## 7. QC và quyết định (Part 6)

- Mẫu: <<n>> ảnh (<<m>> ảnh `can_review` + <<k>> ngẫu nhiên, seed 42); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: <<.../... = ...%>> · theo **ảnh**: <<.../... = ...%>> (ngưỡng Release ≤ 2%)
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: <<>>
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): <<>>
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): <<>>

**Quyết định: <<Release / Hold / Rework>>**, ngày <<YYYY-MM-DD>>. Lý do (đi theo Hold → Rework → Release): <<>>
Nếu Rework / Hold, việc cần làm tiếp: <<>>

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): <<>>
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? <<không / có, như sau: ...>>
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: <<>>
- Dùng cache `data/cache/` thay vì tự chạy? <<không / có, vì ...>>
- Lỗi / bug script đã gặp (nếu có): <<>>
- Một điều bạn làm khác lần sau: <<>>
