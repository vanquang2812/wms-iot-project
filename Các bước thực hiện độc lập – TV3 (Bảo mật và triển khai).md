# Các bước thực hiện độc lập – TV3 (Bảo mật và triển khai)

Đồ án IoT – Hệ thống quản lý cấp nước thông minh · Backend: Python FastAPI + SQLAlchemy + paho-mqtt

## Cách làm độc lập

TV3 làm việc trên một bản backend riêng chỉ chứa phần của mình, chạy cùng Mosquitto và MySQL cục bộ. Những thứ thuộc người khác (thiết bị, ingest telemetry, vòng đời thiết bị, giao diện) được thay bằng bản giả đơn giản. Mỗi bước dưới đây có **việc làm** và **cách kiểm tra xong**; kiểm tra xong là chuyển bước, không chờ ai.

Quy ước chung:

- Mọi bí mật (mật khẩu DB, khóa ký JWT, mật khẩu broker) nằm trong `.env`, không commit; chỉ commit `.env.example`.
- Mỗi bước là một nhánh `feat/...` và một Pull Request nhỏ.
- Chỗ nào phải đoán về phần của người khác thì ghi vào mục "Giả định cần đối chiếu khi ghép" ở cuối tài liệu, rồi làm tiếp.

## Bước 0. Chuẩn bị môi trường

**Việc làm**

- Cài Docker Desktop (hoặc Docker Engine + Compose), Python 3.11+, Git, `mosquitto-clients` (để có `mosquitto_pub`, `mosquitto_sub`, `mosquitto_ctrl`), Postman hoặc curl, OpenSSL.
- Tạo tài khoản GitHub riêng, vào repo nhóm hoặc tạo repo tạm của riêng mình để làm trước.

**Kiểm tra xong:** `docker compose version`, `python --version`, `mosquitto_pub --help` đều chạy.

## Bước 1. Khung repo và quy tắc bí mật

**Việc làm**

- Dựng cấu trúc thư mục bắt buộc: `README.md`, `firmware/`, `backend/`, `frontend/`, `database/`, `deployment/`, `hardware/`, `tests/`, `docs/`, `.env.example`.
- Viết `.gitignore` gồm `.env`, `*.pem`, `*.key`, `__pycache__/`, `.venv/`, thư mục dữ liệu của container.
- Viết `.env.example` với tên biến (không có giá trị thật): thông tin MySQL, `JWT_SECRET`, `JWT_EXPIRE_MINUTES`, `MQTT_HOST`, `MQTT_PORT`, `MQTT_BACKEND_USER`, `MQTT_BACKEND_PASSWORD`, `MQTT_DYNSEC_ADMIN_USER`, `MQTT_DYNSEC_ADMIN_PASSWORD`.
- Bật bảo vệ nhánh `main`, quy ước nhánh `feat/...`, commit theo dạng `feat(backend): ...`.

**Kiểm tra xong:** clone repo mới về, chỉ có `.env.example`, không có file bí mật nào.

## Bước 2. Docker Compose cơ bản

**Việc làm**

- Viết `deployment/docker-compose.yml` với 4 dịch vụ: `mosquitto`, `mysql`, `backend`, `frontend` (nginx). Ở giai đoạn này `frontend` chỉ phục vụ một trang tĩnh giữ chỗ.
- Đặt mạng nội bộ chung, chỉ mở cổng cần thiết ra máy chủ (1883 hoặc 8883 cho MQTT, cổng backend, cổng web). MySQL không mở ra ngoài.
- Thêm `healthcheck` cho mysql và `depends_on` có điều kiện để backend chỉ khởi động sau khi DB sẵn sàng.
- Volume cho dữ liệu MySQL và dữ liệu Mosquitto.
- Viết `backend/Dockerfile`. Lệnh chạy dùng **một worker**: `uvicorn app.main:app --host 0.0.0.0 --port 8000 --workers 1`, vì subscriber MQTT chạy chung tiến trình với API, nhiều worker sẽ xử lý bản tin trùng.

**Kiểm tra xong:** `docker compose up` đưa cả 4 container lên, `docker compose ps` đều ở trạng thái chạy/healthy.

## Bước 3. Mosquitto cơ bản (password\_file + ACL)

