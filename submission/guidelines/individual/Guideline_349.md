# Guideline gán nhãn — V1 — Cá nhân

> Guideline cá nhân được xây dựng theo cấu trúc 5 phần của `Guideline_349.md`, tham chiếu `guideline_v0_starter.md` và tập dữ liệu `images(2).zip`.
> Loại nhãn: bounding box trên ảnh đường phố. Công cụ: CVAT. Định dạng xuất: COCO 1.0.

- Phiên bản: V1
- Ngày: 17/11/2026
- Người viết / duyệt: Nguyễn Tuấn Minh

## 1. Mục tiêu

Guideline này quy định cách gán nhãn bounding box cho các đối tượng là người và phương tiện xuất hiện trong ảnh đường phố. Nhãn được sử dụng để huấn luyện và đánh giá mô hình phát hiện đối tượng (*object detection*), phục vụ các bài toán phân tích giao thông, giám sát đường phố và hỗ trợ hệ thống giao thông thông minh.

Phạm vi dữ liệu gồm 40 ảnh trong tập `images(2).zip`. Ảnh có nhiều bối cảnh như đường phố, bãi đỗ xe, khu chợ, khu vực đông người, ảnh ban đêm, ảnh đen trắng, phương tiện chở hàng, xe ba bánh/tuk-tuk, phương tiện đồ chơi hoặc trưng bày, tượng và ma-nơ-canh.

Rủi ro khi gán sai:
- Nhầm người thật với tượng, ma-nơ-canh hoặc hình in sẽ tạo *false positive*.
- Nhầm tuk-tuk với xe máy hoặc nhầm xe van với xe tải làm sai phân loại phương tiện.
- Bỏ sót người lái và phương tiện bị che khuất làm giảm khả năng phát hiện trong tình huống giao thông đông đúc.

Đơn vị gán nhãn là từng đối tượng riêng biệt. Một người và phương tiện mà người đó điều khiển là hai *instance* độc lập, không gộp thành một box.

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `person` | Một người thật, còn sống, xuất hiện trong cảnh; gồm người đi bộ, người lái hoặc ngồi trên phương tiện. | Người đi bộ, người đội mũ bảo hiểm lái xe máy, người ngồi sau xe. | Tượng, ma-nơ-canh, hình người trên poster/ảnh quảng cáo, hình nộm. |
| `car` | Ô tô chở người thông thường, không thuộc nhóm xe tải, van hoặc bus. | Sedan, SUV, hatchback, station wagon, ô tô con. | Xe đồ chơi/mô hình; ô tô chỉ xuất hiện trên ảnh in hoặc billboard. |
| `truck` | Phương tiện cơ giới có kết cấu/chức năng chính để chở hàng; gồm xe tải và pickup thật. | Xe tải thùng, xe tải chở ô tô, xe bán tải thật. | Xe tải đồ chơi; tượng/điêu khắc mô phỏng xe tải. |
| `van` | Xe chở người/hàng cỡ nhỏ dạng thân kín, khoang thân liền hoặc thùng kín nhỏ; không phải bus hoặc truck rõ ràng. | Van giao hàng, xe van thùng kín cỡ nhỏ. | Xe tải lớn; xe khách/bus có khoang hành khách chuyên dụng. |
| `bus` | Phương tiện chở nhiều hành khách, thân xe lớn và có khoang hành khách đặc trưng. | Xe buýt đô thị, xe khách. | Van chở ít người; xe tải. |
| `motorcycle` | Phương tiện hai bánh có động cơ, gồm xe máy và mô tô. | Xe máy số, xe tay ga, mô tô. | Xe đạp; tuk-tuk/xe ba bánh có động cơ. |
| `bicycle` | Phương tiện hai bánh vận hành bằng sức người, có bàn đạp. | Xe đạp thường, xe đạp có người đạp. | Xe máy; xe ba bánh có động cơ. |
| `three_wheeler` | Phương tiện giao thông có động cơ với cấu hình ba bánh, thường dùng chở người/hàng; gồm tuk-tuk, auto-rickshaw, xe lam. | Tuk-tuk chở khách, xe ba bánh chở hàng. | Xe máy hai bánh; xe đạp ba bánh không động cơ nếu không thuộc mục tiêu nhận diện. |
| `ignore_region` | Vùng hình ảnh không thể gán riêng từng instance một cách đáng tin cậy, chỉ dùng khi cần loại vùng đó khỏi đánh giá/huấn luyện theo cấu hình dự án. | Đám đông dày đặc không thể tách từng người; vùng nhòe không thể nhận diện đối tượng. | Không dùng thay cho việc gán các đối tượng nhìn thấy rõ; không khoanh toàn ảnh chỉ vì ảnh khó. |

