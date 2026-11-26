# Báo cáo Day 13 — Trần Ngọc Quân, 2A202603250

## 1. Kết luận trước
- **Mã số học viên (MSSV):** 2A202603250
- **Họ và tên:** Trần Ngọc Quân
- **Task CVAT:** `D13-2A202603250`
- **10 frame của tôi:** `s01f28`, `s01f46`, `s02f08`, `s02f52`, `s03f03`, `s03f19`, `s04f25`, `s04f41`, `s05f02`, `s05f33`
- **Hệ điều hành:** Windows 10
- **Trình duyệt:** cococ

Sau khi hoàn thành 10 frame được giao, tôi đã gán nhãn tổng cộng 86 cuboid (trung bình 8.6 cuboid/frame, vượt xa yêu cầu tối thiểu 50 cuboid của bài lab). Frame có số cuboid ít nhất là `s05f33` với 4 cuboid vì đây là đoạn cuối sequence khi xe di chuyển trên đoạn đường vắng, các xe khác ở ngoài tầm 50 m hoặc bị che khuất hoàn toàn. Lệnh `selfcheck` ghi nhận 12/86 cuboid có cờ cảnh báo (chiếm 14%, nằm trong mức dung sai thông thường của các class lớn như Bus hoặc class nhỏ như Pedestrian/Cyclist). Tôi tự tin nhất ở việc xác định hướng di chuyển yaw (đã căn chỉnh mũi tên đỏ +X dọc thân xe và đối chiếu đồng thời với ảnh camera cam01/cam07) và kiểm soát cao độ đáy cuboid chạm mặt đường z ≈ −1.70 m; phần còn thử thách nhất là việc ước lượng kích thước đối tượng ở xa (30–45 m) do mật độ điểm LiDAR bị suy giảm mạnh.

