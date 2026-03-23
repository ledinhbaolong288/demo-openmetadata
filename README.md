# Demo OpenMetadata (Docker + PostgreSQL)

## ✅ Mục tiêu
Chạy **OpenMetadata** trên Docker với **PostgreSQL** làm database, và dùng dữ liệu mẫu trong thư mục `Data_src/` (có `orders.json`, `users.json`).

---

## 🛠️ Cấu trúc thư mục
- `docker-compose.yml` – khởi chạy PostgreSQL + OpenMetadata + ingestion
- `Data_src/` – chứa dữ liệu mẫu (được mount vào container)
- `ingestion/ingestion.yml` – recipe để OpenMetadata quét file JSON và ingest metadata

---

## ▶️ Bước 1: Khởi động bằng Docker Compose
Từ thư mục `demo-openmetadata/` chạy:

```bash
docker compose up
```

> 🔎 `openmetadata` sẽ chạy trên `http://localhost:8585`

---

## 🔍 Bước 2: Truy cập UI OpenMetadata
Mở trình duyệt và truy cập:

- `http://localhost:8585` để vào OpenMetadata UI

> 🧠 Mặc định cấu hình dùng `no-auth`, nên không cần đăng nhập.

---

## ▶️ Bước 3: Ingest metadata từ folder `Data_src/`
Khi `docker compose up` chạy, service `ingestion` sẽ chạy 1 lần theo cấu hình `ingestion/ingestion.yml`, rồi tự thoát. Nó sẽ cố gắng quét các file `*.json` trong `Data_src/` và tạo metadata trong OpenMetadata.

Nếu muốn chạy lại ingestion (sau khi thêm/đổi file):

```bash
docker compose run --rm ingestion
```

---

## ✅ Điều chỉnh / mở rộng
- Nếu cần kết nối thêm database (Postgres, MySQL, Snowflake, v.v.), dùng UI OpenMetadata để thêm **Data Source**.
- Nếu muốn ingest khác (CSV, database, API), chỉnh `ingestion/ingestion.yml` theo docs của OpenMetadata.

---

## 🔥 Dọn dẹp
Để dừng và xóa container + volume:

```bash
docker compose down -v
```

---

# 📋 HƯỚNG DẪN CHI TIẾT CẤU HÌNH DOCKER-COMPOSE

## 1️⃣ TỔNG QUAN KIẾN TRÚC

Hệ thống bao gồm 5 dịch vụ chính chạy trên cùng một mạng Docker:

```
docker-compose.yml
├── volumes (3)
│   ├── ingestion-volume-dag-airflow
│   ├── ingestion-volume-dags
│   └── es-data
├── services (5)
│   ├── postgresql (Database)
│   ├── elasticsearch (Search Engine)
│   ├── execute-migrate-all (Migration)
│   ├── openmetadata-server (API Server)
│   └── ingestion (Airflow Ingestion)
└── networks
    └── app_net (172.16.240.0/24)
```

---

## 2️⃣ CẤU HÌNH CHI TIẾT TỪNG SERVICE

### 📦 Service 1: PostgreSQL (Database)

**Chức năng:** Lưu trữ metadata OpenMetadata

**Thông tin container:**
- Container name: `openmetadata_postgresql`
- Image: `docker.getcollate.io/openmetadata/postgresql:1.12.0`
- Restart policy: `always` (tự động khởi động lại khi crash)

**Environment Variables (Cấu hình Database):**
```yaml
POSTGRES_USER: postgres                    # Tên user database
POSTGRES_PASSWORD: postgres                # Mật khẩu user
POSTGRES_DB: openmetadata_db               # Tên database
```

**Port Mapping:**
```yaml
expose: 5432     # Cho phép các service khác kết nối
ports:
  - "5432:5432"  # Cho phép kết nối từ host machine
```

**Volume (Lưu trữ dữ liệu):**
```yaml
./docker-volume/db-data-postgres:/var/lib/postgresql/data
```
📌 Dữ liệu PostgreSQL được lưu vĩnh viễn trên host machine

**Command:**
```bash
"--work_mem=10MB"  # Cấu hình bộ nhớ làm việc cho queries
```

**Health Check:**
```bash
test: psql -U postgres -tAc 'select 1' -d openmetadata_db
interval: 15s      # Kiểm tra mỗi 15 giây
timeout: 10s       # Chờ tối đa 10 giây
retries: 10        # Thử 10 lần trước khi xem là down
```

---

### 🔍 Service 2: Elasticsearch (Search Engine)

