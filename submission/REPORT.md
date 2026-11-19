# Báo cáo Day 08 — 2A202603225 - Ngô Xuân Tùng

> Báo cáo tổng kết thực nghiệm của học viên Ngô Xuân Tùng (MSHV: 2A202603225 - Lớp L2). Mọi con số đều đi kèm mẫu số và nguồn dữ liệu kiểm chứng.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26-n | car=0.45; person, motorbike, bicycle, bus, truck=0.30 (mặc định) | Tôi nhận thấy YOLO26-n xử lý rất nhanh và chuẩn xác các lớp COCO cơ bản trên tập ảnh thử nghiệm (`r01_img002`, `r01_img007`, `r01_img015`). Khi quét ngưỡng trên 4 ảnh gold, lớp `car` đạt recall 100% tại ngưỡng 0.45; đối với các lớp có mẫu quá ít trong gold (< 5 vật: bicycle có 4, bus có 1, truck có 0) hoặc không có ngưỡng nào <= 0.5 đạt recall 90% (person tối đa 58%, motorbike tối đa 89%), tôi quyết định giữ ngưỡng mặc định 0.30 theo khuyến nghị để tránh bỏ sót vật thể thực tế. |
| three_wheeler | YOLOE-26s, prompt: 'tuk-tuk', 'cyclo', 'cargo tricycle' | 0.15 | YOLO26 là closed-set không có nhãn xe ba bánh nên bỏ sót hoàn toàn hoặc gọi nhầm sang truck/motorbike/bicycle (`r01_img002`, `r01_img007`, `r01_img032`). Tôi sử dụng YOLOE để giải quyết điểm mù này (`r03_img002`, `r03_img011`, `r03_img032`); tôi chọn ngưỡng 0.15 vì ngưỡng này giúp đạt recall 100% (7/7 vật gold) và precision 78%. |

