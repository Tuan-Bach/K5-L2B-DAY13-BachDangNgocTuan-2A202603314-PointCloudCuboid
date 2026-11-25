# Báo cáo Day 13 — Bạch Đặng Ngọc Tuấn, 2A202603314

## 1. Kết luận trước
- **Mã số học viên:** `2A202603314` · **Task CVAT:** `D13-2A202603314` · **10 frame được giao:** `s01f03`, `s01f52`, `s02f00`, `s02f33`, `s03f12`, `s03f42`, `s04f28`, `s04f58`, `s05f17`, `s05f45`.
- **Môi trường:** Windows 11 · Trình duyệt Edge / Chrome · **CVAT v2.81.0** (chạy cục bộ qua Docker) · **MSSV bạn ngồi cạnh (đổi kiểm):** `2A202603300`.
- **Tổng quan kết quả:** Tổng cộng đã vẽ **79 cuboid** trên 10 frame (70 `Car`, 6 `Truck`, 2 `Bus`, 1 `Cyclist`). Các frame có ít cuboid nhất là `s02f00`, `s02f33` và `s03f42` (mỗi frame 5 cuboid) do mật độ phương tiện thực tế trong bán kính 30–50 m ở các đoạn đường này ít hơn so với các frame đông như `s04f28` (14 cuboid). Phiếu tự kiểm `selfcheck` ghi nhận **0 / 79 cuboid** có cảnh báo. Phần tin cậy nhất là hướng mũi tên `yaw_deg`, kích thước hộp và độ cao đáy `z_bottom` chạm mặt đường; phần còn hạn chế nhất là một số vật ở rìa xa > 35 m bị thưa điểm.

