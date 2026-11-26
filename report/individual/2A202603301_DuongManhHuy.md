# Báo cáo Day 13 — Dương Mạnh Huy, 2A202603301

## 1. Kết luận trước

- Mã số: 2A202603301 · task `D13-2A202603301`
- 10 frame: s01f00, s01f34, s02f05, s02f32, s03f32, s03f54, s04f30, s04f45, s05f08, s05f25
- Hệ điều hành: Windows 11 Home · trình duyệt: Chrome · CVAT: 2.81.0 (cvat-sdk 2.83.0)
- Ngồi cạnh: 2A202603307

Bảng A có **85 cuboid** (s01f00: 20; s01f34, s03f32: 5; các frame còn lại 6–10). Frame ít nhất là s01f34 và s03f32 (5 hộp): ở hai frame này nhiều cụm điểm không khớp vật nào trên ảnh camera (bụi cây, tường, cột) hoặc là vật xa không chắc hướng, nên tôi không gán; s03f32 ban đầu chỉ có 4 hộp nên tôi vẽ thêm 1. `selfcheck` cờ **7/85** cuboid. Tin nhất ở **đáy**: 68/76 hộp ước lượng được mặt đường có `ground_gap_m` trong ±0,1 m. Yếu nhất ở **hướng** (đã đảo 9 hộp sau khi đối chiếu camera, nhưng chưa đối chiếu hết các xe nằm ngang hoặc ở làn bên trái) và **kích thước** (nhiều hộp dùng kích thước điển hình 4,40 × 1,80 × 1,50 m thay vì đo sát theo điểm), cùng các vật xa 30–50 m chỉ còn vài chục điểm.

Cách dùng CVAT: (1) đổi góc nhìn bằng kéo chuột trái (xoay), chuột phải (dời), con lăn (phóng to); xoay khung 3D tới khi nhìn thẳng xuống để xem toàn cảnh; (2) vẽ bằng nút cuboid → label → **Shape**, nháy đúp để đặt, chỉnh tâm/dài/rộng/yaw ở **Top**, đáy và chiều cao ở **Side/Front**; (3) bật **Appearance → Cuboid orientation** để thấy vạch đỏ `+X` chỉ đầu xe, và xem cuboid trên 7 ảnh camera bằng `python scripts/lab.py preview`.

## 2. Đọc frame (Bước 2)

Hai frame đọc kỹ. Ảnh: `t1-s01f00-3d`, `t1-s01f00-top`, `t1-s01f34-3d`, `t1-s01f34-top`.

| Frame  | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu                                                                                             | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m)                                                                                                                             | Vật xa nhất đọc được (~ m)         | Loại vật thấy                               | Vật phía sau / hai bên                                                             | Cụm chưa đọc được |
| ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------- | ------------------------------------------- | ---------------------------------------------------------------------------------- | ----------------- |
| s01f00 | Phía trước là −y, ngược vạch xanh lá dài, nằm ở **cạnh trái** của ảnh; +y (vạch xanh lá) chỉ sang phải (ra sau xe); +x (vạch đỏ) chỉ xuống dưới (bên trái xe)      | Thấp hơn LiDAR ~1,8 m (trung vị z đáy hộp −1,85 m); **dốc xuống về phía trước**: các xe ~40 m phía trước có z đáy −2,1 đến −2,3 m, trong khi xe gần có z đáy ≈ −1,8 m | ~45 m (xe con phía sau)            | Car, xe máy có người (Cyclist), người đi bộ | Xe con phía sau ~12,7 m (az ≈ 178°); xe máy phía sau bên phải ~10,9 m (az ≈ −113°) | Không có          |
| s01f34 | Phía trước là −y, ngược vạch xanh lá dài, nằm ở **cạnh phải** của ảnh (góc xoay khung 3D khác s01f00); +y chỉ sang trái (ra sau xe), +x chỉ lên trên (bên trái xe) | Thấp hơn LiDAR ~1,8 m (trung vị z đáy −1,78 m); **khá phẳng**: các xe từ 9 m đến 48 m, phía trước lẫn phía sau, đều có z đáy −1,68 đến −1,80 m                        | ~48 m (xe tải phía trước bên trái) | Car, Truck                                  | Xe con phía sau ~9,9 m (az ≈ 176°)                                                 | Không có          |

