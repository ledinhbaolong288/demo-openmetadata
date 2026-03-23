# 📚 OpenMetadata: Hướng Dẫn Chi Tiết

---

# 🎯 PHẦN 1: GIỚI THIỆU VỀ OPENMETADATA

## 1.1 OpenMetadata là gì?

**OpenMetadata** là một nền tảng quản lý metadata mã nguồn mở (Open Source Metadata Management Platform) được thiết kế để:
- **Quản lý toàn bộ metadata** của các dữ liệu trong tổ chức
- **Khám phá và lập chỉ mục** dữ liệu từ nhiều nguồn khác nhau
- **Cộng tác** giữa các team và cải thiện Data Governance
- **Tự động hóa** việc thu thập metadata thông qua Ingestion Pipeline

### 📦 Thành phần chính của OpenMetadata:
```
OpenMetadata
├── Server (API + UI)           # Giao diện web & API để quản lý metadata
├── Connector/Ingestion         # Kết nối & lấy metadata từ các nguồn
├── Metadata Database           # PostgreSQL lưu trữ metadata
├── Search Engine               # Elasticsearch tìm kiếm dữ liệu
└── Airflow Integration         # Tự động hóa ingestion workflows
```

---

## 1.2 Chức năng chính của OpenMetadata

| Chức năng | Mô tả |
|-----------|-------|
| **Data Discovery** | Tìm kiếm và khám phá dữ liệu trong tổ chức |
| **Metadata Collection** | Thu thập metadata từ databases, data warehouses, data lakes |
| **Data Catalog** | Xây dựng catalog tập trung cho tất cả dữ liệu |
| **Data Lineage** | Theo dõi nguồn gốc và luồng dữ liệu |
| **Collaboration** | Gắn tag, comment, ownership cho dataset |
| **Data Governance** | Quản lý policies, glossaries, standards |
| **Automated Ingestion** | Chạy ingestion định kỳ qua Airflow DAGs |
| **API-First** | RESTful API để tích hợp với hệ thống khác |

---

## 1.3 Cách hoạt động của OpenMetadata

### 📊 Kiến trúc tổng thể:

```
┌─────────────────────────────────────────────────────────────┐
│                    OpenMetadata System                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌────────────────────────────────────────────────────────┐ │
│  │         Web UI & REST API (Server)                     │ │
│  │  - Giao diện để xem, quản lý metadata                  │ │
│  │  - API endpoints để integrate với hệ thống khác        │ │
│  └──────────────────────────────────────────────────────  ┘ │
│                          ▲                                  │
│                          │                                  │ 
│  ┌──────────────────────────────────────────────────────┐   │
│  │    Metadata Database (PostgreSQL)                    │   │
│  │  - Lưu trữ tất cả metadata                           │   │
│  │  - Lưu trữ user, permissions, tags                   │   │
│  └──────────────────────────────────────────────────────┘   │
│                          ▲                                  │
│                          │                                  │
│  ┌──────────────────────────────────────────────────────┐   │
│  │    Search Engine (Elasticsearch)                     │   │
│  │  - Indexing metadata để tìm kiếm nhanh               │   │
│  │  - Full-text search across datasets                  │   │
│  └──────────────────────────────────────────────────────┘   │
│                          ▲                                  │
│                          │                                  │
│  ┌──────────────────────────────────────────────────────┐   │
│  │         Ingestion Service (Airflow)                  │   │
│  │  - DAGs để thu thập metadata từ nguồn                │   │
│  │  - Chạy định kỳ hoặc theo trigger                    │   │
│  └──────────────────────────────────────────────────────┘   │
│                          ▲ ▲ ▲ ▲                            │
│              ┌───────────┘ │ │ └───────────┐                │
│              │             │ │             │                │
└──────────────┼─────────────┼─┼─────────────┼─────────────   ┘
               │             │ │             │
       ┌───────▼── ┐  ┌───────▼──┐  ┌────────▼────┐  
       │PostgreSQL │  │ Redshift │  │  Oracle DB  │  
       └───────────┘  └──────────┘  └─────────────┘  
               ▲             ▲             ▲
       ┌───────────────────────────────────────┐
       │         Data Sources                  │
       │  (được connect qua Connectors)        │
       └───────────────────────────────────────┘
```

### 🔄 Quy trình Ingestion Metadata:

```
1. Connector kết nối đến Data Source (DB, DW, Lake, etc.)
   ↓
2. Thu thập metadata: tables, columns, types, descriptions
   ↓
3. Fetch lineage information (nếu có)
   ↓
4. Fetch profiling data (row count, column stats - tuỳ chọn)
   ↓
5. Lưu metadata vào PostgreSQL Database
   ↓
6. Index metadata vào Elasticsearch
   ↓
7. UI & Search Engine hiển thị dữ liệu cho users
```

---

## 1.4 Lợi ích của OpenMetadata

✅ **Centralized Metadata Management** - Tất cả metadata ở 1 nơi
✅ **Quick Data Discovery** - Tìm kiếm dữ liệu nhanh chóng
✅ **Data Lineage Tracking** - Theo dõi luồng dữ liệu từ source đến destination
✅ **Team Collaboration** - Chia sẻ kiến thức qua tags, descriptions
✅ **Automated Workflows** - Ingestion tự động qua Airflow
✅ **Open Source** - Free, customizable, large community
✅ **Multi-Source Support** - Kết nối 50+ loại data sources
✅ **Scalable** - Hỗ trợ từ small teams đến enterprise

---

---

# 🐳 PHẦN 2: HƯỚNG DẪN CHẠY OPENMETADATA VỚI DOCKER

## 2.1 Chuẩn bị môi trường

### ⚙️ Yêu cầu hệ thống:
- **Docker & Docker Compose** (phiên bản mới nhất)
- **RAM**: ít nhất 8GB (khuyến nghị 16GB)
- **Disk**: ít nhất 20GB
- **CPU**: 4 cores (khuyến nghị 8 cores)

### 📥 Kiểm tra cài đặt:
```bash
docker --version
docker compose version
```

---

## 2.2 Cấu trúc Docker Compose

### 🏗️ 5 Services chính:

```yaml
services:
  postgresql:              # Database lưu metadata
  elasticsearch:           # Search engine
  execute-migrate-all:     # Migration service
  openmetadata-server:     # API Server + UI
  ingestion:              # Airflow ingestion
```

