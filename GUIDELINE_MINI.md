# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Các phương tiện cơ giới cá nhân cỡ nhỏ và vừa phục vụ chở người hoặc chở hàng nhẹ dưới 9 chỗ ngồi; bao gồm xe con (sedan, hatchback, coupe), SUV, MPV gia đình nhỏ, taxi, và xe bán tải (pickup truck). Xe đang đỗ hay đang chạy đều tính là `car`. | Dáng xe gầm thấp hoặc SUV 5-7 chỗ; xe bán tải có cabin nhỏ và thùng hàng hở phía sau vẫn gán là `car` (không gán `truck`). |
| `truck` | Xe tải chở hàng hạng trung và hạng nặng, xe tải thùng bệ, xe đầu kéo (tractor-trailer), xe bồn, container, xe ben chuyên dụng chở vật liệu/hàng hóa thương mại. | Thùng xe tách biệt với cabin lái hoặc xe có cấu trúc khung gầm tải lớn; không gán cho xe bán tải / cabin nhỏ. |
| `bus` | Phương tiện công cộng hoặc thương mại chuyên chở nhiều hành khách (xe buýt nội đô, xe khách liên tỉnh, xe đưa đón học sinh/cán bộ) thường từ 16 chỗ ngồi trở lên. | Thân xe dài, cao, nhiều hàng ghế và nhiều ô cửa sổ dọc hai bên thân; kích thước lớn hơn đáng kể so với minivan/van. |
| `van` | Xe van, minivan dạng thân hộp liền khối (box-shaped), xe thương mại cỡ vừa dùng chở bưu phẩm/hàng hóa hoặc chở khách 9–16 chỗ (như Ford Transit, Toyota HiAce, MPV thương mại cỡ lớn). | Thân xe hình hộp, cao hơn sedan/wagon nhưng ngắn và nhỏ hơn xe buýt lớn; không có thùng xe tách rời như xe tải. |

## Quy tắc chung

- **Box bám sát vật thể nghĩa là:** Bounding box phải ôm khít phần thân vỏ nhìn thấy được của xe; không bao gồm bóng xe đổ trên mặt đường, khoảng trống mặt đường, bầu trời hay vật thể bên cạnh; không gộp nhiều xe lân cận vào chung một box.
- **Vật thể bị che khuất:** Nếu xe bị che khuất một phần bởi xe khác, rào chắn hoặc công trình giao thông, vẫn vẽ bounding box quanh toàn bộ phần nhìn thấy được nếu nhận diện rõ đặc trưng thuộc lớp; khi 2 xe chồng lấn nhau, vẽ 2 box riêng biệt cho từng xe.
- **Vật thể bị cắt bởi rìa ảnh:** Vẽ box chạm sát đường biên của ảnh (tọa độ giới hạn trong khoảng [0, 1] hoặc [0, 640]); tuyệt đối không suy đoán phần thân xe bị khuất ngoài khung hình.
- **Vật thể rất nhỏ/mơ hồ:** Ở khoảng cách rất xa (hậu cảnh), nếu vật thể chỉ còn vài pixel mờ nhòe không đủ thông tin hình khối để xác định chính xác một trong 4 lớp của schema thì không tự đoán gán nhãn; các đối tượng ngoài 4 lớp (người đi bộ, xe máy, xe đạp, biển báo, đèn giao thông) không được gán nhãn.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_006.jpg` | Phương tiện màu trắng/xám ở phía gần dưới góc bên trái chỉ hiện ra phần đèn và một chút bánh xe. | Gán `car` | Kích thước của bánh xe nhỏ phù hợp với `car`, nếu nó là `truck` thì không thể thấp được như vậy, `van` không phải bởi vì thường xe van đầu xe không lồi ra nhuhw thế. |
| `drive_020.jpg` | Xe ô tô ở góc dưới cùng bên phải khung hình chỉ lộ ra phần đuôi và đèn hậu | Gán `car` | Xe bị mép ảnh cắt ngang làm mất phần lớn thân xe. Tuy nhiên, chi tiết đèn hậu, biển số và nắp cốp sau vẫn đủ đặc trưng để xác định là xe con. |
| `drive_021.jpg` | Xe ô tô ở phía sau điểm dừng xe bus bị che khuất hơn 85% | Không gán nhãn | Theo schema, đối tượng cắt bởi rìa ảnh (truncated object) nếu không có đủ những đặc trưng thì không đánh nhãn. |
| `drive_026.jpg` | Trong ảnh, các xe bus nối đuôi nhau là trường hợp khá mơ hồ vì nhìn tương tự như một chiếc bus điện có khung kéo | Gán `bus` riêng lẻ cho từng phần | Vì tất cả đều mang đặc trưng của xe `bus` nên cần được đánh nhãn tách rời trong tệp dữ liệu này |

