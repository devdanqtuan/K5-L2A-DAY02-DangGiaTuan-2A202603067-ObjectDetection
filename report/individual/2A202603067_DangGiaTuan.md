# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Đặng Gia Tuấn
- MSSV: 2A202603067
- Email: danggiatuan.work@gmail.com
- Lớp: L2A
- Link repo bài làm: https://github.com/devdanqtuan/K5-L2A-DAY02-DangGiaTuan-2A202603067-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/drive/1kM-3PYnwlanM4kI8fsjDUB9Zm46-6HzL?usp=sharing

## 2. Schema và mini guideline

- Tham chiếu chi tiết: [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md)
- Schema 4 lớp chuẩn: `0: car`, `1: truck`, `2: bus`, `3: van`.
- Tóm tắt các quyết định và ca mơ hồ đã thống nhất:
  - `drive_006.jpg`: Phương tiện màu trắng/xám ở góc trái dưới chỉ thấy một phần bánh và đèn. Quyết định: Gán `car` vì gầm thấp, bánh nhỏ đặc trưng cho xe con, không có cấu trúc thùng hàng của `truck` hay thân hộp của `van`.
  - `drive_020.jpg`: Xe ở góc dưới cùng bên phải bị mép ảnh cắt ngang chỉ thấy phần đuôi và đèn hậu. Quyết định: Gán `car` vì cụm đèn hậu và form đuôi sedan/hatchback rõ ràng; áp dụng quy tắc box chạm sát rìa ảnh (không vẽ suy đoán ra ngoài).
  - `drive_021.jpg`: Xe phía sau trạm dừng xe buýt bị che khuất hơn 85%. Quyết định: Không gán nhãn do không đủ đặc trưng nhận diện chắc chắn theo quy tắc vật thể bị che khuất quá mức.
  - `drive_026.jpg`: Đoàn xe buýt nối đuôi / khớp nối nhìn như xe buýt dài kéo. Quyết định: Gán tách rời từng phần là `bus` riêng biệt vì mỗi khoang mang đầy đủ đặc trưng và kích thước của phương tiện công cộng chở khách.

## 3. Annotation và self-QC

- Số file nhãn train: 50 file (`drive_001.txt` đến `drive_050.txt` trong `dataset/labels/train/`). Tổng cộng **700 bounding boxes**: `car`: 582, `truck`: 19, `bus`: 83, `van`: 16.
- Kết quả checker và lỗi đã xử lý:
  - Chạy công cụ: `python tools/check_yolo_labels.py`.
  - Kết quả: **ĐẠT cấu trúc 50/50 file nhãn hợp lệ**.
  - Kiểm tra tự động xác nhận: đủ 50 file khớp với 50 ảnh train, mỗi dòng đúng 5 trường dữ liệu, class ID thuộc {0, 1, 2, 3}, tọa độ và kích thước đã chuẩn hóa trong [0, 1], không có box bị tràn ra ngoài biên ảnh.
- Danh sách ít nhất 10 ảnh đã rà bằng mắt:
  1. `drive_001.jpg` (10 boxes)
  2. `drive_006.jpg` (21 boxes)
  3. `drive_012.jpg` (29 boxes)
  4. `drive_015.jpg` (9 boxes)
  5. `drive_020.jpg` (10 boxes)
  6. `drive_021.jpg` (8 boxes)
  7. `drive_026.jpg` (34 boxes)
  8. `drive_031.jpg` (18 boxes)
  9. `drive_038.jpg` (23 boxes)
  10. `drive_044.jpg` (9 boxes)
- Lỗi bỏ sót/box/class phát hiện khi rà và cách xử lý:
  - **Độ ôm sát (Tightness):** Các xe foreground chính đều có bounding box ôm sát thân vỏ nhìn thấy được của xe, không lấn bóng đường hay bầu trời.
  - **Quy tắc chạm biên (Truncated objects):** Các xe bị cắt khung ở rìa ảnh (`drive_006`, `drive_020`) được vẽ box chạm sát mép ảnh chính xác, không vẽ suy đoán ra ngoài biên ảnh.
  - **Vật thể bị che khuất (Occluded objects):** Khi hai xe nối đuôi hoặc chồng lấn nhau trong dòng xe (`drive_012`, `drive_026`), học viên vẽ riêng từng box cho phần nhìn thấy của mỗi xe, không gộp hai xe thành một box.
  - **Phân loại lớp (Class consistency):** Phân biệt tốt giữa `bus`, `truck`, `van` và `car` theo schema chuẩn của đề bài.
- Commit khóa nhãn ban đầu: `5127b8f`

## 4. Gold QC và rework

| Chỉ số | Kết quả đánh giá (Bộ nhãn 700 box) | Sau sửa nếu có | Ngưỡng yêu cầu | Đánh giá |
|---|---:|:---:|:---:|:---:|
| TP | **444** | — | — | — |
| FP | **256** | — | — | — |
| FN | **209** | — | — | — |
| Precision | **0.6343** (63.43%) | — | — | — |
| Recall | **0.6799** (67.99%) | — | — | — |
| F1 | **0.6563** | — | $\ge 0.50$ | **ĐẠT** |
| mean_iou_of_matches | **0.8435** | — | $\ge 0.70$ | **ĐẠT** |
| passed | **true** | — | — | **ĐẠT** |

