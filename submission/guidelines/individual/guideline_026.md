# Guideline gán nhãn — Bản nháp r1 (chưa khóa v1.0) — Day 07

- Phiên bản: Bản nháp r1 (rà soát lần 1), chưa phải v1.0
- Ngày: 16/11/2026
- Người viết / duyệt: Hoàng Mạnh Hùng 
- Dữ liệu: 40 ảnh trong `data/images/`. Công cụ: CVAT, Rectangle, xuất COCO 1.0.
- Cách dùng: bản này dùng để thử quy tắc trên ảnh. Mỗi thành viên tự xem ảnh và viết bản cá nhân; nhóm chỉ khóa v1.0 sau khi đối chiếu các bản cá nhân và tự quyết định các ca còn tranh luận.
- Lưu ý: các tham chiếu `img_xxx` giữ nguyên từ bản nháp gốc, **chưa được đối chiếu lại bằng mắt**. Phải xem ảnh trước khi khóa v1.0.

## 1. Mục tiêu

Gán box cho người và phương tiện thật có thể nhận diện trong ảnh đường phố, phục vụ mô hình phát hiện vật thể 2D.

Sai ở đâu thì nguy hiểm: bỏ sót vật thật, gán nhầm vật giả (tượng, hình in, phản chiếu…), và kéo box sang người, xe hoặc hàng hóa bên cạnh.

Nguyên tắc chung: nhất quán quan trọng hơn hoàn hảo. Cùng một tình huống phải cho cùng một nhãn dù ai gán. Gặp ca chưa có quy tắc, chọn phương án khả dĩ nhất, tick `can_review` và ghi vào ghi chú cấp ảnh (mẫu ở Mục 5).

## 2. Lớp và thuộc tính

### Lớp

**R-01 — Mỗi đối tượng thật đủ điều kiện nhận một lớp.** Nhận diện theo hình dáng nhìn thấy; không đoán theo biển số, hãng, người lái hay mục đích chuyến đi.

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `person` | Người thật, kể cả người ngồi hoặc đứng trên xe mở (xe máy, xe đạp, thùng xe tải), nếu phần thấy được nhận ra được là người nhờ đầu hoặc thân và đạt ngưỡng R-05. | Người đi bộ ở `img_034`. | Ma-nơ-canh ở `img_033`; một chi hay mảng màu đơn lẻ chưa đủ để nhận ra (xem R-07); người chỉ thấy qua kính xe kín (xem R-08). |
| `car` | Ô tô chở người dạng sedan, hatchback, wagon hoặc SUV, không có thùng hàng tách khỏi cabin và không có dáng thân van; MPV/minivan kiểu ô tô con (đầu xe có capo nhô rõ phía trước kính lái, hàng cửa sổ hông liền mạch) cũng thuộc `car`. | Sedan bên trái ở `img_023`. | Pickup (là `truck`); xe thân hộp đầu phẳng (là `van`). |
| `truck` | Xe tải, đầu kéo, pickup hoặc xe có thùng/sàn chở hàng tách biệt khỏi cabin; nắp thùng pickup không làm đổi lớp; xe tải thùng kín có cabin riêng vẫn là `truck`. | Pickup tiền cảnh ở `img_001`; xe tải nhỏ cabin xanh ở `img_036`. | Xe thân liền khối, khoang sau nối liền cabin (là `van`). |
| `van` | Xe thân liền khối, khoang sau dạng hộp cao nối liền với cabin, đầu xe gần như phẳng, không có capo nhô rõ; có thể có cửa kính chở khách hoặc vách kín chở hàng. | Xe thân hộp bạc ở `img_005`. | Bus/minibus (ngoài taxonomy); xe tải cabin riêng (là `truck`); MPV có capo (là `car`). |
| `motorcycle` | Xe máy hoặc mô tô hai bánh, kể cả scooter; mô tô gắn sidecar vẫn là mô tô. | Scooter ở `img_006`. | Xe đạp, xích lô, tuk-tuk (ngoài taxonomy). |
| `ignore_region` | Rectangle đánh dấu vùng có vật thuộc lớp mục tiêu nhưng quá nhỏ hoặc tụ dày đến mức không thể tách từng vật. | Cụm vật quá nhỏ hoặc đám đông dày (R-05). | Không dùng để đánh dấu mọi vật ngoài taxonomy. |

Ranh giới còn mở, cần thử mù: `van` và minibus (xe khách nhỏ), MPV và `van`. Dấu hiệu gợi ý cho minibus: thân dài hơn rõ rệt so với van thông thường và hàng cửa sổ hông kéo dài nhiều ô. Phân vân thì áp dụng R-07.