### 📑 Cấu hình chi tiết từng Service:

---

### 📦 **Service 1: PostgreSQL (Database)**

**Chức năng:** Lưu trữ tất cả metadata, users, configurations

```yaml
postgresql:
  image: docker.getcollate.io/openmetadata/postgresql:1.12.0
  container_name: openmetadata_postgresql
  restart: always
  environment:
    POSTGRES_USER:
    POSTGRES_PASSWORD:
    POSTGRES_DB: openmetadata_db
  ports:
    - "5432:5432"
  volumes:
    - ./docker-volume/db-data-postgres:/var/lib/postgresql/data
  healthcheck:
    test: psql -U postgres -tAc 'select 1' -d openmetadata_db
    interval: 15s
    timeout: 10s
    retries: 10
```

**Cấu hình:**
- `POSTGRES_USER`: Tên user connect database (mặc định: postgres)
- `POSTGRES_PASSWORD`: Mật khẩu (mặc định: postgres) **⚠️ Đổi trong production**
- `POSTGRES_DB`: Tên database (mặc định: openmetadata_db)
- `Port 5432`: Port PostgreSQL (để localhost và containers đều kết nối được)

---

### 🔍 **Service 2: Elasticsearch (Search Engine)**

**Chức năng:** Indexing & searching metadata (cực nhanh)

```yaml
elasticsearch:
  image: docker.elastic.co/elasticsearch/elasticsearch:9.3.0
  container_name: openmetadata_elasticsearch
  environment:
    discovery.type: single-node
    ES_JAVA_OPTS: -Xms1024m -Xmx1024m
    xpack.security.enabled: false
  ports:
    - "9200:9200"
    - "9300:9300"
  volumes:
    - es-data:/usr/share/elasticsearch/data
  healthcheck:
    test: curl -s http://localhost:9200/_cluster/health | grep -q '"status":"green\|yellow"'
    interval: 30s
    timeout: 10s
    retries: 5
```

**Cấu hình:**
- `discovery.type: single-node` - Chạy 1 node (không cluster)
- `ES_JAVA_OPTS` - Memory: min 1GB, max 1GB (tùy chỉnh theo hệ thống)
- `xpack.security.enabled: false` - Tắt bảo mật (dev environment)

---

### 🔄 **Service 3: execute-migrate-all (Database Migration)**

**Chức náng:** Tạo schema, tables, indexes trong PostgreSQL lần đầu

```yaml
execute-migrate-all:
  image: docker.getcollate.io/openmetadata/server:1.12.0
  container_name: execute_migrate_all
  command: ./bootstrap/openmetadata-ops.sh migrate
  environment:
    OPENMETADATA_CLUSTER_NAME: openmetadata
    SERVER_PORT: 8585
    LOG_LEVEL: INFO
    DB_DRIVER_CLASS: org.postgresql.Driver
    DB_SCHEME: postgresql
    DB_HOST: postgresql
    DB_PORT: 5432
    DB_USER:
    DB_USER_PASSWORD:
    OM_DATABASE: openmetadata_db
    ELASTICSEARCH_HOST: elasticsearch
    ELASTICSEARCH_PORT: 9200
  depends_on:
    postgresql:
      condition: service_healthy
    elasticsearch:
      condition: service_healthy
```

**What it does:**
1. Connect tới PostgreSQL
2. Tạo tất cả tables, indexes
3. Setup initial data
4. Tự động thoát khi xong

---

### 🚀 **Service 4: openmetadata-server (API Server + UI)**

**Chức năng:** Web UI & REST API để quản lý metadata

```yaml
openmetadata-server:
  image: docker.getcollate.io/openmetadata/server:1.12.0
  container_name: openmetadata_server
  restart: always
  ports:
    - "8585:8585"    # UI + API
    - "8586:8586"    # Admin API
  environment:
    OPENMETADATA_CLUSTER_NAME: openmetadata
    SERVER_PORT: 8585
    SERVER_ADMIN_PORT: 8586
    LOG_LEVEL: INFO
    
    # Database
    DB_DRIVER_CLASS: org.postgresql.Driver
    DB_SCHEME: postgresql
    DB_HOST: postgresql
    DB_PORT: 5432
    DB_USER:
    DB_USER_PASSWORD:
    OM_DATABASE: openmetadata_db
    
    # Elasticsearch
    ELASTICSEARCH_HOST: elasticsearch
    ELASTICSEARCH_PORT: 9200
    
    # Airflow Integration
    PIPELINE_SERVICE_CLIENT_ENDPOINT: http://ingestion:8080
    PIPELINE_SERVICE_CLIENT_ENABLED: true
    AIRFLOW_USERNAME:
    AIRFLOW_PASSWORD:
    
    # Authentication (Basic auth mặc định)
    AUTHENTICATION_PROVIDER: basic
    AUTHENTICATION_ENABLE_SELF_SIGNUP: true
    
    # Authorization
    AUTHORIZER_CLASS_NAME: org.openmetadata.service.security.DefaultAuthorizer
    AUTHORIZER_ADMIN_PRINCIPALS: "[admin]"
    AUTHORIZER_INGESTION_PRINCIPALS: "[ingestion-bot]"
  
  depends_on:
    postgresql:
      condition: service_healthy
    elasticsearch:
      condition: service_healthy
    execute-migrate-all:
      condition: service_completed_successfully
  
  healthcheck:
    test: wget -q --spider http://localhost:8586/healthcheck
    interval: 30s
    timeout: 10s
    retries: 5
```

**Access Point:**
- UI: http://localhost:8585
- API: http://localhost:8585/api/v1
![alt text](image.png)
---

### 🔄 **Service 5: ingestion (Airflow Ingestion)**

**Chức năng:** Chạy Apache Airflow để tự động hóa metadata collection

