# PLAN — Promethium Optimized (PO) thế hệ mới

> Tài liệu kế hoạch nền, lập trên nghiên cứu đa nguồn có phản biện chéo (2026-07-11).
> Nguyên tắc: **đo bằng số thật, không hứa suông**. Mọi con số FPS dưới đây là **ước tính** cho tới khi benchmark trên chính máy master.

---

## 0. TL;DR điều hành

- Pack cũ **không phải fork Fabulously Optimized (FO)** — nó là pack tự tuyển cùng ADN với FO. Để làm PO ta **không cần "mã nguồn" FO**; ta cần (a) danh sách mod + config công khai của FO làm tham chiếu, và (b) **tôn trọng license của từng mod con** khi phân phối.
- Bản mới nhất (1.2.2) đã **âm thầm đổi hướng cực đoan**: bỏ Sodium+Iris, chuyển sang **VulkanMod + Voxy**. Đây là canh bạc — trên GPU **Nvidia (RTX 4060)** lợi ích VulkanMod thường **thấp**, lại **mất shader** và **đụng nhiều mod**. Cần đo A/B, không mặc định giữ.
- **Crash mà master gặp chỉ là 1 dòng config** (`avoid_redundant_framebuffer_switching`), **không phải lỗi version** → **không cần downgrade 10 phiên bản**.
- **"1000 FPS" chỉ đúng ở góc hẹp** (không shader, render distance thấp, cảnh đơn giản, chế độ benchmark). Gameplay thật ~300–600 FPS; bật shader ~60–120 FPS. Trên "i3 cũ + 8GB": **bất khả thi cho pack này** (min RAM pack yêu cầu đã 8.5GB > 8GB tổng). → chuyển sang **mục tiêu FPS phân tầng (tiered) kèm điều kiện**.
- Ba thứ **đã chốt được ngay, không cần bàn**: (1) JVM = **Generational ZGC**, RAM **min=max=8–10GB** (12–16GB là **quá nhiều, sai**); (2) **dọn bloat** (2 mod dynamic-light, 2 mod fps-limit, mod particle/sound/visual đi ngược "Optimized"); (3) **kỷ luật version** — pack cũ lệch version 4 chiều.
- **4 quyết định chiến lược** cần master chốt (§4) trước khi bắt tay.

---

## 1. Hiện trạng đã kiểm chứng (audit pack cũ)

**Định dạng:** instance **Prism/MultiMC** + manifest **CurseForge** (`flame/manifest.json`). 5 bản: 1.0.0 → 1.2.2. Author manifest ghi `yangdawn` (cần master xác nhận đây là handle của mình — liên quan attribution/license nếu phát hành).

**Lệch version nghiêm trọng (bản 1.2.2):**

| Nguồn | MC | Loader | Version pack |
|---|---|---|---|
| Tên thư mục | — | — | 1.2.2 |
| `flame/manifest.json` | 1.21.11 | fabric-0.18.4 | 1.2.0 |
| `instance.cfg` | — | — | Export 1.2.1 / Managed 1.2.0 / name 1.2.2 |
| `mmc-pack.json` (launch thật) | **1.21.10** | **0.19.2** | — |
| jar trong `mods/` | trộn 1.21.9 (28) · 1.21.10 (54) · 1.21.11 (4) · 1.21.5/6 | — | — |

→ Cùng một pack mang **3 số version + 2 mức MC + 2 loader**. Manifest chỉ liệt kê 47 projectID nhưng `mods/` thật có **111 jar**. Đây là vi phạm kỷ luật version phải sửa tận gốc.

**RAM:** `-Xms8512m -Xmx16096m`, **không có tuning GC** (chỉ Xms/Xmx + HeapDumpPath).

**Crash master gặp** (`crash-2026-04-04`): `NullPointerException` tại `GlCommandEncoder.clearColorAndDepthTextures` — **6 mixin chồng lên cùng 1 class** (ImmediatelyFast · Iris · Sodium · RRLS · RenderScale). Root cause: `avoid_redundant_framebuffer_switching=true` của ImmediatelyFast bỏ qua bind framebuffer "thừa", trong khi Iris/RenderScale tự quản render target → texture bị null. **Đây là crash của lần chạy 1.2.1** (stack Sodium/Iris), còn lưu trong thư mục 1.2.2.
**Cách sửa:** đặt `avoid_redundant_framebuffer_switching=false` (1 dòng) HOẶC gỡ ImmediatelyFast. Không liên quan version.

