# AGENT_PLAN_TV3 – Kế hoạch cho agent thực hiện phần TV3 (Bảo mật và triển khai)

Đồ án IoT – Hệ thống quản lý cấp nước thông minh. Tài liệu này dành cho **AI agent** (Antigravity). Hãy đọc toàn bộ trước khi làm bất cứ việc gì.

---

## 0. Quy tắc làm việc của agent (bắt buộc)

### 0.1 Vai trò

Bạn đang giúp **một thành viên (TV3)** của nhóm 4 người làm phần **bảo mật và triển khai** của đồ án. Người dùng phải tự giải thích được code của mình trong buổi vấn đáp, vì vậy bạn vừa là người làm vừa là người giải thích.

### 0.2 Vòng lặp mỗi bước

Với mỗi bước `S0`, `S1`, ... theo thứ tự:

1. Đọc mục của bước đó. Nếu đã có checkpoint `DONE` trong `docs/agent-progress.md` thì bỏ qua.
2. Làm các việc ghi ở **Agent làm**.
3. Chạy các lệnh ở **Xác minh**. Nếu thất bại, tự sửa tối đa 3 lần; vẫn thất bại thì dừng và báo cáo lỗi kèm nguyên nhân nghi ngờ.
4. Nếu bước có **CHECKPOINT NGƯỜI DÙNG** hoặc **REVIEW GATE**, làm đúng theo mục 0.3 và 0.4.
5. Cập nhật `docs/agent-progress.md` (mục 0.6), commit trên nhánh của bước (`feat/s05-auth`, ...) với thông điệp dạng `feat(backend): ...`.
6. Báo cáo cho người dùng bằng tiếng Việt, **tối đa 8 dòng**: đã làm gì, vì sao làm vậy, kiểm tra ra sao, bước tiếp theo.

### 0.3 CHECKPOINT NGƯỜI DÙNG – khi nào dừng

Mọi việc được đánh dấu `🛑 CHECKPOINT NGƯỜI DÙNG` là việc **bạn không được tự làm hoặc không thể tự làm** (cài phần mềm, thao tác trên GitHub, thiết bị thật, chụp ảnh bằng chứng, viết báo cáo bằng lời của người dùng...). Khi gặp:

1. **Dừng hẳn.** Không làm bước tiếp theo, không tự giả định là người dùng đã xong.
2. In khối sau, điền đầy đủ:

```
🛑 CẦN BẠN THỰC HIỆN — <mã checkpoint, ví dụ CP-S01-A>
Việc cần làm:
  1. ...
  2. ...
Cách báo lại: trả lời "xong <mã>" và dán kết quả của các lệnh/ảnh tôi yêu cầu bên dưới.
Tôi sẽ kiểm tra bằng: <lệnh hoặc file cụ thể>
```

3. Khi người dùng báo xong, **chạy lệnh xác minh** đã nêu. Đạt thì ghi `DONE` vào tiến độ và đi tiếp. Không đạt thì nói rõ lỗi, hướng dẫn cách sửa, rồi chờ người dùng báo lại; **không bỏ qua checkpoint**.
4. Nếu không thể xác minh bằng lệnh (ví dụ ảnh chụp), kiểm tra file có tồn tại đúng tên trong `docs/evidence/` và đọc nội dung nếu mở được; sau đó hỏi người dùng xác nhận.

### 0.4 REVIEW GATE – các bước bảo mật quan trọng

Ở các bước ghi `🔍 REVIEW GATE` (S5, S6, S7, S11, S13): sau khi xong và xác minh, **dừng và chờ người dùng đọc code**. In:

- danh sách file đã tạo/sửa,
- giải thích từng hàm quan trọng bằng 1–2 câu,
- 3 câu hỏi mà giảng viên có thể hỏi về phần này, kèm gợi ý trả lời.

Chỉ tiếp tục khi người dùng trả lời "ok" hoặc yêu cầu sửa. Không commit lên nhánh chính trước khi có "ok".

### 0.5 Những điều không được làm

- Không commit bí mật thật (mật khẩu, JWT secret, khóa riêng, chứng thư). Chỉ commit `.env.example`. File `.env` chỉ ở máy người dùng và nằm trong `.gitignore`.
- Không chạy lệnh phá hủy khi chưa hỏi: `rm -rf` ngoài thư mục build, `docker system prune`, `docker volume rm`, `git push --force`, `git reset --hard`, xóa nhánh.
- Không sửa phần của thành viên khác (xem mục 2), trừ các **stub** đã nêu rõ.
- Không đấu hay hướng dẫn đấu điện 220V; dự án chỉ dùng DC thấp áp.
- Không đoán cấu hình/phiên bản. Nếu không chắc (đặc biệt Mosquitto Dynamic Security), hãy đọc tài liệu đúng phiên bản hoặc chạy thử để xác nhận.
- Không tự cấp thêm quyền hay bỏ qua bước kiểm tra để "cho xong".