**Chức năng:** Lập chỉ mục (indexing) và tìm kiếm metadata

**Thông tin container:**
- Container name: `openmetadata_elasticsearch`
- Image: `docker.elastic.co/elasticsearch/elasticsearch:9.3.0`

**Environment Variables:**
```yaml
discovery.type: single-node                           # Chạy single node (không cluster)
ES_JAVA_OPTS: -Xms1024m -Xmx1024m                    # Cấu hình RAM: min 1GB, max 1GB
xpack.security.enabled: false                         # Tắt bảo mật X-Pack (dev environment)
```

**Port Mapping:**
```yaml
ports:
  - "9200:9200"  # REST API port (HTTP)
  - "9300:9300"  # Node communication port
```

**Volume:**
```yaml
es-data:/usr/share/elasticsearch/data  # Lưu dữ liệu persistent
```

**Health Check:**
```bash
curl -s http://localhost:9200/_cluster/health?pretty 
# Kiểm tra status là green hoặc yellow
```

---

### 🔄 Service 3: execute-migrate-all (Database Migration)

**Chức năng:** Tạo schema và migrate database lần đầu

**Thông tin container:**
- Container name: `execute_migrate_all`
- Image: `docker.getcollate.io/openmetadata/server:1.12.0`
- Command: `./bootstrap/openmetadata-ops.sh migrate`

**Environment Variables (Chính):**

**1. Cấu hình Cluster:**
```yaml
OPENMETADATA_CLUSTER_NAME: openmetadata    # Tên cluster
SERVER_PORT: 8585                          # Port API server
SERVER_ADMIN_PORT: 8586                    # Port Admin API
LOG_LEVEL: INFO                            # Mức detail log
MIGRATION_LIMIT_PARAM: 1200                # Batch size migration
```

**2. Cấu hình Authentication & Authorization:**
```yaml
AUTHORIZER_CLASS_NAME: org.openmetadata.service.security.DefaultAuthorizer
AUTHORIZER_REQUEST_FILTER: org.openmetadata.service.security.JwtFilter
AUTHORIZER_ADMIN_PRINCIPALS: [admin]       # Admins được cấp quyền
AUTHORIZER_ALLOWED_REGISTRATION_DOMAIN: ["all"]  # Cho phép tất cả domain đăng ký
AUTHORIZER_INGESTION_PRINCIPALS: [ingestion-bot] # Chủ thể cho ingestion
AUTHORIZER_PRINCIPAL_DOMAIN: open-metadata.org
AUTHORIZER_ALLOWED_DOMAINS: []
AUTHORIZER_ENFORCE_PRINCIPAL_DOMAIN: false
```

**3. Cấu hình Authentication Provider:**
```yaml
AUTHENTICATION_PROVIDER: basic             # Sử dụng basic auth (user/password)
AUTHENTICATION_ENABLE_SELF_SIGNUP: true    # Cho phép đăng ký tài khoản
AUTHENTICATION_CLIENT_TYPE: public

# OIDC Configuration (Google, Azure, etc.) - Optional
AUTHENTICATION_AUTHORITY: https://accounts.google.com
OIDC_SCOPE: openid email profile
OIDC_RESPONSE_TYPE: code
OIDC_CALLBACK: http://localhost:8585/callback
```

**4. JWT Configuration:**
```yaml
RSA_PUBLIC_KEY_FILE_PATH: ./conf/public_key.der
RSA_PRIVATE_KEY_FILE_PATH: ./conf/private_key.der
JWT_ISSUER: open-metadata.org
JWT_KEY_ID: Gb389a-9f76-gdjs-a92j-0242bk94356
```

**5. Database Configuration:**
```yaml
DB_DRIVER_CLASS: org.postgresql.Driver
DB_SCHEME: postgresql
DB_HOST: postgresql              # Tên service PostgreSQL
DB_PORT: 5432
DB_USER: postgres
DB_USER_PASSWORD: postgres
DB_PARAMS: allowPublicKeyRetrieval=true&useSSL=false&serverTimezone=UTC
OM_DATABASE: openmetadata_db
```

**6. Elasticsearch Configuration:**
```yaml
ELASTICSEARCH_HOST: elasticsearch      # Tên service Elasticsearch
ELASTICSEARCH_PORT: 9200
ELASTICSEARCH_SCHEME: http
ELASTICSEARCH_USER: ""
ELASTICSEARCH_PASSWORD: ""
SEARCH_TYPE: elasticsearch
ELASTICSEARCH_CONNECTION_TIMEOUT_SECS: 5
ELASTICSEARCH_SOCKET_TIMEOUT_SECS: 60
ELASTICSEARCH_BATCH_SIZE: 100
```

