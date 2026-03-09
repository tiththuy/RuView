# Nhật Ký Thay Đổi

Tất cả các thay đổi đáng chú ý của dự án này sẽ được ghi lại trong file này.

Định dạng dựa trên [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
và dự án này tuân theo [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Đã thêm
- **Hoàn thiện API giao diện sensing server (ADR-043)** — 14 REST endpoints hoạt động đầy đủ cho quản lý model, ghi CSI, và điều khiển huấn luyện
  - Model CRUD: `GET /api/v1/models`, `GET /api/v1/models/active`, `POST /api/v1/models/load`, `POST /api/v1/models/unload`, `DELETE /api/v1/models/:id`, `GET /api/v1/models/lora/profiles`, `POST /api/v1/models/lora/activate`
  - Ghi CSI: `GET /api/v1/recording/list`, `POST /api/v1/recording/start`, `POST /api/v1/recording/stop`, `DELETE /api/v1/recording/:id`
  - Điều khiển huấn luyện: `GET /api/v1/train/status`, `POST /api/v1/train/start`, `POST /api/v1/train/stop`
  - Ghi chép CSI frames vào file `.jsonl` qua tokio background task
  - Thư mục model/recording được quét khi khởi động, trạng thái được quản lý qua `Arc<RwLock<AppStateInner>>`
- **ADR-044: Cải tiến công cụ provisioning** — Kế hoạch 5 giai đoạn cho phạm vi NVS đầy đủ (7 keys còn thiếu), file cấu hình JSON, mesh presets, đọc lại/xác minh, và tự động phát hiện
- **25 mobile tests thực tế** thay thế các placeholder `it.todo()` — 205 assertions bao gồm components, services, stores, hooks, screens, và utils
- **Project MERIDIAN (ADR-027)** — Tổng quát hóa domain xuyên môi trường cho WiFi pose estimation (1.858 dòng, 72 tests)
  - `HardwareNormalizer` — Nội suy cubic Catmull-Rom lấy mẫu lại bất kỳ CSI phần cứng nào về 56 subcarriers chuẩn; z-score + làm sạch pha
  - `DomainFactorizer` + `GradientReversalLayer` — Tách biệt đối kháng các đặc trưng liên quan tư thế vs đặc trưng môi trường
  - `GeometryEncoder` + `FilmLayer` — Mã hóa vị trí Fourier + DeepSets + FiLM cho triển khai zero-shot khi biết vị trí AP
  - `VirtualDomainAugmentor` — Đa dạng hóa môi trường tổng hợp (tỷ lệ phòng, vật liệu tường, scatterers, nhiễu) cho tăng cường huấn luyện 4x
  - `RapidAdaptation` — Hiệu chỉnh không giám sát 10 giây qua contrastive test-time training + LoRA adapters
  - `CrossDomainEvaluator` — Giao thức đánh giá 6 chỉ số (MPJPE in-domain/cross-domain/few-shot/cross-hardware, tỷ lệ domain gap, tốc độ thích nghi)
- ADR-027: Tổng quát hóa Domain Xuyên Môi trường — 10 trích dẫn SOTA (PerceptAlign, X-Fi ICLR 2025, AM-FM, DGSense, CVPR 2024)
- **Các adapter RSSI đa nền tảng** — Adapter Rust macOS CoreWLAN (`MacosCoreWlanScanner`) và Linux `iw` (`LinuxIwScanner`) với điều kiện `#[cfg(target_os)]`
- Adapter sensing Python macOS CoreWLAN với trình trợ giúp Swift (`mac_wifi.swift`)
- Tạo BSSID tổng hợp macOS (FNV-1a hash) cho redaction BSSID Sonoma 14.4+
- Trình phân tích cú pháp `iw dev <iface> scan` Linux với chuyển đổi tần số sang kênh và chế độ `scan dump` (không cần root)
- ADR-025: macOS CoreWLAN WiFi Sensing (ORCA)

### Đã sửa
- **Sự cố crash sendto ENOMEM (Issue #127)** — Các CSI callbacks trong chế độ promiscuous làm cạn kiệt pool lwIP pbuf gây crash guru meditation. Đã sửa với bộ giới hạn tốc độ 50 Hz trong `csi_collector.c` và backoff ENOMEM 100 ms trong `stream_sender.c`. Đã xác minh trên phần cứng ESP32-S3 (200+ callbacks, không crash)
- **Script provisioning thiếu cờ TDM/edge (Issue #130)** — Đã thêm `--tdm-slot`, `--tdm-total`, `--edge-tier`, `--pres-thresh`, `--fall-thresh`, `--vital-win`, `--vital-int`, `--subk-count` vào `provision.py`
- **WebSocket "RECONNECTING" trên Dashboard/Live Demo** — `sensingService.start()` bây giờ được gọi khi khởi tạo ứng dụng trong `app.js` để WebSocket kết nối ngay thay vì chờ truy cập tab Sensing
- **Cổng WebSocket trên mobile** — `ws.service.ts` `buildWsUrl()` sử dụng cổng same-origin thay vì cổng hardcoded 3001
- **Cấu hình Jest mobile** — `testPathIgnorePatterns` không còn bỏ qua toàn bộ thư mục test một cách thầm lặng
- Đã loại bỏ bộ đếm byte tổng hợp khỏi Python `MacosWifiCollector` — bây giờ báo cáo `tx_bytes=0, rx_bytes=0` thay vì giá trị tăng dần giả

---

## [3.0.0] - 2026-03-01

Phiên bản chính: Model embedding contrastive AETHER, Docker Hub images, và cải tiến toàn diện giao diện người dùng.

### Đã thêm — Model Embedding Contrastive AETHER (ADR-024)
- **Project AETHER** — Học contrastive tự giám sát cho WiFi CSI fingerprinting, tìm kiếm độ tương đồng, và phát hiện bất thường (`9bbe956`)
- Module `embedding.rs`: `ProjectionHead`, `InfoNceLoss`, `CsiAugmenter`, `FingerprintIndex`, `PoseEncoder`, `EmbeddingExtractor` (909 dòng, không có dependencies ML bên ngoài)
- Tiền huấn luyện kiểu SimCLR với 5 augmentations có động lực vật lý (jitter thời gian, che subcarrier, nhiễu Gaussian, xoay pha, tỷ lệ biên độ)
- Cờ CLI: `--pretrain`, `--pretrain-epochs`, `--embed`, `--build-index <type>`
- Bốn loại fingerprint index tương thích HNSW: `env_fingerprint`, `activity_pattern`, `temporal_baseline`, `person_track`
- `PoseEncoder` đa phương thức cho căn chỉnh embedding WiFi-to-camera
- Regularization VICReg để ngăn embedding collapse
- Tổng 53K tham số (55 KB ở INT8) — vừa với ESP32

### Đã thêm — Docker & Triển Khai
- Đã phát hành Docker Hub images: `ruvnet/wifi-densepose:latest` (132 MB Rust) và `ruvnet/wifi-densepose:python` (569 MB) (`add9f19`)
- Dockerfile đa giai đoạn cho Rust sensing server với RuVector crates
- `docker-compose.yml` điều phối cả hai dịch vụ Rust và Python
- Xuất model RVF qua `--export-rvf` và tải qua cờ CLI `--load-rvf`

### Đã thêm — Tài Liệu
- 33 trường hợp sử dụng trên 4 tầng dọc: Hàng ngày, Chuyên biệt, Robotics & Công nghiệp, Cực đoan (`0afd9c5`)
- Bảng so sánh "Tại Sao WiFi Chiến Thắng" (WiFi vs camera vs LIDAR vs thiết bị đeo vs PIR)
- Sơ đồ kiến trúc Mermaid: pipeline end-to-end, chi tiết xử lý tín hiệu, cấu trúc liên kết triển khai (`50f0fc9`)
- Phần Models & Training với liên kết crate RuVector (GitHub + crates.io), bảng thành phần SONA (`965a1cc`)
- Phần container RVF với bảng mục tiêu triển khai (ESP32 0,7 MB đến server 50+ MB)
- Các phần README có thể thu gọn để điều hướng tốt hơn (`478d964`, `99ec980`, `0ebd6be`)
- Cài đặt và Bắt đầu Nhanh được di chuyển lên trên Mục lục (`50acbf7`)
- Thông báo yêu cầu phần cứng CSI (`528b394`)

### Đã sửa
- **Giao diện người dùng tự động phát hiện cổng server từ nguồn gốc trang** — không còn hardcoded `localhost:8080`; hoạt động trên bất kỳ cổng nào (Docker :3000, native :8080, tùy chỉnh) (`3b72f35`, đóng #55)
- **Docker port mismatch** — server bây giờ bind 3000/3001 bên trong container như đã ghi lại (`44b9c30`)
- Đã thêm route WebSocket `/ws/sensing` vào HTTP server để giao diện người dùng chỉ cần một cổng
- Đã sửa tham chiếu API endpoint trong README: `/api/v1/health` → `/health`, `/api/v1/sensing` → `/api/v1/sensing/latest`
- Giới hạn theo dõi đa người đã được sửa: mặc định có thể cấu hình 10, không có giới hạn phần mềm cứng (`e2ce250`)

---

## [2.0.0] - 2026-02-28

Phiên bản chính: Rust sensing server hoàn chỉnh, pipeline huấn luyện DensePose đầy đủ, tích hợp RuVector v2.0.4, firmware ESP32-S3, và 6 bản vá tăng cường bảo mật.

### Đã thêm — Rust Sensing Server
- **REST API đầy đủ tương thích DensePose** phục vụ bởi Axum (`d956c30`)
  - `GET /health` — trạng thái server
  - `GET /api/v1/sensing/latest` — dữ liệu CSI sensing trực tiếp
  - `GET /api/v1/vital-signs` — nhịp thở (6-30 BPM) và nhịp tim (40-120 BPM)
  - `GET /api/v1/pose/current` — 17 COCO keypoints được suy ra từ trường tín hiệu WiFi
  - `GET /api/v1/info` — thông tin build và tính năng server
  - `GET /api/v1/model/info` — metadata container model RVF
  - `ws://host/ws/sensing` — luồng WebSocket thời gian thực
- Ba nguồn dữ liệu: `--source esp32` (UDP CSI), `--source windows` (netsh RSSI), `--source simulated` (tham chiếu tất định)
- Tự động phát hiện: server thăm dò ESP32 UDP và Windows WiFi, rơi về simulated
- Giao diện người dùng Three.js với bộ khung xương 3D, heatmap tín hiệu, đồ thị pha, Doppler bars, bảng vital signs
- Phục vụ giao diện người dùng tĩnh qua cờ `--ui-path`
- Thông lượng: 9.520–11.665 frames/giây (bản release)

### Đã thêm — ADR-021: Phát Hiện vital signs
- `VitalSignDetector` với trích xuất nhịp thở (6-30 BPM) và nhịp tim (40-120 BPM) từ CSI fluctuations (`1192de9`)
- Phân tích phổ FFT với bộ lọc band-pass có thể cấu hình
- Tính điểm confidence dựa trên độ nổi bật đỉnh phổ
- REST endpoint `/api/v1/vital-signs` với đầu ra JSON thời gian thực

### Đã thêm — ADR-023: Pipeline Huấn Luyện DensePose (Giai đoạn 1-8)
- Crate `wifi-densepose-train` với pipeline 8 giai đoạn hoàn chỉnh (`fc409df`, `ec98e40`, `fce1271`)
  - Giai đoạn 1: `DataPipeline` với bộ tải dataset MM-Fi và Wi-Pose
  - Giai đoạn 2: `CsiToPoseTransformer` — cross-attention 4 đầu + GCN 2 lớp trên COCO skeleton
  - Giai đoạn 3: Loss tổng hợp 6 thành phần (MSE, độ dài xương, đối xứng, góc khớp, thời gian, confidence)
  - Giai đoạn 4: `DynamicPersonMatcher` qua ruvector-mincut (gán Hungary O(n^1.5 log n))
  - Giai đoạn 5: `SonaAdapter` — MicroLoRA rank-4 với bảo tồn bộ nhớ EWC++
  - Giai đoạn 6: `SparseInference` — tải model 3 lớp lũy tiến (A: cơ bản, B: tinh chỉnh, C: đầy đủ)
  - Giai đoạn 7: `RvfContainer` — đóng gói model một file với định dạng nhị phân dựa trên segment
  - Giai đoạn 8: Huấn luyện end-to-end với cosine-annealing LR, early stopping, lưu checkpoint
- CLI: `--train`, `--dataset`, `--epochs`, `--save-rvf`, `--load-rvf`, `--export-rvf`
- Benchmark: ~11.665 fps inference, 229 tests đạt

### Đã thêm — ADR-016: Tích Hợp Huấn Luyện RuVector (tất cả 5 crates)
- `ruvector-mincut` → `DynamicPersonMatcher` trong `metrics.rs` + lựa chọn subcarrier (`81ad09d`, `a7dd31c`)
- `ruvector-attn-mincut` → attention anten trong `model.rs` + spectrogram gated nhiễu
- `ruvector-temporal-tensor` → `CompressedCsiBuffer` trong `dataset.rs` + nhịp thở/nhịp tim nén
- `ruvector-solver` → nội suy subcarrier thưa (114→56) + tam giác đo Fresnel
- `ruvector-attention` → spatial attention trong `model.rs` + BVP có trọng số attention
- Đã vendored tất cả 11 RuVector crates trong `vendor/ruvector/` (`d803bfe`)

### Đã thêm — ADR-017: Tích Hợp Tín Hiệu RuVector & MAT (7 điểm tích hợp)
- `gate_spectrogram()` — suppression nhiễu gated attention (`18170d7`)
- `attention_weighted_bvp()` — profile vận tốc có trọng số độ nhạy
- `mincut_subcarrier_partition()` — phân chia dynamic subcarrier nhạy/không nhạy
- `solve_fresnel_geometry()` — ước lượng khoảng cách TX-body-RX
- `CompressedBreathingBuffer` + `CompressedHeartbeatSpectrogram`
- `BreathingDetector` + `HeartbeatDetector` (crate MAT, FFT thực + micro-Doppler)
- Được bật bởi feature `cfg(feature = "ruvector")` (`ab2453e`)

### Đã thêm — ADR-018: Firmware ESP32-S3 & Pipeline CSI Trực Tiếp
- Firmware ESP32-S3 với trích xuất CSI FreeRTOS (`92a5182`)
- Định dạng frame nhị phân ADR-018: `[0xAD, 0x18, len_hi, len_lo, payload]`
- Rust `Esp32Aggregator` nhận UDP frames trên cổng 5005
- `bridge.rs` chuyển đổi cặp I/Q thành vector biên độ/pha
- NVS provisioning cho thông tin xác thực WiFi
- Tài liệu khởi động nhanh binary có sẵn (`696a726`)

### Đã thêm — ADR-014: Xử Lý Tín Hiệu SOTA
- 6 thuật toán, 83 tests (`fcb93cc`)
  - Bộ lọc Hampel (median + MAD, kháng đến 50% contamination)
  - Nhân liên hợp (tỷ lệ anten tham chiếu, triệt nhiễu common-mode)
  - Làm sạch pha (unwrap + detrend tuyến tính, loại bỏ CFO/SFO)
  - Hình học vùng Fresnel (khoảng cách TX-body-RX từ vật lý nguyên lý đầu tiên)
  - Body Velocity Profile (trích xuất micro-Doppler, tăng tốc 5,7x)
  - Spectrogram gated attention (suppression nhiễu có học)

### Đã thêm — ADR-015: Chiến Lược Huấn Luyện Bộ Dữ Liệu Công Khai
- Đặc điểm kỹ thuật bộ dữ liệu MM-Fi và Wi-Pose với liên kết tải xuống (`4babb32`, `5dc2f66`)
- Đã xác minh kích thước bộ dữ liệu, tốc độ lấy mẫu, và định dạng annotation
- Giao thức đánh giá cross-dataset

### Đã thêm — Module Phát Hiện Thảm Họa WiFi-Mat
- Tam giác đo đa AP cho phát hiện người sống sót xuyên tường (`a17b630`, `6b20ff0`)
- Phân loại triage (nhịp thở, nhịp tim, chuyển động)
- Domain events: `survivor_detected`, `survivor_updated`, `alert_created`
- Broadcast WebSocket tại `/ws/mat/stream`

### Đã thêm — Cơ Sở Hạ Tầng
- Trình cài đặt tương tác 7 bước có hướng dẫn với 8 hồ sơ phần cứng (`8583f3e`)
- Hướng dẫn build toàn diện cho Linux, macOS, Windows, Docker, ESP32 (`45f8a0d`)
- 12 Architecture Decision Records (ADR-001 đến ADR-012) (`337dd96`)

### Đã thêm — Giao Diện Người Dùng & Trực Quan Hóa
- Chế độ giao diện người dùng chỉ sensing với trực quan hóa Gaussian splat (`b7e0f07`)
- Mô hình cơ thể 3D Three.js (17 khớp, 16 chi) với các thành phần signal-viz
- Các tab: Dashboard, Hardware, Live Demo, Sensing, Architecture, Performance, Applications
- WebSocket client với kết nối lại tự động và exponential backoff

### Đã thêm — Crate Xử Lý Tín Hiệu Rust
- Rust port hoàn chỉnh của WiFi-DensePose với modular workspace (`6ed69a3`)
  - `wifi-densepose-signal` — Xử lý CSI, làm sạch pha, trích xuất đặc trưng
  - `wifi-densepose-core` — Các kiểu dữ liệu và cấu hình dùng chung
  - `wifi-densepose-nn` — Suy luận neural network (DensePose head, RCNN)
  - `wifi-densepose-hardware` — ESP32 aggregator, các giao diện phần cứng
  - `wifi-densepose-config` — Quản lý cấu hình
- Benchmark và validation tests toàn diện (`3ccb301`)

### Đã thêm — Pipeline Sensing Python
- `WindowsWifiCollector` — Thu thập RSSI qua `netsh wlan show networks`
- `RssiFeatureExtractor` — Phương sai, băng tần phổ (chuyển động 0,5-4 Hz, nhịp thở 0,1-0,5 Hz), điểm thay đổi
- `PresenceClassifier` — Phân loại 3 trạng thái dựa trên quy tắc (ABSENT / PRESENT_STILL / ACTIVE)
- Tính điểm đồng thuận cross-receiver để tăng cường confidence đa AP
- WebSocket sensing server (`ws_server.py`) broadcast JSON ở 2 Hz
- Gói proof CSI tất định để xác minh tái tạo (`v1/data/proof/`)
- Unit tests sensing thiết bị thông thường (`b391638`)

### Đã thay đổi
- Các adapter phần cứng Rust bây giờ trả về lỗi rõ ràng thay vì dữ liệu rỗng thầm lặng (`6e0e539`)

### Đã sửa
- Các bản sửa review cho pipeline huấn luyện end-to-end (`45f0304`)
- Các đường dẫn Dockerfile được cập nhật từ `src/` thành `v1/src/` (`7872987`)
- Hướng dẫn trình cài đặt hồ sơ IoT được cập nhật cho aggregator CLI (`f460097`)
- Tham chiếu `process.env` đã được xóa khỏi ES module trình duyệt (`e320bc9`)

### Hiệu Suất
- Tăng tốc trích xuất Doppler 5,7x qua FFT windowing được tối ưu hóa (`32c75c8`)
- Binary tĩnh 2,1 MB, không có dependencies Python cho Rust server

### Bảo Mật
- Sửa SQL injection trong lệnh status và migrations (`f9d125d`)
- Sửa các lỗ hổng XSS trong các thành phần giao diện người dùng (`5db55fd`)
- Sửa command injection trong statusline.cjs (`4cb01fd`)
- Sửa các lỗ hổng path traversal (`896c4fc`)
- Sửa kết nối WebSocket không an toàn — bắt buộc wss:// trên non-localhost (`ac094d4`)
- Sửa shell injection GitHub Actions (`ab2e7b4`)
- Sửa 10 lỗ hổng bổ sung, loại bỏ 12 trường hợp code dead (`7afdad0`)

---

## [1.1.0] - 2025-06-07

### Đã thêm
- Hệ thống WiFi-DensePose Python hoàn chỉnh với trích xuất dữ liệu CSI và giao diện router
- Các module xử lý CSI và làm sạch pha
- Xử lý hàng loạt cho dữ liệu CSI trong `CSIProcessor` và `PhaseSanitizer`
- Các dịch vụ hardware, pose, và stream cho WiFi-DensePose API
- CSS styles toàn diện cho các thành phần giao diện người dùng và hỗ trợ dark mode
- Tài liệu API và Triển Khai

### Đã sửa
- Liên kết badge cho PyPI và Docker trong README
- Đặc điểm kỹ thuật poolclass tạo async engine

---

## [1.0.0] - 2024-12-01

### Đã thêm
- Phiên bản đầu tiên của WiFi-DensePose
- Ước lượng tư thế người dựa trên WiFi thời gian thực sử dụng Channel State Information (CSI)
- Tích hợp neural network DensePose cho ánh xạ bề mặt cơ thể
- RESTful API với phạm vi endpoint toàn diện
- WebSocket streaming cho dữ liệu pose thời gian thực
- Theo dõi đa người với khả năng cấu hình (mặc định 10, lên đến 50+)
- Phát hiện té ngã và nhận dạng hoạt động
- Cấu hình domain: healthcare, fitness, smart home, security
- Giao diện CLI để quản lý server và cấu hình
- Tầng trừu tượng phần cứng cho nhiều chipset WiFi
- Pipeline làm sạch pha và xử lý tín hiệu
- Xác thực và giới hạn tốc độ
- Quản lý background task
- Hỗ trợ đa nền tảng (Linux, macOS, Windows)

### Tài Liệu
- Hướng dẫn người dùng và tham chiếu API
- Hướng dẫn triển khai và khắc phục sự cố
- Hướng dẫn thiết lập và hiệu chỉnh phần cứng
- Benchmark hiệu suất
- Hướng dẫn đóng góp

[Unreleased]: https://github.com/ruvnet/wifi-densepose/compare/v3.0.0...HEAD
[3.0.0]: https://github.com/ruvnet/wifi-densepose/compare/v2.0.0...v3.0.0
[2.0.0]: https://github.com/ruvnet/wifi-densepose/compare/v1.1.0...v2.0.0
[1.1.0]: https://github.com/ruvnet/wifi-densepose/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/ruvnet/wifi-densepose/releases/tag/v1.0.0