### 0.6 File tiến độ `docs/agent-progress.md`

Tạo ở bước S0, cập nhật sau mỗi bước, để có thể tiếp tục ở phiên làm việc khác. Định dạng:

```
# Tiến độ agent – TV3
| Bước | Trạng thái | Ghi chú |
| S0 | DONE | ... |
| S1 | WAITING_USER (CP-S01-A) | chờ người dùng bật branch protection |
| S2 | TODO | |
## Giả định cần đối chiếu khi ghép
- ...
## Việc còn tồn đọng
- ...
```

Trạng thái hợp lệ: `TODO`, `IN_PROGRESS`, `WAITING_USER (<mã>)`, `REVIEW (<mã>)`, `DONE`, `BLOCKED`.

### 0.7 Quy ước kỹ thuật

- Python 3.11+. Tên biến, hàm, file bằng tiếng Anh. Docstring ngắn bằng tiếng Anh. Giải thích cho người dùng bằng tiếng Việt.
- Mỗi hàm giao diện (mục 2.2) phải có docstring nêu tham số, kết quả, lỗi có thể ném.
- Mỗi bước có test tự động nếu có thể (`pytest`). Test đơn vị dùng SQLite trong bộ nhớ; kiểm thử tích hợp chạy trên MySQL trong Docker.

---

## 1. Bối cảnh dự án (đủ để làm độc lập)

**Phạm vi:** 1 node ESP32 (cảm biến siêu âm mực nước, cảm biến lưu lượng, relay + bơm DC). Phần mềm: backend Python FastAPI + SQLAlchemy + paho-mqtt (subscriber MQTT chạy chung tiến trình với API), web dashboard, Mosquitto, MySQL 8, Docker Compose. Triển khai 4 container: `mosquitto`, `backend`, `mysql`, `frontend` (nginx).

### 1.1 Topic MQTT (`deviceId` là username MQTT, ví dụ `wms-001`)

| Topic | Hướng | Ghi chú |
| --- | --- | --- |
| `wms/{deviceId}/telemetry` | thiết bị → server | QoS 1 |
| `wms/{deviceId}/status` | thiết bị → server | QoS 1, retained (LWT, heartbeat) |
| `wms/{deviceId}/cmd` | server → thiết bị | QoS 1, lệnh bơm |
| `wms/{deviceId}/ack` | thiết bị → server | QoS 1 |
| `wms/{deviceId}/config` | server → thiết bị | QoS 1, retained |

ACL thiết bị (mẫu theo username): `pattern write wms/%u/telemetry`, `pattern write wms/%u/status`, `pattern write wms/%u/ack`, `pattern read wms/%u/cmd`, `pattern read wms/%u/config`. Tài khoản `backend` được đọc/ghi `wms/#`.

### 1.2 Vai trò và quyền

Ký hiệu: ✔ được · ✖ không · ◐ chỉ với thiết bị được gán trong `user_devices`.

| Chức năng | Admin | Operator | Viewer |
| --- | --- | --- | --- |
| Quản lý người dùng (tạo, khóa, đổi vai trò) | ✔ | ✖ | ✖ |
| Xem/sửa tài khoản của mình, đổi mật khẩu | ✔ | ✔ | ✔ |
| Đăng ký, cấp credential, thu hồi thiết bị | ✔ | ✖ | ✖ |
| Gán thiết bị cho người dùng và khu vực | ✔ | ✖ | ✖ |
| Xem danh sách và trạng thái thiết bị | ✔ | ◐ | ◐ |
| Xem dữ liệu hiện tại, lịch sử, biểu đồ | ✔ | ◐ | ◐ |
| Xuất dữ liệu CSV | ✔ | ◐ | ✖ |
| Đổi cấu hình và ngưỡng cảnh báo | ✔ | ◐ | ✖ |
| Gửi lệnh bật/tắt bơm | ✔ | ◐ | ✖ |
| Chuyển thiết bị sang Maintenance | ✔ | ◐ | ✖ |
| Xem cảnh báo | ✔ | ◐ | ◐ |
| Xác nhận (ack) và đóng cảnh báo | ✔ | ◐ | ✖ |
| Xem audit log | ✔ | ✖ | ✖ |