Chỉ **LiDAR** cho biết: (1) khoảng cách theo mét, ví dụ xe con gần nhất ở s01f00 cách 3,5 m; (2) kích thước thật, ví dụ người đi bộ ở s01f00 cao ~1,8 m; (3) độ cao mặt đường ~−1,8 m và vật ở phía sau, hai bên trong vùng 360° mà camera không phủ kín. Chỉ **camera** cho biết: (1) **màu xe**; (2) **loại xe** (xe con, bán tải hay xe buýt; LiDAR chỉ cho một khối điểm cùng kích thước); (3) **hướng xe**, tức đầu hay đuôi: trên point cloud thân xe là một hộp gần đối xứng, phải nhìn ảnh camera mới biết đầu xe ở phía nào.

Tám frame lướt:

| Frame  | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có)                                                                            |
| ------ | ------------------------------------------------ | ------------------------------------------------------------------------------------------- |
| s02f05 | −y (ngược vạch xanh lá dài)                      | Một hộp có z đáy ≈ +0,7 m, cao hơn LiDAR: nghi xe trên cầu vượt / đường cao hơn (chưa chắc) |
| s02f32 | −y (ngược vạch xanh lá dài)                      | Toàn xe con; 5/7 hộp ở phía sau                                                             |
| s03f32 | −y (ngược vạch xanh lá dài)                      | Một hộp có z đáy ≈ +0,2 m (nghi đường cao hơn); hộp Bus vẽ thêm bị lệch 90° (xem §6c)       |
| s03f54 | −y (ngược vạch xanh lá dài)                      | Xe xa nhất (~49,6 m) chỉ 17 điểm, z đáy −0,35 m                                             |
| s04f30 | −y (ngược vạch xanh lá dài)                      | Xe buýt bên trái-phía sau ~16 m, ~3 900 điểm                                                |
| s04f45 | −y (ngược vạch xanh lá dài)                      | Mặt đường thấp hơn các frame khác (trung vị z đáy −1,92 m)                                  |
| s05f08 | −y (ngược vạch xanh lá dài)                      | Nhiều xe con nhất (10), không có người                                                      |
| s05f25 | −y (ngược vạch xanh lá dài)                      | Xe buýt và xe tải lớn ở xa bên phải-phía trước, 40–48 m                                     |

## 3. Ba cuboid có hướng dẫn (Bước 3)

Ảnh: `t2-a-car`, `t2-b-bus-truck`, `t2-c-ped-cyclist`.

| Vật                              | Frame, idx                           | Class   | Mũi tên yaw đúng hướng?                                 | Đáy chạm đường?                           | Đã kiểm Top/Side/Front + ảnh?                     | Kích thước trong bảng 4.3?                                                           |
| -------------------------------- | ------------------------------------ | ------- | ------------------------------------------------------- | ----------------------------------------- | ------------------------------------------------- | ------------------------------------------------------------------------------------ |
| (a) xe con gần                   | s01f00, 1 (3,5 m)                    | Car     | Đúng: yaw 268,1°, vạch đỏ chỉ về đầu xe trên ảnh camera | Có: z đáy −1,80 m, cách mặt đường +0,04 m | Có: Top/Side/Front và ảnh camera trong `t2-a-car` | Dài 4,42 m trong khoảng; **rộng 2,23 m vượt** 1,70–1,90 (sedan) và 2,10 (SUV)        |
| (b) xe buýt hoặc xe tải          | s01f34, 5 (48,5 m; frame 1 không có) | Truck   | Đúng: yaw 265,1°, cùng chiều xe thu dữ liệu             | Có: z đáy −1,77 m, +0,04 m                | Có: `t2-b-bus-truck`                              | 8,78 × 2,00 × 2,79 m: trong khoảng xe tải hộp (5,5–9,0 × 2,00–2,55 × 2,40–3,50)      |
| (c) người đi bộ hoặc xe hai bánh | s01f00, 3 (10,9 m)                   | Cyclist | Đúng: yaw 285,3°, vạch đỏ chỉ về đầu xe máy             | Có: z đáy −1,70 m, ~0,00 m                | Có: `t2-c-ped-cyclist`                            | 1,80 × 0,74 × 1,56 m: trong khoảng xe máy có người (1,8–2,3 × 0,70–0,95 × 1,40–1,80) |

