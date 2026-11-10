# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Bùi Kim Trãi
- MSSV: 2A202603176
- Email: buikimtrai2k5@gmail.com
- Lớp: L2A
- Link repo bài làm: https://github.com/KtraiVla/K5-L2-Day02-BuiKimTrai-2A202603176-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/drive/1ZR78BW2o_4ADY-ZNB3Ez3JBrRxg956WR?usp=sharing

## 2. Schema và mini guideline

Dẫn tới [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). Tóm tắt ba ca mơ hồ đã quyết định:
1. **`drive_001`**: Dãy xe đỗ trong bãi đỗ riêng bên trái ngoài lề đường được phân cách bằng cây xanh và dải phân cách $\rightarrow$ Quyết định không gán do nằm ngoài vùng quan tâm (Region of Interest - ROI) của làn đường lưu thông chính.
2. **`drive_042`**: Chiếc xe khách lớn màu xanh dương và xe tải chở bao hàng: Mắt nhìn trực quan rõ ràng là `bus` và `truck`, nhưng ground truth bị hoán đổi nhãn (xe buýt gán ID 3 `van`, xe tải gán ID 2 `bus`) $\rightarrow$ Ghi nhận sự bất nhất này giữa hình ảnh thực tế và schema ground truth, đồng bộ theo ground truth trong vòng rework theo đúng hướng dẫn ca mơ hồ của lab.
3. **`drive_002`**: Hàng loạt xe nhỏ li ti ở đường chân trời phía sau cầu vượt $\rightarrow$ Quyết định dừng không gán các xe quá nhỏ (< 15px) và mờ nhạt để tránh gây nhiễu cho mô hình.

## 3. Annotation và self-QC

- Số file nhãn train: 50 file (`drive_001.txt` đến `drive_050.txt`).
- Kết quả checker và lỗi đã xử lý: Chạy qua `tools/check_yolo_labels.py` đạt 50/50 file hợp lệ về định dạng YOLO (5 cột, normalized trong khoảng [0, 1], ID lớp từ 0 đến 3).
- Danh sách ít nhất 10 ảnh đã rà bằng mắt: `drive_001`, `drive_002`, `drive_003`, `drive_009`, `drive_010`, `drive_013`, `drive_017`, `drive_018`, `drive_028`, `drive_042`, `drive_049`.
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa: Phát hiện ban đầu gán quá nhiều xe ở bãi đỗ bên lề (như `drive_001`) và xe quá nhỏ ở hậu cảnh (như `drive_002`, `drive_049`); đã tiến hành xóa bỏ hơn 400 box thừa, căn chỉnh lại viền ôm sát mép thân xe.
- Commit khóa nhãn ban đầu: `77cba26`

## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | 471 | 485 |
| FP | 627 | 210 |
| FN | 182 | 168 |
| Precision | 0.4290 | 0.6978 |
| Recall | 0.7213 | 0.7427 |
| F1 | 0.5380 | 0.7196 |
| mean_iou_of_matches | 0.8305 | 0.8326 |
| passed | false | false (F1 đạt 0.72, tiếp tục giải thích rõ vòng rework) |

Dẫn tới `outputs/annotation_evaluation.json` trong repo.

### Phân tích và quyết định Rework:
1. **Khắc phục lỗi Over-labeling (FP quá cao):**
   - Ban đầu có tới 627 box thừa (FP) khiến Precision bị kéo tụt xuống 42.9%. Nguyên nhân là do gán cả các xe trong bãi đỗ tư nhân ngoài lề đường (`drive_001`) và các xe li ti ở chân trời (`drive_002`, `drive_049`).
   - Trong vòng rework, sinh viên đã rà soát và xóa bỏ hơn 400 box thừa, đưa FP giảm mạnh từ 627 xuống 210 box. Kết quả: **Precision tăng vọt từ 42.9% lên gần 70% (0.6978)** và **F1 tăng mạnh từ 0.538 lên 0.7196**.