**7. Pipeline Service (Airflow Integration):**
```yaml
PIPELINE_SERVICE_CLIENT_ENDPOINT: http://ingestion:8080
PIPELINE_SERVICE_CLIENT_ENABLED: true
PIPELINE_SERVICE_CLIENT_CLASS_NAME: org.openmetadata.service.clients.pipeline.airflow.AirflowRESTClient
AIRFLOW_USERNAME: admin
AIRFLOW_PASSWORD: admin
FERNET_KEY: jJ/9sz0g0OHxsfxOoSfdFdmk3ysNmPRnH3TUAbz3IHA=
```

**8. Secret Manager Configuration:**
```yaml
SECRET_MANAGER: db               # Lưu secrets trong database
# AWS Secrets Manager (Optional)
OM_SM_REGION: ""
OM_SM_ACCESS_KEY_ID: ""
# Azure Key Vault (Optional)
OM_SM_VAULT_NAME: ""
OM_SM_CLIENT_ID: ""
```

**9. Email Configuration (Optional):**
```yaml
AUTHORIZER_ENABLE_SMTP: false
OM_EMAIL_ENTITY: OpenMetadata
SMTP_SERVER_ENDPOINT: ""
SMTP_SERVER_PORT: ""
SMTP_SERVER_USERNAME: ""
SMTP_SERVER_PWD: ""
```

**10. Web Security Headers:**
```yaml
WEB_CONF_HSTS_ENABLED: false           # HTTP Strict Transport Security
WEB_CONF_FRAME_OPTION_ENABLED: false   # X-Frame-Options header
WEB_CONF_XSS_PROTECTION_ENABLED: false # X-XSS-Protection header
WEB_CONF_XSS_CSP_ENABLED: false        # Content Security Policy
```

**Dependencies:**
```yaml
depends_on:
  elasticsearch:
    condition: service_healthy
  postgresql:
    condition: service_healthy
```
⚠️ Service này chỉ chạy khi Elasticsearch và PostgreSQL đã healthy

---

### 🚀 Service 4: openmetadata-server (API Server)

**Chức năng:** Chạy OpenMetadata API server và Web UI

**Thông tin container:**
- Container name: `openmetadata_server`
- Image: `docker.getcollate.io/openmetadata/server:1.12.0`
- Restart policy: `always`

**Port Mapping:**
```yaml
expose: [8585, 8586]     # Cho phép internal communication
ports:
  - "8585:8585"  # Web UI & API (http://localhost:8585)
  - "8586:8586"  # Admin API
```

**Environment Variables:**
Giống hệt `execute-migrate-all` service (xem mục 3️⃣)

**Health Check:**
```bash
test: wget -q --spider http://localhost:8586/healthcheck
# Kiểm tra admin API health endpoint mỗi 30 giây (default)
```

**Dependencies:**
```yaml
depends_on:
  elasticsearch:
    condition: service_healthy
  postgresql:
    condition: service_healthy
  execute-migrate-all:
    condition: service_completed_successfully  # Chỉ chạy sau khi migration hoàn tất
```

---

### 🔄 Service 5: ingestion (Airflow Ingestion Service)

**Chức năng:** Chạy Apache Airflow để thực thi metadata ingestion workflows

**Thông tin container:**
- Container name: `openmetadata_ingestion`
- Image: `docker.getcollate.io/openmetadata/ingestion:1.12.0`

**Airflow Environment Variables:**
```yaml
AIRFLOW__API__AUTH_BACKENDS: airflow.api.auth.backend.basic_auth,airflow.api.auth.backend.session
# Cấu hình xác thực API Airflow

AIRFLOW__CORE__EXECUTOR: LocalExecutor
# Chạy DAGs tuần tự (không distributed)

AIRFLOW__OPENMETADATA_AIRFLOW_APIS__DAG_GENERATED_CONFIGS: /opt/airflow/dag_generated_configs
# Thư mục chứa DAGs được generate từ OpenMetadata
```

**Database Configuration (cho Airflow storage):**
```yaml
DB_HOST: postgresql              # Cùng PostgreSQL với OpenMetadata
DB_PORT: 5432
DB_SCHEME: postgresql+psycopg2   # Driver cụ thể cho Airflow
AIRFLOW_DB: openmetadata_db      # Dùng chung database
DB_USER: postgres
DB_PASSWORD: postgres
DB_PROPERTIES: ""                # Tùy chọn kết nối (vd: sslmode=require)
```