Vòng hiệu chuẩn: tôi **không tham gia**, nên không có số liệu so với lớp. Vật **bỏ qua vì không đủ cơ sở**: các cụm điểm trong 30 m mà camera nhìn thấy nhưng không khớp vật nào (nghi bụi cây, tường, cột) không gán; 28 xe ở 30–50 m có hướng không chắc (nằm ngang hoặc ở làn bên trái) đã bỏ thay vì đoán hướng (UNCERTAIN_SCOPE / LOW_VISIBILITY); các vật xa chỉ còn vài điểm không vẽ.

## 4. Bốn hiện tượng chất lượng (Bước 4)

Nguyên nhân ghi là **giả thuyết**. Frame mượn thì ghi rõ nguồn: task chung, hoặc MSSV của bạn cho mượn.

| Hiện tượng                           | Frame / tên ảnh / nguồn                                           | Nguyên nhân (giả thuyết)                                                                                                            | Thấy gì trên point cloud                                                                                                                                         | Hệ quả khi gán (theo guideline)                                                                                                     |
| ------------------------------------ | ----------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| 1. Bóng che                          | s02f00 / `t4-1-bong-che` / task chung `D13-HC485087`              | Giả thuyết: xe buýt sát bên trái (~4 m) chắn tia laser nên phía sau nó không có điểm trả về                                         | Ngay sau xe buýt là một **vùng tối hình quạt** gần như không có điểm, toả ra xa theo hướng tia LiDAR; hai bên vùng đó điểm vẫn dày                               | Vùng trống là "chưa quan sát được", không phải đường trống; vật bị che một phần thì bật **occluded**, cuboid bao phần suy luận được |
| 2. Thưa ở xa                         | s03f54 / `t4-2-thua-o-xa` / frame của tôi                         | Giả thuyết: 40 tia LiDAR toả ra nên mật độ điểm giảm nhanh theo khoảng cách                                                         | Xe con ~49,6 m chỉ còn 17 điểm trong hộp (so với 3 102 điểm của xe ở 3,5 m); chỉ còn thấy **mặt sau xe** (phía quay về LiDAR), không thấy hai bên thân và nóc xe | Thấy ~15–20% thể tích thì vẽ; không thì bỏ qua và ghi **LOW_VISIBILITY**                                                            |
| 3. Phản xạ thấp / ít điểm bất thường | s05f08, hộp 8 (Car, 29,2 m) / `t4-3-phan-xa-thap` / frame của tôi | Giả thuyết: thân xe sơn tối màu hoặc nhiều kính nên phản xạ tia laser yếu, hoặc bị vật khác che một phần (chưa phân biệt chắc được) | Hộp chỉ có **30 điểm**, trong khi xe con cùng tầm (25–33 m) trong bài có trung vị **178 điểm**, tức còn khoảng 1/6                                               | Ghi giả thuyết, **không khẳng định** nguyên nhân; vẫn gán nếu đủ cơ sở                                                              |
| 4. Điểm lạ                           | s02f35 / `t4-4-diem-la` / task chung `D13-HC485087`               | Giả thuyết: phản xạ nhiều lần (multipath) hoặc nhiễu cảm biến                                                                       | Cụm khoảng 46 điểm phía sau bên trái, cách ~29 m, ở z ≈ −13,5 m, tức **thấp hơn mặt đường ~11 m**; chỉ thấy rõ khi xoay nghiêng khung 3D                         | **Không gán** thành vật                                                                                                             |

