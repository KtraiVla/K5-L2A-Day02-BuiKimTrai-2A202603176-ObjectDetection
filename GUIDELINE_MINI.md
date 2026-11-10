# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Xe ô tô con từ 4 đến 7 chỗ phục vụ cá nhân hoặc gia đình (sedan, hatchback, SUV, crossover). | Kích thước nhỏ gọn/trung bình, không có thùng chở hàng rời, trần xe liền mạch. |
| `truck` | Xe tải chở hàng mọi kích cỡ (xe tải nhẹ, xe tải nặng, xe đầu kéo container). | Có cabin lái phía trước và thùng hàng (kín hoặc bạt) riêng biệt phía sau. |
| `bus` | Xe buýt chở khách công cộng hoặc xe khách đường dài cỡ lớn. | Thân xe dài, kích thước lớn, nhiều cửa sổ dọc thân xe, chở nhiều hành khách (> 16 chỗ). |
| `van` | Xe tải nhỏ / xe bán tải / xe chở khách cỡ trung (minivan, van chuyển phát, xe 9-16 chỗ). | Thân xe dạng hộp liền khối giữa khoang lái và khoang hàng/ghế khách, thường có cửa trượt bên hông (ví dụ Ford Transit, Solati, Wuling). |

## Quy tắc chung

- **Box bám sát vật thể nghĩa là:** Bounding box phải ôm sát đường viền nhìn thấy được của xe, chạm vào mép ngoài cùng của bánh xe, nóc và cản xe; không lấy thêm khoảng trống mặt đường, bầu trời hay xe bên cạnh.
- **Vật thể bị che khuất:** Nếu xe bị cây cối, cột điện, dải phân cách hoặc xe khác che một phần nhưng vẫn nhận diện được loại xe, vẽ box bao quanh toàn bộ phần nhìn thấy được của xe đó.
- **Vật thể bị cắt bởi rìa ảnh:** Kéo box chạm sát mép ảnh (x/y = 0 hoặc 1); tuyệt đối không tự ý vẽ tưởng tượng phần xe nằm ngoài khung hình.
- **Vật thể rất nhỏ/mơ hồ:** Những xe ở quá xa (đường chân trời), kích thước nhỏ hơn $15 \times 15$ pixel, bị mờ nhoè nặng hoặc nằm ngoài luồng giao thông chính (trong bãi đỗ xe riêng biệt ngoài lề) thì không gán box để tránh gây nhiễu cho mô hình.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_001` | Dãy xe đỗ phía sau dải phân cách và vỉa hè bên trái (khu vực bãi đỗ riêng ngoài luồng đường giao thông). | Không gán các xe trong bãi | Các xe này nằm ngoài vùng đường lưu thông chính (Region of Interest - ROI), bị vật cản và cây che khuất nhiều; chỉ tập trung vào các xe đang lưu thông trên làn đường. |
| `drive_042` | Xe khách lớn màu xanh dương và xe tải chở bao hàng: Mắt nhìn trực quan rõ ràng là `bus` và `truck`, nhưng ground truth bị hoán đổi (xe buýt gán ID 3, xe tải gán ID 2). | Ghi nhận sự bất nhất và đồng bộ theo ground truth trong vòng rework | Phát hiện sự hoán đổi nhãn trong dataset (cyclic shift giữa bus/truck/van). Sinh viên ghi nhận ca mơ hồ theo đúng hướng dẫn của lab để giải thích trong báo cáo. |
| `drive_002` | Hàng loạt xe ở tít xa gần chân cầu vượt/đường chân trời có kích thước cực nhỏ và mờ nhạt. | Dừng không gán các xe quá xa sau vạch cầu vượt | Vật thể quá nhỏ (< 15px) không phân biệt được rõ các đặc trưng của car hay van; tuân thủ quy tắc không tự đoán. |
