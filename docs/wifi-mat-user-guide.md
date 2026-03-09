# Hướng Dẫn Sử Dụng WiFi-Mat

## Công Cụ Đánh Giá Thương Vong Hàng Loạt Cho Ứng Phó Thảm Họa

WiFi-Mat (Mass Assessment Tool) là một phần mở rộng mô-đun của WiFi-DensePose được thiết kế đặc biệt cho các hoạt động tìm kiếm và cứu nạn. Nó sử dụng WiFi Channel State Information (CSI) để phát hiện và định vị những người sống sót bị mắc kẹt trong đống đổ nát, mảnh vỡ và các công trình sụp đổ trong các trận động đất, sập nhà, tuyết lở và các tình huống thảm họa khác.

---

## Mục Lục

1. [Tổng Quan](#overview)
2. [Tính Năng Chính](#key-features)
3. [Cài Đặt](#installation)
4. [Bắt Đầu Nhanh](#quick-start)
5. [Kiến Trúc](#architecture)
6. [Cấu Hình](#configuration)
7. [Khả Năng Phát Hiện](#detection-capabilities)
8. [Hệ Thống Định Vị](#localization-system)
9. [Phân Loại Thương Vong](#triage-classification)
10. [Hệ Thống Cảnh Báo](#alert-system)
11. [Tài Liệu API](#api-reference)
12. [Thiết Lập Phần Cứng](#hardware-setup)
13. [Hướng Dẫn Triển Khai Thực Địa](#field-deployment-guide)
14. [Khắc Phục Sự Cố](#troubleshooting)
15. [Thực Hành Tốt Nhất](#best-practices)
16. [Lưu Ý An Toàn](#safety-considerations)

---

## Tổng Quan

### WiFi-Mat là gì?

WiFi-Mat tận dụng cùng công nghệ cảm biến dựa trên WiFi như WiFi-DensePose nhưng được tối ưu hóa cho các thách thức đặc thù của ứng phó thảm họa:

- **Phát hiện xuyên tường**: Phát hiện dấu hiệu sự sống qua đống đổ nát, mảnh vỡ và các công trình sụp đổ
- **Không xâm lấn**: Không cần làm xáo trộn các công trình không ổn định trong quá trình đánh giá ban đầu
- **Triển khai nhanh**: Mảng cảm biến di động có thể được thiết lập trong vài phút
- **Phân loại nhiều nạn nhân**: Tự động ưu tiên nỗ lực cứu nạn bằng giao thức START
- **Định vị 3D**: Ước tính vị trí người sống sót bao gồm độ sâu qua đống đổ nát

### Các Trường Hợp Sử Dụng

| Loại Thảm Họa | Phạm Vi Phát Hiện | Độ Sâu Điển Hình | Tỷ Lệ Thành Công |
|--------------|-----------------|---------------|--------------|
| Đống đổ nát động đất | Bán kính 15-30m | Đến 5m | 85-92% |
| Sập nhà | Bán kính 20-40m | Đến 8m | 80-88% |
| Tuyết lở | Bán kính 10-20m | Đến 3m tuyết | 75-85% |
| Sập hầm mỏ | Bán kính 15-25m | Đến 10m | 70-82% |
| Mảnh vỡ lũ lụt | Bán kính 10-15m | Đến 2m | 88-95% |

---

## Tính Năng Chính

### 1. Phát Hiện Dấu Hiệu Sinh Tồn
- **Phát hiện hô hấp**: 0.1-0.5 Hz (4-60 lần thở/phút)
- **Phát hiện nhịp tim**: 0.8-3.3 Hz (30-200 BPM) qua micro-Doppler
- **Phân loại chuyển động**: Chuyển động lớn, nhỏ, run và định kỳ

### 2. Định Vị Người Sống Sót
- **Vị trí 2D**: Độ chính xác ±0.5m với 3+ cảm biến
- **Ước tính độ sâu**: ±0.3m qua đống đổ nát đến 5m
- **Chấm điểm tin cậy**: Lượng hóa độ không chắc chắn theo thời gian thực

### 3. Phân Loại Thương Vong
- **Giao thức START**: Khẩn cấp/Trì hoãn/Nhẹ/Đã tử vong
- **Ưu tiên tự động**: Dựa trên dấu hiệu sinh tồn và khả năng tiếp cận
- **Cập nhật động**: Phân loại lại khi điều kiện thay đổi

### 4. Hệ Thống Cảnh Báo
- **Dựa trên mức độ ưu tiên**: Cảnh báo Nghiêm Trọng/Cao/Trung Bình/Thấp
- **Đa kênh**: Tích hợp âm thanh, hình ảnh, thông báo di động, radio
- **Leo thang**: Tự động leo thang cho người sống sót đang xấu đi

---

## Cài Đặt

### Yêu Cầu Tiên Quyết

```bash
# Rust toolchain (1.70+)
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Các phụ thuộc hệ thống cần thiết (Ubuntu/Debian)
sudo apt-get install -y build-essential pkg-config libssl-dev
```

### Xây Dựng Từ Mã Nguồn

```bash
# Clone repository
git clone https://github.com/ruvnet/wifi-densepose.git
cd wifi-densepose/rust-port/wifi-densepose-rs

# Xây dựng crate wifi-mat
cargo build --release --package wifi-densepose-mat

# Chạy kiểm tra
cargo test --package wifi-densepose-mat

# Xây dựng với tất cả tính năng
cargo build --release --package wifi-densepose-mat --all-features
```

### Cờ Tính Năng

```toml
# Cargo.toml features
[features]
default = ["std"]
std = []
serde = ["dep:serde"]
async = ["tokio"]
hardware = ["wifi-densepose-hardware"]
neural = ["wifi-densepose-nn"]
full = ["serde", "async", "hardware", "neural"]
```

---

## Bắt Đầu Nhanh

### Ví Dụ Cơ Bản

```rust
use wifi_densepose_mat::{
    DisasterResponse, DisasterConfig, DisasterType,
    ScanZone, ZoneBounds,
};

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    // Cấu hình cho ứng phó động đất
    let config = DisasterConfig::builder()
        .disaster_type(DisasterType::Earthquake)
        .sensitivity(0.85)
        .confidence_threshold(0.5)
        .max_depth(5.0)
        .continuous_monitoring(true)
        .build();

    // Khởi tạo hệ thống ứng phó
    let mut response = DisasterResponse::new(config);

    // Khởi tạo sự kiện thảm họa
    let location = geo::Point::new(-122.4194, 37.7749); // San Francisco
    response.initialize_event(location, "Building collapse - Market Street")?;

    // Xác định các vùng quét
    let zone_a = ScanZone::new(
        "North Wing - Ground Floor",
        ZoneBounds::rectangle(0.0, 0.0, 30.0, 20.0),
    );
    response.add_zone(zone_a)?;

    let zone_b = ScanZone::new(
        "South Wing - Basement",
        ZoneBounds::rectangle(30.0, 0.0, 60.0, 20.0),
    );
    response.add_zone(zone_b)?;

    // Bắt đầu quét
    println!("Starting survivor detection scan...");
    response.start_scanning().await?;

    // Lấy danh sách người sống sót được phát hiện
    let survivors = response.survivors();
    println!("Detected {} potential survivors", survivors.len());

    // Lấy người sống sót ưu tiên khẩn cấp
    let immediate = response.survivors_by_triage(TriageStatus::Immediate);
    println!("{} survivors require immediate rescue", immediate.len());

    Ok(())
}
```

### Ví Dụ Phát Hiện Tối Giản

```rust
use wifi_densepose_mat::detection::{
    BreathingDetector, BreathingDetectorConfig,
    DetectionPipeline, DetectionConfig,
};

fn detect_breathing(csi_amplitudes: &[f64], sample_rate: f64) {
    let config = BreathingDetectorConfig::default();
    let detector = BreathingDetector::new(config);

    if let Some(breathing) = detector.detect(csi_amplitudes, sample_rate) {
        println!("Breathing detected!");
        println!("  Rate: {:.1} BPM", breathing.rate_bpm);
        println!("  Pattern: {:?}", breathing.pattern_type);
        println!("  Confidence: {:.2}", breathing.confidence);
    } else {
        println!("No breathing detected");
    }
}
```

---

## Kiến Trúc

### Tổng Quan Hệ Thống

```
┌──────────────────────────────────────────────────────────────────┐
│                        WiFi-Mat System                           │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  │
│  │   Detection     │  │  Localization   │  │    Alerting     │  │
│  │    Context      │  │    Context      │  │    Context      │  │
│  │                 │  │                 │  │                 │  │
│  │ • Breathing     │  │ • Triangulation │  │ • Generator     │  │
│  │ • Heartbeat     │  │ • Depth Est.    │  │ • Dispatcher    │  │
│  │ • Movement      │  │ • Fusion        │  │ • Triage Svc    │  │
│  │ • Pipeline      │  │                 │  │                 │  │
│  └────────┬────────┘  └────────┬────────┘  └────────┬────────┘  │
│           │                    │                    │            │
│           └────────────────────┼────────────────────┘            │
│                                │                                 │
│                    ┌───────────▼───────────┐                     │
│                    │    Integration        │                     │
│                    │       Layer           │                     │
│                    │                       │                     │
│                    │ • SignalAdapter       │                     │
│                    │ • NeuralAdapter       │                     │
│                    │ • HardwareAdapter     │                     │
│                    └───────────┬───────────┘                     │
│                                │                                 │
└────────────────────────────────┼─────────────────────────────────┘
                                 │
              ┌──────────────────┼──────────────────┐
              │                  │                  │
    ┌─────────▼─────────┐ ┌─────▼─────┐ ┌─────────▼─────────┐
    │ wifi-densepose-   │ │ wifi-     │ │ wifi-densepose-   │
    │     signal        │ │ densepose │ │    hardware       │
    │                   │ │   -nn     │ │                   │
    └───────────────────┘ └───────────┘ └───────────────────┘
```

### Mô Hình Miền

```
┌─────────────────────────────────────────────────────────────┐
│                     DisasterEvent                           │
│                   (Aggregate Root)                          │
├─────────────────────────────────────────────────────────────┤
│ - id: DisasterEventId                                       │
│ - disaster_type: DisasterType                               │
│ - location: Point<f64>                                      │
│ - status: EventStatus                                       │
│ - zones: Vec<ScanZone>                                      │
│ - survivors: Vec<Survivor>                                  │
│ - created_at: DateTime<Utc>                                 │
│ - metadata: EventMetadata                                   │
└─────────────────────────────────────────────────────────────┘
         │                              │
         │ chứa                         │ chứa
         ▼                              ▼
┌─────────────────────┐      ┌─────────────────────────────┐
│     ScanZone        │      │         Survivor            │
│     (Entity)        │      │         (Entity)            │
├─────────────────────┤      ├─────────────────────────────┤
│ - id: ScanZoneId    │      │ - id: SurvivorId            │
│ - name: String      │      │ - vital_signs: VitalSigns   │
│ - bounds: ZoneBounds│      │ - location: Option<Coord3D> │
│ - sensors: Vec<...> │      │ - triage: TriageStatus      │
│ - parameters: ...   │      │ - alerts: Vec<Alert>        │
│ - status: ZoneStatus│      │ - metadata: SurvivorMeta    │
└─────────────────────┘      └─────────────────────────────┘
```

---

## Cấu Hình

### Tùy Chọn DisasterConfig

```rust
let config = DisasterConfig {
    // Loại thảm họa (ảnh hưởng đến thuật toán phát hiện)
    disaster_type: DisasterType::Earthquake,

    // Độ nhạy phát hiện (0.0-1.0)
    // Cao hơn = nhiều dương tính giả hơn, ít bỏ sót phát hiện hơn
    sensitivity: 0.8,

    // Độ tin cậy tối thiểu để báo cáo một lần phát hiện
    confidence_threshold: 0.5,

    // Độ sâu tối đa để thử phát hiện (mét)
    max_depth: 5.0,

    // Khoảng thời gian quét tính bằng mili giây
    scan_interval_ms: 500,

    // Tiếp tục quét liên tục
    continuous_monitoring: true,

    // Cấu hình cảnh báo
    alert_config: AlertConfig {
        enable_audio: true,
        enable_push: true,
        escalation_timeout_secs: 300,
        priority_threshold: Priority::Medium,
    },
};
```

### Các Loại Thảm Họa

| Loại | Tối Ưu Hóa | Phù Hợp Nhất |
|------|--------------|----------|
| `Earthquake` | Tăng cường phát hiện micro-chuyển động | Sập nhà |
| `BuildingCollapse` | Xuyên sâu, lọc nhiễu | SAR đô thị |
| `Avalanche` | Bù thân nhiệt lạnh, xuyên tuyết | Cứu hộ núi |
| `Flood` | Bù nhiễu nước | Cứu hộ lũ lụt |
| `MineCollapse` | Xuyên đá, phát hiện khí | Tai nạn mỏ |
| `Explosion` | Mô hình chấn thương nổ | Tai nạn công nghiệp |
| `Unknown` | Mặc định cân bằng | Sử dụng chung |

### ScanParameters

```rust
let params = ScanParameters {
    // Độ nhạy phát hiện cho vùng này
    sensitivity: 0.85,

    // Độ sâu quét tối đa (mét)
    max_depth: 5.0,

    // Mức độ phân giải
    resolution: ScanResolution::High,

    // Bật tính năng phát hiện hô hấp nâng cao
    enhanced_breathing: true,

    // Bật phát hiện nhịp tim (chậm hơn nhưng chính xác hơn)
    heartbeat_detection: true,
};

let zone = ScanZone::with_parameters("Zone A", bounds, params);
```

---

## Khả Năng Phát Hiện

### Phát Hiện Hô Hấp

WiFi-Mat phát hiện hô hấp thông qua các chuyển động định kỳ của thành ngực điều chế tín hiệu WiFi.

```rust
use wifi_densepose_mat::detection::{BreathingDetector, BreathingDetectorConfig};

let config = BreathingDetectorConfig {
    // Dải tần số hô hấp (Hz)
    min_frequency: 0.1,  // 6 BPM
    max_frequency: 0.5,  // 30 BPM

    // Cửa sổ phân tích
    window_seconds: 10.0,

    // Ngưỡng phát hiện
    confidence_threshold: 0.3,

    // Bật phân loại mô hình
    classify_patterns: true,
};

let detector = BreathingDetector::new(config);
let result = detector.detect(&amplitudes, sample_rate);
```

**Các Mô Hình Có Thể Phát Hiện:**
- Thở bình thường
- Thở nông/nhanh
- Thở sâu/chậm
- Thở không đều
- Thở hấp hối (nghiêm trọng)

### Phát Hiện Nhịp Tim

Sử dụng phân tích micro-Doppler để phát hiện các chuyển động cơ thể tinh tế từ nhịp tim.

```rust
use wifi_densepose_mat::detection::{HeartbeatDetector, HeartbeatDetectorConfig};

let config = HeartbeatDetectorConfig {
    // Dải nhịp tim (Hz)
    min_frequency: 0.8,  // 48 BPM
    max_frequency: 3.0,  // 180 BPM

    // Yêu cầu phát hiện hô hấp trước (giảm dương tính giả)
    require_breathing: true,

    // Ngưỡng cao hơn do tín hiệu tinh tế
    confidence_threshold: 0.4,
};

let detector = HeartbeatDetector::new(config);
let result = detector.detect(&phases, sample_rate, Some(breathing_rate));
```

### Phân Loại Chuyển Động

```rust
use wifi_densepose_mat::detection::{MovementClassifier, MovementClassifierConfig};

let classifier = MovementClassifier::new(MovementClassifierConfig::default());
let movement = classifier.classify(&amplitudes, sample_rate);

match movement.movement_type {
    MovementType::Gross => println!("Large movement - likely conscious"),
    MovementType::Fine => println!("Small movement - possible injury"),
    MovementType::Tremor => println!("Tremor detected - possible shock"),
    MovementType::Periodic => println!("Periodic movement - likely breathing only"),
    MovementType::None => println!("No movement detected"),
}
```

---

## Hệ Thống Định Vị

### Tam Giác Đạc

Sử dụng Time-of-Flight và cường độ tín hiệu từ nhiều cảm biến.

```rust
use wifi_densepose_mat::localization::{Triangulator, TriangulationConfig};

let config = TriangulationConfig {
    // Số cảm biến tối thiểu để định vị 2D
    min_sensors: 3,

    // Sử dụng RSSI ngoài CSI
    use_rssi: true,

    // Số vòng lặp tối đa để tối ưu hóa
    max_iterations: 100,

    // Ngưỡng hội tụ
    convergence_threshold: 0.01,
};

let triangulator = Triangulator::new(config);

// Vị trí cảm biến
let sensors = vec![
    SensorPosition { x: 0.0, y: 0.0, z: 1.5, .. },
    SensorPosition { x: 10.0, y: 0.0, z: 1.5, .. },
    SensorPosition { x: 5.0, y: 10.0, z: 1.5, .. },
];

// Đo lường RSSI từ mỗi cảm biến
let measurements = vec![-45.0, -52.0, -48.0];

let position = triangulator.estimate(&sensors, &measurements)?;
println!("Estimated position: ({:.2}, {:.2})", position.x, position.y);
println!("Uncertainty: ±{:.2}m", position.uncertainty);
```

### Ước Tính Độ Sâu

Ước tính độ sâu qua đống đổ nát sử dụng phân tích suy giảm tín hiệu.

```rust
use wifi_densepose_mat::localization::{DepthEstimator, DepthEstimatorConfig};

let config = DepthEstimatorConfig {
    // Hệ số suy giảm vật liệu
    material_model: MaterialModel::MixedDebris,

    // Cường độ tín hiệu tham chiếu (đường ngắm rõ)
    reference_rssi: -30.0,

    // Độ sâu tối đa có thể phát hiện
    max_depth: 8.0,
};

let estimator = DepthEstimator::new(config);
let depth = estimator.estimate(measured_rssi, expected_rssi)?;

println!("Estimated depth: {:.2}m", depth.meters);
println!("Confidence: {:.2}", depth.confidence);
println!("Material: {:?}", depth.estimated_material);
```

### Hợp Nhất Vị Trí

Kết hợp nhiều phương pháp ước tính sử dụng lọc Kalman.

```rust
use wifi_densepose_mat::localization::{PositionFuser, LocalizationService};

let service = LocalizationService::new();

// Ước tính vị trí 3D đầy đủ
let position = service.estimate_position(&vital_signs, &zone)?;

println!("3D Position:");
println!("  X: {:.2}m (±{:.2})", position.x, position.uncertainty.x);
println!("  Y: {:.2}m (±{:.2})", position.y, position.uncertainty.y);
println!("  Z: {:.2}m (±{:.2})", position.z, position.uncertainty.z);
println!("  Total confidence: {:.2}", position.confidence);
```

---

## Phân Loại Thương Vong

### Giao Thức START

WiFi-Mat triển khai giao thức Simple Triage and Rapid Treatment (START):

| Trạng Thái | Tiêu Chí | Hành Động |
|--------|----------|--------|
| **Khẩn Cấp (Đỏ)** | Thở 10-29/phút, không có mạch quay, làm theo lệnh | Cứu trước |
| **Trì Hoãn (Vàng)** | Thở bình thường, có mạch, thương tích không đe dọa tính mạng | Cứu sau |
| **Nhẹ (Xanh)** | Người bị thương có thể đi lại, thương tích nhẹ | Có thể chờ |
| **Đã Tử Vong (Đen)** | Không thở sau khi thông đường thở | Không cứu |

### Phân Loại Tự Động

```rust
use wifi_densepose_mat::domain::triage::{TriageCalculator, TriageStatus};

let calculator = TriageCalculator::new();

// Tính toán phân loại dựa trên dấu hiệu sinh tồn
let vital_signs = VitalSignsReading {
    breathing: Some(BreathingPattern {
        rate_bpm: 24.0,
        pattern_type: BreathingType::Shallow,
        ..
    }),
    heartbeat: Some(HeartbeatSignature {
        rate_bpm: 110.0,
        ..
    }),
    movement: MovementProfile {
        movement_type: MovementType::Fine,
        ..
    },
    ..
};

let triage = calculator.calculate(&vital_signs);

match triage {
    TriageStatus::Immediate => println!("⚠️ IMMEDIATE - Rescue NOW"),
    TriageStatus::Delayed => println!("🟡 DELAYED - Stable for now"),
    TriageStatus::Minor => println!("🟢 MINOR - Walking wounded"),
    TriageStatus::Deceased => println!("⬛ DECEASED - No vital signs"),
    TriageStatus::Unknown => println!("❓ UNKNOWN - Insufficient data"),
}
```

### Các Yếu Tố Phân Loại

```rust
// Truy cập lý giải phân loại chi tiết
let factors = calculator.calculate_with_factors(&vital_signs);

println!("Triage: {:?}", factors.status);
println!("Contributing factors:");
for factor in &factors.contributing_factors {
    println!("  - {} (weight: {:.2})", factor.description, factor.weight);
}
println!("Confidence: {:.2}", factors.confidence);
```

---

## Hệ Thống Cảnh Báo

### Tạo Cảnh Báo

```rust
use wifi_densepose_mat::alerting::{AlertGenerator, AlertConfig};

let config = AlertConfig {
    // Mức ưu tiên tối thiểu để tạo cảnh báo
    priority_threshold: Priority::Medium,

    // Cài đặt leo thang
    escalation_enabled: true,
    escalation_timeout: Duration::from_secs(300),

    // Kênh thông báo
    channels: vec![
        AlertChannel::Audio,
        AlertChannel::Visual,
        AlertChannel::Push,
        AlertChannel::Radio,
    ],
};

let generator = AlertGenerator::new(config);

// Tạo cảnh báo cho một người sống sót
let alert = generator.generate(&survivor)?;

println!("Alert generated:");
println!("  ID: {}", alert.id());
println!("  Priority: {:?}", alert.priority());
println!("  Message: {}", alert.message());
```

### Mức Độ Ưu Tiên Cảnh Báo

| Mức Ưu Tiên | Tiêu Chí | Thời Gian Phản Hồi |
|----------|----------|---------------|
| **Nghiêm Trọng** | Phân loại khẩn cấp, đang xấu đi | < 5 phút |
| **Cao** | Phân loại khẩn cấp, ổn định | < 15 phút |
| **Trung Bình** | Phân loại trì hoãn | < 1 giờ |
| **Thấp** | Phân loại nhẹ | Khi có thể |

### Gửi Cảnh Báo

```rust
use wifi_densepose_mat::alerting::AlertDispatcher;

let dispatcher = AlertDispatcher::new(config);

// Gửi đến tất cả các kênh đã cấu hình
dispatcher.dispatch(alert).await?;

// Gửi đến kênh cụ thể
dispatcher.dispatch_to(alert, AlertChannel::Radio).await?;

// Gửi hàng loạt cho nhiều người sống sót
dispatcher.dispatch_batch(&alerts).await?;
```

---

## Tài Liệu API

### Các Kiểu Dữ Liệu Cốt Lõi

```rust
// Điểm vào chính
pub struct DisasterResponse {
    pub fn new(config: DisasterConfig) -> Self;
    pub fn initialize_event(&mut self, location: Point, desc: &str) -> Result<&DisasterEvent>;
    pub fn add_zone(&mut self, zone: ScanZone) -> Result<()>;
    pub async fn start_scanning(&mut self) -> Result<()>;
    pub fn stop_scanning(&self);
    pub fn survivors(&self) -> Vec<&Survivor>;
    pub fn survivors_by_triage(&self, status: TriageStatus) -> Vec<&Survivor>;
}

// Cấu hình
pub struct DisasterConfig {
    pub disaster_type: DisasterType,
    pub sensitivity: f64,
    pub confidence_threshold: f64,
    pub max_depth: f64,
    pub scan_interval_ms: u64,
    pub continuous_monitoring: bool,
    pub alert_config: AlertConfig,
}

// Các thực thể miền
pub struct Survivor { /* ... */ }
pub struct ScanZone { /* ... */ }
pub struct DisasterEvent { /* ... */ }
pub struct Alert { /* ... */ }

// Các đối tượng giá trị
pub struct VitalSignsReading { /* ... */ }
pub struct BreathingPattern { /* ... */ }
pub struct HeartbeatSignature { /* ... */ }
pub struct Coordinates3D { /* ... */ }
```

### API Phát Hiện

```rust
// Hô hấp
pub struct BreathingDetector {
    pub fn new(config: BreathingDetectorConfig) -> Self;
    pub fn detect(&self, amplitudes: &[f64], sample_rate: f64) -> Option<BreathingPattern>;
}

// Nhịp tim
pub struct HeartbeatDetector {
    pub fn new(config: HeartbeatDetectorConfig) -> Self;
    pub fn detect(&self, phases: &[f64], sample_rate: f64, breathing_rate: Option<f64>) -> Option<HeartbeatSignature>;
}

// Chuyển động
pub struct MovementClassifier {
    pub fn new(config: MovementClassifierConfig) -> Self;
    pub fn classify(&self, amplitudes: &[f64], sample_rate: f64) -> MovementProfile;
}

// Pipeline
pub struct DetectionPipeline {
    pub fn new(config: DetectionConfig) -> Self;
    pub async fn process_zone(&self, zone: &ScanZone) -> Result<Option<VitalSignsReading>>;
    pub fn add_data(&self, amplitudes: &[f64], phases: &[f64]);
}
```

### API Định Vị

```rust
pub struct Triangulator {
    pub fn new(config: TriangulationConfig) -> Self;
    pub fn estimate(&self, sensors: &[SensorPosition], measurements: &[f64]) -> Result<Position2D>;
}

pub struct DepthEstimator {
    pub fn new(config: DepthEstimatorConfig) -> Self;
    pub fn estimate(&self, measured: f64, expected: f64) -> Result<DepthEstimate>;
}

pub struct LocalizationService {
    pub fn new() -> Self;
    pub fn estimate_position(&self, vital_signs: &VitalSignsReading, zone: &ScanZone) -> Result<Coordinates3D>;
}
```

---

## Thiết Lập Phần Cứng

### Yêu Cầu Cảm Biến

| Thành Phần | Tối Thiểu | Khuyến Nghị |
|-----------|---------|-------------|
| Bộ Thu Phát WiFi | 3 | 6-8 |
| Tốc Độ Lấy Mẫu | 100 Hz | 1000 Hz |
| Băng Tần | 2.4 GHz | 5 GHz |
| Loại Ăng Ten | Đa hướng | Định hướng |
| Nguồn Điện | Pin | AC + Pin |

### Mảng Cảm Biến Di Động

```
    [Sensor 1]              [Sensor 2]
         \                    /
          \    SCAN ZONE     /
           \                /
            \              /
             [Sensor 3]---[Sensor 4]
                  |
              [Controller]
                  |
              [Display]
```

### Bố Trí Cảm Biến

```rust
// Ví dụ cấu hình cảm biến cho vùng 30x20m
let sensors = vec![
    SensorPosition {
        id: "S1".into(),
        x: 0.0, y: 0.0, z: 2.0,
        sensor_type: SensorType::Transceiver,
        is_operational: true,
    },
    SensorPosition {
        id: "S2".into(),
        x: 30.0, y: 0.0, z: 2.0,
        sensor_type: SensorType::Transceiver,
        is_operational: true,
    },
    SensorPosition {
        id: "S3".into(),
        x: 0.0, y: 20.0, z: 2.0,
        sensor_type: SensorType::Transceiver,
        is_operational: true,
    },
    SensorPosition {
        id: "S4".into(),
        x: 30.0, y: 20.0, z: 2.0,
        sensor_type: SensorType::Transceiver,
        is_operational: true,
    },
];
```

---

## Hướng Dẫn Triển Khai Thực Địa

### Danh Sách Kiểm Tra Trước Triển Khai

- [ ] Xác minh tất cả cảm biến đã sạc (>80%)
- [ ] Kiểm tra kết nối cảm biến
- [ ] Hiệu chỉnh theo điều kiện địa phương
- [ ] Thiết lập liên lạc với trung tâm chỉ huy
- [ ] Thông báo cho các đội cứu hộ về khả năng của hệ thống

### Các Bước Triển Khai

1. **Đánh Giá Hiện Trường** (5 phút)
   - Xác định vị trí đặt cảm biến an toàn
   - Ghi nhận các nguy hiểm về kết cấu
   - Ước tính thành phần đống đổ nát

2. **Triển Khai Cảm Biến** (10 phút)
   - Đặt cảm biến xung quanh chu vi khu vực tìm kiếm
   - Đảm bảo tối thiểu 3 cảm biến có đường ngắm tới nhau
   - Kết nối với bộ điều khiển

3. **Khởi Tạo Hệ Thống** (2 phút)
   ```rust
   let mut response = DisasterResponse::new(config);
   response.initialize_event(location, description)?;

   for zone in zones {
       response.add_zone(zone)?;
   }
   ```

4. **Hiệu Chỉnh** (5 phút)
   - Chạy hiệu chỉnh nhiễu nền
   - Điều chỉnh độ nhạy dựa trên môi trường

5. **Bắt Đầu Quét** (liên tục)
   ```rust
   response.start_scanning().await?;
   ```

### Diễn Giải Kết Quả

```
┌─────────────────────────────────────────────────────┐
│                  SCAN RESULTS                       │
├─────────────────────────────────────────────────────┤
│  Zone: North Wing - Ground Floor                    │
│  Status: ACTIVE | Scans: 127 | Duration: 10:34     │
├─────────────────────────────────────────────────────┤
│  DETECTIONS:                                        │
│                                                     │
│  [IMMEDIATE] Survivor #1                           │
│    Position: (12.3, 8.7) ±0.5m                     │
│    Depth: 2.1m ±0.3m                               │
│    Breathing: 24 BPM (shallow)                     │
│    Movement: Fine motor                            │
│    Confidence: 87%                                 │
│                                                     │
│  [DELAYED] Survivor #2                             │
│    Position: (22.1, 15.2) ±0.8m                    │
│    Depth: 1.5m ±0.2m                               │
│    Breathing: 16 BPM (normal)                      │
│    Movement: Periodic only                         │
│    Confidence: 92%                                 │
│                                                     │
│  [MINOR] Survivor #3                               │
│    Position: (5.2, 3.1) ±0.3m                      │
│    Depth: 0.3m ±0.1m                               │
│    Breathing: 18 BPM (normal)                      │
│    Movement: Gross motor (likely mobile)           │
│    Confidence: 95%                                 │
└─────────────────────────────────────────────────────┘
```

---

## Khắc Phục Sự Cố

### Các Vấn Đề Thường Gặp

| Vấn Đề | Nguyên Nhân Có Thể | Giải Pháp |
|-------|---------------|----------|
| Không phát hiện | Độ nhạy quá thấp | Tăng `sensitivity` lên 0.9+ |
| Quá nhiều dương tính giả | Độ nhạy quá cao | Giảm `sensitivity` xuống 0.6-0.7 |
| Định vị kém | Không đủ cảm biến | Thêm cảm biến (tối thiểu 3) |
| Phát hiện gián đoạn | Nhiễu tín hiệu | Kiểm tra nguồn điện từ |
| Ước tính độ sâu thất bại | Vật liệu dày đặc | Điều chỉnh `material_model` |

### Lệnh Chẩn Đoán

```rust
// Kiểm tra sức khỏe hệ thống
let health = response.hardware_health();
println!("Sensors: {}/{} operational", health.connected, health.total);

// Xem thống kê phát hiện
let stats = response.detection_stats();
println!("Detection rate: {:.1}%", stats.detection_rate * 100.0);
println!("False positive rate: {:.1}%", stats.false_positive_rate * 100.0);

// Xuất dữ liệu chẩn đoán
response.export_diagnostics("/path/to/diagnostics.json")?;
```

---

## Thực Hành Tốt Nhất

### Tối Ưu Hóa Phát Hiện

1. **Bắt đầu với độ nhạy cao**, giảm nếu có quá nhiều dương tính giả
2. **Bật phát hiện nhịp tim** chỉ khi đã xác nhận hô hấp
3. **Sử dụng loại thảm họa phù hợp** để có thuật toán tối ưu
4. **Tăng thời gian quét** cho các tín hiệu yếu (cửa sổ lên đến 30 giây)

### Tối Ưu Hóa Định Vị

1. **Sử dụng 4+ cảm biến** để định vị 2D đáng tin cậy
2. **Phân tán cảm biến** để bao phủ toàn bộ khu vực tìm kiếm
3. **Gắn ở độ cao nhất quán** (khuyến nghị 1.5-2.0m)
4. **Tính đến lỗi cảm biến** với dự phòng

### Mẹo Vận Hành

1. **Quét theo giai đoạn**: Quét nhanh trước, sau đó quét chi tiết tập trung
2. **Đánh dấu các kết quả dương tính đã xác nhận**: Giảm cảnh báo trùng lặp
3. **Cập nhật vùng động**: Xóa các khu vực đã xử lý
4. **Truyền đạt mức độ tin cậy**: Không phải tất cả các phát hiện đều chắc chắn

---

## Lưu Ý An Toàn

### Hạn Chế

- **Không đáng tin cậy 100%**: Luôn xác minh bằng các phương pháp thứ cấp
- **Yếu tố môi trường**: Kim loại, nước, bê tông dày làm giảm hiệu quả
- **Chỉ phát hiện chuyển động sống**: Không thể phát hiện người bất tỉnh/đã tử vong nếu không thở
- **Giới hạn độ sâu**: Độ chính xác giảm vượt quá độ sâu 5m

### Tích Hợp Với Các Phương Pháp Khác

WiFi-Mat nên được sử dụng kết hợp với:
- Phát hiện âm thanh (thiết bị nghe)
- Đội tìm kiếm chó nghiệp vụ
- Hình ảnh nhiệt
- Thăm dò vật lý

### Rủi Ro Âm Tính Giả

**Kết quả âm tính không đảm bảo vắng mặt người sống sót**. Luôn:
- Quét lại sau khi dọn đổ nát
- Sử dụng nhiều phương pháp quét
- Tiếp tục các thủ tục tìm kiếm thủ công

---

## Hỗ Trợ

- **Tài Liệu**: [ADR-001](/docs/adr/ADR-001-wifi-mat-disaster-detection.md)
- **Mô Hình Miền**: [Đặc Tả DDD](/docs/ddd/wifi-mat-domain-model.md)
- **Vấn Đề**: [GitHub Issues](https://github.com/ruvnet/wifi-densepose/issues)
- **Tài Liệu API**: Chạy `cargo doc --package wifi-densepose-mat --open`

---

*WiFi-Mat được thiết kế để hỗ trợ các hoạt động tìm kiếm và cứu nạn. Đây là công cụ bổ trợ, không thay thế, nhân viên cứu hộ được đào tạo và các giao thức SAR đã được thiết lập.*
