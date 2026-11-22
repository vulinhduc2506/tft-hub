# Tiến độ dự án TFT Hub

> **Trạng thái hiện tại**: **Phase 0 hoàn tất — Sẵn sàng bắt đầu Tuần 1 (Phase 1)**  
> **Bước tiếp theo ngay bây giờ**: Kết nối PostgreSQL 16 local (`tft_hub`), cấu hình `application.properties`, tạo bộ package Modular Monolith và viết Flyway `V1` & `V2`.

---

## Thông tin môi trường & Git (Đã kiểm tra thực tế)
- **Backend Repo**: `e:\Workspace\tft-hub\tft-hub` — Remote `origin`: `https://github.com/vulinhduc2506/tft-hub` (branch `main`, commit `23502e2`).
- **Tech Stack Backend**: Spring Boot `4.1.2`, Java `17`, Flyway, Spring Data JPA, Validation, Lombok, `spring-boot-starter-webmvc`.
- **Công cụ trên máy**:
  - OpenJDK `17.0.21 LTS` (`JAVA_HOME=E:\JDK`)
  - `./mvnw.cmd` (chưa cài `mvn` global)
  - PostgreSQL 16 service Windows `postgresql-x64-16` đang `Running` (không cần Docker ở Phase 1–4)
  - Node `v22.14.0` / npm `10.9.2` (lưu ý: PowerShell đang để `ExecutionPolicy = Undefined`, trước khi làm Angular ở Tuần 5 cần chạy `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` và cài `@angular/cli`)
  - Git `2.48.1`

---

## Checklist theo từng Phase

### Phase 0: Khởi tạo & Chốt nghiệp vụ / Kiến trúc
- [x] Khởi tạo project Spring Boot 4.1.2 + PostgreSQL + Flyway và đẩy lên GitHub (`23502e2`)
- [x] Nghiên cứu thực tế UX/Data của `tactics.tools/items`, `tftacademy.com/tierlist/items`, `tactics.tools/units`
- [x] Chốt 4 Module nghiệp vụ + 3 quyết định tối ưu (Hybrid Tier, Future-proof Schema, Song ngữ VI+EN)
- [x] Chốt kiến trúc Modular Monolith + 2-repo Git strategy
- [x] Thiết lập `AGENTS.md` (Mentor Mode 5 bước) và bộ tài liệu `docs/`

### Phase 1: Database, Flyway & CommunityDragon Sync (Tuần 1–2)
- [ ] **Tuần 1.1**: Tạo DB `tft_hub` trên PostgreSQL 16 local & cấu hình `application.properties` (dùng env var cho password, `ddl-auto=validate`, `open-in-view=false`)
- [ ] **Tuần 1.2**: Tổ chức package Modular Monolith (`common`, `patch`, `catalog`, `comp`, `stats`, `sync`, `admin`) + `GlobalExceptionHandler` (`ProblemDetail`) + `CorsConfig`
- [ ] **Tuần 1.3**: Viết `V1__init_static_schema.sql` & `V2__init_meta_and_stats_schema.sql` (kèm Foreign Keys & Indexes)
- [ ] **Tuần 1.4**: Viết bộ JPA Entities (Mentor mẫu `Patch` & `Champion`, bạn tự viết phần còn lại & Review PR) + Break-It-On-Purpose `ddl-auto=validate`
- [ ] **Tuần 2.1**: Viết `CommunityDragonSyncService` tải song ngữ `vi_vn.json` & `en_us.json` (Idempotent Upsert)
- [ ] **Tuần 2.2**: Phân loại `COMPONENT / CRAFTABLE / EMBLEM / ARTIFACT / RADIANT` và `SILVER / GOLD / PRISMATIC`
- [ ] **Tuần 2.3**: Viết `DevDataSeeder` / `V3__seed_sample_meta.sql` (12–15 bài meta + stats mẫu) & tích hợp Swagger UI

### Phase 2: Backend REST API v1 & Testing (Tuần 3–4)
- [ ] **Tuần 3.1**: API `GET /api/v1/patches` & `GET /api/v1/traits`
- [ ] **Tuần 3.2**: API `GET /api/v1/comps` (JPA `Specification` filter) & `GET /api/v1/comps/{slug}` (Hex board `EARLY/MID/LATE`)
- [ ] **Tuần 3.3**: API `GET /api/v1/units`, `GET /api/v1/units/{apiName}`, `GET /api/v1/units/{apiName}/items` (chống N+1 bằng batch query)
- [ ] **Tuần 4.1**: Xây `TierCalculator` (Hybrid Tier) + API `GET /api/v1/items`, `/items/{apiName}`, `/items/{apiName}/holders`
- [ ] **Tuần 4.2**: API `GET /api/v1/augments` (kèm AVP vòng `2-1 / 3-2 / 4-2` & `bestComps`)
- [ ] **Tuần 4.3**: Unit Tests (JUnit 5 + Mockito) & Controller Tests (`@WebMvcTest`) đạt pass 100%

### Phase 3: Frontend Angular `tft-hub-web` (Tuần 5–7)
- [ ] **Tuần 5**: Setup ExecutionPolicy, Angular CLI, repo `tft-hub-web`, Tailwind Dark Theme, Global Patch/Lang Signals, 4 Shared Components
- [ ] **Tuần 6**: Màn hình `/comps` + Hex Grid 4×7 & Màn hình `/units` (`GRID | LIST`) + Unit Detail Modal (Đồ theo tướng + Đội hình)
- [ ] **Tuần 7**: Màn hình `/items` (`TIER VIEW | STATS TABLE` + Item Detail) & Màn hình `/augments`

### Phase 4: Admin CMS & Spring Security JWT (Tuần 8)
- [ ] **Tuần 8.1**: Backend Spring Security JWT (`ROLE_ADMIN`) + CRUD Comps & Curated Tier override
- [ ] **Tuần 8.2**: Frontend `/admin/login` & Comp Builder trên bàn cờ Hex 4×7

### Phase 5: Tối ưu, Riot API & Deploy (Tuần 9–10)
- [ ] **Tuần 9**: Spring Cache (`@Cacheable`) + *(Tùy chọn)* Riot API Match Crawler (`raw_matches` -> tính AVP/Delta thật)
- [ ] **Tuần 10**: Multi-stage Dockerfile, GitHub Actions CI, Deploy Production & hoàn thiện `README.md`
