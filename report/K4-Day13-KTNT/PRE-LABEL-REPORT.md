# Báo cáo thực hành PointPillars — Day 13

Giữ bản đã điền ngoài Git, trong thư mục nhóm private do LC thu. Đây là kiểm tra formative; không ghi điểm của người khác.

## Nhóm và provenance

- Mã nhóm/phòng: KTNT
- Thành viên: xem `TEAMMATES.md` — Trần Đăng Khang, Lê Thanh Tùng, Nguyễn Công Thành, Đoàn Vĩnh Nguyên (vai trò luân phiên vận hành / kiểm JSON / xem hình học / ghi log qua A–B–C).
- Trạng thái: `executed-by-group` (smoke.json `status: passed`).
- Người thực sự chạy; ngày/giờ; hệ máy/architecture: lượt A — Trần Đăng Khang, lượt B — Lê Thanh Tùng, lượt C — Nguyễn Công Thành (vận hành); 2026-10-01, 07:45–07:51 UTC (docker-load 191 s, A 54 s, B 12 s, C 10 s, QC 7 s); Docker CPU linux/amd64, giới hạn 4 CPU / 4 GB.
- Image tag và image ID; phiên bản repo: `day13-pointpillars:lc-20261001-amd64`, ID `sha256:e03983bd922e…2c82`; repo `0831856d9216…b7b1` (working tree dirty = true), preannotate sha256 `65edf6ac…b5ca`.
- PCD được cấp / frame_id; nơi được phép chạy; fingerprint nếu LC cấp: `input/demo.pcd` — KITTI 000008 (MMDetection3D demo, CC BY-NC-SA 3.0, 17 238 điểm); chạy trên laptop nhóm/máy LC qua gói Student; PCD sha256 `3b5ea3da13e2b19149cab6a8d521c2ca55f2df93f026b5a3f8c273ce70645d60` (nguồn .bin sha256 `3b9de6cc…02d1`).
- Checkpoint: PointPillars KITTI có sẵn trong image; `/opt/PointPillars/pretrained/epoch_160.pth`, sha256 `482dfcf63b93…b5b1` (lấy từ smoke.json).
- Phạm vi: front-window; score threshold: 0.3 (`--from KITTI`, không dùng `--full-scene`), giữ nguyên cho cả A/B/C.
- Giả định kênh thứ tư/intensity và nguồn z_ground: PCD đã bỏ reflectance thật, RGB=0 chỉ là placeholder; adapter dùng kênh hằng số — không phải intensity phục hồi. `z_ground` ước lượng từ chính PCD; PCD đã dịch z +1.73 m so với KITTI gốc (x/y giữ nguyên). `z_model = z_source - z_ground - delta`. Mọi JSON ghi `z_ground = 0.075` m.

## Ba lượt inference thật

A/B/C là ba lượt trên cùng PCD. Runner chạy đủ ba lượt từ một lệnh. Lấy **Số hộp** từ `n_boxes`, **mean_z** từ `mean_z` trong `run-A/B/C/summary.csv`; không tự tính lại hoặc đoán. `mean_z` không phải điểm chất lượng. Mở `side-*.png`, đối chiếu `boxes-*.json` để ghi quan sát. Số hộp không phải đáp án cần khớp nhóm khác.

