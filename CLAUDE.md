# Cấu Hình Claude Code — WiFi-DensePose + Claude Flow V3

## Dự Án: wifi-densepose

Ước lượng tư thế người dựa trên WiFi sử dụng Channel State Information (CSI).
Hai codebase song song: Python v1 (`v1/`) và Rust port (`rust-port/wifi-densepose-rs/`).
### Các Rust Crates Chính
| Crate | Mô tả |
|-------|-------------|
| `wifi-densepose-core` | Các kiểu dữ liệu cốt lõi, traits, kiểu lỗi, CSI frame primitives |
| `wifi-densepose-signal` | Xử lý tín hiệu SOTA + RuvSense multistatic sensing (14 modules) |
| `wifi-densepose-nn` | Suy luận neural network (ONNX, PyTorch, Candle backends) |
| `wifi-densepose-train` | Pipeline huấn luyện với tích hợp ruvector + ruview_metrics |
| `wifi-densepose-mat` | Công Cụ Đánh Giá Thương Vong Hàng Loạt — phát hiện người sống sót sau thảm họa |
| `wifi-densepose-hardware` | ESP32 aggregator, giao thức TDM, firmware channel hopping |
| `wifi-densepose-ruvector` | Tích hợp RuVector v2.0.4 + cross-viewpoint fusion (5 modules) |
| `wifi-densepose-api` | REST API (Axum) |
| `wifi-densepose-db` | Tầng cơ sở dữ liệu (Postgres, SQLite, Redis) |
| `wifi-densepose-config` | Quản lý cấu hình |
| `wifi-densepose-wasm` | WebAssembly bindings cho triển khai trên trình duyệt |
| `wifi-densepose-cli` | Công cụ CLI (`wifi-densepose` binary) |
| `wifi-densepose-sensing-server` | Axum server nhẹ cho giao diện WiFi sensing |
| `wifi-densepose-wifiscan` | Quét WiFi Multi-BSSID (ADR-022) |
| `wifi-densepose-vitals` | Trích xuất vital signs cấp độ CSI từ ESP32 (ADR-021) |

### Các Module RuvSense (`signal/src/ruvsense/`)
| Module | Mục đích |
|--------|-------|
| `multiband.rs` | Hợp nhất CSI frame đa băng tần, coherence đa kênh |
| `phase_align.rs` | Ước lượng lặp LO phase offset, circular mean |
| `multistatic.rs` | Hợp nhất có trọng số attention, geometric diversity |
| `coherence.rs` | Tính điểm Z-score coherence, DriftProfile |
| `coherence_gate.rs` | Quyết định cổng Accept/PredictOnly/Reject/Recalibrate |
| `pose_tracker.rs` | Bộ theo dõi Kalman 17 keypoint với AETHER re-ID embeddings |
| `field_model.rs` | Cấu trúc eigenstructure phòng SVD, trích xuất nhiễu loạn |
| `tomography.rs` | RF tomography, bộ giải ISTA L1, voxel grid |
| `longitudinal.rs` | Thống kê Welford, phát hiện drift sinh cơ học |
| `intention.rs` | Tín hiệu dẫn trước chuyển động (200-500ms) |
| `cross_room.rs` | Nhận dạng môi trường, đồ thị chuyển đổi |
| `gesture.rs` | Bộ phân loại cử chỉ DTW template matching |
| `adversarial.rs` | Phát hiện tín hiệu bất khả thi về vật lý, kiểm tra tính nhất quán đa liên kết |

### Cross-Viewpoint Fusion (`ruvector/src/viewpoint/`)
| Module | Mục đích |
|--------|-------|
| `attention.rs` | CrossViewpointAttention, GeometricBias, softmax với G_bias |
| `geometry.rs` | GeometricDiversityIndex, giới hạn Cramer-Rao, Fisher Information |
| `coherence.rs` | Coherence phasor pha, hysteresis gate |
| `fusion.rs` | MultistaticArray aggregate root, domain events |

### Tích Hợp RuVector v2.0.4 (ADR-016 hoàn tất, ADR-017 đề xuất)
Tất cả 5 ruvector crates được tích hợp trong workspace:
- `ruvector-mincut` → `metrics.rs` (DynamicPersonMatcher) + `subcarrier_selection.rs`
- `ruvector-attn-mincut` → `model.rs` (apply_antenna_attention) + `spectrogram.rs`
- `ruvector-temporal-tensor` → `dataset.rs` (CompressedCsiBuffer) + `breathing.rs`
- `ruvector-solver` → `subcarrier.rs` (nội suy thưa 114→56) + `triangulation.rs`
- `ruvector-attention` → `model.rs` (apply_spatial_attention) + `bvp.rs`