Đây là bản tạm để mọi thứ chạy được trước khi chuyển sang Dynamic Security ở Bước 11.

**Việc làm**

- Viết `deployment/mosquitto/mosquitto.conf`: tắt `allow_anonymous`, dùng `password_file` và `acl_file`, bật persistence.
- Tạo mật khẩu bằng `mosquitto_passwd` cho 2 tài khoản: `wms-001` (thiết bị mẫu) và `backend`.
- Viết `acl_file`: thiết bị theo mẫu `pattern write wms/%u/telemetry`, `pattern write wms/%u/status`, `pattern write wms/%u/ack`, `pattern read wms/%u/cmd`, `pattern read wms/%u/config`; tài khoản `backend` đọc/ghi `wms/#`.

**Kiểm tra xong (dùng `mosquitto_pub/sub` làm thiết bị giả)**

- `wms-001` publish vào `wms/wms-001/telemetry`: thành công.
- `wms-001` publish vào `wms/khac/telemetry` hoặc subscribe `wms/#`: bị từ chối.
- Kết nối không có mật khẩu: bị từ chối.
- `backend` subscribe `wms/#` nhận được bản tin của thiết bị giả.

Đây cũng là bằng chứng sơ bộ cho S4.

## Bước 4. Khung backend riêng và model bảo mật

**Việc làm**

- Tạo dự án FastAPI trong `backend/app/` với: `main.py`, `config.py` (đọc `.env` bằng pydantic-settings), `db.py` (engine, `SessionLocal`, `Base`, `get_db`), thư mục `models/`, `routers/`, `security/`.
- Viết model SQLAlchemy cho phần của TV3:
  - `users`: id, email (unique), password\_hash, full\_name, role, status, created\_at, **failed\_attempts, locked\_until**.
  - `user_devices`: user\_id, device\_id.
  - `audit_logs`: id, user\_id, action, target\_type, target\_id, ip, detail, created\_at.
- Vì bảng `devices` thuộc người khác, tạo một model `Device` **tạm** rất đơn giản (id, device\_code, state) chỉ để khóa ngoại của `user_devices` chạy được.
- Tạo bảng bằng `Base.metadata.create_all` hoặc Alembic; thêm script tạo 3 tài khoản mẫu (Admin, Operator, Viewer) và vài thiết bị giả.

**Kiểm tra xong:** khởi động backend, các bảng xuất hiện trong MySQL, `/docs` mở được.

## Bước 5. Đăng nhập, JWT, mật khẩu

**Việc làm**

- Băm mật khẩu bằng bcrypt (hoặc argon2); không bao giờ lưu hay log mật khẩu thô.
- `POST /api/auth/login`: kiểm tra email + mật khẩu, trả JWT có thời hạn ngắn (chứa `sub`, `role`, `exp`). Thông báo lỗi chung chung, không tiết lộ email có tồn tại hay không.
- `GET /api/me`, `POST /api/auth/change-password` (yêu cầu mật khẩu cũ, mật khẩu mới đủ mạnh), `POST /api/auth/logout` (client bỏ token; nếu muốn thu hồi thật thì thêm danh sách token bị vô hiệu).
- Viết `get_current_user` làm `Depends`: đọc token từ header `Authorization: Bearer`, trả 401 nếu thiếu, sai chữ ký hoặc hết hạn; trả 401/403 nếu tài khoản bị khóa (`status`).

**Kiểm tra xong:** đăng nhập đúng nhận token; token sai/hết hạn nhận 401 (S2); `/api/me` trả đúng người dùng.

## Bước 6. Khóa tạm sau nhiều lần sai (S6)

**Việc làm**

- Mỗi lần đăng nhập sai tăng `failed_attempts`; đủ 5 lần thì đặt `locked_until` (ví dụ 15 phút) và từ chối kể cả khi nhập đúng trong thời gian khóa.
- Đăng nhập đúng thì đặt lại bộ đếm.
- Thêm giới hạn tần suất theo IP cho endpoint login (thư viện `slowapi` hoặc tự đếm).
- Ghi sự kiện khóa tài khoản vào audit log (sau Bước 9).

**Kiểm tra xong:** đăng nhập sai 5 lần liên tiếp thì bị khóa, chụp màn hình; sau thời gian khóa đăng nhập lại được.