```yaml
ingestion:
  image: docker.getcollate.io/openmetadata/ingestion:1.12.0
  container_name: openmetadata_ingestion
  restart: always
  ports:
    - "8080:8080"    # Airflow UI
  environment:
    # Airflow Config
    AIRFLOW__API__AUTH_BACKENDS: airflow.api.auth.backend.basic_auth,airflow.api.auth.backend.session
    AIRFLOW__CORE__EXECUTOR: LocalExecutor
    AIRFLOW__CORE__SQL_ALCHEMY_CONN: postgresql+psycopg2://postgres:postgres@postgresql:5432/openmetadata_db
    
    # Database cho Airflow metadata
    DB_HOST: postgresql
    DB_PORT: 5432
    DB_SCHEME: postgresql+psycopg2
    DB_USER: postgres
    DB_PASSWORD: postgres
    AIRFLOW_DB: openmetadata_db
    
    # Airflow Credentials
    AIRFLOW_USERNAME:
    AIRFLOW_PASSWORD:
    _AIRFLOW_WWW_USER_CREATE_ADMIN: true
  
  volumes:
    - ingestion-volume-dag-airflow:/opt/airflow/dag_generated_configs
    - ingestion-volume-dags:/opt/airflow/dags
  
  entrypoint: /bin/bash
  command: /opt/airflow/ingestion_dependency.sh
  
  depends_on:
    postgresql:
      condition: service_healthy
    elasticsearch:
      condition: service_started
    openmetadata-server:
      condition: service_started
```

**Access Point:**
- Airflow UI: http://localhost:8080
![alt text](image-1.png)
---

## 2.3 Hướng dẫn chạy Docker Compose

### ▶️ Khởi động hệ thống:

```bash
# 1. Tạo thư mục cần thiết
mkdir -p docker-volume/db-data-postgres
mkdir -p Data_src

# 2. Khởi động tất cả services
docker compose up

# 3. Hoặc chạy background
docker compose up -d

# 4. Xem logs
docker compose logs -f

# 5. Xem logs của 1 service
docker compose logs -f openmetadata-server
docker compose logs -f ingestion
```

### ⏳ Thời gian khởi động:

- PostgreSQL: 10-15 giây
- Elasticsearch: 15-20 giây
- execute-migrate-all: 30-60 giây
- openmetadata-server: 20-30 giây (chờ migration xong)
- ingestion: 30-60 giây (setup Airflow)

**Tổng cộng: ~2-3 phút**

### ✅ Kiểm tra status:

```bash
docker compose ps

# Output:
NAME                             STATUS
openmetadata_postgresql          Up 2 minutes (healthy)
openmetadata_elasticsearch       Up 2 minutes (healthy)
execute_migrate_all              Exited (0)
openmetadata_server              Up 1 minute (healthy)
openmetadata_ingestion           Up 1 minute
```

---

## 2.4 Dừng & Làm sạch:

```bash
# Dừng tất cả containers
docker compose down

# Dừng và xóa volumes (DỮ LIỆU SẼ MẤT)
docker compose down -v

# Xóa images
docker rmi $(docker images | grep openmetadata | awk '{print $3}')

# Clean up unused resources
docker system prune -a
```

---

---

# 🎨 PHẦN 3: HƯỚNG DẪN SỬ DỤNG UI OPENMETADATA

## 3.1 Giới thiệu UI

Truy cập: **http://localhost:8585**

Credentials mặc định:
- Username:
- Password:

---

## 3.2 Các trang chính trong UI

### 📊 **Home / Dashboard**
- Hiển thị tổng số databases, tables, users
- Shortcuts để truy cập nhanh
- Recent activities
![alt text](image-2.png)
### 🔍 **Explore (Data Discovery)**
- Tìm kiếm metadata
- Filter theo entity type: Databases, Tables, Schemas, Columns
- Advanced search filters

### 📝 **Data Assets**
- **Databases**: Danh sách tất cả databases
- **Tables**: Tất cả tables từ các databases
- **Schemas**: Các schema được indexed
- **Columns**: Column-level metadata

### 🏷️ **Tags & Glossary**
- Quản lý business glossaries
- Tạo tags để phân loại dữ liệu
- Gắn tags cho tables, columns

### 👥 **Admin Panel**
- **Data Sources**: Thêm/quản lý connections tới databases
- **Ingestion**: Chạy ingestion pipelines
- **Teams & Users**: Quản lý users, roles, permissions
- **Settings**: Cấu hình hệ thống

### 📈 **Lineage & Impact**
- Xem lineage của tables (input → output)
- Impact analysis: nếu change table A, ảnh hưởng gì?

---

## 3.3 Quy trình công việc cơ bản

### 🔗 **Bước 1: Thêm Data Source (Kết nối Database)**

1. **Vào Admin → Data Sources**

2. **Click "Add new Data Source"**
![alt text](image-4.png)
3. **Chọn database type** (PostgreSQL, Redshift, Oracle, Trino, etc.)
![alt text](image-3.png)
4. **Điền thông tin kết nối:**
   ```
   Service Name: my-postgres-db
   Connection Type: MySQL / PostgreSQL / Redshift / Oracle / ...
   Host: localhost hoặc IP address
   Port: 5432 (PostgreSQL), 3306 (MySQL), etc.
   Username: user
   Password: password
   Database: database_name
   ```

5. **Click "Test Connection"** để kiểm tra

6. **Click "Save"** để lưu

### ⚠️ **Quan trọng: Thêm Data Source KHÔNG tự động crawl metadata!**

**Khi bạn thêm Data Source:**
- ✅ **Tạo connection configuration** (host, port, credentials)
- ✅ **Lưu trữ thông tin kết nối** trong OpenMetadata database
- ✅ **Cho phép test connection** để verify
- ❌ **KHÔNG tự động crawl metadata** từ database

**Để crawl metadata, bạn PHẢI thực hiện Bước 2: Tạo Ingestion Pipeline**

---

### 🔄 **Bước 2: Chạy Ingestion (Thu thập Metadata)**

1. **Vào Admin → Ingestion**

2. **Click "Create new Ingestion"**

3. **Chọn Data Source** vừa tạo

4. **Cấu hình Ingestion:**
   ```
   Name: Metadata Agent
   Ingestion Type: Metadata (hoặc Usage/Lineage/Profiling)
   
   Metadata Options:
   - Schema Filter: Pattern để select schemas
   - Table Filter: Pattern để select tables
   - Include View Definitions: Yes/No
   ```
![alt text](image-5.png)
5. **Cấu hình Schedule (nếu cần):**
   ```
   Execution Frequency: Daily, Weekly, Monthly, Once
   Time: 2:00 AM
   Timezone: Asia/Bangkok
   ```
![alt text](image-10.png)
6. **Click "Deploy"** để start ingestion

### 🔍 **Quy trình crawl metadata chi tiết:**