**Vật không thuộc lớp nào thì làm gì:**
- Vật ngoài taxonomy (xe đạp, xích lô, tuk-tuk, xe buýt): không gán box xe; người thật đi trên đó vẫn được xét riêng.
- Vật giả (tượng, hình in, đồ chơi, ma-nơ-canh, bóng, phản chiếu): không gán (R-06).
- Vật mờ, không đủ căn cứ phân lớp: không đoán lớp, ghi ghi chú cấp ảnh (R-07).
- Vùng chỉ có vật quá nhỏ hoặc quá dày thuộc lớp mục tiêu: dùng `ignore_region` theo R-05, không dùng cho các trường hợp trên.

### Thuộc tính

**R-02 — Thuộc tính áp dụng cho năm lớp đối tượng; `ignore_region` không có thuộc tính.** Dùng thuộc tính `occluded_level` tự tạo của Project CVAT, không thay bằng cờ Occluded mặc định của CVAT.

| Thuộc tính | Các mức cố định và ngưỡng |
|---|---|
| `occluded_level` | Ước lượng bằng mắt tỷ lệ phần đối tượng **nằm trong khung** bị vật khác che, theo diện tích bóng dáng:<br>`0` = không thấy bị che (bỏ qua chi tiết che dưới ~5%, ví dụ dây điện mảnh, gương chiếu hậu của xe khác);<br>`1` = từ ~5% đến dưới 25%;<br>`2` = từ 25% đến dưới 60%;<br>`3` = từ 60% trở lên nhưng vẫn nhận ra đối tượng.<br>Sát ranh giới giữa hai mức thì chọn mức cao hơn. Chỉ dùng ước lượng này để chọn thuộc tính, không để nới box. |
| `truncated` | `yes` nếu vật kéo dài ra ngoài mép ảnh; `no` nếu toàn bộ vật ở trong khung hoặc chỉ bị vật khác che. Chạm mép nhưng vẫn thấy vật kết thúc trong khung thì `no`. Có thể vừa bị che vừa bị cắt. |
| `can_review` | Tick khi có đủ căn cứ để vẽ box nhưng còn nghi ngờ lớp, ranh box hoặc mức che. Nếu chưa có đủ căn cứ để vẽ box, ghi ID ảnh, vị trí và lý do vào ghi chú cấp ảnh (mẫu ở Mục 5). |

## 3. Quy tắc vẽ box

- **R-03 — Ôm sát phần nhìn thấy.** Vẽ hình chữ nhật nhỏ nhất bao phần nhìn thấy của cùng một đối tượng trong ảnh gốc.
  - Dung sai: mỗi cạnh lệch không quá ~2 px so với biên thật của phần nhìn thấy. Lý do: với box khoảng 50×50 px, lệch 2 px mỗi trục còn IoU ≈ 0,86, lệch 3 px chỉ còn ≈ 0,79, đã dưới mốc kiểm box 0,85 ở Mục 5.
  - **Vật bị che:** nếu các phần nhìn thấy bị ngăn cách bởi vật che, dùng một box bao các phần đó và khoảng giữa chúng, không kéo ra ngoài phần nhìn thấy. Không kéo box theo phần bị che hoặc đã ra khỏi ảnh.
  - **Vật bị cắt ở mép ảnh:** cạnh box trùng mép ảnh, không vượt ra ngoài.
  - **Tính vào box:** bộ phận gắn liền như gương xe, sidecar và các bộ phận gắn cố định khác. Thùng xe tải/pickup là một phần của xe, box lấy đến thành thùng.
  - **Loại khỏi box:** bóng, phản chiếu, vệt nhòe và hàng hóa. Hàng hóa là vật chở không thuộc cấu tạo xe (đồ buộc sau xe máy, hàng chất vượt thành thùng xe tải).
  - **Với người:** tính quần áo, giày và mũ đang đội; tính đồ đeo sát thân (ba lô, túi đeo chéo) nếu nằm trong dáng người. Không tính đồ cầm, xách, ô/dù và vật vác vươn xa. Nếu ô che mất một phần người, ranh box theo phần người thực sự nhìn thấy, tính mức che theo R-02 và tick `can_review`.
- **R-04 — Tách người khỏi xe và hàng khỏi xe (một vật có nhiều phần).** Mỗi người trên xe có một `person` riêng nếu đủ điều kiện; bản thân xe có một box riêng. Các box có thể chồng lên nhau.
  - Xe đang được chở trên xe khác vẫn là đối tượng riêng nếu là xe thật và đủ điều kiện.
  - Box của xe chở gồm cabin, khung và rơ-moóc đang nối, không mở rộng để bao các xe được chở trên sàn. Cạnh trên của box là đỉnh cao nhất của chính xe chở, kể cả khi xe được chở nhô cao hơn.
