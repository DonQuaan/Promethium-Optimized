# Phase 0 — Checklist đo FPS Baseline (máy master)

> Mục tiêu: có **số FPS THẬT** làm mốc, thay cho con số marketing. "Đo để học, không để khoe."
> Máy master: i9-14900HX · RTX 4060 Laptop (8GB VRAM) · 32GB RAM · Windows 11 · Java 21 (Microsoft).
> Instance đo: `Promethium Optimized 1.2.2` (MC 1.21.10, Fabric 0.19.2, stack VulkanMod).

---

## BƯỚC 0 — Điều kiện chặn: sửa OOM trước (nếu không, pack sập giữa chừng)

Bằng chứng: `hs_err_pid65824.log` = **native OOM** do heap 16GB trên máy 31GB. **Phải sửa trước khi đo.**

**Thay đổi (Claude làm giúp, hoặc master tự làm trong PrismLauncher UI):**
1. RAM: `MaxMemAlloc` 16096 → **10240** ; `MinMemAlloc` 8512 → **10240** (min = max = 10GB).
2. Bật JVM args (Prism → instance → Edit → Settings → Java arguments):
   ```
   -XX:+UseZGC -XX:+ZGenerational -XX:+AlwaysPreTouch -XX:+UseStringDeduplication -XX:+PerfDisableSharedMem
   ```
   *(Không dán -Xms/-Xmx vào đây — slider RAM quản.)*
> ⚠️ **Đóng PrismLauncher trước khi sửa file**, nếu không Prism ghi đè lại khi thoát.

---

## BƯỚC 1 — Chuẩn bị máy (bắt buộc để số đo tái lập)

- [ ] **Cắm sạc** + Windows power plan = **High Performance** (14900HX throttle nặng khi chạy pin).
- [ ] Đóng app nền nặng (Chrome nhiều tab, Discord stream…).
- [ ] Gán GPU rời cho Java: Windows → Settings → Display → Graphics → thêm `javaw.exe` (trong thư mục Java của Prism) → **High performance (RTX 4060)**. Tránh rơi về iGPU Intel.
- [ ] Trong game: VSync **OFF** (đã off), maxFps tạm để **260/uncapped** khi đo trần.

---

## BƯỚC 2 — Công cụ đo (đo cái gì)

- **BetterF3** (đã có trong pack) hoặc **F3**: đọc FPS trực tiếp.
- Nâng cao (khi cần đào nút thắt): cài **spark** (`/spark profiler --thread *` → flame graph trên spark.lucko.me) + **Performance Overlay** (cho 1%-low).
- Số cần ghi: **FPS trung bình** + (nếu có) **1%-low** (chỉ số giật thật). Bỏ 30–60s đầu (warmup).

---

## BƯỚC 3 — Kịch bản đo (Tầng 1: đơn giản, master làm được ngay)

Đứng yên tại mỗi cảnh ~30s (sau warmup), đọc FPS, ghi vào bảng. Lặp mỗi cảnh ở **3 mức render distance**.
Cùng 1 thế giới, teleport về cùng toạ độ mỗi lần để công bằng.

| # | Cảnh (scene) | Nút thắt cô lập | RD 8 | RD 16 | RD 32 |
|---|---|---|---|---|---|
| A | Đồng bằng/rừng thường (baseline) | Tổng hợp | | | |
| B | Rừng rậm/lá dày (jungle) | Chunk-build / GPU | | | |
| C | Trong hang deepslate | Overhead thuần | | | |
| D | Chỗ đông mob (farm/spawner) | CPU / tick | | | |
| E | Base đã xây (nhiều chest/rương) | Block-entity / stutter | | | |

> Ghi thêm điều kiện mỗi lần: shader OFF (VulkanMod không có shader), F3 tắt khi đọc số cuối, fullscreen.

## BƯỚC 4 — (Tầng 2, tùy chọn) Tìm nút thắt bằng spark
Tại cảnh FPS thấp nhất → `/spark profiler --thread *` chạy 60s → gửi link `spark.lucko.me` cho Claude đọc Render thread / GC time / MSPT p95.

---

## Cách đọc kết quả
- FPS thấp mà **GPU rảnh** (Task Manager GPU <50%) → **CPU-bound** (bình thường khi off shader) → tối ưu tick/culling/RD.
- FPS thấp mà **GPU ~100%** → GPU-bound → giảm RD/AO/hiệu ứng.
- Có spike giật dù avg cao → xem 1%-low + GC time trong spark.

## Sau khi có bảng số
Claude dùng bảng này để: (1) chốt **mục tiêu FPS tiered thật**; (2) so sánh trước/sau mỗi tối ưu; (3) quyết định backend (VulkanMod vs Sodium+Nvidium) và version bằng **A/B có số**.