**Lưu ý về thiết kế lớp:** `guideline_v0_starter` chỉ có `car`, `truck`, `van`, `motorcycle`, `person` và `ignore_region`. Bộ nhãn V1 bổ sung `bus`, `bicycle` và `three_wheeler` để phân biệt các loại phương tiện này thay vì ép vào lớp cũ. Khi cấu hình CVAT, cần tạo đúng tên lớp như bảng.

Vật **không thuộc lớp nào** thì không tự tạo lớp mới và không ép vào lớp gần giống. Không gán đối tượng không thuộc ontology; đánh dấu `can_review` và ghi vấn đề cấp ảnh để reviewer thống nhất cách xử lý. Ví dụ: xe đạp điện không thể phân biệt rõ, xe chuyên dụng lạ, xe đồ chơi, xe mô hình hoặc phương tiện giả tưởng.

Quy tắc chọn lớp khi không khớp hoàn toàn:
- Ưu tiên chức năng và cấu tạo thực tế của phương tiện, không dựa riêng vào màu sắc, kích thước trong ảnh hoặc hàng hóa đang chở.
- Pickup thật được xếp `truck`; pickup dạng tượng/điêu khắc hoặc mô hình trưng bày không gán.
- Tuk-tuk/xe lam/auto-rickshaw có động cơ và ba bánh được xếp `three_wheeler`, không xếp `motorcycle`.
- Nếu không thể phân biệt chắc chắn `van` với `truck` do góc chụp/che khuất, chọn lớp có bằng chứng hình dạng rõ nhất; tick `can_review` nếu vẫn còn nghi vấn.

### Thuộc tính

| Thuộc tính | Giá trị cố định | Ý nghĩa / ngưỡng |
|---|---|---|
| `occluded_level` | `0` | Không bị che hoặc phần bị che ước tính dưới 10%; hình dáng chính gần như đầy đủ. |
| `occluded_level` | `1` | Bị che từ 10% đến dưới 40%; vẫn nhìn thấy phần lớn đối tượng và nhận diện dễ. |
| `occluded_level` | `2` | Bị che từ 40% đến dưới 80%; chỉ nhìn thấy một phần đáng kể nhưng vẫn xác định được lớp. |
| `occluded_level` | `3` | Bị che từ 80% trở lên nhưng vẫn còn dấu hiệu đủ để xác định chắc chắn lớp. Nếu không thể xác định lớp thì không tạo box, ghi vấn đề cấp ảnh/`can_review`. |
| `truncated` | `no` | Box không bị giới hạn bởi mép ảnh; đối tượng nằm trọn trong khung ảnh, dù có thể bị vật khác che. |
| `truncated` | `yes` | Một phần đối tượng bị cắt bởi cạnh trên/dưới/trái/phải của ảnh. Chỉ tính cắt bởi biên ảnh, không tính che khuất bởi vật khác. |
| `can_review` | `tick` | Có nghi vấn về lớp, ranh giới box, đối tượng thật/giả hoặc mức che khuất; cần reviewer xác nhận. |
| `can_review` | `unticked` | Đã có thể xác định lớp và box theo quy tắc, không còn nghi vấn đáng kể. |

Tỷ lệ che khuất được ước lượng trên diện tích hình chiếu toàn bộ đối tượng nếu có thể suy ra hợp lý; không cần đo pixel chính xác. Không suy diễn phần bị che nếu không có căn cứ. Với đối tượng chỉ còn một mảnh nhỏ không đủ nhận diện, không gán box chỉ để đặt `occluded_level=3`.

## 3. Quy tắc vẽ box