- **R-05 — Vật quá nhỏ và đám đông. Đo trên ảnh gốc.** Nếu box phần nhìn thấy cao **dưới 20 px**, không gán box đối tượng riêng; khoanh `ignore_region` sát vật hoặc cụm vật quá nhỏ. Từ **20 px trở lên**, gán nếu xác định được lớp và ranh giới.
  - Đám đông: tách những cá thể đủ rõ và chỉ khoanh phần không thể tách; tránh phủ `ignore_region` lên vật đã gán được.
  - Đo theo pixel của ảnh gốc, không theo kích thước hiển thị khi đã zoom.
  - Sát ngưỡng (trong khoảng ±2 px quanh 20 px): tick `can_review`.
- **R-06 — Chỉ gán vật thật thuộc taxonomy (phản chiếu, hình in, tượng, đồ chơi, xe chủ).**
  - Loại tượng, hình in, đồ chơi, ma-nơ-canh, bóng và phản chiếu. Ảnh phản chiếu của xe không tạo box thứ hai.
  - Xe thật nhìn xuyên qua kính thì gán nếu nhận diện được lớp và đạt R-05, tính mức che theo phần nhìn thấy.
  - Xe chủ (ego): không gán phần xe đang chụp từ bên trong (ví dụ cửa hay gương xe của camera).
  - Xe ngoài taxonomy như xe đạp, xích lô, tuk-tuk và bus không có box xe; người thật đi trên đó vẫn được xét riêng.
  - Thật hay giả xét theo bản chất vật thể (xe/người thật còn nguyên chức năng), không theo vị trí đặt (bệ, showroom, xe diễu hành). Xe thật bị trang trí hoặc hóa trang vẫn là xe thật, gán theo dáng xe nền.
  - Dấu hiệu gợi ý một vật là bản sao: kích thước bất thường so với xung quanh, không có bánh hoặc kính thật, chi tiết và chất liệu đồng nhất như tượng, đế gắn liền mặt bệ.
- **R-07 — Vật mờ, khuất nhiều hoặc nghi là xe/người mô hình.** Khi nhận diện được vật và giới hạn phần thấy, gán theo quy tắc thường và bật `can_review` nếu chưa chắc. Nếu chỉ có vệt mờ hoặc không đủ căn cứ phân biệt vật thật với mô hình thu nhỏ hoặc tượng, ghi vùng cần hỏi reviewer và không đoán lớp. Chỉ dùng `ignore_region` theo R-05.
  - Phân vân giữa một lớp trong taxonomy và một loại ngoài taxonomy (ví dụ `van` và minibus): chỉ gán lớp trong taxonomy khi dấu hiệu nghiêng rõ về lớp đó; nếu gần 50/50 thì không gán, ghi vào ghi chú cấp ảnh.
- **R-08 — Người trong cabin kín.** Người chỉ nhìn thấy qua kính của xe kín (ô tô, xe tải, xe buýt) thì **không** gán `person`; người nhìn thấy trực tiếp, không qua kính, trên xe mở thì gán theo R-01. Lý do: kính phản chiếu và làm méo, khiến lớp và ranh box thiếu ổn định giữa các người gán. Đây là đề xuất, cần nhóm chốt (xem câu hỏi mở số 3 ở Mục 5).

## 4. Ca khó