**Port Mapping:**
```yaml
expose: 8080
ports:
  - "8080:8080"  # Airflow UI (http://localhost:8080)
```

**Entry Point & Command:**
```yaml
entrypoint: /bin/bash
command: /opt/airflow/ingestion_dependency.sh
# Script này setup Airflow, DAGs, và chạy
```

**Volumes:**
```yaml
ingestion-volume-dag-airflow:/opt/airflow/dag_generated_configs
# DAGs được generate từ OpenMetadata

ingestion-volume-dags:/opt/airflow/dags
# Thư mục DAGs chính

ingestion-volume-tmp:/tmp
# Thư mục tạm
```

**Dependencies:**
```yaml
depends_on:
  elasticsearch: condition: service_started
  postgresql: condition: service_healthy
  openmetadata-server: condition: service_started
```

---

## 3️⃣ VOLUMES (Lưu trữ dữ liệu persistent)

```yaml
volumes:
  ingestion-volume-dag-airflow:    # DAG configs từ OpenMetadata UI
  ingestion-volume-dags:           # Airflow DAG files
  ingestion-volume-tmp:            # Temporary files
  es-data:                         # Elasticsearch data
```

**Volume mapping:**
```yaml
./docker-volume/db-data-postgres:/var/lib/postgresql/data
# PostgreSQL data lưu vào host machine (persistent)

es-data:/usr/share/elasticsearch/data
# Elasticsearch data (managed volume)
```

---

## 4️⃣ NETWORK CONFIGURATION

```yaml
networks:
  app_net:
    ipam:
      driver: default
      config:
        - subnet: "172.16.240.0/24"  # IP range: 172.16.240.1 - 172.16.240.254
```

**DNS Name Resolution (Internal):**
- `postgresql:5432` → PostgreSQL service
- `elasticsearch:9200` → Elasticsearch service
- `openmetadata-server:8585` → OpenMetadata API
- `ingestion:8080` → Airflow UI

---

## 5️⃣ FLOW KHỞI ĐỘNG

```
1. Start PostgreSQL
   └─> Khởi tạo database openmetadata_db
   └─> Health check: psql test
      
2. Start Elasticsearch
   └─> Khởi tạo cluster single-node
   └─> Health check: curl cluster health
   
3. Start execute-migrate-all (chỉ sau khi 1,2 healthy)
   └─> Chạy database migration
   └─> Tạo schema & tables
   └─> Container tự thoát khi xong
   
4. Start openmetadata-server (chỉ sau khi 1,2,3 done)
   └─> Load cấu hình từ environment
   └─> Khởi động Spring Boot application
   └─> Health check: wget /healthcheck
   └─> UI có sẵn tại localhost:8585
   
5. Start ingestion (chỉ sau khi 1,2,4 started)
   └─> Khởi tạo Airflow database
   └─> Setup executor & scheduler
   └─> Chạy ingestion_dependency.sh
   └─> Airflow UI sẵn tại localhost:8080
```

---

## 6️⃣ CÀI ĐẶT & CHẠY CHI TIẾT

### Bước 1: Clone/Setup thư mục
```bash
# Tạo thư mục
mkdir -p demo-openmetadata
cd demo-openmetadata

# Copy docker-compose.yml vào thư mục này
# (hoặc download từ GitHub OpenMetadata)
```

### Bước 2: Tạo thư mục data
```bash
mkdir -p Data_src
mkdir -p docker-volume/db-data-postgres

# Copy dữ liệu sample
cp your_data/orders.json ./Data_src/
cp your_data/users.json ./Data_src/
```

### Bước 3: Kiểm tra cấu hình environment (optional)
```bash
# File .env (nếu có)
# Định nghĩa các biến environment tùy chỉnh
# OPENMETADATA_CLUSTER_NAME=my-cluster
# SERVER_PORT=8585
# LOG_LEVEL=DEBUG
```

### Bước 4: Build & khởi động
```bash
# Kiểm tra version Docker Compose
docker compose version

# Build images (nếu cần)
docker compose build

# Khởi động tất cả services
docker compose up

# Chạy ở background
docker compose up -d

# Xem logs
docker compose logs -f

# Xem logs của 1 service
docker compose logs -f openmetadata-server
docker compose logs -f postgresql
docker compose logs -f elasticsearch
```

### Bước 5: Kiểm tra health status
```bash
# Xem status tất cả containers
docker compose ps

# Kiểm tra health:
# STATUS sẽ show (healthy) hoặc (unhealthy)
```

### Bước 6: Truy cập các service