## 5. Số từ cuboid của tôi (Bước 5)

Lấy từ `submission/D13-2A202603301-bangA.csv`; số điểm lấy từ cột `points_in` của `-tukiem.csv`. Mặt đường khoảng −1,7 m: z đáy lệch hơn 0,3 m là lơ lửng hoặc chìm.

| Cuboid (frame, idx, class) | z        | h       | z đáy = z − h/2 | dist_m  | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường)                                 |
| -------------------------- | -------- | ------- | --------------- | ------- | --------------- | ------- | ---------------------------------------------------------------------- |
| s01f00, 1, Car             | −1,041 m | 1,516 m | −1,799 m        | 3,54 m  | 3,69 m          | 268,1°  | Chạm đường (cách mặt đường cục bộ +0,04 m)                             |
| s02f05, 4, Bus             | 0,000 m  | 3,171 m | −1,585 m        | 21,00 m | 21,00 m         | 266,4°  | Chạm đường (+0,09 m); z đáy cao hơn −1,7 m vì mặt đường chỗ đó cao hơn |
| s01f00, 10, Car            | −1,814 m | 1,483 m | −2,556 m        | 40,12 m | 40,17 m         | 89,8°   | **Chìm** ~0,29 m so với mặt đường cục bộ (cờ selfcheck)                |

(a) Cuboid **gần nhất**: s01f00:1 Car, `dist_m` = 3,54 m, r = 3,69 m. Cuboid **xa nhất**: s03f54:6 Car, `dist_m` = 49,64 m, r = 49,64 m (z đáy −0,35 m, chỉ 17 điểm: nghi lơ lửng hoặc đường dốc lên, xem §6c).
(b) Người đi bộ s01f00:17: 0,79 × 0,76 × 1,78 m, cả ba chiều trong khoảng người lớn (0,4–0,9 × 0,5–0,9 × 1,50–1,95 m). Xe con s01f00:1: 4,42 × 2,23 × 1,52 m; dài và cao trong khoảng sedan, **rộng 2,23 m vượt** khoảng (1,70–1,90 m, SUV tới 2,10 m). Giả thuyết: hộp bao cả gương chiếu hậu hoặc điểm của vật sát bên; nên thu hẹp lại theo Top.
(c) Xe cùng chiều phía trước s01f34:1 (phía trước bên trái, 8,7 m): `yaw_deg` = 270,2°, gần 270° như mong đợi. Xe ngược chiều s01f00:19 (phía trước bên trái, 25,8 m, hộp tôi tự vẽ): `yaw_deg` = 89,9°, gần 90° như mong đợi.