Mã lỗi: **401** thiếu/sai/hết hạn token · **403** không đủ quyền · **404** không tồn tại hoặc ngoài phạm vi · **422** dữ liệu sai. Quyền kiểm tra **ở backend**; frontend chỉ ẩn nút cho tiện.

### 1.3 Vòng đời thiết bị

`Registered → Provisioned → Active ⇄ Offline`; `Active → Fault → Maintenance → Active`; `Active/Offline/Maintenance → Decommissioned` (thu hồi, không quay lại). Khi **Provisioned**: backend sinh mật khẩu ngẫu nhiên, **hiển thị đúng 1 lần**, chỉ lưu bản băm, tạo tài khoản MQTT trên broker. Khi **Decommissioned**: xóa tài khoản MQTT, thiết bị không kết nối lại được. Lớp bảo vệ thứ hai bắt buộc: backend chỉ ghi dữ liệu của thiết bị đang Active.

### 1.4 Bảng dữ liệu thuộc TV3

| Bảng | Cột chính |
| --- | --- |
| `users` | id, email (unique), password_hash, full_name, role, status, created_at, **failed_attempts, locked_until** |
| `user_devices` | user_id, device_id |
| `audit_logs` | id, user_id, action, target_type, target_id, ip, detail, created_at |

### 1.5 API thuộc TV3

`POST /api/auth/login` · `POST /api/auth/logout` · `POST /api/auth/change-password` · `GET /api/me` · `GET, POST /api/users` · `PATCH /api/users/{id}` · `GET /api/audit-logs` · `WS /ws` (xác thực JWT). Mật khẩu băm bằng bcrypt hoặc argon2, JWT hạn ngắn, khóa tạm sau 5 lần đăng nhập sai.

---

## 2. Ranh giới trách nhiệm

### 2.1 Phần của TV3 (được làm)

Repo và quy ước; Docker Compose; cấu hình Mosquitto (xác thực, ACL, TLS, thu hồi); module auth, user, RBAC, audit log; xác thực WebSocket; `broker_admin`; kiểm thử bảo mật và F1, F8; sơ đồ triển khai, ma trận quyền, threat model; phần báo cáo về người dùng-phân quyền, bảo mật, triển khai.

### 2.2 Giao diện sẽ bàn giao cho người khác (phải ổn định, có docstring)

| Hàm / module | Dùng bởi |
| --- | --- |
| `get_current_user`, `require_role(*roles)`, `can_access_device(user, device_id)`, `require_device_access` | TV2 (endpoint thiết bị), TV4 (qua API) |
| `audit.log(db, user, action, target_type, target_id, ip, detail)` | TV2 (lệnh bơm, config, provision) |
| `broker_admin.create_client / delete_client / disable_client / enable_client` | TV2 (provision, đổi trạng thái) |
| `is_device_active(device_id)` | TV2 (ingest telemetry) |

### 2.3 Phần của người khác (không sửa)

`firmware/` và `hardware/` (TV1); ingest telemetry, vòng đời thiết bị, lệnh + ack, alert engine, `database/` chính thức (TV2); toàn bộ `frontend/` ngoài trang giữ chỗ (TV4). Những chỗ cần mô phỏng, chỉ tạo **stub tạm** nêu trong từng bước và ghi vào "Giả định cần đối chiếu khi ghép".

---

## 3. Các bước thực hiện

### S0 – Chuẩn bị môi trường

**Agent làm**

- Kiểm tra có đủ: `docker`, `docker compose`, `python` (≥ 3.11), `git`, `openssl`, `mosquitto_pub`, `mosquitto_sub`, `mosquitto_passwd`, `mosquitto_ctrl`, `curl`. Liệt kê cái còn thiếu kèm cách cài cho hệ điều hành của người dùng.
- Tạo `docs/agent-progress.md` theo mục 0.6.

**🛑 CHECKPOINT NGƯỜI DÙNG – CP-S00-A:** cài các công cụ còn thiếu, mở Docker Desktop, đăng nhập GitHub cục bộ (`git config user.name/email`).

**Xác minh:** agent chạy lại từng lệnh `--version`/`--help`; `docker info` không báo lỗi.

### S1 – Khung repo và quy tắc bí mật

**Agent làm**

