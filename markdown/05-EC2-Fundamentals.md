# Phần 5 — EC2 Fundamentals

> Khóa học: *Ultimate AWS Certified Solutions Architect Associate 2026* (Stéphane Maarek) — SAA-C03
> Nguồn tham chiếu: `AWS Certified Solutions Architect Slides v48.pdf` (phần "Amazon EC2 – Basics")
> Code kèm theo: `code_v2025-10-27/ec2-fundamentals/ec2-user-data.sh`

---

## Mục lục

| # | Bài giảng | Thời lượng | Loại |
|---|-----------|-----------|------|
| 31 | [AWS Budget Setup](#31-aws-budget-setup) | 5 phút | Video |
| 32 | [EC2 Basics](#32-ec2-basics) | 4 phút | Video |
| 33 | [Create an EC2 Instance with EC2 User Data to have a Website Hands On](#33-create-an-ec2-instance-with-ec2-user-data-to-have-a-website-hands-on) | 14 phút | Video |
| 34 | [EC2 Instance Types Basics](#34-ec2-instance-types-basics) | 6 phút | Video |
| 35 | [Security Groups & Classic Ports Overview](#35-security-groups--classic-ports-overview) | 7 phút | Video |
| 36 | [Security Groups Hands On](#36-security-groups-hands-on) | 5 phút | Video |
| 37 | [SSH Overview](#37-ssh-overview) | 3 phút | Video |
| 38 | [How to SSH using Linux or Mac](#38-how-to-ssh-using-linux-or-mac) | 7 phút | Video |
| 39 | [How to SSH using Windows](#39-how-to-ssh-using-windows) | 6 phút | Video |
| 40 | [How to SSH using Windows 10](#40-how-to-ssh-using-windows-10) | 5 phút | Video |
| 41 | [SSH Troubleshooting](#41-ssh-troubleshooting) | 1 phút | Bài viết |
| 42 | [EC2 Instance Connect](#42-ec2-instance-connect) | 3 phút | Video |
| 43 | [EC2 Instance Roles Demo](#43-ec2-instance-roles-demo) | 4 phút | Video |
| 44 | [EC2 Instance Purchasing Options](#44-ec2-instance-purchasing-options) | 10 phút | Video |
| 45 | [Spot Instances & Spot Fleet](#45-spot-instances--spot-fleet) | 9 phút | Video |
| 46 | [EC2 Instances Launch Types Hands On](#46-ec2-instances-launch-types-hands-on) | 9 phút | Video |
| — | [Trắc nghiệm 2: EC2 Fundamentals Quiz](#trắc-nghiệm-2-ec2-fundamentals-quiz) | — | Quiz |

---

## 31. AWS Budget Setup

Trước khi tạo tài nguyên tính phí (EC2), cần thiết lập **cảnh báo chi phí** để tránh hóa đơn bất ngờ.

### Bước 1 — Cho phép IAM user xem Billing

Mặc định **chỉ Root account** truy cập được Billing Console, kể cả IAM user có `AdministratorAccess`.

1. Đăng nhập bằng **Root account**.
2. Góc trên phải → **Account** → cuộn tới **IAM User and Role Access to Billing Information**.
3. Bấm **Edit** → tick **Activate IAM Access** → **Update**.
4. Từ giờ IAM user (có quyền billing) mới xem được Billing.

### Bước 2 — Tạo Budget

1. Vào **Billing and Cost Management** → **Budgets** → *Create budget*.
2. Chọn loại budget:

| Loại budget | Mục đích |
|-------------|----------|
| **Cost budget** | Cảnh báo khi chi phí ($) vượt ngưỡng |
| **Usage budget** | Cảnh báo khi mức sử dụng (giờ, GB…) vượt ngưỡng |
| **Savings Plans budget** | Theo dõi mức sử dụng Savings Plans |
| **Reservation budget** | Theo dõi mức sử dụng Reserved Instances |

3. Chọn **Cost budget** → *Next*.
4. **Budget amount**: chọn **Recurring budget** → **Monthly**, nhập số tiền (ví dụ `$10`).
5. **Configure alerts** → *Add alert threshold*:
   - **Actual cost** — cảnh báo khi chi phí **thực tế** đạt X% ngân sách.
   - **Forecasted cost** — cảnh báo khi chi phí **dự báo** sẽ đạt X% ngân sách.
   - Nhập ngưỡng (ví dụ `85%`) và **email nhận thông báo**.
6. Review → **Create budget**.

### Khuyến nghị của khóa học

- Tạo **2 alert**: một cho `Actual` (ví dụ 85%) và một cho `Forecasted` (ví dụ 100%).
- **AWS Free Tier**: 750 giờ `t2.micro`/tháng trong 12 tháng đầu — vẫn nên đặt budget đề phòng.
- Có sẵn template **"Zero spend budget"** — báo ngay khi chi phí vượt $0.01.

### Thói quen quan trọng ⚠️

> **Luôn TERMINATE các EC2 instance sau khi làm xong hands-on.** Instance đang chạy tính tiền theo giây, kể cả khi bạn không dùng.

---

## 32. EC2 Basics

### EC2 là gì?

- **EC2 = Elastic Compute Cloud = Infrastructure as a Service (IaaS)**.
- Là dịch vụ **phổ biến nhất** của AWS.
- Hiểu EC2 là nền tảng để hiểu cách Cloud vận hành.

### EC2 chủ yếu gồm các khả năng

| Thành phần | Ý nghĩa |
|------------|---------|
| **EC2** | Thuê **máy ảo** (virtual machines) |
| **EBS** | Lưu dữ liệu trên **ổ đĩa ảo** (virtual drives) |
| **ELB** | **Phân phối tải** giữa các máy (Elastic Load Balancer) |
| **ASG** | **Mở rộng/thu hẹp** dịch vụ tự động (Auto Scaling Group) |

### Các tùy chọn cấu hình & sizing của EC2

Khi tạo một EC2 instance, bạn chọn:

- **Operating System (OS)**: **Linux**, **Windows** hoặc **Mac OS**
- **CPU**: bao nhiêu compute power & cores
- **RAM**: bao nhiêu bộ nhớ truy cập ngẫu nhiên
- **Storage** (dung lượng lưu trữ):
  - **Network-attached** (gắn qua mạng): **EBS** & **EFS**
  - **Hardware** (gắn trực tiếp phần cứng): **EC2 Instance Store**
- **Network card**: tốc độ card mạng, **Public IP address**
- **Firewall rules**: **Security Group**
- **Bootstrap script** (cấu hình khi khởi động lần đầu): **EC2 User Data**

### EC2 User Data ⭐

- Cho phép **bootstrap** instance bằng một script.
- **Bootstrapping** = chạy các lệnh khi máy khởi động.
- Script này **CHỈ chạy MỘT LẦN**, ở **lần khởi động đầu tiên** của instance.
- Dùng để tự động hóa các tác vụ boot:
  - Cài đặt bản cập nhật (updates)
  - Cài đặt phần mềm
  - Tải file thường dùng từ internet
  - Bất cứ gì bạn nghĩ ra
- **EC2 User Data Script chạy với quyền `root`** → không cần `sudo` trong script.

> **Ghi nhớ thi:** User Data càng dài → thời gian boot càng lâu.

---

## 33. Create an EC2 Instance with EC2 User Data to have a Website Hands On

Mục tiêu: khởi chạy một EC2 instance chạy Linux, dùng User Data để tự động cài web server, rồi truy cập website qua trình duyệt.

### Script User Data của khóa học

Nội dung file `code_v2025-10-27/ec2-fundamentals/ec2-user-data.sh`:

```bash
#!/bin/bash
# Use this for your user data (script from top to bottom)
# install httpd (Linux 2 version)
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Hello World from $(hostname -f)</h1>" > /var/www/html/index.html
```

Giải thích từng dòng:

| Dòng | Ý nghĩa |
|------|---------|
| `#!/bin/bash` | Shebang — chỉ định script chạy bằng bash |
| `yum update -y` | Cập nhật toàn bộ package (`-y` = tự động đồng ý) |
| `yum install -y httpd` | Cài Apache HTTP Server |
| `systemctl start httpd` | Khởi động service ngay lập tức |
| `systemctl enable httpd` | Bật tự khởi động cùng máy (sau reboot vẫn chạy) |
| `echo ... > /var/www/html/index.html` | Tạo trang chủ; `$(hostname -f)` in ra hostname đầy đủ của instance |

### Các bước launch instance

1. Console → **EC2** → **Instances** → *Launch instances*.
2. **Name and tags**: đặt tên (ví dụ `My First Instance`).
3. **Application and OS Images (AMI)**: chọn **Amazon Linux 2023 AMI** (hoặc Amazon Linux 2) — có nhãn **Free tier eligible**.
   - **AMI = Amazon Machine Image** — bản mẫu (template) chứa OS + phần mềm cài sẵn.
4. **Instance type**: chọn **`t2.micro`** (Free tier eligible).
5. **Key pair (login)**:
   - *Create new key pair* → đặt tên (ví dụ `EC2 Tutorial`).
   - **Key pair type**: `RSA`
   - **Private key file format**:
     - **`.pem`** — cho Mac / Linux / Windows 10+ (OpenSSH)
     - **`.ppk`** — cho **PuTTY** trên Windows
   - File sẽ tự tải về — **giữ kỹ, chỉ tải được một lần**.
   - Hoặc chọn **Proceed without a key pair** nếu chỉ dùng EC2 Instance Connect.
6. **Network settings** → *Edit*:
   - Tạo security group mới (ví dụ `launch-wizard-1`).
   - Tick **Allow SSH traffic from** → `Anywhere (0.0.0.0/0)`
   - Tick **Allow HTTP traffic from the internet**
7. **Configure storage**: mặc định `8 GiB` `gp3` (Free tier tối đa 30 GB).
8. **Advanced details** → cuộn xuống cuối → ô **User data** → **dán script bash ở trên**.
9. Bấm **Launch instance**.

### Kiểm chứng kết quả

1. Đợi **Instance state = Running** và **Status check = 2/2 checks passed**.
2. Copy **Public IPv4 address** (ví dụ `35.180.x.x`).
3. Mở trình duyệt: `http://35.180.x.x` — **PHẢI dùng `http://`, KHÔNG dùng `https://`** (chưa cấu hình SSL).
4. Thấy dòng chữ: `Hello World from ip-172-31-x-x.eu-west-3.compute.internal`

### Stop / Start / Terminate

| Hành động | Kết quả | Tính phí |
|-----------|---------|----------|
| **Stop** | Máy tắt, dữ liệu trên **EBS được giữ nguyên**. **Public IP thay đổi** khi start lại. | Không tính phí compute, **vẫn tính phí EBS** |
| **Start** | Máy bật lại. **User Data KHÔNG chạy lại**. | Tính phí bình thường |
| **Reboot** | Khởi động lại, **giữ nguyên Public IP** | Tính phí bình thường |
| **Terminate** | **Xóa vĩnh viễn** instance, EBS root volume bị xóa theo (mặc định) | Ngừng tính phí |

### Lỗi thường gặp

| Triệu chứng | Nguyên nhân |
|-------------|-------------|
| Trang web **timeout** | Security Group chưa mở port **80** |
| Truy cập được nhưng **connection refused** | `httpd` chưa chạy — script User Data lỗi |
| Dùng `https://` không vào được | Chưa có SSL — phải dùng `http://` |
| Sửa User Data mà không thấy đổi | User Data **chỉ chạy lần đầu** — phải tạo instance mới |

---

## 34. EC2 Instance Types Basics

### Quy ước đặt tên của AWS ⭐

```
m5.2xlarge
│ │  └──── size: kích thước trong nhóm (nano, micro, small, medium,
│ │              large, xlarge, 2xlarge, 4xlarge, …)
│ └─────── generation: thế hệ (AWS cải tiến theo thời gian)
└───────── instance class: nhóm instance (m = general purpose)
```

- **`m`** — instance class (nhóm)
- **`5`** — generation (thế hệ)
- **`2xlarge`** — size trong nhóm đó

Tra cứu chính thức: https://aws.amazon.com/ec2/instance-types/

### Các nhóm instance chính

#### 1. General Purpose (Đa dụng)

- Tốt cho **đa dạng workload**: web server, code repository.
- **Cân bằng** giữa:
  - **Compute** (tính toán)
  - **Memory** (bộ nhớ)
  - **Networking** (mạng)
- Ví dụ họ: `t2`, `t3`, `m5`, `m6i`
- **Khóa học dùng `t2.micro`** — đây là instance General Purpose.

#### 2. Compute Optimized (Tối ưu tính toán)

Tốt cho tác vụ **cần bộ xử lý hiệu năng cao**:

- Batch processing workloads (xử lý theo lô)
- Media transcoding (chuyển mã media)
- High performance web servers
- **High performance computing (HPC)**
- Scientific modeling & machine learning
- Dedicated gaming servers

Ví dụ họ: `c5`, `c6g`, `c7i` (**c** = compute)

#### 3. Memory Optimized (Tối ưu bộ nhớ)

Hiệu năng nhanh cho workload **xử lý tập dữ liệu lớn trong bộ nhớ**:

- Cơ sở dữ liệu quan hệ / phi quan hệ hiệu năng cao
- Distributed web scale cache stores
- In-memory databases tối ưu cho **BI (business intelligence)**
- Ứng dụng xử lý **thời gian thực** dữ liệu lớn phi cấu trúc

Ví dụ họ: `r5`, `r6g`, `x1`, `z1d` (**r** = RAM)

#### 4. Storage Optimized (Tối ưu lưu trữ)

Tốt cho tác vụ **đọc/ghi tuần tự cường độ cao** trên tập dữ liệu lớn ở **local storage**:

- **OLTP** (online transaction processing) tần suất cao
- Relational & NoSQL databases
- Cache cho in-memory database (ví dụ **Redis**)
- Data warehousing applications
- Distributed file systems

Ví dụ họ: `i3`, `d2`, `h1` (**i** = IOPS, **d** = dense storage)

### Bảng ví dụ (từ slide)

| Instance | vCPU | Mem (GiB) | Storage | Network Performance | EBS Bandwidth (Mbps) |
|----------|------|-----------|---------|---------------------|----------------------|
| `t2.micro` | 1 | 1 | EBS-Only | Low to Moderate | — |
| `t2.xlarge` | 4 | 16 | EBS-Only | Moderate | — |
| `c5d.4xlarge` | 16 | 32 | 1 x 400 NVMe SSD | Up to 10 Gbps | 4,750 |
| `r5.16xlarge` | 64 | 512 | EBS Only | 20 Gbps | 13,600 |
| `m5.8xlarge` | 32 | 128 | EBS Only | 10 Gbps | 6,800 |

> **Website hữu ích để so sánh instance types:** https://instances.vantage.sh

### Mẹo nhớ chữ cái đầu

| Chữ | Nhóm | Ghi nhớ |
|-----|------|---------|
| **t**, **m** | General Purpose | **M**edium / balanced |
| **c** | Compute Optimized | **C**ompute / CPU |
| **r**, **x**, **z** | Memory Optimized | **R**AM |
| **i**, **d**, **h** | Storage Optimized | **I**OPS / **D**ense |
| **p**, **g**, **inf** | Accelerated Computing | **G**PU |

---

## 35. Security Groups & Classic Ports Overview

### Security Group là gì?

- **Nền tảng của bảo mật mạng trong AWS** (fundamental of network security).
- Kiểm soát cách **traffic được phép vào hoặc ra** khỏi EC2 instance.
- Security group **chỉ chứa các RULE ALLOW** (không có rule DENY).
- Rule có thể **tham chiếu theo IP** hoặc **theo Security Group khác**.

### Vai trò như "firewall"

Security Group hoạt động như **tường lửa** trên EC2 instance, điều tiết:

- **Access to Ports** (truy cập tới các port)
- **Authorised IP ranges** — cả **IPv4** và **IPv6**
- **Inbound network** (từ bên ngoài **vào** instance)
- **Outbound network** (từ instance **ra** ngoài)

### Sơ đồ hoạt động

```
                      ┌─────────────────────────┐
Máy bạn (IP X.X.X.X)  │  Security Group 1       │
   ── port 22 ───────►│  INBOUND                │──►  EC2 Instance
   (được cho phép)    │  Lọc IP/Port bằng Rules │      IP X.X.X.X
                      └─────────────────────────┘
Máy khác
   ── port 22 ───X    (không được cho phép → bị chặn)

                      ┌─────────────────────────┐
EC2 Instance ────────►│  Security Group 1       │──►  WWW
                      │  OUTBOUND               │     (Any IP – Any Port)
                      └─────────────────────────┘
```

### Những điều cần biết (Good to know) ⭐

- Có thể **gắn vào nhiều instance** cùng lúc; một instance có thể gắn **nhiều SG**.
- **Bị khóa vào một cặp Region + VPC** — đổi Region là phải tạo SG mới.
- **Nằm "bên ngoài" EC2** — nếu traffic bị chặn, **EC2 instance thậm chí không nhìn thấy** gói tin đó.
- Nên **duy trì một security group riêng cho SSH access**.
- **Mặc định: mọi inbound traffic bị CHẶN (blocked)**.
- **Mặc định: mọi outbound traffic được PHÉP (authorised)**.

### Chẩn đoán lỗi ⭐⭐ (rất hay ra thi)

| Triệu chứng | Nguyên nhân |
|-------------|-------------|
| Ứng dụng **không truy cập được (time out)** | → **Lỗi Security Group** (port chưa mở / IP không được phép) |
| Ứng dụng báo **"connection refused"** | → **Lỗi ứng dụng**, hoặc ứng dụng **chưa được khởi chạy** |

> Đây là một trong những cặp câu hỏi kinh điển nhất của kỳ thi. **Timeout = Security Group. Connection refused = Application.**

### Tham chiếu Security Group khác

Thay vì mở theo IP, bạn có thể cho phép traffic từ **các instance mang một Security Group cụ thể**:

```
Security Group 1 (INBOUND):
  ├── Authorising Security Group 1  → EC2 gắn SG1 ── port 123 ──► ✅ được phép
  └── Authorising Security Group 2  → EC2 gắn SG2 ── port 123 ──► ✅ được phép
                                       EC2 gắn SG3 ── port 123 ──► ❌ bị chặn
```

**Lợi ích:** không cần biết IP của các instance — rất hữu ích khi dùng Load Balancer + Auto Scaling (IP thay đổi liên tục).

### Classic Ports cần thuộc lòng ⭐

| Port | Giao thức | Mục đích |
|------|-----------|----------|
| **22** | **SSH** (Secure Shell) | Đăng nhập vào Linux instance |
| **21** | **FTP** (File Transfer Protocol) | Upload file lên file share |
| **22** | **SFTP** (Secure File Transfer Protocol) | Upload file **qua SSH** |
| **80** | **HTTP** | Truy cập website **không bảo mật** |
| **443** | **HTTPS** | Truy cập website **có bảo mật** |
| **3389** | **RDP** (Remote Desktop Protocol) | Đăng nhập vào **Windows** instance |

> Lưu ý: **22 dùng cho cả SSH và SFTP** (vì SFTP chạy trên nền SSH).

---

## 36. Security Groups Hands On

### Xem và sửa Security Group

1. EC2 → **Instances** → chọn instance → tab **Security**.
2. Thấy Security Group đang gắn → bấm vào tên SG.
3. Tab **Inbound rules** → *Edit inbound rules*.

### Cấu trúc một rule

| Trường | Ý nghĩa |
|--------|---------|
| **Type** | Loại (SSH, HTTP, HTTPS, Custom TCP, All traffic…) — chọn Type sẽ tự điền port |
| **Protocol** | TCP / UDP / ICMP |
| **Port range** | Cổng (22, 80, 443, hoặc dải như `8000-9000`) |
| **Source** | `Anywhere-IPv4 (0.0.0.0/0)`, `My IP`, `Custom`, hoặc **một Security Group khác** |
| **Description** | Ghi chú (khuyến khích điền) |

### Thí nghiệm minh họa lỗi timeout

1. Thêm rule: **Custom TCP**, port `4567`, source `0.0.0.0/0`.
2. SSH vào instance, chạy một web server tạm:
   ```bash
   sudo yum install -y nc
   while true; do echo -e "HTTP/1.1 200 OK\n\nHello" | nc -l 4567; done
   ```
3. Truy cập `http://<public-ip>:4567` → thấy `Hello`.
4. **Xóa rule port 4567** khỏi Security Group → refresh → trang **treo rồi timeout**.
5. → Chứng minh: **timeout = lỗi Security Group**.

### Gắn nhiều Security Group

1. EC2 → **Security Groups** → *Create security group* (ví dụ `my-first-security-group`).
2. Thêm inbound rule cho port `22` (SSH).
3. Quay lại instance → **Actions** → **Security** → **Change security groups**.
4. Thêm SG mới vào → **Save**.
5. Instance giờ có **2 SG** — quyền là **hợp (union)** của cả hai (rule là **allow**, cộng dồn lại).

### Best practice cho SSH

- Đặt Source của rule SSH là **`My IP`** thay vì `0.0.0.0/0` để giảm rủi ro.
- Tách một Security Group riêng chuyên cho SSH → dễ bật/tắt truy cập quản trị.

---

## 37. SSH Overview

**SSH** là một trong những kỹ năng quan trọng nhất trong IT: cho phép bạn **điều khiển một máy từ xa bằng command line**.

### Bảng tổng hợp phương thức kết nối (SSH Summary Table)

| Hệ điều hành của bạn | Công cụ dùng được |
|----------------------|-------------------|
| **Mac** | SSH ✅ / EC2 Instance Connect ✅ |
| **Linux** | SSH ✅ / EC2 Instance Connect ✅ |
| **Windows < 10** | PuTTY ✅ / EC2 Instance Connect ✅ |
| **Windows >= 10** | SSH ✅ / PuTTY ✅ / EC2 Instance Connect ✅ |

### Nên xem bài giảng nào?

- **Mac / Linux** → bài *SSH on Mac/Linux* (bài 38)
- **Windows**:
  - Bài *PuTTY* (bài 39)
  - Nếu là **Windows 10** → bài *SSH on Windows 10* (bài 40)
- **Tất cả** → bài *EC2 Instance Connect* (bài 42)

### Lời khuyên của giảng viên

- **Học viên gặp nhiều vấn đề nhất với SSH.**
- Nếu không hoạt động, làm theo thứ tự:
  1. **Xem lại bài giảng** — có thể bạn đã bỏ sót bước nào đó.
  2. **Đọc hướng dẫn troubleshooting** (bài 41).
  3. **Thử EC2 Instance Connect**.
- **Chỉ cần MỘT phương thức hoạt động** (SSH, PuTTY hoặc EC2 Instance Connect) là đủ.
- Nếu không cách nào chạy được thì **cũng không sao** — khóa học **không dùng SSH nhiều**.

---

## 38. How to SSH using Linux or Mac

### Cú pháp lệnh SSH

```bash
ssh -i <path-to-key.pem> ec2-user@<public-ip>
```

Ví dụ:

```bash
ssh -i EC2Tutorial.pem ec2-user@35.180.242.162
```

| Thành phần | Ý nghĩa |
|------------|---------|
| `-i` | Chỉ định **identity file** (private key `.pem`) |
| `ec2-user` | **Username mặc định của Amazon Linux** |
| `@35.180.242.162` | **Public IPv4** của instance |

### Username mặc định theo AMI

| AMI | Username |
|-----|----------|
| Amazon Linux / Amazon Linux 2 / 2023 | `ec2-user` |
| Ubuntu | `ubuntu` |
| Debian | `admin` |
| RHEL | `ec2-user` hoặc `root` |
| CentOS | `centos` hoặc `ec2-user` |
| SUSE | `ec2-user` hoặc `root` |

### Lỗi kinh điển: "Permissions 0644 are too open"

```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: UNPROTECTED PRIVATE KEY FILE!               @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0644 for 'EC2Tutorial.pem' are too open.
```

**Nguyên nhân:** SSH từ chối dùng private key mà người khác cũng đọc được.

**Cách sửa:**

```bash
chmod 0400 EC2Tutorial.pem
```

> `0400` = chỉ **chủ sở hữu** được **đọc**, không ai khác truy cập được.

### Lần kết nối đầu tiên

```
The authenticity of host '35.180.242.162' can't be established.
Are you sure you want to continue connecting (yes/no)?
```

Gõ **`yes`** → host được lưu vào `~/.ssh/known_hosts`.

### Sau khi đăng nhập thành công

Bạn thấy banner Amazon Linux và prompt:

```
[ec2-user@ip-172-31-XX-XX ~]$
```

Thử vài lệnh:

```bash
whoami                    # ec2-user
hostname -f               # ip-172-31-x-x.eu-west-3.compute.internal
ping google.com           # kiểm tra kết nối internet
sudo su                   # chuyển sang quyền root
exit                      # thoát khỏi phiên SSH
```

### Ghi chú

- **Public IP đổi mỗi lần stop/start** → phải sửa lệnh SSH.
- Không thể SSH bằng **private IP** vì bạn không ở cùng mạng nội bộ với instance.

---

## 39. How to SSH using Windows

Dành cho **Windows < 10** (hoặc ai thích dùng PuTTY).

### Bước 1 — Cài PuTTY

Tải bộ cài từ: https://www.putty.org — cài cả **PuTTY** và **PuTTYgen**.

### Bước 2 — Chuyển `.pem` sang `.ppk` bằng PuTTYgen

PuTTY **không đọc được file `.pem`**, cần chuyển sang định dạng `.ppk`:

1. Mở **PuTTYgen**.
2. **Load** → đổi bộ lọc file thành **All Files (*.*)** → chọn file `.pem`.
3. Thông báo "Successfully imported foreign key" → **OK**.
4. Bấm **Save private key** → cảnh báo passphrase → chọn **Yes**.
5. Đặt tên và lưu file `.ppk`.

> **Mẹo:** Khi tạo Key pair trong Console, có thể chọn thẳng định dạng **`.ppk`** để bỏ qua bước chuyển đổi này.

### Bước 3 — Kết nối bằng PuTTY

1. Mở **PuTTY**.
2. Mục **Session** → **Host Name**: `ec2-user@<public-ip>`
   (hoặc chỉ nhập public IP, PuTTY sẽ hỏi username sau)
3. **Port**: `22`, **Connection type**: `SSH`
4. Panel trái: **Connection** → **SSH** → **Auth** → **Credentials**
5. **Private key file for authentication** → **Browse** → chọn file `.ppk`
6. Quay lại **Session** → nhập tên vào **Saved Sessions** → **Save** (để lần sau dùng lại)
7. Bấm **Open**.
8. Cảnh báo bảo mật lần đầu → **Accept**.

### Lỗi thường gặp trên PuTTY

| Lỗi | Cách xử lý |
|-----|-----------|
| `Network error: Connection timed out` | Security Group chưa mở port 22, hoặc sai Public IP |
| `Server refused our key` | Sai file `.ppk`, hoặc sai username |
| `No supported authentication methods available` | Chưa nạp private key vào phần Auth |

---

## 40. How to SSH using Windows 10

**Windows 10 trở lên có sẵn OpenSSH client** — không cần PuTTY nữa.

### Kiểm tra OpenSSH đã có chưa

Mở **Command Prompt** hoặc **PowerShell**:

```bash
ssh
```

Nếu hiện trang usage của `ssh` → đã có sẵn. Nếu không:
**Settings** → **Apps** → **Optional features** → **Add a feature** → cài **OpenSSH Client**.

### Kết nối

```bash
cd Downloads
ssh -i EC2Tutorial.pem ec2-user@35.180.242.162
```

### Lỗi "Unprotected private key file" trên Windows

```
Permissions for 'EC2Tutorial.pem' are too open.
```

Windows không có `chmod`, phải sửa quyền qua giao diện:

1. Chuột phải file `.pem` → **Properties** → tab **Security** → **Advanced**.
2. Bấm **Disable inheritance** → chọn *Remove all inherited permissions from this object*.
3. Bấm **Add** → **Select a principal** → nhập **username Windows của bạn** → **Check Names** → **OK**.
4. Tick **Full control** → **OK** → **Apply** → **OK**.
5. Kết quả: **chỉ user của bạn** có quyền trên file này.
6. Chạy lại lệnh `ssh` → thành công.

> Nếu ở bước 3 không chắc username, mở Command Prompt gõ `whoami` để xem.

---

## 41. SSH Troubleshooting

> Đây là bài **article** (bài viết), tổng hợp các lỗi SSH phổ biến.

### Bảng chẩn đoán

| Lỗi / triệu chứng | Nguyên nhân | Cách khắc phục |
|-------------------|-------------|----------------|
| **`Connection timed out`** | **Security Group** chưa mở port 22, hoặc IP nguồn không được phép | Thêm inbound rule: **SSH, port 22, source `0.0.0.0/0`** hoặc `My IP` |
| **`Connection timed out`** (dù SG đúng) | Đang dùng **mạng công ty / VPN / firewall** chặn port 22 | Đổi mạng (dùng 4G/hotspot điện thoại), hoặc dùng **EC2 Instance Connect** |
| **`Connection refused`** | Instance mới boot chưa xong, hoặc sshd chưa chạy | Đợi status check `2/2`, hoặc **reboot** instance |
| **`Permission denied (publickey)`** | Sai **username** hoặc sai **key file** | Amazon Linux → `ec2-user`; Ubuntu → `ubuntu` |
| **`WARNING: UNPROTECTED PRIVATE KEY FILE!`** | Quyền file `.pem` quá mở | Mac/Linux: `chmod 0400 key.pem`; Windows: sửa quyền qua Properties → Security |
| **`No such file or directory`** | Sai đường dẫn tới file `.pem` | `cd` vào thư mục chứa key, hoặc dùng đường dẫn tuyệt đối |
| **`Host key verification failed`** | Public IP đã được dùng lại bởi instance khác | Xóa dòng tương ứng trong `~/.ssh/known_hosts` |
| **`Server refused our key`** (PuTTY) | File `.ppk` sai hoặc chưa convert đúng | Convert lại bằng PuTTYgen |

### Nguyên tắc vàng

1. Kiểm tra **Public IP** có đúng không (IP đổi sau mỗi lần stop/start).
2. Kiểm tra **Security Group** có mở port 22 không.
3. Kiểm tra **username** có đúng với AMI không.
4. Kiểm tra **quyền file `.pem`**.
5. Nếu vẫn không được → dùng **EC2 Instance Connect** (chạy trong trình duyệt, không cần key file).

---

## 42. EC2 Instance Connect

### Đặc điểm (nguyên văn slide)

- **Kết nối tới EC2 instance ngay trong trình duyệt** của bạn.
- **Không cần dùng key file** đã tải về.
- **"Phép màu"**: AWS **upload một key tạm thời (temporary key)** lên EC2 instance.
- **Chỉ hoạt động sẵn (out-of-the-box) với Amazon Linux 2** (và các bản Amazon Linux mới hơn).
- **Vẫn phải đảm bảo port 22 được mở!**

### Cách dùng

1. EC2 → **Instances** → chọn instance → bấm **Connect**.
2. Chọn tab **EC2 Instance Connect**.
3. **Username**: `ec2-user` (điền sẵn).
4. Bấm **Connect** → mở tab mới với terminal ngay trên trình duyệt.

### Cơ chế hoạt động

```
Console → AWS sinh cặp key SSH tạm thời (sống ~60 giây)
        → Push public key lên instance metadata
        → Trình duyệt SSH vào instance bằng private key tạm
        → Key tự động hết hạn
```

### Ưu / nhược điểm

| Ưu điểm | Nhược điểm |
|---------|-----------|
| Không cần quản lý file `.pem` | Chỉ hoạt động sẵn với Amazon Linux 2+ |
| Không lo lỗi permission | **Vẫn cần port 22 mở** trong Security Group |
| Chạy được từ mọi máy có trình duyệt | Instance phải có **Public IP** |
| Có thể kiểm soát bằng **IAM policy** (`ec2-instance-connect:SendSSHPublicKey`) | Không dùng được cho Ubuntu/Windows nếu chưa cài thêm |

> **Lưu ý quan trọng:** Nếu mạng của bạn chặn port 22 outbound thì EC2 Instance Connect vẫn **hoạt động** — vì kết nối đi qua hạ tầng AWS chứ không phải trực tiếp từ máy bạn.

---

## 43. EC2 Instance Roles Demo

Bài này áp dụng kiến thức **IAM Roles** (bài 25–26) vào thực tế với EC2.

### Chứng minh vấn đề

1. SSH / EC2 Instance Connect vào instance.
2. Chạy:
   ```bash
   aws iam list-users
   ```
3. Kết quả:
   ```
   Unable to locate credentials. You can configure credentials by running "aws configure".
   ```
   → Instance chưa có credentials.

### ❌ Cách SAI (tuyệt đối không làm)

```bash
aws configure
# nhập Access Key ID + Secret Access Key
```

**Vì sao sai:**

- Access key nằm **plaintext** trong `~/.aws/credentials` **ngay trên máy EC2**.
- Ai vào được instance (hoặc chiếm được qua lỗ hổng ứng dụng) đều **đọc được key**.
- Nếu tạo AMI từ instance này → **key bị nhân bản** ra mọi instance mới.
- Key **không tự rotate**.

> **Đây là câu hỏi bẫy kinh điển của kỳ thi.** Bất kỳ đáp án nào có "lưu access key trên EC2" đều SAI.

### ✅ Cách ĐÚNG — gắn IAM Role

1. EC2 → chọn instance → **Actions** → **Security** → **Modify IAM role**.
2. Chọn role đã tạo (ví dụ `DemoRoleForEC2` với policy `IAMReadOnlyAccess`).
3. **Update IAM role**.
4. Quay lại terminal của instance, chạy lại:
   ```bash
   aws iam list-users
   ```
   → **Thành công!** — không cần `aws configure`, credentials được cấp tự động.

### Kiểm chứng credentials tạm thời

Từ trong instance, truy vấn **Instance Metadata Service**:

```bash
# Lấy tên role đang gắn
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/

# Lấy credentials tạm thời của role đó
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/DemoRoleForEC2
```

Kết quả JSON chứa `AccessKeyId` (bắt đầu bằng **`ASIA`** — temporary), `SecretAccessKey`, **`Token`** và **`Expiration`**.

- `169.254.169.254` là **link-local address** — chỉ truy cập được **từ bên trong** instance.
- Credentials **tự động rotate** trước khi hết hạn.

### Thử giới hạn quyền

```bash
aws s3 ls
```
→ Bị từ chối (`AccessDenied`) vì role chỉ có `IAMReadOnlyAccess` → chứng minh **least privilege** hoạt động.

### Tóm tắt cần nhớ ⭐

| | Access Key trên EC2 | IAM Role |
|---|---|---|
| Bảo mật | ❌ Rất kém | ✅ Tốt |
| Credentials | Vĩnh viễn, plaintext | Tạm thời, tự rotate |
| Nhân bản qua AMI | ❌ Bị lộ | ✅ Không vấn đề |
| Thay đổi quyền | Phải sửa trên từng máy | Sửa policy → áp dụng ngay |
| **Khuyến nghị của AWS** | **KHÔNG BAO GIỜ** | **LUÔN LUÔN** |

---

## 44. EC2 Instance Purchasing Options

Có **7 tùy chọn mua** EC2 (nguyên văn slide):

| # | Tùy chọn | Mô tả ngắn |
|---|----------|-----------|
| 1 | **On-Demand Instances** | Workload ngắn, giá dễ đoán, trả theo giây |
| 2 | **Reserved Instances** (1 & 3 năm) | Workload dài hạn |
| 3 | **Convertible Reserved Instances** (1 & 3 năm) | Workload dài hạn, **linh hoạt đổi instance** |
| 4 | **Savings Plans** (1 & 3 năm) | **Cam kết một mức chi tiêu**, workload dài hạn |
| 5 | **Spot Instances** | Workload ngắn, **rẻ**, **có thể bị mất instance** (kém tin cậy) |
| 6 | **Dedicated Hosts** | Thuê **nguyên máy chủ vật lý**, kiểm soát vị trí đặt instance |
| 7 | **Dedicated Instances** | **Không khách hàng nào khác chia sẻ phần cứng** với bạn |
| 8 | **Capacity Reservations** | **Đặt trước capacity** trong một AZ cụ thể, thời hạn tùy ý |

---

### 1. EC2 On-Demand

- **Trả tiền theo mức dùng thực tế**:
  - **Linux hoặc Windows**: tính tiền **theo GIÂY**, sau phút đầu tiên
  - **Mọi HĐH khác**: tính tiền **theo GIỜ**
- **Chi phí cao nhất** nhưng **không phải trả trước** (no upfront payment).
- **Không cam kết dài hạn**.
- **Khuyến nghị**: workload **ngắn hạn**, **không bị gián đoạn**, khi **không đoán trước** được hành vi ứng dụng.

---

### 2. EC2 Reserved Instances (RI)

- Giảm giá **tới 72%** so với On-Demand.
- Bạn **đặt trước các thuộc tính cụ thể**: **Instance Type**, **Region**, **Tenancy**, **OS**.
- **Reservation Period**: **1 năm** (+ giảm giá) hoặc **3 năm** (+++ giảm giá).
- **Payment Options**:
  - **No Upfront** (+) — không trả trước
  - **Partial Upfront** (++) — trả trước một phần
  - **All Upfront** (+++) — trả trước toàn bộ (giảm nhiều nhất)
- **Reserved Instance's Scope**: **Regional** hoặc **Zonal** (Zonal = đặt trước capacity trong một AZ).
- **Khuyến nghị**: ứng dụng có mức dùng **ổn định (steady-state)** — ví dụ điển hình là **database**.
- Có thể **mua và bán lại** trên **Reserved Instance Marketplace**.

#### Convertible Reserved Instance

- **Có thể thay đổi**: EC2 instance type, instance family, OS, scope và tenancy.
- Giảm giá **tới 66%** (thấp hơn RI thường vì linh hoạt hơn).

> **Ghi chú từ slide:** con số % giảm giá thay đổi theo thời gian — **đề thi không yêu cầu nhớ con số chính xác**, chỉ cần nắm thứ tự tương đối.

---

### 3. EC2 Savings Plans

- Giảm giá theo mức dùng dài hạn — **tới 72%, ngang với RI**.
- **Cam kết một MỨC CHI TIÊU** (ví dụ **$10/giờ** trong 1 hoặc 3 năm) — không phải cam kết instance cụ thể.
- Phần dùng **vượt quá** Savings Plans được tính theo **giá On-Demand**.
- **Bị khóa vào**: một **instance family cụ thể** & một **AWS Region cụ thể** (ví dụ **M5 ở us-east-1**).
- **Linh hoạt (flexible) trên**:
  - **Instance Size** (ví dụ `m5.xlarge` → `m5.2xlarge`)
  - **OS** (Linux, Windows)
  - **Tenancy** (Host, Dedicated, Default)

> **RI vs Savings Plans:** RI khóa vào **instance type cụ thể**; Savings Plans khóa vào **family + region** nhưng linh hoạt về size/OS/tenancy → **dễ dùng hơn**.

---

### 4. EC2 Spot Instances

- Giảm giá **tới 90%** so với On-Demand — **rẻ nhất trong AWS**.
- Là instance mà bạn **có thể "mất" bất cứ lúc nào** nếu **max price của bạn < spot price hiện tại**.
- **Instance tiết kiệm chi phí NHẤT trong AWS**.
- **Phù hợp** cho workload **chịu được lỗi (resilient to failure)**:
  - **Batch jobs**
  - **Data analysis**
  - **Image processing**
  - Bất kỳ **distributed workload** nào
  - Workload có **thời gian bắt đầu/kết thúc linh hoạt**
- **KHÔNG phù hợp** cho **critical jobs** hoặc **databases**.

*(Chi tiết hơn ở bài 45)*

---

### 5. EC2 Dedicated Hosts

- Một **máy chủ vật lý** với capacity EC2 **dành riêng hoàn toàn cho bạn**.
- Cho phép đáp ứng **yêu cầu tuân thủ (compliance)** và dùng **license phần mềm gắn với server** (per-socket, per-core, per-VM licenses).
- **Purchasing Options**:
  - **On-Demand** — trả theo giây cho Dedicated Host đang hoạt động
  - **Reserved** — 1 hoặc 3 năm (No Upfront, Partial Upfront, All Upfront)
- **Tùy chọn ĐẮT NHẤT**.
- **Hữu ích cho**:
  - Phần mềm có **mô hình license phức tạp** — **BYOL (Bring Your Own License)**
  - Công ty có **yêu cầu quản lý/tuân thủ nghiêm ngặt**

---

### 6. EC2 Dedicated Instances

- Instance chạy trên **phần cứng dành riêng cho bạn**.
- **Có thể chia sẻ phần cứng** với các instance khác **trong cùng tài khoản**.
- **Không kiểm soát được vị trí đặt instance** (có thể bị chuyển sang phần cứng khác sau khi **Stop / Start**).

#### So sánh Dedicated Hosts vs Dedicated Instances ⭐

| | **Dedicated Hosts** | **Dedicated Instances** |
|---|---|---|
| Cấp độ | **Nguyên máy chủ vật lý** | **Instance** trên phần cứng riêng |
| Kiểm soát vị trí (placement) | ✅ **Có** | ❌ **Không** |
| Nhìn thấy socket/core vật lý | ✅ Có | ❌ Không |
| Chia sẻ HW với account khác | ❌ Không | ❌ Không |
| Chia sẻ HW với **cùng account** | ❌ Không | ✅ **Có thể** |
| BYOL license theo socket/core | ✅ Hỗ trợ | ❌ Không |
| Giá | **Đắt nhất** | Đắt (nhưng rẻ hơn Host) |

> **Mẹo thi:** Đề nhắc tới **"BYOL"**, **"per-socket / per-core license"**, hoặc **"kiểm soát instance placement"** → đáp án là **Dedicated Hosts**.

---

### 7. EC2 Capacity Reservations

- **Đặt trước capacity On-Demand** trong **một AZ cụ thể**, **thời hạn tùy ý**.
- **Luôn có sẵn capacity EC2** khi bạn cần.
- **Không cam kết thời gian** (tạo/hủy bất cứ lúc nào), **KHÔNG có giảm giá billing**.
- Có thể **kết hợp với Regional Reserved Instances và Savings Plans** để được hưởng giảm giá.
- **Bị tính tiền theo giá On-Demand dù bạn có chạy instance hay không**.
- **Phù hợp**: workload **ngắn hạn, không bị gián đoạn**, **bắt buộc phải nằm trong một AZ cụ thể**.

---

### Ví von "khách sạn / resort" (từ slide) 🏨

| Tùy chọn | Ví von |
|----------|--------|
| **On-Demand** | Đến và ở resort bất cứ khi nào thích, **trả giá đầy đủ** |
| **Reserved** | **Lên kế hoạch trước**, ở lâu dài → được **giảm giá tốt** |
| **Savings Plans** | **Trả một số tiền cố định mỗi giờ** trong một khoảng thời gian, được ở **bất kỳ loại phòng nào** (King, Suite, Sea View…) |
| **Spot Instances** | Khách sạn cho **đấu giá phòng trống**, người trả cao nhất được phòng. **Có thể bị đuổi ra bất cứ lúc nào** |
| **Dedicated Hosts** | **Thuê nguyên một tòa nhà** của resort |
| **Capacity Reservations** | **Đặt phòng trong một khoảng thời gian, trả full giá kể cả không đến ở** |

---

### Bảng so sánh giá (ví dụ `m4.large` – `us-east-1`, từ slide)

| Price Type | Price (per hour) |
|------------|------------------|
| **On-Demand** | **$0.10** |
| **Spot Instance** (Spot Price) | **$0.038 – $0.039** (giảm tới 61%) |
| **Reserved Instance (1 year)** | $0.062 (No Upfront) – $0.058 (All Upfront) |
| **Reserved Instance (3 years)** | **$0.043 (No Upfront) – $0.037 (All Upfront)** |
| **EC2 Savings Plan (1 year)** | $0.062 (No Upfront) – $0.058 (All Upfront) |
| **Reserved Convertible Instance (1 year)** | $0.071 (No Upfront) – $0.066 (All Upfront) |
| **Dedicated Host** | On-Demand Price |
| **Dedicated Host Reservation** | Giảm tới 70% |
| **Capacity Reservations** | **On-Demand Price** |

> Nhận xét: **Spot rẻ nhất** → **RI 3 năm All Upfront** → **RI 1 năm / Savings Plan** → **Convertible RI** → **On-Demand / Capacity Reservations** → **Dedicated Host** (đắt nhất).

---

### Cheat sheet chọn tùy chọn mua ⭐

| Tình huống trong đề thi | Đáp án |
|-------------------------|--------|
| Workload ngắn, không đoán trước được | **On-Demand** |
| Database chạy 24/7 suốt 3 năm | **Reserved Instances** |
| Cam kết chi tiêu, muốn linh hoạt đổi size/OS | **Savings Plans** |
| Batch job / phân tích dữ liệu, chịu được gián đoạn, **rẻ nhất** | **Spot Instances** |
| Cần **BYOL**, license theo socket/core, compliance nghiêm ngặt | **Dedicated Hosts** |
| Cần phần cứng riêng, không cần kiểm soát placement | **Dedicated Instances** |
| Đảm bảo **luôn có capacity** trong một AZ cụ thể | **Capacity Reservations** |

---

## 45. Spot Instances & Spot Fleet

### EC2 Spot Instance Requests

- Giảm giá **tới 90%** so với On-Demand.
- **Định nghĩa max spot price** — nhận được instance khi **spot price hiện tại < max price của bạn**.
- **Spot price theo giờ biến động** dựa trên **cung và cầu (offer and capacity)**.
- Nếu **spot price hiện tại > max price của bạn**, bạn có thể chọn **stop** hoặc **terminate** instance với **thời gian ân hạn 2 PHÚT (2 minutes grace period)** ⭐
- Dùng cho **batch jobs**, **data analysis**, hoặc workload **chịu được lỗi**.
- **Không tốt** cho **critical jobs** hoặc **databases**.

### Cách terminate Spot Instances ⭐⭐

Đây là điểm rất hay ra thi:

1. Bạn **chỉ hủy được Spot Instance request** ở trạng thái **`open`**, **`active`**, hoặc **`disabled`**.
2. **Hủy Spot Request KHÔNG terminate các instance** đang chạy.
3. **Đúng thứ tự:**
   - **Bước 1: HỦY (cancel) Spot Request trước**
   - **Bước 2: TERMINATE các Spot Instance liên quan sau**

> ⚠️ Nếu terminate instance trước mà không hủy request → **Spot Request sẽ tự động khởi chạy instance mới**! Đây chính là bẫy của câu hỏi.

Tài liệu: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-requests.html

### Vòng đời Spot Request

| Trạng thái | Ý nghĩa |
|-----------|---------|
| `open` | Request đã gửi, đang chờ được đáp ứng |
| `active` | Request đã được đáp ứng, instance đang chạy |
| `failed` | Request lỗi (sai cấu hình) |
| `closed` | Spot price vượt max price, hoặc capacity không còn |
| `cancelled` | Bạn đã hủy request |
| `disabled` | Bị tạm dừng (với persistent request) |

Loại request:

- **One-time**: đáp ứng một lần rồi kết thúc.
- **Persistent**: **tự động khởi chạy lại** instance khi bị interrupt → phải nhớ **cancel request** trước khi terminate.

---

### Spot Fleets ⭐

- **Spot Fleet = tập hợp các Spot Instances + (tùy chọn) On-Demand Instances**.
- Spot Fleet sẽ **cố gắng đạt target capacity** trong **ràng buộc về giá**.
- Định nghĩa các **launch pools** khả dĩ: **instance type** (`m5.large`), **OS**, **Availability Zone**.
  - Có thể có **nhiều launch pool** để fleet tự chọn.
- Spot Fleet **ngừng khởi chạy instance** khi **đạt capacity** hoặc **đạt max cost**.

### Các chiến lược phân bổ Spot Instances (Allocation Strategies) ⭐⭐

| Chiến lược | Cơ chế | Dùng khi |
|-----------|--------|----------|
| **`lowestPrice`** | Lấy từ **pool có giá thấp nhất** | **Tối ưu chi phí**, workload **ngắn** |
| **`diversified`** | **Phân bố đều trên tất cả các pool** | **Tối ưu độ khả dụng (availability)**, workload **dài** |
| **`capacityOptimized`** | Chọn **pool có capacity tối ưu** cho số instance cần | Giảm nguy cơ bị interrupt |
| **`priceCapacityOptimized`** ⭐ **(khuyến nghị)** | Chọn các pool có **capacity cao nhất**, sau đó chọn pool có **giá thấp nhất** trong số đó | **Lựa chọn tốt nhất cho hầu hết workload** |

> **Kết luận:** Spot Fleets cho phép **tự động request Spot Instances với giá thấp nhất**.

### Spot Instance vs Spot Fleet

| | **Spot Instance** | **Spot Fleet** |
|---|---|---|
| Số loại instance | Một loại cố định | **Nhiều launch pool** |
| Khi pool hết capacity | Instance bị mất, chờ | **Tự chuyển sang pool khác** |
| Kết hợp On-Demand | ❌ Không | ✅ Có (optional) |
| Tối ưu tự động | ❌ Không | ✅ Theo allocation strategy |

---

## 46. EC2 Instances Launch Types Hands On

### 1. Xem giá On-Demand

1. EC2 → **Instances** → *Launch instances*.
2. Chọn AMI + instance type → xem cột giá **On-Demand price** hiển thị ngay bên cạnh mỗi instance type.
3. Trang so sánh chính thức: https://aws.amazon.com/ec2/pricing/on-demand/

### 2. Xem Reserved Instances

1. EC2 → menu trái → **Instances** → **Reserved Instances**.
2. Bấm **Purchase Reserved Instances** → mở form tìm kiếm:
   - **Platform**: Linux/UNIX, Windows…
   - **Tenancy**: Default / Dedicated
   - **Instance type**: `t3.small`, `m5.large`…
   - **Term**: 1 year / 3 years
   - **Payment option**: No Upfront / Partial Upfront / All Upfront
   - **Offering class**: **Standard** / **Convertible**
3. Bấm **Search** → thấy bảng giá **Upfront price** và **Hourly price**.
4. ⚠️ **KHÔNG bấm mua** trong lúc học — đây là cam kết tài chính thật, không hủy được.

### 3. Xem Savings Plans

1. Console → tìm **Cost Management** → **Savings Plans**.
2. **Purchase Savings Plans** → chọn:
   - **Savings Plan type**: **Compute Savings Plans** (linh hoạt nhất) / **EC2 Instance Savings Plans** / **SageMaker Savings Plans**
   - **Term**: 1 năm / 3 năm
   - **Payment option**
   - **Hourly commitment** ($/giờ)
3. Xem **Recommendations** — AWS gợi ý mức cam kết dựa trên lịch sử dùng thật.
4. ⚠️ **Không mua** trong lúc học.

### 4. Spot Requests

1. EC2 → menu trái → **Spot Requests**.
2. Bấm **Pricing history** → xem biểu đồ **spot price biến động theo thời gian** cho từng instance type / AZ.
3. **Request Spot Instances**:
   - Chọn **Load balancing workloads** hoặc **Flexible workloads**
   - **Total target capacity**
   - **Set your maximum price** (hoặc để mặc định = On-Demand price)
   - Chọn **allocation strategy** (khuyến nghị **`priceCapacityOptimized`**)
4. Nếu có tạo thật, **nhớ đúng thứ tự dọn dẹp**:
   ```
   Bước 1: Cancel Spot Request
   Bước 2: Terminate Spot Instances
   ```

### 5. Dedicated Hosts

1. EC2 → menu trái → **Dedicated Hosts** → **Allocate Dedicated Host**.
2. Xem các tùy chọn: **Instance family**, **Availability Zone**, **Instance type support**.
3. ⚠️ **Rất đắt** — không allocate trong lúc học.

### 6. Capacity Reservations

1. EC2 → menu trái → **Capacity Reservations** → **Create Capacity Reservation**.
2. Chọn: instance type, platform, **Availability Zone**, số lượng, thời hạn.
3. ⚠️ **Tính tiền ngay từ lúc tạo** dù không chạy instance nào — không tạo trong lúc học.

### Checklist dọn dẹp sau hands-on ⚠️

- [ ] **Terminate** tất cả EC2 instance đang chạy
- [ ] **Cancel** mọi Spot Request (trước khi terminate instance)
- [ ] **Release** Elastic IP nếu có tạo (bị tính phí khi không gắn vào instance nào)
- [ ] **Cancel** Capacity Reservations nếu có
- [ ] Kiểm tra **Billing Dashboard** sau 24h

---

## Trắc nghiệm 2: EC2 Fundamentals Quiz

### Các điểm dễ bị bẫy

| Câu hỏi thường gặp | Đáp án đúng |
|--------------------|-------------|
| EC2 User Data chạy mấy lần? | **Một lần duy nhất**, ở lần boot đầu tiên |
| EC2 User Data chạy với quyền gì? | **Root** (không cần `sudo`) |
| Ứng dụng **timeout** khi truy cập | → **Lỗi Security Group** |
| Ứng dụng báo **connection refused** | → **Lỗi ứng dụng** / app chưa chạy |
| Mặc định inbound traffic? | **Bị chặn (blocked)** |
| Mặc định outbound traffic? | **Được phép (authorised)** |
| Security Group có rule DENY không? | **KHÔNG** — chỉ có rule ALLOW |
| Security Group gắn được vào bao nhiêu instance? | **Nhiều** — và một instance gắn được nhiều SG |
| Security Group bị khóa vào đâu? | **Region + VPC** |
| Port SSH / RDP / HTTP / HTTPS? | **22 / 3389 / 80 / 443** |
| Instance type nào cho **HPC, media transcoding**? | **Compute Optimized** (`c`) |
| Instance type nào cho **in-memory database, BI**? | **Memory Optimized** (`r`, `x`, `z`) |
| Instance type nào cho **OLTP tần suất cao, data warehouse**? | **Storage Optimized** (`i`, `d`, `h`) |
| Trong `m5.2xlarge`, `5` là gì? | **Generation** (thế hệ) |
| Ứng dụng trên EC2 cần gọi API AWS — cách an toàn nhất? | **Gắn IAM Role vào instance** |
| Có nên chạy `aws configure` trên EC2 không? | **KHÔNG BAO GIỜ** |
| Purchasing option **rẻ nhất**? | **Spot Instances** (giảm tới 90%) |
| Purchasing option **đắt nhất**? | **Dedicated Hosts** |
| Cần **BYOL** / license theo socket-core? | **Dedicated Hosts** |
| Database chạy ổn định 3 năm? | **Reserved Instances** |
| Muốn giảm giá nhưng linh hoạt về instance size & OS? | **Savings Plans** |
| Cần đảm bảo **luôn có capacity** trong một AZ? | **Capacity Reservations** |
| Capacity Reservations có giảm giá không? | **KHÔNG** — trả giá On-Demand |
| Spot Instance bị lấy lại — có bao nhiêu thời gian? | **2 phút grace period** |
| Muốn terminate Spot Instance — làm gì trước? | **Cancel Spot Request TRƯỚC**, rồi mới terminate |
| Allocation strategy nào được **khuyến nghị**? | **`priceCapacityOptimized`** |
| Allocation strategy nào cho **availability cao nhất**? | **`diversified`** |
| Allocation strategy nào **rẻ nhất**? | **`lowestPrice`** |
| Dedicated Hosts vs Dedicated Instances — khác gì? | **Hosts kiểm soát được placement**, Instances thì không |
| EC2 Instance Connect có cần mở port 22 không? | **CÓ** — vẫn phải mở |
| EC2 Instance Connect hoạt động sẵn với AMI nào? | **Amazon Linux 2** (và mới hơn) |
| Stop rồi Start instance — Public IP thế nào? | **Có thể thay đổi** |
| Stop instance — dữ liệu EBS ra sao? | **Được giữ nguyên** |

### Checklist tự kiểm tra trước khi làm quiz

- [ ] Thuộc **6 classic ports** (21, 22, 80, 443, 3389)
- [ ] Phân biệt **timeout** vs **connection refused**
- [ ] Nhớ **4 nhóm instance type** và use case
- [ ] Đọc được **quy ước đặt tên** `m5.2xlarge`
- [ ] Thuộc **7 purchasing options** và tình huống dùng
- [ ] Nhớ **thứ tự cancel Spot Request → terminate instance**
- [ ] Nhớ **4 allocation strategies** của Spot Fleet
- [ ] Nắm chắc: **EC2 gọi AWS API → dùng IAM Role, không dùng access key**

---

## Thuật ngữ Anh — Việt

| Tiếng Anh | Tiếng Việt |
|-----------|-----------|
| Elastic Compute Cloud (EC2) | Điện toán đám mây co giãn |
| Infrastructure as a Service (IaaS) | Hạ tầng dưới dạng dịch vụ |
| Instance | Máy ảo / thực thể |
| AMI (Amazon Machine Image) | Ảnh máy — bản mẫu OS + phần mềm |
| Bootstrap / User Data | Kịch bản khởi tạo khi máy boot lần đầu |
| Security Group | Nhóm bảo mật (tường lửa cho EC2) |
| Inbound / Outbound | Traffic vào / ra |
| Key pair | Cặp khóa (public/private) |
| Public IP / Private IP | IP công khai / IP nội bộ |
| Instance type / family / generation / size | Loại / nhóm / thế hệ / kích thước instance |
| General Purpose | Đa dụng |
| Compute Optimized | Tối ưu tính toán |
| Memory Optimized | Tối ưu bộ nhớ |
| Storage Optimized | Tối ưu lưu trữ |
| On-Demand | Trả theo nhu cầu |
| Reserved Instance | Instance đặt trước |
| Savings Plans | Gói tiết kiệm (cam kết chi tiêu) |
| Spot Instance | Instance giá đấu (có thể bị thu hồi) |
| Spot Fleet | Đội Spot Instance |
| Dedicated Host | Máy chủ vật lý riêng |
| Dedicated Instance | Instance trên phần cứng riêng |
| Capacity Reservation | Đặt trước năng lực tính toán |
| Tenancy | Chế độ chiếm dụng phần cứng |
| BYOL (Bring Your Own License) | Mang license của bạn sang dùng |
| Grace period | Thời gian ân hạn |
| Allocation strategy | Chiến lược phân bổ |
| Terminate / Stop / Reboot | Hủy vĩnh viễn / Dừng / Khởi động lại |

---

*Ghi chú: các phần Hands On được tóm tắt lại các bước thao tác chính trên AWS Console. Giao diện Console có thể thay đổi theo thời gian — logic và khái niệm vẫn giữ nguyên. Luôn TERMINATE instance sau khi thực hành xong để tránh phát sinh chi phí.*
