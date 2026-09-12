# Phần 11 — Classic Solutions Architecture Discussions

> Khóa học: *Ultimate AWS Certified Solutions Architect Associate 2026* (Stéphane Maarek) — SAA-C03
> Nguồn tham chiếu: `AWS Certified Solutions Architect Slides v48.pdf` (phần "Classic Solutions Architecture")

---

## Mục lục

| # | Bài giảng | Thời lượng | Loại |
|---|-----------|-----------|------|
| 122 | [Solutions Architecture Discussions Overview](#122-solutions-architecture-discussions-overview) | 1 phút | Video |
| 123 | [WhatsTheTime.com](#123-whatsthetimecom) | 11 phút | Video |
| 124 | [MyClothes.com](#124-myclothescom) | 10 phút | Video |
| 125 | [MyWordPress.com](#125-mywordpresscom) | 5 phút | Video |
| 126 | [Instantiating applications quickly](#126-instantiating-applications-quickly) | 3 phút | Video |
| 127 | [Beanstalk Overview](#127-beanstalk-overview) | 5 phút | Video |
| 128 | [Beanstalk Hands On](#128-beanstalk-hands-on) | 8 phút | Video |
| — | [Trắc nghiệm 8: Classic Solutions Architecture Discussions Quiz](#trắc-nghiệm-8-classic-solutions-architecture-discussions-quiz) | — | Quiz |

---

## 122. Solutions Architecture Discussions Overview

### Section Introduction ⭐

- ⭐ **Những kiến trúc giải pháp này là PHẦN HAY NHẤT của khóa học**
- ⭐ **Hãy hiểu cách TẤT CẢ công nghệ chúng ta đã học phối hợp với nhau**
- ⭐⭐ **Đây là phần bạn cần thoải mái 100%**
- ⭐ **Chúng ta sẽ thấy sự TIẾN HÓA trong tư duy của một Solutions Architect qua nhiều case study mẫu:**

| Case study | Chủ đề chính |
|------------|-------------|
| **WhatIsTheTime.com** | ⭐ **Stateless web app** (không cần database) |
| **MyClothes.com** | ⭐ **Stateful web app** (giỏ hàng, session, 3-tier) |
| **MyWordPress.com** | ⭐ **Stateful web app** (lưu ảnh chia sẻ, EBS vs EFS) |
| **Instantiating applications quickly** | ⭐ **Tăng tốc khởi tạo hạ tầng** |
| **Beanstalk** | ⭐ **Nền tảng triển khai cho developer** |

### Vì sao phần này quan trọng cho kỳ thi? ⭐

> Đề thi SAA-C03 **không hỏi định nghĩa** mà hỏi **tình huống kiến trúc**: *"Công ty X gặp vấn đề Y, giải pháp nào là tốt nhất?"*.
> Phần này dạy bạn **cách suy nghĩ tiến hóa từ đơn giản → phức tạp**, đúng thứ tự mà đề thi hay dẫn dắt.

### Tư duy chung xuyên suốt

```
Bắt đầu ĐƠN GIẢN
   │
   ├─► Gặp vấn đề (downtime, không scale được, mất dữ liệu…)
   │
   ├─► Thêm MỘT thành phần giải quyết đúng vấn đề đó
   │
   └─► Lặp lại → kiến trúc trưởng thành dần
```

---

## 123. WhatsTheTime.com

### Đề bài — Stateless Web App ⭐

- ⭐ **WhatIsTheTime.com cho phép mọi người biết bây giờ là mấy giờ**
- ⭐⭐ **Chúng ta KHÔNG cần database** (đây là ứng dụng **stateless**)
- ⭐ **Muốn bắt đầu nhỏ và CÓ THỂ CHẤP NHẬN downtime**
- ⭐ **Muốn scale đầy đủ CẢ chiều dọc lẫn chiều ngang, KHÔNG downtime**
- **Hãy đi qua hành trình của một Solutions Architect cho ứng dụng này**

---

### 🔹 Bước 1 — Starting simple (Bắt đầu đơn giản)

```
   User ──What time is it?──► [Elastic IP Address] ──► Public EC2
        ◄────── 5:30 pm! ──────────────────────────────┘
```

| Thành phần | Vai trò |
|-----------|---------|
| **1 EC2 instance public** | Chạy ứng dụng |
| ⭐ **Elastic IP Address** | **Giữ IP cố định** để user luôn truy cập được |

**Vấn đề:** chỉ có **một máy** → không chịu được tải, hỏng là mất dịch vụ.

---

### 🔹 Bước 2 — Scaling vertically (Scale theo chiều DỌC)

```
   User ──What time is it?──► [Elastic IP Address] ──► Public EC2 (t2.micro → M5)
        ◄── 5:30 pm! / 6:30 pm! / 7:30 pm! ──────────────┘
                  ▲
        ⚠️ DOWNTIME while upgrading to M5
```

- ⭐ **Nâng cấp instance type** (ví dụ `t2.micro` → `M5`) để phục vụ nhiều user hơn.
- ⚠️⭐⭐ **PHẢI STOP instance để đổi instance type → CÓ DOWNTIME**
- ⭐ **Elastic IP giữ nguyên** → user không phải đổi địa chỉ.

**Vấn đề:** ⭐ **Vertical scaling có GIỚI HẠN phần cứng** và **gây downtime**.

---

### 🔹 Bước 3 — Scaling horizontally (Scale theo chiều NGANG)

```
   User ──What time is it?──► Public EC2 instance #1 ──► 5:30 pm!
        ──What time is it?──► Public EC2 instance #2 ──► 6:30 pm!
        ──What time is it?──► Public EC2 instance #3 ──► 7:30 pm!
```

- ⭐ **Thêm NHIỀU EC2 instance**, mỗi cái có Elastic IP riêng.

⚠️ **Vấn đề:** ⭐⭐ **Elastic IP bị giới hạn 5 cái/account** → không scale ngang được nhiều.

---

### 🔹 Bước 4 — Dùng Route 53 thay Elastic IP ⭐

```
   DNS Query for api.whatisthetime.com
   ⭐ A Record — TTL 1 hour
            │
   User ────┼──► Public EC2 instance #1 (No Elastic IP) ──► 5:30 pm!
            ├──► Public EC2 instance #2 (No Elastic IP) ──► 6:30 pm!
            └──► Public EC2 instance #3 (No Elastic IP) ──► 7:30 pm!
```

- ⭐ **Bỏ Elastic IP**, dùng **Route 53 với A Record** trỏ tới public IP của các instance.
- ⭐ **TTL = 1 giờ**.

---

### 🔹 Bước 5 — Vấn đề khi thêm/bớt instance ⭐⭐

```
   DNS Query for api.whatisthetime.com
   ⭐ A Record — TTL 1 hour
            │
   User ────┼──► ❌ INSTANCE IS GONE!   ← client vẫn cache IP cũ 1 giờ!
            ├──► Public EC2 instance #2 ──► 6:30 pm!
            └──► Public EC2 instance #3 ──► 7:30 pm!
```

⚠️⭐⭐ **Đây là bài học quan trọng nhất về TTL:**

> **Khi một instance bị xóa, client vẫn CACHE IP cũ trong suốt TTL (1 giờ)**
> → **user gặp lỗi trong tối đa 1 giờ**.

**Bài học:** ⭐ **TTL phải đặt THẤP** khi hạ tầng thay đổi thường xuyên — nhưng kể cả vậy vẫn không hoàn hảo.

---

### 🔹 Bước 6 — Thêm Load Balancer ⭐⭐

```
   DNS Query for api.whatisthetime.com
   ⭐ Alias Record
            │
   User ────► ELB + Health Checks ──┬──► Private EC2 instance (AZ 1)
                                     └──► Private EC2 instance (AZ 1)
              ▲ Restricted Security groups rules
```

Thay đổi quan trọng:

| Trước | Sau | Lợi ích |
|-------|-----|---------|
| **A Record** | ⭐ **Alias Record** trỏ tới ELB | Miễn phí, tự cập nhật IP, dùng được root domain |
| **Public EC2** | ⭐⭐ **PRIVATE EC2 instances** | **Không truy cập trực tiếp từ internet** |
| Không kiểm tra | ⭐ **ELB + Health Checks** | Tự loại bỏ instance chết |
| SG mở rộng | ⭐⭐ **Restricted Security groups rules** | SG của EC2 chỉ cho phép traffic **từ SG của ELB** |

> ⭐ **Giải quyết triệt để vấn đề TTL ở bước 5:** client chỉ biết DNS của ELB, không biết IP instance nào cả.

---

### 🔹 Bước 7 — Thêm Auto Scaling Group ⭐

```
   DNS Query for api.whatisthetime.com — Alias Record
            │
   User ────► ELB + Health Checks ──┬──► ┌─ Auto Scaling group ─┐
                                     └──►│  EC2    EC2    EC2    │ (AZ 1)
                                          └──────────────────────┘
```

- ⭐ **ASG tự động thêm/bớt instance** theo tải.
- ⭐ **ASG tự động đăng ký instance mới vào ELB**.
- ⭐ **ASG tự tạo lại instance khi có cái chết**.

---

### 🔹 Bước 8 — Making our app Multi-AZ ⭐⭐

```
   DNS Query for api.whatisthetime.com — Alias Record
            │
   User ────► ⭐ ELB + Health Checks + MULTI AZ
                    │            │            │
              ┌─────▼──────┬─────▼──────┬─────▼──────┐
              │    AZ 1    │    AZ 2    │    AZ 3    │
              │   EC2      │   EC2      │   EC2      │
              └────────────┴────────────┴────────────┘
                  ⭐ Auto Scaling group (Availability zone 1 to 3)
```

- ⭐⭐ **Trải cả ELB lẫn ASG trên NHIỀU AZ** → **sống sót khi mất một Data Center**.

---

### 🔹 Bước 9 — Tối ưu chi phí ⭐

```
   ⭐ Minimum 2 AZ => Let's reserve capacity

   Minimum capacity = reserved instances = COST SAVINGS
```

- ⭐⭐ **Phần dung lượng TỐI THIỂU (minimum capacity) luôn chạy 24/7 → mua Reserved Instances để tiết kiệm chi phí**
- Phần scale thêm khi tải cao → dùng **On-Demand**.

> **Đây là mô hình tối ưu chi phí kinh điển:** **RI cho baseline + On-Demand cho peak**.

---

### 📋 In this lecture we've discussed… ⭐⭐

Tổng kết các khái niệm của bài này (nguyên văn slide):

| # | Chủ đề |
|---|--------|
| 1 | ⭐ **Public vs Private IP và EC2 instances** |
| 2 | ⭐ **Elastic IP vs Route 53 vs Load Balancers** |
| 3 | ⭐ **Route 53 TTL, A records và Alias Records** |
| 4 | ⭐ **Bảo trì EC2 instance THỦ CÔNG vs Auto Scaling Groups** |
| 5 | ⭐ **Multi-AZ để sống sót qua thảm họa** |
| 6 | ⭐ **ELB Health Checks** |
| 7 | ⭐ **Security Group Rules** |
| 8 | ⭐ **Đặt trước dung lượng (Reservation of capacity) để tiết kiệm chi phí khi có thể** |

### Bảng tiến hóa kiến trúc ⭐

| Bước | Kiến trúc | Vấn đề còn lại |
|------|-----------|----------------|
| 1 | 1 EC2 + Elastic IP | Không scale, hỏng là chết |
| 2 | Scale dọc (đổi instance type) | ⚠️ **Downtime**, giới hạn phần cứng |
| 3 | Nhiều EC2 + nhiều Elastic IP | ⚠️ **Giới hạn 5 Elastic IP** |
| 4 | Route 53 A Record + public IP | ⚠️ **TTL cache IP đã chết** |
| 5 | + ELB + Alias Record + Private EC2 | Vẫn phải thêm/bớt máy thủ công |
| 6 | + Auto Scaling Group | Chỉ 1 AZ → mất AZ là chết |
| 7 | + Multi-AZ | Chi phí cao |
| 8 | + Reserved Instances cho baseline | ✅ **Kiến trúc hoàn chỉnh** |

---

## 124. MyClothes.com

### Đề bài — Stateful Web App ⭐⭐

- ⭐ **MyClothes.com cho phép mọi người MUA QUẦN ÁO trực tuyến**
- ⭐⭐ **Có GIỎ HÀNG (shopping cart)**
- ⭐ **Website có HÀNG TRĂM user cùng lúc**
- ⭐ **Cần scale, duy trì horizontal scalability và giữ web app STATELESS NHẤT CÓ THỂ**
- ⭐⭐ **Người dùng KHÔNG được mất giỏ hàng của họ**
- ⭐ **Người dùng phải có thông tin chi tiết (địa chỉ, v.v.) trong một DATABASE**

---

### 🔹 Bước 1 — Xuất phát từ kiến trúc của WhatIsTheTime.com

```
   ELB (Multi AZ) ──┬──► AZ 1: EC2
                    ├──► AZ 2: EC2
                    └──► AZ 3: EC2
              ⭐ Auto Scaling group
```

⚠️ **Vấn đề:** ⭐⭐ **User gửi request tới instance A (bỏ đồ vào giỏ), request sau đi tới instance B → MẤT GIỎ HÀNG!**

---

### 🔹 Bước 2 — Introduce Stickiness (Session Affinity) ⭐

```
   ELB ──⭐ ELB Stickiness──┬──► AZ 1: EC2  ← user LUÔN về đúng instance này
                            ├──► AZ 2: EC2
                            └──► AZ 3: EC2
```

- ⭐ **Bật ELB Sticky Sessions** → cùng một client luôn về cùng một instance → **giỏ hàng được giữ**.

⚠️ **Vấn đề:** ⭐⭐ **Nếu instance đó CHẾT (hoặc ASG scale-in) → user VẪN MẤT giỏ hàng**. Và stickiness **gây mất cân bằng tải**.

---

### 🔹 Bước 3 — Introduce User Cookies ⭐⭐

```
   Client ──⭐ Send shopping cart content in Web Cookies──► ELB ──┬──► EC2 (AZ 1)
                                                                   ├──► EC2 (AZ 2)
                                                                   └──► EC2 (AZ 3)
```

- ⭐ **Lưu TOÀN BỘ nội dung giỏ hàng trong Web Cookie ở phía client**.
- ✅ ⭐⭐ **Ứng dụng trở nên STATELESS** — instance nào phục vụ cũng được.

### ⚠️⭐⭐ Nhược điểm của cách này (nguyên văn slide):

| # | Nhược điểm |
|---|-----------|
| 1 | ⭐ **HTTP requests NẶNG hơn** (mỗi request mang theo toàn bộ giỏ hàng) |
| 2 | ⭐⭐ **RỦI RO BẢO MẬT — cookie có thể bị sửa đổi (altered)** |
| 3 | ⭐ **Cookie PHẢI được xác thực (validated)** ở phía server |
| 4 | ⭐⭐ **Cookie phải NHỎ HƠN 4KB** |

> **Con số 4KB rất hay ra thi.**

---

### 🔹 Bước 4 — ElastiCache cho Session Store ⭐⭐⭐

```
   Client ──⭐ Send session_id in Web Cookies──► ELB ──┬──► EC2 (AZ 1)
                                                        ├──► EC2 (AZ 2)
                                                        └──► EC2 (AZ 3)
                                                              │
                                          ⭐ Store / retrieve session data
                                                              │
                                         ┌────────────────────┴────────────┐
                                         ▼                                 ▼
                                  ElastiCache (Multi AZ)      ⭐ Amazon DynamoDB (thay thế)
```

- ⭐⭐ **Chỉ gửi `session_id` trong cookie** (rất nhỏ, an toàn hơn)
- ⭐ **Dữ liệu session được LƯU/LẤY từ ElastiCache**
- ⭐⭐ **Giải pháp THAY THẾ: Amazon DynamoDB**

### So sánh 3 cách quản lý session ⭐⭐

| Cách | Ưu điểm | Nhược điểm |
|------|---------|-----------|
| **ELB Stickiness** | Đơn giản, không sửa code | ⚠️ Mất session khi instance chết, mất cân bằng tải |
| **Cookie chứa toàn bộ data** | Stateless hoàn toàn, không cần hạ tầng thêm | ⚠️ **< 4KB**, request nặng, **rủi ro bảo mật**, phải validate |
| ⭐ **ElastiCache / DynamoDB** | ⭐ **Stateless + an toàn + nhanh (sub-ms)** | Cần thêm hạ tầng, phải sửa code |

> **Đáp án đề thi thường là ElastiCache hoặc DynamoDB** khi hỏi "lưu session cho ứng dụng phân tán".

---

### 🔹 Bước 5 — Storing User Data in a database ⭐

```
   ELB ──┬──► EC2 (AZ 1) ──┐
         ├──► EC2 (AZ 2) ──┼── ElastiCache (session)
         └──► EC2 (AZ 3) ──┘
                │
       ⭐ Store / retrieve user data (address, name, etc)
                ▼
         Amazon RDS (Multi AZ)
```

- ⭐ **Dữ liệu người dùng lâu dài** (địa chỉ, tên…) → lưu trong **RDS**.
- ⭐ Phân biệt rõ: **ElastiCache = dữ liệu tạm (session)**, **RDS = dữ liệu bền vững (user data)**.

---

### 🔹 Bước 6 — Scaling Reads ⭐

```
              ┌── writes ──► RDS Master
   EC2 ───────┤                  │ ⭐ replication
              └── reads ───► RDS Read Replicas
```

- ⭐ **Thêm RDS Read Replicas** để **scale khả năng ĐỌC** (tối đa **15 replica**).

---

### 🔹 Bước 7 — Scaling Reads (Alternative) — Lazy Loading ⭐⭐

```
   EC2 ──⭐ Read from cache──► ElastiCache ──cache hit?──┐
                                    │ miss               │ hit → trả về ngay
                                    ▼                    │
                              ⭐ Read/write ──► RDS ──────┘
```

- ⭐⭐ **Giải pháp THAY THẾ cho Read Replicas: dùng ElastiCache với pattern LAZY LOADING**
- ⭐ **Cache hit** → trả về ngay, không chạm tới RDS.
- ⭐ **Cache miss** → đọc từ RDS rồi ghi vào cache.

> ⭐ **ElastiCache ở đây phục vụ HAI mục đích:** (1) lưu session, (2) cache dữ liệu từ RDS.

---

### 🔹 Bước 8 — Multi AZ — Survive disasters ⭐

```
   ⭐ Auto Scaling group (AZ 1, 2, 3)
   ⭐ ElastiCache Multi AZ
   ⭐ RDS Multi AZ
```

- ⭐ **MỌI tầng đều Multi-AZ**: ASG, ElastiCache, RDS → **sống sót khi mất một AZ**.

---

### 🔹 Bước 9 — Security Groups ⭐⭐⭐

Đây là **mô hình bảo mật phân tầng kinh điển** (nguyên văn slide):

```
   Internet
      │ ⭐ Open HTTP / HTTPS to 0.0.0.0/0
      ▼
   ┌──────────────┐
   │     ELB      │  SG: cho phép 80/443 từ BẤT CỨ ĐÂU
   └──────┬───────┘
          │ ⭐ Restrict traffic to EC2 Security group FROM THE LB
          ▼
   ┌──────────────┐
   │  EC2 (ASG)   │  SG: chỉ cho phép từ SG của ELB
   └──┬────────┬──┘
      │        │
      │        └─⭐ Restrict traffic to RDS Security group FROM THE EC2 security group
      │                    ▼
      │                   RDS         SG: chỉ cho phép từ SG của EC2
      │
      └─⭐ Restrict traffic to ElastiCache Security group FROM THE EC2 security group
                   ▼
              ElastiCache            SG: chỉ cho phép từ SG của EC2
```

### ⭐⭐ Nguyên tắc vàng:

> **Security Group tham chiếu LẪN NHAU (referencing each other), KHÔNG dùng IP.**
> Mỗi tầng chỉ cho phép traffic **từ đúng tầng phía trước nó**.

---

### 📋 In this lecture we've discussed… — 3-tier architectures for web applications ⭐⭐

Tổng kết (nguyên văn slide):

| Thành phần | Vai trò |
|-----------|---------|
| ⭐ **ELB sticky sessions** | Giữ user ở cùng instance |
| ⭐ **Web clients để lưu cookies và làm web app stateless** | Cookie chứa session_id |
| ⭐ **ElastiCache** | • **Lưu session** (thay thế: **DynamoDB**)<br>• **Cache dữ liệu từ RDS**<br>• **Multi AZ** |
| ⭐ **RDS** | • **Lưu dữ liệu người dùng**<br>• **Read replicas để scale reads**<br>• **Multi AZ cho disaster recovery** |
| ⭐ **Bảo mật chặt chẽ** | **Security groups tham chiếu lẫn nhau** |

---

## 125. MyWordPress.com

### Đề bài ⭐

- ⭐ **Tạo một website WordPress CÓ KHẢ NĂNG SCALE HOÀN TOÀN**
- ⭐⭐ **Website đó phải TRUY CẬP và HIỂN THỊ ĐÚNG các ảnh được upload**
- ⭐ **Dữ liệu người dùng và nội dung blog phải được lưu trong một MySQL database**

---

### 🔹 Bước 1 — RDS layer

```
   ELB ──┬──► AZ 1: EC2 ──┐
         ├──► AZ 2: EC2 ──┼──► ⭐ RDS Multi AZ (MySQL)
         └──► AZ 3: EC2 ──┘
      ⭐ Auto Scaling group
```

- ⭐ **RDS MySQL Multi-AZ** để lưu nội dung blog và user data.

---

### 🔹 Bước 2 — Scaling with Aurora: Multi AZ & Read Replicas ⭐

```
   ELB ──┬──► AZ 1: EC2 ──┐
         ├──► AZ 2: EC2 ──┼──► ⭐ Aurora MySQL
         └──► AZ 3: EC2 ──┘      • Multi AZ
      Auto Scaling group          • Read Replicas
```

- ⭐⭐ **Nâng cấp lên Aurora MySQL** để có **Multi-AZ và Read Replicas DỄ DÀNG hơn**.

---

### 🔹 Bước 3 — Storing images with EBS (single instance) ⭐

```
   ┌──── Availability zone 1 (Multi AZ) ────┐
   │   EC2 ──⭐ Send image──► Amazon EBS Volume
   └─────────────────────────────────────────┘
```

- ✅ **Hoạt động tốt khi CHỈ CÓ MỘT instance**.

---

### 🔹 Bước 4 — ⚠️ Vấn đề với EBS khi scale ngang ⭐⭐

```
   ┌── AZ 1 ──┐              ┌── AZ 2 ──┐
   │   EC2    │              │   EC2    │
   │    │     │              │    │     │
   │ Send image             │ Send image
   │    ▼     │              │    ▼     │
   │ EBS Volume A            │ EBS Volume B
   └──────────┘              └──────────┘
        ⚠️⭐⭐ Ảnh upload lên instance 1 KHÔNG thấy được từ instance 2!
```

⚠️⭐⭐ **Đây là vấn đề cốt lõi của bài này:**

> **EBS volume chỉ gắn được vào MỘT instance và bị khóa trong MỘT AZ**
> → **mỗi instance có một tập ảnh riêng** → **user upload ảnh xong, lần sau vào lại không thấy ảnh!**

---

### 🔹 Bước 5 — Storing images with EFS ⭐⭐⭐

```
   ┌── AZ 1 ──┐                      ┌── AZ 2 ──┐
   │   EC2    │                      │   EC2    │
   │    │     │                      │    │     │
   │  ⭐ ENI   │                      │  ⭐ ENI   │
   └────┼─────┘                      └────┼─────┘
        └──────────► ⭐ EFS ◄──────────────┘
                (Send image — dùng chung)
```

- ⭐⭐ **EFS là network file system dùng chung, mount được lên NHIỀU instance XUYÊN AZ**
- ⭐ **Mỗi instance kết nối tới EFS qua một ENI (mount target) trong AZ của nó**
- ✅ **MỌI instance đều thấy CÙNG một tập ảnh** → vấn đề được giải quyết triệt để.

---

### 📋 In this lecture we've discussed… ⭐⭐

Nguyên văn slide:

| # | Chủ đề |
|---|--------|
| 1 | ⭐ **Aurora Database để có Multi-AZ và Read-Replicas DỄ DÀNG** |
| 2 | ⭐⭐ **Lưu dữ liệu trong EBS (ứng dụng MỘT instance)** |
| 3 | ⭐⭐ **VS Lưu dữ liệu trong EFS (ứng dụng PHÂN TÁN)** |

### ⭐⭐ Bảng quyết định EBS vs EFS (rất hay ra thi)

| Tình huống | Chọn |
|-----------|------|
| ⭐ **Ứng dụng chạy trên MỘT instance** | **EBS** |
| ⭐ **Ứng dụng PHÂN TÁN, nhiều instance cùng đọc/ghi một tập file** | **EFS** |
| ⭐ **WordPress với ảnh upload + Auto Scaling** | ⭐⭐ **EFS** |
| Cần IOPS cực cao, dữ liệu tạm | Instance Store |

> **Mẹo thi:** Đề nhắc **"WordPress"** + **"nhiều instance"** + **"shared media/uploads"** → đáp án gần như luôn là **EFS**.

---

## 126. Instantiating applications quickly

### Vấn đề ⭐

**Khi khởi chạy một full stack (EC2, EBS, RDS), có thể mất nhiều thời gian để:**

- ⭐ **Cài đặt ứng dụng** (Install applications)
- ⭐ **Chèn dữ liệu ban đầu (hoặc dữ liệu khôi phục)** (Insert initial or recovery data)
- ⭐ **Cấu hình mọi thứ** (Configure everything)
- ⭐ **Khởi chạy ứng dụng** (Launch the application)

> ⭐ **Chúng ta có thể tận dụng cloud để TĂNG TỐC việc đó!**

---

### ⭐⭐⭐ Giải pháp theo từng loại tài nguyên (BẢNG QUAN TRỌNG NHẤT BÀI NÀY)

#### 🔹 EC2 Instances

| Cách | Mô tả |
|------|-------|
| ⭐⭐ **Golden AMI** | **Cài sẵn ứng dụng, dependency của OS, v.v. TRƯỚC**, rồi **launch EC2 instance TỪ Golden AMI** |
| ⭐ **Bootstrap using User Data** | **Dùng script User Data cho cấu hình ĐỘNG (dynamic configuration)** |
| ⭐⭐ **Hybrid** | **Trộn Golden AMI và User Data** — ví dụ: ⭐ **Elastic Beanstalk** |

#### 🔹 RDS Databases

| Cách | Mô tả |
|------|-------|
| ⭐⭐ **Restore from a snapshot** | **Database sẽ có sẵn SCHEMA và DỮ LIỆU!** |

#### 🔹 EBS Volumes

| Cách | Mô tả |
|------|-------|
| ⭐⭐ **Restore from a snapshot** | **Đĩa sẽ được FORMAT SẴN và CÓ SẴN DỮ LIỆU!** |

### Bảng tổng hợp ⭐

| Tài nguyên | Kỹ thuật tăng tốc | Kết quả |
|-----------|-------------------|---------|
| **EC2** | ⭐ **Golden AMI** | Boot nhanh, phần mềm cài sẵn |
| **EC2** | ⭐ **User Data** | Cấu hình động lúc chạy |
| **EC2** | ⭐ **Hybrid (Beanstalk)** | Kết hợp cả hai |
| **RDS** | ⭐ **Restore from snapshot** | Có sẵn schema + data |
| **EBS** | ⭐ **Restore from snapshot** | Có sẵn filesystem + data |

> ⭐ **Liên hệ với phần 8:** Bài 85 có lời khuyên *"dùng AMI ready-to-use để giảm cooldown period của ASG"* — đây chính là **Golden AMI**.

---

### Typical architecture: Web App 3-tier ⭐⭐⭐

Đây là **kiến trúc 3 tầng chuẩn** — hình mẫu cho rất nhiều câu hỏi thi:

```
                          Route 53
                             │
   ┌─────────────────── PUBLIC SUBNET ───────────────────┐
   │                        ELB                           │
   └─────────────────────────┬────────────────────────────┘
                             │
   ┌─────────────────── PRIVATE SUBNET ──────────────────┐
   │   ⭐ Auto Scaling group                               │
   │   AZ 1: EC2      AZ 2: EC2      AZ 3: EC2            │
   └──────────┬───────────────────────────┬───────────────┘
              │                           │
   ┌──────────┼────────── DATA SUBNET ────┼───────────────┐
   │          ▼                           ▼               │
   │   ⭐ ElastiCache (Multi AZ)    ⭐ Amazon RDS           │
   │   Store/retrieve session      Read / write data      │
   │   data + Cached data                                  │
   └───────────────────────────────────────────────────────┘
```

### ⭐⭐ Ba tầng (3 tiers)

| Tầng | Subnet | Thành phần | Nhiệm vụ |
|------|--------|-----------|----------|
| **Tier 1 — Presentation** | ⭐ **PUBLIC SUBNET** | **Route 53 + ELB** | Tiếp nhận traffic từ internet |
| **Tier 2 — Application** | ⭐ **PRIVATE SUBNET** | **ASG + EC2** | Xử lý logic nghiệp vụ |
| **Tier 3 — Data** | ⭐ **DATA SUBNET** | **ElastiCache + RDS** | Lưu trữ session, cache và dữ liệu bền vững |

> ⭐⭐ **Ghi nhớ:** Chỉ tầng đầu tiên nằm ở **public subnet**. EC2 và database đều ở **private subnet** — đây là câu trả lời chuẩn cho mọi câu hỏi về bảo mật kiến trúc web.

---

## 127. Beanstalk Overview

### Developer problems on AWS ⭐

Những vấn đề developer gặp phải (nguyên văn slide):

- ⭐ **Quản lý hạ tầng (Managing infrastructure)**
- ⭐ **Triển khai code (Deploying Code)**
- ⭐ **Cấu hình tất cả database, load balancer, v.v.**
- ⭐ **Lo lắng về scaling (Scaling concerns)**
- ⭐⭐ **Hầu hết web app có CÙNG MỘT kiến trúc (ALB + ASG)**
- ⭐ **Tất cả điều developer muốn chỉ là CODE CỦA HỌ CHẠY ĐƯỢC!**
- ⭐ **Và tốt nhất là NHẤT QUÁN giữa các ứng dụng và môi trường khác nhau**

---

### Elastic Beanstalk — Overview ⭐⭐

- ⭐⭐ **Elastic Beanstalk là GÓC NHÌN HƯỚNG DEVELOPER về việc triển khai ứng dụng trên AWS**
- ⭐ **Nó dùng TẤT CẢ các thành phần ta đã học: EC2, ASG, ELB, RDS, …**
- ⭐ **Là MANAGED SERVICE**:
  - ⭐ **Tự động xử lý capacity provisioning, load balancing, scaling, application health monitoring, instance configuration, …**
  - ⭐⭐ **CHỈ MÃ NGUỒN ỨNG DỤNG là trách nhiệm của developer**
- ⭐ **Chúng ta VẪN có TOÀN QUYỀN kiểm soát cấu hình**
- ⭐⭐ **Beanstalk MIỄN PHÍ nhưng bạn trả tiền cho các instance bên dưới**

---

### Elastic Beanstalk — Components ⭐⭐

| Component | Định nghĩa |
|-----------|-----------|
| ⭐ **Application** | **Tập hợp các thành phần Elastic Beanstalk** (environments, versions, configurations, …) |
| ⭐ **Application Version** | **Một phiên bản (iteration) của mã nguồn ứng dụng** |
| ⭐ **Environment** | • **Tập hợp tài nguyên AWS chạy MỘT application version** (⭐ **chỉ một version tại một thời điểm**)<br>• ⭐⭐ **Tiers: Web Server Environment Tier & Worker Environment Tier**<br>• ⭐ **Có thể tạo NHIỀU environment (dev, test, prod, …)** |

### Quy trình làm việc ⭐

```
   Create Application ──► Upload Version ──► Launch Environment
                                                    │
                              ┌─────────────────────┘
                              ▼
                     Manage Environment
                              │
              ┌───────────────┴──────────────┐
              ▼                              ▼
        update version              deploy new version
```

---

### Elastic Beanstalk — Supported Platforms ⭐

| Ngôn ngữ / Nền tảng |
|---------------------|
| **Go** |
| **Java SE** |
| **Java with Tomcat** |
| **.NET Core on Linux** |
| **.NET on Windows Server** |
| **Node.js** |
| **PHP** |
| **Python** |
| **Ruby** |
| **Packer Builder** |
| ⭐ **Single Container Docker** |
| ⭐ **Multi-container Docker** |
| ⭐ **Preconfigured Docker** |

---

### ⭐⭐⭐ Web Server Tier vs. Worker Tier (rất hay ra thi)

#### Web Environment

```
   myapp.us-east-1.elasticbeanstalk.com
              │
             ELB
              │
   ┌──── Security Group ────┐
   │  Auto Scaling group     │
   │  AZ 1: EC2 (Web Server) │
   │  AZ 2: EC2 (Web Server) │
   └─────────────────────────┘
```

#### Worker Environment

```
        ⭐ SQS Queue
         │  │  │  (SQS messages)
         ▼  ▼  ▼   ⭐ pull messages
   ┌──── Security Group ────┐
   │  Auto Scaling group     │
   │  AZ 1: EC2 (Worker)     │
   │  AZ 2: EC2 (Worker)     │
   └─────────────────────────┘
```

### Đặc điểm Worker Tier ⭐⭐

- ⭐⭐ **Scale dựa trên SỐ LƯỢNG SQS MESSAGES**
- ⭐ **Có thể đẩy message vào SQS queue TỪ một Web Server Tier khác**

### Bảng so sánh ⭐

| | **Web Server Tier** | **Worker Tier** |
|---|---|---|
| **Nhận việc từ** | ⭐ **HTTP request qua ELB** | ⭐ **SQS Queue** |
| **Có URL public** | ✅ `myapp.us-east-1.elasticbeanstalk.com` | ❌ Không |
| **Scale theo** | CPU, request count | ⭐⭐ **Số lượng SQS message** |
| **Use case** | Website, API | ⭐ **Xử lý nền lâu (video encoding, gửi email, báo cáo)** |

> **Mẫu kiến trúc kinh điển:** ⭐ **Web Tier nhận request → đẩy job vào SQS → Worker Tier xử lý nền** → người dùng không phải chờ.

---

### ⭐⭐ Elastic Beanstalk Deployment Modes

| | **Single Instance** | **High Availability with Load Balancer** |
|---|---|---|
| **Dùng cho** | ⭐ **Great for DEV** | ⭐ **Great for PROD** |
| **Compute** | **1 EC2 Instance** + ⭐ **Elastic IP** | ⭐ **ALB + Auto Scaling Group** (AZ 1, AZ 2) |
| **Database** | ⭐ **RDS Master** (một mình) | ⭐ **RDS Master + RDS Standby** (Multi-AZ) |
| **Chi phí** | Thấp | Cao |

```
   Single Instance (DEV)              High Availability (PROD)
   ┌── AZ 1 ──────────┐               ┌── AZ 1 ──┐  ┌── AZ 2 ──┐
   │  Elastic IP       │                     ALB
   │  EC2 Instance     │               │   EC2    │  │   EC2    │
   │  RDS Master       │               │          │  │          │
   └───────────────────┘               │RDS Master│  │RDS Standby│
                                       └──────────┘  └──────────┘
                                        ⭐ Auto Scaling Group
```

> **Mẹo thi:** Đề nhắc **"môi trường dev, tiết kiệm chi phí"** → **Single Instance**. Đề nhắc **"production, high availability"** → **Load Balanced + ASG**.

---

## 128. Beanstalk Hands On

### Bước 1 — Tạo Application & Environment

1. Console → tìm **Elastic Beanstalk** → **Create application**.
2. **Environment tier**: ⭐ chọn
   - **Web server environment** (có URL, phục vụ HTTP), hoặc
   - **Worker environment** (xử lý SQS)
3. **Application information**:
   - **Application name**: `MyFirstApp`
4. **Environment information**:
   - **Environment name**: `MyFirstApp-env` (tự điền)
   - **Domain**: kiểm tra tính khả dụng → URL sẽ là `myfirstapp-env.eba-xxxx.us-east-1.elasticbeanstalk.com`
5. **Platform**:
   - **Platform type**: `Managed platform`
   - **Platform**: ví dụ ⭐ **Node.js** / **PHP** / **Python**
   - **Platform branch / version**: để mặc định (Recommended)
6. **Application code**:
   - ⭐ **Sample application** (khuyến nghị khi học)
   - hoặc **Upload your code** (file `.zip` / `.war`)
7. **Presets**: ⭐
   - **Single instance (free tier eligible)** — cho dev/học
   - **High availability** — ALB + ASG (⚠️ tốn phí)

### Bước 2 — Cấu hình Service Access ⭐

1. **Service role**: `Create and use new service role` (hoặc chọn role có sẵn)
2. **EC2 key pair**: chọn key nếu muốn SSH vào
3. ⭐ **EC2 instance profile**: chọn/tạo role — thường cần policy:
   - `AWSElasticBeanstalkWebTier`
   - `AWSElasticBeanstalkWorkerTier`
   - `AWSElasticBeanstalkMulticontainerDocker`

> ⚠️ Nếu thiếu instance profile, việc tạo environment sẽ **thất bại** — đây là lỗi phổ biến nhất.

### Bước 3 — Các bước cấu hình còn lại

| Step | Nội dung |
|------|----------|
| **Set up networking, database and tags** | VPC, subnet, ⭐ **có thể tạo RDS kèm theo environment** |
| **Configure instance traffic and scaling** | Instance type, ⭐ **Min/Max instances**, scaling triggers |
| **Configure updates, monitoring, and logging** | ⭐ **Health reporting (Basic/Enhanced)**, deployment policy, log streaming tới CloudWatch |

4. **Review** → **Submit** → đợi **~5–10 phút**.

### Bước 4 — Quan sát những gì Beanstalk TẠO RA ⭐⭐

Đây là phần quan trọng nhất của bài hands-on — Beanstalk **tự động tạo** cho bạn:

| Dịch vụ | Tài nguyên được tạo |
|---------|---------------------|
| **EC2** | ⭐ **Instance(s)** chạy ứng dụng |
| **EC2** | ⭐ **Security Groups** |
| **EC2** | ⭐ **Auto Scaling Group** (nếu chọn High availability) |
| **EC2** | ⭐ **Load Balancer** (nếu chọn High availability) |
| **CloudFormation** | ⭐⭐ **Một STACK mô tả toàn bộ hạ tầng** |
| **S3** | ⭐ **Bucket `elasticbeanstalk-<region>-<account-id>`** chứa application version |
| **CloudWatch** | **Alarms và log groups** |

> ⭐⭐ **Điểm mấu chốt:** Vào **CloudFormation → Stacks** để thấy Beanstalk thực chất **chỉ là một lớp bọc trên CloudFormation**. Nó không phải "phép màu" — nó tạo ra đúng những tài nguyên bạn đã học ở các phần trước.

### Bước 5 — Truy cập và quản lý ứng dụng

1. Bấm vào **Domain URL** ở đầu trang → thấy trang mẫu của platform đã chọn.
2. Tab **Health**: xem trạng thái instance (`Ok`, `Warning`, `Degraded`, `Severe`).
3. Tab **Logs** → **Request logs** → `Last 100 lines` hoặc `Full logs`.
4. Tab **Monitoring**: biểu đồ CPU, request count, latency.
5. Tab **Events**: nhật ký mọi thao tác Beanstalk thực hiện.

### Bước 6 — Deploy version mới ⭐

1. Sửa code → nén thành `.zip`.
2. Environment → **Upload and deploy**:
   - Chọn file `.zip`
   - **Version label**: `v2`
3. **Deploy** → Beanstalk cập nhật instance theo **deployment policy** đã cấu hình.
4. ⭐ **Application versions** → thấy lịch sử các version, có thể ⭐ **rollback** về version cũ.

### Bước 7 — Dọn dẹp ⚠️

```
Elastic Beanstalk → Environments → chọn environment → Actions → Terminate environment
Elastic Beanstalk → Applications → chọn application → Actions → Delete application
```

Sau đó kiểm tra:

- [ ] **CloudFormation** → stack đã bị xóa chưa
- [ ] **EC2** → instance đã terminate chưa
- [ ] **EC2** → Load Balancer, Target Group đã xóa chưa
- [ ] ⚠️ ⭐ **S3** → bucket `elasticbeanstalk-*` **KHÔNG tự xóa** — phải xóa thủ công (có Bucket Policy chặn xóa, phải gỡ policy trước)
- [ ] **RDS** → nếu đã tạo DB kèm environment, kiểm tra nó đã bị xóa chưa

> ⚠️ ⭐ **Terminate environment là cách đúng** — đừng xóa từng EC2/ELB thủ công, vì CloudFormation stack sẽ tạo lại hoặc rơi vào trạng thái lỗi.

---

## Trắc nghiệm 8: Classic Solutions Architecture Discussions Quiz

### Các điểm dễ bị bẫy

| Câu hỏi thường gặp | Đáp án đúng |
|--------------------|-------------|
| Scale dọc (đổi instance type) có downtime không? | ⭐ **CÓ** — phải stop instance |
| Vì sao không dùng nhiều Elastic IP để scale ngang? | ⭐ **Giới hạn 5 Elastic IP/account** |
| Dùng A Record trỏ tới EC2 public IP, instance bị xóa thì sao? | ⭐⭐ **Client vẫn cache IP chết trong suốt TTL** |
| Giải pháp triệt để cho vấn đề TTL trên là gì? | ⭐ **Đặt ELB phía trước + dùng Alias Record** |
| Sau khi có ELB, EC2 nên đặt ở đâu? | ⭐⭐ **Private subnet** (private instances) |
| SG của EC2 nên cho phép traffic từ đâu? | ⭐⭐ **Từ SG của Load Balancer** (không dùng IP) |
| Phần capacity tối thiểu chạy 24/7 nên mua gì? | ⭐ **Reserved Instances** |
| Vấn đề khi user gửi request tới instance khác trong ASG? | ⭐ **Mất giỏ hàng (session)** |
| Giải pháp đơn giản nhất giữ session? | **ELB Stickiness** (nhưng mất khi instance chết) |
| Cookie lưu giỏ hàng có giới hạn kích thước bao nhiêu? | ⭐⭐ **Phải NHỎ HƠN 4KB** |
| Nhược điểm của lưu toàn bộ giỏ hàng trong cookie? | ⭐ **Request nặng, rủi ro bảo mật (cookie bị sửa), phải validate, < 4KB** |
| Giải pháp tốt nhất lưu session cho app phân tán? | ⭐⭐ **ElastiCache** (thay thế: **DynamoDB**) |
| Cookie nên chứa gì thay vì toàn bộ data? | ⭐ **`session_id`** |
| Dữ liệu user lâu dài (địa chỉ, tên) lưu ở đâu? | ⭐ **RDS** |
| Hai cách scale đọc cho RDS? | ⭐ **Read Replicas** hoặc **ElastiCache + Lazy Loading** |
| ElastiCache trong MyClothes.com phục vụ mấy mục đích? | ⭐ **2: lưu session + cache dữ liệu RDS** |
| Nguyên tắc cấu hình Security Group giữa các tầng? | ⭐⭐ **Tham chiếu SG lẫn nhau, không dùng IP** |
| WordPress nhiều instance, ảnh upload lưu ở đâu? | ⭐⭐ **EFS** (không phải EBS) |
| Vì sao EBS không hợp với WordPress multi-instance? | ⭐⭐ **EBS gắn 1 instance, khóa 1 AZ → mỗi máy một tập ảnh** |
| EBS phù hợp khi nào? | ⭐ **Ứng dụng một instance** |
| EFS phù hợp khi nào? | ⭐ **Ứng dụng phân tán, nhiều instance chung file** |
| Instance kết nối EFS qua gì? | ⭐ **ENI (mount target)** trong AZ của nó |
| Database nào cho WordPress dễ Multi-AZ + Read Replicas? | ⭐ **Aurora MySQL** |
| Tăng tốc khởi tạo EC2 bằng cách nào? | ⭐⭐ **Golden AMI** |
| Cấu hình động lúc chạy dùng gì? | ⭐ **User Data script** |
| Dịch vụ nào là ví dụ của Hybrid (AMI + User Data)? | ⭐ **Elastic Beanstalk** |
| Tăng tốc khởi tạo RDS / EBS? | ⭐⭐ **Restore from a snapshot** |
| Kiến trúc 3-tier: tầng nào ở public subnet? | ⭐⭐ **CHỈ ELB** (Route 53 + ELB) |
| EC2 và RDS nằm ở subnet nào? | ⭐ **Private subnet / Data subnet** |
| Beanstalk có tính phí không? | ⭐⭐ **MIỄN PHÍ — chỉ trả tiền cho instance bên dưới** |
| Developer chịu trách nhiệm gì trong Beanstalk? | ⭐ **CHỈ mã nguồn ứng dụng** |
| 3 thành phần của Beanstalk? | ⭐ **Application / Application Version / Environment** |
| Một Environment chạy được mấy version cùng lúc? | ⭐ **CHỈ MỘT** |
| Beanstalk có mấy loại environment tier? | ⭐⭐ **2: Web Server Tier & Worker Tier** |
| Worker Tier nhận việc từ đâu? | ⭐⭐ **SQS Queue** |
| Worker Tier scale dựa trên gì? | ⭐⭐ **Số lượng SQS messages** |
| 2 deployment mode của Beanstalk? | ⭐ **Single Instance (dev)** và **HA with Load Balancer (prod)** |
| Single Instance mode có gì? | **1 EC2 + Elastic IP + RDS Master** |
| HA mode có gì? | **ALB + ASG (2 AZ) + RDS Master + Standby** |
| Beanstalk dùng dịch vụ nào bên dưới để tạo hạ tầng? | ⭐⭐ **CloudFormation** |
| Beanstalk lưu application version ở đâu? | ⭐ **S3 bucket `elasticbeanstalk-*`** |

### Checklist tự kiểm tra trước khi làm quiz

- [ ] Nhớ **hành trình tiến hóa của WhatIsTheTime.com** (8 bước) và **vấn đề của từng bước**
- [ ] Nhớ **vấn đề TTL** khi dùng A Record trỏ public IP → giải pháp **ELB + Alias**
- [ ] Nhớ **giới hạn 5 Elastic IP** là lý do không scale ngang bằng Elastic IP
- [ ] Thuộc **3 cách quản lý session**: Stickiness / Cookie (**< 4KB**) / **ElastiCache-DynamoDB**
- [ ] Nhớ **4 nhược điểm của cookie**: request nặng, bảo mật, phải validate, < 4KB
- [ ] Nhớ **ElastiCache 2 công dụng**: session store + cache RDS (Lazy Loading)
- [ ] Nhớ **mô hình Security Group phân tầng** — SG tham chiếu lẫn nhau
- [ ] Thuộc **EBS (1 instance) vs EFS (phân tán)** — WordPress → **EFS**
- [ ] Nhớ **Golden AMI / User Data / Hybrid** và **restore from snapshot** cho RDS-EBS
- [ ] Nhớ **kiến trúc 3-tier**: public (ELB) / private (EC2+ASG) / data (RDS+ElastiCache)
- [ ] Nhớ **Beanstalk miễn phí**, **3 components**, **2 tiers**, **2 deployment modes**
- [ ] Nhớ **Worker Tier ↔ SQS** và **Beanstalk chạy trên CloudFormation**

---

## Thuật ngữ Anh — Việt

| Tiếng Anh | Tiếng Việt |
|-----------|-----------|
| Solutions Architecture | Kiến trúc giải pháp |
| Case study | Tình huống nghiên cứu mẫu |
| Stateless / Stateful | Không lưu trạng thái / Có lưu trạng thái |
| Scaling vertically | Mở rộng theo chiều dọc (tăng kích thước máy) |
| Scaling horizontally | Mở rộng theo chiều ngang (tăng số máy) |
| Downtime | Thời gian gián đoạn dịch vụ |
| Shopping cart | Giỏ hàng |
| Session | Phiên làm việc của người dùng |
| Session Affinity / Stickiness | Phiên dính (giữ user ở cùng một máy) |
| Web Cookies | Cookie lưu ở trình duyệt |
| Altered | Bị sửa đổi (trái phép) |
| Validated | Được xác thực |
| Session Store | Kho lưu phiên |
| Lazy Loading | Nạp lười (chỉ cache khi có người đọc) |
| Cache hit / miss | Trúng / trượt bộ nhớ đệm |
| Read Replicas | Bản sao chỉ đọc |
| Multi AZ | Đa vùng sẵn sàng |
| Survive disasters | Sống sót qua thảm họa |
| Referencing each other | Tham chiếu lẫn nhau (giữa các Security Group) |
| 3-tier architecture | Kiến trúc ba tầng |
| Public / Private / Data subnet | Mạng con công khai / riêng tư / dữ liệu |
| Reserved instances | Instance đặt trước (giá rẻ hơn) |
| Cost savings | Tiết kiệm chi phí |
| Instantiating | Khởi tạo |
| Golden AMI | Ảnh máy chuẩn đã cài sẵn phần mềm |
| Bootstrap | Khởi tạo cấu hình lúc máy khởi động |
| Restore from a snapshot | Khôi phục từ bản chụp |
| Schema | Lược đồ cơ sở dữ liệu |
| Elastic Beanstalk | Nền tảng triển khai ứng dụng của AWS |
| Developer centric | Hướng lập trình viên |
| Capacity provisioning | Cấp phát năng lực tính toán |
| Application / Application Version | Ứng dụng / Phiên bản ứng dụng |
| Environment | Môi trường (tập tài nguyên chạy một version) |
| Web Server Tier / Worker Tier | Tầng máy chủ web / Tầng xử lý nền |
| SQS Queue | Hàng đợi tin nhắn |
| Pull messages | Lấy tin nhắn về xử lý |
| Deployment Mode | Chế độ triển khai |
| Managed platform | Nền tảng được quản lý |
| Instance profile | Hồ sơ quyền gắn vào EC2 |
| Rollback | Quay về phiên bản trước |
| Terminate environment | Hủy môi trường |

---

*Ghi chú: đây là chương lý thuyết kiến trúc, phần lớn là sơ đồ trên slide — tôi đã dựng lại bằng sơ đồ ASCII theo đúng từng bước tiến hóa mà giảng viên trình bày. Chỉ bài 128 (Beanstalk Hands On) là thao tác Console; nhớ **Terminate environment** thay vì xóa thủ công từng tài nguyên, và **xóa bucket `elasticbeanstalk-*` trong S3** vì nó không tự xóa.*