- Tạo cấu trúc: `README.md`, `firmware/`, `backend/`, `frontend/`, `database/`, `deployment/`, `hardware/`, `tests/`, `docs/` (kèm `docs/diagrams/`, `docs/evidence/`), `.env.example`.
- Viết `.gitignore`: `.env`, `*.pem`, `*.key`, `*.crt` (trừ CA công khai nếu người dùng muốn commit), `__pycache__/`, `.venv/`, `*.log`, dữ liệu container.
- Viết `.env.example` với các biến (không có giá trị thật): `MYSQL_ROOT_PASSWORD`, `MYSQL_DATABASE`, `MYSQL_USER`, `MYSQL_PASSWORD`, `JWT_SECRET`, `JWT_EXPIRE_MINUTES`, `MQTT_HOST`, `MQTT_PORT`, `MQTT_BACKEND_USER`, `MQTT_BACKEND_PASSWORD`, `MQTT_DYNSEC_ADMIN_USER`, `MQTT_DYNSEC_ADMIN_PASSWORD`.
- Viết script `deployment/scripts/make_env.sh` sinh `.env` cục bộ với giá trị ngẫu nhiên (`openssl rand`), chỉ chạy khi `.env` chưa tồn tại.

**🛑 CHECKPOINT NGƯỜI DÙNG – CP-S01-A:** tạo repo trên GitHub (hoặc dùng repo nhóm), cấp quyền, bật **branch protection** cho `main` (yêu cầu Pull Request và ít nhất 1 review), tạo GitHub Projects/Issues nếu nhóm dùng. Agent không thao tác phần này.

**Xác minh**

- `git status` sau khi chạy `make_env.sh` không hiển thị `.env`.
- `git check-ignore -v .env` cho kết quả khớp.
- `grep -rn "JWT_SECRET=" .env.example` không chứa giá trị thật.

### S2 – Docker Compose cơ bản

**Agent làm**

- Viết `deployment/docker-compose.yml` với 4 dịch vụ: `mosquitto`, `mysql` (có `healthcheck` bằng `mysqladmin ping`), `backend` (`depends_on` với điều kiện `service_healthy`), `frontend` (nginx phục vụ một trang tĩnh giữ chỗ).
- Mạng nội bộ chung; chỉ mở cổng cần thiết (MQTT, backend, web); **không** mở cổng MySQL ra máy chủ.
- Volume cho dữ liệu MySQL và Mosquitto.
- `backend/Dockerfile`: lệnh chạy **một worker** `uvicorn app.main:app --host 0.0.0.0 --port 8000 --workers 1` (subscriber MQTT chạy chung tiến trình; nhiều worker sẽ xử lý trùng bản tin). Thêm comment giải thích lý do.
- Tạo tạm `backend/requirements.txt` và `backend/app/main.py` tối thiểu (endpoint `/health`) để container build được.

**Xác minh**

- `docker compose -f deployment/docker-compose.yml config` hợp lệ.
- `docker compose up -d --build` rồi `docker compose ps`: 4 container chạy/healthy.
- `curl -s localhost:8000/health` trả 200.

### S3 – Mosquitto cơ bản (password_file + ACL)

Đây là bản tạm để chạy được trước khi chuyển sang Dynamic Security ở S11.

**Agent làm**

- Viết `deployment/mosquitto/mosquitto.conf`: `allow_anonymous false`, `password_file`, `acl_file`, `persistence true`.
- Tạo file mật khẩu bằng `mosquitto_passwd` cho `wms-001` và `backend` bằng giá trị lấy từ `.env` (không in mật khẩu ra màn hình, không commit file mật khẩu).
- Viết `acl_file` theo mục 1.1.
- Viết script `tests/security/s03_broker_acl.sh` kiểm tra tự động.

**Xác minh** (script tự chạy, in PASS/FAIL từng dòng)

1. `wms-001` publish `wms/wms-001/telemetry` → thành công.
2. `wms-001` publish `wms/khac/telemetry` → bị từ chối hoặc không phân phối.
3. `wms-001` subscribe `wms/#` → không nhận được gì.
4. Kết nối không mật khẩu → bị từ chối.
5. `backend` subscribe `wms/#` nhận được bản tin ở mục 1.

Lưu log vào `docs/evidence/s4-acl-log.txt` (đây là bằng chứng cho S4 của bài kiểm thử bảo mật).

### S4 – Khung backend và model bảo mật

**Agent làm**

