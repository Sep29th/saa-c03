# Phần 9 — AWS Fundamentals: RDS + Aurora + ElastiCache

> Khóa học: *Ultimate AWS Certified Solutions Architect Associate 2026* (Stéphane Maarek) — SAA-C03
> Nguồn tham chiếu: `AWS Certified Solutions Architect Slides v48.pdf` (phần "RDS, Aurora & ElastiCache")

---

## Mục lục

| # | Bài giảng | Thời lượng | Loại |
|---|-----------|-----------|------|
| 88 | [Amazon RDS Overview](#88-amazon-rds-overview) | 4 phút | Video |
| 89 | [RDS Read Replicas vs Multi AZ](#89-rds-read-replicas-vs-multi-az) | 7 phút | Video |
| 90 | [Amazon RDS Hands On](#90-amazon-rds-hands-on) | 9 phút | Video |
| 91 | [RDS Custom for Oracle and Microsoft SQL Server](#91-rds-custom-for-oracle-and-microsoft-sql-server) | 2 phút | Video |
| 92 | [Amazon Aurora](#92-amazon-aurora) | 6 phút | Video |
| 93 | [Amazon Aurora - Hands On](#93-amazon-aurora---hands-on) | 8 phút | Video |
| 94 | [Amazon Aurora - Advanced Concepts](#94-amazon-aurora---advanced-concepts) | 8 phút | Video |
| 95 | [RDS & Aurora - Backup and Monitoring](#95-rds--aurora---backup-and-monitoring) | 6 phút | Video |
| 96 | [RDS Security](#96-rds-security) | 3 phút | Video |
| 97 | [RDS Proxy](#97-rds-proxy) | 5 phút | Video |
| 98 | [ElastiCache Overview](#98-elasticache-overview) | 5 phút | Video |
| 99 | [ElastiCache Hands On](#99-elasticache-hands-on) | 5 phút | Video |
| 100 | [ElastiCache for Solution Architects](#100-elasticache-for-solution-architects) | 3 phút | Video |
| 101 | [List of Ports to be familiar with](#101-list-of-ports-to-be-familiar-with) | 1 phút | Bài viết |
| — | [Trắc nghiệm 6: RDS, Aurora, & ElastiCache Quiz](#trắc-nghiệm-6-rds-aurora--elasticache-quiz) | — | Quiz |

---

## 88. Amazon RDS Overview

### RDS là gì?

- **RDS = Relational Database Service** (Dịch vụ Cơ sở dữ liệu Quan hệ).
- Là **dịch vụ DB được QUẢN LÝ (managed)** cho các database dùng **SQL làm ngôn ngữ truy vấn**.
- Cho phép bạn **tạo database trên cloud được AWS quản lý**.

### 7 database engine được hỗ trợ ⭐

| Engine | Ghi chú |
|--------|---------|
| **Postgres** | Mã nguồn mở |
| **MySQL** | Mã nguồn mở |
| **MariaDB** | Mã nguồn mở |
| **Oracle** | Thương mại |
| **Microsoft SQL Server** | Thương mại |
| **IBM DB2** | Thương mại |
| **Aurora** | ⭐ **Database độc quyền của AWS** |

---

### Ưu điểm của RDS so với tự triển khai DB trên EC2 ⭐⭐

**RDS là một dịch vụ được quản lý (managed service):**

- **Automated provisioning, OS patching** (tự động cấp phát, vá lỗi hệ điều hành)
- **Continuous backups và restore về một timestamp cụ thể (Point in Time Restore!)** ⭐
- **Monitoring dashboards**
- **Read replicas để cải thiện hiệu năng ĐỌC** ⭐
- **Multi-AZ setup cho DR (Disaster Recovery)** ⭐
- **Maintenance windows cho việc nâng cấp**
- **Khả năng scaling (cả VERTICAL và HORIZONTAL)** ⭐
- **Storage được hỗ trợ bởi EBS** ⭐

### ⚠️ **NHƯNG bạn KHÔNG THỂ SSH vào instance của RDS** ⭐⭐

> **Đây là câu hỏi bẫy kinh điển.** Nếu đề yêu cầu **truy cập OS / SSH vào database** → đáp án **KHÔNG phải RDS thường**, mà là **RDS Custom** (bài 91) hoặc **tự cài DB trên EC2**.

---

### RDS – Storage Auto Scaling ⭐⭐

- **Giúp TĂNG dung lượng lưu trữ trên RDS DB instance một cách ĐỘNG (dynamically)**.
- **Khi RDS phát hiện bạn sắp hết dung lượng trống, nó tự động scale**.
- **Tránh phải scale storage thủ công**.
- **Bạn PHẢI đặt Maximum Storage Threshold** (giới hạn tối đa cho dung lượng DB) ⭐

### ⭐ Ba điều kiện để RDS tự động thay đổi storage (rất hay ra thi):

| # | Điều kiện |
|---|-----------|
| 1 | **Dung lượng trống ÍT HƠN 10% dung lượng đã cấp phát** |
| 2 | **Tình trạng thiếu dung lượng kéo dài ÍT NHẤT 5 PHÚT** |
| 3 | **ĐÃ 6 GIỜ trôi qua kể từ lần thay đổi gần nhất** |

- **Hữu ích cho các ứng dụng có workload KHÔNG ĐOÁN TRƯỚC ĐƯỢC (unpredictable)** ⭐
- **Hỗ trợ TẤT CẢ các RDS database engine**.

---

## 89. RDS Read Replicas vs Multi AZ

Đây là **một trong những chủ đề ra thi nhiều nhất** của phần này.

---

### 🔵 RDS Read Replicas — cho khả năng mở rộng ĐỌC (read scalability)

#### Đặc điểm ⭐⭐

- **Tối đa 15 Read Replicas** ⭐
- Có thể đặt **trong cùng AZ, khác AZ (Cross AZ), hoặc khác Region (Cross Region)** ⭐
- **Replication là ASYNC (bất đồng bộ)**, nên **reads là "eventually consistent"** (nhất quán cuối cùng) ⭐⭐
- **Replica có thể được PROMOTE (thăng cấp) thành DB độc lập của riêng nó** ⭐
- **Ứng dụng PHẢI CẬP NHẬT connection string** để tận dụng read replica ⭐

```
                 Application
        reads   /    │ writes    \  reads
               /     │            \
   RDS DB     ◄──ASYNC──  RDS DB  ──ASYNC──►  RDS DB
   read replica      replication   instance   replication   read replica
```

#### Use Cases ⭐

- Bạn có **production database đang chịu tải bình thường**.
- Bạn muốn **chạy một ứng dụng báo cáo (reporting) để phân tích số liệu**.
- Bạn **tạo một Read Replica để chạy workload mới ở đó**.
- ⭐ **Ứng dụng production KHÔNG BỊ ẢNH HƯỞNG**.
- ⚠️ **Read replicas CHỈ dùng cho câu lệnh SELECT (= đọc)** — **KHÔNG dùng cho INSERT, UPDATE, DELETE** ⭐⭐

#### Network Cost ⭐⭐

- **Trong AWS, có chi phí mạng khi dữ liệu đi từ AZ này sang AZ khác.**
- ⭐ **Với RDS Read Replicas TRONG CÙNG MỘT REGION, bạn KHÔNG PHẢI TRẢ phí đó.**

| Kịch bản | Chi phí |
|----------|---------|
| **Same Region / Different AZ** (`us-east-1a` → `us-east-1b`) | ✅ **MIỄN PHÍ** |
| **Cross-Region** (`us-east-1a` → `eu-west-1b`) | 💰 **CÓ TÍNH PHÍ ($$$)** |

> **Bẫy thi:** "Read replica qua AZ khác có tốn tiền mạng không?" → **KHÔNG** (cùng Region). "Qua Region khác?" → **CÓ**.

---

### 🟢 RDS Multi-AZ (Disaster Recovery)

#### Đặc điểm ⭐⭐

- **SYNC replication (đồng bộ)** ⭐ — khác hẳn Read Replica (ASYNC)
- **MỘT DNS name duy nhất — ứng dụng TỰ ĐỘNG failover sang standby** ⭐
- **Tăng tính SẴN SÀNG (availability)**
- **Failover khi: mất AZ, mất mạng, lỗi instance hoặc lỗi storage** ⭐
- **KHÔNG cần can thiệp thủ công vào ứng dụng**
- ⚠️ **KHÔNG dùng để scaling** ⭐⭐
- **Lưu ý: Read Replicas CÓ THỂ được thiết lập là Multi-AZ cho mục đích DR**

```
              Application
         writes │      │ reads
                ▼      ▼
    ┌──── One DNS name – automatic failover ────┐
    │                                            │
 RDS Master DB  ◄────SYNC replication────►  RDS DB instance
 instance (AZ A)                             standby (AZ B)
```

> ⭐ **Standby instance KHÔNG phục vụ đọc.** Nó chỉ nằm chờ để failover. Đây là điểm khác biệt cốt lõi với Read Replica.

---

### RDS – From Single-AZ to Multi-AZ ⭐

Chuyển từ Single-AZ sang Multi-AZ:

- **KHÔNG có downtime (zero downtime operation)** — **không cần dừng DB** ⭐
- **Chỉ cần bấm "Modify" cho database**
- **Bên trong, những việc sau xảy ra:**
  1. **Một snapshot được tạo**
  2. **Một DB mới được restore từ snapshot đó ở một AZ mới**
  3. **Synchronization (đồng bộ) được thiết lập giữa hai database**

```
  RDS DB instance ──snapshot──► DB snapshot ──restore──► Standby DB
         ▲                                                    │
         └────────────── SYNC Replication ────────────────────┘
```

---

### ⭐⭐⭐ BẢNG SO SÁNH QUAN TRỌNG NHẤT: Read Replicas vs Multi-AZ

| | **Read Replicas** | **Multi-AZ** |
|---|---|---|
| **Mục đích** | **Scale ĐỌC (read scalability)** | **Disaster Recovery / High Availability** |
| **Kiểu replication** | **ASYNC** (eventually consistent) | **SYNC** (đồng bộ) |
| **Số lượng** | **Tối đa 15** | **1 standby** |
| **Phạm vi** | Same AZ / Cross AZ / **Cross Region** | **Cross AZ** (trong cùng Region) |
| **Phục vụ đọc?** | ✅ **CÓ** | ❌ **KHÔNG** (standby chỉ nằm chờ) |
| **Connection string** | ⚠️ **Ứng dụng phải đổi** | ✅ **MỘT DNS name, tự động failover** |
| **Dùng để scaling?** | ✅ **Có** | ❌ **KHÔNG** |
| **Chi phí mạng inter-AZ** | ✅ Miễn phí (cùng Region) | Miễn phí |
| **Promote thành DB riêng?** | ✅ Có | — |

### Mẹo nhận diện đề thi ⭐

| Từ khóa trong đề | Đáp án |
|------------------|--------|
| "cải thiện hiệu năng **ĐỌC**", "reporting/analytics" | **Read Replicas** |
| "**scale reads**", "giảm tải cho DB chính" | **Read Replicas** |
| "**high availability**", "sống sót khi mất AZ" | **Multi-AZ** |
| "**disaster recovery**", "automatic failover" | **Multi-AZ** |
| "eventually consistent" | **Read Replicas (ASYNC)** |
| "không cần đổi code ứng dụng khi DB lỗi" | **Multi-AZ** (một DNS name) |
| "read replica ở Region khác" | **Cross-Region Read Replica** (tốn phí mạng) |

---

## 90. Amazon RDS Hands On

### Bước 1 — Tạo RDS Database

1. Console → tìm **RDS** → **Databases** → **Create database**.
2. **Choose a database creation method**:
   - **Standard create** (đầy đủ tùy chọn) ⭐
   - **Easy create** (dùng preset)
3. **Engine options**: chọn **MySQL** (hoặc PostgreSQL).
4. **Templates**: ⭐ chọn **Free tier** (nếu còn trong 12 tháng đầu)
   - Các lựa chọn khác: `Production` (mặc định Multi-AZ), `Dev/Test`
5. **Settings**:
   - **DB instance identifier**: `database-1`
   - **Master username**: `admin`
   - **Credentials management**:
     - ⭐ **Managed in AWS Secrets Manager** (khuyến nghị — tự động rotate)
     - hoặc **Self managed** → nhập master password
6. **Instance configuration**: `db.t3.micro` (Free tier)
7. **Storage**:
   - **Storage type**: `gp3` / `gp2`
   - **Allocated storage**: `20 GiB`
   - ⭐ **Storage autoscaling**: tick **Enable storage autoscaling** → **Maximum storage threshold**: `1000 GiB`
8. **Availability & durability**: (không có ở Free tier)
   - `Single-AZ DB instance deployment`
   - **`Multi-AZ DB instance deployment`** ⭐ (1 standby)
   - `Multi-AZ DB Cluster deployment` (2 readable standby)
9. **Connectivity**:
   - **VPC**: default
   - ⭐ **Public access**: `No` (khuyến nghị bảo mật) / `Yes` (để kết nối từ máy cá nhân)
   - **VPC security group**: tạo mới, mở port **3306**
10. **Database authentication**:
    - `Password authentication`
    - ⭐ **`Password and IAM database authentication`**
    - `Password and Kerberos authentication`
11. **Additional configuration**:
    - **Initial database name**: `mydb`
    - ⭐ **Backup retention period**: `7 days` (0 = tắt automated backup)
    - **Backup window**
    - ⭐ **Enable Enhanced monitoring**
    - **Log exports**: `Error log`, `General log`, `Slow query log` → gửi tới CloudWatch Logs
    - **Maintenance window**, **Auto minor version upgrade**
    - ⚠️ **Deletion protection** (nên bật cho production)
12. **Create database** → đợi status **`Creating` → `Available`** (~10 phút).

### Bước 2 — Khám phá các tính năng

1. Chọn database → tab **Connectivity & security**:
   - **Endpoint** (ví dụ `database-1.abc123.eu-west-3.rds.amazonaws.com`) ⭐ — đây là thứ ứng dụng dùng để kết nối
   - **Port**: `3306`
2. Tab **Monitoring** → xem các biểu đồ CloudWatch: CPU, DB connections, Free storage space, Read/Write IOPS…
3. Tab **Logs & events** → xem log và **Recent events**.
4. Tab **Maintenance & backups** → thấy **automated backups** và **snapshots**.

### Bước 3 — Tạo Read Replica ⭐

1. Chọn database → **Actions** → **Create read replica**.
2. Cấu hình:
   - **DB instance identifier**: `database-1-replica`
   - **Destination Region**: cùng Region hoặc Region khác (**Cross-Region Read Replica**)
   - **Multi-AZ deployment**: có thể bật cho replica
3. **Create read replica**.
4. Quan sát: replica có **endpoint RIÊNG** → ứng dụng phải **đổi connection string** để dùng nó ⭐

### Bước 4 — Bật Multi-AZ ⭐

1. Chọn database → **Modify**.
2. **Availability & durability** → **Multi-AZ DB instance deployment**.
3. **Continue** → chọn **Apply immediately** (hoặc chờ maintenance window).
4. Status: `Modifying` → `Available` — ⭐ **không có downtime**, endpoint **KHÔNG đổi**.

### Bước 5 — Các thao tác khác

| Thao tác | Vị trí |
|----------|--------|
| **Take snapshot** | Actions → Take snapshot |
| **Restore to point in time** ⭐ | Actions → Restore to point in time |
| **Stop temporarily** | Actions → Stop temporarily (tối đa **7 ngày**, sau đó tự start lại) |
| **Reboot** | Actions → Reboot |
| **Delete** | Actions → Delete (tắt Deletion protection trước) |

### Bước 6 — Dọn dẹp ⚠️

1. **Xóa Read Replica trước** (Actions → Delete).
2. **Xóa database chính**:
   - Nếu đã bật **Deletion protection** → **Modify** để tắt trước.
   - Delete → bỏ tick **Create final snapshot** (nếu không cần) → gõ `delete me` xác nhận.
3. Kiểm tra **Snapshots** → xóa snapshot còn sót (⚠️ **snapshot vẫn tính tiền**).

---

## 91. RDS Custom for Oracle and Microsoft SQL Server

### RDS Custom là gì? ⭐

- **Managed Oracle và Microsoft SQL Server Database VỚI khả năng tùy biến OS và database** ⭐⭐

> ⚠️ **Chỉ hỗ trợ Oracle và Microsoft SQL Server** — đây là điểm hay ra thi.

### RDS vs RDS Custom ⭐⭐

| | **RDS** | **RDS Custom** |
|---|---|---|
| Mức tự động | **Tự động hóa setup, operation, và scaling** của database trên AWS | **Cho phép truy cập database và OS bên dưới** để bạn áp dụng tùy biến |
| Ai quản lý | **TOÀN BỘ database và OS được AWS quản lý** | ⭐ **Bạn có FULL ADMIN ACCESS vào OS bên dưới và database** |
| SSH | ❌ **KHÔNG** | ✅ **CÓ** (SSH hoặc SSM Session Manager) |
| Engine hỗ trợ | 7 engine | ⭐ **CHỈ Oracle & Microsoft SQL Server** |

### Với RDS Custom, bạn có thể ⭐

- **Configure settings** (cấu hình thiết lập)
- **Install patches** (cài bản vá)
- **Enable native features** (bật các tính năng gốc của DB)
- ⭐ **Truy cập EC2 Instance bên dưới bằng SSH hoặc SSM Session Manager**

### Quy trình tùy biến ⭐

```
User ──apply customizations──► [Automation Mode DISABLED] ──SSH──► EC2 Instance
```

- ⭐ **PHẢI DE-ACTIVATE (tắt) Automation Mode** để thực hiện tùy biến
- ⭐ **Nên TAKE A DB SNAPSHOT trước khi tùy biến** (an toàn)

> **Mẹo thi:** Đề nhắc **"cần quyền admin trên OS của database"**, **"cài agent/patch riêng"**, **"Oracle/SQL Server"** → **RDS Custom**.

---

## 92. Amazon Aurora

### Aurora là gì? ⭐⭐

- **Aurora là công nghệ ĐỘC QUYỀN của AWS (KHÔNG mã nguồn mở)** ⭐
- **Hỗ trợ cả Postgres và MySQL làm Aurora DB** — nghĩa là **driver của bạn hoạt động y như Aurora là một database Postgres hoặc MySQL** ⭐
- **Aurora được "tối ưu cho AWS cloud"** và tuyên bố:
  - ⭐ **Hiệu năng cao hơn 5 LẦN so với MySQL trên RDS**
  - ⭐ **Hiệu năng cao hơn 3 LẦN so với Postgres trên RDS**
- ⭐ **Storage của Aurora TỰ ĐỘNG TĂNG theo bước 10GB, tối đa 256 TB**
- ⭐ **Aurora có thể có tới 15 replica**, và **quá trình replication NHANH HƠN MySQL (độ trễ replica dưới 10 ms)**
- ⭐ **Failover trong Aurora là TỨC THÌ (instantaneous)** — nó **HA (High Availability) native**
- ⭐ **Aurora tốn kém hơn RDS (đắt hơn 20%) — nhưng hiệu quả hơn**

### Các con số cần nhớ ⭐⭐

| Thông số | Giá trị |
|----------|---------|
| Hiệu năng vs MySQL trên RDS | **5x** |
| Hiệu năng vs Postgres trên RDS | **3x** |
| Storage tăng theo bước | **10 GB** |
| Storage tối đa | **256 TB** |
| Số Read Replica tối đa | **15** |
| Replica lag | **dưới 10 ms** |
| Chi phí so với RDS | **đắt hơn 20%** |

---

### Aurora High Availability and Read Scaling ⭐⭐⭐

#### Kiến trúc storage (rất hay ra thi)

- ⭐ **6 BẢN SAO dữ liệu trên 3 AZ**:
  - ⭐ **Cần 4 trong 6 bản sao để GHI (writes)**
  - ⭐ **Cần 3 trong 6 bản sao để ĐỌC (reads)**
- ⭐ **Tự phục hồi (self healing) với peer-to-peer replication**
- ⭐ **Storage được striped (phân dải) trên HÀNG TRĂM volume**

#### Kiến trúc compute

- ⭐ **MỘT Aurora Instance nhận WRITE (master)**
- ⭐ **Automated failover cho master TRONG DƯỚI 30 GIÂY**
- ⭐ **Master + tối đa 15 Aurora Read Replicas phục vụ READ**
- ⭐ **Hỗ trợ Cross Region Replication**

```
      AZ 1              AZ 2                    AZ 3
   ┌───────┐        ┌───────┬───────┐      ┌───────┬───────┐
   │   W   │        │   R   │   R   │      │   R   │   R   │
   └───┬───┘        └───┬───┴───┬───┘      └───┬───┴───┬───┘
       └────────────────┴───────┴──────────────┴───────┘
                              │
       ┌──────────────────────▼──────────────────────────┐
       │        Shared Storage Volume                     │
       │  6 bản sao / 3 AZ                                │
       │  Replication + Self Healing + Auto Expanding     │
       │  Ghi: cần 4/6   —   Đọc: cần 3/6                 │
       └──────────────────────────────────────────────────┘
```

---

### Aurora DB Cluster — Endpoints ⭐⭐

Đây là khái niệm **cực kỳ quan trọng**:

```
                          client
                         /      \
        Writer Endpoint /        \ Reader Endpoint
     (Pointing to the master)   (Connection Load Balancing)
              │                        │
              ▼                        ▼
           ┌─────┐        ┌────┬────┬────┬────┬────┐
           │  W  │        │ R  │ R  │ R  │ R  │ R  │  ◄── Auto Scaling
           └──┬──┘        └─┬──┴─┬──┴─┬──┴─┬──┴─┬──┘
              └─────────────┴────┴────┴────┴────┘
                            │
              ┌─────────────▼──────────────────────┐
              │      Shared Storage Volume          │
              │  Auto Expanding from 10G to 256 TB  │
              └─────────────────────────────────────┘
```

| Endpoint | Chức năng |
|----------|-----------|
| **Writer Endpoint** ⭐ | **Trỏ tới MASTER**. Khi master failover, endpoint này **tự động trỏ sang master mới** → ứng dụng **không cần đổi connection string** |
| **Reader Endpoint** ⭐ | **Load balancing kết nối** tới các Read Replica. Khi thêm/bớt replica, endpoint tự cập nhật |

> **Điểm mấu chốt:** Ứng dụng chỉ cần biết **2 endpoint**, không cần biết địa chỉ của từng instance.

---

### Features of Aurora ⭐

- **Automatic fail-over** (tự động chuyển đổi dự phòng)
- **Backup and Recovery**
- **Isolation and security**
- **Industry compliance**
- **Push-button scaling**
- ⭐ **Automated Patching with ZERO DOWNTIME**
- **Advanced Monitoring**
- **Routine Maintenance**
- ⭐⭐ **Backtrack: khôi phục dữ liệu về BẤT KỲ THỜI ĐIỂM NÀO mà KHÔNG cần dùng backup**

> **Backtrack là tính năng độc quyền của Aurora** — đề hỏi "quay ngược DB về thời điểm trước mà không restore từ backup" → **Aurora Backtrack**.

---

## 93. Amazon Aurora - Hands On

### Bước 1 — Tạo Aurora Cluster

1. RDS → **Create database** → **Standard create**.
2. **Engine options**: ⭐ **Aurora (MySQL Compatible)** hoặc **Aurora (PostgreSQL Compatible)**.
3. **Engine version**: chọn phiên bản.
4. **Templates**: `Dev/Test` (Aurora **không có Free tier** ⚠️).
5. **Settings**:
   - **DB cluster identifier**: `database-aurora`
   - **Master username / password**
6. **Cluster storage configuration**:
   - **Aurora Standard** (thông thường)
   - **Aurora I/O-Optimized** (khi I/O chiếm > 25% chi phí)
7. **Instance configuration**: `db.t3.medium` (rẻ nhất) hoặc `db.r6g.large`
8. ⭐ **Availability & durability** → **Multi-AZ deployment**:
   - `Don't create an Aurora Replica`
   - **`Create an Aurora Replica or Reader node in a different AZ`** (khuyến nghị)
9. **Connectivity**: VPC, security group (port **3306** cho MySQL / **5432** cho PostgreSQL).
10. **Create database** → đợi ~10–15 phút.

### Bước 2 — Quan sát cấu trúc Cluster ⭐

Sau khi tạo xong, trong danh sách **Databases** bạn thấy **cấu trúc phân cấp**:

```
database-aurora                      ← Regional cluster
├── database-aurora-instance-1       ← Role: Writer instance
└── database-aurora-instance-2       ← Role: Reader instance
```

Chọn cluster → tab **Connectivity & security** → thấy **các endpoint** ⭐:

| Endpoint | Type | Dùng cho |
|----------|------|----------|
| `database-aurora.cluster-abc.rds.amazonaws.com` | **Writer** | Ghi (trỏ tới master) |
| `database-aurora.cluster-ro-abc.rds.amazonaws.com` | **Reader** | Đọc (load balance các replica) |
| `database-aurora-instance-1.abc.rds.amazonaws.com` | **Instance** | Trỏ trực tiếp tới 1 instance |

> ⭐ Chú ý chữ **`cluster-ro-`** trong reader endpoint — `ro` = read only.

### Bước 3 — Thêm Reader Instance

1. Chọn cluster → **Actions** → **Add reader**.
2. Cấu hình instance class, AZ → **Add reader**.
3. → Reader endpoint **tự động** bao gồm instance mới, **không cần đổi gì ở ứng dụng** ⭐

### Bước 4 — Thử Failover ⭐⭐

1. Chọn **Writer instance** → **Actions** → **Failover**.
2. Quan sát:
   - Writer cũ → chuyển thành **Reader**
   - Reader cũ → được **promote thành Writer**
   - ⭐ **Writer endpoint tự động trỏ sang instance mới** — ứng dụng không cần làm gì
3. Thời gian failover: **dưới 30 giây**.

### Bước 5 — Xem các tính năng nâng cao

| Tính năng | Vị trí |
|-----------|--------|
| **Auto Scaling cho replica** | Cluster → Actions → **Add replica auto scaling** |
| **Custom Endpoint** | Cluster → tab Connectivity & security → Custom endpoints → **Create custom endpoint** |
| **Backtrack** (chỉ Aurora MySQL) | Cluster → Actions → **Backtrack** |
| **Clone** | Cluster → Actions → **Create clone** |
| **Cross-Region Read Replica** | Actions → **Add AWS Region** |
| **Global Database** | Actions → **Add AWS Region** (Global) |

### Bước 6 — Dọn dẹp ⚠️

> ⚠️ **Aurora RẤT ĐẮT** — nhớ xóa ngay sau khi thực hành!

1. **Xóa các Reader instance trước**.
2. **Xóa Writer instance** → cluster tự bị xóa (hoặc xóa cluster).
3. Tắt **Deletion protection** nếu đã bật.
4. Kiểm tra **Snapshots** → xóa snapshot còn sót.

---

## 94. Amazon Aurora - Advanced Concepts

---

### 1️⃣ Aurora Replicas — Auto Scaling ⭐

- Khi **nhiều request đổ vào Reader Endpoint** → **CPU Usage của các replica tăng**.
- Aurora **tự động thêm replica mới** (Replicas Auto Scaling).
- ⭐ **Reader Endpoint được MỞ RỘNG (Endpoint Extended)** để bao gồm các replica mới → **ứng dụng không cần thay đổi gì**.

```
   Client ──Many Requests──► Writer Endpoint
                          └► Reader Endpoint ──► [W] [R] [R] [R] [R]
                                  ▲ Endpoint Extended    ▲
                                                   Replicas Auto Scaling
                                                   (khi CPU Usage cao)
```

---

### 2️⃣ Aurora — Custom Endpoints ⭐⭐

- **Định nghĩa MỘT TẬP CON các Aurora Instance làm một Custom Endpoint** ⭐
- **Ví dụ: chạy các truy vấn phân tích (analytical queries) trên các replica CỤ THỂ**
- ⭐ **Reader Endpoint NÓI CHUNG KHÔNG CÒN ĐƯỢC DÙNG sau khi định nghĩa Custom Endpoints**

```
   Client ──Queries──► Writer Endpoint ──► [W]

          ──Analytical Queries──► Custom Endpoint ──► [R] [R]  (db.r5.2xlarge)

                       Reader Endpoint ──────────────► [R] [R]  (db.r3.large)
```

> **Use case điển hình:** replica nhỏ (`db.r3.large`) phục vụ traffic thường, replica lớn (`db.r5.2xlarge`) gom vào Custom Endpoint để chạy báo cáo nặng.

---

### 3️⃣ Aurora Serverless ⭐⭐

- ⭐ **Tự động khởi tạo database và auto-scaling dựa trên mức sử dụng THỰC TẾ**
- ⭐ **Tốt cho workload KHÔNG THƯỜNG XUYÊN (infrequent), NGẮT QUÃNG (intermittent) hoặc KHÔNG ĐOÁN TRƯỚC ĐƯỢC (unpredictable)**
- ⭐ **KHÔNG cần capacity planning**
- ⭐ **Trả tiền THEO GIÂY (pay per second)**, **có thể tiết kiệm chi phí hơn**

```
   Client ──► Proxy Fleet (managed by Aurora) ──► Shared storage Volume
```

> **Mẹo thi:** "workload không đoán trước", "không muốn capacity planning", "trả theo giây" → **Aurora Serverless**.

---

### 4️⃣ Global Aurora ⭐⭐⭐

#### Aurora Cross Region Read Replicas

- **Hữu ích cho disaster recovery**
- **Đơn giản để triển khai**

#### Aurora Global Database (⭐ ĐƯỢC KHUYẾN NGHỊ)

| Đặc điểm | Giá trị |
|----------|---------|
| **Primary Region** | ⭐ **1 Region chính (read / write)** |
| **Secondary Regions** | ⭐ **Tối đa 10 Region phụ (chỉ đọc)**, **replication lag DƯỚI 1 GIÂY** |
| **Read Replicas mỗi Region phụ** | ⭐ **Tối đa 16** |
| **Lợi ích** | **Giảm độ trễ (latency)** cho người dùng toàn cầu |
| **Promote Region khác (cho DR)** | ⭐ **RTO < 1 PHÚT** |
| **Thời gian replication cross-region điển hình** | ⭐ **dưới 1 giây** |

```
   us-east-1 - PRIMARY region          eu-west-1 - SECONDARY region
   Applications: Read / Write   ──replication──►   Applications: Read Only
                                (< 1 giây)
```

### Các con số Global Aurora cần thuộc ⭐

- **1** primary region
- **10** secondary regions
- **16** read replicas per secondary region
- **< 1 giây** replication lag
- **< 1 phút** RTO khi promote

---

### 5️⃣ Aurora Machine Learning ⭐

- ⭐ **Cho phép thêm dự đoán dựa trên ML vào ứng dụng của bạn THÔNG QUA SQL**
- **Tích hợp đơn giản, tối ưu và bảo mật giữa Aurora và các dịch vụ AWS ML**
- ⭐ **Các dịch vụ được hỗ trợ:**

| Dịch vụ | Dùng cho |
|---------|----------|
| **Amazon SageMaker** | **Dùng với BẤT KỲ mô hình ML nào** |
| **Amazon Comprehend** | ⭐ **Phân tích cảm xúc (sentiment analysis)** |

- ⭐ **Bạn KHÔNG cần có kinh nghiệm về ML**
- ⭐ **Use cases: fraud detection, ads targeting, sentiment analysis, product recommendations**

```
   Application ──SQL query (Recommended products?)──► Amazon Aurora
                                                          │  data (user profile, shopping history)
                                                          ▼
                                            Amazon SageMaker / Amazon Comprehend
                                                          │  predictions (red shirt, blue pants)
   Application ◄──query results (red shirt, blue …)───────┘
```

---

### 6️⃣ Babelfish for Aurora PostgreSQL ⭐⭐

- ⭐ **Cho phép Aurora PostgreSQL HIỂU các lệnh dành cho MS SQL Server** (ví dụ **T-SQL**)
- ⭐ **Do đó, ứng dụng dựa trên Microsoft SQL Server CÓ THỂ chạy trên Aurora PostgreSQL**
- ⭐ **Yêu cầu KHÔNG hoặc RẤT ÍT thay đổi code** (vẫn dùng chính **MS SQL Server client driver** cũ)
- **Cùng một ứng dụng có thể được dùng sau khi migrate database** (dùng **AWS SCT** và **DMS**)

```
   Application                          Application
   SQL Server Client Driver             PostgreSQL Driver
          │ T-SQL                              │ PL/pgSQL
          ▼                                    ▼
      ┌────────────────── Aurora PostgreSQL ──────────────────┐
      │  T-SQL ──► Babelfish ──► PostgreSQL                    │
      └────────────────────────────────────────────────────────┘
                          ▲ migrate (AWS SCT + DMS)
```

> **Mẹo thi:** "migrate ứng dụng SQL Server sang Aurora mà không sửa code" → **Babelfish**.

---

### 7️⃣ Aurora Database Cloning ⭐⭐

- ⭐ **Tạo một Aurora DB Cluster MỚI từ một cluster đã có**
- ⭐ **NHANH HƠN snapshot & restore**
- ⭐ **Dùng giao thức COPY-ON-WRITE**:
  - **Ban đầu, cluster mới dùng CHUNG data volume với cluster gốc** (nhanh và hiệu quả — **không cần copy gì cả**)
  - **Khi có cập nhật trên cluster mới, storage bổ sung mới được cấp phát và dữ liệu mới được copy để tách ra**
- ⭐ **RẤT NHANH & TIẾT KIỆM CHI PHÍ**
- ⭐ **Hữu ích để tạo database "STAGING" từ database "PRODUCTION" mà KHÔNG ẢNH HƯỞNG tới production**

```
   Production Aurora ──clone (copy-on-write)──► Staging Aurora
```

> **So sánh 3 cách tạo bản sao:**
> | Cách | Tốc độ | Ảnh hưởng production | Use case |
> |---|---|---|---|
> | **Snapshot & Restore** | Chậm | Không | Backup/khôi phục |
> | **Read Replica** | Nhanh | Nhẹ | Scale đọc |
> | **Database Cloning** ⭐ | **Nhanh nhất** | **Không** | **Tạo môi trường staging/test** |

---

## 95. RDS & Aurora - Backup and Monitoring

---

### RDS Backups ⭐⭐

#### Automated backups

- ⭐ **Full backup HÀNG NGÀY của database** (trong **backup window**)
- ⭐ **Transaction logs được backup bởi RDS MỖI 5 PHÚT**
- ⭐⭐ **=> Khả năng RESTORE VỀ BẤT KỲ THỜI ĐIỂM NÀO** (từ backup cũ nhất tới **5 phút trước**)
- ⭐ **Retention 1 đến 35 ngày**, **đặt 0 để TẮT automated backups**

#### Manual DB Snapshots

- **Được kích hoạt THỦ CÔNG bởi người dùng**
- ⭐ **Giữ backup BAO LÂU TÙY BẠN**

#### ⭐⭐ Mẹo quan trọng về chi phí (từ slide):

> **Với một RDS database đã STOPPED, bạn VẪN PHẢI TRẢ TIỀN cho storage.**
> **Nếu định dừng nó trong thời gian dài, bạn nên SNAPSHOT & RESTORE thay vì stop.**

---

### Aurora Backups ⭐

#### Automated backups

- ⭐ **1 đến 35 ngày — KHÔNG THỂ TẮT (cannot be disabled)** ⭐⭐
- **Point-in-time recovery trong khoảng thời gian đó**

#### Manual DB Snapshots

- **Được kích hoạt thủ công bởi người dùng**
- **Giữ backup bao lâu tùy bạn**

### ⭐ Khác biệt RDS vs Aurora về backup:

| | **RDS** | **Aurora** |
|---|---|---|
| Automated backup retention | **1–35 ngày** | **1–35 ngày** |
| **Có thể TẮT automated backup?** | ✅ **CÓ (đặt 0)** | ❌ **KHÔNG THỂ TẮT** ⭐ |

---

### RDS & Aurora Restore options ⭐

- ⭐ **Restore một RDS/Aurora backup hoặc snapshot sẽ TẠO RA MỘT DATABASE MỚI** (không ghi đè lên DB cũ)

#### Restoring MySQL RDS database từ S3 ⭐

1. **Tạo backup của on-premises database của bạn**
2. **Lưu nó trên Amazon S3** (object storage)
3. **Restore file backup lên một RDS instance MỚI chạy MySQL**

#### Restoring MySQL Aurora cluster từ S3 ⭐

1. **Tạo backup của on-premises database bằng Percona XtraBackup** ⭐⭐
2. **Lưu file backup trên Amazon S3**
3. **Restore file backup lên một Aurora cluster MỚI chạy MySQL**

> **Bẫy thi:** Công cụ backup để migrate on-premises MySQL sang **Aurora** là **Percona XtraBackup**. Đây là chi tiết rất hay được hỏi.

---

### Monitoring (bổ sung)

| Công cụ | Mục đích |
|---------|----------|
| **CloudWatch Metrics** | CPU, connections, free storage, IOPS, replica lag… |
| **Enhanced Monitoring** ⭐ | **Metric ở cấp OS** (process list, chi tiết CPU) — granularity tới 1 giây |
| **Performance Insights** ⭐ | **Phân tích câu query nào gây tải**, theo waits / SQL / host / user |
| **CloudWatch Logs** | Export error log, slow query log, general log, audit log |
| **Events / Event Subscriptions** | Thông báo qua **SNS** khi có sự kiện (failover, backup, deletion…) |

---

## 96. RDS Security

### RDS & Aurora Security ⭐⭐

#### 1. At-rest encryption (mã hóa khi lưu trữ)

- ⭐ **Mã hóa database master & replicas bằng AWS KMS — PHẢI được định nghĩa TẠI THỜI ĐIỂM KHỞI TẠO (launch time)** ⭐⭐
- ⭐ **Nếu MASTER KHÔNG được mã hóa, thì các read replica KHÔNG THỂ được mã hóa**
- ⭐⭐ **Để mã hóa một database CHƯA được mã hóa: phải đi qua DB SNAPSHOT & RESTORE dưới dạng đã mã hóa**

> Quy trình này **giống hệt** với cách mã hóa EBS volume ở phần 7: **snapshot → copy/restore có mã hóa → dùng bản mới**.

#### 2. In-flight encryption (mã hóa khi truyền)

- ⭐ **TLS-ready MẶC ĐỊNH**, dùng **AWS TLS root certificates ở phía client**

#### 3. IAM Authentication ⭐

- ⭐ **Dùng IAM roles để kết nối tới database THAY VÌ username/password**

#### 4. Security Groups ⭐

- ⭐ **Kiểm soát truy cập MẠNG tới RDS / Aurora DB của bạn**

#### 5. SSH

- ⭐⭐ **KHÔNG có SSH — NGOẠI TRỪ trên RDS Custom**

#### 6. Audit Logs ⭐

- ⭐ **Có thể bật Audit Logs và gửi tới CloudWatch Logs để lưu giữ lâu hơn**

---

### Bảng tổng hợp RDS Security ⭐

| Lớp bảo mật | Cơ chế | Ghi chú quan trọng |
|-------------|--------|--------------------|
| **Encryption at rest** | **AWS KMS** | ⭐ Phải bật **lúc launch**; master chưa mã hóa → replica không mã hóa được |
| **Encryption in flight** | **TLS** | Sẵn sàng mặc định |
| **Authentication** | Password / **IAM** / Kerberos | IAM = không cần lưu password |
| **Network** | **Security Groups** | Kiểm soát ai kết nối được |
| **OS access** | ❌ Không SSH | ⭐ Trừ **RDS Custom** |
| **Audit** | **Audit Logs → CloudWatch Logs** | Lưu giữ dài hạn |

---

## 97. RDS Proxy

### Amazon RDS Proxy là gì? ⭐⭐

- ⭐ **Database proxy được QUẢN LÝ HOÀN TOÀN (fully managed) cho RDS**
- ⭐ **Cho phép ứng dụng POOL (gộp) và SHARE (chia sẻ) các kết nối DB** đã thiết lập với database

### Lợi ích ⭐⭐

| Lợi ích | Chi tiết |
|---------|----------|
| **Tăng hiệu quả database** | **Giảm áp lực lên tài nguyên DB** (ví dụ **CPU, RAM**) và **giảm thiểu số kết nối mở (và timeout)** ⭐ |
| **Kiến trúc** | ⭐ **Serverless, autoscaling, highly available (multi-AZ)** |
| **Failover** | ⭐⭐ **GIẢM thời gian failover của RDS & Aurora TỚI 66%** |
| **Engine hỗ trợ** | ⭐ **RDS (MySQL, PostgreSQL, MariaDB, MS SQL Server)** và **Aurora (MySQL, PostgreSQL)** |
| **Code** | ⭐ **KHÔNG cần thay đổi code với hầu hết ứng dụng** |
| **Bảo mật** | ⭐ **Bắt buộc IAM Authentication cho DB**, và **lưu credentials an toàn trong AWS Secrets Manager** |
| **Truy cập** | ⭐⭐ **RDS Proxy KHÔNG BAO GIỜ truy cập được công khai (never publicly accessible)** — **phải truy cập TỪ TRONG VPC** |

### Sơ đồ

```
┌──────────────────────── VPC ────────────────────────┐
│                                                      │
│   Lambda functions ──IAM Authentication──►           │
│         …                                            │
│                        ┌──── Private subnet ────┐    │
│                        │  RDS Proxy ──► RDS DB  │    │
│                        │                Instance│    │
│                        └────────────────────────┘    │
└──────────────────────────────────────────────────────┘
```

### ⭐⭐ Use case kinh điển: Lambda + RDS

> **Vấn đề:** Hàng nghìn Lambda function chạy đồng thời → mỗi function mở một kết nối DB → **database bị quá tải kết nối, hết RAM, timeout**.
>
> **Giải pháp:** **RDS Proxy** đứng giữa, **gộp và tái sử dụng kết nối** → database chỉ thấy một số ít kết nối ổn định.

### Mẹo thi ⭐

| Từ khóa trong đề | Đáp án |
|------------------|--------|
| "quá nhiều **kết nối DB**", "connection timeout", "Lambda + RDS" | **RDS Proxy** |
| "giảm **failover time** tới 66%" | **RDS Proxy** |
| "bắt buộc **IAM Authentication** cho database" | **RDS Proxy** |
| "proxy có public endpoint không?" | ❌ **KHÔNG** — chỉ trong VPC |

---

## 98. ElastiCache Overview

### ElastiCache là gì? ⭐

- ⭐ **Giống như RDS là để có Relational Database được quản lý… thì ElastiCache là để có REDIS hoặc MEMCACHED được quản lý**
- ⭐ **Cache là in-memory database với hiệu năng RẤT CAO, độ trễ THẤP**
- ⭐ **Giúp giảm tải cho database với các workload đọc nhiều (read intensive)**
- ⭐⭐ **Giúp làm cho ứng dụng của bạn TRỞ NÊN STATELESS (không trạng thái)**
- **AWS lo: OS maintenance/patching, optimizations, setup, configuration, monitoring, failure recovery và backups**
- ⚠️ ⭐ **Sử dụng ElastiCache đòi hỏi THAY ĐỔI CODE ỨNG DỤNG NHIỀU (heavy application code changes)**

> **Bẫy thi:** ElastiCache **không phải** giải pháp "cắm vào là chạy". Nếu đề nói "không muốn sửa code" → ElastiCache **không phải** đáp án.

---

### Solution Architecture 1 — DB Cache ⭐⭐

```
                        ┌── Cache hit ──► trả về ngay
   application ──► Amazon ElastiCache
                        └── Cache miss ──► Read from DB ──► Amazon RDS
                                    ◄── Write to cache ────┘
```

- ⭐ **Ứng dụng truy vấn ElastiCache; nếu không có, lấy từ RDS rồi LƯU vào ElastiCache**
- ⭐ **Giúp giảm tải cho RDS**
- ⚠️ ⭐⭐ **Cache PHẢI có một CHIẾN LƯỢC INVALIDATION (làm mất hiệu lực) để đảm bảo chỉ dữ liệu MỚI NHẤT được dùng trong đó**

---

### Solution Architecture 2 — User Session Store ⭐⭐

```
   User ──login──► [application instance 1] ──Write session──► Amazon ElastiCache
                                                                       │
   User ──hit another instance──► [application instance 2] ◄──Retrieve session──┘
                                  → user đã đăng nhập sẵn!
```

Luồng hoạt động:
1. **User đăng nhập vào bất kỳ instance nào của ứng dụng**
2. **Ứng dụng GHI dữ liệu session vào ElastiCache**
3. **User truy cập vào một instance KHÁC của ứng dụng**
4. ⭐ **Instance đó LẤY dữ liệu session ra và user đã ở trạng thái đăng nhập sẵn**

> **Đây chính là cách làm ứng dụng "stateless"** — bất kỳ instance nào cũng phục vụ được bất kỳ user nào. Đây là **giải pháp thay thế cho Sticky Sessions** ở phần 8.

---

### ⭐⭐⭐ ElastiCache — Redis vs Memcached (BẢNG QUAN TRỌNG NHẤT)

| **REDIS** | **MEMCACHED** |
|-----------|---------------|
| ⭐ **Multi-AZ với Auto-Failover** | ⭐ **Multi-node cho PARTITIONING dữ liệu (sharding)** |
| ⭐ **Read Replicas để scale reads và có high availability** | ⭐ **KHÔNG có high availability (không replication)** |
| ⭐ **Data Durability nhờ AOF persistence** | ⭐ **KHÔNG bền vững (non persistent)** |
| ⭐ **Backup and restore features** | **Backup and restore (Serverless)** |
| ⭐ **Hỗ trợ Sets và Sorted Sets** | ⭐ **Kiến trúc ĐA LUỒNG (multi-threaded)** |
| → **Replication** | → **Sharding** |

### Bảng so sánh dạng đối chiếu ⭐

| Tiêu chí | **Redis** | **Memcached** |
|----------|-----------|---------------|
| **High Availability** | ✅ **Multi-AZ + Auto-Failover** | ❌ **Không** |
| **Replication** | ✅ **Read Replicas** | ❌ Không |
| **Sharding / Partitioning** | (có Cluster Mode) | ✅ **Multi-node sharding** |
| **Persistence (bền vững)** | ✅ **AOF persistence** | ❌ **Non persistent** |
| **Backup & Restore** | ✅ Có | Có (Serverless) |
| **Kiểu dữ liệu nâng cao** | ✅ **Sets, Sorted Sets** | ❌ Chỉ key-value đơn giản |
| **Đa luồng** | ❌ (đơn luồng) | ✅ **Multi-threaded** |
| **Từ khóa nhận diện** | **HA, failover, persistence, leaderboard** | **sharding, multi-threaded, cache thuần** |

### Mẹo nhớ ⭐

```
REDIS     → cần HA, cần bền vững, cần cấu trúc dữ liệu phức tạp
MEMCACHED → chỉ cần cache đơn thuần, sharding, đa luồng, mất dữ liệu cũng OK
```

---

## 99. ElastiCache Hands On

### Bước 1 — Tạo ElastiCache cluster

1. Console → tìm **ElastiCache**.
2. Menu trái → chọn **Redis OSS caches** (hoặc **Memcached caches**).
3. **Create Redis OSS cache**:
   - **Deployment option**:
     - ⭐ **Serverless** (AWS tự quản lý capacity)
     - **Design your own cache** → **Cluster cache** (tự chọn node type)
   - **Creation method**: `Easy create` / `Configure and create new cluster`
4. **Cluster settings**:
   - **Name**: `my-redis-cache`
   - **Node type**: `cache.t3.micro` (rẻ nhất)
   - ⭐ **Number of replicas**: `0` (để tiết kiệm khi học) hoặc `1–5`
   - ⭐ **Cluster mode**: `Disabled` (một shard) / `Enabled` (nhiều shard)
   - ⭐ **Multi-AZ**: bật để có auto-failover (cần ≥ 1 replica)
5. **Connectivity**:
   - **Subnet groups**: tạo mới trong VPC của bạn
   - ⚠️ **Security group**: mở port ⭐ **6379** (Redis) hoặc ⭐ **11211** (Memcached)
6. **Security**:
   - ⭐ **Encryption at rest** (KMS)
   - ⭐ **Encryption in transit** (TLS)
   - ⭐ **Redis AUTH** — đặt token/password
7. **Backup**: bật/tắt automatic backups, retention period.
8. **Maintenance**: maintenance window, SNS notifications.
9. **Create** → đợi status `creating` → **`available`** (~5–10 phút).

### Bước 2 — Xem thông tin kết nối

Chọn cluster → xem:

| Thông tin | Ví dụ |
|-----------|-------|
| **Primary endpoint** ⭐ | `my-redis-cache.abc123.ng.0001.euw3.cache.amazonaws.com:6379` |
| **Reader endpoint** | `my-redis-cache-ro.abc123...` (khi có replica) |
| **Configuration endpoint** | (khi Cluster Mode Enabled) |

### Bước 3 — Kết nối thử từ EC2 ⭐

> ⚠️ **ElastiCache KHÔNG truy cập được từ internet** — bắt buộc kết nối **từ bên trong VPC** (giống RDS Proxy).

SSH vào một EC2 instance trong cùng VPC:

```bash
# Cài Redis CLI
sudo yum install -y redis6      # Amazon Linux 2023
# hoặc: sudo amazon-linux-extras install redis6

# Kết nối (không TLS)
redis-cli -h my-redis-cache.abc123.ng.0001.euw3.cache.amazonaws.com -p 6379

# Kết nối có TLS + AUTH
redis-cli -h <endpoint> -p 6379 --tls -a <auth-token>
```

Thử vài lệnh Redis:

```
SET mykey "Hello ElastiCache"
GET mykey
TTL mykey
EXPIRE mykey 60
INFO replication
```

### Bước 4 — Thử Sorted Set (chuẩn bị cho bài 100)

```
ZADD leaderboard 100 "player1"
ZADD leaderboard 250 "player2"
ZADD leaderboard 175 "player3"
ZREVRANGE leaderboard 0 -1 WITHSCORES
```

→ Kết quả trả về **đã được xếp hạng tự động**: `player2 (250)`, `player3 (175)`, `player1 (100)`.

### Bước 5 — Dọn dẹp ⚠️

1. ElastiCache → chọn cluster → **Actions** → **Delete**.
2. Chọn **không tạo final backup** (nếu không cần) → xác nhận.
3. Xóa **Subnet group** và **Security group** đã tạo (tùy chọn).

---

## 100. ElastiCache for Solution Architects

---

### Patterns for ElastiCache ⭐⭐⭐

Ba mẫu thiết kế cần nắm:

| Pattern | Mô tả |
|---------|-------|
| **Lazy Loading** ⭐ | **TOÀN BỘ dữ liệu đọc được cache lại**, ⚠️ **dữ liệu CÓ THỂ trở nên CŨ (stale) trong cache** |
| **Write Through** ⭐ | **THÊM hoặc CẬP NHẬT dữ liệu trong cache NGAY KHI ghi vào DB** → ⭐ **KHÔNG có dữ liệu cũ (no stale data)** |
| **Session Store** ⭐ | **Lưu dữ liệu session tạm thời trong cache** (dùng **tính năng TTL**) |

```
                        ┌── Cache hit ──► trả về
   application ──► Amazon ElastiCache
                        └── Cache miss ──► Read from DB ──► Amazon RDS
                                    ◄── Write to cache ────┘
              (Lazy Loading illustrated)
```

### So sánh Lazy Loading vs Write Through ⭐

| | **Lazy Loading** | **Write Through** |
|---|---|---|
| **Khi nào ghi vào cache** | Khi **cache miss** (đọc từ DB rồi mới ghi) | Khi **ghi vào DB** |
| **Dữ liệu stale** | ⚠️ **CÓ THỂ bị cũ** | ✅ **KHÔNG bị cũ** |
| **Cache chứa gì** | **Chỉ dữ liệu được đọc** (tiết kiệm bộ nhớ) | **Mọi dữ liệu được ghi** (kể cả không ai đọc) |
| **Độ trễ lần đọc đầu** | Chậm (cache miss) | Nhanh (đã có sẵn) |
| **Cache miss penalty** | Có | Ít |

### 💬 Câu nói kinh điển (từ slide)

> **"There are only two hard things in Computer Science: cache invalidation and naming things."**
> *(Chỉ có hai việc khó trong Khoa học Máy tính: làm mất hiệu lực cache và đặt tên.)*

---

### ElastiCache — Cache Security ⭐⭐

| Cơ chế | Chi tiết |
|--------|----------|
| **IAM Authentication** | ⭐ **ElastiCache hỗ trợ IAM Authentication CHO REDIS**<br>⚠️ ⭐⭐ **IAM policies trên ElastiCache CHỈ được dùng cho bảo mật ở CẤP AWS API** — không dùng để xác thực vào cache |
| **Redis AUTH** ⭐ | • **Bạn có thể đặt một "password/token" khi tạo Redis cluster**<br>• **Đây là lớp bảo mật BỔ SUNG cho cache** (trên nền security groups)<br>• ⭐ **Hỗ trợ SSL in-flight encryption** |
| **Memcached** | ⭐ **Hỗ trợ SASL-based authentication** (nâng cao) |

```
   Client ──► EC2 (EC2 Security group)
                    │  SSL encryption
                    │  Redis AUTH
                    ▼
              Redis (Redis Security group)
```

> **Bẫy thi:** "IAM policy có kiểm soát được ai đọc/ghi vào Redis không?" → **KHÔNG**. IAM chỉ kiểm soát **AWS API** (tạo/xóa cluster). Muốn bảo vệ dữ liệu trong cache → **Redis AUTH**.

---

### ElastiCache — Redis Use Case: Gaming Leaderboards ⭐⭐

- ⭐ **Bảng xếp hạng game (Gaming Leaderboards) có độ phức tạp tính toán cao**
- ⭐⭐ **Redis SORTED SETS đảm bảo CẢ tính DUY NHẤT (uniqueness) VÀ THỨ TỰ phần tử (element ordering)**
- ⭐ **Mỗi khi một phần tử mới được thêm vào, nó được XẾP HẠNG THEO THỜI GIAN THỰC, rồi được thêm vào ĐÚNG THỨ TỰ**

```
   Clients ──► ElastiCache for Redis ──┐
           ──► ElastiCache for Redis ──┼──► Real-time Leaderboard  1. / 2. / 3.
           ──► ElastiCache for Redis ──┘
```

> **Mẹo thi:** Đề nhắc **"leaderboard"**, **"real-time ranking"**, **"uniqueness + ordering"** → **Redis Sorted Sets**.

---

## 101. List of Ports to be familiar with

> Đây là bài **article** (bài viết) — giảng viên tổng hợp danh sách các port cần thuộc lòng cho kỳ thi.

### ⭐⭐ Important Ports (Port thông dụng)

| Port | Giao thức / Dịch vụ |
|------|---------------------|
| **21** | **FTP** |
| **22** | **SSH** |
| **22** | **SFTP** (chạy trên nền SSH) |
| **80** | **HTTP** |
| **443** | **HTTPS** |
| **3389** | **RDP** (Remote Desktop Protocol — Windows) |

### ⭐⭐⭐ RDS Database Ports (Port cơ sở dữ liệu) — RẤT HAY RA THI

| Port | Database |
|------|----------|
| **5432** | **PostgreSQL** ⭐ |
| **3306** | **MySQL** ⭐ |
| **3306** | **MariaDB** (giống MySQL) |
| **1521** | **Oracle RDS** ⭐ |
| **1433** | **MS SQL Server** ⭐ |
| **50000** | **IBM DB2** |
| **5439** | **Amazon Redshift** ⭐ (không thuộc RDS nhưng hay hỏi cùng) |

### Port của các dịch vụ cache (bổ sung, hay dùng trong phần này)

| Port | Dịch vụ |
|------|---------|
| **6379** | **Redis / ElastiCache for Redis** ⭐ |
| **11211** | **Memcached / ElastiCache for Memcached** ⭐ |
| **2049** | **NFS / EFS** ⭐ (từ phần 7) |

### ⚠️ Lưu ý quan trọng từ giảng viên

> **ĐỪNG NHẦM LẪN giữa port của database và port thông dụng.**
> Đặc biệt dễ nhầm: **MySQL 3306** vs **PostgreSQL 5432** vs **Oracle 1521** vs **MS SQL 1433**.

### Mẹo nhớ ⭐

```
3306  MySQL/MariaDB    → "33" như hai chữ M úp ngược
5432  PostgreSQL       → dãy số đếm ngược 5-4-3-2
1521  Oracle           → Oracle "15-21"
1433  MS SQL Server    → Microsoft "14-33"
5439  Redshift         → giống Postgres (5432) nhưng đổi đuôi (Redshift dựa trên Postgres!)
6379  Redis            → "MERZ" trên bàn phím điện thoại
11211 Memcached        → 11-2-11
```

> ⭐ Ghi nhớ hay: **Redshift (5439)** gần với **PostgreSQL (5432)** vì **Redshift được xây dựng dựa trên PostgreSQL**.

---

## Trắc nghiệm 6: RDS, Aurora, & ElastiCache Quiz

### Các điểm dễ bị bẫy

| Câu hỏi thường gặp | Đáp án đúng |
|--------------------|-------------|
| RDS hỗ trợ mấy engine? | **7**: Postgres, MySQL, MariaDB, Oracle, MS SQL Server, IBM DB2, Aurora |
| Có SSH vào RDS được không? | ❌ **KHÔNG** — trừ **RDS Custom** |
| RDS storage nằm trên gì? | **EBS** |
| Storage Auto Scaling kích hoạt khi nào? | **Free storage < 10%**, kéo dài **≥ 5 phút**, **6 giờ** kể từ lần đổi trước |
| Storage Auto Scaling bắt buộc đặt gì? | **Maximum Storage Threshold** |
| Read Replicas tối đa? | **15** |
| Read Replicas replication kiểu gì? | **ASYNC** → eventually consistent |
| Multi-AZ replication kiểu gì? | **SYNC** |
| Multi-AZ dùng để scaling không? | ❌ **KHÔNG** — chỉ cho **DR/HA** |
| Standby của Multi-AZ có phục vụ đọc? | ❌ **KHÔNG** |
| Read Replica có đổi connection string? | ✅ **CÓ** — Multi-AZ thì **KHÔNG** (một DNS name) |
| Read replica cùng Region khác AZ tốn phí mạng? | ❌ **KHÔNG** |
| Read replica **Cross-Region** tốn phí? | ✅ **CÓ** |
| Read replica chạy được lệnh gì? | **CHỈ SELECT** (không INSERT/UPDATE/DELETE) |
| Chuyển Single-AZ → Multi-AZ có downtime? | ❌ **KHÔNG** (zero downtime) |
| RDS Custom hỗ trợ engine nào? | ⭐ **CHỈ Oracle và MS SQL Server** |
| RDS Custom cần làm gì trước khi tùy biến? | **Tắt Automation Mode** + **take snapshot** |
| Aurora nhanh hơn MySQL/Postgres trên RDS bao nhiêu? | **5x / 3x** |
| Aurora storage tăng theo bước bao nhiêu, tối đa? | **10 GB**, tối đa **256 TB** |
| Aurora giữ mấy bản sao trên mấy AZ? | ⭐ **6 bản sao trên 3 AZ** |
| Aurora cần bao nhiêu bản sao để ghi / đọc? | ⭐ **4/6 để GHI**, **3/6 để ĐỌC** |
| Aurora failover master mất bao lâu? | **dưới 30 giây** |
| Aurora replica lag? | **dưới 10 ms** |
| Aurora đắt hơn RDS bao nhiêu? | **20%** |
| Aurora có 2 endpoint chính nào? | **Writer Endpoint** và **Reader Endpoint** |
| Reader Endpoint làm gì? | **Connection load balancing** giữa các replica |
| Tính năng khôi phục Aurora không dùng backup? | ⭐ **Backtrack** |
| Sau khi tạo Custom Endpoint thì Reader Endpoint? | **Thường không còn được dùng** |
| Aurora Serverless dùng cho workload nào? | **Không thường xuyên, ngắt quãng, không đoán trước** |
| Aurora Global Database: mấy secondary region? | ⭐ **Tối đa 10** |
| Aurora Global Database: replication lag? | ⭐ **< 1 giây** |
| Aurora Global Database: RTO khi promote? | ⭐ **< 1 phút** |
| Read replicas mỗi secondary region? | ⭐ **Tối đa 16** |
| Aurora ML tích hợp dịch vụ nào? | **SageMaker** (mọi model) và **Comprehend** (sentiment analysis) |
| Chạy ứng dụng SQL Server trên Aurora PostgreSQL? | ⭐ **Babelfish** |
| Tạo staging DB từ production nhanh nhất? | ⭐ **Aurora Database Cloning** (copy-on-write) |
| RDS automated backup retention? | **1–35 ngày**, **0 = tắt** |
| Aurora automated backup retention? | **1–35 ngày**, ⭐ **KHÔNG THỂ TẮT** |
| RDS backup transaction logs mỗi bao lâu? | ⭐ **5 phút** |
| Restore về thời điểm gần nhất là bao lâu trước? | **5 phút trước** |
| RDS đã stop có tính tiền không? | ⭐ **CÓ — vẫn trả tiền storage** → nên **snapshot & restore** |
| Restore backup tạo ra cái gì? | ⭐ **Một database MỚI** |
| Công cụ backup để migrate on-prem MySQL sang Aurora? | ⭐ **Percona XtraBackup** |
| Mã hóa RDS phải bật khi nào? | ⭐ **Lúc launch time** |
| Master chưa mã hóa thì replica? | ⭐ **KHÔNG mã hóa được** |
| Mã hóa DB chưa mã hóa làm sao? | ⭐ **Snapshot & restore dưới dạng đã mã hóa** |
| RDS Proxy giảm failover time bao nhiêu? | ⭐ **Tới 66%** |
| RDS Proxy có public endpoint không? | ❌ ⭐ **KHÔNG BAO GIỜ** — chỉ từ VPC |
| RDS Proxy giải quyết vấn đề gì? | ⭐ **Quá nhiều kết nối DB** (Lambda + RDS) |
| RDS Proxy lưu credentials ở đâu? | **AWS Secrets Manager** |
| ElastiCache giúp ứng dụng thành gì? | ⭐ **Stateless** |
| Dùng ElastiCache có cần sửa code? | ⚠️ ⭐ **CÓ — nhiều** |
| Redis vs Memcached: cái nào có Multi-AZ + failover? | ⭐ **Redis** |
| Redis vs Memcached: cái nào có sharding multi-node? | ⭐ **Memcached** |
| Redis vs Memcached: cái nào multi-threaded? | ⭐ **Memcached** |
| Redis vs Memcached: cái nào persistent (AOF)? | ⭐ **Redis** |
| Redis vs Memcached: cái nào có Sorted Sets? | ⭐ **Redis** |
| Leaderboard game dùng gì? | ⭐ **Redis Sorted Sets** |
| Pattern nào có thể gây stale data? | ⭐ **Lazy Loading** |
| Pattern nào không có stale data? | ⭐ **Write Through** |
| IAM policy trên ElastiCache dùng cho gì? | ⭐ **CHỈ bảo mật ở cấp AWS API** |
| Bảo vệ dữ liệu trong Redis cache bằng gì? | ⭐ **Redis AUTH** (+ SSL) |
| Memcached xác thực bằng gì? | **SASL-based authentication** |
| Port MySQL / PostgreSQL / Oracle / MS SQL? | ⭐ **3306 / 5432 / 1521 / 1433** |
| Port Redis / Memcached? | ⭐ **6379 / 11211** |
| Port Redshift? | **5439** |

### Checklist tự kiểm tra trước khi làm quiz

- [ ] Thuộc **bảng Read Replicas vs Multi-AZ** (ASYNC vs SYNC, scale vs DR)
- [ ] Nhớ **RDS không SSH được** — trừ **RDS Custom** (chỉ Oracle & MS SQL)
- [ ] Nhớ **3 điều kiện Storage Auto Scaling**: **< 10%**, **5 phút**, **6 giờ**
- [ ] Thuộc các con số Aurora: **5x/3x**, **10GB→256TB**, **6 bản sao/3 AZ**, **4/6 ghi – 3/6 đọc**, **15 replica**, **< 30 giây failover**, **+20% chi phí**
- [ ] Nhớ Global Aurora: **10 secondary regions**, **16 replica/region**, **< 1 giây lag**, **< 1 phút RTO**
- [ ] Phân biệt **Backtrack** (Aurora) vs **Point-in-Time Restore** (RDS) vs **Cloning**
- [ ] Nhớ **Aurora automated backup KHÔNG tắt được**, RDS thì tắt được (0)
- [ ] Nhớ **Percona XtraBackup** cho migrate sang Aurora
- [ ] Nhớ **mã hóa RDS phải bật lúc launch**, chưa mã hóa → **snapshot & restore**
- [ ] Nhớ **RDS Proxy: 66% failover, không public, giải quyết Lambda connection storm**
- [ ] Thuộc **bảng Redis vs Memcached**
- [ ] Phân biệt **Lazy Loading** (stale) vs **Write Through** (no stale)
- [ ] Thuộc **danh sách port** (đặc biệt 3306 / 5432 / 1521 / 1433 / 6379 / 11211)

---

## Thuật ngữ Anh — Việt

| Tiếng Anh | Tiếng Việt |
|-----------|-----------|
| Relational Database Service (RDS) | Dịch vụ cơ sở dữ liệu quan hệ |
| Managed service | Dịch vụ được quản lý |
| Provisioning | Cấp phát tài nguyên |
| OS patching | Vá lỗi hệ điều hành |
| Point in Time Restore | Khôi phục về một thời điểm |
| Maintenance window | Khung giờ bảo trì |
| Storage Auto Scaling | Tự động mở rộng dung lượng |
| Maximum Storage Threshold | Ngưỡng dung lượng tối đa |
| Read Replica | Bản sao chỉ đọc |
| ASYNC / SYNC replication | Sao chép bất đồng bộ / đồng bộ |
| Eventually consistent | Nhất quán cuối cùng |
| Promote | Thăng cấp (thành DB độc lập) |
| Multi-AZ | Đa vùng sẵn sàng |
| Standby | Máy dự phòng chờ |
| Failover | Chuyển đổi dự phòng |
| Disaster Recovery (DR) | Khôi phục sau thảm họa |
| RTO (Recovery Time Objective) | Mục tiêu thời gian khôi phục |
| RDS Custom | RDS có quyền tùy biến OS |
| Automation Mode | Chế độ tự động hóa |
| Proprietary technology | Công nghệ độc quyền |
| Self healing | Tự phục hồi |
| Striped storage | Lưu trữ phân dải |
| Shared storage volume | Ổ lưu trữ dùng chung |
| Writer / Reader Endpoint | Điểm ghi / điểm đọc |
| Custom Endpoint | Điểm truy cập tùy chỉnh |
| Connection Load Balancing | Cân bằng tải kết nối |
| Backtrack | Quay ngược thời gian dữ liệu |
| Serverless | Không máy chủ (tự động cấp phát) |
| Capacity planning | Hoạch định năng lực |
| Global Database | Cơ sở dữ liệu toàn cầu |
| Replication lag | Độ trễ sao chép |
| Copy-on-write | Sao chép khi ghi |
| Database Cloning | Nhân bản cơ sở dữ liệu |
| Staging database | Cơ sở dữ liệu môi trường thử |
| Automated backup | Sao lưu tự động |
| Transaction logs | Nhật ký giao dịch |
| Retention | Thời hạn lưu giữ |
| At-rest / In-flight encryption | Mã hóa khi lưu / khi truyền |
| Audit Logs | Nhật ký kiểm toán |
| RDS Proxy | Proxy cho cơ sở dữ liệu |
| Connection pooling | Gộp/chia sẻ kết nối |
| Secrets Manager | Dịch vụ quản lý bí mật |
| In-memory database | Cơ sở dữ liệu trong bộ nhớ |
| Cache hit / Cache miss | Trúng cache / Trượt cache |
| Cache invalidation | Làm mất hiệu lực cache |
| Stale data | Dữ liệu cũ/lỗi thời |
| Stateless | Không lưu trạng thái |
| Lazy Loading | Nạp lười (chỉ khi cần) |
| Write Through | Ghi xuyên (ghi cả DB lẫn cache) |
| Session Store | Kho lưu phiên đăng nhập |
| TTL (Time To Live) | Thời gian sống của dữ liệu |
| Sharding / Partitioning | Phân mảnh dữ liệu |
| Persistence (AOF) | Tính bền vững (ghi log lệnh) |
| Sorted Sets | Tập hợp có thứ tự |
| Multi-threaded | Đa luồng |
| Redis AUTH | Cơ chế xác thực của Redis |
| Leaderboard | Bảng xếp hạng |

---

*Ghi chú: các phần Hands On được tóm tắt lại các bước thao tác chính trên AWS Console. Giao diện Console có thể thay đổi theo thời gian — logic và khái niệm vẫn giữ nguyên. ⚠️ Aurora và ElastiCache KHÔNG nằm trong Free Tier — nhớ xóa cluster ngay sau khi thực hành xong.*