| Rule ID | Quy tắc | Cách thực hiện |
|---|---|---|
| R01 | Box ôm sát | Vẽ hình chữ nhật nhỏ nhất bao trọn phần nhìn thấy của instance. Không chừa khoảng trống lớn, không ôm cả bóng đổ hoặc nền đường. |
| R02 | Phần bị che | Chỉ vẽ box quanh phần nhìn thấy; không kéo box xuyên qua vật che để ước lượng hình dạng khuất. Gán `occluded_level` theo mức che. |
| R03 | Cắt mép ảnh | Nếu đối tượng đi ra ngoài biên ảnh, vẽ box đến đúng mép ảnh, đặt `truncated=yes`. Không tự mở rộng box ra ngoài khung. |
| R04 | Người và xe | Gán `person` và phương tiện thành hai box độc lập. Người lái xe máy không phải một lớp `rider`; vẫn là `person` + `motorcycle`. Người ngồi trên xe vẫn gán `person` nếu nhìn thấy đủ để nhận diện. |
| R05 | Nhiều phương tiện chồng lấp | Mỗi phương tiện nhìn thấy và nhận diện được là một instance riêng. Không gộp nhiều xe thành một box. Nếu chỉ thấy một phần xe nhưng nhận diện được, vẫn gán box phần nhìn thấy. |
| R06 | Xe chở xe | Gán `truck` cho xe chuyên chở. Gán thêm `car` cho từng ô tô được chở nếu từng xe là phương tiện thật, nhìn thấy đủ để nhận diện. Không gán phần khung/giá chở hàng như một xe riêng. |
| R07 | Đối tượng giả/phi thực | Không gán `person` cho tượng, ma-nơ-canh, hình nộm, ảnh in, poster hoặc hình phản chiếu. Không gán phương tiện đồ chơi, mô hình thu nhỏ, xe giả tưởng/đồ trang trí. |
| R08 | Phương tiện trưng bày | Phương tiện thật, kích thước thật, có cấu tạo phương tiện giao thông thì vẫn gán lớp tương ứng dù đang đỗ hoặc trưng bày. Nếu là tượng/điêu khắc/mô hình thì không gán. |
| R09 | Phản chiếu | Không gán đối tượng chỉ xuất hiện trong gương chiếu hậu, kính hoặc bề mặt phản chiếu để tránh tạo bản sao của cảnh. Chỉ gán đối tượng trực tiếp trong cảnh. |
| R10 | Billboard/hình ảnh trên màn hình | Không gán người/xe nằm trong ảnh quảng cáo, biển quảng cáo, tranh, màn hình hoặc ảnh in. |
| R11 | Vật quá nhỏ/nhòe | Nếu không đủ chi tiết để xác định chắc chắn lớp hoặc ranh giới đối tượng, không đoán. Bỏ qua instance và ghi `can_review`/vấn đề cấp ảnh nếu ảnh có nhiều trường hợp. |
| R12 | Đám đông | Tách từng `person` khi có thể xác định từng cá thể. Chỉ dùng `ignore_region` cho cụm quá dày/nhòe không thể tách đáng tin cậy; không dùng `ignore_region` thay cho các box cá thể nhìn rõ. |
| R13 | Ảnh tối/đen trắng | Vẫn gán nhãn nếu đối tượng nhận diện được từ hình dáng và ngữ cảnh trực quan. Không loại ảnh chỉ vì thiếu màu hoặc chụp ban đêm. |
| R14 | Không thuộc ontology | Không ép đối tượng vào lớp gần giống. Ghi lại ảnh, vị trí và mô tả; tick `can_review` để thống nhất cách xử lý. |

## 4. Ca khó