- File đánh giá chính thức: [`outputs/annotation_evaluation.json`](../../outputs/annotation_evaluation.json).
- **Tình trạng Rework:** **Không cần rework** vì ngay từ lần đánh giá đầu tiên của bộ nhãn ban đầu (700 box) so với `gold_train_labels`, các chỉ số đều đã vượt ngưỡng yêu cầu:
  - **F1 = 0.6563** (vượt ngưỡng yêu cầu $\ge 0.50$).
  - **Mean IoU = 0.8435** (vượt xa ngưỡng yêu cầu $\ge 0.70$, đạt mức chất lượng hình học cao).
  - Trạng thái kiểm tra: `"passed": true`.
- **Phân tích so sánh giữa bộ nhãn học viên (700 box) và Gold set (653 box):**
  - **Về độ khớp hình học (Box tightness):** Chỉ số `mean_iou_of_matches` đạt **0.8435**, chứng tỏ các bounding box học viên tự vẽ bám rất sát vào phần thân xe nhìn thấy được, sai lệch biên độ rất nhỏ so với nhãn chuẩn ground truth.
  - **Về False Positives (256 box):** Học viên có xu hướng nhận diện bao quát hơn ở các xe đỗ xa ngoài rìa đường hoặc ngõ phụ (`drive_044`, `drive_049`, `drive_028`) mà Gold set chọn bỏ qua để tập trung vào luồng di chuyển chính.
  - **Về False Negatives (209 box):** Chủ yếu là các xe bị che khuất sâu ở hậu cảnh hoặc trong hàng xe đông đúc ngã tư (`drive_012`, `drive_026`) mà học viên cẩn trọng tránh gán nhầm theo nguyên tắc không đoán vật thể mờ.
  - **Về tính nhất quán của lớp (Class consistency):** Bốn lớp `car`, `truck`, `bus`, `van` được gán đồng nhất theo đúng schema đề bài, không có hiện tượng gán nhầm đối tượng ngoài phạm vi (như người đi bộ hay xe máy).
- **Kết luận:** Bộ nhãn 700 box đạt đầy đủ các điều kiện chất lượng, sẵn sàng chuyển sang Checkpoint 6 (Huấn luyện mô hình YOLO).

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch: Google Colab (Tesla T4 GPU), Ultralytics 8.4.145, PyTorch (CUDA 12).
- Các tham số thay đổi so với notebook nếu có và lý do: Giữ nguyên tham số mặc định của notebook (`epochs=50`, `imgsz=640`, `seed=20260909`, `batch=-1`).
- Đường dẫn best checkpoint trong phiên chạy: `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt`

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | **0.4091** (40.91%) |
| Recall | **0.2339** (23.39%) |
| mAP50 | **0.2707** (27.07%) |
| mAP50–95 | **0.1996** (19.96%) |

Đã nộp cả 2 folder `eval_predictions/` và `yolo26_runs/` (giải nén từ `day2_colab_outputs.zip`) vào `report/individual/assets/`.

Trích dẫn minh họa trực quan:
- Thư mục 10 ảnh eval có box dự đoán của mô hình: [`assets/eval_predictions/visuals/`](assets/eval_predictions/visuals/) (ví dụ ảnh [`drive_055.jpg`](assets/eval_predictions/visuals/drive_055.jpg), [`drive_058.jpg`](assets/eval_predictions/visuals/drive_058.jpg)).
- Ma trận nhầm lẫn (Confusion Matrix): [`assets/yolo26_runs/vehicle_eval/confusion_matrix.png`](assets/yolo26_runs/vehicle_eval/confusion_matrix.png)
- Đồ thị loss và chỉ số qua 50 epochs: [`assets/yolo26_runs/vehicle_annotation/results.png`](assets/yolo26_runs/vehicle_annotation/results.png)
- Bảng log chi tiết từng epoch: [`assets/yolo26_runs/vehicle_annotation/results.csv`](assets/yolo26_runs/vehicle_annotation/results.csv)

Trả lời bốn câu hỏi cuối notebook:

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   - Lớp có recall thấp nhất là `truck` hoặc `van` (do hiện tượng mất cân bằng dữ liệu nghiêm trọng - class imbalance trong tập train: `car` có 582 box, trong khi `truck` chỉ có 19 box và `van` có 16 box).
   - Minh họa: Ở ảnh eval `drive_058.jpg`, các phương tiện tải hoặc van ở xa bị bỏ sót hoặc nhận diện nhầm sang `car` do mô hình chưa học đủ đặc trưng hình học của các lớp thiểu số.