| Service | URL | Credentials |
|---------|-----|-------------|
| OpenMetadata UI | http://localhost:8585 | admin / admin |
| OpenMetadata API | http://localhost:8585/api/v1 | - |
| Airflow UI | http://localhost:8080 | admin / admin |
| Elasticsearch | http://localhost:9200 | - |
| PostgreSQL | localhost:5432 | postgres / postgres |

---

## 7️⃣ TROUBLESHOOTING

### ❌ Service khiêung không lên (unhealthy)

**PostgreSQL không start:**
```bash
# Xem logs
docker compose logs postgresql

# Kiểm tra port 5432 có bị chiếm
netstat -an | grep 5432

# Xóa volume cũ & restart
docker compose down -v
docker compose up
```

**Elasticsearch không start:**
```bash
# Xem logs
docker compose logs elasticsearch

# Kiểm tra RAM đủ (cần ít nhất 2GB)
free -h

# Tăng virtual memory (Linux)
sysctl -w vm.max_map_count=262144
```

**OpenMetadata không lên:**
```bash
# Chờ execute-migrate-all hoàn tất
docker compose logs execute-migrate-all

# Kiểm tra database connectivity
docker compose exec postgresql psql -U postgres -d openmetadata_db -c "SELECT 1"

# Kiểm tra Elasticsearch
docker compose exec openmetadata-server curl -s http://elasticsearch:9200/_cluster/health
```

### ❌ Port bị chiếm

```bash
# Thay đổi port trong docker-compose.yml
# ports:
#   - "9200:8585"  # OpenMetadata chạy trên port 9200
```

### ❌ Dung lượng disk đầy

```bash
# Xóa volumes cũ
docker volume prune

# Xóa images cũ
docker image prune
```

---

## 8️⃣ CUSTOMIZATION EXAMPLES

### Thay đổi mật khẩu database
```yaml
# docker-compose.yml
postgresql:
  environment:
    POSTGRES_PASSWORD: your_secure_password  # Đổi password

execute-migrate-all & openmetadata-server:
  environment:
    DB_USER_PASSWORD: your_secure_password   # Đổi password
```

### Thay đổi heap memory
```yaml
execute-migrate-all & openmetadata-server:
  environment:
    OPENMETADATA_HEAP_OPTS: -Xmx2G -Xms2G   # Tăng lên 2GB
```

### Bật Authentication (OIDC - Google)
```yaml
openmetadata-server:
  environment:
    AUTHENTICATION_PROVIDER: oidc
    OIDC_TYPE: google
    OIDC_CLIENT_ID: your-client-id
    OIDC_CLIENT_SECRET: your-client-secret
    OIDC_DISCOVERY_URI: https://accounts.google.com/.well-known/openid-configuration
```

### Bật HTTPS
```yaml
openmetadata-server:
  ports:
    - "8585:8585"
  # Cần setup reverse proxy (nginx/Traefik) hoặc mTLS
```

---

## 9️⃣ ENVIRONMENT VARIABLES TÓMLƯỢC

| Variable | Mồng | Ý nghĩa |
|----------|------|---------|
| `OPENMETADATA_CLUSTER_NAME` | openmetadata | Tên cluster |
| `SERVER_PORT` | 8585 | Port OpenMetadata API |
| `LOG_LEVEL` | INFO | DEBUG/INFO/WARN/ERROR |
| `DB_HOST` | postgresql | Host database |
| `DB_PORT` | 5432 | Port database |
| `DB_USER` | postgres | User database |
| `DB_USER_PASSWORD` | postgres | Password database |
| `ELASTICSEARCH_HOST` | elasticsearch | Host Elasticsearch |
| `ELASTICSEARCH_PORT` | 9200 | Port Elasticsearch |
| `AUTHENTICATION_PROVIDER` | basic | basic/oidc/ldap/saml |
| `OPENMETADATA_HEAP_OPTS` | -Xmx1G -Xms1G | Java heap memory |
| `AIRFLOW_USERNAME` | admin | Airflow username |
| `AIRFLOW_PASSWORD` | admin | Airflow password |

---

## 🔟 RESOURCES & DOCUMENTATION

- 📖 [OpenMetadata Documentation](https://docs.open-metadata.org/)
- 🐳 [Docker Compose Reference](https://docs.docker.com/compose/compose-file/)
- 🐘 [PostgreSQL Docker Image](https://hub.docker.com/_/postgres)
- 🔍 [Elasticsearch Docker Image](https://www.docker.elastic.co/)
- 🌬️ [Apache Airflow Documentation](https://airflow.apache.org/docs/)