- Lệnh `merge_prelabels.py` tôi đã dùng (dán nguyên): `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "car=0.45,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM tôi chọn** (SAM 1 / 2.1 / 3) và lý do: Tôi chọn SAM 2.1 tiny (`sam2.1_t.pt`) vì môi trường máy của tôi chạy Windows ở chế độ CPU (không GPU rời). Bản SAM 2.1 tiny tối ưu thời gian suy luận (~5.12s/ảnh so với hàng chục giây của bản base), tiêu tốn ít bộ nhớ RAM mà vẫn sinh ra polygon chi tiết (64–96 đỉnh) ôm sát đường biên của người và xe (`r04_img004`, `r04_img013`).
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Tôi đã sửa thế nào? Đó là lỗi phân loại sai bản chất do danh mục đóng của COCO không có lớp xe ba bánh: ví dụ ở `r01_img002` chiếc tuk-tuk bị YOLO26 gán thành `truck` (điểm 0.37), ở `r01_img032` chiếc xích lô bị gán thành `bicycle`. Khi tôi nâng ngưỡng lên 0.5 (`r02_img002`, `r02_img032`), box chỉ biến mất thành lỗi bỏ sót chứ không thể chuyển đúng về `three_wheeler`. Tôi đã khắc phục bằng cách kết hợp model open-vocabulary (YOLOE) với các text prompt cụ thể cho `three_wheeler`, đồng thời xếp YOLOE lên trước YOLO26 khi chạy merge để ưu tiên nhãn ba bánh.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (tôi ghi nhận từ ảnh nào): 
  - Điểm cao mà sai: Ở `img_002.jpg`, chiếc xe bán tải ở rìa bên phải bị YOLOE gán nhầm thành `three_wheeler` với điểm 0.37 (`r03_img002`); chiếc tuk-tuk bị YOLO26 tự tin gán thành `truck` với điểm 0.37 (`r01_img002`).
  - Điểm thấp mà đúng: Ở `img_011.jpg`, chiếc xe tuk-tuk bị che khuất ở mép trái được YOLOE phát hiện đúng `three_wheeler` dù điểm tin cậy chỉ đạt 0.28 (`r03_img011`).
- Giới hạn khi tôi chọn ngưỡng trên chính tập gold (4 ảnh): Tập gold của tôi chỉ gồm 4 ảnh với tổng số 61 đối tượng (mẫu nhỏ); các lớp như bicycle (4), bus (1), truck (0) quá ít để có ý nghĩa thống kê. Việc tôi chọn ngưỡng và đánh giá trên cùng một tập dữ liệu nhỏ dẫn đến hiện tượng in-sample bias (số đo lạc quan), do đó độ chính xác thực tế khi áp dụng lên 24 ảnh batch sẽ thấp hơn so với trên gold.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 191 | 191/348 = 54.9% | 348 box máy |
| edited | 94 | 94/348 = 27.0% | 348 box máy |
| deleted | 63 | 63/348 = 18.1% | 348 box máy |
| added | 141 | 141/426 = 33.1% | 426 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? Lớp `three_wheeler` là tệ nhất: tôi phải xóa tới 36 box trên tổng số 49 box máy gợi ý (tỷ lệ xóa > 73%), số box tôi giữ nguyên (accepted) chỉ có 7 box. Nguyên nhân do YOLOE sinh ra nhiều box trùng lặp quanh cùng một vật và nhận diện nhầm ô tô, xe tải, vật thể ven đường thành xe ba bánh (ví dụ ở `img_002.jpg`, `img_023.jpg`). Ngoài ra lớp `bicycle` cũng bị máy bỏ sót rất nhiều (tôi phải tự vẽ thêm 8 box trong khi máy chỉ bắt đúng 3 box).
- So với gold: precision / recall của **máy** đạt Precision 85.7% (KTC 95%: 72–93%) (36/42), Recall 65.5% (KTC 95%: 52–77%) (mẫu số: 55 đối tượng gold).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: Tôi bắt được 7/12 = 58.3% bẫy (KTC 95%: 32–81%). 5 bẫy tôi để lọt gồm: 2 bẫy viền lỏng `loose` (`img_003.jpg`, `img_035.jpg`), 1 bẫy thừa `extra` (`img_034.jpg`), 1 bẫy sai lớp `wrong_class` (`img_018.jpg`), 1 bẫy sót `missing` (`img_021.jpg`).
- Vì sao lọt bẫy (tôi đã bỏ sót bước nào của checklist)? Tôi nhận thấy mình đã bỏ sót Bước 1 trong checklist (không tắt hiển thị nhãn máy bằng phím `H` trong 15 giây đầu để quét mắt đếm tổng thể, dẫn đến việc nhìn theo gợi ý của AI và bỏ lọt bẫy sót `missing` ở `img_021.jpg` cũng như bẫy thừa `extra` ở `img_034.jpg`). Đồng thời tôi cũng vội ở Bước 3 khi không phóng to đủ lớn để nắn khít tiếp tuyến viền hộp, dẫn đến để sót 2 bẫy `loose` ở `img_003.jpg` và `img_035.jpg`, và không đọc kỹ nhãn thuộc tính ở `img_018.jpg`.
- Accepted cao mà bắt bẫy thấp nói gì về cách tôi duyệt? Khi tỷ lệ accepted của tôi đạt 54.9% nhưng tỷ lệ bắt bẫy chỉ đạt 58.3% (dưới ngưỡng 80%), con số này phản ánh rõ nét việc tôi đã mắc phải 'Thiên kiến tự động hóa' (Automation Bias). Tôi có tâm lý ỷ lại và tin tưởng một cách thụ động vào các bounding box do máy vẽ sẵn; tôi chủ yếu chỉ can thiệp vào các lỗi sai quá rõ ràng mà lơ là việc kiểm tra độ khít của viền và tìm kiếm các đối tượng mà máy đã bỏ sót hoàn toàn.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): Gán tay gold (Part 1): tôi mất 5.5 phút/ảnh (22 phút/4 ảnh) vs Duyệt batch có AI (Part 4): tôi mất 3.13 phút/ảnh (75 phút/24 ảnh). Thiên lệch (tôi gán gold trước nên đã quen cảnh): Tôi nhận thức rõ có sự thiên lệch do tôi đã tự gán 4 ảnh gold từ trước, nên mắt tôi đã quen thuộc với bối cảnh đường phố, góc chụp và các loại xe. Vì vậy tốc độ duyệt ở tập batch nhanh hơn một phần đến từ kinh nghiệm cá nhân chứ không hoàn toàn 100% nhờ AI.
- Kết luận: Theo tôi, máy **rất đáng dùng** đối với các lớp phổ biến và có hình thái ổn định như `person` (tôi accepted 120 box) và `car` (tôi accepted 20 box), giúp tôi tiết kiệm được hơn 43% thời gian so với việc phải vẽ lại từng hộp từ đầu. Tuy nhiên, đối với các lớp đặc thù như `three_wheeler` (tôi phải xóa 36 box do máy nhận diện nhầm và trùng lặp) hoặc `bicycle` (tôi phải tự bổ sung 8 box), máy chưa thực sự đáng tin cậy và tôi vẫn phải đóng vai trò kiểm soát chính.
- **Ba quy tắc giảm thiên kiến của tôi** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm cụ thể):
  1. *Nguyên tắc "Tắt mắt máy 15 giây"*: Với mỗi ảnh mới mở ra trên CVAT, tôi luôn ấn phím tắt `H` để ẩn hoàn toàn các box máy trong 15–20 giây đầu; tôi tự quét mắt đếm nhẩm số người và xe thực tế trên ảnh để ghi nhận các vật bị sót trước khi bật lại gợi ý của AI.
  2. *Quy tắc phóng to 4 góc (Zoom checklist)*: Tôi bắt buộc phóng to $\ge 150\%$ tại từng đối tượng để kiểm tra tiếp tuyến 4 cạnh viền hộp (không để lọt bẫy `loose`), đồng thời nhìn rõ tên lớp hiển thị trên nhãn trước khi bấm next (loại trừ bẫy `wrong_class`).
  3. *Ngưỡng thời gian tối thiểu theo mật độ*: Tôi đặt quy định giữ tối thiểu 2.0 phút/ảnh cho những bức ảnh có mật độ giao thông đông ($\ge 15$ đối tượng), tuyệt đối không lướt vội dưới 1 phút/ảnh để tránh tâm lý 'thấy kín hộp là an tâm'.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013.jpg | SAM 2.1 (`sam2.1_t.pt`) | 3 (2 pos, 1 neg) | Không gồm người lái, mask bám sát thân xe tuk-tuk | Mask phủ liền mạch, điểm neg tách sạch nền phía sau xe taxi | `tuktuk_img013_sam2.jpg` |
| img_013.jpg | SAM 1 (`mobile_sam.pt`) | 3 (2 pos, 1 neg) | Mask bám thân xe nhưng viền mái che bị lẹm | Bị vỡ thành 2 polygon rời rạc (diện tích nhỏ hơn SAM 2.1), viền lởm chởm | `tuktuk_img013_sam1.jpg` |
| img_004.jpg | SAM 2.1 (`sam2.1_t.pt`) | 3 (2 pos, 1 neg) | Không gồm người đứng cạnh, bao trọn thân xe tuk-tuk | Khung xe và bánh xe được tách gọn, điểm neg ở vùng giáp ranh ngăn lấn sang người | `tuktuk_img004_sam2.jpg` |
| img_017.jpg | SAM 2.1 (`sam2.1_t.pt`) | 3 (2 pos, 1 neg) | Không gồm người lái và hàng hóa | Mái che xe tuk-tuk tách khít biên; phần gầm xe giáp bóng đổ mặt đường hơi lấn nhẹ | `tuktuk_img017_sam2.jpg` |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp tôi, quy tắc nào tôi thấy còn thiếu? 
  - Quy tắc giúp ích: Quy tắc 4.2.3 (bóng đổ không thuộc vật, kính xe thuộc vật) và 4.2.4 (hai vật chạm nhau không vẽ đè lên nhau) đã giúp tôi định hướng vị trí đặt các click âm (neg) để bóc tách xe khỏi bóng râm và người ngồi trong xe.
  - Quy tắc tôi thấy còn thiếu: Guideline v1.0 chưa có hướng dẫn xử lý các khoảng hở xuyên thấu ở khung nan sắt của xe ba bánh hoặc xích lô (SAM thường tự tô kín khoảng trống giữa các thanh chắn hoặc bị rách polygon khi độ tương phản thấp).
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: Trên môi trường CVAT Docker local chạy CPU của tôi, module Serverless/Nuclio cho SAM rất nặng và dễ bị timeout; tôi chạy offline qua script `sam2_points.py` giúp kiểm soát chính xác tọa độ click pixel (x, y), thời gian phản hồi nhanh (~7-10s trên CPU) và tự động lưu ảnh mask vào thư mục nộp bài.

## 6. Calibration (Part 6)

- Số quyết định tôi đã đưa ra: 4 quyết định trong `calibration_decisions.csv` (CAL-01 đến CAL-04); quan trọng nhất là quyết định `CAL-01`: Tôi chuẩn hóa quy tắc phân loại cho các loại xe ba bánh có mui hở (tuk-tuk, xích lô, xe lam) đều quy về lớp `three_wheeler`, đồng thời quy định xóa bỏ các box trùng lặp do YOLOE sinh ra và ngăn YOLO26 gán nhầm thành `truck`/`car`.
- Quy tắc mới tôi đã đưa vào `guideline_v1.1_calibrated.md`: Tôi bổ sung quy tắc phân định xe đạp điện vs xe máy điện dựa vào bàn đạp cơ học (CAL-02), quy tắc nhận diện xe ba bánh buồng lái hở (CAL-01), và tiêu chuẩn kỹ thuật nắn khít viền hộp $\le 3\text{px}$ với yêu cầu zoom $\ge 150\%$ để khắc phục lỗi viền lỏng (CAL-03).
- Nguyên nhân gốc mà tôi rút ra: Tôi nhận thấy có sự kết hợp của cả 3 nguyên nhân: guideline v1.0 ban đầu còn bỏ ngỏ câu hỏi về xe điện và xe ba bánh hở (guideline mơ hồ); bản thân tôi trong quá trình duyệt chưa tuân thủ triệt để bước phóng to rà soát 4 góc viền nên để lọt bẫy loose (chưa nắm vững quy tắc); và các mô hình detector COCO chưa được huấn luyện riêng cho phương tiện giao thông Đông Nam Á nên sinh nhiều box rác (lỗi máy).

## 7. QC và quyết định (Part 6)

- Mẫu kiểm tra của tôi: 5 ảnh (0 ảnh `can_review` + 5 ngẫu nhiên, seed 42); người kiểm: Tôi (Ngô Xuân Tùng) (**hạn chế**: Tôi vừa là người duyệt nhãn vừa là người tự kiểm tra QC nên khó tránh khỏi thiên kiến chủ quan).
- Lỗi theo **đối tượng**: 2/81 = 2.5% · theo **ảnh**: 2/5 = 40.0% (ngưỡng Release ≤ 2%)
- Top lỗi tôi ghi nhận: `sai_vi_tri_kich_thuoc` (2 lỗi, do viền box bị lỏng ở `img_003.jpg` và `img_035.jpg`).
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): Với việc phát hiện 2 ảnh có lỗi trên 5 ảnh mẫu, cận trên khoảng tin cậy 95% (Wilson) của tỉ lệ ảnh lỗi thật lên tới 77.0%. Tôi nhận thấy cỡ mẫu 5 ảnh là quá nhỏ để có thể khẳng định chắc chắn về mặt toán học rằng chất lượng dữ liệu thực tế đạt ngưỡng ≤ 2%.
- Script kiểm được / không kiểm được (theo quan sát của tôi): Script kiểm tra được cấu trúc thư mục, định dạng JSON COCO, độ hợp lệ của taxonomy, sự khớp nối giữa các file nhãn và tỉ lệ số học. Script **không** kiểm tra được box có thực tế ôm sát viền đối tượng hay không, không phát hiện được đối tượng bị cả tôi và máy cùng bỏ sót trên ảnh gốc ngoài gold, và không giám sát được tôi có tuân thủ quy trình checklist hay không.

**Quyết định của tôi: Rework**, ngày 2026-11-19. Lý do (đi theo cây quyết định Hold → Rework → Release): Tôi đối chiếu cây quyết định: Guideline v1.1 của tôi đã bổ sung đầy đủ giải quyết các câu hỏi mơ hồ (không thuộc diện Hold); tuy nhiên tỉ lệ ảnh có lỗi trong mẫu QC của tôi lên tới 40.0% (vượt xa ngưỡng Release ≤ 2%) và tỉ lệ bắt bẫy chỉ đạt 58.3% (chưa đạt ngưỡng 80%). Để đảm bảo tính trung thực và chất lượng khoa học dữ liệu, tôi quyết định chọn **Rework**.
Nếu Rework / Hold, việc tôi cần làm tiếp: Tôi sẽ mở lại Task `batch-review` trên CVAT, tập trung rà soát lại 5 ảnh có bẫy bị lọt (`img_003.jpg`, `img_035.jpg`, `img_018.jpg`, `img_021.jpg`, `img_034.jpg`), phóng to nắn khít tiếp tuyến các box viền lỏng, bổ sung các đối tượng bị bỏ sót, sau đó xuất file cập nhật `submission/labels/qc_fixed.json` và chạy lại quy trình build release.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài**: Tôi sử dụng trợ lý Antigravity để hỗ trợ tra cứu cú pháp câu lệnh PowerShell, hướng dẫn quy trình theo giáo trình bài lab, kiểm tra các tiêu chí trong checklist và hỗ trợ tính toán các số đo thống kê. Toàn bộ thao tác gán nhãn gold và duyệt 24 ảnh batch trên CVAT tôi đều tự thực hiện thủ công bằng tay.
- **Gold có bị nhiễm không**: Tập gold của tôi hoàn toàn không bị nhiễm; tôi đã tự gán tay độc lập 4 ảnh gold trên CVAT ngay từ Part 1 trước khi chạy bất kỳ câu lệnh AI pre-label nào.
- **Máy / GPU / CVAT dùng; vướng mắc cài đặt**: Tôi sử dụng máy tính chạy Windows 11, thực nghiệm hoàn toàn trên CPU (không GPU rời), CVAT chạy qua Docker v2.74.1. Quá trình cài đặt và thực hiện thuận lợi, chỉ cần chú ý bọc các tham số PowerShell trong dấu nháy kép để tránh lỗi tách chuỗi.
- **Dùng cache `data/cache/` thay vì tự chạy?**: Không, tôi đã tự chạy 100% tất cả các script và model (`yolo26`, `yoloe`, `sam2_points`) trên máy của mình.
- **Lỗi / bug script đã gặp**: Không gặp lỗi nghiêm trọng trong mã nguồn; tôi nhận thấy khi chạy trên PowerShell cần lưu ý bọc dấu ngoặc kép quanh các giá trị có chứa dấu phẩy hoặc dấu bằng (`--thr`, `--split`, `--pos`, `--neg`).
- **Một điều tôi sẽ làm khác lần sau**: Nếu làm lại, ngay từ đầu tôi sẽ tuân thủ nghiêm ngặt quy tắc tắt hiển thị nhãn máy (`H`) trong 20–30 giây đầu ở mỗi ảnh để tự quan sát toàn cảnh trước, tránh bị định hướng bởi các box gợi ý của AI để không bỏ lọt bẫy viền lỏng và bẫy bỏ sót.