**Cách dùng CVAT 3D:**
1. **Đổi góc nhìn:** Kéo chuột trái để xoay khung 3D (xoay nhìn thẳng từ trên xuống để xem toàn cảnh), kéo chuột phải để dời khung, lăn con lăn để phóng to/thu nhỏ, nháy đúp chỗ trống để về góc nhìn mặc định.
2. **Vẽ và chỉnh cuboid:** Chọn label và bấm **Shape** (không dùng Track), nháy đúp vào cụm điểm để đặt hộp, sau đó tinh chỉnh tâm/kích thước/góc xoay ở khung **Top** và chỉnh độ cao đáy chạm mặt đường ở **Side/Front**.
3. **Bật hướng và xem trên ảnh:** Bật **Cuboid orientation** trong mục **Appearance** (vạch đỏ ngắn `+X` chỉ về đầu xe) và chạy `python scripts/lab.py preview` để chiếu các cuboid lên 7 ảnh camera.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ: `s01f03` và `s01f52`. Ảnh: `t1-s01f03-3d.png`, `t1-s01f03-top.png`, `t1-s01f52-3d.png`, `t1-s01f52-top.png`.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f03` | Phía trước (`-y`) ở cạnh dưới/hướng xe tiến trong ảnh top-down; `+x` (đỏ) sang trái xe, `+y` (xanh lá) ra sau xe | Mặt đường tương đối phẳng; thấp hơn LiDAR khoảng `1.69 m` (`z ≈ -1.69 m`) | ~`37.35 m` (xe `Car` #11) | `Car`, `Truck` | Có nhiều xe con ở làn bên trái (`x > 0`, `12–20 m`) và phía sau (`y > 0`, `6–15 m`) | Các cụm điểm thưa ở rìa > 50 m không rõ hình dạng xe |
| `s01f52` | Phía trước (`-y`) ngược hướng trục `+y`; `+x` (đỏ) sang trái xe, `+y` (xanh lá) ra sau xe | Mặt đường phẳng; thấp hơn LiDAR khoảng `1.75 m` (`z ≈ -1.75 m`) | ~`21.82 m` (xe `Car` #7) | `Car` | Có xe chạy phía sau (`y ≈ +6.49 m` và `+21.61 m`) và bên trái (`x ≈ +5.86 m` đến `+8.90 m`) | Các cụm điểm rải rác dưới mặt đường (`z < -2.3 m`) và rìa cây/công trình bên lề |

**3 điều chỉ camera cho biết và 3 điều chỉ LiDAR cho biết (từ `s01f03` và `s01f52`):**
- **Chỉ camera cho biết:** (1) Màu sắc thân xe và nhận diện phân biệt đầu xe (đèn pha) với đuôi xe (đèn hậu); (2) Phân biệt chính xác loại xe (xe con SUV vs xe tải thùng vs xe buýt) ở khoảng cách xa hoặc khi điểm thưa; (3) Vạch kẻ đường, dải phân cách và biển báo giao thông bằng chữ/màu sắc.
- **Chỉ LiDAR cho biết:** (1) Khoảng cách hình học chính xác theo mét (`dist_m`, `x`, `y`, `z`) quanh 360° kể cả vùng mù giữa các camera; (2) Kích thước 3D thực (`length`, `width`, `height` theo mét) và độ cao đáy so với mặt đường (`z_bottom ≈ -1.7 m`); (3) Các điểm phản xạ bất thường hoặc bóng che hình học phía sau vật cản lớn.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| `s02f00` | Hướng `-y` (ngược trục `+y` xanh lá) | Có xe buýt lớn (`Bus` #1, dài `12.44 m`) ở phía trước bên trái (`dist = 9.25 m`) |
| `s02f33` | Hướng `-y` (ngược trục `+y` xanh lá) | Có các xe đi ngược chiều (`yaw ≈ 90°`) ở làn đối diện |
| `s03f12` | Hướng `-y` (ngược trục `+y` xanh lá) | Xe gần nhất cách chỉ `4.54 m` (`Car` #1) ngay phía sau bên trái |
| `s03f42` | Hướng `-y` (ngược trục `+y` xanh lá) | Mật độ phương tiện thưa, chỉ có 5 xe rõ nét trong phạm vi |
| `s04f28` | Hướng `-y` (ngược trục `+y` xanh lá) | Mật độ phương tiện dày nhất (14 cuboid), nhiều xe xếp nối đuôi và có `Cyclist` (#14) chạy phía sau bên phải |
| `s04f58` | Hướng `-y` (ngược trục `+y` xanh lá) | Có xe ở rất xa (`Car` #6 ở `45.05 m`) chỉ còn 61 điểm |
| `s05f17` | Hướng `-y` (ngược trục `+y` xanh lá) | Có xe tải dài `10.52 m` (`Truck` #1) ở bên trái (`12.70 m`) |
| `s05f45` | Hướng `-y` (ngược trục `+y` xanh lá) | Nhiều xe đi song song và nối đuôi gần nhau ở cự ly `10–30 m` |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f03`, `#1` | `Car` | Đúng (`268.0°`, chỉ về `-y`) | Đúng (`z_bottom = -1.726 m`) | Đã kiểm Top/Side/Front + `preview` | Đúng (`4.41 × 1.86 × 1.45 m`, nằm trong chuẩn Sedan) |
| (b) xe buýt hoặc xe tải | `s01f03`, `#9` | `Truck` | Đúng (`88.2°`, ngược chiều) | Đúng (`z_bottom = -1.996 m`) | Đã kiểm Top/Side/Front + `preview` | Đúng (`6.26 × 2.18 × 3.12 m`, nằm trong chuẩn xe tải nhẹ) |
| (c) người đi bộ hoặc xe hai bánh | `s04f28`, `#14` | `Cyclist` | Đúng (`264.2°`, chỉ về hướng di chuyển `-y`) | Đúng (`z_bottom = -1.649 m`) | Đã kiểm Top/Side/Front + `preview` | `1.96 × 1.07 × 1.70 m` (khớp chuẩn xe máy có người, chiều rộng nhỉnh ~0.12 m bao trọn tay lái/gương) |

- **Vòng hiệu chuẩn & vật bỏ qua:** Làm bài cá nhân trên CVAT cục bộ; các cụm điểm ở rất xa (> 50 m) hoặc quá thưa điểm (< 15% thể tích cuboid, không phân biệt được rõ ràng trên camera tương ứng) được bỏ qua theo quy tắc `LOW_VISIBILITY` / `UNCERTAIN_SCOPE` để tránh đoán mò làm sai lệch IoU.