- Tạo `backend/app/`: `main.py`, `config.py` (pydantic-settings đọc `.env`), `db.py` (engine `mysql+pymysql`, `SessionLocal`, `Base`, `get_db`), `models/`, `routers/`, `security/`, `services/`.
- Viết model `User`, `UserDevice`, `AuditLog` theo mục 1.4.
- Viết **stub tạm** `models/device_stub.py` (id, device_code, state) chỉ để khóa ngoại của `user_devices` chạy được; ghi vào "Giả định cần đối chiếu".
- Script `database/seed_dev.py` tạo 3 tài khoản mẫu (Admin, Operator, Viewer, mật khẩu ngẫu nhiên in ra **một lần** ở console hoặc lấy từ `.env`) và 2–3 thiết bị giả, gán Operator A và Operator B hai thiết bị khác nhau.
- Thiết lập `pytest` với SQLite trong bộ nhớ.

**Xác minh:** khởi động backend, các bảng xuất hiện trong MySQL (`SHOW TABLES`), `/docs` mở được, `pytest` chạy (dù chưa có test nội dung).

### S5 – Đăng nhập, JWT, mật khẩu 🔍 REVIEW GATE

**Agent làm**

- Băm mật khẩu bằng `bcrypt` hoặc `argon2-cffi` (dùng trực tiếp thư viện; `passlib` có thể lỗi tương thích với bcrypt bản mới).
- `POST /api/auth/login`: kiểm tra email + mật khẩu, trả JWT hạn ngắn (`sub`, `role`, `exp`), dùng PyJWT. Thông báo lỗi chung, không lộ email có tồn tại hay không. Tài khoản `status` bị khóa thì từ chối.
- `GET /api/me`, `POST /api/auth/change-password` (cần mật khẩu cũ, kiểm tra độ mạnh mật khẩu mới), `POST /api/auth/logout`.
- `get_current_user` (`Depends`): đọc `Authorization: Bearer`, trả 401 nếu thiếu/sai chữ ký/hết hạn.
- Không bao giờ ghi mật khẩu hay token vào log.

**Xác minh (pytest + curl)**

- Đăng nhập đúng nhận token; sai mật khẩu nhận lỗi chung.
- Token sai, token hết hạn, không có token đều nhận **401** (bằng chứng S2); lưu kết quả vào `docs/evidence/s2-401.txt`.
- Phản hồi không có trường `password_hash`.

**REVIEW GATE** theo mục 0.4.

### S6 – Khóa tạm sau nhiều lần sai 🔍 REVIEW GATE

**Agent làm**

- Mỗi lần đăng nhập sai tăng `failed_attempts`; đủ 5 lần đặt `locked_until` (ví dụ 15 phút) và từ chối kể cả khi nhập đúng trong thời gian khóa. Đăng nhập đúng thì đặt lại bộ đếm.
- Giới hạn tần suất theo IP cho `/api/auth/login` (`slowapi` hoặc cách tự viết, nêu rõ lựa chọn).

**Xác minh:** test mô phỏng 5 lần sai liên tiếp rồi lần thứ 6 đúng mật khẩu vẫn bị từ chối; sau khi `locked_until` trôi qua (test đổi đồng hồ) đăng nhập lại được. Lưu kết quả vào `docs/evidence/s6-lockout.txt` (bằng chứng S6).

**REVIEW GATE** theo mục 0.4.

### S7 – RBAC dùng chung 🔍 REVIEW GATE

**Agent làm**

- Viết `security/rbac.py` với `require_role(*roles)` (403), `can_access_device(user, device_id)` (Admin luôn được; Operator và Viewer chỉ khi có dòng trong `user_devices`), `require_device_access(device_id)` (404 khi ngoài phạm vi, không tiết lộ thiết bị có tồn tại).
- Mã hóa ma trận quyền mục 1.2 thành **một bảng cấu hình trong code** (hành động → vai trò được phép, và cờ "chỉ thiết bị được gán"), để đối chiếu trực tiếp với sơ đồ ma trận.
- Viết **router giả** `routers/fake_devices.py` (xem thiết bị, gửi lệnh bơm, đổi cấu hình, xem cảnh báo, xuất CSV) chỉ để chứng minh RBAC; đánh dấu rõ "STUB – TV2 sẽ thay" và ghi vào giả định ghép.

**Xác minh:** pytest kiểm tra **từng ô** của ma trận quyền với 3 vai trò và 2 thiết bị gán khác nhau, kết quả 200/403/404 đúng. Operator A gọi thiết bị của B nhận 403 hoặc 404 (bằng chứng S1, lưu `docs/evidence/s1-cross-device.txt`).