### Các Quyết Định Kiến Trúc
43 ADRs trong `docs/adr/` (ADR-001 đến ADR-043). Các ADR quan trọng:
- ADR-014: Xử lý tín hiệu SOTA (Đã chấp nhận)
- ADR-015: Bộ dữ liệu huấn luyện MM-Fi + Wi-Pose (Đã chấp nhận)
- ADR-016: Tích hợp pipeline huấn luyện RuVector (Đã chấp nhận — hoàn tất)
- ADR-017: Tích hợp tín hiệu RuVector + MAT (Đề xuất — mục tiêu tiếp theo)
- ADR-024: Contrastive CSI embedding / AETHER (Đã chấp nhận)
- ADR-027: Tổng quát hóa domain xuyên môi trường / MERIDIAN (Đã chấp nhận)
- ADR-028: Kiểm toán khả năng ESP32 + xác minh witness (Đã chấp nhận)
- ADR-029: Chế độ multistatic sensing RuvSense (Đề xuất)
- ADR-030: Mô hình field liên tục RuvSense (Đề xuất)
- ADR-031: Chế độ RF sensing-first RuView (Đề xuất)
- ADR-032: Tăng cường bảo mật multistatic mesh (Đề xuất)

### Lệnh Build & Test (repo này)
```bash
# Rust — kiểm thử toàn workspace (1,031+ tests, ~2 phút)
cd rust-port/wifi-densepose-rs
cargo test --workspace --no-default-features

# Rust — kiểm tra một crate (không cần GPU)
cargo check -p wifi-densepose-train --no-default-features

# Rust — phát hành crates (theo thứ tự dependency)
cargo publish -p wifi-densepose-core --no-default-features
cargo publish -p wifi-densepose-signal --no-default-features
# ... xem thứ tự phát hành crate bên dưới

# Python — xác minh proof tất định (SHA-256)
python v1/data/proof/verify.py

# Python — bộ kiểm thử
cd v1 && python -m pytest tests/ -x -q
```

### Thứ Tự Phát Hành Crate
Các crate phải được phát hành theo thứ tự dependency:
1. `wifi-densepose-core` (không có deps nội bộ)
2. `wifi-densepose-vitals` (không có deps nội bộ)
3. `wifi-densepose-wifiscan` (không có deps nội bộ)
4. `wifi-densepose-hardware` (không có deps nội bộ)
5. `wifi-densepose-config` (không có deps nội bộ)
6. `wifi-densepose-db` (không có deps nội bộ)
7. `wifi-densepose-signal` (phụ thuộc vào core)
8. `wifi-densepose-nn` (không có deps nội bộ, chỉ workspace)
9. `wifi-densepose-ruvector` (không có deps nội bộ, chỉ workspace)
10. `wifi-densepose-train` (phụ thuộc vào signal, nn)
11. `wifi-densepose-mat` (phụ thuộc vào core, signal, nn)
12. `wifi-densepose-api` (không có deps nội bộ)
13. `wifi-densepose-wasm` (phụ thuộc vào mat)
14. `wifi-densepose-sensing-server` (phụ thuộc vào wifiscan)
15. `wifi-densepose-cli` (phụ thuộc vào mat)

### Xác Thực & Kiểm Chứng Witness (ADR-028)

**Sau bất kỳ thay đổi code quan trọng nào, hãy chạy xác thực đầy đủ:**

```bash
# 1. Rust tests — phải đạt 1,031+ passed, 0 failed
cd rust-port/wifi-densepose-rs
cargo test --workspace --no-default-features

# 2. Python proof — phải in VERDICT: PASS
cd ../..
python v1/data/proof/verify.py

# 3. Tạo witness bundle (bao gồm cả hai ở trên + firmware hashes)
bash scripts/generate-witness-bundle.sh

# 4. Tự xác minh bundle — phải đạt 7/7 PASS
cd dist/witness-bundle-ADR028-*/
bash VERIFY.sh
```

**Nếu Python proof hash thay đổi** (ví dụ: cập nhật phiên bản numpy/scipy):
```bash
# Tạo lại hash dự kiến, sau đó xác minh nó đạt
python v1/data/proof/verify.py --generate-hash
python v1/data/proof/verify.py
```

