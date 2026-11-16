# Guideline cá nhân Day 07 — 057

- Phiên bản: đề xuất cá nhân, chuẩn bị từ bản nháp có AI hỗ trợ để 057 rà lại.
- Dữ liệu: 40 ảnh trong `data/images/`. Công cụ: CVAT, Rectangle, xuất COCO 1.0.
- Phạm vi: quyết định lớp, cách vẽ hộp và cách xử lý các ca mơ hồ trước khi nhóm khóa v1.0.

## 1. Mục tiêu

Gán box cho người và phương tiện thật có thể nhận diện trong ảnh đường phố, phục vụ mô hình phát hiện vật thể 2D. Cần tránh bỏ sót vật thật, gán nhầm vật giả và kéo box sang người, xe hoặc hàng hóa bên cạnh.

## 2. Lớp và thuộc tính

**R-01 — Mỗi đối tượng thật đủ điều kiện nhận một lớp.** Nhận diện theo hình dáng nhìn thấy; không đoán theo biển số, hãng, người lái hay mục đích chuyến đi.

| Lớp | Dấu hiệu để chọn | Ví dụ và phản ví dụ |
|---|---|---|
| `person` | Người thật, kể cả người ngồi hoặc đứng trên xe, nếu phần thấy được nhận ra và đủ chiều cao. | Người đi bộ ở `img_034`; loại ma-nơ-canh ở `img_033`. |
| `car` | Ô tô chở người dạng sedan, hatchback, wagon hoặc SUV, không có thùng hàng tách khỏi cabin và không có dáng thân van. | Sedan bên trái ở `img_023`; không dùng cho pickup. |
| `truck` | Xe tải, đầu kéo, pickup hoặc xe có thùng/sàn chở hàng tách biệt khỏi cabin. Nắp thùng pickup không làm đổi lớp. | Pickup tiền cảnh ở `img_001`; xe tải nhỏ cabin xanh ở `img_036`. |
| `van` | Xe thân liền khối, khoang sau dạng hộp cao và nối liền với cabin; có thể có cửa kính chở khách hoặc vách kín chở hàng. | Xe thân hộp bạc ở `img_005`; loại bus/minibus và xe tải cabin riêng. |
| `motorcycle` | Xe máy hoặc mô tô hai bánh, kể cả scooter; mô tô gắn sidecar vẫn là mô tô. | Scooter ở `img_006`; không dùng cho xe đạp, xích lô và tuk-tuk. |
| `ignore_region` | Rectangle đánh dấu vùng có vật thuộc lớp mục tiêu nhưng quá nhỏ hoặc tụ dày đến mức không thể tách từng vật. | Chỉ khoanh vùng cần bỏ qua; không dùng để đánh dấu mọi vật ngoài taxonomy. |

**R-02 — Thuộc tính áp dụng cho năm lớp đối tượng; `ignore_region` không có thuộc tính.**

| Thuộc tính | Cách điền |
|---|---|
| `occluded_level` | Ước lượng tỷ lệ phần đối tượng **nằm trong khung** bị vật khác che theo diện tích bóng dáng, làm tròn nếu cần: `0` = 0%; `1` = trên 0% đến dưới 25%; `2` = từ 25% đến dưới 60%; `3` = từ 60% trở lên nhưng vẫn nhận ra đối tượng. Chỉ dùng ước lượng này để chọn thuộc tính, không để nới box. |
| `truncated` | `yes` nếu vật kéo dài ra ngoài mép ảnh; `no` nếu toàn bộ vật ở trong khung hoặc chỉ bị vật khác che. Chạm mép nhưng vẫn thấy vật kết thúc trong khung thì `no`. Có thể vừa bị che vừa bị cắt. |
| `can_review` | Tick khi có đủ căn cứ để vẽ box nhưng còn nghi ngờ lớp, ranh box hoặc mức che. Nếu chưa có đủ căn cứ để vẽ box, ghi ID ảnh, vị trí và lý do vào ghi chú cấp ảnh. |

Dùng thuộc tính `occluded_level` tự tạo của Project CVAT, không thay bằng cờ Occluded mặc định của CVAT.

## 3. Quy tắc vẽ box

- **R-03 — Ôm sát phần nhìn thấy.** Vẽ hình chữ nhật nhỏ nhất bao phần nhìn thấy của cùng một đối tượng trong ảnh gốc. Nếu các phần nhìn thấy bị ngăn cách bởi vật che, dùng một box bao các phần đó. Không kéo box theo phần bị che hoặc đã ra khỏi ảnh. Tính bộ phận gắn liền như gương xe và sidecar; loại bóng, phản chiếu, vệt nhòe và hàng hóa. Với người, tính quần áo, giày và mũ đang đội; không tính đồ cầm tay.
- **R-04 — Tách người khỏi xe và hàng khỏi xe.** Mỗi người trên xe có một `person` riêng nếu đủ điều kiện; bản thân xe có một box. Các box có thể chồng lên nhau. Xe đang được vận chuyển vẫn là đối tượng riêng nếu là xe thật và đủ điều kiện. Box xe vận chuyển gồm cabin, khung và rơ-moóc đang nối, không mở rộng để bao xe chở trên sàn.
- **R-05 — Đo trên ảnh gốc.** Nếu box phần nhìn thấy cao **dưới 20 px**, không gán box đối tượng riêng; khoanh `ignore_region` sát vật hoặc cụm vật quá nhỏ. Từ **20 px trở lên**, gán nếu xác định được lớp và ranh giới. Với đám đông, tách những cá thể đủ rõ và chỉ khoanh phần không thể tách; tránh phủ lên vật đã gán được.
- **R-06 — Chỉ gán vật thật thuộc taxonomy.** Loại tượng, hình in, đồ chơi, ma-nơ-canh, bóng và phản chiếu. Một xe thật nhìn xuyên qua kính vẫn có thể gán; ảnh phản chiếu của nó không tạo box thứ hai. Không gán phần xe đang chụp từ bên trong (ví dụ cửa hay gương xe của camera). Xe ngoài taxonomy như xe đạp, xích lô, tuk-tuk và bus không có box xe; người thật đi trên đó vẫn được xét riêng.
- **R-07 — Vật mờ, khuất nhiều hoặc nghi là mô hình.** Khi nhận diện được vật và giới hạn phần thấy, gán theo quy tắc thường và bật `can_review` nếu chưa chắc. Nếu chỉ có vệt mờ hoặc không đủ căn cứ phân biệt vật thật với mô hình, ghi vùng cần hỏi reviewer và không đoán lớp. Chỉ dùng `ignore_region` theo R-05.