**REVIEW GATE** theo mục 0.4.

### S8 – API quản lý người dùng

**Agent làm**

- `GET, POST /api/users`, `PATCH /api/users/{id}` (role, status), API gán/bỏ gán thiết bị cho người dùng; chỉ Admin.
- Validate bằng schema pydantic (email hợp lệ, role thuộc `admin|operator|viewer`); không trả `password_hash`.
- Không cho Admin cuối cùng tự khóa hoặc tự hạ quyền.

**Xác minh:** Operator/Viewer gọi nhận 403; Admin tạo, khóa, đổi vai trò, gán thiết bị thành công; pytest xanh.

### S9 – Audit log

**Agent làm**

- Viết `audit.log(...)` (mục 2.2) và gọi khi: đăng nhập thành công/thất bại, khóa tài khoản, đổi mật khẩu, tạo/khóa người dùng, đổi vai trò, gán thiết bị, thao tác bị từ chối quyền.
- `GET /api/audit-logs` chỉ Admin, lọc theo người dùng và khoảng thời gian, có phân trang. Không ghi dữ liệu nhạy cảm vào `detail`.

**Xác minh:** thực hiện các thao tác trên rồi truy vấn log: đúng người, đúng hành động, đúng IP; Operator gọi `/api/audit-logs` nhận 403.

### S10 – Xác thực WebSocket

**Agent làm**

- `WS /ws`: trình duyệt không gửi được header `Authorization`, nên dùng **bản tin đầu tiên** `{"type":"auth","token":"..."}` trong vòng 5 giây; sai/hết hạn/không gửi thì đóng kết nối. Đóng kết nối khi tài khoản bị khóa.
- Viết hàm lọc sự kiện theo phạm vi dùng `can_access_device`. Thêm bộ phát sự kiện giả định kỳ để thử.

**Xác minh:** kết nối không xác thực bị từ chối; hai người dùng khác phạm vi chỉ nhận sự kiện của mình (viết test WebSocket hoặc script).

### S11 – Quản lý tài khoản thiết bị trên broker (Dynamic Security) 🔍 REVIEW GATE

**Agent làm**

1. Kiểm tra phiên bản Mosquitto trong image đang dùng và **đọc tài liệu đúng phiên bản** về plugin Dynamic Security (đường dẫn `mosquitto_dynamic_security.so`, cách khởi tạo file cấu hình bằng `mosquitto_ctrl dynsec init`, cách dùng `%u` trong ACL của role). Ghi kết luận vào `docs/notes/dynsec.md`.
2. Cấu hình broker nạp plugin; tạo tài khoản quản trị dynsec **tách riêng** với tài khoản `backend`; tạo **role thiết bị** có ACL tương ứng mục 1.1.
3. Viết `services/broker_admin.py` bằng paho-mqtt gửi lệnh JSON tới `$CONTROL/dynamic-security/v1` và đọc phản hồi ở `$CONTROL/dynamic-security/v1/response`: `create_client`, `delete_client`, `disable_client`, `enable_client`. Sinh mật khẩu bằng `secrets.token_urlsafe`, chỉ trả mật khẩu thô **một lần** cho nơi gọi, lưu bản băm.
4. **Phương án dự phòng:** nếu dynsec không dùng được, viết bản thay thế cùng giao diện hàm bằng cách cập nhật `password_file`/`acl_file` rồi cho broker nạp lại (không mount docker socket vào backend). Hỏi người dùng trước khi chọn phương án dự phòng.

**Xác minh (script `tests/security/s11_broker_admin.py`)**

1. `create_client("wms-test", ...)` rồi kết nối bằng tài khoản đó: thành công và ACL đúng.
2. `disable_client` rồi kết nối lại: bị từ chối.
3. `delete_client` khi thiết bị đang kết nối: bị ngắt và không vào lại được (bằng chứng S3 mức broker; lưu `docs/evidence/s3-broker-revoke.txt`).

**REVIEW GATE** theo mục 0.4, kèm giải thích vì sao chọn dynsec thay vì sửa file và khởi động lại container.

### S12 – Chặn thiết bị không hợp lệ ở backend

**Agent làm**

- Viết `is_device_active(device_id)` đọc trạng thái từ model stub (ghi vào giả định ghép).
- Viết subscriber **thử** `tests/tools/guard_probe.py` (không phải ingest thật): bản tin của thiết bị không Active bị bỏ và ghi log.
- Viết bộ kiểm tra dùng lại cho TV2: giới hạn kích thước payload (ví dụ 2 KB) và giới hạn tần suất theo thiết bị.