Các ví dụ dưới đây mô tả cách áp dụng quy tắc trên một vùng cụ thể, không phải bộ nhãn cho toàn ảnh. Các ca ghi "chưa chốt" cần đánh dấu `can_review` trên box hoặc ghi vấn đề cấp ảnh, chờ reviewer quyết.

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| `img_006` | Xe máy bên trái có người lái và người ngồi sau | R-04 | Một `motorcycle` và hai `person` nếu cả hai người nhận diện được và đạt ngưỡng; không gộp thành một box. Người đi xe đạp bên phải vẫn có thể là `person`. | Mỗi người và mỗi xe là một đối tượng riêng. |
| `img_002` | Xe tải chở nhiều ô tô | R-04 | Một `truck` cho cụm cabin và rơ-moóc đang nối; xét từng xe được chở riêng, kiểm lại lớp từng xe theo dấu hiệu thực thấy và ghi mức che khi khung sàn che thân xe. | Box xe chở không bao xe được chở; xe được chở vẫn là xe thật. |
| `img_004`, `img_016` | Tuk-tuk | R-06 | Không gán box cho tuk-tuk. Ở `img_016`, scooter xanh và người lái được xét riêng theo R-04. | Tuk-tuk ngoài taxonomy. Chưa chốt: chờ nhóm quyết. |
| `img_015`, `img_020` | Ô tô ở mép ảnh và cảnh trong gương | R-02, R-03, R-06 | Ô tô cắt ngang mép ở `img_015` vẫn có box phần thấy, cạnh box trùng mép ảnh, `truncated=yes`; không gán vật chỉ thấy qua phản chiếu trong gương ở `img_020`. | Box ôm phần thấy; phản chiếu không phải vật thật. |
| `img_018`, `img_033`, `img_038` | Chân dung trên bảng, ma-nơ-canh, xe mô hình trong tủ | R-06 | Không có box cho các vật giả này; vẫn xét người và xe thật ở các vùng khác của ảnh. | Vật giả không thuộc đối tượng cần gán. |
| `img_019`, `img_027`, `img_028` | Xe diễu hành màu vàng, pickup trên bệ, pickup chở hình ngựa | R-06, R-07 | Xét theo bản chất vật thể, không theo vị trí đặt. Pickup trên bệ ở `img_027` có dấu hiệu vật trưng bày; nếu chưa phân định chắc thì `can_review` và ghi ca để reviewer chốt. Với pickup thật ở `img_028`, chỉ bao xe, không bao hình ngựa. | Thật/giả theo bản chất; hình ngựa là hàng/vật in, không thuộc xe. Chưa chốt: `img_027`. |
| `img_021`, `img_030` | Người và xe chồng lấp | R-05 | Gán cá thể đủ rõ ở tiền cảnh; đo trên ảnh gốc và chỉ khoanh ignore ở phần nhỏ hoặc quá dày không thể tách. | Tránh phủ ignore lên vật gán được. |
| `img_037` | Pickup có thùng cao gắn sau cabin và xe buýt phía trước | R-01, R-06 | `truck` cho pickup; xe buýt ngoài taxonomy nên không gán. | Cabin và thùng tách nhau. |
| (tìm ảnh minh họa trong 40 ảnh) | Người ngồi trong ô tô, thấy qua kính | R-08 | Không gán `person`. | Kính phản chiếu và làm méo; chờ nhóm chốt. |
| (tìm ảnh minh họa) | Người cầm ô / xe máy chở hàng cồng kềnh | R-03 | Ô và hàng không tính vào box; ranh box theo phần người/xe thấy được; tick `can_review` nếu ô che người. | Đồ cầm và hàng hóa không thuộc người/xe. |
| (tìm ảnh minh họa) | MPV hoặc minibus | R-01, R-07 | MPV kiểu ô tô con → `car`; minibus ngoài taxonomy; phân vân gần 50/50 thì không gán và ghi chú. | Chưa thử mù ranh giới car/van/minibus. |

Trước khi khóa v1.0, nhóm nên thử mù trên cùng hai vùng: mỗi người tự ghi lớp, số box và thuộc tính, rồi so câu trả lời. Chốt rõ cách xử lý tuk-tuk, xe thật cải trang, vật trưng bày và ranh giới van/truck. Nếu đổi quyết định, sửa ví dụ và định nghĩa lớp tương ứng ở Mục 2.

## 5. QC và phiên bản

### Ngưỡng QC áp dụng

- Giữ nguyên `rules` của taxonomy: IoU ghép cặp từ 0,50; IoU kiểm box là 0,85 với box cao từ 50 px, 0,70 với box thấp hơn.
- Mốc IAA (độ đồng thuận giữa các người gán) của lab: IoU trung bình từ 0,85 và cùng lớp từ 95% trên cặp ghép; thiếu lượt gán thì chưa đo. Xem cả box không ghép và ca tham chiếu của coach.
- Duyệt đủ 40 ảnh, ghi số lỗi kèm mẫu số theo ảnh và theo đối tượng. Thiếu bằng chứng thì Hold; trên 2% ảnh lỗi hoặc lỗi lặp thì Rework; Release khi đủ bằng chứng đạt và reviewer xác nhận. Một ảnh lỗi trên 40 ảnh là 2,5%, nên với 40 ảnh thực tế là không chấp nhận ảnh lỗi nào.
- "Ảnh lỗi" = ảnh có ít nhất một trong: sót đối tượng đủ điều kiện, sai lớp, thiếu hoặc sai thuộc tính, box lệch quá dung sai R-03, gán vật giả hoặc vật ngoài taxonomy.

### Ai quyết ca mơ hồ