## 4. Bốn hiện tượng chất lượng (Bước 4)
Nguyên nhân ghi bên dưới là **giả thuyết**.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `s01f03` / `t4-1-bong-che.png` / task cá nhân | Giả thuyết: xe ở gần chắn tia quét LiDAR khiến tia không tới được phần thân sau của các xe nối đuôi phía sau | Phía sau cụm xe gần (`#4`, `#5`) xuất hiện khoảng tối không có điểm mặt đường và các xe sau bị mất một phần sườn | Bật cờ `Occluded` và suy luận kích thước hộp bao trọn phần bị che dựa vào ảnh camera và kích thước chuẩn |
| 2. Thưa ở xa | `s04f58` / `t4-2-thua-o-xa.png` / task cá nhân | Giả thuyết: các tia LiDAR phân kỳ theo góc quét nên mật độ tia trên một mét vuông giảm mạnh khi khoảng cách > 40 m | Xe `Car` #6 ở `45.05 m` chỉ còn vài vạch điểm thưa thớt (`61` điểm) không hiện rõ đủ 4 góc xe | Chỉ vẽ khi nhận diện được (> 15–20% thể tích và kiểm chứng trên `cam01`/`cam03`), dùng bảng 4.3 giữ kích thước hợp lý |
| 3. Phản xạ thấp / ít điểm bất thường | `s01f03` / `t4-3-phan-xa-thap.png` / task cá nhân | Giả thuyết: bề mặt xe tối màu hoặc kính chắn gió hấp thụ/khúc xạ tia laser làm số tia phản hồi về cảm biến rất ít | Một số vùng thân xe hoặc phần nóc/kính trên xe `Car` #11 bị hổng điểm, số điểm ít hơn các vật sáng màu cùng tầm | Không thu hẹp hộp theo phần điểm ít ỏi còn lại; nới hộp theo đúng khung gầm/nóc xe đối chiếu từ camera |
| 4. Điểm lạ | `s01f52` / `t4-4-diem-la.png` / task cá nhân | Giả thuyết: phản xạ đa đường (multipath) từ mặt đường ướt/kính xe hoặc nhiễu cảm biến tạo ra điểm ảo dưới mặt đất | Trong frame `s01f52` có `899` điểm nằm dưới ngưỡng `z = -2.3 m` (sâu hơn mặt đường `-1.75 m`) | Tuyệt đối không gán nhãn các cụm điểm lẻ loi giữa không trung hoặc dưới mặt đất thành vật thể |

## 5. Số từ cuboid của tôi (Bước 5)
Lấy từ `submission/D13-2A202603314-bangA.csv` và `submission/D13-2A202603314-tukiem.csv`.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `s03f12, #1, Car` | `-0.991 m` | `1.560 m` | `-1.771 m` | `4.54 m` | `4.65 m` | `270.1°` | Chạm đường (`-1.771 m` sát `-1.7 m`) |
| `s02f00, #1, Bus` | `-0.140 m` | `3.279 m` | `-1.780 m` | `9.25 m` | `9.25 m` | `270.0°` | Chạm đường (`-1.780 m` sát `-1.7 m`) |
| `s04f58, #6, Car` | `-0.836 m` | `1.617 m` | `-1.645 m` | `45.05 m` | `45.06 m` | `269.3°` | Chạm đường (`-1.645 m` sát `-1.7 m`) |

- **(a) Cuboid gần nhất và xa nhất:** Gần nhất là `s03f12, #1, Car` với `dist_m = 4.54 m` (`r = 4.65 m`). Xa nhất là `s04f58, #6, Car` với `dist_m = 45.05 m` (`r = 45.06 m`).
- **(b) So sánh kích thước với Bảng 4.3:**
  - `Car` (`s03f12, #1`): `l = 4.217 m`, `w = 1.891 m`, `h = 1.560 m` — nằm hoàn toàn trong khoảng tham chiếu Sedan/Hatchback (`3.8–5.0 m × 1.70–1.90 m × 1.40–1.60 m`).
  - `Cyclist` (`s04f28, #14`): `l = 1.964 m`, `w = 1.066 m`, `h = 1.703 m` — chiều dài và chiều cao nằm trong chuẩn Xe máy có người (`1.8–2.3 m` và `1.40–1.80 m`); chiều rộng `1.066 m` hơi rộng hơn `0.95 m` khoảng `0.12 m` do bao trọn phần gương chiếu hậu và tư thế người lái ở khoảng cách `11.79 m`.