2. **Khắc phục lỗi Class Mismatch (Hoán đổi nhãn trong dataset):**
   - Tại ảnh `drive_042`, sinh viên phát hiện xe khách thực tế mắt nhìn là `bus` nhưng ground truth lại gán là `van` (ID 3), xe tải chở hàng gán là `bus` (ID 2). Hiện tượng này cũng xảy ra tương tự tại một số ảnh khác như `drive_009`, `drive_019`, `drive_026`.
   - Sinh viên đã tạo công cụ trực quan hóa so sánh (`visualize_compare`) để đối chiếu vị trí box và ghi nhận hiện tượng này vào mini guideline.
3. **Độ bám sát box:** Chỉ số Mean IoU đạt **0.8326** ($\ge 0.80$), chứng tỏ kỹ thuật vẽ box ôm rất sát thân xe, đạt mức chất lượng Xuất sắc về mặt hình học.

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch: Google Colab GPU Tesla T4 (device "0"), PyTorch 2.5+, Ultralytics phiên bản mới nhất. Huấn luyện mô hình `yolo26n.pt` với 50 epochs, imgsz 640, batch -1, seed 20260909.
- Các tham số thay đổi so với notebook nếu có và lý do: Giữ nguyên cấu hình chuẩn theo hướng dẫn của notebook.
- Đường dẫn best checkpoint trong phiên chạy: `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt`

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.1938 |
| Recall | 0.4401 |
| mAP50 | 0.3536 |
| mAP50–95 | 0.2009 |

Đường dẫn tài nguyên minh chứng (dẫn link tương đối cạnh báo cáo):
- Thư mục ảnh dự đoán: [assets/eval_predictions/visuals/drive_051.jpg](assets/eval_predictions/visuals/drive_051.jpg), [assets/eval_predictions/visuals/drive_052.jpg](assets/eval_predictions/visuals/drive_052.jpg), [assets/eval_predictions/visuals/drive_055.jpg](assets/eval_predictions/visuals/drive_055.jpg)...
- Log huấn luyện và ma trận nhầm lẫn: [assets/yolo26_runs/vehicle_annotation/results.csv](assets/yolo26_runs/vehicle_annotation/results.csv), [assets/yolo26_runs/vehicle_annotation/confusion_matrix.png](assets/yolo26_runs/vehicle_annotation/confusion_matrix.png).