| Lượt | delta | Pillar XY | Số hộp | mean_z | File JSON/Side/CSV | Quan sát có bằng chứng |
| --- | --- | --- | --- | --- | --- | --- |
| A | 0 | 0.16 | 1 | 0.330 | `run-A/boxes-demo-delta-0-voxel-0.16.json`, `side-demo-delta-0-voxel-0.16.png`, `summary.csv` | Chỉ 1 hộp `vehicles` x≈13.15, y≈-0.45, score 0.32 (sát ngưỡng); trên Side đáy hộp xuống ≈ -0.4 m, dưới đường z=0, trong khi cụm điểm xe x≈6–25 m không có hộp. |
| B | 1.73 | 0.16 | 13 | 1.034 | `run-B/boxes-demo-delta-1.73-voxel-0.16.json`, `side-…-1.73-voxel-0.16.png`, `summary.csv` | 10 vehicles + 1 two-wheels + 2 pedestrian; 4 hộp score ≥ 0.84 (x≈8.1, 14.8, 6.5, 33.7). Đáy hộp vùng x≈3–27 m nằm quanh z≈0, khớp mặt đất; nhiều hộp xa (x 34–56 m) trùng vùng điểm thưa/cao. |
| C | 1.73 | 0.32 | 6 | 1.091 | `run-C/boxes-demo-delta-1.73-voxel-0.32.json`, `side-…-1.73-voxel-0.32.png`, `summary.csv` | Cả 6 hộp đều là `pedestrian` (score 0.30–0.81), không còn hộp `vehicles` nào, kể cả chỗ B có xe score 0.93 (x≈14.8, y≈-1.1 → C có pedestrian x≈13.2, y≈-0.95). |

- A/B — chỉ đổi delta: A có 1 hộp; B có 13 hộp. Trong `side-…delta-0…png` chỉ có một hộp ở x≈11–15 m, đáy chìm dưới z=0, còn `side-…delta-1.73…png` có hộp trải x≈3–56 m và đáy bám z≈0. Hộp A (x 13.15, y -0.45, yaw 2.67) không trùng hộp B nào và không phải hộp B dịch z → đây là chạy lại model trên input khác (độ cao điểm lệch 1.73 m so với giả định sensor KITTI), không chỉ dịch hộp cũ. Điều em còn chưa chắc: không có nhãn thật nên chưa biết bao nhiêu trong 13 hộp B là đúng; hộp score thấp (pedestrian 0.32–0.34, two-wheels 0.38) có thể là false positive.
- B/C — chỉ đổi pillar: B có 13 hộp; C có 6 hộp. Ảnh `side-…voxel-0.32.png` khác `side-…voxel-0.16.png` ở chỗ mất toàn bộ hộp xe dài ~3.6–4.2 m, thay bằng hộp hẹp ~0.6–1.1 m. Số lượng/lớp/vị trí: 13 → 6; lớp từ chủ yếu vehicles sang 100% pedestrian; một số vị trí gần nhau (B xe x≈14.77,y≈-1.08 ↔ C ped x≈13.24,y≈-0.95; B two-wheels x≈10.32,y≈5.25 ↔ C ped x≈10.46,y≈4.93). Có đủ bằng chứng để kết luận tốt hơn không? Không — checkpoint được train với pillar 0.16, đổi 0.32 làm lệch biểu diễn input nên kết quả C nhiều khả năng sai lớp; nhưng không có nhãn thật nên không kết luận B “đúng”, chỉ nói C không nhất quán với B.
- Giới hạn ROI và góc Side ảnh hưởng cách đọc miss/yaw thế nào? Chỉ chạy front-window nên vật ngoài cửa sổ (x<0, |y| lớn) không tính là miss. Side là hình chiếu x-z: các vật khác y bị chồng lên nhau (ví dụ x≈8–11 m có 3 hộp B chồng), không thấy y và yaw → phải đối chiếu tọa độ/yaw trong JSON, không suy yaw từ ảnh Side; đường z=0 chỉ là tham chiếu, không phải mặt đường cục bộ.
- JSON nào còn chưa đủ cơ sở để import? Cần kiểm gì tiếp? Không file nào nên import (đây là KITTI demo, không phải job Robotaxi). Riêng A (1 hộp, đáy dưới mặt đất, score 0.32) và C (toàn pedestrian) rõ ràng không đủ cơ sở; B cũng cần kiểm: xem BEV/3D từng hộp, hộp score < 0.5, hộp xa x > 40 m, xác nhận đáy hộp khớp mặt đất cục bộ và yaw khớp hướng đám điểm.

## Ca QC có kiểm soát — không import CVAT

