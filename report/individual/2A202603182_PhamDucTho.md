# Báo cáo Day 13 — Phạm Đức Thọ, 2A202603182

## 1. Kết luận trước
- **Thông tin bài làm:** Mã số sinh viên `2A202603182`, task cá nhân `D13-2A202603182` trên CVAT local. Bộ 10 frame được giao: `s01f26`, `s01f59`, `s02f13`, `s02f34`, `s03f15`, `s03f44`, `s04f07`, `s04f30`, `s05f19`, `s05f36`. Môi trường: Windows 11, trình duyệt Chrome/Edge, CVAT `v2.82.1`. Bạn ngồi cạnh (đổi kiểm): *(cập nhật MSSV bạn đổi kiểm sau khi kiểm chéo)*.
- **Tổng quan kết quả:** Đã vẽ tổng cộng **53 cuboid** dạng `Shape` trên 10 frame (`s01f26`: 8 cuboid; 9 frame còn lại mỗi frame 5 cuboid), gồm 50 `Car`, 2 `Cyclist` và 1 `Truck`. Phiếu tự kiểm `selfcheck` gắn cờ cảnh báo **10/53 cuboid** (chủ yếu liên quan đến độ cao đáy so với mặt đường, mép hộp sát điểm hoặc kích thước biên của class). Phần tự tin nhất là xác định đúng hệ trục VF (`-y` phía trước) và căn hướng đầu xe (`yaw_deg` sát 270° cho xe cùng chiều và 90° cho xe ngược chiều); phần còn yếu cần chú ý thêm là căn sát mặt đường ở góc nhìn Side/Front cho các xe ở xa (> 20 m) và ước lượng chính xác chiều cao/chiều dài khi vật bị che khuất một phần.
- **Cách dùng CVAT 3D:**
  1. **Đổi góc nhìn:** Kéo chuột trái để xoay khung 3D (xoay thẳng từ trên xuống để quan sát toàn cảnh 360°), kéo chuột phải để dời khung, lăn con lăn để phóng to/thu nhỏ, nháy đúp vào vùng trống để về góc nhìn mặc định; đổi camera bên phải bằng biểu tượng bánh răng trên từng khung ảnh.
  2. **Vẽ và chỉnh cuboid:** Chọn công cụ vẽ cuboid → chọn đúng class → bấm **Shape** (không dùng Track) → nháy đúp lên cụm điểm để đặt hộp; chọn hộp rồi chỉnh tâm, dài, rộng, xoay yaw ở khung **Top**, chỉnh độ cao và đáy chạm đường ở khung **Side / Front** (không xoay ở Side/Front).
  3. **Bật mũi tên hướng và kiểm tra trên ảnh camera:** Bật **Appearance → Cuboid orientation** để căn vạch đỏ ngắn (`+X` của cuboid) chỉ về đầu xe; bấm **Save** (`Ctrl+S`) rồi chạy `python scripts/lab.py preview` để chiếu các cuboid lên 7 ảnh camera và góc nhìn từ trên xuống trong `work/xem/`.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ: `s01f26` (ảnh `t1-s01f26-3d.png`, `t1-s01f26-top.png`) và `s01f59` (ảnh `t1-s01f59-3d.png`, `t1-s01f59-top.png` — ảnh nhìn từ trên xuống có đánh dấu phía trước `-y`, `+x`, `+y`).

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f26` | Phía trước (`-y`) hướng lên cạnh trên (khớp `cam01`); `+x` chỉ sang trái; `+y` chỉ xuống dưới (phía sau xe) | Đường tương đối phẳng (hơi nghiêng nhẹ theo độ dốc mặt đường); thấp hơn tâm LiDAR khoảng `~1,7 m` | Khoảng `35–45 m` | Xe con (`Car`), phương tiện lưu thông hai chiều, cây cối và vật cố định ven đường | Xe con phía sau bên phải cách `~7 m` (`#2`), xe con bên phải cách `~10,5 m` (`#3`) và các xe phía sau cách `~18–27 m` | Cụm điểm thưa ở xa (> 35 m) phía rìa đường bên phải, ít tia quét nên chưa đủ cơ sở xác định loại vật |
| `s01f59` | Phía trước (`-y`) hướng lên cạnh trên (khớp `cam01`); `+x` chỉ sang trái; `+y` chỉ xuống dưới (phía sau xe) | Mặt đường phẳng; thấp hơn tâm LiDAR khoảng `~1,7 m` | Khoảng `35–40 m` | Xe con (`Car`) di chuyển cùng chiều nhiều làn, dải phân cách và kiến trúc ven đường | Xe con sát bên trái cách `~3,0 m` (`#4`, đối chiếu `cam06`/`cam05`) và xe con ngay phía sau cách `~7,0 m` (`#1`, đối chiếu `cam09`) | Cụm điểm rời rạc phía xa sau đuôi xe (> 35 m) bị che khuất một phần bởi xe đi ngay phía sau |