Khi bạn **Deploy** ingestion pipeline:

```
1. OpenMetadata tạo Ingestion Configuration
   └─ JSON config với thông tin Data Source + Ingestion options

2. Configuration được gửi tới Airflow
   └─ Airflow tạo DAG (Directed Acyclic Graph) từ config

3. DAG được trigger (manual hoặc scheduled)
   ├─ Task 1: Connect tới database sử dụng credentials từ Data Source
   ├─ Task 2: Query system tables (information_schema, pg_catalog, etc.)
   ├─ Task 3: Fetch metadata: tables, columns, types, constraints, indexes
   ├─ Task 4: Process & validate metadata
   ├─ Task 5: Send metadata tới OpenMetadata API (/api/v1/tables)
   └─ Task 6: Elasticsearch indexing cho search

4. Metadata hiển thị trong UI
   └─ Tables, columns, schemas appear trong Explore page
   └─ Searchable qua Elasticsearch
```

### ⏱️ **Thời gian crawl metadata:**

- **Small database** (< 100 tables): 1-5 phút
- **Medium database** (100-1000 tables): 5-15 phút  
- **Large database** (> 1000 tables): 15-60 phút
- **Very large** (> 10k tables): 1-4 giờ

**Factors ảnh hưởng:**
- Network latency tới database
- Database performance
- Number of tables/columns
- Include profiling data (Yes/No)
- Airflow worker resources

---

### 🔄 **Bước 3: Monitor Ingestion Progress**

Sau khi Deploy ingestion, bạn có thể monitor progress qua 2 cách:

#### 📊 **Cách 1: Monitor từ OpenMetadata UI**

1. **Vào Admin → Ingestion**
2. **Click vào ingestion vừa tạo**
3. **Xem Pipeline Status:**
   ```
   Status: Running / Success / Failed
   Last Run: Thời gian chạy cuối
   Next Run: Thời gian chạy tiếp theo (nếu scheduled)
   Duration: Thời gian thực hiện
   ```
4. **Xem Logs:**
   - Click "View Logs" để xem detailed logs
   - Logs show từng bước: connecting, fetching tables, sending to API

#### 📊 **Cách 2: Monitor từ Airflow UI**

1. **Truy cập Airflow**: http://localhost:8080
2. **Tìm DAG tương ứng:**
   ```
   DAG ID: openmetadata_ingestion_dag_<data_source_name>
   Ví dụ: openmetadata_ingestion_dag_my_postgres
   ```
3. **Xem DAG Runs:**
   - Click vào DAG
   - Xem Graph View: flow của tasks
   - Click vào task để xem logs
4. **Monitor Tasks:**
   ```
   Task 1: metadata-extraction → Fetch metadata từ database
   Task 2: metadata-validation → Validate data
   Task 3: metadata-sink → Send tới OpenMetadata API
   Task 4: elasticsearch-index → Index cho search
   ```

#### 📈 **Cách 3: Monitor qua API**

```bash
# Check ingestion status
curl -X GET "http://localhost:8585/api/v1/services/ingestionPipelines/{ingestion_id}"

# Response
{
  "id": "12345",
  "name": "my-postgres-ingestion",
  "status": "SUCCESS",
  "startDate": "2024-03-23T10:00:00Z",
  "endDate": "2024-03-23T10:05:00Z",
  "executionSummary": {
    "totalRecords": 150,
    "successRecords": 150,
    "failedRecords": 0
  }
}
```

---

### 🔍 **Bước 4: Verify Metadata đã được crawl**

Sau khi ingestion hoàn tất:

1. **Vào Explore page**
2. **Search database name**: "my-postgres-db"
3. **Xem kết quả:**
   ```
   - Database: my-postgres-db
   - Schemas: public, analytics, etc.
   - Tables: users, orders, products, etc.
   - Columns: id, name, email, created_at, etc.
   ```

4. **Click vào table để xem chi tiết:**
   ```
   - Column names & types
   - Row count (nếu enable profiling)
   - Sample data
   - Tags, descriptions
   ```

![alt text](image-6.png)
Schema trên UI Openmetadata
![alt text](image-7.png)
Schema thực tế trên Database
### 🔄 **Tự động crawl metadata định kỳ**

Để metadata luôn up-to-date:

1. **Trong Ingestion Configuration, set Schedule:**
   ```
   Enable Scheduling: Yes
   Frequency: Daily
   Time: 02:00 AM
   ```
![alt text](image-8.png)
Kiểm tra trạng thái

2. **Deploy ingestion**

3. **Airflow sẽ tự động trigger mỗi ngày lúc 2:00 AM**

**Lợi ích của scheduled ingestion:**
- ✅ Metadata luôn mới
- ✅ Consistent & reliable  
- ✅ No manual intervention
- ✅ Monitor qua Airflow UI
- ✅ Logs & alerts nếu fail

---

### 🔎 **Bước 3: Tìm kiếm & Khám phá Dữ liệu**

1. **Vào Explore**

2. **Tìm kiếm:**
   ```
   - Search by table name: "users", "orders"
   - Search by description: "customer data"
   - Filter by Database: "my-postgres-db"
   - Filter by Schema: "public"
   ```

3. **Xem chi tiết table:**
   - Columns & Types
   - Row count, sample data
   - Tags, descriptions
   - Owners
   - Lineage

---

### 🏷️ **Bước 4: Gắn Metadata (Tags, Descriptions, Owners)**

1. **Mở một table**

2. **Gắn Tags:**
   - Click "Add Tag"
   - Chọn hoặc tạo tag mới: "PII", "Critical", "Customer", etc.
![alt text](image-9.png)
Gán tag dữ liệu nhạy cảm
3. **Thêm Description:**
   - Click vào description field
   - Thêm text mô tả table/column

4. **Gắn Owner:**
   - Click "Add Owner"
   - Chọn user hoặc team

---

## 3.4 Các tính năng nâng cao

### 📊 **Data Lineage (Luồng dữ liệu)**

Xem cách data flows từ source đến destination:
```
Raw Table (source_db.raw_users)
        ↓
Transformation (ETL pipeline)
        ↓
Processed Table (analytics_db.dim_users)
        ↓
BI Report (Tableau "Customer Dashboard")
```

**Cách xem:**
1. Mở một table
2. Click tab "Lineage"
3. Xem upstream (sources) và downstream (consumers)