Bảng dưới đây ghi lại các trường hợp tiêu biểu được lập cho tập dữ liệu. Quyết định áp dụng theo ontology V1. Nếu chưa chốt được trường hợp cụ thể, đánh dấu `can_review` trên box hoặc ghi vấn đề cấp ảnh; dùng mã quy tắc trong nhật ký QC.

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| `img_001` | Xe tải nhỏ chở hàng; người đứng cạnh và người trên xe | R04, R01 | Gán `truck` cho xe; gán `person` riêng cho từng người nhìn thấy đủ rõ. | Người và xe là các instance độc lập. |
| `img_002` | Xe tải chuyên chở nhiều ô tô | R06 | Gán `truck` cho xe chở; gán `car` riêng cho từng ô tô thật nhìn thấy đủ để nhận diện. | Không gộp xe chở và hàng hóa thành một box. |
| `img_003` | Người gánh hàng, xe đạp/xe máy trong cảnh chợ | R04, R05 | Gán `person` và phương tiện riêng nếu nhận diện được; không gán thúng hàng thành phương tiện. | Hàng hóa không phải lớp mục tiêu. |
| `img_004` | Tuk-tuk/xe ba bánh có động cơ | R14 | Gán `three_wheeler`; không gán `motorcycle`. | Khác cấu hình hai bánh của `motorcycle`. |
| `img_005` | Nhiều người ngồi trên xe máy, che khuất lẫn nhau | R02, R04 | Mỗi người nhìn thấy rõ gán `person`; mỗi xe máy nhận diện được gán `motorcycle`; đặt `occluded_level` phù hợp. | Không tạo lớp `rider`. |
| `img_006` | Dòng xe máy và người tham gia giao thông | R05, R12 | Tách từng `motorcycle` và `person` có thể nhận diện; vùng quá dày không tách được mới dùng `ignore_region`. | Không gộp cả dòng xe thành một box. |
| `img_007` | Xe con và xe van trong khu vực gara | R01, R14 | Gán `car` cho ô tô con; `van` cho xe thân kín cỡ nhỏ nếu phân biệt được. | Không phân loại theo màu hoặc vị trí đỗ. |
| `img_008` | Người mặc áo phản quang cạnh nhiều xe máy | R04, R05 | Gán `person` và `motorcycle` độc lập. | Trang phục không làm thay đổi lớp `person`. |
| `img_009` | Phương tiện cỡ lớn/xe buýt hoặc xe khách trong phố | R14 | Gán `bus` nếu thân xe và khoang chở khách cho thấy rõ xe buýt/xe khách; nếu chỉ là van nhỏ thì gán `van`. | Nếu ảnh không đủ bằng chứng phân biệt, tick `can_review`. |
| `img_010` | Người và xe máy trong khu chợ, nhiều vật che | R02, R05 | Gán từng `person`/`motorcycle` nhìn thấy; box chỉ ôm phần nhìn thấy. | Không vẽ box xuyên qua quầy/hàng hóa. |
| `img_011` | Đối tượng người/xe bị che hoặc góc chụp khó | R02, R11 | Chỉ gán đối tượng nhận diện được; đặt `occluded_level`; trường hợp không rõ lớp thì bỏ qua và review. | Không suy đoán phần khuất. |
| `img_012` | Xe chở hàng với hàng hóa che một phần phương tiện | R02, R06 | Gán phương tiện nếu nhận diện được; hàng hóa không gán lớp mục tiêu. | Box theo phần xe nhìn thấy. |
| `img_013` | Cảnh đường phố ban đêm | R13 | Gán người/phương tiện nhận diện được dù ảnh tối; không gán đèn, biển hiệu hoặc bóng phản chiếu. | Điều kiện ánh sáng không phải lý do tự động bỏ nhãn. |
| `img_014` | Người đi bộ bị nhòe chuyển động | R11, R13 | Gán `person` nếu hình dáng người vẫn nhận diện được; box ôm phần nhìn thấy và tick `can_review` nếu ranh giới không chắc. | Không nhầm vệt nhòe với một người khác. |
| `img_015` | Ô tô bị cắt ở cạnh ảnh | R03 | Gán `car`, box kết thúc tại biên ảnh, `truncated=yes`. | Không loại xe chỉ vì không thấy toàn bộ. |
| `img_016` | Tuk-tuk/xe ba bánh cùng xe máy và người | R04, R14 | Tuk-tuk gán `three_wheeler`; xe máy hai bánh gán `motorcycle`; người gán `person`. | Phân loại từng phương tiện theo cấu tạo. |
| `img_017` | Tượng người/nhân vật trang trí ngoài đường | R07 | Không gán `person` cho tượng. | Đối tượng không phải người thật. |
| `img_018` | Hình phương tiện trên billboard/quảng cáo | R10 | Không gán `car` cho hình xe trên biển quảng cáo; chỉ gán phương tiện thật dưới mặt đường nếu có. | Hình in không phải đối tượng vật lý trong cảnh. |
| `img_019` | Xe ô tô đồ chơi/xe giả tưởng màu vàng | R07 | Không gán `car`. | Đồ chơi/xe mô hình không phải phương tiện giao thông thật. |
| `img_020` | Xe hoặc người xuất hiện trong gương chiếu hậu | R09 | Không gán đối tượng chỉ xuất hiện trong phần phản chiếu của gương. | Tránh nhân đôi đối tượng. |
| `img_021` | Đám đông người và nhiều xe máy | R05, R12 | Tách từng `person`/`motorcycle` nhận diện được; chỉ khoanh `ignore_region` cho vùng không thể tách cá thể. | Không thay toàn bộ cảnh đông đúc bằng một `ignore_region`. |
| `img_022` | Ảnh đen trắng, đối tượng nhỏ/khó nhìn | R11, R13 | Gán đối tượng nếu vẫn nhận diện chắc chắn; nếu không, bỏ qua và ghi review. | Không suy đoán từ ngữ cảnh đơn thuần. |
| `img_023` | Người đi xe đạp trên đường cùng ô tô | R04, R05 | Gán `person`, `bicycle` và `car` thành các box riêng nếu nhìn thấy. | Không gộp người với xe đạp. |
| `img_024` | Xe thật và mô hình/xe trưng bày trên nóc | R07, R08 | Gán xe kích thước thật; không gán xe mô hình/đồ chơi đặt trên xe. | Phân biệt theo kích thước, chất liệu và ngữ cảnh. |
| `img_025` | Cảnh phố ven đường, phương tiện ở xa | R11, R13 | Gán các phương tiện/người đủ rõ; bỏ qua điểm quá nhỏ không xác định được lớp. | Không tạo box tùy đoán. |
| `img_026` | Người đi xe máy trong cảnh đêm | R04, R13 | Gán `person` và `motorcycle` riêng; dùng `can_review` nếu bị tối/che đáng kể. | Người lái không phải một phần của box xe. |
| `img_027` | Tượng/điêu khắc hình xe bán tải | R07 | Không gán `truck` dù hình dạng giống pickup. | Đây là vật trưng bày dạng tượng, không phải xe thật. |
| `img_028` | Xe pickup cổ kích thước thật, có vật trang trí trên thùng | R08 | Gán `truck` cho pickup thật; không gán tượng/đồ trang trí trên thùng. | Xe thật vẫn là phương tiện dù đang trưng bày. |
| `img_029` | Người/xe đạp/xe máy trong ảnh đen trắng | R04, R13 | Gán từng đối tượng nhìn thấy; phân biệt `bicycle` và `motorcycle` theo cấu tạo. | Không để ảnh đơn sắc làm thay đổi ontology. |
| `img_030` | Nhiều xe máy đỗ sát nhau | R05 | Mỗi xe máy phân biệt được là một `motorcycle` riêng; không gộp cả bãi xe. | Box ôm sát từng xe. |
| `img_031` | Đường phố ban đêm, mặt đường ướt và phản chiếu | R09, R13 | Gán phương tiện/người thật; bỏ qua hình phản chiếu trên mặt đường. | Không tạo box cho ảnh phản chiếu. |
| `img_032` | Người đứng thành hàng, xe ở mép ảnh | R03, R12 | Gán từng `person`; phương tiện bị cắt mép gán `truncated=yes`. | Chỉ dùng `ignore_region` nếu cụm người không thể tách. |
| `img_033` | Ma-nơ-canh trưng bày trong cửa hàng | R07 | Không gán `person`. | Ma-nơ-canh không phải người thật. |
| `img_034` | Người đi bộ qua đường và xe cộ | R04, R05 | Gán `person` và từng phương tiện nhìn thấy được. | Tách người khỏi phương tiện. |
| `img_035` | Xe ba bánh/tuk-tuk trên đường | R14 | Gán `three_wheeler`; người điều khiển gán `person` nếu nhìn thấy. | Không gán `motorcycle` cho toàn bộ xe ba bánh. |
| `img_036` | Xe van màu xanh | R01, R14 | Gán `van` nếu đúng kiểu thân van cỡ nhỏ; nếu cấu tạo là xe tải chở hàng thì gán `truck`. | Dựa vào kiểu thân xe và công năng. |
| `img_037` | Xe van và xe buýt/xe lớn trên đường | R14 | Gán `van` và `bus` theo từng phương tiện; nếu xe lớn là xe tải thì dùng `truck`. | Không gom mọi xe lớn vào `truck`. |
| `img_038` | Các xe ô tô mô hình trong cửa hàng | R07 | Không gán `car` cho các xe mô hình trên kệ. | Đồ chơi không thuộc phạm vi. |
| `img_039` | Cảnh phố đông, người và xe ở xa | R05, R11, R12 | Gán từng instance rõ; bỏ qua đối tượng quá nhỏ/không đủ căn cứ; review vùng đông không tách được. | Không gán hàng loạt box thiếu chắc chắn. |
| `img_040` | Người lái xe máy cạnh ô tô | R04, R05 | Gán `person`, `motorcycle` và `car` độc lập. | Không dùng nhãn `rider` hoặc gộp người với xe. |