- Người gán tick `can_review` hoặc ghi ghi chú cấp ảnh; reviewer chốt.
- Ca còn tranh luận do nhóm tự quyết định khi đối chiếu các bản cá nhân, trước khi khóa v1.0.
- Khóa v1.0 trước khi gán độc lập; sau IAA, ghi từng ca calibration cùng ID ảnh, mã quy tắc, người chốt, phạm vi sửa và kết quả kiểm lại. Nếu thay đổi cách gán trong v1.1 thì rà lại cả lô theo hướng dẫn của repo.

### Checklist tự kiểm trước khi nộp mỗi ảnh

- [ ] Mọi đối tượng thật, đủ điều kiện, đều có box; không đối tượng nào có hai box.
- [ ] Box ôm sát, lệch không quá ~2 px mỗi cạnh; vật cắt ở mép có cạnh trùng mép ảnh.
- [ ] Đủ `occluded_level` và `truncated` cho mọi box (trừ `ignore_region`).
- [ ] Không có box cho tượng, hình in, ma-nơ-canh, phản chiếu, người trong cabin kín, xe ngoài taxonomy.
- [ ] Mỗi `ignore_region` có lý do theo R-05; không phủ lên vật đã gán.
- [ ] Ca nghi ngờ đã tick `can_review` hoặc đã ghi ghi chú cấp ảnh.

### Mẫu ghi chú cấp ảnh

| Ảnh | Vị trí (mô tả hoặc toạ độ) | Lý do không gán / nghi ngờ | Người ghi | Kết quả reviewer |
|---|---|---|---|---|
| | | | | |

### Câu hỏi mở cần nhóm chốt trước v1.0

1. **Xe ngoài taxonomy (tuk-tuk, xe buýt, xe đạp):** không gán box nghĩa là mô hình sẽ coi chúng là nền. Chấp nhận, hay đề xuất thêm lớp ở bản sau?
2. **Vật mờ không đủ căn cứ (R-07):** hiện chỉ ghi chú, không box, không `ignore_region`, nên vùng đó vẫn bị coi là nền. Có nên mở thêm cho `ignore_region` dùng với vật "chắc là thật nhưng không phân lớp được"?
3. **Người trong cabin kín (R-08):** bỏ qua để giảm bất đồng, hay gán khi thấy rõ?
4. **Ngưỡng 20 px theo chiều cao (R-05):** ô tô nhìn ngang rộng nhưng thấp có thể dưới 20 px cao mà vẫn rõ. Giữ nguyên, hay dùng cạnh dài hơn cho các lớp xe?
5. **Ranh giới `car`/`van`/minibus** (MPV, xe khách nhỏ): các dấu hiệu gợi ý ở R-01 có đủ để hai người gán giống nhau không? Cần thử mù.
6. **Xe thật trưng bày** (ví dụ `img_027`): gán hay không?
7. **`ignore_region` khi xuất COCO:** cần xác nhận cách xuất (danh mục riêng hay cờ `iscrowd`) để người dùng dữ liệu không nhầm.

### Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| Bản nháp | Đề xuất cách phân lớp, vẽ box, gán thuộc tính và xử lý ca biên để nhóm thử. | Khởi tạo. |
| Bản nháp r1 | Rà soát lần 1: thêm nguyên tắc nhất quán (Mục 1); làm rõ định nghĩa `person`, `car` (MPV), `truck`, `van`; sửa ngưỡng `occluded_level` (mức 0 chấp nhận che dưới ~5%, sát ranh giới chọn mức cao hơn); thêm dung sai box ~2 px/cạnh (R-03); làm rõ box vật bị che, bị cắt, hàng hóa, thùng xe; thêm quy tắc đồ đeo/ô cho người (R-03); làm rõ xe chở và xe được chở (R-04); đo theo pixel ảnh gốc và tick `can_review` khi sát ngưỡng (R-05); xe thật nhìn qua kính thì gán nếu nhận diện được, thật/giả theo bản chất vật thể (R-06); đổi "mô hình" thành "xe/người mô hình" và thêm quy tắc phân vân với lớp ngoài taxonomy (R-07); thêm R-08 người trong cabin kín; thêm ca chung chưa gắn ảnh (Mục 4); định nghĩa "ảnh lỗi", checklist và mẫu ghi chú cấp ảnh (Mục 5); thêm danh sách câu hỏi mở. | Các ca ở Mục 4 và câu hỏi mở ở Mục 5. Chưa phải v1.0. |
| v1.0 | Điền sau khi nhóm thống nhất và đối chiếu taxonomy. | |
| v1.1 | Điền sau calibration với các ID ca và phạm vi thay đổi thật. | |