---

## 2. Chẩn đoán — 4 vấn đề gốc của pack cũ

1. **Mâu thuẫn triết lý.** Pack tên "Optimized" nhưng nhồi **content nặng** (Tectonic/Nullscape/Explorify worldgen, AmbientSounds/Sound Physics, particlerain/visuality, connected-glass/fusion). Content này **kéo FPS xuống** và mở rộng scope ngoài mục tiêu tối ưu. → Phải quyết bản chất pack (§4-A).
2. **Chọn render backend chưa được kiểm chứng.** VulkanMod là canh bạc trên phần cứng Nvidia; đồng thời vẫn để lại **mod hướng Sodium/OpenGL** (ImmediatelyFast, MoreCulling) → thừa hoặc lỗi dưới Vulkan. → Phải A/B (§4-B).
3. **Bloat & trùng lặp.** 2 engine dynamic-light (LambDynamicLights + Dynamic Lights cũ mc1.17), 2 fps-limiter (Dynamic FPS + FpsReducer2), nhiều mod visual/sound/particle. Trùng chức năng = xung đột + phí tài nguyên.
4. **Không có tầng đo lường & JVM tuning.** Chưa từng đo FPS có kiểm soát; JVM thiếu GC → mọi tuyên bố hiệu năng đều cảm tính.

---

## 3. Phản biện "1000 FPS" → Mục tiêu FPS phân tầng

**Sự thật kỹ thuật:** Minecraft Java bị chặn bởi **1 luồng CPU**; khi tắt shader, GPU gần như rảnh → "RTX siêu cũ" không phải biến quyết định ở low-end, **CPU i3 cũ mới là trần**. Chi phí render distance là **bậc hai** `(2d+1)²`. Bật shader → lập tức **GPU-bound**.

**Mục tiêu FPS tiered (ước tính — chốt bằng benchmark):** mỗi số **phải kèm 3 điều kiện** `[render distance] × [shader on/off] × [scene]`.

| Máy | Shader OFF, RD 4–8, cảnh đơn giản (benchmark) | Shader OFF, RD 12–16, gameplay | Shader ON, RD 12–16 @1080p |
|---|---|---|---|
| **Master** (i9-14900HX + RTX 4060 Laptop + 32GB) | **800–1200+ FPS** ← chỗ "1000 FPS" trung thực | 300–600 FPS | 60–120 FPS |
| Mid-range (i5/R5 + RTX 3060/4060 + 16GB) | 400–700 | 150–300 | 45–90 |
| Low-end (i3 cũ / GTX 1650–RTX 2060 + **16GB**) | 100–250 | 60–120 | <30 (khuyên tắt) |

**"i3 cũ + 8GB tổng":** mâu thuẫn với chính pack (min alloc 8.5GB) → OOM/GC thrash. Kỳ vọng thực tế 30–60 FPS giật, RD ≤6, tắt content nặng. **Tuyệt đối không hứa 1000 FPS ở đây.**

**Cách phát ngôn trung thực mà vẫn mạnh:** *"Đạt ~1000 FPS ở chế độ hiệu năng tối đa (RD 8, không shader) trên máy cấu hình cao; ~300–600 FPS khi chơi thật; ~60–120 FPS khi bật shader."* Hoặc nhãn **"1000+ FPS-capable"** kèm chú thích điều kiện.

---

## 4. Bốn quyết định chiến lược cần master chốt

### A. Bản chất PO
- **A1 — Pure Performance** (giống FO): chỉ mod tối ưu, giữ vanilla gameplay → **FPS cao nhất, ổn định nhất**.
- **A2 — Content + Performance**: giữ biomes/structures/sound đẹp → FPS thấp hơn, phức tạp hơn.
- **A3 — Tách 2 SKU** (khuyến nghị): `PO Performance` (lite, cho máy yếu/8GB, mục tiêu 60–144 FPS ổn định) + `PO Full` (content, cho máy mạnh). Đúng tinh thần "một lời hứa cho mỗi cấu hình".