**So sánh thông tin giữa Camera và LiDAR (từ `s01f26` và `s01f59`):**
- **3 điều chỉ Camera cho biết:** (1) Màu sắc xe và chi tiết bề mặt/đèn tín hiệu giúp phân biệt rõ đầu xe với đuôi xe; (2) Đặc điểm nhận dạng class chi tiết (phân biệt xe bán tải với SUV, người đang lái xe hai bánh với người dắt xe); (3) Vạch kẻ đường, tín hiệu giao thông và trạng thái môi trường ánh sáng.
- **3 điều chỉ LiDAR cho biết:** (1) Khoảng cách hình học chính xác theo mét (`x`, `y`, `z`, `dist_m`) tới từng vật thể; (2) Kích thước 3D thực tế (`length`, `width`, `height`) và độ cao đáy so với mặt đường; (3) Cấu trúc toàn cảnh 360° liên tục quanh xe, kể cả các vùng nằm trong góc mù giữa các camera.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| `s02f13` | Cạnh trên (`-y` hướng lên theo `cam01`) | Có nhiều xe đi ngược chiều ở làn bên trái (`yaw_deg` quanh 90°), có xe thân dài cần đối chiếu kỹ giữa Car và Truck |
| `s02f34` | Cạnh trên (`-y` hướng lên theo `cam01`) | Xe ở tầm 25–30 m bắt đầu thưa điểm, khó căn chính xác mặt đường ở góc Side/Front |
| `s03f15` | Cạnh trên (`-y` hướng lên theo `cam01`) | Có xe tải/bán tải phía trước bên trái (`#1`) và một xe con bị che khuất một phần (`#3`, đã bật `occluded`) |
| `s03f44` | Cạnh trên (`-y` hướng lên theo `cam01`) | Xuất hiện xe hai bánh có người lái (`Cyclist`, `#4`) ở phía sau bên phải cách `~12,5 m`, cần đổi sang camera sau-phải để đối chiếu |
| `s04f07` | Cạnh trên (`-y` hướng lên theo `cam01`) | Các xe đi cùng chiều trải dài từ `~5 m` đến `~25 m` cả phía trước và phía sau xe thu dữ liệu |
| `s04f30` | Cạnh trên (`-y` hướng lên theo `cam01`) | Có `Cyclist` ở phía sau bên phải (`#5`, `~8,2 m`) và xe ngược chiều bên trái (`#4`, `yaw_deg ≈ 90,6°`) |
| `s05f19` | Cạnh trên (`-y` hướng lên theo `cam01`) | Mặt đường vùng xung quanh một số xe có độ cao thay đổi nhẹ khiến đáy hộp dễ bị lệch nếu chỉ nhìn ở góc Top |
| `s05f36` | Cạnh trên (`-y` hướng lên theo `cam01`) | Có xe con ở khá xa phía trước (`#3`, `dist_m = 36,12 m`) mật độ điểm giảm mạnh so với các xe gần |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh minh họa trong `report/individual/images/`: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f26`, idx `2` | `Car` | Đúng (`yaw_deg = 272.7°`, chỉ về đầu xe cùng chiều) | Hơi chìm nhẹ (`z_bottom = -2.072 m`, cần nâng nhẹ ~0,3 m) | Có | Đúng (`l = 4.511 m`, `w = 1.946 m`, `h = 1.954 m` — trong khoảng SUV/Car) |
| (b) xe buýt hoặc xe tải | `s03f15`, idx `1` | `Truck` | Đúng (`yaw_deg = 270.0°`, chỉ thẳng về đầu xe) | Khá sát (`z_bottom = -1.464 m`) | Có | Dài `5.022 m`, rộng `1.978 m` sát biên bán tải; cao `1.661 m` hơi thấp so với ngưỡng `1.80 m` của bảng 4.3 |
| (c) người đi bộ hoặc xe hai bánh | `s04f30`, idx `5` | `Cyclist` | Đúng (`yaw_deg = 264.1°`, theo hướng di chuyển) | Chạm đường tốt (`z_bottom = -1.636 m`) | Có | Đúng (`l = 1.874 m`, `w = 0.878 m`, `h = 1.706 m` — khớp xe máy có người) |

- **Vòng hiệu chuẩn và các vật bỏ qua:** Qua đối chiếu guideline ở vòng hiệu chuẩn (`D13-HC485087`), điểm dễ lệch nhất giữa các học viên là độ cao đáy hộp (`z_bottom`) và việc căn vạch đỏ `+X` đúng chiều đầu xe; quy tắc thống nhất là luôn chỉnh đáy trên cả Side và Front, bật `Cuboid orientation` và đối chiếu ảnh camera trước khi lưu. Các vật bỏ qua vì không đủ cơ sở: các cụm điểm nhỏ ở xa khoảng `40–50 m` hoặc bị che khuất nặng ở rìa đường (không đạt ngưỡng 15–20% thể tích quan sát được, thuộc loại `LOW_VISIBILITY` / `UNCERTAIN_CLASS`) đã được bỏ qua thay vì đoán bừa.

## 4. Bốn hiện tượng chất lượng (Bước 4)
Nguyên nhân dưới đây đều được ghi dưới dạng **giả thuyết**.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `t4-1-bong-che.png` (task cá nhân `D13-2A202603182` / đối chiếu `s02f00` task chung) | Giả thuyết: tia laser từ LiDAR bị thân xe lớn hoặc vật chắn ở gần chặn đường truyền thẳng, không chiếu tới được vùng phía sau | Tạo thành một vùng quạt tối hoàn toàn trống điểm hoặc rất thưa điểm ngay phía sau vật chắn, trong khi hai bên vùng đó điểm vẫn dày | Vùng trống là vùng chưa quan sát được, không kết luận là đường trống; nếu thấy vật bị che một phần thì bật `occluded` (`Q`) và vẽ cuboid bao cả phần thể tích suy luận được |
| 2. Thưa ở xa | `t4-2-thua-o-xa.png` (task cá nhân `D13-2A202603182`, ví dụ vùng `> 30 m` ở `s05f36`) | Giả thuyết: các tia laser của LiDAR 40 tia phân kỳ theo góc đứng và góc ngang nên khoảng cách giữa các tia giãn rộng khi ra xa | Xe cách `30–50 m` chỉ còn lại vài chục đến hơn một trăm điểm rời rạc, mất nhiều chi tiết khung vỏ và khó thấy rõ bánh xe tiếp đất | Chỉ vẽ khi quan sát được tối thiểu ~15–20% thể tích hoặc đủ cơ sở suy luận kích thước từ camera + bảng 4.3; nếu không đủ cơ sở thì bỏ qua và ghi `LOW_VISIBILITY` |
| 3. Phản xạ thấp / ít điểm bất thường | `t4-3-phan-xa-thap.png` (task cá nhân `D13-2A202603182`) | Giả thuyết: bề mặt xe tối màu, kính chắn gió hoặc bề mặt nghiêng hấp thụ/phản xạ lệch hướng tia laser nên cảm biến thu về ít điểm phản hồi hơn | Một số phần thân xe (đặc biệt vùng kính, mũi xe tối màu) bị khuyết điểm hoặc mật độ điểm thấp hơn hẳn so với các xe sáng màu ở cùng khoảng cách | Ghi nhận nguyên nhân ở mức giả thuyết, không khẳng định chắc chắn; kết hợp ảnh camera và bảng 4.3 để bao trọn phần thân xe bị khuyết điểm thay vì thu hẹp hộp theo đám điểm còn lại |
| 4. Điểm lạ | `t4-4-diem-la.png` (task chung `D13-HC485087`, frame `s02f35`) | Giả thuyết: phản xạ đa đường (multipath) từ mặt đường/bề mặt phản chiếu hoặc nhiễu đo thời gian bay của cảm biến | Xuất hiện cụm điểm nằm lẻ loi hoặc tụt sâu xuống dưới mặt đường (ở `s02f35` là cụm điểm phía sau bên trái ở độ cao thấp hơn mặt đường nhiều mét) | Không gán nhãn thành vật thể vì đây là nhiễu/sai lệch hình học, tránh tạo nhãn dương tính giả (false positive) làm sai lệch dữ liệu huấn luyện |

## 5. Số từ cuboid của tôi (Bước 5)
Số liệu lấy trực tiếp từ `submission/D13-2A202603182-bangA.csv` và cột `points_in`, `ground_gap_m` của `submission/D13-2A202603182-tukiem.csv`.

| Cuboid (frame, idx, class) | z (m) | h (m) | z đáy = z − h/2 (m) | dist_m (m) | r = √(x²+y²+z²) (m) | yaw_deg (°) | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `s01f59`, #3, `Car` | -0.951 | 1.480 | -1.691 | 10.19 | 10.23 | 267.1 | Chạm đường tốt (`z_bottom` sát mức −1,7 m; `ground_gap_m = 0.11 m`) |
| `s05f19`, #3, `Car` | -0.537 | 1.429 | -1.251 | 10.72 | 10.73 | 274.7 | Lơ lửng (`z_bottom` cao hơn −1,7 m khoảng 0,45 m; `ground_gap_m = +0.47 m`, bị cờ) |
| `s02f34`, #4, `Car` | -1.851 | 1.535 | -2.618 | 26.30 | 26.36 | 81.9 | Chìm (`z_bottom` thấp hơn −1,7 m khoảng 0,92 m; `ground_gap_m = -0.64 m`, bị cờ) |

- **(a) Cuboid gần nhất và xa nhất:**
  - **Gần nhất:** `s01f59`, idx `4` (`Car`) có `dist_m = 2.96 m`, khoảng cách 3D `r = 3.10 m` (`points_in = 3235`).
  - **Xa nhất:** `s05f36`, idx `3` (`Car`) có `dist_m = 36.12 m`, khoảng cách 3D `r = 36.12 m` (`points_in = 128`). Toàn bộ 53 cuboid đều nằm trong ngưỡng `≤ 50 m`.
- **(b) So sánh kích thước với Bảng 4.3:**
  - **Xe hai bánh có người lái (`Cyclist`, `s04f30`, idx `5`):** `l = 1.874 m`, `w = 0.878 m`, `h = 1.706 m` — cả 3 chiều đều nằm trọn trong khoảng chuẩn của xe máy có người (dài 1,8–2,3 m; rộng 0,70–0,95 m; cao 1,4–1,8 m).
  - **Xe con (`Car`, `s01f59`, idx `3`):** `l = 4.635 m`, `w = 1.699 m`, `h = 1.480 m` — chiều dài và chiều cao đúng chuẩn sedan/hatchback (dài 3,8–5,0 m; cao 1,40–1,60 m), chiều rộng `1.699 m` sát biên dưới `1.70 m` (chênh `0.001 m`, nằm gọn trong sai số ±5%). Riêng trường hợp `s02f13`, idx `5` (`Car`) có `l = 6.595 m` vượt khoảng 3,8–5,3 m do hộp bị kéo dư phần đầu/đuôi hoặc cần kiểm tra lại class bán tải/xe tải.
- **(c) Góc `yaw_deg`:**
  - Xe đi cùng chiều phía trước (`s01f59`, idx `2`, `Car`): `yaw_deg = 270.6°` (rất sát `270°`).
  - Xe đi ngược chiều (`s02f13`, idx `3`, `Car`): `yaw_deg = 90.2°` (rất sát `90°`).

Mật độ theo khoảng cách (4 cuboid `Car` ở 4 khoảng `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m (m) | Số điểm (`points_in`) |
|---|---|---|
| `s01f59`, #1, `Car` | 6.98 | 1364 |
| `s03f15`, #5, `Car` | 15.00 | 465 |
| `s02f34`, #3, `Car` | 26.22 | 173 |
| `s05f36`, #3, `Car` | 36.12 | 128 |