## 4. Ca khó để nhóm thử

Các ví dụ dưới đây mô tả cách áp dụng đề xuất trên một vùng cụ thể, không phải bộ nhãn cho toàn ảnh.

| Ảnh | Vùng cần xem | Quy tắc và cách áp dụng dự kiến |
|---|---|---|
| `img_006` | Xe máy bên trái có người lái và người ngồi sau | **R-04:** một `motorcycle` và hai `person` nếu cả hai người nhận diện được và đạt ngưỡng. Không gộp thành một box. Người đi xe đạp bên phải vẫn có thể là `person`. |
| `img_002` | Xe tải chở nhiều ô tô | **R-04:** một `truck` cho cụm cabin và rơ-moóc đang nối; xét từng xe được chở riêng. Kiểm lại lớp từng xe theo dấu hiệu thực thấy và ghi mức che khi khung sàn che thân xe. |
| `img_004`, `img_016` | Tuk-tuk | **R-06:** đề xuất không gán box cho tuk-tuk. Ở `img_016`, scooter xanh và người lái được xét riêng theo R-04. |
| `img_015`, `img_020` | Ô tô ở mép ảnh và cảnh trong gương | **R-02/R-03/R-06:** ô tô cắt ngang mép ở `img_015` vẫn có box phần thấy và `truncated=yes`; không gán vật chỉ thấy qua phản chiếu trong gương ở `img_020`. |
| `img_018`, `img_033`, `img_038` | Chân dung trên bảng, ma-nơ-canh, xe mô hình trong tủ | **R-06:** không có box cho các vật giả này; vẫn xét người và xe thật ở các vùng khác của ảnh. |
| `img_019`, `img_027`, `img_028` | Xe diễu hành màu vàng, pickup trên bệ, pickup chở hình ngựa | **R-06/R-07:** thử phân biệt xe thật cải trang với mô hình dựa trên dấu hiệu trong ảnh. Pickup trên bệ ở `img_027` có dấu hiệu vật trưng bày; nếu chưa phân định chắc, ghi ca để reviewer chốt. Với pickup thật ở `img_028`, chỉ bao xe, không bao hình ngựa. |
| `img_021`, `img_030` | Người và xe chồng lấp | **R-05:** gán cá thể đủ rõ ở tiền cảnh; đo trên ảnh gốc và chỉ khoanh ignore ở phần nhỏ hoặc quá dày không thể tách. |
| `img_037` | Pickup có thùng cao gắn sau cabin và xe buýt phía trước | **R-01/R-06:** đề xuất `truck` cho pickup vì cabin và thùng tách nhau; xe buýt ngoài taxonomy. |

Trước khi khóa v1.0, nhóm nên thử mù trên cùng hai vùng: mỗi người tự ghi lớp, số box và thuộc tính, rồi so câu trả lời. Chốt rõ cách xử lý tuk-tuk, xe thật cải trang, vật trưng bày và ranh giới van/truck. Nếu đổi đề xuất, sửa ví dụ và định nghĩa lớp tương ứng trong taxonomy.

## 5. QC và phiên bản

- Giữ nguyên `rules` của taxonomy: IoU ghép cặp từ 0,50; IoU kiểm box là 0,85 với box cao từ 50 px, 0,70 với box thấp hơn. Mốc IAA của lab: IoU trung bình từ 0,85 và cùng lớp từ 95% trên cặp ghép; thiếu lượt gán thì chưa đo. Xem cả box không ghép và ca tham chiếu của coach.
- Khoá v1.0 trước khi gán độc lập; sau IAA, ghi từng ca calibration cùng ID ảnh, mã quy tắc, người chốt, phạm vi sửa và kết quả kiểm lại. Nếu thay đổi cách gán trong v1.1 thì rà lại cả lô theo hướng dẫn của repo.
- Duyệt đủ 40 ảnh, ghi số lỗi kèm mẫu số theo ảnh và theo đối tượng. Thiếu bằng chứng thì Hold; trên 2% ảnh lỗi hoặc lỗi lặp thì Rework; Release khi đủ bằng chứng đạt và reviewer xác nhận. Một ảnh lỗi trên 40 ảnh là 2,5%.

| Phiên bản | Nội dung |
|---|---|
| Bản nháp | Đề xuất cách phân lớp, vẽ box, gán thuộc tính và xử lý ca biên để nhóm thử. |
| v1.0 | Điền sau khi nhóm thống nhất và đối chiếu taxonomy. |
| v1.1 | Điền sau calibration với các ID ca và phạm vi thay đổi thật. |
