# Tiến độ agent – TV3

| Bước | Trạng thái | Ghi chú |
| --- | --- | --- |
| S0 | DONE | Đã xác minh đủ công cụ (`docker`, `docker compose`, `python` 3.13.3, `git`, `openssl`, `mosquitto_*`, `curl`), cấu hình Git và khởi tạo repo |
| S1 | WAITING_USER (CP-S01-A) | Đã tạo khung repo, `.gitignore`, `.env.example`, `make_env.sh` và xác minh ẩn `.env`; chờ người dùng tạo repo GitHub & bật branch protection cho `main` |
| S2 | TODO | Docker Compose cơ bản |
| S3 | TODO | Mosquitto cơ bản (password_file + ACL) |
| S4 | TODO | Khung backend và model bảo mật |
| S5 | TODO | Đăng nhập, JWT, mật khẩu (REVIEW GATE) |
| S6 | TODO | Khóa tạm sau nhiều lần sai (REVIEW GATE) |
| S7 | TODO | RBAC dùng chung (REVIEW GATE) |
| S8 | TODO | API quản lý người dùng |
| S9 | TODO | Audit log |
| S10 | TODO | Xác thực WebSocket |
| S11 | TODO | Quản lý tài khoản thiết bị trên broker – Dynamic Security (REVIEW GATE) |
| S12 | TODO | Chặn thiết bị không hợp lệ ở backend |
| S13 | TODO | TLS cho MQTT (CP-S13-A + REVIEW GATE) |
| S14 | TODO | Bộ kiểm thử bảo mật và chức năng của TV3 (CP-S14-A) |
| S15 | TODO | Sơ đồ, báo cáo và README (CP-S15-A) |
| S16 | TODO | Chuẩn bị ghép nối và hoàn tất (CP-S16-A) |

## Giả định cần đối chiếu khi ghép
- TV3 sở hữu 3 bảng `users`, `user_devices`, `audit_logs`; bảng `users` có thêm `failed_attempts`, `locked_until`.
- Bảng `devices` (của TV2) có khóa chính `id`, `device_code` và cột `state` (`Registered`, `Provisioned`, `Active`, `Offline`, `Fault`, `Maintenance`, `Decommissioned`).
- Backend chạy 1 worker (`--workers 1`) vì MQTT subscriber chạy chung tiến trình với API.
- WebSocket `/ws` xác thực bằng bản tin JSON đầu tiên `{"type":"auth","token":"..."}` trong vòng 5 giây.
- Tên biến môi trường thống nhất theo `.env.example` của TV3.
- Tài khoản MQTT `backend` có quyền đọc/ghi `wms/#`; tài khoản quản trị Dynamic Security tách riêng.
- Mã lỗi HTTP: `401` thiếu/sai/hết hạn token, `403` không đủ quyền, `404` không tồn tại hoặc ngoài phạm vi thiết bị được gán, `422` dữ liệu sai.

## Việc còn tồn đọng
- Hoàn tất CP-S01-A (tạo repo trên GitHub, đẩy nhánh `main` và bật branch protection yêu cầu Pull Request + ít nhất 1 review).