---

### 📈 **Data Profiling**

Thống kê về data:
- Row count
- Column count
- NULL values %
- Average, Min, Max (cho numeric columns)
- Unique values (cho categorical columns)

**Cách chạy:**
1. Editor Ingestion Configuration
2. Enable "Profile Data"
3. Deploy ingestion

---

### 🔐 **Governance & Policies**

**Quản lý quy tắc dữ liệu:**
- **Glossary**: Định nghĩa business terms
- **Data Quality Rules**: Validate dữ liệu
- **Data Access Policies**: Kiểm soát ai có thể truy cập data nào

---

---

# 🗄️ PHẦN 4: KẾT NỐI CÁC DATABASE VỚI OPENMETADATA

## 4.1 PostgreSQL Connection

### 📋 Thông tin cần chuẩn bị:
```
Service Name: my-postgres
Database Type: PostgreSQL
Host: localhost hoặc IP
Port: 5432
Username: postgres
Password: your_password
Database: your_db
```

### 🔧 Cấu hình Chi tiết:

**docker-compose.yml (nếu chạy PostgreSQL riêng):**
```yaml
my-postgres-db:
  image: postgres:15
  environment:
    POSTGRES_USER: postgres
    POSTGRES_PASSWORD: your_secure_password
    POSTGRES_DB: your_database
  ports:
    - "5432:5432"
  volumes:
    - postgres_data:/var/lib/postgresql/data
```

**Bước thêm vào OpenMetadata:**
1. Admin → Data Sources → "Add new Data Source"
2. Select **PostgreSQL**
3. Điền form:
   ```
   Service Name: my-postgres
   Host: postgres (nếu cùng docker network) hoặc localhost
   Port: 5432
   Username: postgres
   Password: your_secure_password
   Database: your_database
   ```
4. Test Connection → Save

### ⚙️ Advanced Configuration (Tuỳ chọn):

```
Connection Arguments:
  - sslmode: require/disable
  - application_name: openmetadata
  - connection_timeout: 30

Ingestion Configuration:
  - Include Views: Yes/No
  - Schema Name Pattern: (regex) "public|analytics"
  - Table Name Pattern: (regex) except system tables
  - Include Table Comments: Yes
```

---

## 4.2 Amazon Redshift Connection

### 🔴 Thông tin cần chuẩn bị:
```
Service Name: my-redshift
Database Type: Redshift
Host: your-cluster.123456.us-east-1.redshift.amazonaws.com
Port: 5439
Username: awsuser
Password: your_password
Database: dev
```

### 🔧 Cấu hình Chi tiết:

**Thêm vào OpenMetadata:**
1. Admin → Data Sources → "Add new Data Source"
2. Select **Redshift**
3. Điền form:
   ```
   Service Name: my-redshift-prod
   Host: your-cluster-xyz.c123456.us-east-1.redshift.amazonaws.com
   Port: 5439
   Username: awsuser
   Password: your_redshift_password
   Database: prod_db
   ```
4. Test Connection → Save

### ⚙️ Advanced Configuration:

```
Connection Arguments:
  - region: us-east-1
  - iam: true/false
  
Ingestion Configuration:
  - Include Spectrum Tables: Yes/No
  - Include Unload Tables: Yes/No
  - Include Materialized Views: Yes
  - Schema Filter: "public|analytics|staging"
  - Enable Profiling: Yes
```

### 🔑 IAM Authentication (nâng cao):

Nếu sử dụng IAM roles thay vì username/password:

```
Connection Type: IAM
AWS Access Key ID: YOUR_ACCESS_KEY
AWS Secret Access Key: YOUR_SECRET_KEY
AWS Region: us-east-1
Cluster ID: your-cluster
Database: prod_db
```

---

## 4.3 Oracle Database Connection

### 🔶 Thông tin cần chuẩn bị:
```
Service Name: my-oracle
Database Type: Oracle
Host: oracle-server.company.com hoặc IP
Port: 1521
Username: oracle_user
Password: your_password
Database (Service Name): ORCL hoặc XE
```

### 🔧 Cấu hình Chi tiết:

**Thêm vào OpenMetadata:**
1. Admin → Data Sources → "Add new Data Source"
2. Select **Oracle**
3. Điền form:
   ```
   Service Name: my-oracle-prod
   Host: 192.168.1.100 hoặc oracle.company.com
   Port: 1521
   Username: sys (hoặc other_user)
   Password: oracle_password
   Service Name: ORCL
   ```
4. Test Connection → Save

### ⚙️ Advanced Configuration:

```
Connection Type: Service Name hoặc SID

If SID:
  - SID: ORCL

Connection Arguments:
  - thick_mode: true/false
  - thick_mode_lib_dir: /path/to/oracle/lib (nếu thick_mode=true)

Ingestion Configuration:
  - Include Views: Yes
  - Include Materialized Views: Yes
  - Owner Schema: HR (hoặc schema name cần ingest)
  - Table Name Pattern: (regex)
  - Enable Profiling: Yes
```

### 📦 Requirement:

Nếu chạy trong Docker, vào Docker Compose, ingestion service cần có Oracle client:

```yaml
ingestion:
  build:
    context: .
    dockerfile: Dockerfile
    args:
      - ORACLE_HOME=/opt/oracle/instantclient
  environment:
    LD_LIBRARY_PATH: /opt/oracle/instantclient:$LD_LIBRARY_PATH
```

---

## 4.4 Trino Connection

### 🟣 Thông tin cần chuẩn bị:
```
Service Name: my-trino
Database Type: Trino
Host: trino-coordinator.company.com hoặc localhost
Port: 8080
Username: trino_user
Catalog: postgres hoặc hive (tuỳ config)
Schema: default hoặc tên schema
```

### 🔧 Cấu hình Chi tiết:

**Thêm vào OpenMetadata:**
1. Admin → Data Sources → "Add new Data Source"
2. Select **Trino**
3. Điền form:
   ```
   Service Name: my-trino-cluster
   Host: trino-coordinator-node-1
   Port: 8080
   Username: admin
   Password: (bỏ trống nếu không có auth)
   Catalog: hive (catalog name - là connector trong Trino)
   ```
4. Test Connection → Save

### ⚙️ Advanced Configuration:

```
Connection Arguments:
  - ssl: true/false
  - verify_ssl: Yes/No
  - auth: basic/oauth/custom

Ingestion Configuration:
  - Catalog: hive hoặc postgres (tuỳ từng catalog)
  - Schema Pattern: (regex) "public|analytics"
  - Include View Definitions: Yes
  - Enable Profiling: Yes
```

### 🔐 Authentication (nâng cao):

```
LDAP:
  - auth_type: ldap
  - username: ldap_user
  - password: ldap_password

Kerberos (nếu Trino integrated với Kerberos):
  - auth_type: kerberos
  - keytab: /path/to/keytab
  - principal: user@REALM
```

---

## 4.5 So sánh các cách Connection

| Database | Driver | Port | Auth | Special |
|----------|--------|------|------|---------|
| PostgreSQL | psycopg2 | 5432 | username/password | SSL option |
| Redshift | psycopg2 | 5439 | IAM / username | AWS specific |
| Oracle | cx_Oracle | 1521 | username/password | SID or Service Name |
| Trino | trino-python | 8080 | optional | Catalog required |

---

## 4.6 Ingestion Configuration cho tất cả Databases

Sau khi thêm data source, cần setup Ingestion:

### 📋 Common Ingestion Options:

```yaml
Ingestion Name: daily-metadata-ingest
Database/Source: my-postgres (hoặc data source name)

Metadata Ingestion:
  Include Views: Yes/No
  Include Table Comments: Yes
  Include Column Comments: Yes
  Schema Name Pattern: (regex) "^public$|^analytics$"
  Table Name Pattern: (regex) "^(?!tmp_).*"  # Exclude tables starting with tmp_
  Bulk Size: 10

Usage Ingestion (Optional):
  Enable Usage: Yes/No
  Sample Data: Yes
  Query Log Location: /path/to/logs

Lineage Ingestion (Optional):
  Enable Lineage: Yes/No
  DBT Manifest Path: ./target/manifest.json
  
Data Profiling (Optional):
  Enable Profiling: Yes/No
  Profile Sample Size: 100
  Number of Workers: 4
```

### ⏱️ Scheduling:

```
Frequency: Daily
Time: 02:00 AM
Timezone: Asia/Bangkok
Or: Weekly, Monthly, Once
```

---

---

# 🔄 PHẦN 5: OPENMETADATA VỚI APACHE AIRFLOW

## 5.1 Airflow là gì & tại sao dùng với OpenMetadata?

### 🌬️ Apache Airflow:
- **Workflow Orchestration Platform** - Tự động hóa các công việc theo lịch trình
- **DAG-based**: Directed Acyclic Graph = Một chuỗi tasks có dependencies
- **Scheduler**: Chạy DAGs vào thời gian định sẵn
- **Monitoring**: Theo dõi execution status, logs, metrics

### 🎯 Tại sao dùng Airflow với OpenMetadata?

```
Manual Ingestion (Click button mỗi lần):
  ❌ Không consistent
  ❌ Có thể quên
  ❌ Metadata không up-to-date

Scheduled Ingestion (Airflow):
  ✅ Chạy tự động theo lịch
  ✅ Consistent & reliable
  ✅ Metadata luôn mới
  ✅ Easy to monitor & debug
  ✅ Có thể trigger từ code
```

---

## 5.2 Cấu trúc Airflow trong OpenMetadata

### 📁 Thư mục Airflow:

```
/opt/airflow/
├── dags/
│   ├── openmetadata_ingestion_dag.py
│   ├── custom_dag.py
│   └── ...
├── dag_generated_configs/
│   ├── ingestion_config_1.json
│   ├── ingestion_config_2.json
│   └── ...
├── logs/
│   ├── openmetadata_ingestion_dag/
│   └── ...
└── airflow.cfg
```

### 🔧 Airflow Components:

```
Scheduler: Chạy DAGs theo schedule
├─ Check DAGs mỗi phút
├─ Nếu tới scheduled time → Trigger DAG

Executor: Chạy tasks
├─ LocalExecutor (1 machine, sequential)
├─ CeleryExecutor (distributed, nhiều machines)

Webserver: UI để xem DAGs, logs, etc.
├─ http://localhost:8080
├─ Monitor DAG runs
├─ Trigger DAGs manually

Database: Lưu DAG/Task metadata
└─ PostgreSQL (cùng với OpenMetadata)
```

---

## 5.3 OpenMetadata Ingestion DAGs

### 📊 Flow công việc:

```
1. OpenMetadata UI
   └─> Admin → Ingestion → Create new Ingestion

2. OpenMetadata ghi cấu hình vào Database
   └─> JSON config lưu tại dag_generated_configs/

3. Airflow Scheduler phát hiện config mới
   └─> Generate DAG từ config

4. DAG được trigger vào scheduled time
   ├─ Task 1: Connect tới Data Source
   ├─ Task 2: Fetch metadata (tables, columns, etc.)
   ├─ Task 3: Process & validate metadata
   ├─ Task 4: Send metadata tới OpenMetadata API
   └─ Task 5: Update Elasticsearch index

5. Kết quả:
   └─> Metadata show lên UI, searchable via Elasticsearch
```

### 🔍 Xem log Ingestion:

```bash
# Xem logs từ docker
docker compose logs -f ingestion

# Hoặc vào Airflow UI
# http://localhost:8080
# → DAGs → openmetadata_ingestion_dag
# → Click vào DAG Run
# → Xem logs từng task
```

---

## 5.4 Cách trigger Ingestion từ Code

### 🚀 Method 1: Sử dụng OpenMetadata API

**HTTP Request để trigger ingestion:**

```python
import requests
import json

# Base URL
OPENMETADATA_BASE_URL = "http://localhost:8585/api/v1"

# Step 1: Get access token (nếu cần)
# headers = {"Authorization": "Bearer YOUR_TOKEN"}
headers = {}  # Nếu dùng default credentials

# Step 2: Trigger ingestion
ingestion_id = "12345678-1234-1234-1234-123456789abc"  # Từ OpenMetadata UI

response = requests.post(
    f"{OPENMETADATA_BASE_URL}/services/ingestionPipelines/{ingestion_id}/trigger",
    headers=headers,
    json={}
)

print(f"Status: {response.status_code}")
print(f"Response: {response.json()}")
```

**Response:**
```json
{
  "executionId": "run-12345",
  "status": "QUEUED",
  "startTime": "2024-03-23T10:00:00Z"
}
```