Mật độ theo khoảng cách (4 cuboid ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m  | Số điểm (`points_in`) |
| -------------------------- | ------- | --------------------- |
| s01f00, 1, Car             | 3,54 m  | 3 102                 |
| s05f08, 4, Car             | 14,75 m | 1 007                 |
| s05f08, 7, Car             | 23,62 m | 403                   |
| s04f30, 9, Car             | 39,58 m | 92                    |

Quy luật: xa gấp khoảng 1,6–2 lần thì số điểm còn khoảng 1/3–1/4; từ 3,5 m ra 40 m (gấp ~11 lần) số điểm giảm ~34 lần, nhanh hơn 1/d. So với đường cong ở slide (xe con, trung vị: 718 điểm ở 10–20 m, 214 ở 20–30 m, 72 ở 30–40 m), số của tôi cùng xu hướng nhưng cao hơn một chút (1 007; 403; 92), có thể vì một số hộp của tôi bao rộng hơn xe thật nên đếm thêm điểm sát mép.

Tự kiểm, hai cuboid **đã sửa** sau khi xem `preview` / `selfcheck`:

- s01f00:17: ban đầu gán Cyclist 1,80 × 0,70 × 1,65 m; sửa thành **Pedestrian** 0,79 × 0,76 × 1,78 m, xoay yaw 335° → 63° (s01f00:18 sửa tương tự).
- s05f25:4 Car (17,1 m): **đảo hướng** 263° → 88° (xe ngược chiều), nới 4,91 × 2,10 → 4,98 × 2,14 m. Tổng cộng tôi đảo hướng 9 hộp và bật occluded 3 hộp.

Các cảnh báo **giữ nguyên** (7/85) và lý do:

- s01f00:10 Car (chìm 0,29 m): khi đảo hướng tôi đã hạ hộp 0,26 m; chưa nâng lại (xem §6c).
- s01f00:18 Pedestrian (21 điểm sát mép): người đứng sát người khác (s01f00:17 cách ~0,8 m), điểm ngoài mép là của người bên cạnh.
- s02f05:6 Cyclist (2,59 × 1,21 × 1,60 m, 27,7 m): ngoài khoảng xe máy nhưng nằm trong khoảng **xe ba bánh có người** của bảng 4.3 (2,5–3,5 × 1,00–1,50 × 1,50–2,00 m); giả thuyết là xe ba bánh.
- s02f32:4 Car (rộng 2,32 m): chỉ vượt ngưỡng công cụ 0,02 m; giả thuyết: xe cỡ lớn (SUV/van) hoặc hộp bao cả gương.
- s04f45:4 Truck (cao 1,70 m): sát ngưỡng 1,7 m của công cụ; giả thuyết: bán tải thấp hoặc hộp chưa bao hết thùng xe (cách 34,6 m, 117 điểm).
- s01f00:20 Car và s03f32:5 Bus: **hộp lỗi chưa sửa** (kích thước sai, length < width), ghi ở §6c.

Đổi kiểm với 2A202603307: bạn ấy chọn 2 con số trong mục 5 và đối chiếu với bảng A mở trên màn hình máy tôi; cả hai **khớp**, không phải sửa. Tôi làm ngược lại với bài của bạn ấy; chỉ xem trên màn hình, không gửi file.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

(a) Model 3D dự đoán vị trí, kích thước và hướng của vật theo **mét** trong không gian quanh xe, nên nhãn phải đo bằng mét: cùng một chiếc xe, kích thước theo pixel đổi theo khoảng cách và camera, còn theo mét thì không đổi, và xe tự lái cần khoảng cách thật để phanh, tránh. Vật thưa điểm cần quy tắc riêng vì nếu nhãn "đoán" hộp cho cụm vài điểm, model sẽ học bịa ra vật từ nhiễu; còn bỏ qua có quy tắc (ngưỡng 15–20% thể tích, LOW_VISIBILITY) giúp nhãn nhất quán và model học đúng giới hạn của cảm biến.

(b) Dùng AI: có, được coach cho phép. Công cụ, câu hỏi chính và cách tôi dùng kết quả được trình bày trong **báo cáo dùng AI riêng** đi kèm bài nộp.

(c) Điều còn chưa chắc:

- **Hướng**: tôi đã đảo 9 hộp sau khi đối chiếu camera, nhưng chưa đối chiếu hết các xe nằm ngang hoặc ở làn bên trái, nên có thể còn hộp ngược đầu.
- Nhiều hộp dùng kích thước điển hình 4,40 × 1,80 × 1,50 m chứ không đo sát theo điểm.
- s01f00:10 bị hạ 0,26 m khi sửa hướng; `selfcheck` báo chìm 0,29 m, nên nâng đáy lại.
- s01f00:20 (dài 1,36 m, lơ lửng 1,38 m) và s03f32:5 (Bus 3,12 × 8,39 m, lệch 90°) là hộp lỗi chưa sửa.
- Một số hộp có z đáy cao hơn LiDAR (s02f05, s03f32) hoặc lệch mặt đường ở ~40–50 m: chưa rõ là cầu vượt, đường dốc hay hộp lơ lửng.