| Ca | Số hộp lệch z / tổng hộp | Lượng lệch | Class/x/y/yaw có đổi? | Dừng batch, kiểm từng hộp hay chưa rõ? | Bằng chứng |
| --- | --- | --- | --- | --- | --- |
| case-correct | 0 / 13 | 0 m | Không | Không dừng — bản sao prediction B giữ đúng phép chuyển ngược | `qc-cases/case-correct.json`, `side-correct.png` |
| case-batch-z | 13 / 13 | -1.805 m (= delta 1.73 + z_ground 0.075) | Không | Dừng cả batch — mọi hộp lệch cùng một lượng, dấu hiệu quên phép z ngược; sửa script/chuyển đổi rồi chạy lại, không sửa tay từng hộp | `qc-cases/case-batch-z.json` (vd hộp 1 z 0.92 → -0.88), `side-batch-z.png` |
| case-one-box-z | 1 / 13 | -1.805 m (chỉ hộp 1 vehicles x≈8.09, y≈1.21: z 0.92 → -0.88) | Không | Không dừng batch; kiểm và sửa riêng hộp đó, ghi lại để theo dõi nếu lặp lại | `qc-cases/case-one-box-z.json`, `side-one-box-z.png` |

Ba ca do helper tạo biến đổi có chủ đích từ prediction B (`source_prediction_sha256 c2a8db24…cc80`, `training_only: true`); không phải kết quả inference riêng, không phải nhãn đúng và không import vào CVAT.

## Nhận xét cá nhân

Mỗi thành viên tự viết một mục: vai trò đã làm; một quan sát A/B/C có dẫn file hoặc hộp/vùng; diễn giải phép z thuận/ngược; một quyết định lỗi batch và hành động; điều chưa chắc. Chỉ đọc kết quả chuẩn bị trước thì ghi rõ chưa tự chạy.

### Trần Đăng Khang

- **Vai trò:** A — vận hành; B — ghi log; C — xem hình học.
- **Quan sát:** Lượt A (`run-A/side-demo-delta-0-voxel-0.16.png`) chỉ ra 1 hộp `vehicles` ở x≈13.15, y≈-0.45, score 0.32, đáy hộp xuống khoảng -0.4 m (dưới đường z=0). Cụm điểm xe dày ở x≈6–25 m không có hộp nào.
- **Phép z:** Thuận: `z_model = z_source - z_ground - delta`. Ngược khi xuất: `z_source = z_model + z_ground + delta`. Với delta=0, độ cao điểm đưa vào model lệch khoảng 1.73 m so với giả định sensor KITTI, nên model gần như không nhận ra xe.
- **Quyết định lỗi batch:** `case-batch-z` có 13/13 hộp lệch cùng -1.805 m → dừng cả batch, báo người phụ trách script, sửa bước chuyển ngược rồi chạy lại; không sửa tay từng hộp.
- **Điều chưa chắc:** Hộp duy nhất của A có phải xe thật hay chỉ là phát hiện ngẫu nhiên sát ngưỡng 0.3.

### Lê Thanh Tùng

- **Vai trò:** A — kiểm JSON; B — vận hành; C — ghi log.
- **Quan sát:** `run-B/boxes-demo-delta-1.73-voxel-0.16.json` có 13 hộp (10 vehicles, 1 two-wheels, 2 pedestrian). Hộp A (x 13.15, y -0.45, yaw 2.67) không trùng hộp B nào; hộp B gần nhất ở x 14.77, y -1.08, yaw -0.30 → đây là kết quả chạy lại model, không phải hộp cũ bị dịch z.
- **Phép z:** JSON đã ở hệ nguồn (đã cộng lại `z_ground + delta` = 0.075 + 1.73), nên khi đọc không được cộng thêm lần nữa. Nếu cộng hai lần, hộp sẽ nổi lên khoảng 1.8 m.
- **Quyết định lỗi batch:** `case-one-box-z` chỉ hộp đầu (x≈8.09, y≈1.21) có z từ 0.92 thành -0.88, các hộp còn lại giữ nguyên → không dừng batch, đánh dấu và sửa riêng hộp đó, ghi vào log để theo dõi.
- **Điều chưa chắc:** Các hộp score thấp ở B (pedestrian 0.32–0.34, two-wheels 0.38) có phải false positive không.