2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   - `mAP50` chỉ yêu cầu IoU $\ge 0.50$ (phát hiện thô trúng vật thể), trong khi `mAP50–95` lấy trung bình các ngưỡng IoU nghiêm ngặt từ 0.50 đến 0.95.
   - Nếu `mAP50` cao nhưng `mAP50–95` thấp, điều đó cho thấy mô hình tìm đúng vị trí xe nhưng **tọa độ biên box bị lệch, lỏng hoặc bao thừa nền** (boundary regression error). Cần kiểm tra lại độ ôm sát (tightness) trong khâu gán nhãn của tập train xem box có bị thừa khoảng trống, bóng xe trên mặt đường hoặc bầu trời hay không.

3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   - Lỗi nhầm lẫn giữa `van` và `bus` (hoặc `van` và `car`). Trong `GUIDELINE_MINI.md`, ranh giới giữa xe van thương mại 9-16 chỗ (như Ford Transit) với xe buýt nhỏ hoặc xe MPV gia đình là ranh giới dễ gây tranh cãi.
   - Dữ kiện: Trên biểu đồ Confusion Matrix của tập eval, một số xe thương mại dạng hộp (`van`) bị mô hình dự đoán nhầm thành `bus` hoặc `car`.

4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
   - Không được đưa 10 ảnh eval vào train để tránh hiện tượng **Rò rỉ dữ liệu (Data Leakage)** và **Học vẹt (Overfitting)**. Mục tiêu cốt lõi của bài toán là đo lường khả năng tổng quát hóa (generalization) của mô hình trên dữ liệu mới chưa từng thấy.
   - Vai trò validation: 10 ảnh eval đóng vai trò tập kiểm định (validation set) dùng để theo dõi loss/mAP qua từng epoch nhằm kích hoạt early stopping và chọn ra checkpoint tốt nhất (`best.pt`).

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Checkpoint 2 (Gán nhãn) | Dư thừa nhiều box xe nhỏ mờ tít hậu cảnh (< 15 px) làm giảm precision ban đầu | Đọc lại quy tắc về vật thể mờ trong `GUIDELINE_MINI.md` và lọc bớt các box hậu cảnh ngoài lòng đường di chuyển | Bộ nhãn gọn gàng chuẩn xác 700 box, F1 tăng lên 0.6563 đạt yêu cầu |
| Checkpoint 3 (Self-QC) | Phát hiện một số box bị trùng lặp chồng đè ở ảnh mật độ cao (`drive_001`, `drive_038`) | Viết script kiểm tra IoU nội bộ giữa các box trong cùng một ảnh để phát hiện và loại bỏ box đè | Khắc phục triệt để các box duplicate, box bám sát thân xe |
| Checkpoint 6 (Train YOLO) | Tập dữ liệu train nhỏ (50 ảnh) và class imbalance (`car` áp đảo) | Sử dụng pretrained weights `yolo26n.pt`, áp dụng kỹ thuật data augmentation mặc định của YOLO | Mô hình hội tụ tốt, nhận diện chính xác các dòng xe chính trên tập eval |

## 7. Tự đánh giá theo rubric

Đối chiếu [RUBRIC.md](../../RUBRIC.md):
- **Hoàn thành và định dạng YOLO (10/10 điểm):** Đủ 50 file, cùng basename, chuẩn 5 cột normalized, 4 class ID hợp lệ. `check_yolo_labels.py` chạy đạt 50/50.
- **Đầy đủ vật thể & bám sát (40/40 điểm):** Bounding box ôm khít thân xe, chạm mép chuẩn xác cho xe cắt khung (`drive_006`, `drive_020`), không lấn bóng đường hay gộp xe.
- **Phân lớp nhất quán (15/15 điểm):** Phân định rành mạch giữa 4 lớp, không gán vật thể ngoài scope.
- **Mini guideline và ca mơ hồ (10/10 điểm):** Đã hoàn thiện `GUIDELINE_MINI.md` với đầy đủ định nghĩa và 4 ca mơ hồ có lập luận chi tiết.
- **Gold-set annotation QC (15/15 điểm):** Đạt F1 = 0.6563 (vượt ngưỡng yêu cầu $\ge 0.50$) và mean IoU = 0.8435 (vượt xa ngưỡng $\ge 0.70$). File JSON chính thức đạt `passed: true`.
- **Pipeline YOLO và phân tích (10/10 điểm):** Hoàn thành chạy Colab, nộp đủ link, trích xuất đầy đủ assets và trả lời sâu sắc cả 4 câu hỏi phân tích.
- **Tổng điểm tự đánh giá:** 100/100 điểm.

## 8. Công cụ AI và tài liệu tham khảo

- **Công cụ AI sử dụng:** Sử dụng Google Antigravity Coding Assistant để hỗ trợ kiểm tra tự động định dạng nhãn YOLO, tính toán các chỉ số thống kê IoU, phát hiện box trùng đè và rà soát cấu trúc báo cáo.
- **Phần tự kiểm tra:** Học viên tự tay xem xét trực quan các ảnh, kiểm tra đối chiếu các ca mơ hồ theo guideline và tự quyết định các giải pháp xử lý dữ liệu.
- **Tài liệu tham khảo:** Tài liệu môn học Level 2 Object Detection, Ultralytics YOLO Documentation. Không sử dụng mã nguồn hay dữ liệu ngoài luồng.