---

### 🚀 Method 2: Trigger qua Airflow API

**HTTP Request:**

```python
import requests
import json
from datetime import datetime

# Airflow URL
AIRFLOW_BASE_URL = "http://localhost:8080/api/v1"

# Airflow Credentials
AIRFLOW_USER = "admin"
AIRFLOW_PASSWORD = "admin"

# Step 1: Authenticate
response = requests.post(
    f"{AIRFLOW_BASE_URL}/auth/login",
    json={"username": AIRFLOW_USER, "password": AIRFLOW_PASSWORD}
)
token = response.json()["access_token"]
headers = {"Authorization": f"Bearer {token}"}

# Step 2: Trigger DAG
dag_id = "openmetadata_ingestion_dag_my_postgres"  # DAG name

response = requests.post(
    f"{AIRFLOW_BASE_URL}/dags/{dag_id}/dagRuns",
    headers=headers,
    json={
        "conf": {},
        "execution_date": datetime.utcnow().isoformat() + "Z"
    }
)

print(f"Status: {response.status_code}")
print(f"DAG Run: {response.json()['dag_run_id']}")
```

**Response:**
```json
{
  "dag_id": "openmetadata_ingestion_dag_my_postgres",
  "dag_run_id": "manual__2024-03-23",
  "state": "queued",
  "start_date": "2024-03-23T10:00:00Z"
}
```

---

### 🚀 Method 3: Python Script (Using OpenMetadata SDK)

```python
# pip install openmetadata-ingestion

from metadata.workflow.workflow import Workflow
from metadata.ingestion.models.pipeline_state import PipelineState

# Configuration
config = {
    "source": {
        "type": "postgres",
        "serviceName": "my-postgres",
        "serviceConnection": {
            "config": {
                "type": "Postgres",
                "hostPort": "localhost:5432",
                "username": "postgres",
                "password": "postgres",
                "database": "mydb"
            }
        }
    },
    "processor": {
        "type": "default"
    },
    "sink": {
        "type": "metadata-rest",
        "config": {}
    },
    "workflowConfig": {
        "loggerLevel": "INFO",
        "openMetadataServerConfig": {
            "hostPort": "http://localhost:8585",
            "authProvider": "no-auth"
        }
    }
}

# Run ingestion
workflow = Workflow.create(config)
status = workflow.execute()
workflow.stop()

if status == PipelineState.SUCCESS:
    print("✅ Ingestion successful!")
else:
    print(f"❌ Ingestion failed: {status}")
```

---

## 5.5 Schedule Ingestion vào thời gian cụ thể

### 🕐 Cách 1: Setup từ UI OpenMetadata

1. **Admin → Ingestion → Create new Ingestion**
2. **Điền tất cả thông tin**
3. **Mục Schedule:**
   ```
   Enable Scheduling: Yes
   Frequency: Daily / Weekly / Monthly / Once
   Scheduling Time: 02:00 AM
   Timezone: Asia/Bangkok
   ```
4. **Deploy**

→ Airflow sẽ tự động trigger vào 2:00 AM mỗi ngày

---

### 🕐 Cách 2: Viết Custom DAG

**File: `/opt/airflow/dags/my_custom_ingestion_dag.py`**

```python
from datetime import datetime, timedelta
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.operators.http_operator import SimpleHttpOperator
import requests

# DAG Definition
default_args = {
    'owner': 'openmetadata',
    'retries': 2,
    'retry_delay': timedelta(minutes=5),
}

dag = DAG(
    'my_custom_ingestion_dag',
    default_args=default_args,
    description='Custom ingestion pipeline',
    schedule_interval='0 2 * * *',  # 2:00 AM daily
    start_date=datetime(2024, 3, 1),
    catchup=False,
)

# Step 1: Python task - trigger ingestion via API
def trigger_ingestion(**context):
    ingestion_id = "12345678-1234-1234-1234-123456789abc"
    url = f"http://openmetadata-server:8585/api/v1/services/ingestionPipelines/{ingestion_id}/trigger"
    
    response = requests.post(url, json={})
    
    if response.status_code == 200:
        print(f"✅ Ingestion triggered: {response.json()}")
        return response.json()['executionId']
    else:
        raise Exception(f"❌ Failed to trigger ingestion: {response.text}")

# Step 2: Wait task - chờ ingestion hoàn tất
def wait_for_completion(**context):
    execution_id = context['task_instance'].xcom_pull(task_ids='trigger_task')
    # Lôgic chờ ở đây
    print(f"Waiting for execution {execution_id} to complete...")

# Define tasks
trigger_task = PythonOperator(
    task_id='trigger_ingestion',
    python_callable=trigger_ingestion,
    dag=dag,
)

wait_task = PythonOperator(
    task_id='wait_for_ingestion',
    python_callable=wait_for_completion,
    dag=dag,
)

# Define dependencies
trigger_task >> wait_task
```

---

### 🕐 Cách 3: Trigger via Cron Expression

**Airflow dùng cron expressions:**

```
schedule_interval='0 2 * * *'     # 2:00 AM mỗi ngày
schedule_interval='0 0 * * MON'   # Thứ Hai 12:00 AM
schedule_interval='*/15 * * * *'  # Mỗi 15 phút
schedule_interval='0 2 1 * *'     # 2:00 AM mỗi 1 tháng
```

---

## 5.6 Monitoring Ingestion

### 📊 Xem từ Airflow UI:

1. **Truy cập Airflow**: http://localhost:8080

2. **DAGs tab**:
   - Danh sách tất cả DAGs
   - Click vào `openmetadata_ingestion_dag` hoặc custom DAG

3. **DAG Details**:
   - Tree View: Hiển thị cây tasks
   - Graph View: Flow diagram
   - Calendar View: Historical runs

4. **DAG Runs**:
   - Xem tất cả runs (executions)
   - Status: Success / Failed / Running
   - Duration: Mất bao lâu
   - Logs: Click vào task → xem detailed logs

### 📊 Xem từ OpenMetadata UI:

1. **Admin → Ingestion**
   - Danh sách tất cả ingestions
   - Last Run: Thời gian chạy cuối
   - Status: Successful / Failed / Running
   - Logs: Click vào ingestion → xem logs

---

## 5.7 Troubleshooting Ingestion