**Nội dung witness bundle** (`dist/witness-bundle-ADR028-<sha>.tar.gz`):
- `WITNESS-LOG-028.md` — Ma trận attestation 33 hàng với bằng chứng cho mỗi khả năng
- `ADR-028-esp32-capability-audit.md` — Kết quả kiểm toán đầy đủ
- `proof/verify.py` + `expected_features.sha256` — Bằng chứng pipeline tất định
- `test-results/rust-workspace-tests.log` — Kết quả đầy đủ cargo test
- `firmware-manifest/source-hashes.txt` — SHA-256 của tất cả 7 file firmware ESP32
- `crate-manifest/versions.txt` — Tất cả 15 crates với phiên bản
- `VERIFY.sh` — Tự xác minh bằng một lệnh dành cho người nhận

**Các artifact proof chính:**
- `v1/data/proof/verify.py` — Trust Switch: đưa tín hiệu tham chiếu qua pipeline sản xuất, hash đầu ra
- `v1/data/proof/expected_features.sha256` — Hash dự kiến đã xuất bản
- `v1/data/proof/sample_csi_data.json` — 1.000 CSI frames tổng hợp (seed=42)
- `docs/WITNESS-LOG-028.md` — Quy trình xác minh tái tạo 11 bước
- `docs/adr/ADR-028-esp32-capability-audit.md` — Hồ sơ kiểm toán đầy đủ