### B. Render backend (quyết bằng A/B benchmark, không giáo điều)
- **B1 — Sodium + Iris (+ Nvidium cho chế độ max-FPS)**: chuẩn FO, chín, ổn định, **có shader đẹp**; Nvidium mesh-shader là **đòn bẩy FPS lớn nhất trên Nvidia** (nhưng tắt khi bật shader, và tốn VRAM — 4060 Laptop chỉ 8GB).
- **B2 — VulkanMod** (pack đang dùng): loại nút thắt OpenGL, nhưng **mất shader**, đụng ImmediatelyFast/MoreCulling, còn alpha; lợi ích trên Nvidia thường thấp.
- → **Đề xuất:** đo A/B thật trên máy master rồi chốt. Trực giác kỹ thuật nghiêng về **B1** cho phần cứng Nvidia.

### C. MC version (quyết bằng benchmark + bối cảnh content)
- Đề xuất version ban đầu là 1.21.1 **đã bị phản biện bác bỏ**: (1) tiền đề "chỉ 1.21.1 có Nvidium" **sai** — đã có port **Alphadium cho 1.21.11**; (2) downgrade sẽ **phá gần hết content** đang có ở 1.21.10/11; (3) crash chỉ là config.
- → **Đề xuất:** **ở lại 1.21.10/1.21.11** (hoặc lùi nhẹ **1.21.8** — điểm ngọt "mới mà đã ổn sau đợt viết lại Blaze3D"), giữ content, **sửa config** thay vì đổi nền. Chốt cuối bằng benchmark.

### D. Mục tiêu FPS
- Chấp nhận **mục tiêu tiered kèm điều kiện** (§3) thay cho "1000 FPS" trần trụi.

---

## 5. Nền tảng kỹ thuật ĐÃ CHỐT (không cần bàn thêm)

