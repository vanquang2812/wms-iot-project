# Hệ thống quản lý cấp nước thông minh (Smart Water Management System – WMS)

Đồ án học phần IoT và ứng dụng – Nhóm 4 sinh viên (Phân hệ TV3: Bảo mật và triển khai).

## Cấu trúc thư mục

- `firmware/`: Mã nguồn cho node ESP32 (TV1)
- `hardware/`: Sơ đồ đấu nối và danh mục linh kiện (TV1)
- `backend/`: Dịch vụ FastAPI + SQLAlchemy + paho-mqtt (TV2, TV3)
- `database/`: Schema, migration và script tạo dữ liệu mẫu (TV2, TV3)
- `frontend/`: Giao diện Web Dashboard (TV4)
- `deployment/`: Docker Compose, cấu hình Mosquitto, script sinh biến môi trường và chứng thư TLS (TV3)
- `tests/`: Bộ kiểm thử đơn vị, tích hợp và bảo mật (TV1, TV2, TV3, TV4)
- `docs/`: Tài liệu thiết kế, sơ đồ (`docs/diagrams/`) và bằng chứng kiểm thử (`docs/evidence/`)

## Khởi tạo môi trường cục bộ

1. Sinh file `.env` cục bộ từ `.env.example`:
   ```bash
   bash deployment/scripts/make_env.sh
   ```
2. Khởi chạy các dịch vụ bằng Docker Compose (xem chi tiết từ bước S2):
   ```bash
   docker compose -f deployment/docker-compose.yml up -d --build
   ```