### ❌ Ingestion Failed

**Log ở Airflow UI:**
```
[ERROR] Connection refused to localhost:5432
```

**Giải pháp:**
- Kiểm tra PostgreSQL container có chạy không: `docker compose ps postgresql`
- Kiểm tra credentials: username, password, database name
- Xem logs: `docker compose logs postgresql`

---

### ❌ Task Timeout

**Log:**
```
[ERROR] Task timed out after 3600 seconds
```

**Giải pháp:**
- Tăng timeout trong DAG config:
  ```python
  execution_timeout=timedelta(hours=2)
  ```
- Hoặc check data source - có thể dataset quá lớn

---

### ❌ Memory Error

**Log:**
```
[ERROR] java.lang.OutOfMemoryError: Java heap space
```

**Giải pháp:**
- Tăng memory cho Airflow container:
  ```yaml
  ingestion:
    environment:
      - JAVA_OPTS=-Xmx2g -Xms1g
  ```

---

## 5.8 Advanced: Custom Ingestion Pipeline

### 📝 Viết custom connector:

```python
# File: custom_connector.py

from metadata.ingestion.source.database.common_db_source import CommonDbSource
from metadata.ingestion.models.pipeline_state import PipelineState

class CustomDataSource(CommonDbSource):
    """Custom data source connector"""
    
    def _get_tables(self):
        """Fetch tables từ custom source"""
        # Logic để connect & fetch tables
        tables = []
        # ...
        return tables
    
    def get_table_description(self, schema_name: str, table_name: str) -> str:
        """Get table description"""
        return "Table description"
    
    def get_column_description(self, schema_name: str, table_name: str, column_name: str) -> str:
        """Get column description"""
        return "Column description"

# Usage trong DAG:
config = {
    "source": {
        "type": "custom",
        "serviceName": "my-custom-source",
        "sourceClass": "custom_connector.CustomDataSource"
    },
    # ... rest of config
}
```

---

## 5.9 Airflow Commands útil

```bash
# Xem DAGs
airflow dags list

# Trigger DAG manually
airflow dags trigger dag_id

# Xem DAG details
airflow dags show dag_id

# Xem task logs
airflow tasks logs dag_id task_id run_id

# Backfill (chạy lại historical runs)
airflow dags backfill dag_id \
  --start-date 2024-03-01 \
  --end-date 2024-03-10

# Pause/Unpause DAG
airflow dags pause dag_id
airflow dags unpause dag_id
```

---

## 5.10 Flow Tổng hợp: Từ Tạo Data Source đến Scheduled Ingestion

```
1. Tạo Data Source (PostgreSQL)
   └─ Admin → Data Sources → Add PostgreSQL

2. Tạo Ingestion Configuration
   └─ Admin → Ingestion → Create Ingestion
   └─ Link tới PostgreSQL data source
   └─ Cấu hình metadata ingestion options
   └─ Set schedule: Daily 2:00 AM
   └─ Deploy

3. OpenMetadata lưu config
   └─ Config JSON → db_generated_configs/
   └─ API call từ OpenMetadata → Airflow

4. Airflow Scheduler phát hiện config mới
   └─ Generate DAG từ config
   └─ Đăng ký DAG vào scheduler

5. Mỗi ngày lúc 2:00 AM
   └─ Scheduler trigger DAG
   └─ Task 1: Connect PostgreSQL
   └─ Task 2: Fetch metadata
   └─ Task 3: Send to OpenMetadata API
   └─ Task 4: Index to Elasticsearch

6. Kết quả
   └─ Metadata appear trong OpenMetadata UI
   └─ Searchable qua Explore page
   └─ Lineage, profiling, tags visible
```

---

---

# 🎓 PHẦN 6: BEST PRACTICES & ADVANCED TIPS

## 6.1 Performance Optimization

✅ **Elasticsearch tuning:**
```yaml
elasticsearch:
  environment:
    ES_JAVA_OPTS: -Xms2g -Xmx2g  # Tăng lên 2GB
    indices.memory.index_buffer_size: 30%
```

✅ **PostgreSQL tuning:**
```
shared_buffers: 256MB
work_mem: 16MB
maintenance_work_mem: 64MB
```

✅ **Ingestion optimization:**
- Enable batch processing: `BATCH_SIZE: 500`
- Parallel workers cho profiling: `WORKERS: 4`
- Schema/table filter patterns để ingest chỉ cần thiết

---

## 6.2 Data Governance Best Practices

✅ Tạo **Glossary** để standardize terminology
✅ Gắn **Owners** cho tất cả critical tables
✅ Sử dụng **Tags** để phân loại: PII, Critical, Archive, etc.
✅ Document **Data Quality Rules** cho important datasets
✅ Regular **Access Reviews** qua Data Access Policies

---

## 6.3 Backup & Disaster Recovery

**Backup PostgreSQL:**
```bash
docker compose exec postgresql pg_dump -U postgres openmetadata_db > backup.sql

# Restore
docker compose exec -T postgresql psql -U postgres openmetadata_db < backup.sql
```

**Backup Elasticsearch:**
```bash
# Create snapshot
curl -X PUT http://localhost:9200/_snapshot/my_backup

# Backup
curl -X PUT http://localhost:9200/_snapshot/my_backup/snapshot_1
```

---

---

# 📞 PHẦN 7: RESOURCES & SUPPORT

## 📖 Documentation:
- [OpenMetadata Official Docs](https://docs.open-metadata.org/)
- [Apache Airflow Docs](https://airflow.apache.org/docs/)
- [PostgreSQL Docs](https://www.postgresql.org/docs/)

## 🐛 Troubleshooting:
- [GitHub Issues](https://github.com/open-metadata/OpenMetadata/issues)
- [Community Discussions](https://github.com/open-metadata/OpenMetadata/discussions)
- Logs: `docker compose logs <service_name>`

## 🆘 Common Issues:

| Issue | Solution |
|-------|----------|
| Services unhealthy | Give them more time to start (2-3 min) |
| Memory error | Increase heap: `OPENMETADATA_HEAP_OPTS: -Xmx2G` |
| Connection refused | Check service is running: `docker compose ps` |
| Port already in use | Change ports in docker-compose.yml |
| Ingestion stuck | Check logs: `docker compose logs ingestion` |

---

Last Updated: 2026-03-23
Status: Active
Maintainer: Data Team