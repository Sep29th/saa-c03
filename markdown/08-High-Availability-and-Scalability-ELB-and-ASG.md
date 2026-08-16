# Phần 8 — High Availability and Scalability: ELB & ASG

> Khóa học: *Ultimate AWS Certified Solutions Architect Associate 2026* (Stéphane Maarek) — SAA-C03
> Nguồn tham chiếu: `AWS Certified Solutions Architect Slides v48.pdf` (phần "High Availability & Scalability")

---

## Mục lục

| # | Bài giảng | Thời lượng | Loại |
|---|-----------|-----------|------|
| 70 | [High Availability and Scalability](#70-high-availability-and-scalability) | 5 phút | Video |
| 71 | [Elastic Load Balancing (ELB) Overview](#71-elastic-load-balancing-elb-overview) | 6 phút | Video |
| 72 | [Application Load Balancer (ALB)](#72-application-load-balancer-alb) | 6 phút | Video |
| 73 | [ALB - Hands On - Part 1](#73-application-load-balancer-alb---hands-on---part-1) | 9 phút | Video |
| 74 | [ALB - Hands On - Part 2](#74-application-load-balancer-alb---hands-on---part-2) | 6 phút | Video |
| 75 | [Network Load Balancer (NLB)](#75-network-load-balancer-nlb) | 3 phút | Video |
| 76 | [NLB - Hands On](#76-network-load-balancer-nlb---hands-on) | 6 phút | Video |
| 77 | [Gateway Load Balancer (GWLB)](#77-gateway-load-balancer-gwlb) | 4 phút | Video |
| 78 | [ELB - Sticky Sessions](#78-elastic-load-balancer---sticky-sessions) | 6 phút | Video |
| 79 | [ELB - Cross Zone Load Balancing](#79-elastic-load-balancer---cross-zone-load-balancing) | 6 phút | Video |
| 80 | [ELB - SSL Certificates](#80-elastic-load-balancer---ssl-certificates) | 6 phút | Video |
| 81 | [ELB - SSL Certificates - Hands On](#81-elastic-load-balancer---ssl-certificates---hands-on) | 2 phút | Video |
| 82 | [ELB - Connection Draining](#82-elastic-load-balancer---connection-draining) | 2 phút | Video |
| 83 | [Auto Scaling Groups (ASG) Overview](#83-auto-scaling-groups-asg-overview) | 5 phút | Video |
| 84 | [Auto Scaling Groups Hands On](#84-auto-scaling-groups-hands-on) | 9 phút | Video |
| 85 | [ASG - Scaling Policies](#85-auto-scaling-groups---scaling-policies) | 4 phút | Video |
| 86 | [ASG - Scaling Policies Hands On](#86-auto-scaling-groups---scaling-policies-hands-on) | 9 phút | Video |
| — | [Trắc nghiệm 5: High Availability & Scalability Quiz](#trắc-nghiệm-5-high-availability--scalability-quiz) | — | Quiz |

---

## 70. High Availability and Scalability

### Scalability là gì?

- **Scalability (khả năng mở rộng)** nghĩa là ứng dụng/hệ thống **có thể xử lý tải lớn hơn bằng cách thích ứng**.
- Có **hai loại scalability**:
  - **Vertical Scalability** (mở rộng theo chiều dọc)
  - **Horizontal Scalability** (= **elasticity**, mở rộng theo chiều ngang)
- **Scalability có liên quan nhưng KHÁC với High Availability**.

> Giảng viên dùng ví dụ **tổng đài (call center)** để phân biệt.

---

### 1️⃣ Vertical Scalability (Mở rộng theo chiều DỌC)

- Nghĩa là **tăng KÍCH THƯỚC của instance**.
- Ví dụ: ứng dụng đang chạy trên **`t2.micro`** → **scale dọc** = chạy nó trên **`t2.large`**.
- **Rất phổ biến cho hệ thống KHÔNG phân tán**, ví dụ như **database**.
- **RDS, ElastiCache** là các dịch vụ **có thể scale theo chiều dọc**.
- ⚠️ **Thường có GIỚI HẠN về mức có thể scale dọc** (giới hạn phần cứng).

**Ví von:** thay **nhân viên tổng đài junior** bằng **nhân viên senior** (một người nhưng giỏi hơn).

---

### 2️⃣ Horizontal Scalability (Mở rộng theo chiều NGANG)

- Nghĩa là **tăng SỐ LƯỢNG instance / hệ thống** cho ứng dụng.
- **Horizontal scaling ngụ ý hệ thống phân tán (distributed systems)**.
- **Rất phổ biến cho web application / ứng dụng hiện đại**.
- **Dễ scale ngang nhờ các dịch vụ cloud như Amazon EC2**.

**Ví von:** thuê **thêm nhiều nhân viên tổng đài** (nhiều người cùng làm).

---

### 3️⃣ High Availability (Tính sẵn sàng cao)

- **HA thường đi đôi với horizontal scaling**.
- **High availability nghĩa là chạy ứng dụng/hệ thống ở ÍT NHẤT 2 DATA CENTER (== Availability Zones)** ⭐
- **Mục tiêu của HA là SỐNG SÓT khi mất một data center**.
- HA có thể là:
  - **Passive (thụ động)** — ví dụ **RDS Multi-AZ**
  - **Active (chủ động)** — ví dụ **horizontal scaling**

**Ví von:** **tòa nhà thứ nhất ở New York** và **tòa nhà thứ hai ở San Francisco**.

---

### High Availability & Scalability cho EC2 ⭐

| Khái niệm | Cách làm | Dịch vụ AWS |
|-----------|----------|-------------|
| **Vertical Scaling** | **Tăng kích thước instance** (= **scale up / down**)<br>Từ: `t2.nano` – 0.5G RAM, 1 vCPU<br>Đến: `u-12tb1.metal` – **12.3 TB RAM, 448 vCPUs** | Thay đổi instance type |
| **Horizontal Scaling** | **Tăng SỐ LƯỢNG instance** (= **scale out / in**) | **Auto Scaling Group**<br>**Load Balancer** |
| **High Availability** | **Chạy instance cho cùng ứng dụng trên NHIỀU AZ** | **Auto Scaling Group multi-AZ**<br>**Load Balancer multi-AZ** |

### Thuật ngữ cần nhớ chính xác ⭐⭐

| Thuật ngữ | Ý nghĩa |
|-----------|---------|
| **Scale UP / DOWN** | **Vertical** — tăng/giảm **kích thước** instance |
| **Scale OUT / IN** | **Horizontal** — tăng/giảm **số lượng** instance |

> **Bẫy thi:** Đề dùng "scale out" → **thêm instance** (horizontal). "Scale up" → **instance to hơn** (vertical). Đừng nhầm.

---

## 71. Elastic Load Balancing (ELB) Overview

### Load balancing là gì?

**Load Balancers là các server chuyển tiếp traffic tới nhiều server (ví dụ EC2 instances) ở phía sau (downstream).**

```
              Users
                │
                ▼
      ┌───────────────────┐
      │ Elastic Load      │
      │ Balancer          │
      └─────┬──────┬──────┘
            │      │      \
        ┌───▼─┐ ┌──▼──┐ ┌──▼──┐
        │ EC2 │ │ EC2 │ │ EC2 │
        └─────┘ └─────┘ └─────┘
```

### Vì sao dùng Load Balancer? ⭐

- **Phân tán tải (spread load)** trên nhiều instance phía sau
- **Cung cấp MỘT điểm truy cập duy nhất (DNS)** cho ứng dụng ⭐
- **Xử lý mượt mà lỗi của các instance phía sau** (seamlessly handle failures)
- **Kiểm tra sức khỏe (health checks) định kỳ** các instance
- **Cung cấp SSL termination (HTTPS)** cho website ⭐
- **Áp dụng stickiness bằng cookies**
- **High availability xuyên các zone**
- **Tách traffic public khỏi traffic private**

### Vì sao dùng ELB (Elastic Load Balancer) thay vì tự dựng? ⭐

- **ELB là load balancer ĐƯỢC QUẢN LÝ (managed)**:
  - **AWS đảm bảo nó luôn hoạt động**
  - **AWS lo việc upgrade, maintenance, high availability**
  - **AWS chỉ cung cấp một vài "núm vặn" cấu hình**
- **Tự dựng load balancer thì rẻ hơn, NHƯNG tốn RẤT NHIỀU công sức của bạn**.
- **Tích hợp sẵn với nhiều dịch vụ AWS**:
  - **EC2, EC2 Auto Scaling Groups, Amazon ECS**
  - **AWS Certificate Manager (ACM), CloudWatch**
  - **Route 53, AWS WAF, AWS Global Accelerator**

---

### Health Checks ⭐⭐

- **Health Checks là CỰC KỲ QUAN TRỌNG với Load Balancer**.
- Chúng **cho load balancer biết instance mà nó chuyển traffic tới có sẵn sàng trả lời request hay không**.
- **Health check được thực hiện trên một PORT và một ROUTE** (**`/health` là phổ biến**).
- **Nếu response KHÔNG phải 200 (OK) → instance được coi là UNHEALTHY** ⭐

```
Elastic Load Balancer ──► Protocol: HTTP
                          Port: 4567
                          Endpoint: /health   ──► EC2 Instance
```

> Instance `unhealthy` sẽ **KHÔNG nhận traffic** cho tới khi health check pass trở lại.

---

### 4 loại Load Balancer được quản lý trên AWS ⭐⭐

| Load Balancer | Thế hệ | Năm | Giao thức hỗ trợ | Layer |
|---------------|--------|-----|------------------|-------|
| **Classic Load Balancer (CLB)** | **v1 – thế hệ CŨ** | **2009** | **HTTP, HTTPS, TCP, SSL (secure TCP)** | Layer 4 & 7 |
| **Application Load Balancer (ALB)** | **v2 – thế hệ MỚI** | **2016** | **HTTP, HTTPS, WebSocket** | **Layer 7** |
| **Network Load Balancer (NLB)** | **v2 – thế hệ MỚI** | **2017** | **TCP, TLS (secure TCP), UDP** | **Layer 4** |
| **Gateway Load Balancer (GWLB)** | — | **2020** | **IP Protocol** | **Layer 3 (Network layer)** |

- **Nhìn chung, ĐƯỢC KHUYẾN NGHỊ dùng các load balancer thế hệ MỚI** vì chúng có nhiều tính năng hơn.
- **Một số load balancer có thể được thiết lập là INTERNAL (private) hoặc EXTERNAL (public) ELB** ⭐

> **Ghi nhớ:** CLB đã **deprecated** — AWS khuyến khích chuyển sang ALB/NLB.

---

### Load Balancer Security Groups ⭐⭐

Mô hình bảo mật chuẩn (rất hay ra thi):

```
   Users
     │  HTTPS / HTTP
     │  From anywhere (0.0.0.0/0)
     ▼
┌──────────────────┐
│  LOAD BALANCER   │  ← Load Balancer Security Group:
└────────┬─────────┘     Allow HTTP/HTTPS from ANYWHERE
         │
         │  HTTP Restricted to Load Balancer
         ▼
     ┌───────┐
     │  EC2  │  ← Application Security Group:
     └───────┘     Allow traffic ONLY FROM Load Balancer
```

| Security Group | Rule |
|----------------|------|
| **Load Balancer SG** | **Inbound: HTTP (80) & HTTPS (443) từ `0.0.0.0/0`** (bất cứ đâu) |
| **Application (EC2) SG** | **Inbound: HTTP (80) CHỈ từ Security Group của Load Balancer** ⭐ |

> **Điểm mấu chốt:** Security Group của EC2 **tham chiếu tới Security Group của Load Balancer**, không dùng IP. Nhờ vậy EC2 **không thể bị truy cập trực tiếp** từ internet.

---

### Classic Load Balancer (v1) — thế hệ cũ

- **Hỗ trợ TCP (Layer 4), HTTP & HTTPS (Layer 7)**
- **Health checks dựa trên TCP hoặc HTTP**
- **Hostname cố định: `XXX.region.elb.amazonaws.com`**

```
Client ──listener──► CLB ──internal──► EC2
```

> CLB đã lỗi thời, chỉ cần biết khái niệm cơ bản và các hạn chế của nó (đặc biệt: **chỉ hỗ trợ MỘT SSL certificate**).

---

## 72. Application Load Balancer (ALB)

### Đặc điểm cốt lõi ⭐

- **ALB là Layer 7 (HTTP)** ⭐
- **Load balancing tới NHIỀU ứng dụng HTTP trên nhiều máy** (**target groups**)
- **Load balancing tới NHIỀU ứng dụng trên CÙNG MỘT máy** (ví dụ: **containers**)
- **Hỗ trợ HTTP/2 và WebSocket**
- **Hỗ trợ redirect** (ví dụ **từ HTTP sang HTTPS**) ⭐

### Routing tables tới các Target Group khác nhau ⭐⭐

ALB có thể định tuyến dựa trên:

| Loại routing | Ví dụ |
|--------------|-------|
| **Routing dựa trên PATH trong URL** | `example.com/users` & `example.com/posts` |
| **Routing dựa trên HOSTNAME trong URL** | `one.example.com` & `other.example.com` |
| **Routing dựa trên Query String, Headers** | `example.com/users?id=123&order=false` |

### Ưu điểm cho kiến trúc hiện đại

- **ALB rất phù hợp cho micro services & ứng dụng dựa trên container** (ví dụ: **Docker & Amazon ECS**) ⭐
- **Có tính năng port mapping để redirect tới port động trong ECS** ⭐
- **So sánh:** với CLB, ta sẽ cần **NHIỀU Classic Load Balancer cho mỗi ứng dụng** — rất tốn kém.

### Sơ đồ HTTP Based Traffic

```
  WWW ──Route /user───┐
                      ├──► External ALB (v2) ──Health Check──► Target Group for Users app
  WWW ──Route /search─┘                      ──Health Check──► Target Group for Search app
```

---

### ALB — Target Groups ⭐⭐

ALB có thể route tới các loại target sau:

| Target | Ghi chú |
|--------|---------|
| **EC2 instances** (có thể được quản lý bởi **Auto Scaling Group**) | **HTTP** |
| **ECS tasks** (được quản lý bởi chính ECS) | **HTTP** |
| **Lambda functions** ⭐ | **HTTP request được dịch thành JSON event** |
| **IP Addresses** | **PHẢI là private IP** ⭐ |

- **ALB có thể route tới NHIỀU target group**
- **Health checks được thực hiện ở CẤP ĐỘ TARGET GROUP** ⭐

> **Ghi nhớ thi:** ALB là **load balancer DUY NHẤT** có thể route tới **Lambda function**.

### Ví dụ: Query String / Parameters Routing

```
  WWW ──?Platform=Mobile───┐
       Requests            ├──► External ALB ──► Target Group 1: AWS – EC2 based
       ──?Platform=Desktop─┘                 ──► Target Group 2: On-premises – Private IP routing
```

> Đây là cách **kiến trúc hybrid**: một phần traffic đi tới EC2 trên AWS, một phần đi tới hệ thống on-premises qua private IP.

---

### ALB — Good to Know ⭐⭐

- **Hostname cố định** (`XXX.region.elb.amazonaws.com`)
- **Application server KHÔNG nhìn thấy IP của client một cách trực tiếp** ⭐
- **IP thật của client được chèn vào header `X-Forwarded-For`** ⭐⭐
- **Cũng lấy được Port qua `X-Forwarded-Port` và protocol qua `X-Forwarded-Proto`**

```
Client IP              Load Balancer IP
12.34.56.78  ────────► (Private IP)  ──connection termination──►  EC2 Instance
                                                                   thấy IP của LB,
                                                                   không thấy 12.34.56.78
                                                     → phải đọc header X-Forwarded-For
```

> **Câu hỏi kinh điển:** "Ứng dụng sau ALB cần biết IP thật của client — làm thế nào?" → **Đọc header `X-Forwarded-For`**.

---

## 73. Application Load Balancer (ALB) - Hands On - Part 1

### Bước 1 — Chuẩn bị EC2 instances

1. Launch **2 EC2 instance** ở **2 AZ khác nhau** (ví dụ `eu-west-3a` và `eu-west-3b`).
2. Dùng **User Data** để cài web server (script từ phần 5):
   ```bash
   #!/bin/bash
   yum update -y
   yum install -y httpd
   systemctl start httpd
   systemctl enable httpd
   echo "<h1>Hello World from $(hostname -f)</h1>" > /var/www/html/index.html
   ```
3. Security Group cho instance: mở **HTTP (80)** và **SSH (22)**.
4. Kiểm tra: truy cập public IP của từng instance → thấy hostname **khác nhau**.

### Bước 2 — Tạo Application Load Balancer

1. EC2 → menu trái → **Load Balancing** → **Load Balancers** → **Create load balancer**.
2. Chọn **Application Load Balancer** → **Create**.
3. **Basic configuration**:
   - **Load balancer name**: `DemoALB`
   - **Scheme**: **Internet-facing** (public) hoặc **Internal** (private)
   - **IP address type**: `IPv4`
4. **Network mapping**:
   - **VPC**: default VPC
   - **Mappings**: ⭐ **chọn ÍT NHẤT 2 Availability Zone** (bắt buộc cho ALB)
5. **Security groups**: tạo SG mới `DemoALB-SG` với inbound **HTTP (80) từ `0.0.0.0/0`**.

### Bước 3 — Tạo Target Group

1. Trong phần **Listeners and routing** → **Create target group** (mở tab mới):
   - **Target type**: **Instances** (hoặc IP addresses / Lambda function / Application Load Balancer)
   - **Target group name**: `DemoTargetGroup`
   - **Protocol / Port**: `HTTP` / `80`
   - **Health checks**:
     - **Health check path**: `/` (hoặc `/health`)
     - **Advanced**: Healthy threshold `5`, Unhealthy threshold `2`, Timeout `5s`, Interval `30s`, **Success codes `200`**
2. **Next** → chọn **2 instance** → **Include as pending below** → **Create target group**.

### Bước 4 — Hoàn tất tạo ALB

1. Quay lại tab tạo ALB → **Refresh** danh sách target group → chọn `DemoTargetGroup`.
2. **Create load balancer**.
3. Đợi **State: `Provisioning` → `Active`** (khoảng 2–3 phút).

### Bước 5 — Kiểm chứng

1. Copy **DNS name** của ALB (ví dụ `DemoALB-123456.eu-west-3.elb.amazonaws.com`).
2. Truy cập bằng trình duyệt → thấy `Hello World from ip-172-31-x-x`.
3. **Refresh nhiều lần** → ⭐ **hostname LUÂN PHIÊN giữa 2 instance** → load balancing hoạt động!

### Bước 6 — Siết chặt Security Group ⭐

Đây là bước quan trọng nhất về bảo mật:

1. Sửa **Security Group của các EC2 instance**:
   - **Xóa** rule `HTTP từ 0.0.0.0/0`
   - **Thêm** rule: **HTTP (80), Source = Security Group của ALB** (`DemoALB-SG`)
2. Kiểm chứng:
   - Truy cập **trực tiếp public IP của EC2** → ❌ **timeout** (bị chặn)
   - Truy cập qua **DNS name của ALB** → ✅ **vẫn hoạt động**

> **Đây chính là mô hình bảo mật chuẩn:** chỉ ALB được phép chạm vào EC2.

---

## 74. Application Load Balancer (ALB) - Hands On - Part 2

### 1. Kiểm tra Health Checks hoạt động

1. **Stop** một trong hai instance.
2. Vào **Target Groups** → tab **Targets** → quan sát:
   - Instance đó chuyển từ `healthy` → **`unhealthy`** → **`unused`**
3. Refresh DNS của ALB → **chỉ còn thấy hostname của instance còn sống** ⭐
4. **Start** lại instance → sau vài chu kỳ health check → quay lại `healthy`.

> **Đây chính là "seamlessly handle failures"** — người dùng không hề bị gián đoạn.

### 2. Tạo Listener Rules (routing nâng cao) ⭐

1. Load Balancers → chọn ALB → tab **Listeners and rules** → chọn listener `HTTP:80` → **Manage rules** → **Add rule**.
2. **Add condition** — chọn một trong:

| Condition | Ví dụ giá trị |
|-----------|--------------|
| **Host header** | `one.example.com` |
| **Path** | `/users*`, `/api/*` |
| **HTTP request method** | `GET`, `POST` |
| **Source IP** | `10.0.0.0/8` |
| **Query string** | `Platform` = `Mobile` |
| **HTTP header** | `User-Agent` chứa `Mobile` |

3. **Add action** — chọn một trong:

| Action | Mô tả |
|--------|-------|
| **Forward to target groups** | Chuyển tới target group cụ thể |
| **Redirect to URL** | Chuyển hướng (ví dụ HTTP → HTTPS, hoặc sang domain khác) |
| **Return fixed response** | Trả về nội dung cố định (ví dụ `404 Not Found`) |

4. Đặt **Priority** (số nhỏ = ưu tiên cao) → **Save changes**.

### 3. Thử nghiệm Fixed Response

1. Tạo rule: **Path = `/test`** → Action: **Return fixed response**
   - Status code: `200`
   - Response body: `Đây là fixed response từ ALB`
   - Content type: `text/plain`
2. Truy cập `http://<alb-dns>/test` → thấy nội dung cố định (**không đi tới EC2 nào**).

### 4. Thử nghiệm Query String Routing

1. Tạo rule: **Query string** `Platform = Mobile` → forward tới target group khác.
2. Truy cập `http://<alb-dns>/?Platform=Mobile`.

### 5. Redirect HTTP → HTTPS

1. Sửa **Default action** của listener `HTTP:80`:
   - **Redirect to URL**
   - Protocol: `HTTPS`, Port: `443`
   - Status code: `HTTP 301 (Permanently moved)`
2. Truy cập `http://<alb-dns>` → **tự động chuyển sang `https://`**.

> (Cần có HTTPS listener + certificate — xem bài 80–81.)

### 6. Dọn dẹp

- Xóa các listener rule đã thêm.
- Giữ lại ALB + target group cho bài sau, hoặc xóa nếu dừng thực hành.

---

## 75. Network Load Balancer (NLB)

### Đặc điểm cốt lõi ⭐⭐

**Network Load Balancer (Layer 4) cho phép:**

- **Chuyển tiếp traffic TCP & UDP tới instance của bạn** ⭐
- **Xử lý HÀNG TRIỆU request mỗi giây** ⭐
- **Độ trễ CỰC THẤP (ultra-low latency)** ⭐

**Điểm đặc biệt về IP:**

- **NLB có MỘT STATIC IP cho MỖI AZ**, và **hỗ trợ gán Elastic IP** ⭐⭐
  → **rất hữu ích khi cần whitelist các IP cụ thể**

**Use case:** **NLB được dùng cho hiệu năng cực cao, traffic TCP hoặc UDP**.

### Sơ đồ TCP (Layer 4) Based Traffic

```
  WWW ──TCP + Rules──┐
                     ├──► External NLB (v2) ──Health Check──► Target Group for Users app
  WWW ──TCP + Rules──┘                      ──Health Check──► Target Group for Search app
```

---

### NLB — Target Groups ⭐

| Target | Ghi chú |
|--------|---------|
| **EC2 instances** | — |
| **IP Addresses** | **PHẢI là private IP** ⭐ |
| **Application Load Balancer** ⭐ | **NLB có thể trỏ tới ALB!** |

- **Health Checks hỗ trợ các giao thức TCP, HTTP và HTTPS** ⭐

```
   NLB ──► Target Group (EC2 Instances):  i-1234567890abcdef0, i-1234567890abcdef0
   NLB ──► Target Group (IP Addresses):   192.168.1.118, 10.0.4.21
   NLB ──► Target Group (Application Load Balancer)
```

> **Mẹo kiến trúc:** Đặt **NLB trước ALB** → có được **static IP của NLB** kết hợp với **routing Layer 7 của ALB**. Đây là câu trả lời cho: *"cần static IP nhưng cũng cần routing theo path/hostname"*.

---

### ALB vs NLB — Bảng so sánh ⭐⭐

| | **ALB** | **NLB** |
|---|---|---|
| **Layer** | **Layer 7 (HTTP)** | **Layer 4 (TCP/UDP)** |
| **Giao thức** | HTTP, HTTPS, WebSocket | **TCP, TLS, UDP** |
| **Static IP** | ❌ Không (chỉ DNS name) | ✅ **Có — 1 static IP/AZ, hỗ trợ Elastic IP** |
| **Hiệu năng** | Tốt | ⭐ **Hàng triệu req/s, ultra-low latency** |
| **Routing theo path/hostname/header** | ✅ **Có** | ❌ Không |
| **Target: Lambda** | ✅ **Có** | ❌ Không |
| **Target: ALB** | ❌ Không | ✅ **Có** |
| **Health check protocols** | HTTP, HTTPS | **TCP, HTTP, HTTPS** |
| **Cross-Zone LB mặc định** | ✅ **Bật** (miễn phí) | ❌ **Tắt** (bật thì tính phí) |
| **Use case** | Web app, micro services, container | **Hiệu năng cực cao, TCP/UDP, gaming, IoT, cần static IP** |

### Cheat sheet chọn Load Balancer ⭐

| Từ khóa trong đề | Đáp án |
|------------------|--------|
| "HTTP/HTTPS", "route theo path/hostname", "micro services", "container/ECS" | **ALB** |
| "route tới **Lambda**" | **ALB** |
| "**static IP**", "**Elastic IP**", "whitelist IP" | **NLB** |
| "**TCP/UDP**", "millions of requests per second", "**ultra-low latency**" | **NLB** |
| "gaming server", "IoT", "**UDP**" | **NLB** |
| "3rd party **firewall** / **IDS/IPS**", "**deep packet inspection**" | **GWLB** |
| "cần cả static IP **và** routing Layer 7" | **NLB đứng trước ALB** |

---

## 76. Network Load Balancer (NLB) - Hands On

### Bước 1 — Tạo NLB

1. EC2 → **Load Balancers** → **Create load balancer** → **Network Load Balancer**.
2. **Basic configuration**:
   - **Name**: `DemoNLB`
   - **Scheme**: `Internet-facing`
3. **Network mapping**: chọn **≥ 2 AZ**.
   - ⭐ Với mỗi AZ, có thể chọn **IPv4 address**: `Assigned by AWS` hoặc **`Use an Elastic IP address`**
4. **Listeners and routing**: **TCP : 80** → chọn target group.

### Bước 2 — Tạo Target Group cho NLB

1. **Create target group**:
   - **Target type**: `Instances`
   - **Name**: `DemoNLBTargetGroup`
   - **Protocol / Port**: ⭐ **`TCP` / `80`** (không phải HTTP như ALB)
   - **Health check protocol**: có thể chọn **`TCP`**, `HTTP`, hoặc `HTTPS`
2. Đăng ký 2 instance → **Create target group**.

### Bước 3 — ⚠️ Sửa Security Group của EC2 (điểm hay sai nhất)

> **NLB KHÔNG có Security Group riêng theo cách của ALB.** Traffic tới EC2 **giữ nguyên IP nguồn của client** (khi target type = Instance), nên bạn **không thể tham chiếu SG của NLB**.

Phải mở Security Group của EC2 cho:

- **HTTP (80) từ `0.0.0.0/0`** (traffic của client), **hoặc**
- **HTTP (80) từ dải CIDR của VPC** (cho health check của NLB)

Nếu không, target sẽ mãi ở trạng thái **`unhealthy`**.

### Bước 4 — Kiểm chứng

1. Đợi NLB `Active` và target `healthy`.
2. Truy cập **DNS name của NLB** → thấy trang web, refresh → luân phiên giữa 2 instance.
3. ⭐ **`nslookup <nlb-dns>`** → trả về **các static IP** (một IP cho mỗi AZ) — khác hẳn ALB.

```bash
nslookup DemoNLB-abc123.elb.eu-west-3.amazonaws.com
```

### Bước 5 — Dọn dẹp

- Xóa NLB → xóa target group của NLB.

---

## 77. Gateway Load Balancer (GWLB)

### GWLB là gì? ⭐

- **Triển khai, mở rộng và quản lý một đội (fleet) các thiết bị mạng ảo của bên thứ 3 trong AWS**.
- **Ví dụ**: **Firewalls**, **Intrusion Detection and Prevention Systems (IDS/IPS)**, **Deep Packet Inspection Systems**, **payload manipulation**, …
- **Hoạt động ở Layer 3 (Network Layer) — IP Packets** ⭐

### GWLB kết hợp 2 chức năng ⭐⭐

| Chức năng | Mô tả |
|-----------|-------|
| **Transparent Network Gateway** | **MỘT điểm vào/ra duy nhất cho TOÀN BỘ traffic** |
| **Load Balancer** | **Phân phối traffic tới các virtual appliance của bạn** |

### Giao thức ⭐

- **Dùng giao thức GENEVE trên PORT 6081** ⭐⭐ (con số rất hay ra thi)

### Sơ đồ luồng traffic

```
   Users              Route              ┌──────────────────┐        Application
  (source) ──traffic──► Table ──────────►│ Gateway          │──────► (destination)
                                          │ Load Balancer    │
                                          └────────┬─────────┘
                                                   │
                                          ┌────────▼─────────┐
                                          │  Target Group    │
                                          │ 3rd Party        │
                                          │ Security Virtual │
                                          │ Appliances       │
                                          └──────────────────┘
```

**Luồng hoạt động:** Traffic từ user **KHÔNG đi thẳng** tới ứng dụng. **Route Table** chuyển hướng nó qua **GWLB** → GWLB gửi tới **các security appliance để kiểm tra** → nếu hợp lệ, traffic mới được chuyển tiếp tới ứng dụng.

> **Điểm quan trọng:** GWLB **hoàn toàn trong suốt (transparent)** — ứng dụng và người dùng không biết nó tồn tại.

### GWLB — Target Groups

| Target | Ghi chú |
|--------|---------|
| **EC2 instances** | — |
| **IP Addresses** | **PHẢI là private IP** ⭐ |

> Lưu ý: GWLB **KHÔNG** hỗ trợ target là Lambda hay ALB.

### Cheat sheet GWLB ⭐

| Từ khóa trong đề | Đáp án |
|------------------|--------|
| "**3rd party network virtual appliances**" | **GWLB** |
| "**firewall**, **IDS/IPS**, **deep packet inspection**" | **GWLB** |
| "**GENEVE protocol**", "**port 6081**" | **GWLB** |
| "kiểm tra toàn bộ traffic trước khi tới ứng dụng, trong suốt" | **GWLB** |
| "Layer 3 / IP packets" | **GWLB** |

---

## 78. Elastic Load Balancer - Sticky Sessions

### Sticky Sessions (Session Affinity) là gì? ⭐

- **Có thể triển khai stickiness để CÙNG MỘT client LUÔN được chuyển hướng tới CÙNG MỘT instance** phía sau load balancer.
- **Hoạt động với: Classic Load Balancer, Application Load Balancer, và Network Load Balancer** ⭐
- **Với cả CLB & ALB, "cookie" dùng cho stickiness có thời hạn (expiration date) mà BẠN kiểm soát**.
- **Use case: đảm bảo người dùng KHÔNG MẤT dữ liệu session của họ** ⭐
- ⚠️ **Bật stickiness CÓ THỂ gây MẤT CÂN BẰNG tải trên các EC2 instance phía sau** ⭐

```
   Client 1        Client 2        Client 3
      │               │               │
      └───────┐       │       ┌───────┘
              ▼       ▼       ▼
        ┌──────────┐   ┌──────────┐
        │   EC2    │   │   EC2    │
        └──────────┘   └──────────┘
        (Client 1 & 2)   (Client 3)   ← tải không đều
```

---

### Sticky Sessions — Cookie Names ⭐⭐

Đây là phần **rất hay ra thi**:

#### 1. Application-based Cookies

| Loại | Chi tiết |
|------|----------|
| **Custom cookie** | • **Được sinh ra bởi TARGET** (ứng dụng của bạn)<br>• **Có thể bao gồm bất kỳ custom attribute nào ứng dụng cần**<br>• **Tên cookie phải được chỉ định RIÊNG cho từng target group**<br>• ⚠️ **KHÔNG dùng `AWSALB`, `AWSALBAPP`, hoặc `AWSALBTG`** (được ELB dành riêng) |
| **Application cookie** | • **Được sinh ra bởi LOAD BALANCER**<br>• **Tên cookie là `AWSALBAPP`** |

#### 2. Duration-based Cookies

- **Cookie được sinh ra bởi LOAD BALANCER**
- **Tên cookie là `AWSALB` cho ALB**, **`AWSELB` cho CLB** ⭐

### Bảng tổng hợp tên cookie ⭐

| Loại cookie | Ai sinh ra | Tên cookie |
|-------------|-----------|------------|
| **Custom cookie** (application-based) | **Target (ứng dụng)** | Bạn tự đặt (**không được** dùng `AWSALB`, `AWSALBAPP`, `AWSALBTG`) |
| **Application cookie** (application-based) | **Load Balancer** | **`AWSALBAPP`** |
| **Duration-based cookie** — ALB | **Load Balancer** | **`AWSALB`** |
| **Duration-based cookie** — CLB | **Load Balancer** | **`AWSELB`** |

### Cấu hình trên Console

Target Groups → chọn target group → tab **Attributes** → **Edit**:
- **Stickiness**: `Enabled`
- **Stickiness type**: `Load balancer generated cookie` / `Application-based cookie`
- **Stickiness duration**: từ **1 giây đến 7 ngày**

---

## 79. Elastic Load Balancer - Cross Zone Load Balancing

### Khái niệm ⭐⭐

#### ✅ **VỚI Cross Zone Load Balancing:**

> **Mỗi load balancer instance phân phối ĐỀU tới TẤT CẢ các instance đã đăng ký trong TẤT CẢ các AZ.**

```
        AZ 1 (2 instances)              AZ 2 (8 instances)
   50% traffic ──┐                    50% traffic ──┐
                 ▼                                   ▼
           ┌────┬────┐            ┌──┬──┬──┬──┬──┬──┬──┬──┐
           │10% │10% │            │10│10│10│10│10│10│10│10│
           └────┴────┘            └──┴──┴──┴──┴──┴──┴──┴──┘

   → MỌI instance nhận 10% → CÂN BẰNG HOÀN HẢO ✅
```

#### ❌ **KHÔNG có Cross Zone Load Balancing:**

> **Request được phân phối trong các instance của NODE (thuộc AZ đó) của Elastic Load Balancer.**

```
        AZ 1 (2 instances)              AZ 2 (8 instances)
   50% traffic ──┐                    50% traffic ──┐
                 ▼                                   ▼
           ┌────┬────┐            ┌────┬────┬────┬────┬────┬────┬────┬────┐
           │25% │25% │            │6.25│6.25│6.25│6.25│6.25│6.25│6.25│6.25│
           └────┴────┘            └────┴────┴────┴────┴────┴────┴────┴────┘

   → Instance ở AZ 1 nhận 25%, ở AZ 2 chỉ nhận 6.25% → MẤT CÂN BẰNG ❌
```

> **Điểm mấu chốt:** Không có Cross-Zone, traffic chia đều **theo AZ**, không phải **theo instance**. AZ ít instance hơn → mỗi instance chịu tải nặng hơn.

---

### Mặc định và chi phí theo từng loại LB ⭐⭐⭐

Đây là **bảng phải thuộc lòng**:

| Load Balancer | Mặc định | Chi phí inter-AZ data |
|---------------|----------|----------------------|
| **Application Load Balancer** | ✅ **BẬT mặc định** (có thể **tắt ở cấp Target Group**) | ✅ **KHÔNG tính phí** |
| **Network Load Balancer** & **Gateway Load Balancer** | ❌ **TẮT mặc định** | 💰 **CÓ TÍNH PHÍ ($)** nếu bật |
| **Classic Load Balancer** | ❌ **TẮT mặc định** | ✅ **KHÔNG tính phí** nếu bật |

### Mẹo nhớ ⭐

```
ALB   → BẬT sẵn, MIỄN PHÍ          (dễ dùng nhất)
NLB/GWLB → TẮT sẵn, TÍNH TIỀN      (phải trả tiền để bật)
CLB   → TẮT sẵn, MIỄN PHÍ          (cũ nhưng miễn phí)
```

> **Bẫy thi:** Đề hỏi "ALB có tính phí cross-zone không?" → **KHÔNG**. "NLB thì sao?" → **CÓ**.

---

## 80. Elastic Load Balancer - SSL Certificates

### SSL/TLS — Kiến thức cơ bản ⭐

- **SSL Certificate cho phép traffic giữa client và load balancer được MÃ HÓA KHI TRUYỀN (in-flight / in-transit encryption)** ⭐
- **SSL** = **Secure Sockets Layer**, dùng để **mã hóa kết nối**
- **TLS** = **Transport Layer Security**, là **phiên bản MỚI HƠN** ⭐
- **Ngày nay chủ yếu dùng TLS certificate, nhưng người ta vẫn quen gọi là SSL** ⭐
- **Public SSL certificate được cấp bởi các Certificate Authority (CA)**:
  - **Comodo, Symantec, GoDaddy, GlobalSign, Digicert, Let's Encrypt**, v.v.
- **SSL certificate có ngày hết hạn (do bạn đặt) và PHẢI được gia hạn** ⭐

### Mô hình mã hóa

```
   Users ──HTTPS (encrypted) over WWW──► LOAD BALANCER ──HTTP over private VPC──► EC2 Instance
```

> **Điểm quan trọng:** Traffic từ user tới LB được **mã hóa**; từ LB tới EC2 có thể là **HTTP thường** vì đã nằm trong **private VPC**. Đây gọi là **SSL Termination** (kết thúc SSL tại load balancer).

---

### Load Balancer — SSL Certificates ⭐⭐

- **Load balancer dùng X.509 certificate (SSL/TLS server certificate)** ⭐
- **Bạn có thể quản lý certificate bằng ACM (AWS Certificate Manager)** ⭐
- **Hoặc bạn có thể tạo/upload certificate của riêng mình**
- **HTTPS listener**:
  - **PHẢI chỉ định một DEFAULT certificate** ⭐
  - **Có thể thêm danh sách tùy chọn các certificate để hỗ trợ NHIỀU DOMAIN**
  - **Client có thể dùng SNI (Server Name Indication) để chỉ định hostname họ muốn tới** ⭐
  - **Có khả năng chỉ định SECURITY POLICY để hỗ trợ các phiên bản SSL/TLS cũ** (cho legacy client) ⭐

---

### SSL — Server Name Indication (SNI) ⭐⭐⭐

Đây là khái niệm **rất hay ra thi**:

- **SNI giải quyết bài toán NẠP NHIỀU SSL CERTIFICATE lên MỘT web server** (để phục vụ nhiều website)
- Nó là **giao thức "mới hơn"**, và **YÊU CẦU CLIENT phải chỉ ra hostname của server đích trong SSL handshake ban đầu** ⭐
- **Server sẽ tìm certificate ĐÚNG, hoặc trả về certificate MẶC ĐỊNH**

### ⭐⭐ Ghi chú CỰC KỲ quan trọng về SNI:

> **SNI CHỈ hoạt động với ALB & NLB (thế hệ mới) và CloudFront.**
> **SNI KHÔNG hoạt động với CLB (thế hệ cũ).** ⭐⭐

### Sơ đồ SNI

```
                          "Tôi muốn www.mycorp.com"
   Client ──────────────────────────────────────────►  ALB
                                                        │
                                    ┌───────────────────┴───────────────────┐
                                    │  Chọn ĐÚNG SSL cert                    │
                                    ▼                                        ▼
                        SSL Cert: www.mycorp.com          SSL Cert: Domain1.example.com
                                    │                                        │
                                    ▼                                        ▼
                        Target group for                      Target group for
                        www.mycorp.com                        Domain1.example.com
```

---

### Elastic Load Balancers — SSL Certificates theo từng loại ⭐⭐

**Bảng phải thuộc lòng:**

| Load Balancer | Hỗ trợ SSL Certificate |
|---------------|------------------------|
| **Classic Load Balancer (v1)** | • **Chỉ hỗ trợ MỘT SSL certificate** ⭐<br>• **PHẢI dùng NHIỀU CLB** cho nhiều hostname với nhiều SSL certificate |
| **Application Load Balancer (v2)** | • **Hỗ trợ NHIỀU listener với NHIỀU SSL certificate**<br>• **Dùng Server Name Indication (SNI)** để làm được điều đó ⭐ |
| **Network Load Balancer (v2)** | • **Hỗ trợ NHIỀU listener với NHIỀU SSL certificate**<br>• **Dùng Server Name Indication (SNI)** để làm được điều đó ⭐ |

> **Câu hỏi kinh điển:** "Cần phục vụ nhiều domain với nhiều SSL certificate trên một load balancer — dùng gì?" → **ALB hoặc NLB với SNI**. **Không phải CLB.**

---

## 81. Elastic Load Balancer - SSL Certificates - Hands On

### 1. Xem cấu hình SSL trên Console

1. EC2 → **Load Balancers** → chọn ALB → tab **Listeners and rules**.
2. **Add listener**:
   - **Protocol**: **HTTPS**, **Port**: **443**
   - **Default action**: Forward to target group
3. **Secure listener settings**:
   - **Security policy**: ⭐ chọn policy (ví dụ `ELBSecurityPolicy-TLS13-1-2-2021-06`)
     → policy quyết định **phiên bản TLS và cipher suite** nào được chấp nhận; chọn policy cũ hơn nếu cần hỗ trợ **legacy client**
   - **Default SSL/TLS certificate**:
     - **From ACM** (khuyến nghị) — chọn certificate đã có trong AWS Certificate Manager
     - **From IAM** — certificate đã upload vào IAM
     - **Import certificate** — tự upload certificate + private key

### 2. Xem tùy chọn nhiều certificate (SNI)

- Sau khi tạo HTTPS listener → chọn listener → tab **Certificates** → **Add certificate**.
- ⭐ Thêm nhiều certificate cho nhiều domain → **ALB tự dùng SNI** để chọn đúng cert theo hostname client yêu cầu.

### 3. Redirect HTTP → HTTPS ⭐

1. Chọn listener `HTTP:80` → **Edit default action**.
2. Chọn **Redirect to URL**:
   - **Protocol**: `HTTPS`
   - **Port**: `443`
   - **Status code**: `HTTP_301` (Permanently moved)
3. **Save** → mọi truy cập `http://` tự động chuyển sang `https://`.

### 4. Lưu ý thực hành ⚠️

- Để thực sự dùng HTTPS, bạn cần **một domain name thật** (mua qua Route 53 hoặc nơi khác) — **không thể cấp SSL certificate cho DNS name mặc định của ALB** (`*.elb.amazonaws.com`).
- **ACM cấp certificate MIỄN PHÍ** cho domain bạn sở hữu, và **TỰ ĐỘNG GIA HẠN** ⭐
- Vì lý do trên, bài hands-on này chỉ **xem giao diện cấu hình**, chưa hoàn tất được HTTPS thật. Phần Route 53 sau này sẽ làm đầy đủ.

### Ghi nhớ về ACM ⭐

| Đặc điểm | Chi tiết |
|----------|----------|
| Chi phí | **Miễn phí** cho public certificate |
| Gia hạn | ⭐ **Tự động** (không lo hết hạn) |
| Xác thực | **DNS validation** (khuyến nghị) hoặc **Email validation** |
| Dùng được với | **ALB, NLB, CloudFront, API Gateway** (**KHÔNG** dùng trực tiếp trên EC2) |
| Phạm vi | Certificate **theo Region** (CloudFront yêu cầu cert ở **`us-east-1`**) |

---

## 82. Elastic Load Balancer - Connection Draining

### Tên gọi theo từng loại LB ⭐⭐

| Load Balancer | Tên tính năng |
|---------------|---------------|
| **Classic Load Balancer (CLB)** | **Connection Draining** |
| **Application Load Balancer (ALB)** & **Network Load Balancer (NLB)** | **Deregistration Delay** ⭐ |

> **Bẫy thi:** Hai tên khác nhau nhưng **cùng một tính năng**. Đề có thể dùng bất kỳ tên nào.

### Tính năng làm gì? ⭐

- Là **thời gian để hoàn thành các "in-flight requests" (request đang xử lý dở)** trong khi instance **đang được de-register hoặc bị unhealthy**.
- **NGỪNG gửi request MỚI** tới EC2 instance đang de-register ⭐

### Con số cần nhớ ⭐⭐

| Thông số | Giá trị |
|----------|---------|
| **Khoảng giá trị** | **Từ 1 đến 3600 giây** |
| **Mặc định** | **300 giây (5 phút)** ⭐ |
| **Tắt tính năng** | **Đặt giá trị = 0** |

- **Đặt giá trị THẤP nếu request của bạn NGẮN** ⭐

### Sơ đồ

```
                          ┌─────────────────────────────┐
   Users ──────► ELB ────►│ EC2 Instance  [DRAINING]    │
                    │      │ waiting for existing        │
                    │      │ connections to complete     │
                    │      └─────────────────────────────┘
                    │
                    │  new connections established
                    ├─────────► EC2 Instance
                    └─────────► EC2 Instance
```

### Ứng dụng thực tế

| Tình huống | Nên đặt giá trị |
|-----------|----------------|
| API trả về nhanh (< 1 giây) | **Thấp** (ví dụ 30s) → deploy nhanh hơn |
| Upload file lớn / xử lý lâu | **Cao** (ví dụ 900–3600s) → không cắt ngang người dùng |
| Không quan tâm request dở dang | **0** (tắt) |

> **Vì sao quan trọng?** Khi ASG scale-in hoặc bạn deploy phiên bản mới, instance bị gỡ khỏi target group. Không có draining → **request đang xử lý bị cắt đứt** → người dùng thấy lỗi.

---

## 83. Auto Scaling Groups (ASG) Overview

### Vì sao cần ASG?

- **Trong thực tế, tải trên website và ứng dụng có thể THAY ĐỔI**.
- **Trên cloud, bạn có thể tạo và loại bỏ server RẤT NHANH**.

### Mục tiêu của Auto Scaling Group ⭐⭐

- **Scale OUT (THÊM EC2 instance)** để đáp ứng **tải TĂNG**
- **Scale IN (BỚT EC2 instance)** để đáp ứng **tải GIẢM**
- **Đảm bảo có một số lượng EC2 instance TỐI THIỂU và TỐI ĐA đang chạy**
- **Tự động ĐĂNG KÝ instance mới vào Load Balancer** ⭐
- **Tạo lại EC2 instance khi một instance trước đó bị terminate** (ví dụ: nếu unhealthy) ⭐

### 💰 **ASG là MIỄN PHÍ (bạn chỉ trả tiền cho các EC2 instance bên dưới)** ⭐

---

### Auto Scaling Group trong AWS

```
┌───────────────── Auto Scaling Group ─────────────────┐
│                                                       │
│  ┌───┐ ┌───┐  ← Minimum Capacity (tối thiểu)         │
│  │EC2│ │EC2│                                          │
│  └───┘ └───┘                                          │
│  ┌───┐ ┌───┐  ← Desired Capacity (mong muốn)         │
│  │EC2│ │EC2│                                          │
│  └───┘ └───┘                                          │
│  ┌───┐ ┌───┐  ← Maximum Capacity (tối đa)            │
│  │EC2│ │EC2│        ▲                                 │
│  └───┘ └───┘   Scale Out as Needed                    │
└───────────────────────────────────────────────────────┘
```

### Ba thông số dung lượng ⭐⭐

| Thông số | Ý nghĩa |
|----------|---------|
| **Minimum Capacity** | Số instance **TỐI THIỂU** luôn phải chạy |
| **Desired Capacity** | Số instance **MONG MUỐN** hiện tại (ASG luôn cố duy trì con số này) |
| **Maximum Capacity** | Số instance **TỐI ĐA** được phép |

> **Quy tắc:** `Min ≤ Desired ≤ Max`. ASG **không bao giờ** vượt Max hay xuống dưới Min.

---

### ASG kết hợp với Load Balancer ⭐

```
                        Users
                          │
                          ▼
            ┌───────────────────────────┐
            │  Elastic Load Balancer    │  ⭐ ELB có thể kiểm tra
            └────┬──────┬──────┬────────┘     sức khỏe EC2 instances!
                 │      │      │
    ┌────────────▼──────▼──────▼──────────────┐
    │        Auto Scaling Group                │
    │  ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐   │
    │  │EC2│ │EC2│ │EC2│ │EC2│ │EC2│ │EC2│   │
    │  └───┘ └───┘ └───┘ └───┘ └───┘ └───┘   │
    └──────────────────────────────────────────┘
```

> **ELB có thể kiểm tra sức khỏe (health) của các EC2 instance!** ⭐
> Khi ASG dùng **ELB health check** thay vì EC2 health check, instance bị lỗi ở tầng ứng dụng (dù VM vẫn chạy) cũng sẽ **bị thay thế**.

---

### Auto Scaling Group Attributes ⭐⭐

ASG được cấu hình thông qua:

#### 1. Launch Template (các "Launch Configuration" cũ đã **DEPRECATED**) ⭐

Launch Template chứa:

| Thành phần | Ghi chú |
|------------|---------|
| **AMI + Instance Type** | Ảnh máy và loại instance |
| **EC2 User Data** | Script khởi tạo |
| **EBS Volumes** | Cấu hình ổ đĩa |
| **Security Groups** | Tường lửa |
| **SSH Key Pair** | Khóa truy cập |
| **IAM Roles for your EC2 Instances** | Quyền cho instance ⭐ |
| **Network + Subnets Information** | VPC và subnet (quyết định AZ) |
| **Load Balancer Information** | Gắn với LB nào |

#### 2. Min Size / Max Size / Initial Capacity

#### 3. Scaling Policies

```
        ┌────────── ASG Launch Template ──────────┐
        │  AMI    Instance Type    SSH Key Pair    │
        │  Security Groups   EBS Volumes  IAM Role │
        │  VPC + Subnets     Load Balancer   …     │
        └──────────────────────────────────────────┘
```

> **Ghi nhớ thi:** Nếu đề nhắc **"Launch Configuration"** → đó là **công nghệ cũ đã deprecated**, câu trả lời đúng thường là **Launch Template**.

---

### Auto Scaling — CloudWatch Alarms & Scaling ⭐

- **Có thể scale một ASG dựa trên CloudWatch alarms** ⭐
- **Một alarm giám sát một METRIC** (ví dụ **Average CPU**, hoặc một **custom metric**)
- **Các metric như Average CPU được tính TRÊN TOÀN BỘ instance của ASG** ⭐
- **Dựa trên alarm:**
  - **Tạo scale-out policies** (tăng số lượng instance)
  - **Tạo scale-in policies** (giảm số lượng instance)

```
┌────── Auto Scaling Group ──────┐
│ ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐ │      ┌──────────────┐
│ │EC2│ │EC2│ │EC2│ │EC2│ │EC2│ │◄─────│ CloudWatch   │
│ └───┘ └───┘ └───┘ └───┘ └───┘ │      │ Alarm        │
└────────────────────────────────┘      └──────────────┘
              trigger Scaling
```

---

## 84. Auto Scaling Groups Hands On

### Bước 1 — Tạo Launch Template

1. EC2 → menu trái → **Instances** → **Launch Templates** → **Create launch template**.
2. Điền:
   - **Launch template name**: `MyLaunchTemplate`
   - **Template version description**: `v1`
   - ⚠️ Tick **"Provide guidance to help me set up a template that I can use with EC2 Auto Scaling"**
3. Cấu hình:
   - **AMI**: Amazon Linux 2023
   - **Instance type**: `t2.micro`
   - **Key pair**: chọn key có sẵn
   - **Network settings**: ⭐ **KHÔNG chọn subnet** (để ASG quyết định) — chỉ chọn **Security Group**
   - **Advanced details** → **User data**:
     ```bash
     #!/bin/bash
     yum update -y
     yum install -y httpd
     systemctl start httpd
     systemctl enable httpd
     echo "<h1>Hello World from $(hostname -f)</h1>" > /var/www/html/index.html
     ```
4. **Create launch template**.

### Bước 2 — Tạo Auto Scaling Group

1. EC2 → menu trái → **Auto Scaling** → **Auto Scaling Groups** → **Create Auto Scaling group**.
2. **Step 1 — Choose launch template**:
   - **Name**: `MyFirstASG`
   - **Launch template**: chọn `MyLaunchTemplate`
3. **Step 2 — Choose instance launch options**:
   - **VPC**: default
   - **Availability Zones and subnets**: ⭐ **chọn ÍT NHẤT 2 AZ** (để có High Availability)
4. **Step 3 — Configure advanced options**:
   - **Load balancing**: **Attach to an existing load balancer** → chọn target group của ALB đã tạo ở bài 73
   - **Health checks**: ⭐ tick **Turn on Elastic Load Balancing health checks**
   - **Health check grace period**: `300` giây (thời gian chờ trước khi bắt đầu health check)
5. **Step 4 — Configure group size and scaling**:
   - **Desired capacity**: `2`
   - **Min desired capacity**: `1`
   - **Max desired capacity**: `4`
   - **Scaling policies**: `None` (sẽ thêm ở bài 86)
6. **Step 5 — Add notifications** (tùy chọn, dùng SNS)
7. **Step 6 — Add tags**: ví dụ `Name = ASG-Instance`
8. **Review** → **Create Auto Scaling group**.

### Bước 3 — Quan sát ASG hoạt động ⭐

1. Vào **Instances** → thấy ASG **tự động launch 2 instance**.
2. Tab **Activity** của ASG → xem lịch sử:
   ```
   Launching a new EC2 instance: i-xxx
   Successful — Launching a new EC2 instance
   ```
3. Tab **Instance management** → thấy trạng thái **`InService`** / `Pending`.
4. Truy cập **DNS name của ALB** → thấy các instance mới của ASG trả lời.

### Bước 4 — Thử nghiệm tự phục hồi (self-healing) ⭐⭐

1. **Terminate thủ công một instance** của ASG.
2. Quan sát tab **Activity**:
   ```
   Terminating EC2 instance: i-xxx
   Launching a new EC2 instance: i-yyy   ← ASG TỰ TẠO LẠI!
   ```
3. → Chứng minh: **ASG luôn duy trì Desired Capacity**.

### Bước 5 — Thay đổi Desired Capacity thủ công

1. ASG → **Edit** → đổi **Desired capacity** từ `2` → `3`.
2. Quan sát → ASG **launch thêm 1 instance**.
3. Đổi lại về `2` → ASG **terminate bớt 1 instance**.

### Bước 6 — Xem Launch Template được dùng

- ASG → tab **Details** → **Launch template** → có thể **update sang version mới** của template mà không cần tạo lại ASG.

### Dọn dẹp

- Giữ ASG lại cho bài 86, hoặc **Delete** ASG (thao tác này **tự terminate hết instance**).

---

## 85. Auto Scaling Groups - Scaling Policies

### Tổng quan các loại Scaling Policy ⭐⭐

```
Scaling Policies
├── Dynamic Scaling
│   ├── Target Tracking Scaling
│   └── Simple / Step Scaling
├── Scheduled Scaling
└── Predictive Scaling
```

---

### 1️⃣ Dynamic Scaling — Target Tracking Scaling ⭐

- **ĐƠN GIẢN NHẤT để thiết lập (Simple to set-up)** ⭐
- **Ví dụ: Tôi muốn CPU trung bình của ASG duy trì ở khoảng 40%**

> ASG tự động tính toán cần thêm/bớt bao nhiêu instance để giữ metric ở mức mục tiêu. **Đây là lựa chọn được khuyến nghị cho hầu hết trường hợp.**

---

### 2️⃣ Dynamic Scaling — Simple / Step Scaling ⭐

- **Khi một CloudWatch alarm được kích hoạt** (ví dụ **CPU > 70%**) → **THÊM 2 units**
- **Khi một CloudWatch alarm được kích hoạt** (ví dụ **CPU < 30%**) → **BỚT 1 unit**

> Bạn kiểm soát chi tiết hơn, nhưng phải tự cấu hình alarm và ngưỡng.

---

### 3️⃣ Scheduled Scaling ⭐

- **Dự đoán trước việc scaling dựa trên các mẫu sử dụng ĐÃ BIẾT (known usage patterns)**
- **Ví dụ: tăng min capacity lên 10 vào 5 giờ chiều các ngày Thứ Sáu** ⭐

> Dùng khi bạn **biết trước** lịch tải (khuyến mãi, giờ cao điểm cố định, sự kiện).

---

### 4️⃣ Predictive Scaling ⭐

- **Liên tục DỰ BÁO tải và lên lịch scaling TRƯỚC** (continuously forecast load and schedule scaling ahead)

> Dùng **Machine Learning** phân tích lịch sử metric để dự đoán, khác với Scheduled Scaling (bạn phải tự đặt lịch).

---

### Bảng so sánh 4 loại Scaling Policy ⭐⭐

| Policy | Cơ chế | Khi nào dùng |
|--------|--------|-------------|
| **Target Tracking** | Giữ metric ở **giá trị mục tiêu** (ví dụ CPU = 40%) | **Đơn giản nhất**, phù hợp đa số trường hợp |
| **Simple / Step Scaling** | Dựa trên **CloudWatch alarm**, cộng/trừ số lượng cụ thể | Cần kiểm soát chi tiết theo từng ngưỡng |
| **Scheduled Scaling** | Theo **lịch cố định** bạn đặt | **Biết trước** mẫu tải (thứ Sáu 5pm) |
| **Predictive Scaling** | **ML dự báo** tải và scale trước | Tải có **chu kỳ** nhưng bạn không muốn tự đặt lịch |

### Mẹo nhớ ⭐

| Từ khóa trong đề | Đáp án |
|------------------|--------|
| "giữ CPU trung bình ở mức X%", "đơn giản nhất" | **Target Tracking** |
| "khi alarm kích hoạt thì thêm N instance" | **Simple / Step Scaling** |
| "biết trước lịch", "mỗi thứ Sáu lúc 5pm" | **Scheduled Scaling** |
| "**dự báo (forecast)** tải, scale **trước** khi cần" | **Predictive Scaling** |

---

### Good metrics to scale on ⭐⭐

Các metric tốt để scale (rất hay ra thi):

| Metric | Ý nghĩa |
|--------|---------|
| **`CPUUtilization`** | **Mức sử dụng CPU trung bình trên các instance** |
| **`RequestCountPerTarget`** ⭐ | **Đảm bảo SỐ REQUEST TRÊN MỖI EC2 INSTANCE là ổn định** |
| **Average Network In / Out** | Nếu ứng dụng của bạn **bị giới hạn bởi mạng (network bound)** |
| **Any custom metric** | Bất kỳ metric tùy chỉnh nào bạn **push qua CloudWatch** |

```
   Users
     │
     ▼
  Application Load Balancer
     │  RequestCountPerTarget
     │  Target Value: 3
     ▼
  Auto Scaling group
```

> **Ghi nhớ:** `RequestCountPerTarget` là metric **đặc trưng của ALB** — đề hỏi "scale theo số request mỗi instance" → chính là nó.

---

### Auto Scaling Groups — Scaling Cooldowns ⭐⭐

- **Sau khi một hoạt động scaling xảy ra, bạn ở trong COOLDOWN PERIOD (mặc định 300 giây)** ⭐
- **Trong cooldown period, ASG sẽ KHÔNG launch hoặc terminate thêm instance nào** (để **các metric có thời gian ổn định lại**) ⭐

```
   Scaling Action Occurs
            │
            ▼
   ┌──────────────────────┐
   │ Default Cooldown     │ ── Yes ──► Ignore Action
   │ in effect?           │
   └──────────┬───────────┘
              │ No
              ▼
   Launch or Terminate Instance
```

### ⭐ Lời khuyên từ slide

> **Dùng một AMI "ready-to-use" (đã cài sẵn phần mềm) để GIẢM thời gian cấu hình**, nhờ đó **phục vụ request nhanh hơn** và **giảm được cooldown period**.

Điều này liên kết trực tiếp với kiến thức **AMI** ở phần 7: thay vì để User Data cài đặt phần mềm mỗi lần boot (mất vài phút), hãy **build sẵn một golden AMI**.

---

## 86. Auto Scaling Groups - Scaling Policies Hands On

### Bước 1 — Tạo Target Tracking Scaling Policy ⭐

1. ASG → chọn `MyFirstASG` → tab **Automatic scaling**.
2. **Dynamic scaling policies** → **Create dynamic scaling policy**.
3. Cấu hình:
   - **Policy type**: **Target tracking scaling**
   - **Scaling policy name**: `Target Tracking Policy`
   - **Metric type**: **Average CPU utilization**
   - **Target value**: `40`
   - **Instance warmup**: `300` giây
   - Tùy chọn: tick **Disable scale in to create only a scale-out policy**
4. **Create**.

> ⭐ AWS **tự động tạo 2 CloudWatch alarm** (một cho scale-out, một cho scale-in) — xem ở CloudWatch → Alarms.

### Bước 2 — Tạo Simple / Step Scaling Policy

1. Trước tiên tạo CloudWatch alarm:
   - CloudWatch → **Alarms** → **Create alarm**
   - **Select metric** → **EC2** → **By Auto Scaling Group** → chọn ASG → **CPUUtilization**
   - **Conditions**: `Greater than 70`
   - **Notification**: bỏ qua hoặc chọn SNS topic
2. Quay lại ASG → **Create dynamic scaling policy**:
   - **Policy type**: **Step scaling** (hoặc **Simple scaling**)
   - **CloudWatch alarm**: chọn alarm vừa tạo
   - **Take the action**: `Add` `2` `capacity units` when `70 <= CPUUtilization < +infinity`
3. **Create**.

### Bước 3 — Tạo Scheduled Action ⭐

1. ASG → tab **Automatic scaling** → **Scheduled actions** → **Create scheduled action**.
2. Cấu hình:
   - **Name**: `scale-up-friday-evening`
   - **Desired capacity**: `10`, **Min**: `10`, **Max**: `15`
   - **Recurrence**: **Cron** → `0 17 * * 5` (17:00 mỗi Thứ Sáu)
   - **Time zone**: chọn múi giờ phù hợp
   - **Start time / End time**
3. **Create**.

### Bước 4 — Predictive Scaling

1. ASG → tab **Automatic scaling** → **Predictive scaling policies** → **Create predictive scaling policy**.
2. Cấu hình:
   - **Metric**: `CPU utilization` / `Network In` / `ALB request count per target` / custom
   - **Target utilization**: ví dụ `40`
   - **Mode**: **Forecast and scale** (thực thi) hoặc **Forecast only** (chỉ dự báo, để đánh giá trước)
3. ⚠️ Predictive Scaling cần **ít nhất 24 giờ dữ liệu lịch sử** để bắt đầu dự báo (tốt nhất là 14 ngày).

### Bước 5 — Kiểm tra Cooldown

1. ASG → **Edit** → mục **Default cooldown**: mặc định **300 giây**.
2. Xem tab **Activity** khi có scaling: các hành động cách nhau ít nhất bằng cooldown.

### Bước 6 — Tạo tải để kích hoạt scale-out (tùy chọn)

SSH vào một instance và tạo tải CPU:

```bash
sudo yum install -y stress
stress --cpu 4 --timeout 600
```

Quan sát:
1. CloudWatch metric `CPUUtilization` tăng.
2. Alarm chuyển sang trạng thái `In alarm`.
3. ASG **Activity** ghi nhận `Launching a new EC2 instance`.

### Bước 7 — Dọn dẹp toàn bộ ⚠️

Thứ tự dọn dẹp đúng:

1. **Xóa Auto Scaling Group** (thao tác này **tự động terminate hết instance**)
   ```
   Auto Scaling Groups → chọn ASG → Delete → gõ "delete" xác nhận
   ```
2. **Xóa Load Balancer** (ALB và NLB)
3. **Xóa Target Groups**
4. **Xóa Launch Template**
5. **Xóa CloudWatch Alarms** đã tạo
6. **Terminate** các EC2 instance còn sót (tạo thủ công ở bài 73)
7. **Xóa Security Groups** đã tạo (tùy chọn)

> ⚠️ **Nếu xóa instance trước khi xóa ASG → ASG sẽ tự tạo lại instance mới!** Luôn **xóa ASG trước**.

---

## Trắc nghiệm 5: High Availability & Scalability Quiz

### Các điểm dễ bị bẫy

| Câu hỏi thường gặp | Đáp án đúng |
|--------------------|-------------|
| Vertical scaling = gì? | **Tăng KÍCH THƯỚC instance** (scale up/down) |
| Horizontal scaling = gì? | **Tăng SỐ LƯỢNG instance** (scale out/in) |
| Dịch vụ nào scale theo chiều dọc? | **RDS, ElastiCache** |
| High Availability nghĩa là gì? | Chạy ứng dụng ở **ít nhất 2 AZ** |
| ELB làm health check dựa vào gì? | **Port + route** (thường `/health`), response phải là **200 OK** |
| AWS có mấy loại managed Load Balancer? | **4**: CLB, ALB, NLB, GWLB |
| ALB hoạt động ở Layer nào? | **Layer 7 (HTTP)** |
| NLB hoạt động ở Layer nào? | **Layer 4 (TCP/UDP)** |
| GWLB hoạt động ở Layer nào? | **Layer 3 (IP packets)** |
| ALB route được theo gì? | **Path, hostname, query string, headers, HTTP method, source IP** |
| Load balancer nào route được tới **Lambda**? | **ALB** |
| Load balancer nào có **static IP / Elastic IP**? | **NLB** |
| NLB route được tới ALB không? | **CÓ** |
| Cần static IP + routing Layer 7 → làm sao? | **NLB đứng trước ALB** |
| Ứng dụng sau ALB muốn biết IP thật của client? | Đọc header **`X-Forwarded-For`** |
| Header lấy port và protocol gốc? | **`X-Forwarded-Port`**, **`X-Forwarded-Proto`** |
| Health check của ALB ở cấp nào? | **Target Group** |
| ALB target là IP thì phải là loại IP nào? | **Private IP** |
| GWLB dùng giao thức gì, port nào? | **GENEVE, port 6081** |
| GWLB dùng cho gì? | **3rd party virtual appliances**: firewall, IDS/IPS, deep packet inspection |
| GWLB kết hợp 2 chức năng nào? | **Transparent Network Gateway + Load Balancer** |
| Sticky Sessions hoạt động với LB nào? | **CLB, ALB, NLB** |
| Nhược điểm của Sticky Sessions? | **Có thể gây mất cân bằng tải** |
| Tên cookie duration-based của ALB? | **`AWSALB`** |
| Tên cookie duration-based của CLB? | **`AWSELB`** |
| Tên application cookie do LB sinh ra? | **`AWSALBAPP`** |
| Cookie nào bị cấm dùng làm custom cookie? | **`AWSALB`, `AWSALBAPP`, `AWSALBTG`** |
| Cross-Zone LB: ALB mặc định? | **BẬT**, **không tính phí** inter-AZ |
| Cross-Zone LB: NLB & GWLB mặc định? | **TẮT**, **CÓ tính phí** nếu bật |
| Cross-Zone LB: CLB mặc định? | **TẮT**, **không tính phí** nếu bật |
| Không có Cross-Zone thì traffic chia thế nào? | **Chia đều theo AZ**, không theo instance → mất cân bằng |
| SSL vs TLS? | **TLS là phiên bản mới hơn của SSL** |
| Load balancer dùng loại certificate nào? | **X.509** |
| Quản lý certificate bằng dịch vụ nào? | **ACM (AWS Certificate Manager)** |
| SNI dùng để làm gì? | **Nạp NHIỀU SSL certificate lên MỘT server** |
| SNI hoạt động với LB nào? | **ALB & NLB** (và **CloudFront**) — **KHÔNG** với CLB |
| CLB hỗ trợ mấy SSL certificate? | **CHỈ MỘT** — phải dùng nhiều CLB |
| Hỗ trợ client TLS cũ → dùng gì? | **Security Policy** |
| Connection Draining là tên gọi của LB nào? | **CLB** |
| Deregistration Delay là tên gọi của LB nào? | **ALB & NLB** |
| Connection Draining khoảng giá trị? | **1 đến 3600 giây** |
| Connection Draining mặc định? | **300 giây** |
| Tắt Connection Draining thế nào? | **Đặt giá trị = 0** |
| ASG có tính phí không? | **KHÔNG** — chỉ trả tiền cho EC2 bên dưới |
| ASG dùng gì để định nghĩa instance? | **Launch Template** (Launch Configuration đã **deprecated**) |
| Launch Template chứa những gì? | AMI, instance type, User Data, EBS, SG, key pair, **IAM Role**, network/subnet, LB info |
| ASG có 3 thông số dung lượng nào? | **Min / Desired / Max** |
| Instance bị terminate thì ASG làm gì? | **Tự tạo lại instance mới** |
| Scaling policy nào đơn giản nhất? | **Target Tracking** |
| Scaling policy nào dùng CloudWatch alarm + cộng/trừ cụ thể? | **Simple / Step Scaling** |
| Scaling policy nào cho lịch biết trước? | **Scheduled Scaling** |
| Scaling policy nào dùng dự báo (forecast)? | **Predictive Scaling** |
| Metric nào tốt để scale theo số request? | **`RequestCountPerTarget`** |
| Cooldown period mặc định? | **300 giây** |
| Trong cooldown, ASG làm gì? | **KHÔNG launch/terminate instance** để metric ổn định |
| Làm sao giảm cooldown period? | **Dùng AMI ready-to-use** (giảm thời gian cấu hình) |
| Xóa ASG hay xóa instance trước? | **Xóa ASG TRƯỚC** (nếu không ASG sẽ tạo lại instance) |

### Checklist tự kiểm tra trước khi làm quiz

- [ ] Phân biệt **scale up/down (vertical)** vs **scale out/in (horizontal)**
- [ ] Nhớ **4 loại LB** + Layer + giao thức + use case
- [ ] Nhớ **ALB → Lambda**, **NLB → static IP + ALB**, **GWLB → GENEVE/6081**
- [ ] Nhớ **`X-Forwarded-For`** để lấy IP thật của client
- [ ] Thuộc **bảng Cross-Zone**: ALB bật/miễn phí, NLB-GWLB tắt/tính phí, CLB tắt/miễn phí
- [ ] Nhớ **tên cookie**: `AWSALB` (ALB), `AWSELB` (CLB), `AWSALBAPP` (application cookie)
- [ ] Nhớ **SNI chỉ dùng được với ALB/NLB/CloudFront**, CLB chỉ 1 cert
- [ ] Nhớ **Connection Draining (CLB) = Deregistration Delay (ALB/NLB)**, **1–3600s, mặc định 300s**
- [ ] Nhớ **Launch Template thay thế Launch Configuration**
- [ ] Thuộc **4 loại Scaling Policy** và từ khóa nhận diện
- [ ] Nhớ **cooldown mặc định 300 giây**

---

## Thuật ngữ Anh — Việt

| Tiếng Anh | Tiếng Việt |
|-----------|-----------|
| Scalability | Khả năng mở rộng |
| Elasticity | Tính co giãn |
| Vertical Scaling (scale up/down) | Mở rộng theo chiều dọc (tăng/giảm kích thước) |
| Horizontal Scaling (scale out/in) | Mở rộng theo chiều ngang (tăng/giảm số lượng) |
| High Availability (HA) | Tính sẵn sàng cao |
| Distributed system | Hệ thống phân tán |
| Load Balancer | Bộ cân bằng tải |
| Downstream | Phía sau (các server đích) |
| Health check | Kiểm tra sức khỏe |
| Healthy / Unhealthy | Khỏe mạnh / Không khỏe |
| Listener | Bộ lắng nghe (giao thức + port) |
| Target Group | Nhóm đích |
| Internal (private) / External (public) ELB | Cân bằng tải nội bộ / công khai |
| SSL Termination | Kết thúc SSL tại load balancer |
| Path-based routing | Định tuyến theo đường dẫn |
| Host-based routing | Định tuyến theo tên miền |
| Port mapping | Ánh xạ cổng |
| Connection termination | Kết thúc kết nối (tại LB) |
| Static IP | Địa chỉ IP tĩnh |
| Whitelisting | Danh sách cho phép |
| Virtual appliance | Thiết bị ảo (firewall, IDS/IPS…) |
| Transparent Network Gateway | Cổng mạng trong suốt |
| Deep Packet Inspection | Kiểm tra sâu gói tin |
| Sticky Sessions / Session Affinity | Phiên dính (giữ client ở cùng một instance) |
| Cross-Zone Load Balancing | Cân bằng tải xuyên vùng |
| Inter-AZ data | Dữ liệu truyền giữa các AZ |
| Certificate Authority (CA) | Tổ chức cấp chứng chỉ |
| In-flight / in-transit encryption | Mã hóa khi truyền |
| Server Name Indication (SNI) | Chỉ báo tên máy chủ |
| Security policy | Chính sách bảo mật (phiên bản TLS/cipher) |
| Connection Draining / Deregistration Delay | Rút cạn kết nối / Độ trễ hủy đăng ký |
| In-flight request | Request đang xử lý dở |
| De-register | Hủy đăng ký (khỏi target group) |
| Auto Scaling Group (ASG) | Nhóm tự động mở rộng |
| Launch Template | Mẫu khởi chạy |
| Launch Configuration | Cấu hình khởi chạy (đã lỗi thời) |
| Minimum / Desired / Maximum Capacity | Dung lượng tối thiểu / mong muốn / tối đa |
| Scaling Policy | Chính sách mở rộng |
| Target Tracking Scaling | Mở rộng bám mục tiêu |
| Simple / Step Scaling | Mở rộng đơn giản / theo bậc |
| Scheduled Scaling | Mở rộng theo lịch |
| Predictive Scaling | Mở rộng dự báo |
| Cooldown period | Khoảng thời gian nghỉ giữa các lần scaling |
| Warmup | Thời gian làm nóng instance mới |
| Self-healing | Tự phục hồi |

---

*Ghi chú: các phần Hands On được tóm tắt lại các bước thao tác chính trên AWS Console. Giao diện Console có thể thay đổi theo thời gian — logic và khái niệm vẫn giữ nguyên. Khi dọn dẹp phần này, LUÔN xóa Auto Scaling Group TRƯỚC, nếu không ASG sẽ tự tạo lại các instance bạn vừa terminate.*