### 5.1 JVM & RAM (áp dụng ngay cho máy master)
- **GC = Generational ZGC** (không phải Aikar/G1 — Aikar's flags là **G1-only cho server**, sai công cụ cho client Java 21). Máy 24-core/32GB đủ sức hấp thụ overhead ZGC; pause sub-millisecond.
- **JVM args** (đặt ở Prism → *KHÔNG* nhét `-Xms/-Xmx` vào ô này, để slider quản):
  ```
  -XX:+UseZGC -XX:+ZGenerational -XX:+AlwaysPreTouch -XX:+UseStringDeduplication -XX:+PerfDisableSharedMem
  ```
- **RAM = min = max = 8GB** (nâng 10GB nếu OOM). **12–16GB là quá nhiều** → GC pause + lãng phí. Đặt cả 2 slider Prism = 8192.
- **Java = Microsoft/Temurin OpenJDK 21** (đang có). **Không GraalVM** (chỉ hỗ trợ G1, mất ZGC).

### 5.2 Phương pháp benchmark (bắt buộc — "đo để học, không để khoe")
- **Metric:** avg FPS **+ 1%-low (và 0.1%-low)** — frametime graph phải phẳng.
- **Công cụ:** `spark` (`/spark profiler --thread *` → flame graph trên spark.lucko.me, đọc Render thread + GC time + MSPT p95) · Performance Overlay mod · BetterF3 (đã có sẵn trong pack) · đối chiếu PresentMon/CapFrameX ở cấp OS.
- **Điều kiện cố định để tái lập:** RD cố định (test 8/16/32), SD cố định, VSync OFF, cùng seed + cùng toạ độ (teleport), **cắm sạc + power plan High Performance** (14900HX throttle nặng khi chạy pin), bỏ 30–60s warmup (JIT + load chunk), chạy kịch bản scripted 60–120s, lặp ≥3 lần.
- **Scene chuẩn** (mỗi cảnh cô lập 1 nút thắt): rừng baseline · jungle/lá dày (chunk-build/GPU) · hang deepslate (overhead) · đông mob (CPU/tick) · base nhiều block-entity (stutter đời thực).
- **Sau MỖI thay đổi lớn → profile lại**, so 1%-low/p95 trước–sau mới kết luận "xong".

### 5.3 Dọn bloat (an toàn, làm sớm)
- Bỏ **Dynamic Lights (cũ, mc1.17)** — trùng LambDynamicLights.
- Tắt phần idle của **FpsReducer2** — nhường Dynamic FPS làm chủ idle.
- Đánh giá lại **mod content/visual/sound** theo quyết định §4-A.

### 5.4 Nguyên tắc chống xung đột mixin (bài học từ crash)
1. **Một chủ sở hữu renderer/tầng** — không chạy song song 2 backend (Sodium ⊥ VulkanMod).
2. **Đếm mixin trên "hot shared class"** (GlCommandEncoder, LevelRenderer, RenderTarget, GameRenderer): >2 mod cùng method = mong manh. Dùng chính mục "Mixins in Stacktrace" của crash-report làm công cụ audit.
3. **Khi 2 mod đụng: tắt cái dùng `@Redirect`/`@Overwrite`, giữ cái dùng `@Inject`.**
4. **Verification gate:** sau mỗi thay đổi tầng render → launch 1 lần, `grep` `logs/latest.log` tìm `Mixin apply failed` / `Mixins in Stacktrace` **trước khi** tuyên bố ổn.

---

## 6. Lộ trình (roadmap)

| Phase | Mục tiêu | Đầu ra |
|---|---|---|
| **P0 — Baseline** | Sửa config crash (1 dòng) → pack chạy ổn → **đo FPS thật** trên máy master theo §5.2, lập bảng {RD × shader × scene} | Bảng số baseline THẬT (thay số marketing) |
| **P1 — Dọn & nền** | Áp JVM §5.1, dọn bloat §5.3, sửa lệch version §7, dựng repo git | Pack "sạch" chạy ổn định + đo lại |
| **P2 — A/B lớn** | A/B **backend** (B1 vs B2) và **version** (C) bằng benchmark; chốt §4-B, §4-C bằng số | Quyết định backend + version có bằng chứng |
| **P3 — Config cực đoan** | Tách RD/SD (SD 32→6, RD native thấp + Voxy LOD), entity/leaf culling, dynamic-fps, Sodium/VulkanMod extreme settings; mỗi bước verify lại | Pack tối ưu cực hạn + bảng FPS tiered đã kiểm chứng |
| **P4 — (sau) Mod riêng** | Chỉ khi P0–P3 lộ ra **nút thắt còn sót** mà mod hiện có chưa xử → thiết kế mod Fabric nhắm đúng điểm đó | Spec mod dựa trên profiling thật |

---

## 7. Kỷ luật version & repo (bắt buộc theo doctrine)

- **Khởi tạo git** cho dự án (hiện chưa phải git repo). Mỗi deploy: **tăng version theo scheme riêng + tạo git tag kèm mô tả**. Không tự tăng major khi chưa master duyệt.
- **Scheme đề xuất:** `MAJOR.MINOR.PATCH` — MAJOR = đổi nền lớn (backend/MC version), MINOR = thêm/bớt mod, PATCH = tinh chỉnh config. **Một nguồn version duy nhất** (manifest = tên thư mục = tag), chấm dứt lệch 4 chiều.
- Cấu trúc self-contained: `CLAUDE.md` riêng cho project + thư mục `docs/benchmarks/` lưu số đo từng phase.

---

## 8. Rủi ro & pháp lý

- **License mod:** 47–111 mod, mỗi cái license riêng (Sodium = Polyform Shield, Lithium/Iris/Nvidium = LGPL, FerriteCore/C2ME/EntityCulling = MIT…). Được FO tham chiếu **không** cấp quyền các mod đó. Nếu **phát hành** PO → phải rà license từng mod (đặc biệt điều khoản phân phối lại). **Việc cần làm trước khi public.**
- **VRAM 8GB (4060 Laptop):** Nvidium/Voxy ở render distance lớn dễ cạn VRAM → stutter/crash. Phải đo.
- **Laptop throttle:** 14900HX/4060 Laptop giới hạn TGP/nhiệt — benchmark phải cắm sạc, so với desktop cùng tên là khập khiễng.
- **Attribution:** xác nhận quyền với `yangdawn`/pack gốc trước khi phát hành công khai.

---

## 9. Nguồn tham chiếu chính
Sodium/Nvidium/Lithium (modrinth) · VulkanMod (github xCollateral, deepwiki) · brucethemoose Minecraft-Performance-Flags · Oracle Gen ZGC (JEP 439/474) · spark.lucko.me · nemez.net CPU testing · minecraft.wiki (version history, simulation distance) · FabricMC blog 26.2 · Alphadium (Nvidium port 1.21.11). *(Danh sách URL đầy đủ trong log nghiên cứu.)*
