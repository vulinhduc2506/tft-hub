# Sổ tay Học tập & Kinh nghiệm (Learning Log)

> **Mục đích**: Sau mỗi buổi code/học, tự ghi ngắn gọn **3–5 dòng bằng chính ngôn từ của bạn** (tự diễn đạt lại thay vì copy lý thuyết). Sau 10 tuần, đây sẽ là tài liệu ôn tập phỏng vấn Java Intern/Fresher quý giá nhất của chính bạn.

---

## Mẫu ghi chép (Giữ nguyên form này cho các tuần sau)

```markdown
### [Ngày - Tuần X.Y] Chủ đề
- **Đã làm được gì**: ...
- **Bản chất hiểu được (Why / Under the hood)**: ...
- **Lỗi đã gặp (hoặc tự tạo) & Cách xử lý**: ...
- **Câu trả lời Active Recall cho buổi tới**: ...
```

---

## Nhật ký thực tế

### [2026-11-22 - Phase 0] Chốt nghiệp vụ, Kiến trúc & Thiết kế Database
- **Đã làm được gì**: Phân tích UX/Data của 3 trang tham khảo (`tactics.tools/items`, `tftacademy.com/tierlist/items`, `tactics.tools/units`), chốt nghiệp vụ kết hợp thành 4 module (`/comps`, `/units`, `/items`, `/augments`), chốt ERD 14 bảng và hợp đồng API `v1`.
- **Bản chất hiểu được (Why / Under the hood)**:
  - **Modular Monolith vs Microservices**: Dữ liệu TFT liên kết chéo quá dày (1 bảng `champion_item_stats` phục vụ cả chiều Tướng -> Đồ lẫn Đồ -> Top Holders). Chọn Modular Monolith chia package rõ (`catalog`, `comp`, `stats`, `sync`, `admin`) giúp join nhanh trong 1 DB, không bị over-engineering như Microservices.
  - **Hybrid Tier**: `COALESCE(curated_tier, calculated_tier)` vừa tự tính tier theo số liệu khi ra patch mới, vừa giữ khả năng chấm tay kiểu chuyên gia như TFT Academy.
  - **Song ngữ VI + EN**: Sync đồng thời 2 file CommunityDragon theo `api_name` vào cùng 1 bảng giúp tìm kiếm cả tiếng Việt lẫn tiếng Anh mà không phải nhân đôi dữ liệu.
- **Lỗi / Lưu ý môi trường đã phát hiện**: PowerShell trên máy đang khóa chạy script `.ps1` (`npm.ps1` bị chặn nếu không gọi `npm.cmd`); PostgreSQL 16 service local đang chạy sẵn nên không cần cài Docker trước khi bắt đầu Backend.
- **Việc tiếp theo**: Bắt đầu **Tuần 1** — Kết nối PostgreSQL 16 local và viết Flyway migration `V1` & `V2`.