**Xác minh:** đặt thiết bị ở `Decommissioned`, publish giả, kiểm tra bộ lọc bỏ bản tin; payload quá lớn hoặc quá dồn bị từ chối.

### S13 – TLS cho MQTT 🔍 REVIEW GATE

**Agent làm**

- Viết `deployment/certs/gen_certs.sh` sinh CA riêng và chứng thư máy chủ bằng OpenSSL, với `CN`/`SAN` là địa chỉ hoặc hostname thiết bị sẽ dùng để kết nối (đọc từ biến, hỏi người dùng nếu chưa rõ).
- Cấu hình listener `8883` có TLS trong Mosquitto; để `1883` chỉ dùng khi phát triển và có cờ tắt ở bản cuối.
- Khóa riêng vào volume/thư mục bị `.gitignore`; không commit.
- Chuẩn bị file CA công khai và `docs/notes/tls-for-esp32.md` hướng dẫn TV1 nạp CA vào firmware (`WiFiClientSecure::setCACert`; ESP32 cần đồng bộ giờ qua NTP để xác thực chứng thư).
- Nếu khả thi, cấu hình nginx phục vụ HTTPS bằng chứng thư tự ký.

**🛑 CHECKPOINT NGƯỜI DÙNG – CP-S13-A:**

1. Gửi file CA công khai và `tls-for-esp32.md` cho TV1.
2. Cài Wireshark, bắt gói MQTT qua cổng 8883 trong lúc agent publish thử, chụp ảnh thấy dữ liệu đã mã hóa, lưu `docs/evidence/s-tls-wireshark.png`.
3. (Khi TV1 sẵn sàng) thử kết nối ESP32 thật qua TLS và dán log.

**Xác minh**

- `mosquitto_pub --cafile ca.crt -p 8883 ...` thành công; không có `--cafile` thì thất bại.
- Kiểm tra file ảnh Wireshark tồn tại trong `docs/evidence/`.

**REVIEW GATE** theo mục 0.4.

### S14 – Bộ kiểm thử bảo mật và chức năng của TV3

**Agent làm**

Viết script/pytest trong `tests/` cho từng mục, lưu kết quả vào `docs/evidence/`:

| Mã | Kịch bản | Kỳ vọng |
| --- | --- | --- |
| F1 | Đăng nhập 3 vai trò, thử từng chức năng | Mỗi vai trò chỉ làm được phần của mình |
| S1 | Operator A truy cập thiết bị của B | 403 hoặc 404 |
| S2 | Token sai/hết hạn | 401 |
| S3 / F8 | Thu hồi thiết bị rồi kết nối/publish lại | Broker từ chối, backend không ghi |
| S4 | Thiết bị publish vào topic không được phép | Bị từ chối (ACL) |
| S5 | Payload sai (`level_pct` = 999, thiếu `ts`, chuỗi script) | Bị loại, có log, không lưu (**chạy lại sau khi ghép với ingest của TV2**) |
| S6 | Đăng nhập sai nhiều lần | Bị khóa tạm |

Viết sẵn bộ payload độc hại `tests/security/payloads_s05.json` và script gửi để chạy lại khi ghép.

**🛑 CHECKPOINT NGƯỜI DÙNG – CP-S14-A:** chụp ảnh màn hình cho F1 (cần giao diện của TV4) và các ảnh curl/Postman cho S1, S2, S6 nếu muốn dùng ảnh thay cho file log; đặt vào `docs/evidence/` theo tên mà agent liệt kê. Agent kiểm tra file tồn tại.

**Xác minh:** `pytest tests/` xanh; chạy các script `tests/security/*` đều PASS; chạy `gitleaks detect` (nếu đã cài) không phát hiện bí mật.

### S15 – Sơ đồ, báo cáo và README

**Agent làm (bản NHÁP, người dùng sẽ viết lại bằng lời của mình)**