## Bước 7. RBAC dùng chung

**Việc làm**

- Viết module `security/rbac.py` với 3 thành phần dùng ở mọi endpoint:
  - `require_role(*roles)`: chặn theo vai trò, trả 403.
  - `can_access_device(user, device_id)`: Admin luôn được; Operator và Viewer chỉ khi có dòng trong `user_devices`.
  - `require_device_access(device_id)`: dùng `can_access_device`, trả 404 khi thiết bị ngoài phạm vi (không tiết lộ thiết bị có tồn tại).
- Mã hóa ma trận quyền (mục 8 của tài liệu khung) thành một bảng cấu hình trong code để dễ đối chiếu với sơ đồ.
- Viết **router giả** `routers/fake_devices.py` gồm vài endpoint mô phỏng (xem thiết bị, gửi lệnh bơm, đổi cấu hình, xem cảnh báo, xuất CSV) chỉ để chứng minh RBAC hoạt động; sau này TV2 thay bằng endpoint thật và gắn các `Depends` của bạn.

**Kiểm tra xong (viết bằng pytest + TestClient):** với 3 vai trò và 2 thiết bị gán khác nhau, mọi ô trong ma trận quyền cho đúng kết quả 200/403/404. Operator A gọi thiết bị của B nhận 403 hoặc 404 (S1).

## Bước 8. API quản lý người dùng

**Việc làm**

- `GET, POST /api/users` và `PATCH /api/users/{id}` (đổi role, status), chỉ Admin.
- API gán thiết bị cho người dùng (thêm/xóa dòng `user_devices`), chỉ Admin.
- Validate đầu vào bằng schema pydantic (email hợp lệ, role thuộc danh sách cho phép); không bao giờ trả `password_hash` ra ngoài.
- Không cho Admin tự khóa hoặc tự hạ quyền chính mình nếu đó là Admin cuối cùng.

**Kiểm tra xong:** Operator/Viewer gọi các API này nhận 403; Admin tạo, khóa, đổi vai trò, gán thiết bị thành công.

## Bước 9. Audit log

**Việc làm**

- Viết hàm dùng chung `audit.log(db, user, action, target_type, target_id, ip, detail)`.
- Ghi log cho: đăng nhập thành công/thất bại, khóa tài khoản, đổi mật khẩu, tạo/khóa người dùng, đổi vai trò, gán thiết bị, và các thao tác bị từ chối quyền.
- `GET /api/audit-logs` chỉ Admin, có lọc theo người dùng và khoảng thời gian, có phân trang.
- Không ghi dữ liệu nhạy cảm (mật khẩu, token) vào cột `detail`.

**Kiểm tra xong:** thực hiện các thao tác trên rồi xem log đúng người, đúng hành động, đúng IP.

## Bước 10. Xác thực WebSocket

**Việc làm**

- Tạo endpoint `WS /ws` nhận JWT (trình duyệt không gửi được header, nên chọn một cách và ghi lại: token trong query string, hoặc bản tin đầu tiên sau khi kết nối). Khuyến nghị bản tin đầu tiên để token không nằm trong log truy cập.
- Từ chối kết nối nếu token sai/hết hạn; đóng kết nối khi tài khoản bị khóa.
- Viết sẵn hàm lọc sự kiện theo phạm vi: người dùng chỉ nhận sự kiện của thiết bị mình được xem (dùng `can_access_device`).
- Dùng một sự kiện giả định kỳ để thử.

**Kiểm tra xong:** WebSocket không token bị từ chối; hai người dùng khác phạm vi chỉ thấy sự kiện của mình (thử bằng client WebSocket đơn giản hoặc Postman).

## Bước 11. Quản lý tài khoản thiết bị trên broker (Dynamic Security)

**Việc làm**

- Kiểm tra phiên bản Mosquitto đang dùng có plugin Dynamic Security (`mosquitto_dynamic_security.so`) và đọc tài liệu của nó, đặc biệt việc dùng `%u` trong ACL.
- Cấu hình broker nạp plugin, tạo tài khoản quản trị dynsec (tách biệt với tài khoản `backend`), khai báo một **role thiết bị** có các ACL theo `wms/%u/...` như Bước 3.
- Viết module `broker_admin.py` bằng paho-mqtt, gửi lệnh JSON tới `$CONTROL/dynamic-security/v1` và đọc phản hồi ở `$CONTROL/dynamic-security/v1/response`, với các hàm:
  - `create_client(username, password)`: tạo client và gán role thiết bị.
  - `delete_client(username)`: xóa client (broker ngắt kết nối ngay).
  - `disable_client(username)` và `enable_client(username)`: phục vụ trạng thái Maintenance.