- **Quy luật mật độ và tự kiểm:** Khi khoảng cách tăng gấp ~2 lần (từ `6.98 m` lên `15.00 m`), số điểm trong cuboid giảm còn khoảng 1/3 (`1364` → `465` điểm); khi xa gấp ~4–5 lần (`26–36 m`), số điểm giảm sâu chỉ còn khoảng 1/8 đến 1/10 (`173` và `128` điểm). Xu hướng này khớp với đường cong trung vị của bài giảng (xe con khoảng 1968 điểm ở 0–10 m, 718 ở 10–20 m, 214 ở 20–30 m, 72 ở 30–40 m, 19 ở 60–80 m).
- **Tự kiểm sau `preview` và `selfcheck`:** Đã rà soát các ảnh trong `work/xem/` và 10 cảnh báo trong `tukiem.csv`. Hai cuboid cần ưu tiên chỉnh lại trên CVAT (hoặc đã rà soát để sửa trước bản chốt): `s02f34 #4` (hộp đang chìm `ground_gap_m = -0.64 m`, cần nâng đáy ở góc Side/Front) và `s05f19 #3` (hộp lơ lửng `ground_gap_m = +0.47 m`, cần hạ đáy chạm mặt đường). Các cảnh báo về "điểm sát ngoài mép" ở `s01f26 #7` (51 điểm) và `s04f30 #2` (115 điểm) được giữ nguyên vì các điểm sát mép thuộc về mặt đường hoặc vật thể kế cận, nới thêm hộp sẽ gây bao dư.
- **Đổi kiểm với bạn bên cạnh:** *(Cập nhật MSSV bạn ngồi cạnh sau khi đối chiếu trực tiếp 2 con số trên màn hình, ví dụ `s01f59 #4 dist_m = 2.96 m` và `s05f36 #3 dist_m = 36.12 m`: khớp hoàn toàn với Bảng A).*

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
- **(a) Góc nhìn từ mô hình học máy 3D:** Nhãn 3D bắt buộc đo theo đơn vị **mét** thay vì pixel vì kích thước pixel trên ảnh camera thay đổi theo khoảng cách (phối cảnh), trong khi mô hình nhận thức xe tự lái cần dự đoán vị trí, kích thước thực và khoảng cách an toàn trong không gian 3D vật lý để lập kế hoạch chuyển động. Các vật thưa điểm cần quy tắc riêng (ngưỡng 15–20% thể tích và nguyên tắc "không đoán") vì nếu người gán nhãn tự bịa tâm, kích thước hoặc hướng khi thiếu điểm, nhãn sai sẽ trở thành nhiễu (noisy ground truth) làm mô hình học sai tương quan giữa point cloud và cuboid.
- **(b) Cách dùng AI:**
  - **Công cụ:** Antigravity (trợ lý AI trong IDE).
  - **Câu hỏi chính:** Hỏi hướng dẫn từng bước của repo, chạy lệnh cài đặt `.venv`, tạo task CVAT, chạy `sync`/`pack`/`check_submission.py`, hỏi khái niệm hệ trục VF (`-y`, `+x`, `+y`), khác biệt giữa `Shape` và `Track`, và hỗ trợ tổng hợp khung báo cáo từ số liệu Bảng A/Phiếu tự kiểm để tự rà soát lại.
  - **Cách sử dụng kết quả:** Dùng để khắc phục lỗi cấu hình môi trường/CVAT nhanh chóng, hiểu đúng quy tắc gán nhãn, và đối chiếu kiểm tra lại toàn bộ số liệu, nhận xét trong báo cáo trước khi nộp.
- **(c) Điều còn chưa chắc:** Một số cuboid ở xa (> 20 m) hoặc có độ dốc mặt đường thay đổi (như `s02f34 #4`, `s05f19 #3`, `s05f19 #4`) vẫn còn chênh lệch độ cao đáy trên `tukiem.csv`, và một trường hợp (`s02f13 #5`, `length = 6.595 m` hoặc `s03f15 #1`, `height = 1.661 m`) nằm sát/ngoài biên kích thước bảng 4.3 cần kiểm tra kỹ lại trên CVAT nếu có thêm thời gian chỉnh sửa.
