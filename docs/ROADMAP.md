# Lộ trình phát triển TFT Hub (10 Tuần — 5 Phases)

Lộ trình được thiết kế cho **Java Intern**, bám sát môi trường thực tế trên máy (**OpenJDK 17**, `./mvnw.cmd`, **PostgreSQL 16 local**, **Node v22**) và áp dụng **Chế độ Mentor 5 bước** (học hiểu bản chất, tự viết phần lặp lại, thí nghiệm Break-It-On-Purpose, Active Recall).

---

## Phase 1 (Tuần 1–2): Database, Flyway & Đồng bộ dữ liệu CommunityDragon

### Tuần 1: Kết nối PostgreSQL 16, Flyway Migration & JPA Entities cốt lõi
- **Mục tiêu học**: Tại sao dùng Flyway thay `ddl-auto=update`; cấu trúc Modular Monolith; tránh lỗi `@Data` và `FetchType.EAGER` trên Entity; Global Exception Handling với `ProblemDetail`.
- **Công việc**:
  1. Tạo database `tft_hub` trên PostgreSQL 16 local đang chạy (`postgresql-x64-16`).
  2. Cấu hình [application.properties](file:///e:/Workspace/tft-hub/tft-hub/src/main/resources/application.properties) dùng biến môi trường `${DB_PASSWORD:postgres}` và bật `spring.jpa.hibernate.ddl-auto=validate`, `spring.jpa.open-in-view=false`.
  3. Tạo cấu trúc package Modular Monolith: `common`, `patch`, `catalog`, `comp`, `stats`, `sync`, `admin`.
  4. Viết migration `V1__init_static_schema.sql` và `V2__init_meta_and_stats_schema.sql` kèm đầy đủ Foreign Key và Index theo `docs/ARCHITECTURE_AND_ERD.md`.
  5. Tạo bộ JPA Entities (Mentor viết mẫu `Patch` & `Champion`, bạn tự viết phần còn lại để Review PR).
  6. Cài đặt `GlobalExceptionHandler` (`@RestControllerAdvice` trả `ProblemDetail`) và `CorsConfig`.
- **Break-It-On-Purpose**: Thử sửa tên 1 cột trong Entity rồi chạy `./mvnw.cmd spring-boot:run` để quan sát `ddl-auto=validate` chặn khởi động như thế nào khi lệch migration.
- **Checkpoint**: `./mvnw.cmd spring-boot:run` khởi động sạch sẽ, Flyway tạo đủ 14 bảng và Hibernate validate pass 100%.

### Tuần 2: Đồng bộ song ngữ CommunityDragon & Seed Meta mẫu
- **Mục tiêu học**: `RestClient` của Spring; parse JSON bằng Jackson; thiết kế thuật toán **Upsert / Idempotent Sync** (chạy lại nhiều lần không nhân đôi dữ liệu); tích hợp Swagger/OpenAPI (`springdoc-openapi`).
- **Công việc**:
  1. Viết `CommunityDragonSyncService` tải song song `vi_vn.json` và `en_us.json`, gộp theo `api_name`.
  2. Phân loại chuẩn `items` (`COMPONENT`, `CRAFTABLE`, `EMBLEM` + cờ `is_craftable_emblem`, `ARTIFACT` cho đồ Ornn, `RADIANT`) và `augments` (`SILVER`, `GOLD`, `PRISMATIC`).
  3. Tạo `POST /api/v1/admin/sync/cdragon?set=...` (tạm mở trong profile `dev`, khóa bằng JWT ở Phase 4).
  4. Viết `DevDataSeeder` (hoặc `V3__seed_sample_meta.sql`) nạp 12–15 đội hình meta mẫu (`S/A/B/C`) cùng stats giả lập hợp lý cho tướng, đồ, ấn, đồ Ornn và lõi nâng cấp.
  5. Tích hợp Swagger UI để kiểm thử API.
- **Checkpoint**: Kiểm tra DB thấy đủ Tướng, Tộc/Hệ, Đồ, Ấn, Đồ Ornn, Lõi kèm tên song ngữ Anh–Việt và URL ảnh hợp lệ.

---

## Phase 2 (Tuần 3–4): REST API cho toàn bộ 4 Module (Backend)

### Tuần 3: API Đội Hình (`/comps`) & Tướng (`/units`)
- **Mục tiêu học**: Java `record` DTO; **JPA `Specification`** cho dynamic filter; xử lý **N+1 Query** bằng 2-step batch query (`IN (:ids)`) + gom `Map` trong Service.
- **Công việc**:
  1. API `GET /api/v1/patches` và `GET /api/v1/traits`.
  2. API `GET /api/v1/comps` (filter theo `tier`, `playstyle`, `trait`, `champion`, `search`) và `GET /api/v1/comps/{slug}` (đủ `EARLY / MID / LATE` hex board 4×7 + augments).
  3. API `GET /api/v1/units` (trả về cả dữ liệu cho `GRID` & `LIST` kèm `topItems` và `bestComps`).
  4. API `GET /api/v1/units/{apiName}` (hỗ trợ `starLevel=0|1|2|3`, phân bố sao, phân bố hạng 1–8) và `GET /api/v1/units/{apiName}/items` (Tier list đồ theo tướng sort theo `delta`).
- **Break-It-On-Purpose**: Bật `spring.jpa.show-sql=true` khi truy vấn danh sách bằng lazy getter trong vòng lặp để đếm số lượng query N+1 phát sinh, sau đó refactor sang batch query để giảm về số câu SQL cố định.
- **Checkpoint**: Gọi `/api/v1/comps` và `/api/v1/units` chỉ phát sinh 2–4 câu SQL cố định bất kể số lượng record.

### Tuần 4: API Đồ / Ấn / Đồ Ornn (`/items`), Lõi (`/augments`) & Testing
- **Mục tiêu học**: Hybrid Tier `COALESCE(curated_tier, calculated_tier)`; **Unit Test** (JUnit 5 + Mockito) và **Controller Slice Test** (`@WebMvcTest`).
- **Công việc**:
  1. Hoàn thiện `TierCalculator` tính `calculated_tier` dựa trên ngưỡng `AVP`/`Delta`.
  2. API `GET /api/v1/items` (phục vụ cả `TIER VIEW` lẫn `STATS TABLE` cho `CRAFTABLE`, `EMBLEM`, `ARTIFACT`, `RADIANT`), `GET /api/v1/items/{apiName}`, `GET /api/v1/items/{apiName}/holders`.
  3. API `GET /api/v1/augments` (lọc bậc `SILVER/GOLD/PRISMATIC`, nhóm, vòng `2-1/3-2/4-2`, trả kèm `bestComps`).
  4. Viết Unit Test cho `TierCalculator` + các Service chính và `@WebMvcTest` cho mỗi Controller.
- **Checkpoint**: `./mvnw.cmd test` xanh 100%; toàn bộ API `v1` test mượt trên Swagger.

---

## Phase 3 (Tuần 5–7): Frontend Angular — UI/UX kết hợp TFT Academy & tactics.tools

> **Lưu ý trước khi bắt đầu Tuần 5**: Cấp quyền PowerShell (`Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`), cài Angular CLI (`npm install -g @angular/cli`), và tạo repo thứ 2 `e:\Workspace\tft-hub\tft-hub-web`.

### Tuần 5: Khởi tạo Angular, Dark Gaming Theme & Shared Components
- **Mục tiêu học**: Standalone Components, **Angular Signals** (`signal`, `computed`, `effect`), Control Flow mới (`@if`, `@for`), Typed `HttpClient` services, Tailwind CSS.
- **Công việc**:
  1. Khởi tạo `tft-hub-web` + cài Tailwind CSS, cấu hình theme tối (`#0b0e14`, `#141923`, viền giá vàng 1–5, badge `S/A/B/C`, tô màu `AVP` & `Delta` theo quy chuẩn `tactics.tools`).
  2. Xây Top Navigation (`Đội Hình` | `Tướng` | `Trang Bị` | `Ấn` | `Đồ Ornn` | `Lõi`), Global Patch/Lang (`VI | EN`) Selector dùng Signal Service.
  3. Xây 4 Shared Components: `ChampionTileComponent`, `ItemBadgeComponent` (popover công thức + mô tả), `TierBadgeComponent`, `StatsTableComponent` (click header để sort).
- **Checkpoint**: Frontend gọi được API `/api/v1/patches` và `/api/v1/traits` từ Spring Boot mà không vướng lỗi CORS.

### Tuần 6: Màn hình `/comps` (Hex Grid 4×7) & `/units` (GRID/LIST + Modal)
- **Mục tiêu học**: Đồng bộ filter với URL Query Params; vẽ lưới lục giác 4×7 bằng CSS Grid + `clip-path`; Lazy Loading routes; Modal chi tiết tái sử dụng.
- **Công việc**:
  1. Trang `/comps` (kiểu TFT Academy + stats): lọc Tier/Playstyle/Tộc Hệ/Tướng, card tóm tắt, click vào `/comps/:slug` xem bàn cờ lục giác 4×7 (`EARLY / MID / LATE`), ưu tiên đi chợ, mẹo chơi, lõi khuyên dùng.
  2. Trang `/units` (kiểu `tactics.tools/units`): chuyển `GRID` (nhóm theo Cost 1–5) | `LIST` (bảng có `3★ Rate` & `3★ Place`), bổ sung Search/Cost/Trait filter.
  3. **Unit Detail Modal / Page**: lọc `Any / 1★ / 2★ / 3★`, biểu đồ phân bố sao & hạng 1–8, tab **Đồ theo tướng (Delta)** và tab **Đội hình mạnh**.
- **Checkpoint**: Có thể click xuyên suốt giữa Trang Tướng -> Đồ chuẩn theo tướng -> Đội hình meta trên bàn cờ 4×7.

### Tuần 7: Màn hình `/items` (Hybrid Tier + Stats) & `/augments`
- **Mục tiêu học**: RxJS `debounceTime` + `distinctUntilChanged` + `switchMap` cho ô search; tái sử dụng cấu trúc `TIER VIEW | STATS TABLE`.
- **Công việc**:
  1. Trang `/items` (cũng alias route cho `/emblems` & `/artifacts` nếu cần): 5 tab (`Craftables`, `Artifacts`, `Emblems`, `Radiants`, `All`), nút chuyển **`TIER VIEW`** (hàng S/A/B/C kiểu TFT Academy) | **`STATS TABLE`** (bảng số liệu kiểu tactics.tools), riêng tab Ấn có lọc `Ghép được / Không ghép được`.
  2. **Item Detail (`/items/:apiName`)**: phân bố hạng 1–8 + 3 sub-tab `Tướng cầm ngon nhất (Delta)`, `Đội hình đi kèm`, `Đồ hay đi chung`.
  3. Trang `/augments`: lọc `Silver / Gold / Prismatic`, `Category`, tab vòng `2-1 / 3-2 / 4-2`, kèm danh sách **Đội hình mạnh đi kèm**.
- **Checkpoint**: Toàn bộ 4 module hoạt động trơn tru trên Desktop & Tablet, hỗ trợ chuyển nhanh ngôn ngữ `VI | EN`.

---

## Phase 4 (Tuần 8): Admin CMS & Spring Security JWT

- **Mục tiêu học**: `spring-boot-starter-security`, Stateless **JWT Authentication**, phân quyền `ROLE_ADMIN`, Angular `HttpInterceptor` & `CanActivateFn`.
- **Công việc**:
  1. Backend: Khóa toàn bộ `/api/v1/admin/**`, giữ public toàn bộ `GET /api/v1/**`; cài đặt Login JWT; viết API CRUD cho `comps` và override `curated_tier`.
  2. Frontend: Trang `/admin/login`, bảng quản trị đội hình, và **Comp Builder** trực quan (click đặt tướng lên bàn cờ Hex 4×7, gán 3 đồ, chọn lõi khuyên dùng, xuất bản ngay).
- **Checkpoint**: Đăng nhập Admin, tạo/chỉnh sửa 1 bài meta trên bàn cờ 4×7 và thấy xuất hiện tức thì ngoài trang chủ `/comps`.

---

## Phase 5 (Tuần 9–10): Caching, Dữ liệu thật từ Riot API & Deploy

### Tuần 9: Spring Cache & (Tùy chọn) Riot API Match Aggregator
- **Mục tiêu học**: Read-heavy Caching (`@Cacheable`, `@CacheEvict` với Caffeine hoặc Redis); `@Scheduled` background jobs; SQL Aggregation (`EXPLAIN ANALYZE`).
- **Công việc**:
  1. Áp dụng `@Cacheable` cho các API thống kê/tier list và `@CacheEvict` khi sync hoặc Admin cập nhật bài.
  2. *(Nâng cao)* Kết nối Riot Games TFT API: lấy trận đấu bậc Cao Thủ/Thách Đấu vào bảng `raw_matches` (JSONB), tính `AVP` và `Delta` thật đổ vào các bảng `*_stats`.
- **Break-It-On-Purpose**: Chạy `EXPLAIN ANALYZE` trên bảng `champion_item_stats` có và không có composite index để đo thời gian quét bảng.

### Tuần 10: Dockerize, GitHub Actions CI & Deploy Production
- **Mục tiêu học**: Multi-stage `Dockerfile`, Nginx fallback cho Angular SPA (`try_files`), GitHub Actions CI (`./mvnw test` + `ng build`), quản lý biến môi trường production.
- **Công việc**:
  1. Viết `Dockerfile` cho `tft-hub` và `tft-hub-web` + `docker-compose.yml` hoàn chỉnh.
  2. Thiết lập GitHub Actions chạy test tự động khi push/PR.
  3. Deploy PostgreSQL (Neon/Supabase), Backend (Render/Railway/Fly.io), Frontend (Vercel/Cloudflare Pages) và hoàn thiện `README.md` kèm ảnh demo.