- Viết hàm sinh mật khẩu ngẫu nhiên đủ mạnh (`secrets.token_urlsafe`) và hàm băm bản lưu; chỉ trả mật khẩu thô **một lần** cho nơi gọi.
- Nếu Dynamic Security gặp trở ngại, giữ phương án dự phòng: cập nhật `password_file` và `acl_file` rồi cho broker nạp lại. Giữ cùng giao diện hàm để thay thế mà không đổi code phía gọi.

**Kiểm tra xong (bằng script thử, chưa cần TV2):**

1. `create_client` rồi dùng `mosquitto_pub` kết nối bằng tài khoản đó: thành công.
2. `disable_client` rồi kết nối lại: bị từ chối.
3. `delete_client` khi đang kết nối: kết nối bị ngắt và không vào lại được (S3 ở mức broker).

## Bước 12. Chặn thiết bị không hợp lệ ở backend

**Việc làm**

- Viết hàm giả `is_device_active(device_id)` đọc trạng thái từ model `Device` tạm; đây là "lớp bảo vệ thứ hai" mà phần ingest của TV2 sẽ gọi trước khi lưu dữ liệu.
- Viết thử một subscriber nhỏ (chỉ để kiểm tra, không phải ingest thật) dùng hàm này: bản tin của thiết bị không Active bị bỏ và ghi log.
- Thêm hai chốt chống lạm dụng để TV2 dùng lại: giới hạn kích thước payload và giới hạn tần suất theo thiết bị.

**Kiểm tra xong:** thiết bị chuyển sang Decommissioned vẫn publish được (giả lập broker chưa chặn) nhưng backend không lưu gì.

## Bước 13. TLS cho MQTT (và HTTPS nếu làm được)

**Việc làm**

- Viết script `deployment/certs/gen_certs.sh` sinh CA riêng và chứng thư máy chủ bằng OpenSSL. Đặt đúng tên máy (CN/SAN) là địa chỉ hoặc hostname mà thiết bị sẽ dùng để kết nối.
- Cấu hình listener `8883` có TLS trong Mosquitto; giữ 1883 chỉ cho môi trường phát triển và tắt ở bản cuối.
- Đưa khóa riêng vào volume, không vào repo; commit script chứ không commit khóa/chứng thư (`*.pem`, `*.key` đã nằm trong `.gitignore`).
- Chuẩn bị file CA công khai và một đoạn hướng dẫn ngắn để TV1 nạp vào firmware (ESP32 cần có giờ đúng qua NTP thì mới xác thực chứng thư được).
- Nếu thời gian cho phép, cho nginx phục vụ HTTPS bằng chứng thư tự ký.

**Kiểm tra xong:** `mosquitto_pub --cafile ...` qua 8883 thành công; kết nối không có CA bị từ chối; Wireshark chỉ thấy dữ liệu đã mã hóa.

## Bước 14. Bộ kiểm thử bảo mật và chức năng của TV3

Viết thành script/pytest trong `tests/` và lưu kết quả kèm bằng chứng.

| Mã | Kịch bản | Kỳ vọng | Bằng chứng |
| --- | --- | --- | --- |
| F1 | Đăng nhập 3 vai trò, thử từng chức năng | Mỗi vai trò chỉ làm được phần mình | Ảnh chụp/kết quả pytest |
| S1 | Operator A truy cập thiết bị của B | 403 hoặc 404 | Ảnh curl/Postman |
| S2 | Token sai hoặc hết hạn | 401 | Ảnh chụp |
| S3 / F8 | Thu hồi thiết bị rồi kết nối/publish lại | Broker từ chối, backend không ghi | Log broker + DB |
| S4 | Thiết bị publish vào topic không được phép | Bị từ chối (ACL) | Log `mosquitto_pub` |
| S5 | Payload sai (`level_pct` = 999, thiếu `ts`, chuỗi script) | Bị loại, có log, không lưu | Log backend |
| S6 | Đăng nhập sai nhiều lần | Bị khóa tạm | Ảnh chụp |