### Nguyễn Công Thành

- **Vai trò:** A — xem hình học; B — kiểm JSON; C — vận hành.
- **Quan sát:** Lượt C (`run-C/boxes-demo-delta-1.73-voxel-0.32.json`) có 6 hộp, tất cả là `pedestrian`, không còn hộp `vehicles`. Tại vị trí B có xe score 0.93 (x≈14.77, y≈-1.08), C chỉ có pedestrian ở x≈13.24, y≈-0.95 với kích thước 1.07×0.68 m.
- **Phép z:** B và C cùng delta=1.73 và z_ground=0.075, nên khác biệt giữa B và C không đến từ phép z mà từ cỡ pillar 0.32 m. Checkpoint được train với 0.16 m, nên biểu diễn đầu vào đã lệch.
- **Quyết định lỗi batch:** Nếu cả lô dùng nhầm cấu hình pillar 0.32 (lớp bị đổi hàng loạt) thì đó cũng là lỗi batch: dừng pipeline, sửa cấu hình về đúng giá trị checkpoint và chạy lại.
- **Điều chưa chắc:** Không có nhãn thật nên chỉ kết luận được là C không nhất quán với B, chưa khẳng định được B đúng.

### Đoàn Vĩnh Nguyên

- **Vai trò:** A — ghi log; B — xem hình học; C — kiểm JSON.
- **Quan sát:** Trên `run-B/side-demo-delta-1.73-voxel-0.16.png`, đáy các hộp vùng x≈3–27 m bám quanh z≈0. Vùng x≈8–11 m có 3 hộp chồng nhau trên hình chiếu x-z (khác nhau về y: 1.21, 4.23, 5.25), nên phải đọc JSON mới tách được.
- **Phép z:** Trên `qc-cases/side-batch-z.png`, mọi hộp bị kéo xuống cùng 1.805 m, đúng bằng `z_ground + delta`; đây là dấu hiệu quên phép chuyển ngược `z_source = z_model + z_ground + delta`.
- **Quyết định lỗi batch:** Khi mọi hộp lệch cùng một lượng mà class/x/y/yaw không đổi → lỗi hệ thống, dừng batch và không đưa file nào vào CVAT.
- **Điều chưa chắc:** Ảnh Side không cho thấy y và yaw, và đường z=0 chỉ là tham chiếu, không phải mặt đường cục bộ ở mọi vị trí.

## LC ghi nhận riêng

- Quyền dùng PCD/image và đúng ca: PCD KITTI 000008 trong gói Student (CC BY-NC-SA 3.0, học thuật phi thương mại, sha256 khớp provenance); image `day13-pointpillars:lc-20261001-amd64` từ archive trong ZIP; chạy ngày 2026-10-01. Không dùng dữ liệu Robotaxi. — *LC xác nhận:*
- Có chạy thật / chỉ phân tích; còn cần lượt thực hành bổ sung: Chạy thật trên máy nhóm (smoke.json `status: passed`, đủ A/B/C + QC). Chưa thấy cần lượt bổ sung. — *LC xác nhận:*
- Output đủ, giữ bản gốc, không đưa ca lỗi vào CVAT: Đủ `smoke.json`, `run-A/B/C` (JSON/Side/CSV), `qc-cases` (3 JSON + 3 Side + manifest); giữ nguyên không sửa; không import file nào vào CVAT. — *LC xác nhận:*
- Nhận xét từng thành viên và quyết định dừng pipeline: Đủ 4 nhận xét cá nhân. Quyết định chung: `case-batch-z` → dừng cả batch, sửa phép z ngược; `case-one-box-z` → sửa riêng hộp; cấu hình C (pillar 0.32) không dùng. — *LC xác nhận:*
- Đồng ý chuyển sang chỉnh/QC / cần bổ sung; lý do: Nhóm đề nghị chuyển sang chỉnh/QC Robotaxi trong CVAT vì đã hoàn thành A/B/C, ca QC và nhận xét. — *LC quyết định:*