- Mã Mermaid trong `docs/diagrams/`: sơ đồ triển khai (4 container, mạng, cổng, volume); ma trận vai trò-quyền (sinh từ bảng cấu hình ở S7); mô hình mối đe dọa (bảng mối đe dọa → biện pháp → mã kiểm thử); sequence đăng nhập → gọi API có JWT.
- Dàn ý kèm gợi ý nội dung cho 3 mục báo cáo: **Người dùng và phân quyền**, **Bảo mật và quyền riêng tư** (dữ liệu cá nhân thu thập: email, họ tên, IP trong audit log; không thu vị trí, hình ảnh, âm thanh, dữ liệu sức khỏe; chính sách lưu telemetry), **Triển khai**. Ghi rõ lý do chọn từng công nghệ (bcrypt/argon2, JWT, Dynamic Security, Docker Compose).
- Viết phần README của TV3: cài đặt, sao chép `.env.example`, sinh chứng thư, `docker compose up`, tài khoản mẫu, và mục **Công cụ AI đã dùng** (ghi Antigravity cùng phạm vi sử dụng).

**🛑 CHECKPOINT NGƯỜI DÙNG – CP-S15-A:**

1. Xuất các sơ đồ Mermaid sang PNG/SVG (ví dụ qua mermaid.live) và lưu vào `docs/diagrams/`, hoặc vẽ lại bằng draw.io.
2. Đọc, hiểu và **viết lại phần báo cáo bằng lời của mình**; đối chiếu với code. Agent không nộp bản nháp thay người dùng.
3. Tập trả lời các câu hỏi vấn đáp gợi ý của agent về từng phần.

**Xác minh:** các file PNG/SVG có trong `docs/diagrams/`; README chạy theo từng bước trên máy sạch (agent mô phỏng bằng cách clone vào thư mục mới và chạy `docker compose up`).

### S16 – Chuẩn bị ghép nối và hoàn tất

**Agent làm**

- Kiểm tra mọi hàm giao diện ở mục 2.2 đã có docstring và test.
- Chạy toàn bộ bộ kiểm thử bước S14 lần cuối; clone sạch và `docker compose up` từ đầu.
- Hoàn thiện danh sách ghép trong `docs/agent-progress.md`, mục "Giả định cần đối chiếu khi ghép":
  1. Thay `device_stub` bằng model `Device` thật của TV2 (giữ cột `id`, `state`).
  2. Gắn `require_role` và `require_device_access` vào endpoint thật, bỏ `fake_devices.py`.
  3. Gọi `audit.log` ở thao tác của TV2 (lệnh bơm, cấu hình, provision).
  4. Gọi `broker_admin` trong luồng provision và đổi trạng thái của TV2.
  5. Cho ingest của TV2 gọi `is_device_active` và các bộ giới hạn payload/tần suất.
  6. Chạy lại F1, S1–S6 trên hệ thống đã ghép; bổ sung F8 với thiết bị thật; diễn tập bước demo 1, 9, 10.
- Soạn mô tả Pull Request: phạm vi, cách kiểm tra, giả định ghép, người review đề xuất (TV2 cho backend/bảo mật, TV4 cho xác thực/API).

**🛑 CHECKPOINT NGƯỜI DÙNG – CP-S16-A:**

1. Tự mở Pull Request và xin review từ TV2 và TV4 (agent không đẩy lên nhánh chính).
2. Khi nhóm đã đóng băng code: chạy lần cuối `gitleaks`, tạo tag `v1.0.0`, cấp quyền cho giảng viên nếu repo private.

**Xác minh:** `git log` cho thấy mỗi bước một nhánh/commit rõ nghĩa; không có bí mật trong lịch sử; `docs/agent-progress.md` hiển thị mọi bước `DONE`.

---

## 4. Tổng hợp các điểm dừng

| Mã | Bước | Người dùng làm |
| --- | --- | --- |
| CP-S00-A | S0 | Cài công cụ, mở Docker, cấu hình Git |
| CP-S01-A | S1 | Tạo repo, bật branch protection, cấp quyền |
| REVIEW | S5, S6, S7, S11, S13 | Đọc code, trả lời "ok" |
| CP-S13-A | S13 | Gửi CA cho TV1, chụp Wireshark, thử ESP32 thật |
| CP-S14-A | S14 | Chụp ảnh bằng chứng F1, S1, S2, S6 |
| CP-S15-A | S15 | Xuất sơ đồ, tự viết báo cáo, luyện vấn đáp |
| CP-S16-A | S16 | Mở Pull Request, chạy gitleaks, tag `v1.0.0` |

## 5. Câu mở đầu để bắt đầu

Khi bắt đầu một phiên mới, đọc `docs/agent-progress.md`, báo cho người dùng bước đang dở và việc tiếp theo, rồi tiếp tục theo mục 0.2. Nếu chưa có file tiến độ, bắt đầu từ **S0**.