- **(c) Góc `yaw_deg` theo chiều đi:** Xe đi cùng chiều phía trước (`s03f12, #1, Car`) có `yaw_deg = 270.1°` (rất sát `270°`); xe đi ngược chiều (`s02f33, #1, Car`) có `yaw_deg = 89.9°` (rất sát `90°`).

**Mật độ theo khoảng cách (4 cuboid `Car` ở các `dist_m` khác nhau):**

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| `s03f42, #1, Car` | `7.94 m` | `1814` |
| `s01f03, #3, Car` | `15.16 m` | `554` |
| `s02f33, #3, Car` | `24.93 m` | `193` |
| `s04f58, #6, Car` | `45.05 m` | `61` |

- **Quy luật:** Khi khoảng cách tăng gấp ~2 lần (từ `7.94 m` lên `15.16 m`), số điểm LiDAR trong hộp giảm còn khoảng 1/3 (`1814` xuống `554` điểm); ở `24.93 m` còn `193` điểm và ở `45.05 m` chỉ còn `61` điểm — phù hợp với đường cong mật độ chuẩn trên slide bài giảng (`~1968` điểm ở `0–10 m`, `~718` ở `10–20 m`, `~214` ở `20–30 m`, `~72` ở `30–40 m`).
- **Tự kiểm:** Sau khi chạy `preview` và `selfcheck`, đã sửa các hộp lỗi như `s01f52 #7, #9` (bị lơ lửng 0 điểm), `s02f00 #6` (bị chìm `-0.83 m`), `s05f17 #3` (bị lơ lửng `+0.50 m`), và bổ sung `Cyclist` `s04f28 #14`. Kết quả `selfcheck` cuối cùng giảm từ 14 cảnh báo xuống **0 / 79 cuboid có cảnh báo**.
- **Đổi kiểm:** Đã đổi kiểm chéo trên màn hình với bạn có MSSV **`2A202603300`**. Bạn đã chọn đối chiếu 2 con số trong mục 5 (`dist_m = 4.54 m` của `s03f12, #1, Car` và `z_bottom = -1.780 m` của `s02f00, #1, Bus`) với `submission/D13-2A202603314-bangA.csv`; cả hai con số đều khớp chính xác, không phát hiện sai lệch.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
- **(a) Ý nghĩa đối với mô hình học máy:** Nhãn 3D phải đo bằng mét (`x, y, z, l, w, h`) thay vì pixel vì các thuật toán lập kế hoạch đường đi và tránh va chạm của xe tự hành cần biết khoảng cách vật lý thực và thể tích không gian 3D bất biến theo góc nhìn camera. Vật thưa điểm cần quy tắc riêng (ngưỡng 15–20% và bảng kích thước tham chiếu) để tránh mô hình học sai kích thước thu nhỏ của xe hoặc gán nhãn nhầm nhiễu LiDAR thành vật thể thật.
- **(b) Sử dụng AI:**
  1. **Công cụ:** Antigravity AI Assistant.
  2. **Câu hỏi chính:** Kiểm tra cấu hình task CVAT, đối chiếu số liệu từ `bangA.csv`/`tukiem.csv` và kiểm tra các cảnh báo độ cao/kích thước cuboid.
  3. **Cách dùng kết quả:** Dùng để phát hiện nhanh các hộp bị chìm/lơ lửng và tổng hợp số liệu chính xác vào báo cáo sau khi tự chỉnh sửa trực tiếp trong CVAT.
- **(c) Điều còn chưa chắc:** Một số frame có thể còn các đối tượng nhỏ (`Pedestrian`, `Cyclist`) ở khoảng cách xa hoặc bị che khuất một phần chưa được gán nhãn đầy đủ do điểm LiDAR quá thưa.