Với S5, phần kiểm tra dữ liệu bản tin do ingest của TV2 thực hiện; bạn viết sẵn bộ payload độc hại và script gửi để chạy lại sau khi ghép.

Bổ sung: chạy `gitleaks` trên repo để chứng minh không lộ bí mật.

## Bước 15. Sơ đồ và tài liệu của TV3

**Sơ đồ** (draw.io hoặc mermaid.live, xuất PNG/SVG vào `docs/diagrams/`)

- Sơ đồ triển khai: 4 container, mạng, cổng, volume.
- Ma trận vai trò và quyền (lấy từ bảng cấu hình ở Bước 7).
- Mô hình mối đe dọa: bảng mối đe dọa, biện pháp, mã kiểm thử tương ứng.
- Sequence đăng nhập → gọi API có JWT, và (phối hợp TV2 sau) sequence onboarding/thu hồi thiết bị.

**Phần báo cáo**

- Người dùng và phân quyền: vai trò, ma trận, cách kiểm quyền ở backend.
- Bảo mật và quyền riêng tư: threat model, kết quả S1–S6, dữ liệu cá nhân thu thập (email, họ tên, IP trong audit log), biện pháp bảo vệ, chính sách lưu telemetry.
- Triển khai: Docker Compose, TLS, hướng dẫn chạy từ đầu.
- Ghi rõ lý do chọn từng công nghệ ở phần mình: bcrypt/argon2, JWT, Dynamic Security, Docker Compose.

**README (phần của TV3):** cài đặt, sao chép `.env.example` thành `.env`, sinh chứng thư, chạy `docker compose up`, tài khoản mẫu.

## Bước 16. Chuẩn bị ghép nối

**Việc làm**

- Đảm bảo các hàm giao diện sau ổn định và có docstring, để người khác dùng không cần đọc code bên trong: `get_current_user`, `require_role`, `require_device_access`, `can_access_device`, `audit.log`, `broker_admin.create_client/delete_client/disable_client/enable_client`, `is_device_active`.
- Chạy toàn bộ bộ kiểm thử Bước 14 trên bản riêng của mình một lượt cuối.
- Chạy `docker compose up` từ một bản clone sạch để chắc không thiếu file hay biến môi trường.
- Tạo Pull Request, nhờ người cùng cặp review (TV2 cho backend, TV4 cho xác thực và API).

Khi ghép, các việc thường phải làm:

1. Thay model `Device` tạm bằng model thật của TV2, giữ nguyên tên cột `id`, `state`.
2. Gắn `require_device_access` và `require_role` vào các endpoint thật, bỏ router giả.
3. Gọi `audit.log` ở các thao tác của TV2 (lệnh bơm, cấu hình, provision).
4. Gọi `broker_admin` trong luồng `provision` và `state` của TV2.
5. Cho ingest của TV2 gọi `is_device_active`, giới hạn payload và tần suất.
6. Chạy lại F1, S1–S6 trên hệ thống đã ghép, bổ sung F8 với thiết bị thật và hoàn thiện bước demo 1, 9, 10.

## Giả định cần đối chiếu khi ghép

Khi làm độc lập, các điểm sau là **giả định của TV3**; nếu người khác làm khác thì sửa lúc ghép:

- TV3 sở hữu 3 bảng `users`, `user_devices`, `audit_logs`; `users` có thêm `failed_attempts`, `locked_until`.
- Bảng `devices` có khóa chính `id` và cột `state` với các giá trị theo vòng đời (Registered, Provisioned, Active, Offline, Fault, Maintenance, Decommissioned).
- Backend dùng một tiến trình (`--workers 1`) và subscriber MQTT chạy chung với API.
- WebSocket xác thực bằng JWT gửi trong bản tin đầu tiên.
- Tên biến môi trường như trong `.env.example` của TV3.
- Tài khoản MQTT `backend` có quyền đọc/ghi `wms/#`; tài khoản quản trị Dynamic Security tách riêng.
- Mã lỗi: 401 token sai/thiếu, 403 không đủ quyền, 404 ngoài phạm vi hoặc không tồn tại, 422 dữ liệu sai.