## 5. QC và phiên bản

### Ngưỡng QC đề xuất

| Hạng mục QC | Ngưỡng chấp nhận đề xuất | Cách kiểm tra |
|---|---|---|
| Sai lớp (*classification error*) | ≤ 2% số box được kiểm tra | Đối chiếu định nghĩa lớp; ưu tiên kiểm tra `three_wheeler`, `van`/`truck`, `bus`. |
| Box không ôm sát / sai ranh giới | ≤ 5% số box được kiểm tra | Kiểm tra box quá rộng, thiếu phần nhìn thấy hoặc gộp nhiều instance. |
| Bỏ sót đối tượng rõ ràng | ≤ 3% đối tượng mục tiêu trong ảnh mẫu | Đặc biệt người lái xe, xe bị che một phần và xe ở rìa ảnh. |
| Thuộc tính sai | ≤ 5% số box có thuộc tính | Kiểm tra `occluded_level`, `truncated` và `can_review`. |
| Nhầm vật thật với tượng/đồ chơi/hình in | 0 lỗi nghiêm trọng trong mẫu QC | Bắt buộc kiểm tra nhóm ảnh ca khó. |

Các ngưỡng trên là ngưỡng đề xuất cho guideline cá nhân, không phải ngưỡng được quy định trong hai file mẫu. Nếu giảng viên/nhóm có ngưỡng QC chính thức, ưu tiên áp dụng ngưỡng chính thức.

