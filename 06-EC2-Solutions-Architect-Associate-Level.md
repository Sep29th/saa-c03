# Phần 6 — EC2: Solutions Architect Associate Level

> Khóa học: *Ultimate AWS Certified Solutions Architect Associate 2026* (Stéphane Maarek) — SAA-C03
> Nguồn tham chiếu: `AWS Certified Solutions Architect Slides v48.pdf` (phần "Amazon EC2 – Associate")

---

## Mục lục

| # | Bài giảng | Thời lượng | Loại |
|---|-----------|-----------|------|
| 47 | [Private vs Public vs Elastic IP](#47-private-vs-public-vs-elastic-ip) | 5 phút | Video |
| 48 | [Private vs Public vs Elastic IP Hands On](#48-private-vs-public-vs-elastic-ip-hands-on) | 6 phút | Video |
| 49 | [EC2 Placement Groups](#49-ec2-placement-groups) | 6 phút | Video |
| 50 | [EC2 Placement Groups - Hands On](#50-ec2-placement-groups---hands-on) | 2 phút | Video |
| 51 | [Elastic Network Interfaces (ENI) - Overview](#51-elastic-network-interfaces-eni---overview) | 2 phút | Video |
| 52 | [Elastic Network Interfaces (ENI) - Hands On](#52-elastic-network-interfaces-eni---hands-on) | 5 phút | Video |
| 53 | [ENI - Extra Reading](#53-eni---extra-reading) | 1 phút | Bài viết |
| 54 | [EC2 Hibernate](#54-ec2-hibernate) | 3 phút | Video |
| 55 | [EC2 Hibernate - Hands On](#55-ec2-hibernate---hands-on) | 4 phút | Video |
| — | [Trắc nghiệm 3: EC2 SAA Level Quiz](#trắc-nghiệm-3-ec2-saa-level-quiz) | — | Quiz |

---

## 47. Private vs Public vs Elastic IP

### IPv4 vs IPv6

Mạng có **hai loại IP**:

| | IPv4 | IPv6 |
|---|------|------|
| Ví dụ | `1.160.10.240` | `3ffe:1900:4545:3:200:f8ff:fe21:67cf` |
| Định dạng | `[0-255].[0-255].[0-255].[0-255]` | 8 nhóm hexa |
| Số địa chỉ public | **3.7 tỷ** địa chỉ | Gần như vô hạn |
| Mức phổ biến | **Vẫn phổ biến nhất trên internet** | Mới hơn, giải quyết bài toán **IoT** |

> **Khóa học chỉ dùng IPv4.**

### Public IP vs Private IP — Khác biệt cơ bản ⭐

#### Public IP

- Máy **có thể được nhận diện trên internet (WWW)**.
- **Phải duy nhất trên toàn bộ web** — không hai máy nào có cùng public IP.
- **Có thể định vị địa lý (geo-locate) dễ dàng**.

#### Private IP

- Máy **chỉ được nhận diện trong mạng nội bộ (private network)**.
- IP phải **duy nhất trong mạng nội bộ đó**.
- **NHƯNG hai mạng nội bộ khác nhau (hai công ty) CÓ THỂ dùng chung dải IP giống hệt nhau**.
- Máy kết nối ra WWW thông qua **NAT + Internet Gateway** (đóng vai trò proxy).
- **Chỉ một dải IP nhất định** được dùng làm private IP.

### Sơ đồ minh họa (từ slide)

```
                    Server (public):          Web Server (public):
                    211.139.37.43             79.216.59.75
                            │                        │
                            └────────  WWW  ─────────┘
                                    ╱       ╲
              Internet Gateway     ╱         ╲    Internet Gateway
              (public):           ╱           ╲   (public):
              149.140.72.10      ╱             ╲  253.144.139.205
                    │                                  │
        ┌───────────────────────┐         ┌───────────────────────┐
        │      Company A        │         │      Company B        │
        │   Private Network     │         │   Private Network     │
        │   192.168.0.1/22      │         │   192.168.0.1/22      │
        └───────────────────────┘         └───────────────────────┘
             ▲ Hai công ty dùng CHUNG dải private IP → vẫn hợp lệ!
```

### Các dải Private IP (RFC 1918)

| Dải | CIDR | Số địa chỉ |
|-----|------|-----------|
| `10.0.0.0` – `10.255.255.255` | `10.0.0.0/8` | ~16.7 triệu |
| `172.16.0.0` – `172.31.255.255` | `172.16.0.0/12` | ~1 triệu |
| `192.168.0.0` – `192.168.255.255` | `192.168.0.0/16` | ~65 nghìn |

> **AWS VPC mặc định dùng `172.31.0.0/16`** — đó là lý do EC2 instance thường có private IP dạng `172.31.x.x`.

### Trong AWS EC2 (Hands-On context)

**Mặc định, mỗi EC2 instance đi kèm:**

- **Một private IP** — cho mạng nội bộ AWS
- **Một public IP** — cho WWW

**Khi SSH vào EC2 instance:**

- **KHÔNG dùng được private IP** vì bạn **không ở cùng mạng** với instance.
- **Chỉ dùng được public IP**.

**Quan trọng:** Nếu máy bị **stop rồi start lại**, **public IP có thể thay đổi**. ⭐

### Elastic IP ⭐

**Vấn đề:** Khi stop rồi start EC2 instance, **public IP đổi** → mọi thứ trỏ tới IP cũ đều hỏng.

**Giải pháp: Elastic IP**

- Là **một public IPv4** mà **bạn sở hữu chừng nào bạn chưa xóa nó**.
- **Gắn được vào MỘT instance tại một thời điểm**.
- Với Elastic IP, bạn có thể **che giấu lỗi của một instance hoặc phần mềm** bằng cách **nhanh chóng remap địa chỉ sang instance khác** trong tài khoản của bạn.

### Giới hạn và khuyến nghị ⭐⭐

- **Chỉ được có 5 Elastic IP trong một tài khoản** (có thể xin AWS tăng thêm).
- **Nhìn chung: HÃY TRÁNH dùng Elastic IP**, vì:
  - Chúng **thường phản ánh quyết định kiến trúc kém** (poor architectural decisions).
- **Thay vào đó:**
  - Dùng **public IP ngẫu nhiên** và **đăng ký một DNS name** trỏ tới nó.
  - Hoặc (như sẽ học sau) **dùng Load Balancer và không dùng public IP** cho instance.

> **Ghi nhớ thi:** Câu hỏi "làm sao có IP cố định cho EC2?" → **Elastic IP**. Nhưng câu hỏi "kiến trúc tốt nhất là gì?" → **Load Balancer / Route 53 DNS**, không phải Elastic IP.

### Chi phí Elastic IP 💰

- **Miễn phí** khi được **gắn vào một instance đang chạy**.
- **BỊ TÍNH TIỀN** khi:
  - Không gắn vào instance nào (idle)
  - Gắn vào instance đã **stopped**
  - Gắn vào một ENI chưa được attach
- Từ **1/2/2024**, AWS tính phí cho **mọi public IPv4**, kể cả đang được dùng (~$0.005/giờ ≈ $3.6/tháng).

### Bảng tổng hợp 3 loại IP ⭐

| | **Private IP** | **Public IP** | **Elastic IP** |
|---|---|---|---|
| Phạm vi | Mạng nội bộ AWS | Toàn internet | Toàn internet |
| Duy nhất | Trong VPC | Toàn cầu | Toàn cầu |
| Đổi khi Stop/Start | ❌ **Không đổi** | ✅ **Có thể đổi** | ❌ **Không đổi** |
| Bạn sở hữu | Không | Không | ✅ **Có** (đến khi release) |
| SSH từ internet | ❌ Không được | ✅ Được | ✅ Được |
| Giới hạn | Theo dải VPC | — | **5 / account** (mặc định) |
| Chi phí | Miễn phí | Có phí (từ 2024) | Có phí, **đặc biệt khi không dùng** |

---

## 48. Private vs Public vs Elastic IP Hands On

### 1. Quan sát Private IP và Public IP

1. EC2 → **Instances** → chọn instance → tab **Details**.
2. Xem các trường:

| Trường | Ví dụ | Ghi chú |
|--------|-------|---------|
| **Public IPv4 address** | `35.180.242.162` | Truy cập từ internet |
| **Private IPv4 addresses** | `172.31.20.15` | Chỉ trong VPC |
| **Public IPv4 DNS** | `ec2-35-180-242-162.eu-west-3.compute.amazonaws.com` | Tên DNS công khai |
| **Private IP DNS name** | `ip-172-31-20-15.eu-west-3.compute.internal` | Tên DNS nội bộ |

3. SSH vào instance, chạy:
   ```bash
   hostname -f          # in ra private DNS name
   ip addr              # thấy interface eth0 mang PRIVATE IP, không thấy public IP
   curl ifconfig.me     # hỏi bên ngoài xem IP public của mình là gì
   ```

> **Nhận xét quan trọng:** Instance **không "biết"** public IP của chính nó — public IP được ánh xạ bởi **Internet Gateway thông qua NAT**. Bên trong OS chỉ thấy private IP.

### 2. Chứng minh Public IP thay đổi sau Stop/Start

1. Ghi lại Public IPv4 hiện tại (ví dụ `35.180.242.162`).
2. **Instance state** → **Stop instance** → đợi trạng thái `Stopped`.
3. Quan sát: **Public IPv4 biến mất**, **Private IPv4 vẫn còn**.
4. **Instance state** → **Start instance**.
5. Quan sát: **Public IPv4 là một địa chỉ HOÀN TOÀN MỚI** (ví dụ `13.36.114.87`), còn **Private IPv4 giữ nguyên**.
6. Lệnh SSH cũ giờ đã hỏng → phải cập nhật IP mới.

### 3. Tạo và gắn Elastic IP

1. EC2 → menu trái → **Network & Security** → **Elastic IPs**.
2. Bấm **Allocate Elastic IP address**:
   - **Network Border Group**: giữ mặc định (theo Region hiện tại)
   - **Public IPv4 address pool**: *Amazon's pool of IPv4 addresses*
3. Bấm **Allocate** → nhận được một Elastic IP (ví dụ `52.47.100.5`).
4. Chọn Elastic IP → **Actions** → **Associate Elastic IP address**:
   - **Resource type**: `Instance`
   - **Instance**: chọn instance của bạn
   - **Private IP address**: chọn private IP tương ứng
5. Bấm **Associate**.
6. Quay lại instance → **Public IPv4 address giờ chính là Elastic IP**.

### 4. Kiểm chứng Elastic IP không đổi

1. **Stop instance** → **Start instance**.
2. Quan sát: **Public IPv4 vẫn giữ nguyên** giá trị Elastic IP → mục tiêu đạt được.

### 5. Dọn dẹp ⚠️ (rất quan trọng)

Elastic IP **không gắn vào instance nào sẽ bị tính phí**:

1. Elastic IPs → chọn địa chỉ → **Actions** → **Disassociate Elastic IP address**.
2. Chọn lại → **Actions** → **Release Elastic IP addresses** → **Release**.

> ⚠️ Nếu bạn **terminate instance** mà **quên release Elastic IP**, AWS **vẫn tính tiền** địa chỉ đó cho đến khi bạn release.

### Checklist dọn dẹp

- [ ] Disassociate Elastic IP
- [ ] Release Elastic IP
- [ ] Terminate instance (nếu không dùng nữa)

---

## 49. EC2 Placement Groups

### Placement Group là gì?

- Đôi khi bạn **muốn kiểm soát chiến lược đặt (placement strategy) các EC2 instance**.
- Chiến lược đó được định nghĩa bằng **Placement Groups**.
- Khi tạo một placement group, bạn chỉ định **một trong các strategy** sau:

| Strategy | Mô tả (nguyên văn slide) |
|----------|--------------------------|
| **Cluster** | Gom (clusters) các instance vào một **nhóm độ trễ thấp trong MỘT Availability Zone** |
| **Spread** | **Trải (spreads) các instance trên phần cứng khác nhau** — **tối đa 7 instance / group / AZ** |
| **Partition** | **Trải các instance trên nhiều partition khác nhau** (dựa trên các bộ rack khác nhau) trong một AZ. Mở rộng tới **hàng trăm EC2 instance / group** (Hadoop, Cassandra, Kafka) |

- Placement Group **miễn phí**.

---

### 1️⃣ Cluster Placement Group

```
        ┌──────────── Same AZ ─────────────┐
        │  ┌─────┐ ┌─────┐ ┌─────┐         │
        │  │ EC2 │ │ EC2 │ │ EC2 │         │   Placement group: Cluster
        │  └─────┘ └─────┘ └─────┘         │   Low latency
        │  ┌─────┐ ┌─────┐ ┌─────┐         │   10 Gbps network
        │  │ EC2 │ │ EC2 │ │ EC2 │         │
        │  └─────┘ └─────┘ └─────┘         │
        └──────────────────────────────────┘
```

**Đặc điểm:** tất cả instance nằm trong **cùng một rack**, **cùng một AZ**.

| | Nội dung |
|---|---|
| **Pros (Ưu điểm)** | **Mạng tuyệt vời** — băng thông **10 Gbps** giữa các instance khi bật **Enhanced Networking** (khuyến nghị) |
| **Cons (Nhược điểm)** | **Nếu AZ hỏng, TẤT CẢ instance hỏng cùng lúc** |
| **Use case** | • **Big Data job cần hoàn thành nhanh**<br>• Ứng dụng cần **độ trễ cực thấp** và **thông lượng mạng cao** |

> **Rủi ro cao nhất, hiệu năng cao nhất.**

---

### 2️⃣ Spread Placement Group

```
   us-east-1a        us-east-1b        us-east-1c
   ┌─────────┐      ┌─────────┐      ┌─────────┐
   │  EC2    │      │  EC2    │      │  EC2    │
   │Hardware1│      │Hardware3│      │Hardware5│
   └─────────┘      └─────────┘      └─────────┘
   ┌─────────┐      ┌─────────┐      ┌─────────┐
   │  EC2    │      │  EC2    │      │  EC2    │
   │Hardware2│      │Hardware4│      │Hardware6│
   └─────────┘      └─────────┘      └─────────┘
```

**Đặc điểm:** mỗi instance nằm trên **một phần cứng vật lý riêng biệt**.

| | Nội dung |
|---|---|
| **Pros (Ưu điểm)** | • **Trải được qua nhiều Availability Zone (AZ)**<br>• **Giảm rủi ro lỗi đồng thời** (simultaneous failure)<br>• Các EC2 instance nằm trên **phần cứng vật lý khác nhau** |
| **Cons (Nhược điểm)** | **Giới hạn 7 instance / AZ / placement group** ⭐ |
| **Use case** | • Ứng dụng cần **tối đa hóa high availability**<br>• **Ứng dụng critical** nơi **mỗi instance phải được cô lập khỏi lỗi của instance khác** |

> **Rủi ro thấp nhất, nhưng số lượng instance bị giới hạn.**

---

### 3️⃣ Partition Placement Group

```
        us-east-1a                         us-east-1b
  ┌──────────┬──────────┐            ┌──────────┐
  │ EC2  EC2 │ EC2  EC2 │            │ EC2  EC2 │
  │ EC2  EC2 │ EC2  EC2 │            │ EC2  EC2 │
  ├──────────┼──────────┤            ├──────────┤
  │Partition1│Partition2│            │Partition3│
  └──────────┴──────────┘            └──────────┘
   (rack set A) (rack set B)          (rack set C)
```

**Đặc điểm (nguyên văn slide):**

- **Tối đa 7 partition / AZ** ⭐
- **Trải được qua nhiều AZ trong cùng một Region**
- **Tới hàng trăm (100s) EC2 instance**
- **Instance trong một partition KHÔNG chia sẻ rack với instance ở partition khác**
- **Một partition hỏng có thể ảnh hưởng nhiều EC2 nhưng KHÔNG ảnh hưởng các partition khác**
- **EC2 instance truy cập được thông tin partition dưới dạng METADATA** ⭐
- **Use cases**: **HDFS, HBase, Cassandra, Kafka** (các hệ phân tán "partition-aware")

Lấy thông tin partition từ bên trong instance:

```bash
curl http://169.254.169.254/latest/meta-data/placement/partition-number
```

> **Cân bằng giữa Cluster và Spread**: vừa mở rộng được số lượng lớn, vừa cô lập lỗi theo nhóm.

---

### Bảng so sánh 3 Placement Groups ⭐⭐

| | **Cluster** | **Spread** | **Partition** |
|---|---|---|---|
| **Vị trí đặt** | Cùng 1 rack, cùng 1 AZ | Mỗi instance 1 phần cứng riêng | Nhóm theo partition (bộ rack riêng) |
| **Trải qua nhiều AZ** | ❌ Không (1 AZ) | ✅ Có | ✅ Có (cùng Region) |
| **Giới hạn số lượng** | Theo capacity của AZ | **7 instance / AZ** | **7 partition / AZ**, **hàng trăm instance** |
| **Độ trễ mạng** | ⭐ **Thấp nhất (10 Gbps)** | Bình thường | Bình thường |
| **Chống lỗi** | ❌ Kém nhất | ⭐ **Tốt nhất** | ✅ Tốt (theo nhóm) |
| **Metadata partition** | ❌ | ❌ | ✅ **Có** |
| **Use case điển hình** | Big Data nhanh, HPC, low latency | Ứng dụng critical, HA tối đa | **HDFS, HBase, Cassandra, Kafka** |

### Mẹo nhớ ⭐

| Từ khóa trong đề | Đáp án |
|------------------|--------|
| "low latency", "high network throughput", "big data job hoàn thành nhanh" | **Cluster** |
| "maximize high availability", "critical application", "isolate each instance" | **Spread** |
| "**HDFS / HBase / Cassandra / Kafka**", "hàng trăm instance", "distributed & replicated" | **Partition** |
| "tối đa 7 instance mỗi AZ" | **Spread** |
| "tối đa 7 partition mỗi AZ" | **Partition** |

---

## 50. EC2 Placement Groups - Hands On

### Tạo Placement Group

1. EC2 → menu trái → **Network & Security** → **Placement Groups**.
2. Bấm **Create placement group**.
3. Điền:
   - **Name**: ví dụ `my-cluster-placement-group`
   - **Placement strategy**: chọn **Cluster** / **Spread** / **Partition**
   - Nếu chọn **Partition**: nhập **Number of partitions** (1–7)
   - Nếu chọn **Spread**: chọn **Spread level** = `Rack` hoặc `Host` (Host chỉ dùng cho Outposts)
4. Bấm **Create group**.

Tạo thử cả 3 loại để xem giao diện khác nhau:

| Tên ví dụ | Strategy | Ghi chú |
|-----------|----------|---------|
| `my-cluster-placement-group` | Cluster | Không có tùy chọn thêm |
| `my-spread-placement-group` | Spread | Chọn spread level |
| `my-partition-placement-group` | Partition | Nhập số partition (ví dụ 3) |

### Gán instance vào Placement Group

**Khi launch instance mới:**

1. *Launch instances* → **Advanced details** → cuộn tới **Placement group name**.
2. Chọn placement group đã tạo (hoặc *Create new placement group*).

**Với instance đã có:**

1. Instance phải ở trạng thái **Stopped**.
2. **Actions** → **Instance settings** → **Modify instance placement**.
3. Chọn placement group → **Save**.

> ⚠️ **Không thể** thay đổi placement group của một instance đang **running**.

### Lưu ý thực hành

- Placement Group **hoàn toàn miễn phí** — chỉ trả tiền cho instance bên trong.
- Với **Cluster**: nên **launch tất cả instance cùng lúc** để AWS tìm được capacity trên cùng rack; nếu launch dần từng cái có thể gặp lỗi `InsufficientInstanceCapacity`.
- Với **Cluster**: nên dùng **cùng một instance type** cho mọi instance trong group.
- **Dọn dẹp**: chọn placement group → **Delete** (phải xóa/terminate hết instance bên trong trước).

---

## 51. Elastic Network Interfaces (ENI) - Overview

### ENI là gì?

- Là một **thành phần logic (logical component) trong VPC**, đại diện cho một **card mạng ảo (virtual network card)**.

### ENI có thể có các thuộc tính sau ⭐

- **Primary private IPv4**, và **một hoặc nhiều secondary IPv4**
- **Một Elastic IP (IPv4) cho MỖI private IPv4**
- **Một Public IPv4**
- **Một hoặc nhiều security groups**
- **Một MAC address**

### Đặc điểm quan trọng ⭐⭐

- Bạn có thể **tạo ENI độc lập** và **gắn (attach) chúng "on the fly"** — **di chuyển** giữa các EC2 instance để phục vụ **failover**.
- ENI **bị ràng buộc vào một Availability Zone (AZ) cụ thể** — **không thể di chuyển ENI sang AZ khác**.

### Sơ đồ (từ slide)

```
┌──────────────── Availability Zone ─────────────────┐
│                                                     │
│   ┌─────┐   Eth0 – primary ENI                     │
│   │ EC2 │───  192.168.0.31                          │
│   │     │                                           │
│   │     │───  Eth1 – secondary ENI                  │
│   └─────┘     192.168.0.42  ──┐                     │
│                               │ Can be moved        │
│   ┌─────┐   Eth0 – primary ENI│                     │
│   │ EC2 │◄──────────────────────┘                   │
│   └─────┘                                           │
└─────────────────────────────────────────────────────┘
```

### Các loại ENI

| Loại | Mô tả |
|------|-------|
| **Primary ENI (`eth0`)** | Tạo tự động khi launch instance. **Không thể detach** khỏi instance |
| **Secondary ENI (`eth1`, `eth2`…)** | Tạo thủ công, **attach/detach tự do**, di chuyển được sang instance khác |

### Use cases điển hình ⭐

| Use case | Cách hoạt động |
|----------|----------------|
| **Failover / High availability** | Instance A hỏng → detach ENI khỏi A → attach vào instance B → **IP và MAC được giữ nguyên** → traffic tự động chuyển |
| **License gắn với MAC address** | Phần mềm cấp phép theo MAC → giữ ENI thì giữ được license |
| **Management network riêng** | `eth0` cho traffic ứng dụng, `eth1` cho traffic quản trị (security group khác nhau) |
| **Dual-homed instance** | Instance thuộc hai subnet khác nhau trong cùng AZ |
| **Low-budget HA** | Không cần Load Balancer, chỉ cần chuyển ENI khi instance chính hỏng |

### Giới hạn

- Số ENI tối đa và số IP mỗi ENI **phụ thuộc vào instance type** (ví dụ `t2.micro` = 2 ENI, mỗi ENI 2 IPv4).
- ENI **không được tính phí riêng**; nhưng **Elastic IP gắn vào ENI chưa attach thì bị tính phí**.

---

## 52. Elastic Network Interfaces (ENI) - Hands On

### 1. Xem ENI của instance hiện có

1. EC2 → menu trái → **Network & Security** → **Network Interfaces**.
2. Thấy ENI được tạo tự động cho mỗi instance đang chạy, với các cột:
   - **Network interface ID** (`eni-0a1b2c3d…`)
   - **Subnet ID**, **Availability Zone**
   - **Primary private IPv4 address**
   - **Security groups**
   - **MAC address**
   - **Status**: `in-use` / `available`
   - **Description**: `Primary network interface`

### 2. Tạo ENI độc lập

1. **Create network interface**:
   - **Description**: `my-eni`
   - **Subnet**: chọn subnet **trong AZ mà instance của bạn đang chạy** (ví dụ `eu-west-3a`)
   - **Private IPv4 address**: *Auto-assign* hoặc *Custom*
   - **Security groups**: chọn một SG
2. **Create network interface** → ENI mới có status **`available`**.

### 3. Attach ENI vào instance

1. Chọn ENI vừa tạo → **Actions** → **Attach**.
2. Chọn instance (chỉ hiện các instance **trong cùng AZ**) → **Attach**.
3. Status chuyển thành **`in-use`**.
4. Quay lại EC2 → Instances → tab **Networking** → thấy **2 network interface** (`eth0` và `eth1`).

### 4. Di chuyển ENI sang instance khác (mô phỏng failover)

1. Chọn ENI → **Actions** → **Detach** → xác nhận.
2. Status trở về **`available`**.
3. **Actions** → **Attach** → chọn **instance thứ hai** (cùng AZ).
4. → **Private IP và MAC address được giữ nguyên** dù đã đổi máy.

### 5. Chứng minh ràng buộc AZ

- Thử attach ENI (tạo ở `eu-west-3a`) vào một instance ở `eu-west-3b`
  → **instance đó KHÔNG xuất hiện trong danh sách** → ENI **bị khóa vào một AZ**.

### 6. Dọn dẹp

1. Chọn ENI → **Actions** → **Detach**.
2. Chọn ENI → **Actions** → **Delete**.

> **Lưu ý:** ENI được tạo tự động cùng instance (primary ENI, `eth0`) sẽ **tự bị xóa khi terminate instance** — đó là hành vi của thuộc tính **`Delete on termination = true`**. ENI bạn tự tạo có `Delete on termination = false` → phải xóa thủ công.

---

## 53. ENI - Extra Reading

> Đây là bài **article** (bài viết) — giảng viên cung cấp link đọc thêm về ENI.

### Tài liệu tham khảo chính thức

- **Elastic Network Interfaces (User Guide)**:
  `https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html`
- **Blog AWS về ENI**:
  `https://aws.amazon.com/blogs/aws/new-elastic-network-interfaces-in-the-virtual-private-cloud/`

### Các ý bổ sung đáng chú ý

#### Số ENI và IP tối đa theo instance type

| Instance type | Max ENI | IPv4 / ENI |
|---------------|---------|-----------|
| `t2.nano` | 2 | 2 |
| `t2.micro` | 2 | 2 |
| `t3.medium` | 3 | 6 |
| `m5.large` | 3 | 10 |
| `m5.24xlarge` | 15 | 50 |

→ Instance càng lớn, càng gắn được nhiều ENI và nhiều IP.

#### Thuộc tính `Source/Destination Check`

- **Mặc định `enabled`**: instance chỉ nhận/gửi traffic có địa chỉ đích/nguồn là chính nó.
- **Phải `disable`** nếu instance đóng vai trò **NAT instance**, **router**, hoặc **firewall ảo** (cần forward traffic hộ máy khác).

#### `Delete on termination`

| Loại ENI | Giá trị mặc định | Hệ quả |
|----------|-----------------|--------|
| Primary ENI (`eth0`) | `true` | Xóa cùng instance |
| ENI bạn tự tạo | `false` | **Tồn tại sau khi instance bị terminate** — phải xóa thủ công |

#### ENI được nhiều dịch vụ AWS dùng ngầm

Rất nhiều dịch vụ AWS tạo ENI trong VPC của bạn để hoạt động:

- **Lambda** trong VPC
- **RDS** instance
- **ELB / ALB / NLB**
- **VPC Endpoints (Interface Endpoint)** — bản chất là ENI
- **NAT Gateway**, **Transit Gateway**
- **ECS/EKS** với awsvpc network mode

→ Đó là lý do bạn thấy nhiều ENI lạ trong danh sách mà không tự tạo.

#### Các loại ENI đặc biệt

| Loại | Mô tả |
|------|-------|
| **ENI (thường)** | Card mạng ảo tiêu chuẩn |
| **ENA (Elastic Network Adapter)** | Enhanced Networking, băng thông tới 100 Gbps |
| **EFA (Elastic Fabric Adapter)** | Cho **HPC / Machine Learning**, hỗ trợ **MPI**, bypass OS kernel — dùng với **Cluster Placement Group** |

> **Ghi nhớ thi:** Đề nhắc **"HPC"**, **"MPI"**, **"tight-coupled workload"** → **EFA + Cluster Placement Group**.

---

## 54. EC2 Hibernate

### Ôn lại: Stop và Terminate

Chúng ta đã biết có thể **stop** và **terminate** instance:

- **Stop** — dữ liệu trên đĩa (**EBS**) **được giữ nguyên** cho lần start tiếp theo.
- **Terminate** — mọi **EBS volume (root)** được cấu hình **destroy** cũng **bị mất**.

### Khi start, điều gì xảy ra?

- **Lần start ĐẦU TIÊN**: **OS boot** & **EC2 User Data script chạy**.
- **Các lần start SAU**: chỉ **OS boot up**.
- Sau đó **ứng dụng khởi động**, **cache được làm nóng (warmed up)** — và việc đó **có thể mất nhiều thời gian!** ⭐

→ Đây chính là vấn đề mà **EC2 Hibernate** giải quyết.

### Giới thiệu EC2 Hibernate ⭐

- **Trạng thái in-memory (RAM) được BẢO TOÀN (preserved)**.
- **Instance boot NHANH HƠN NHIỀU** (OS **không bị stop/restart**).
- **Cơ chế bên dưới (under the hood)**: **trạng thái RAM được ghi vào một file trong root EBS volume**.
- **Root EBS volume BẮT BUỘC phải được MÃ HÓA (encrypted)** ⭐

### Sơ đồ vòng đời (từ slide)

```
  EC2 Instance          Hibernate         Hibernation      Start
    Running    ────────►  Stopping  ────────► Stopped  ────────► Running
     [RAM]                 [RAM]              [RAM đã lưu]        [RAM]
       │                     │                                      ▲
       │                     ▼                                      │
       │              Root EBS Volume ────────────────────────────────
       └───────────►    (Encrypted)         Shutdown
                       RAM state file
```

### Use cases ⭐

- **Long-running processing** (tiến trình chạy dài)
- **Saving the RAM state** (cần lưu trạng thái bộ nhớ)
- **Services that take time to initialize** (dịch vụ mất nhiều thời gian khởi tạo)

Ví dụ thực tế: máy chủ có cache lớn (in-memory cache) mất 20 phút để warm up; JVM/ứng dụng doanh nghiệp khởi động chậm; môi trường dev cần giữ nguyên trạng thái làm việc.

---

### EC2 Hibernate — Good to know (điều kiện ràng buộc) ⭐⭐

Đây là phần **rất hay ra thi**:

| Điều kiện | Yêu cầu |
|-----------|---------|
| **Supported Instance Families** | **C3, C4, C5, I3, M3, M4, R3, R4, T2, T3, …** |
| **Instance RAM Size** | **PHẢI NHỎ HƠN 150 GB** ⭐ |
| **Instance Size** | **KHÔNG hỗ trợ bare metal instances** |
| **AMI** | **Amazon Linux 2, Linux AMI, Ubuntu, RHEL, CentOS & Windows…** |
| **Root Volume** | **PHẢI là EBS**, **được MÃ HÓA**, **KHÔNG phải instance store**, và **đủ LỚN** ⭐ |
| **Purchasing options** | Khả dụng cho **On-Demand, Reserved và Spot Instances** |
| **Thời gian hibernate tối đa** | **Một instance KHÔNG THỂ hibernate quá 60 NGÀY** ⭐ |

> **3 con số phải nhớ:** RAM **< 150 GB**, tối đa **60 ngày**, root volume **phải encrypted**.

### So sánh Stop vs Hibernate vs Terminate ⭐

| | **Stop** | **Hibernate** | **Terminate** |
|---|---|---|---|
| Dữ liệu EBS | ✅ Giữ | ✅ Giữ | ❌ Mất (root volume) |
| **Trạng thái RAM** | ❌ **Mất** | ✅ **Được bảo toàn** | ❌ Mất |
| OS | Shutdown hoàn toàn | **Không stop/restart** | Bị hủy |
| Tốc độ khởi động lại | Chậm (boot + app + cache) | ⭐ **Nhanh** | — |
| Public IP | Thay đổi | Thay đổi | — |
| Private IP | Giữ nguyên | Giữ nguyên | — |
| User Data chạy lại | ❌ Không | ❌ Không | (instance mới → có) |
| Tính phí compute | ❌ Không | ❌ Không | ❌ Không |
| Tính phí EBS | ✅ Có | ✅ Có (**+ dung lượng lưu RAM**) | ❌ Không |

> ⚠️ Hibernate **vẫn tính phí EBS**, và root volume phải đủ lớn để chứa **toàn bộ RAM** → chi phí lưu trữ cao hơn Stop thường.

---

## 55. EC2 Hibernate - Hands On

### Launch instance có bật Hibernate

⚠️ **Hibernate CHỈ bật được lúc launch** — không thể bật cho instance đã tồn tại.

1. EC2 → *Launch instances*.
2. **Name**: ví dụ `hibernate-demo`.
3. **AMI**: **Amazon Linux 2** (hoặc Amazon Linux 2023).
4. **Instance type**: `t2.micro` hoặc `t3.micro`.
5. **Key pair**: chọn key có sẵn.
6. **Network settings**: cho phép **SSH (port 22)**.
7. **Configure storage** → mở **Advanced**:
   - **Size**: tăng lên **ít nhất 30 GiB** (phải đủ chứa RAM + OS)
   - **Encrypted**: chọn **Yes** ⭐ (bắt buộc)
   - **KMS key**: `aws/ebs` (default key)
8. **Advanced details** → cuộn tới **Stop - Hibernate behavior** → chọn **Enable** ⭐
9. **Launch instance**.

> Nếu quên tick **Encrypted**, tùy chọn Hibernate sẽ **bị vô hiệu hóa** hoặc launch sẽ báo lỗi.

### Kiểm chứng Hibernate hoạt động

1. SSH vào instance.
2. Ghi lại thời điểm boot:
   ```bash
   uptime -s          # thời điểm hệ thống khởi động
   uptime             # thời gian đã chạy
   ```
3. Tạo một tiến trình chiếm RAM để làm bằng chứng:
   ```bash
   # Ví dụ: mở một file và giữ nó trong bộ nhớ
   echo "trang thai truoc khi hibernate" > /tmp/test.txt
   ```
4. Quay lại Console → chọn instance → **Instance state** → **Hibernate instance**.
5. Đợi trạng thái chuyển sang **`Stopped`**, cột **State transition reason** ghi rõ:
   `Client.UserInitiatedHibernate: User initiated hibernate`
6. **Instance state** → **Start instance**.
7. SSH lại vào (⚠️ **Public IP đã đổi** — dùng IP mới).
8. Chạy lại:
   ```bash
   uptime -s          # ⭐ VẪN LÀ THỜI ĐIỂM BOOT CŨ → OS không hề restart!
   uptime             # thời gian chạy tính liên tục từ lần boot đầu
   ```

> **Đây chính là bằng chứng:** với **Stop thường**, `uptime -s` sẽ hiển thị thời điểm boot **mới**. Với **Hibernate**, nó giữ nguyên **thời điểm boot cũ** → chứng minh OS không bị restart, RAM được khôi phục.

### So sánh trực quan

| Hành động | `uptime -s` sau khi start | RAM |
|-----------|---------------------------|-----|
| **Stop → Start** | **Thời điểm mới** | Mất sạch |
| **Hibernate → Start** | **Thời điểm cũ** ⭐ | Được khôi phục |

### Dọn dẹp ⚠️

1. **Instance state** → **Terminate instance**.
2. Kiểm tra **Volumes** → đảm bảo EBS volume 30 GB đã bị xóa (nếu `Delete on termination = true`).
3. Nếu volume vẫn còn → chọn → **Actions** → **Delete volume** (30 GB vượt Free Tier nếu để lâu).

---

## Trắc nghiệm 3: EC2 SAA Level Quiz

### Các điểm dễ bị bẫy

| Câu hỏi thường gặp | Đáp án đúng |
|--------------------|-------------|
| Stop rồi Start instance — Public IP thế nào? | **Thay đổi** |
| Stop rồi Start instance — Private IP thế nào? | **Giữ nguyên** |
| Muốn EC2 có **public IP cố định**? | **Elastic IP** |
| Elastic IP tối đa bao nhiêu trong 1 account? | **5** (xin AWS tăng được) |
| Elastic IP gắn được vào mấy instance cùng lúc? | **Một** |
| Elastic IP có bị tính phí không? | **Có**, đặc biệt khi **không gắn vào instance đang chạy** |
| Kiến trúc **tốt hơn** Elastic IP là gì? | **DNS name (Route 53)** hoặc **Load Balancer** |
| Hai công ty khác nhau có thể dùng chung dải private IP? | **CÓ** |
| SSH vào EC2 bằng private IP được không? | **KHÔNG** — phải dùng public IP (trừ khi bạn ở trong VPC) |
| Placement group cho **độ trễ thấp, big data nhanh**? | **Cluster** |
| Placement group cho **HA tối đa, cô lập lỗi từng instance**? | **Spread** |
| Placement group cho **HDFS, HBase, Cassandra, Kafka**? | **Partition** |
| Spread group tối đa bao nhiêu instance mỗi AZ? | **7** |
| Partition group tối đa bao nhiêu partition mỗi AZ? | **7** |
| Placement group nào **chỉ nằm trong 1 AZ**? | **Cluster** |
| Nhược điểm lớn nhất của Cluster? | **AZ hỏng → tất cả instance hỏng cùng lúc** |
| Placement group nào cho EC2 **đọc được partition qua metadata**? | **Partition** |
| Placement group có tốn phí không? | **KHÔNG** — miễn phí |
| Đổi placement group của instance đang chạy được không? | **KHÔNG** — phải **stop** trước |
| ENI bị ràng buộc vào đâu? | **Một Availability Zone cụ thể** |
| ENI di chuyển được giữa các AZ không? | **KHÔNG** |
| ENI dùng để làm gì (use case chính)? | **Failover** — chuyển nhanh sang instance khác |
| ENI có những thuộc tính gì? | Primary + secondary IPv4, Elastic IP/private IP, 1 public IPv4, security groups, **MAC address** |
| ENI bạn tự tạo có tự xóa khi terminate instance? | **KHÔNG** — phải xóa thủ công |
| Hibernate bảo toàn cái gì? | **Trạng thái RAM (in-memory state)** |
| RAM được lưu ở đâu khi hibernate? | **File trong root EBS volume** |
| Root volume khi hibernate phải thế nào? | **Là EBS**, **được MÃ HÓA**, không phải instance store, **đủ lớn** |
| RAM tối đa để hibernate được? | **Nhỏ hơn 150 GB** |
| Hibernate tối đa bao nhiêu ngày? | **60 ngày** |
| Hibernate có hỗ trợ bare metal không? | **KHÔNG** |
| Hibernate dùng được với Spot Instance không? | **CÓ** (On-Demand, Reserved và Spot) |
| Bật Hibernate cho instance đã chạy được không? | **KHÔNG** — chỉ bật lúc launch |
| User Data có chạy lại sau khi Hibernate → Start? | **KHÔNG** |
| Use case của Hibernate? | Long-running processing, lưu RAM state, service khởi tạo lâu |

### Checklist tự kiểm tra trước khi làm quiz

- [ ] Phân biệt **Private / Public / Elastic IP** và hành vi khi Stop/Start
- [ ] Nhớ giới hạn **5 Elastic IP** và lời khuyên **tránh dùng Elastic IP**
- [ ] Thuộc **3 placement strategy** + use case + giới hạn **7**
- [ ] Nhớ **Cluster = 1 AZ**, **Spread & Partition = nhiều AZ**
- [ ] Nhớ **Partition ↔ HDFS/HBase/Cassandra/Kafka**
- [ ] Nắm **ENI khóa vào 1 AZ**, dùng cho **failover**
- [ ] Thuộc **3 con số Hibernate**: RAM **< 150 GB**, tối đa **60 ngày**, root volume **phải encrypted**
- [ ] Phân biệt **Stop vs Hibernate vs Terminate** (đặc biệt: RAM)

---

## Thuật ngữ Anh — Việt

| Tiếng Anh | Tiếng Việt |
|-----------|-----------|
| Public IP | IP công khai (định danh trên internet) |
| Private IP | IP nội bộ (chỉ trong mạng riêng) |
| Elastic IP | IP công khai cố định do bạn sở hữu |
| NAT (Network Address Translation) | Dịch địa chỉ mạng |
| Internet Gateway | Cổng ra internet của VPC |
| Geo-locate | Định vị địa lý |
| Remap | Ánh xạ lại (chuyển IP sang instance khác) |
| Allocate / Release (Elastic IP) | Cấp phát / Trả lại |
| Associate / Disassociate | Gắn / Gỡ liên kết |
| Placement Group | Nhóm đặt instance |
| Cluster strategy | Chiến lược gom cụm (cùng rack, cùng AZ) |
| Spread strategy | Chiến lược trải (mỗi instance một phần cứng) |
| Partition strategy | Chiến lược phân vùng (theo bộ rack) |
| Rack | Tủ rack chứa máy chủ trong data center |
| Enhanced Networking | Mạng tăng cường (băng thông cao) |
| Low latency | Độ trễ thấp |
| High availability (HA) | Tính sẵn sàng cao |
| Simultaneous failure | Lỗi đồng thời |
| Elastic Network Interface (ENI) | Card mạng ảo co giãn |
| Primary / Secondary ENI | ENI chính / ENI phụ |
| MAC address | Địa chỉ vật lý của card mạng |
| Attach / Detach | Gắn vào / Tháo ra |
| Failover | Chuyển đổi dự phòng khi sự cố |
| Source/Destination Check | Kiểm tra nguồn/đích của gói tin |
| Hibernate | Ngủ đông (bảo toàn RAM) |
| In-memory (RAM) state | Trạng thái bộ nhớ |
| Preserved | Được bảo toàn |
| Boot / Warm up cache | Khởi động / Làm nóng bộ đệm |
| Bare metal instance | Instance chạy trực tiếp trên phần cứng |
| Encrypted volume | Ổ đĩa được mã hóa |

---

*Ghi chú: các phần Hands On được tóm tắt lại các bước thao tác chính trên AWS Console. Giao diện Console có thể thay đổi theo thời gian — logic và khái niệm vẫn giữ nguyên. Nhớ RELEASE Elastic IP và TERMINATE instance sau khi thực hành để tránh phát sinh chi phí.*