### Trả lời bốn câu hỏi cuối notebook:

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   - Lớp `truck` và `bus` có recall thấp nhất. Nguyên nhân do hiện tượng mất cân bằng số lượng lớp (class imbalance) trầm trọng trong tập dữ liệu train 50 ảnh: lớp `car` chiếm đại đa số (> 80%), trong khi `truck` và `bus` chỉ xuất hiện rất ít. Ảnh minh họa tại [assets/eval_predictions/visuals/drive_052.jpg](assets/eval_predictions/visuals/drive_052.jpg): chiếc xe buýt lớn và xe tải ở hậu cảnh bị mô hình bỏ sót hoặc nhận diện nhầm sang `car`.
2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   - Cần kiểm tra **độ khít (tightness)** và độ chính xác hình học của bounding box trong tập nhãn. mAP50 chỉ đòi hỏi độ phủ IoU $\ge 0.50$ (đã được tính là phát hiện đúng), trong khi mAP50–95 tính trung bình trên các ngưỡng IoU khắt khe từ 0.50 đến 0.95 (bước nhảy 0.05). Nếu mAP50 cao mà mAP50–95 thấp, chứng tỏ mô hình định vị đúng đối tượng nhưng viền box bị lỏng, lem ra nền hoặc lệch tâm.
3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   - Lỗi nhầm lẫn giữa các lớp xe to: `bus`, `truck` và `van`. Dữ kiện hỗ trợ: Ma trận nhầm lẫn (`confusion_matrix.png`) trên tập eval cho thấy dự đoán có sự giao thoa nhầm lẫn giữa các lớp này; đồng thời trong quá trình kiểm tra nhãn ground truth tại `drive_042` và `drive_009`, sinh viên phát hiện xe khách thực tế bị gán nhãn `van` (ID 3) và xe tải thùng bị gán nhãn `bus` (ID 2). Ranh giới định nghĩa về hình dáng xe giữa guideline và nhãn gốc của bộ dữ liệu còn có độ lệch.
4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
   - Không được dùng 10 ảnh eval để train vì điều đó sẽ gây ra hiện tượng rò rỉ dữ liệu (data leakage), mô hình sẽ học vẹt (overfit) trên các ảnh này và chỉ số đánh giá không còn khách quan. Vai trò của 10 ảnh eval là tập kiểm thử độc lập (validation) dùng để giám sát quá trình huấn luyện qua từng epoch, đánh giá khả năng tổng quát hóa (generalization) của mô hình trên dữ liệu mới chưa từng thấy và lựa chọn ra checkpoint tốt nhất (`best.pt`).

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Chấm nhãn đợt 1 (Gold QC) | F1 chỉ đạt 0.538 do False Positive quá cao (FP = 627) vì vẽ cả xe trong bãi đỗ bên lề và xe ở chân trời. | Xác định lại vùng quan tâm (ROI), rà soát trên CVAT và xóa bỏ hơn 400 box xe thừa ở ngoài làn đường chính. | FP giảm từ 627 xuống 210, Precision tăng từ 42.9% lên 69.8%, F1 tăng lên gần 0.72. |
| Phân loại xe to (`drive_042`) | Trực quan là xe khách lớn và xe tải chở bao, nhưng ground truth gán nhãn lệch thành van và bus. | Viết script hiển thị trực quan so sánh (visualize compare), ghi nhận ca mơ hồ theo guideline và đồng bộ theo ground truth trong rework. | Hiểu rõ bản chất dữ liệu, giải thích thuyết phục trong báo cáo. |

## 7. Tự đánh giá theo rubric

Đối chiếu [RUBRIC.md](../../RUBRIC.md):
- **Hoàn thành và định dạng YOLO (10đ):** Đạt 50/50 file nhãn, 5 cột normalized, ID đúng.
- **Đầy đủ vật thể (20đ):** Đã bao phủ các xe lưu thông trên làn đường chính, kể cả xe bị che khuất.
- **Độ bám sát và hình học box (20đ):** Mean IoU đạt 0.8326, đạt mức chất lượng Xuất sắc (> 0.80).
- **Phân lớp nhất quán (15đ):** Đã phân biệt 4 lớp theo schema và xử lý các ca hoán đổi.
- **Mini guideline và quyết định mơ hồ (10đ):** Đã hoàn thiện định nghĩa và 3 ca mơ hồ cụ thể trong `GUIDELINE_MINI.md`.
- **Gold-set annotation QC (15đ):** Có đầy đủ bảng số liệu trước/sau và phân tích vòng rework chi tiết.
- **Pipeline YOLO và phân tích (10đ):** Chạy Colab thành công, có đầy đủ link Colab, assets dự đoán, bảng chỉ số và trả lời đầy đủ 4 câu hỏi phân tích.

## 8. Công cụ AI và tài liệu tham khảo

- Sử dụng Google DeepMind Antigravity Assistant hỗ trợ phân tích ma trận lỗi FP/FN từ file evaluation JSON, viết script kiểm tra trực quan so sánh nhãn (`visualize_compare`) để phát hiện vùng quan tâm (ROI) và lỗi lệch nhãn xe to. Tự tay thực hiện toàn bộ việc rà soát, chỉnh sửa nhãn trên CVAT và chạy Colab.
- Tham khảo tài liệu Ultralytics YOLO26 và hướng dẫn thực hành của giảng viên.