### Quy trình xử lý ca mơ hồ

1. Annotator đánh dấu `can_review` trên box có nghi vấn; nếu vấn đề ảnh hưởng toàn ảnh hoặc không có box phù hợp, ghi chú image-level.
2. Ghi rõ tên ảnh, vị trí tương đối, lớp đang phân vân, hai phương án và Rule ID liên quan.
3. Reviewer đối chiếu định nghĩa lớp và ca tương tự trong mục 4; không tự thêm lớp nếu chưa thống nhất.
4. Nếu quyết định làm thay đổi ontology hoặc quy tắc gán nhãn, cập nhật guideline và tăng phiên bản.

### Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| V0 | Bản khởi đầu gồm `car`, `truck`, `van`, `motorcycle`, `person`, `ignore_region`; thuộc tính `occluded_level`, `truncated`, `can_review`. | Theo `guideline_v0_starter.md`. |
| V1 | Bổ sung `bus`, `bicycle`, `three_wheeler`; định nghĩa lại lớp, ngưỡng thuộc tính, quy tắc box và 40 ca khó; bổ sung QC. | Phù hợp các trường hợp trong `images(2).zip`: tuk-tuk, xe đạp, bus/van, xe chở xe, tượng, ma-nơ-canh, xe đồ chơi, billboard, phản chiếu và ảnh bị cắt. |

**Ghi chú triển khai CVAT:** Tạo các label đúng chính tả như mục 2; cấu hình các attribute cho từng shape theo khả năng hỗ trợ của CVAT. Nếu CVAT không cho đặt `ignore_region` như một vùng loại trừ trong pipeline COCO, cần thống nhất với reviewer cách export/loại vùng này trước khi gán hàng loạt.