**Cách dùng CVAT (3 dòng):**
- **Đổi góc nhìn:** Kéo chuột trái để xoay (xoay nhìn thẳng xuống để xem toàn cảnh BEV), kéo chuột phải để dời, lăn chuột để phóng to/thu nhỏ. Nháy đúp vào khoảng trống để về góc nhìn mặc định.
- **Vẽ và chỉnh cuboid:** Chọn nhãn ở thanh công cụ trái → bấm Shape → nháy đúp vào cụm điểm để đặt hộp. Ở khung Top chỉnh vị trí tâm, dài, rộng và xoay yaw; ở khung Side và Front chỉ di chuyển lên/xuống và kéo cạnh để đáy chạm đường (không xoay ở Side/Front).
- **Mũi tên hướng & kiểm tra:** Bật *Cuboid orientation* trong Appearance để hiện vạch đỏ ngắn $+X$, xoay cho vạch đỏ chỉ về đầu xe; chạy `python scripts/lab.py preview` để chiếu cuboid lên 7 ảnh camera kiểm tra tính khớp.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ: `s01f28`, `s01f46`. Ảnh: `t1-s01f28-3d.png`, `t1-s01f28-top.png`, `t1-s01f46-3d.png`, `t1-s01f46-top.png` (ảnh nhìn từ trên xuống, có đánh dấu phía trước, +x, +y).

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f28 | Phía trước ở cạnh dưới (hướng −y); +x chỉ sang trái, +y chỉ ra phía sau | Mặt đường phẳng, cao độ đồng đều, thấp hơn LiDAR khoảng 1.70 m | ~21.1 m (Car #9 ở x=11.40 m, y=17.79 m) | Car, Truck, Pedestrian | Phía sau có Car #4 (y=−12.08 m), Car #7 (y=−8.63 m); bên trái có 2 Pedestrian (#2, #5); bên phải có Truck #3 | Cụm điểm thưa ở xa >35 m bên phải do bị cây cối và biển báo che khuất |
| s01f46 | Phía trước ở cạnh dưới (hướng −y); +x chỉ sang trái, +y chỉ ra phía sau | Mặt đường phẳng, thấp hơn LiDAR khoảng 1.70 m | ~16.2 m (Car #7 ở y=−15.94 m) | Truck, Car, Pedestrian, Cyclist | Phía trước-trái có Truck #1 rất lớn (dist 3.22 m); phía sau có cụm Car #3, #4, #5, #7; bên trái có Pedestrian #6, #9 và Cyclist #8, #10 | Vùng bóng che hoàn toàn phía sau thân xe tải Truck #1 ở sườn trái |

Ba điều **chỉ camera** cho biết:
1. Nhận diện màu sắc, biển số và hoa văn bề mặt đối tượng (ví dụ nhận biết xe màu trắng/đen, đọc chữ trên biển báo giao thông).
2. Phân biệt chính xác phân lớp chi tiết mà LiDAR khó phân giải khi ít điểm: phân biệt xe bán tải chở hàng (Truck) với xe SUV chở người (Car) qua thùng hàng hở phía sau; phân biệt người dắt xe đạp (Pedestrian) với người đang đạp xe (Cyclist).
3. Đọc được trạng thái tín hiệu đèn giao thông, đèn phanh đuôi xe và hướng ánh mắt/khuôn mặt của người đi bộ.

Ba điều **chỉ LiDAR** cho biết:
1. Khoảng cách vật lý 3D tuyệt đối tính bằng mét (x, y, z) từ cảm biến tới từng điểm bề mặt mà không phụ thuộc vào biến dạng quang học.
2. Đo đạc chính xác kích thước hình học 3D (dài, rộng, cao theo mét) không bị ảnh hưởng bởi hiệu ứng thị sai hay phối cảnh xa gần của ống kính 2D.
3. Vùng quét toàn cảnh 360° liên tục quanh xe mà không có góc mù quang học giữa các camera và hoạt động ổn định bất kể điều kiện ngược sáng hay chói lóa.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| s02f08 | Cạnh dưới (hướng −y) | Xe buýt lớn Bus #2 và #3 kích thước rộng 3.3 m chắn tầm nhìn sườn trái |
| s02f52 | Cạnh dưới (hướng −y) | Mật độ xe thưa dần, có xe Car #6 ở rất gần phía trước (dist 5.79 m) |
| s03f03 | Cạnh dưới (hướng −y) | Nhiều ô tô chạy nối đuôi nhau ở cự ly gần (Car #7 cách 3.89 m) |
| s03f19 | Cạnh dưới (hướng −y) | Mật độ đông nhất (16 cuboid), nhiều xe hai bánh di chuyển hỗn hợp sát mép đường |
| s04f25 | Cạnh dưới (hướng −y) | Cụm người đi bộ trên vỉa hè bên trái bị che khuất một phần bởi cột đèn |
| s04f41 | Cạnh dưới (hướng −y) | Có xe Car #6 ở xa nhất (dist 41.25 m) chỉ còn hơn 100 điểm LiDAR |
| s05f02 | Cạnh dưới (hướng −y) | Bus #2 kích thước lớn xuất hiện ở làn đối diện (dist 13.93 m) |
| s05f33 | Cạnh dưới (hướng −y) | Đoạn đường thông thoáng, chỉ có 4 cuboid trong phạm vi 50 m |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | s01f28, #1 | Car | Đúng, vạch đỏ chỉ đầu xe (yaw = 270.9°, đi về −y) | Đúng, z = −0.94 m, h = 1.56 m → z đáy = −1.72 m (chạm đường −1.70 m) | Đã kiểm đủ 3 view và ảnh cam01 | Dài 4.46 m (chuẩn 3.8–5.0 m), Rộng 2.08 m (chuẩn 1.70–2.10 m), Cao 1.56 m (chuẩn 1.40–1.60 m) |
| (b) xe buýt hoặc xe tải | s01f28, #3 | Truck | Đúng, vạch đỏ chỉ đầu xe tải (yaw = 90.0°, đi về +y) | Đúng, z = −0.54 m, h = 2.13 m → z đáy = −1.61 m | Đã kiểm đủ 3 view và ảnh cam07/cam08 | Dài 6.20 m (chuẩn 5.5–9.0 m), Rộng 2.15 m (chuẩn 2.00–2.55 m), Cao 2.13 m (chuẩn) |
| (c) người đi bộ hoặc xe hai bánh | s01f28, #2 | Pedestrian | Đúng, vạch đỏ chỉ theo hướng bước đi (yaw = 90.0°) | Đúng, z = −0.37 m, h = 1.70 m → z đáy = −1.22 m (chân chạm vỉa hè cao hơn đường ~0.48 m) | Đã kiểm đủ 3 view và ảnh cam06 | Dài 0.65 m (chuẩn 0.4–0.9 m), Rộng 0.65 m (chuẩn 0.5–0.9 m), Cao 1.70 m (chuẩn 1.50–1.95 m) |

Vòng hiệu chuẩn: Ban đầu các bạn thường kéo hộp ôm cả vệt phản xạ chạm đất hoặc kéo hộp quá sát mép khiến các điểm gương xe bị lọt ra ngoài; một số bạn nhầm lẫn giữa Truck (bán tải) và Car (SUV). Cả lớp đã thống nhất cách hiểu: xe bán tải có thùng chở hàng tách biệt bắt buộc gán nhãn Truck; mũi tên yaw vạch đỏ +X luôn chỉ về phía trước đầu xe; đáy hộp phải khớp mặt tiếp đất của bánh xe trên view Side/Front.
Các vật bỏ qua vì không đủ cơ sở: Trong frame s05f33, có một số cụm điểm ở cự ly >48 m chỉ có 3–4 điểm tán xạ rời rạc không đủ cấu trúc 15–20% thể tích và hoàn toàn không nhìn thấy được trên camera, được ghi nhận loại UNCERTAIN_SCOPE / LOW_VISIBILITY và bỏ qua theo đúng guideline.

## 4. Bốn hiện tượng chất lượng (Bước 4)
Nguyên nhân ghi là **giả thuyết**. Frame mượn thì ghi rõ nguồn: task chung, hoặc MSSV của bạn cho mượn.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | s01f46 / t4-1-bong-che.png | Tia laser LiDAR truyền thẳng bị thân xe tải Truck #1 ở sát sườn trái che chắn hoàn toàn đường truyền tia | Khoảng trống hình nan quạt không có điểm phản xạ nào ngay phía sau thân xe tải, trong khi mặt đường hai bên vẫn dày điểm | Không được coi vùng trống là đường vắng; với vật bị che một phần thì bật thuộc tính occluded và ước lượng bao trọn thể tích suy luận được |
| 2. Thưa ở xa | s04f41 (#6) / t4-2-thua-o-xa.png | Chùm tia phát laser có độ phân kỳ góc cố định nên mật độ tia giảm theo tỷ lệ nghịch với bình phương khoảng cách | Xe ở cự ly 41.25 m chỉ còn 104 điểm LiDAR, các đường quét tạo thành các vệt ngang rời rạc trên bề mặt đón tia | Chỉ gán nhãn khi thấy tối thiểu 15–20% thể tích và suy luận được từ camera; cố định kích thước theo Bảng 4.3; không đủ cơ sở thì ghi LOW_VISIBILITY và bỏ qua |
| 3. Phản xạ thấp / ít điểm bất thường | s01f28 (#1) / t4-3-phan-xa-thap.png | Bề mặt xe màu sơn tối hấp thụ mạnh bước sóng laser hồng ngoại, kính chắn gió làm tán xạ hoặc khúc xạ tia thay vì phản xạ ngược lại máy thu | Xe tối màu và các ô kính chắn gió có mật độ điểm thưa hơn rõ rệt so với xe màu sáng ở cùng khoảng cách | Không co nhỏ hộp chỉ ôm cụm điểm thưa mà phải mở rộng cuboid đủ kích thước tiêu chuẩn của xe theo Bảng 4.3 |
| 4. Điểm lạ | s02f35 / t4-4-diem-la.png / task chung D13-HC485087 | Hiện tượng phản xạ đa đường (multipath) khi tia laser đập vào vũng nước/bề mặt bóng trên đường rồi dội vào gầm xe, làm kéo dài thời gian bay (ToF) | Cụm khoảng 46 điểm nằm sâu dưới mặt đường ở độ sâu z ≈ −13.5 m (thấp hơn mặt đường xung quanh khoảng 11 m) | Tuyệt đối không gán nhãn thành vật thể vì đây là nhiễu hình học vật lý của cảm biến |

## 5. Số từ cuboid của tôi (Bước 5)
Lấy từ `submission/D13-2A202603250-bangA.csv`; số điểm lấy từ cột `points_in` của `-tukiem.csv`. Mặt đường khoảng −1,7 m: z đáy lệch hơn 0,3 m là lơ lửng hoặc chìm.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| s01f28, #1, Car | −0.94 m | 1.56 m | −1.72 m | 6.70 m | 6.76 m | 270.9° | Chạm đường chuẩn (lệch 0.02 m so với mặt đường −1.70 m) |
| s01f46, #1, Truck | −0.60 m | 2.02 m | −1.61 m | 3.22 m | 3.27 m | 269.9° | Chạm đường tốt (lệch 0.09 m so với mặt đường −1.70 m) |
| s01f28, #3, Truck | −0.54 m | 2.13 m | −1.61 m | 11.57 m | 11.59 m | 90.0° | Chạm đường tốt sát mép vỉa hè |

(a) Cuboid gần nhất và xa nhất:
- Gần nhất: `s01f46`, #1, Truck: `dist_m` = 3.22 m, r = 3.27 m (ở ngay cạnh trái xe thu dữ liệu tại x = −3.13 m, y = 0.73 m, z = −0.60 m).
- Xa nhất: `s04f41`, #6, Car: `dist_m` = 41.25 m, r = 41.26 m (ở phía sau xe thu dữ liệu tại x = 9.91 m, y = 40.04 m, z = −0.99 m, đảm bảo nằm trong phạm vi quy định ≤ 50 m).

(b) Kích thước l, w, h so với Bảng 4.3:
- Xe con (`s01f28` #1, Car): dài 4.46 m (chuẩn sedan 3.8–5.0 m), rộng 2.08 m (chuẩn SUV/sedan 1.70–2.10 m), cao 1.56 m (chuẩn 1.40–1.60 m). Tất cả các chiều đều nằm trong khoảng quy chuẩn.
- Người đi bộ (`s01f46` #6, Pedestrian): dài 0.52 m (chuẩn 0.4–0.9 m), rộng 0.51 m (chuẩn 0.5–0.9 m), cao 1.66 m (chuẩn 1.50–1.95 m). Đều nằm đúng trong khoảng tham chiếu.
- Người đi bộ (`s01f46` #9, Pedestrian): dài 0.21 m hơi nhỏ hơn cận dưới 0.3 m do người đi bộ bị che khuất một phần thân bởi xe máy bên cạnh nên các điểm phản xạ bị khuyết.

(c) Góc `yaw_deg` của xe cùng chiều và xe ngược chiều:
- Xe cùng chiều phía trước: `s01f28` #1 có `yaw_deg` = 270.9°, `s01f46` #3 có `yaw_deg` = 269.1°, `s03f03` #1 có `yaw_deg` = 270.0° → Rất sát giá trị lý thuyết 270° (hướng về phía trước −y).
- Xe ngược chiều: `s01f28` #6 có `yaw_deg` = 90.0°, `s01f28` #7 có `yaw_deg` = 90.0°, `s04f41` #1 có `yaw_deg` = 90.0° → Đúng tuyệt đối giá trị lý thuyết 90° (hướng về phía sau +y).

Mật độ theo khoảng cách (4 cuboid ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| s03f03, #7, Car | 3.89 m | 2172 |
| s04f25, #1, Car | 10.77 m | 2037 |
| s03f03, #8, Car | 16.88 m | 754 |
| s04f41, #6, Car | 41.25 m | 104 |

Quy luật mật độ điểm: Khi khoảng cách tăng gấp khoảng 4 lần (từ 10.77 m lên 41.25 m), số điểm LiDAR nằm trong cuboid giảm khoảng 20 lần (từ 2037 điểm xuống 104 điểm). Kết quả này hoàn toàn tương thích với đường cong suy giảm mật độ ở slide bài giảng (xe con: ~1968 điểm ở 0–10 m, 718 ở 10–20 m, 214 ở 20–30 m, 72 ở 30–40 m).

Tự kiểm tra và chỉnh sửa:
- Hai cuboid đã sửa sau `preview` / `selfcheck`:
  1. `s02f08` #6 (Cyclist): Ban đầu vạch mũi tên yaw bị xoay ngang dẫn đến cờ cảnh báo `length < width`. Tôi đã mở lại CVAT trên khung Top, xoay vạch đỏ chỉ dọc theo hướng xe di chuyển và kéo cạnh dọc mũi tên làm cạnh dài.
  2. `s03f03` #9 (Car): Ban đầu hộp bị nới hơi rộng sang lề đường khiến chiều rộng $w = 2.41\text{ m}$ vượt ngưỡng. Tôi đã chỉnh lại trên Top/Side để ôm sát các điểm thân xe, giảm bớt điểm thừa ngoài lề.
- Những cảnh báo giữ nguyên kèm lý do:
  - Cảnh báo chiều rộng $w = 3.30\text{ m}$ của Bus tại `s02f08` (#2, #3) và `s05f02` (#2): Giữ nguyên vì đây là xe khách/xe buýt cỡ lớn thực tế có gương chiếu hậu và bậc lên xuống nhô rộng hơn xe con thông thường.
  - Cảnh báo "sát ngay ngoài mép" ở `s03f19` #4 và `s04f25` #3: Giữ nguyên vì người đi bộ và xe đạp đang di chuyển sát vỉa hè và lan can bảo vệ, các điểm sát mép thuộc về vật thể môi trường.
  - Cảnh báo "đáy thấp hơn mặt đường 0.99 m" ở `s05f02` #5: Giữ nguyên vì xe nằm ở đoạn đường dốc thoải dần xuống so với vị trí đứng của xe thu dữ liệu.


## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
(a) Ý nghĩa từ góc nhìn mô hình học:
- Nhãn 3D phải đo bằng mét chứ không phải pixel vì hệ thống tự hành điều khiển xe trong không gian vật lý thực 3 chiều; mét là đơn vị khoảng cách thực giúp bộ điều khiển tính toán chính xác cự ly an toàn, thời gian va chạm (Time-to-Collision - TTC) và lập quỹ đạo di chuyển tránh va chạm độc lập với thông số tiêu cự camera.
- Vật thưa điểm cần có quy tắc kích thước riêng (Bảng 4.3) vì khi vật ở xa hoặc bị che khuất chỉ còn vài điểm rời rạc, nếu không có kích thước chuẩn tiên nghiệm, mô hình học sâu sẽ học nhầm kích thước bị co rúm theo cụm điểm nhìn thấy, gây sai lệch lớn về thể tích và dẫn tới hiện tượng dự đoán hộp bị giật cục (flickering) giữa các frame liên tiếp.

(b) Cách dùng AI (3 dòng):
- Công cụ: Antigravity Assistant để hoàn thành báo cáo.

(c) Điều còn chưa chắc:
- Một số đối tượng ở ranh giới xa 40–50 m trong frame `s04f41` và `s05f33` có mật độ điểm rất thưa, dù đã đối chiếu ảnh camera nhưng hình ảnh bị mờ ở góc xa nên việc phân định chi tiết giữa sedan và SUV cỡ nhỏ mang tính suy luận tương đối.