### Nhánh
Nhánh mặc định: `main`
Nhánh tính năng đang hoạt động: `ruvsense-full-implementation` (PR #77)

---

## Quy Tắc Hành Vi (Luôn Áp Dụng)

- Làm đúng những gì được yêu cầu; không hơn, không kém
- KHÔNG BAO GIỜ tạo file trừ khi thực sự cần thiết để đạt mục tiêu
- LUÔN ưu tiên chỉnh sửa file hiện có thay vì tạo file mới
- KHÔNG BAO GIỜ chủ động tạo file tài liệu (*.md) hay README trừ khi được yêu cầu rõ ràng
- KHÔNG BAO GIỜ lưu file làm việc, text/mds, hay tests vào thư mục root
- Không liên tục kiểm tra trạng thái sau khi khởi động swarm — chờ kết quả
- LUÔN đọc file trước khi chỉnh sửa
- KHÔNG BAO GIỜ commit secrets, credentials, hay .env files

## Tổ Chức File

- KHÔNG BAO GIỜ lưu vào thư mục root — sử dụng các thư mục bên dưới
- `docs/adr/` — Architecture Decision Records (43 ADRs)
- `docs/ddd/` — Các mô hình Domain-Driven Design
- `rust-port/wifi-densepose-rs/crates/` — Rust workspace crates (15 crates)
- `rust-port/wifi-densepose-rs/crates/wifi-densepose-signal/src/ruvsense/` — Các module multistatic RuvSense (14 files)
- `rust-port/wifi-densepose-rs/crates/wifi-densepose-ruvector/src/viewpoint/` — Cross-viewpoint fusion (5 files)
- `rust-port/wifi-densepose-rs/crates/wifi-densepose-hardware/src/esp32/` — Giao thức TDM ESP32
- `firmware/esp32-csi-node/main/` — Firmware C ESP32 (channel hopping, NVS config, TDM)
- `v1/src/` — Mã nguồn Python (core, hardware, services, api)
- `v1/data/proof/` — Các gói proof CSI tất định
- `.claude-flow/` — Trạng thái điều phối Claude Flow (được commit để chia sẻ nhóm)
- `.claude/` — Cài đặt Claude Code, agents, memory (được commit để chia sẻ nhóm)

## Kiến Trúc Dự Án

- Tuân theo Domain-Driven Design với các bounded contexts
- Giữ file dưới 500 dòng
- Sử dụng typed interfaces cho tất cả public APIs
- Ưu tiên TDD London School (mock-first) cho code mới
- Sử dụng event sourcing cho các thay đổi trạng thái
- Đảm bảo xác thực đầu vào tại các ranh giới hệ thống

### Cấu Hình Dự Án

- **Topology**: hierarchical-mesh
- **Max Agents**: 15
- **Memory**: hybrid
- **HNSW**: Đã bật
- **Neural**: Đã bật

## Danh Sách Kiểm Tra Trước Khi Merge

Trước khi merge bất kỳ PR nào, xác minh từng mục có áp dụng và đã được giải quyết:

1. **Rust tests đạt** — `cargo test --workspace --no-default-features` (1,031+ passed, 0 failed)
2. **Python proof đạt** — `python v1/data/proof/verify.py` (VERDICT: PASS)
3. **README.md** — Cập nhật bảng platform, mô tả crate, bảng hardware, tóm tắt tính năng nếu phạm vi thay đổi
4. **CLAUDE.md** — Cập nhật bảng crate, danh sách ADR, bảng module, phiên bản nếu phạm vi thay đổi
5. **CHANGELOG.md** — Thêm mục dưới `[Unreleased]` với những gì đã thêm/sửa/thay đổi
6. **Hướng dẫn người dùng** (`docs/user-guide.md`) — Cập nhật nếu thêm nguồn dữ liệu mới, cờ CLI, hay bước thiết lập
7. **Chỉ mục ADR** — Cập nhật số lượng ADR trong bảng tài liệu README nếu tạo ADR mới
8. **Witness bundle** — Tạo lại nếu tests hay proof hash thay đổi: `bash scripts/generate-witness-bundle.sh`
9. **Docker Hub image** — Chỉ build lại nếu Dockerfile, dependencies, hay hành vi runtime thay đổi
10. **Phát hành Crate** — Chỉ cần nếu crate được phát hành lên crates.io và public API thay đổi
11. **`.gitignore`** — Thêm bất kỳ build artifacts hay binaries mới
12. **Kiểm toán bảo mật** — Chạy review bảo mật cho các module mới liên quan đến ranh giới hardware/network

## Build & Test

```bash
# Build
npm run build

# Test
npm test

# Lint
npm run lint
```

- LUÔN chạy tests sau khi thay đổi code
- LUÔN xác minh build thành công trước khi commit

## Quy Tắc Bảo Mật

- KHÔNG BAO GIỜ hardcode API keys, secrets, hay credentials trong file mã nguồn
- KHÔNG BAO GIỜ commit .env files hay bất kỳ file nào chứa secrets
- Luôn xác thực đầu vào người dùng tại các ranh giới hệ thống
- Luôn làm sạch file paths để ngăn chặn directory traversal
- Chạy `npx @claude-flow/cli@latest security scan` sau các thay đổi liên quan đến bảo mật

## Đồng Thời: 1 MESSAGE = TẤT CẢ CÁC THAO TÁC LIÊN QUAN

- Tất cả các thao tác PHẢI được thực hiện đồng thời/song song trong một message
- Sử dụng Task tool của Claude Code để khởi động agents, không chỉ MCP
- LUÔN gộp TẤT CẢ todos trong MỘT lần gọi TodoWrite (tối thiểu 5-10+)
- LUÔN khởi động TẤT CẢ agents trong MỘT message với đầy đủ hướng dẫn qua Task tool
- LUÔN gộp TẤT CẢ các lần đọc/ghi/chỉnh sửa file trong MỘT message
- LUÔN gộp TẤT CẢ lệnh Bash trong MỘT message

## Điều Phối Swarm

- PHẢI khởi tạo swarm bằng công cụ CLI khi bắt đầu các task phức tạp
- PHẢI khởi động agents đồng thời bằng Task tool của Claude Code
- Không bao giờ chỉ dùng công cụ CLI để thực thi — Task tool agents mới là những người thực sự làm việc
- PHẢI gọi công cụ CLI VÀ Task tool trong MỘT message cho công việc phức tạp

### Định Tuyến Mô Hình 3 Tầng (ADR-026)

| Tầng | Xử lý | Độ trễ | Chi phí | Trường hợp sử dụng |
|------|---------|---------|------|----------|
| **1** | Agent Booster (WASM) | <1ms | $0 | Biến đổi đơn giản (var→const, thêm types) — Bỏ qua LLM |
| **2** | Haiku | ~500ms | $0.0002 | Task đơn giản, độ phức tạp thấp (<30%) |
| **3** | Sonnet/Opus | 2-5s | $0.003-0.015 | Lý luận phức tạp, kiến trúc, bảo mật (>30%) |

- Luôn kiểm tra `[AGENT_BOOSTER_AVAILABLE]` hoặc `[TASK_MODEL_RECOMMENDATION]` trước khi khởi động agents
- Sử dụng trực tiếp Edit tool khi `[AGENT_BOOSTER_AVAILABLE]`

## Cấu Hình Swarm & Chống Drift

- LUÔN sử dụng hierarchical topology cho coding swarms
- Giữ maxAgents ở mức 6-8 để điều phối chặt chẽ
- Sử dụng chiến lược specialized cho ranh giới vai trò rõ ràng
- Sử dụng đồng thuận `raft` cho hive-mind (leader duy trì trạng thái có thẩm quyền)
- Chạy checkpoints thường xuyên qua `post-task` hooks
- Giữ shared memory namespace cho tất cả agents

```bash
npx @claude-flow/cli@latest swarm init --topology hierarchical --max-agents 8 --strategy specialized
```

## Quy Tắc Thực Thi Swarm

- LUÔN sử dụng `run_in_background: true` cho tất cả lần gọi Task của agent
- LUÔN đặt TẤT CẢ lần gọi Task của agent trong MỘT message để thực thi song song
- Sau khi khởi động, DỪNG LẠI — KHÔNG thêm lần gọi tool hay kiểm tra trạng thái
- Không bao giờ poll TaskOutput hay kiểm tra trạng thái swarm — tin tưởng agents sẽ trả về
- Khi kết quả agent đến, xem xét TẤT CẢ kết quả trước khi tiếp tục

## Lệnh V3 CLI

### Các Lệnh Cốt Lõi

| Lệnh | Lệnh con | Mô tả |
|---------|-------------|-------------|
| `init` | 4 | Khởi tạo dự án |
| `agent` | 8 | Quản lý vòng đời agent |
| `swarm` | 6 | Điều phối swarm đa agent |
| `memory` | 11 | Memory AgentDB với tìm kiếm HNSW |
| `task` | 6 | Tạo và quản lý vòng đời task |
| `session` | 7 | Quản lý trạng thái session |
| `hooks` | 17 | Hooks tự học + 12 workers |
| `hive-mind` | 6 | Đồng thuận chịu lỗi Byzantine |

### Ví Dụ CLI Nhanh

```bash
npx @claude-flow/cli@latest init --wizard
npx @claude-flow/cli@latest agent spawn -t coder --name my-coder
npx @claude-flow/cli@latest swarm init --v3-mode
npx @claude-flow/cli@latest memory search --query "authentication patterns"
npx @claude-flow/cli@latest doctor --fix
```

## Các Agent Khả Dụng (60+ Loại)

### Phát Triển Cốt Lõi
`coder`, `reviewer`, `tester`, `planner`, `researcher`

### Chuyên Biệt
`security-architect`, `security-auditor`, `memory-specialist`, `performance-engineer`

### Điều Phối Swarm
`hierarchical-coordinator`, `mesh-coordinator`, `adaptive-coordinator`

### GitHub & Repository
`pr-manager`, `code-review-swarm`, `issue-tracker`, `release-manager`

### Phương Pháp SPARC
`sparc-coord`, `sparc-coder`, `specification`, `pseudocode`, `architecture`

## Tham Chiếu Lệnh Memory

```bash
# Lưu trữ (BẮT BUỘC: --key, --value; TÙY CHỌN: --namespace, --ttl, --tags)
npx @claude-flow/cli@latest memory store --key "pattern-auth" --value "JWT with refresh" --namespace patterns

# Tìm kiếm (BẮT BUỘC: --query; TÙY CHỌN: --namespace, --limit, --threshold)
npx @claude-flow/cli@latest memory search --query "authentication patterns"

# Liệt kê (TÙY CHỌN: --namespace, --limit)
npx @claude-flow/cli@latest memory list --namespace patterns --limit 10

# Truy xuất (BẮT BUỘC: --key; TÙY CHỌN: --namespace)
npx @claude-flow/cli@latest memory retrieve --key "pattern-auth" --namespace patterns
```

## Thiết Lập Nhanh

```bash
claude mcp add claude-flow -- npx -y @claude-flow/cli@latest
npx @claude-flow/cli@latest daemon start
npx @claude-flow/cli@latest doctor --fix
```

## Claude Code vs Công Cụ CLI

- Task tool của Claude Code xử lý TẤT CẢ thực thi: agents, thao tác file, tạo code, git
- Công cụ CLI xử lý điều phối qua Bash: swarm init, memory, hooks, routing
- KHÔNG BAO GIỜ sử dụng công cụ CLI thay thế cho Task tool agents

## Hỗ Trợ

- Tài liệu: https://github.com/ruvnet/claude-flow
- Vấn đề: https://github.com/ruvnet/claude-flow/issues
