# Phần 10 — Amazon Route 53

> Khóa học: *Ultimate AWS Certified Solutions Architect Associate 2026* (Stéphane Maarek) — SAA-C03
> Nguồn tham chiếu: `AWS Certified Solutions Architect Slides v48.pdf` (phần "Amazon Route 53")
> Code kèm theo: `code_v2025-10-27/route53/user-data.sh`

---

## Mục lục

| # | Bài giảng | Thời lượng | Loại |
|---|-----------|-----------|------|
| 102 | [What is a DNS?](#102-what-is-a-dns) | 6 phút | Video |
| 103 | [Route 53 Overview](#103-route-53-overview) | 6 phút | Video |
| 104 | [Route 53 - Registering a domain](#104-route-53---registering-a-domain) | 3 phút | Video |
| 105 | [Route 53 - Creating our first records](#105-route-53---creating-our-first-records) | 4 phút | Video |
| 106 | [Route 53 - EC2 Setup](#106-route-53---ec2-setup) | 6 phút | Video |
| 107 | [Route 53 - TTL](#107-route-53---ttl) | 5 phút | Video |
| 108 | [Route 53 CNAME vs Alias](#108-route-53-cname-vs-alias) | 7 phút | Video |
| 109 | [Routing Policy - Simple](#109-routing-policy---simple) | 4 phút | Video |
| 110 | [Routing Policy - Weighted](#110-routing-policy---weighted) | 5 phút | Video |
| 111 | [Routing Policy - Latency](#111-routing-policy---latency) | 5 phút | Video |
| 112 | [Route 53 - Health Checks](#112-route-53---health-checks) | 5 phút | Video |
| 113 | [Route 53 - Health Checks Hands On](#113-route-53---health-checks-hands-on) | 5 phút | Video |
| 114 | [Routing Policy - Failover](#114-routing-policy---failover) | 4 phút | Video |
| 115 | [Routing Policy - Geolocation](#115-routing-policy---geolocation) | 4 phút | Video |
| 116 | [Routing Policy - Geoproximity](#116-routing-policy---geoproximity) | 3 phút | Video |
| 117 | [Routing Policy - IP-based](#117-routing-policy---ip-based) | 2 phút | Video |
| 118 | [Routing Policy - Multi Value](#118-routing-policy---multi-value) | 4 phút | Video |
| 119 | [3rd Party Domains & Route 53](#119-3rd-party-domains--route-53) | 2 phút | Video |
| 120 | [Route 53 Resolvers & Hybrid DNS](#120-route-53-resolvers--hybrid-dns) | 3 phút | Video |
| 121 | [Route 53 - Section Cleanup](#121-route-53---section-cleanup) | 1 phút | Video |
| — | [Trắc nghiệm 7: Route 53 Quiz](#trắc-nghiệm-7-route-53-quiz) | — | Quiz |

---

## 102. What is a DNS?

### DNS là gì?

- **DNS = Domain Name System** — **dịch hostname thân thiện với con người thành địa chỉ IP của máy** ⭐
  - `www.google.com` => `172.217.18.36`
- ⭐ **DNS là XƯƠNG SỐNG của Internet**
- ⭐ **DNS dùng cấu trúc đặt tên PHÂN CẤP (hierarchical naming structure)**

```
   .              ← Root
   └── .com       ← TLD
       └── example.com          ← SLD
           ├── www.example.com   ← Sub Domain
           └── api.example.com
```

---

### DNS Terminologies ⭐⭐

| Thuật ngữ | Định nghĩa |
|-----------|-----------|
| **Domain Registrar** | Nơi đăng ký tên miền — **Amazon Route 53, GoDaddy**, … |
| **DNS Records** | **A, AAAA, CNAME, NS**, … |
| **Zone File** | ⭐ **Chứa các DNS record** |
| **Name Server** | ⭐ **Phân giải các truy vấn DNS** (Authoritative hoặc Non-Authoritative) |
| **Top Level Domain (TLD)** | **`.com`, `.us`, `.in`, `.gov`, `.org`**, … |
| **Second Level Domain (SLD)** | **`amazon.com`, `google.com`**, … |

### Phân rã một URL đầy đủ ⭐

```
   http:// api . www . example . com .
   └─┬──┘  └┬┘  └┬┘  └──┬───┘ └┬┘ └┬┘
  Protocol  │  Sub Domain  SLD  TLD Root
            └──────────────────────────┘
              FQDN (Fully Qualified Domain Name)
```

> ⭐ **FQDN = Fully Qualified Domain Name** — tên miền đầy đủ, bao gồm cả dấu chấm root ở cuối.

---

### How DNS Works ⭐⭐ (quy trình phân giải)

```
  Web Browser                Local DNS Server        Root DNS Server
  (muốn vào example.com)     (do công ty bạn hoặc    (Managed by ICANN)
        │                     ISP cấp phát)                 │
        │ ① example.com? ──────►│                           │
        │                       │ ② example.com? ──────────►│
        │                       │◄── .com NS 1.2.3.4 ───────│
        │                       │
        │                       │ ③ example.com? ──────────► TLD DNS Server (.com)
        │                       │                            (Managed by IANA
        │                       │◄── example.com NS 5.6.7.8   — nhánh của ICANN)
        │                       │
        │                       │ ④ example.com? ──────────► SLD DNS Server
        │                       │                            (example.com)
        │                       │◄── example.com IP           (Managed by Domain
        │                       │    9.10.11.12                Registrar, ví dụ
        │◄── 9.10.11.12 + TTL ──│                              Amazon Registrar, Inc.)
        │
        └──────────► Web Server (example.com) IP: 9.10.11.12
```

### Ai quản lý cái gì ⭐

| Thành phần | Được quản lý bởi |
|------------|------------------|
| **Root DNS Server** | ⭐ **ICANN** |
| **TLD DNS Server (.com)** | ⭐ **IANA** (nhánh của ICANN) |
| **SLD DNS Server (example.com)** | ⭐ **Domain Registrar** (ví dụ Amazon Registrar, Inc.) |
| **Local DNS Server** | **Công ty bạn** hoặc **ISP cấp phát động** |

> ⭐ Kết quả được **cache lại theo TTL** ở Local DNS Server và ở trình duyệt.

---

## 103. Route 53 Overview

### Amazon Route 53 là gì? ⭐⭐

- ⭐ **DNS có tính sẵn sàng cao, có khả năng mở rộng, được quản lý hoàn toàn, và AUTHORITATIVE**
  - ⭐ **Authoritative = KHÁCH HÀNG (bạn) có thể CẬP NHẬT các DNS record**
- ⭐ **Route 53 cũng là một Domain Registrar** (có thể mua tên miền tại đây)
- ⭐ **Có khả năng kiểm tra sức khỏe (health check) các tài nguyên của bạn**
- ⭐⭐ **Là dịch vụ AWS DUY NHẤT cung cấp SLA sẵn sàng 100%**
- ⭐ **Vì sao tên là "Route 53"? — 53 là tham chiếu tới PORT DNS truyền thống**

```
   Client ──example.com?──► Amazon Route 53
          ◄── 54.22.33.44 ──┘
          │
          └──────────► AWS Cloud: EC2 Instance (Public IP 54.22.33.44)
```

---

### Route 53 — Records ⭐⭐

**Record định nghĩa cách bạn muốn định tuyến traffic cho một domain.**

**Mỗi record chứa:**

| Thành phần | Ví dụ |
|------------|-------|
| **Domain/subdomain Name** | `example.com` |
| **Record Type** | `A` hoặc `AAAA` |
| **Value** | `12.34.56.78` |
| **Routing Policy** | ⭐ **Cách Route 53 phản hồi các truy vấn** |
| **TTL** | ⭐ **Khoảng thời gian record được cache tại các DNS Resolver** |

### Các loại DNS record Route 53 hỗ trợ ⭐

| Nhóm | Các loại |
|------|----------|
| ⭐ **(BẮT BUỘC PHẢI BIẾT)** | **A / AAAA / CNAME / NS** |
| (nâng cao) | CAA / DS / MX / NAPTR / PTR / SOA / TXT / SPF / SRV |

---

### Route 53 — Record Types ⭐⭐

| Type | Chức năng |
|------|-----------|
| **A** | ⭐ **Ánh xạ một hostname tới IPv4** |
| **AAAA** | ⭐ **Ánh xạ một hostname tới IPv6** |
| **CNAME** | ⭐ **Ánh xạ một hostname tới một hostname KHÁC**<br>• **Target phải là một domain name CÓ record A hoặc AAAA**<br>• ⭐⭐ **KHÔNG THỂ tạo CNAME cho node cao nhất của DNS namespace (Zone Apex)**<br>• Ví dụ: **không tạo được cho `example.com`**, nhưng **tạo được cho `www.example.com`** |
| **NS** | ⭐ **Name Servers cho Hosted Zone** — **kiểm soát cách traffic được định tuyến cho domain** |

---

### Route 53 — Hosted Zones ⭐⭐

- ⭐ **Là một "container" chứa các record định nghĩa cách định tuyến traffic tới một domain và các subdomain của nó**

| Loại | Mô tả |
|------|-------|
| **Public Hosted Zones** ⭐ | **Chứa các record chỉ định cách định tuyến traffic TRÊN INTERNET** (public domain names)<br>Ví dụ: `application1.mypublicdomain.com` |
| **Private Hosted Zones** ⭐ | **Chứa các record chỉ định cách định tuyến traffic BÊN TRONG MỘT HOẶC NHIỀU VPC** (private domain names)<br>Ví dụ: `application1.company.internal` |

### 💰 **Bạn trả $0.50/tháng cho MỖI hosted zone** ⭐

### Sơ đồ Public vs Private Hosted Zone

```
        Public Hosted Zone                      Private Hosted Zone
   Client ──example.com?──► 54.22.33.44    ┌──────── VPC ─────────┐
        │                                   │ db.example.internal? │
        ├──► S3 Bucket                      │                      │
        ├──► CloudFront                     │  EC2 (webapp.example.internal)
        ├──► Application Load Balancer      │  EC2 (api.example.internal)
        └──► EC2 Instance (Public IP)       │  DB  (db.example.internal)
                                             │      → Private IP 10.0.0.35
                                             └──────────────────────┘
```

> **Bẫy thi:** Tên miền nội bộ (`.internal`, `.local`) chỉ phân giải được **từ trong VPC** đã được liên kết với Private Hosted Zone.

---

## 104. Route 53 - Registering a domain

### Đăng ký tên miền qua Route 53

1. Console → **Route 53** → menu trái → **Registered domains** → **Register domains**.
2. **Search for domain**: nhập tên miền muốn mua (ví dụ `stephanetheteacher.com`).
   - Xem giá theo từng TLD (`.com` ~ $14/năm, `.link` ~ $5/năm — rẻ nhất để học).
3. **Add to cart** → **Choose**.
4. **Contact information**: điền thông tin liên hệ (Registrant / Administrative / Technical).
5. ⭐ **Privacy protection**: bật để **ẩn thông tin cá nhân khỏi WHOIS công khai** (miễn phí với Route 53).
6. **Auto-renew**: bật/tắt tự động gia hạn.
7. Tick đồng ý điều khoản → **Submit**.
8. ⏳ **Chờ xác nhận qua email** → trạng thái `Pending` → `Successful` (thường **10 phút – vài giờ**).

### Kết quả

⭐ **Route 53 TỰ ĐỘNG tạo một Public Hosted Zone** cho tên miền vừa mua, kèm sẵn:
- **Bản ghi `NS`** (4 name server của AWS)
- **Bản ghi `SOA`** (Start of Authority)

### Lưu ý chi phí ⚠️

| Khoản | Chi phí |
|-------|---------|
| **Đăng ký domain** | **~$3–15/năm** tùy TLD (⚠️ **không hoàn tiền**) |
| **Hosted Zone** | ⭐ **$0.50/tháng** |
| **DNS queries** | ~$0.40 / 1 triệu query đầu tiên |
| **Health Check** | ~$0.50/tháng mỗi health check |

> ⚠️ Đăng ký domain **không nằm trong Free Tier**. Nếu chỉ học lý thuyết, bạn có thể bỏ qua bài này và các bài hands-on phụ thuộc vào nó.

---

## 105. Route 53 - Creating our first records

### Tạo record đầu tiên

1. Route 53 → **Hosted zones** → chọn hosted zone của bạn.
2. **Create record**.
3. **Record name**: `test` (sẽ thành `test.example.com`) — để trống nếu muốn dùng **root domain**.
4. **Record type**: `A – Routes traffic to an IPv4 address and some AWS resources`.
5. **Value**: nhập một IP, ví dụ `11.22.33.44`.
6. **TTL (seconds)**: `300`.
7. **Routing policy**: `Simple routing`.
8. **Create records**.

### Kiểm chứng bằng dòng lệnh ⭐

```bash
# Cách 1 — nslookup
nslookup test.example.com

# Cách 2 — dig (chi tiết hơn, thấy cả TTL)
dig test.example.com
```

Kết quả `dig` sẽ hiển thị:

```
;; ANSWER SECTION:
test.example.com.    300    IN    A    11.22.33.44
                      ▲                    ▲
                     TTL                 Value
```

### Kiểm chứng trên trình duyệt

- Truy cập `http://test.example.com` → trình duyệt phân giải ra IP đã cấu hình.

### Ghi chú về NS records ⭐

Trong hosted zone luôn có sẵn:

| Type | Ý nghĩa |
|------|---------|
| **NS** | ⭐ **4 name server của AWS** phục vụ hosted zone này — Domain Registrar phải trỏ tới đây |
| **SOA** | Thông tin quản trị của zone (primary name server, email admin, refresh/retry/expire) |

> ⚠️ **KHÔNG xóa hoặc sửa bản ghi NS và SOA** — làm vậy domain sẽ ngừng phân giải.

---

## 106. Route 53 - EC2 Setup

### Mục tiêu

Chuẩn bị **3 EC2 instance ở 3 Region khác nhau** + **1 Application Load Balancer**, làm nền tảng cho toàn bộ các bài Routing Policy phía sau.

### Script User Data chính thức của khóa học ⭐

Nội dung file `code_v2025-10-27/route53/user-data.sh`:

```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
# updated script to make it work with Amazon Linux 2023
CHECK_IMDSV1_ENABLED=$(curl -s -o /dev/null -w "%{http_code}" http://169.254.169.254/latest/meta-data/)
if [[ "$CHECK_IMDSV1_ENABLED" -eq 200 ]]
then
    EC2_AVAIL_ZONE="$(curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone)"
else
    EC2_AVAIL_ZONE="$(TOKEN=`curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"` && curl -s -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/placement/availability-zone)"
fi
echo "<h1>Hello world from $(hostname -f) in AZ $EC2_AVAIL_ZONE </h1>" > /var/www/html/index.html
```

### Giải thích script ⭐

| Phần | Ý nghĩa |
|------|---------|
| `yum install -y httpd` + `systemctl start/enable` | Cài và bật Apache web server |
| `CHECK_IMDSV1_ENABLED=$(curl ... -w "%{http_code}" ...)` | ⭐ **Kiểm tra Instance Metadata Service v1 có bật không** (trả về `200` nếu có) |
| Nhánh `if` (IMDSv1) | Gọi metadata trực tiếp — cách cũ, đơn giản |
| Nhánh `else` (**IMDSv2**) | ⭐ **Lấy TOKEN bằng `PUT /latest/api/token` trước**, rồi gửi kèm header `X-aws-ec2-metadata-token` — bắt buộc trên **Amazon Linux 2023** |
| `.../placement/availability-zone` | ⭐ Lấy **AZ của chính instance** từ metadata |
| `echo "<h1>Hello world from $(hostname -f) in AZ $EC2_AVAIL_ZONE </h1>"` | Trang chủ hiển thị **hostname + AZ** → dễ nhận biết instance nào đang trả lời |

> ⭐ **IMDSv2 an toàn hơn IMDSv1** vì yêu cầu token, chống được lỗ hổng SSRF. Amazon Linux 2023 mặc định bắt buộc IMDSv2 — đó là lý do script phải có nhánh xử lý.

### Các bước triển khai

1. **Tạo instance ở Region 1** (ví dụ `us-east-1` — N. Virginia):
   - AMI: Amazon Linux 2023, type `t2.micro`
   - Security Group: mở **HTTP (80)** từ `0.0.0.0/0`
   - **User data**: dán script ở trên
2. **Lặp lại ở Region 2** (`eu-west-1` — Ireland) và **Region 3** (`ap-southeast-1` — Singapore).
3. Ghi lại **Public IP** của cả 3 instance.
4. Truy cập từng IP → thấy `Hello world from ip-xxx in AZ us-east-1a` (AZ khác nhau).

### Tạo Application Load Balancer ⭐

1. Ở **một Region** (ví dụ `us-east-1`), tạo **ALB** + **Target Group** trỏ tới instance của Region đó.
2. Ghi lại **DNS name của ALB** — sẽ dùng để minh họa **Alias record** ở bài 108.

> 💡 Ghi lại bảng này để dùng cho các bài sau:
>
> | Region | Public IP | Ghi chú |
> |---|---|---|
> | `us-east-1` | `54.x.x.x` | + có ALB |
> | `eu-west-1` | `52.x.x.x` | |
> | `ap-southeast-1` | `13.x.x.x` | |

---

## 107. Route 53 - TTL

### TTL là gì?

**TTL (Time To Live)** = khoảng thời gian **client cache lại kết quả DNS** trước khi hỏi lại Route 53.

```
   Client ──DNS Request: myapp.example.com?──► Amazon Route 53
          ◄── A 12.34.56.78 (with TTL) ────────┘
          │
          │  Client sẽ CACHE kết quả trong đúng TTL của record
          │
          └──HTTP Request──► Web Server ──HTTP Response──►
```

### So sánh High TTL vs Low TTL ⭐⭐

| | **High TTL** (ví dụ **24 giờ**) | **Low TTL** (ví dụ **60 giây**) |
|---|---|---|
| **Traffic tới Route 53** | ⭐ **ÍT hơn** | ⭐ **NHIỀU hơn ($$)** |
| **Chi phí** | Thấp | **Cao hơn** |
| **Độ mới của record** | ⚠️ ⭐ **Record CÓ THỂ bị lỗi thời (outdated)** | ⭐ **Record lỗi thời trong thời gian NGẮN hơn** |
| **Đổi record** | Khó (phải chờ cache hết hạn) | ⭐ **DỄ thay đổi record** |

### ⭐⭐ Quy tắc quan trọng:

> **NGOẠI TRỪ Alias records, TTL là BẮT BUỘC cho MỖI DNS record.**

### Chiến lược thực tế ⭐

| Tình huống | TTL nên đặt |
|-----------|-------------|
| Record ổn định, ít đổi | **Cao** (24 giờ) — tiết kiệm chi phí |
| **Sắp thay đổi IP / migration** | ⭐ **Hạ TTL xuống thấp (60s) TRƯỚC vài ngày**, đổi record, rồi mới nâng TTL lên lại |
| Ứng dụng cần failover nhanh | **Thấp** (60s) |

> **Đây là kỹ thuật kinh điển khi migration:** hạ TTL trước → thực hiện đổi → chờ ổn định → nâng TTL lại.

### Thực hành

1. Sửa record `test.example.com`, đổi **TTL từ 300 → 60**.
2. Chạy `dig test.example.com` nhiều lần → quan sát cột TTL **đếm ngược** (60 → 59 → 58…).
3. Khi TTL về 0, truy vấn tiếp theo mới thực sự đi tới Route 53.

---

## 108. Route 53 CNAME vs Alias

### Vấn đề ⭐

**Các tài nguyên AWS (Load Balancer, CloudFront…) đưa ra một hostname của AWS:**
- `lb1-1234.us-east-2.elb.amazonaws.com`
- **và bạn muốn dùng `myapp.mydomain.com`**

---

### ⭐⭐⭐ CNAME vs Alias (BẢNG QUAN TRỌNG NHẤT PHẦN NÀY)

| | **CNAME** | **Alias** |
|---|---|---|
| **Trỏ tới** | ⭐ **Bất kỳ hostname NÀO khác**<br>(`app.mydomain.com` => `blabla.anything.com`) | ⭐ **Một TÀI NGUYÊN AWS**<br>(`app.mydomain.com` => `blabla.amazonaws.com`) |
| **Root domain (Zone Apex)** | ❌ ⭐⭐ **CHỈ DÙNG CHO NON-ROOT DOMAIN**<br>(chỉ `something.mydomain.com`) | ✅ ⭐⭐ **HOẠT ĐỘNG cho CẢ ROOT DOMAIN và NON-ROOT DOMAIN**<br>(được cả `mydomain.com`) |
| **Chi phí** | Tính phí như record thường | ⭐ **MIỄN PHÍ (free of charge)** |
| **Health check** | Không có sẵn | ⭐ **Có native health check** |
| **TTL** | Bắt buộc đặt | ⭐ **KHÔNG THỂ đặt TTL** (AWS tự quản lý) |
| **Record type** | `CNAME` | ⭐ **LUÔN là `A` hoặc `AAAA`** (cho IPv4/IPv6) |

---

### Route 53 — Alias Records ⭐⭐

- ⭐ **Ánh xạ một hostname tới một tài nguyên AWS**
- ⭐ **Là phần MỞ RỘNG của chức năng DNS** (không phải chuẩn DNS gốc)
- ⭐ **TỰ ĐỘNG nhận biết thay đổi địa chỉ IP của tài nguyên**
- ⭐ **Khác với CNAME, nó dùng được cho NODE CAO NHẤT của DNS namespace (Zone Apex)** — ví dụ `example.com`
- ⭐ **Alias Record LUÔN có type A/AAAA** cho tài nguyên AWS (IPv4 / IPv6)
- ⭐ **Bạn KHÔNG THỂ đặt TTL**

```
   Record Name    Type    Value
   example.com     A      MyALB-123456789.us-east-1.elb.amazonaws.com
                          (AWS-Managed — IP addresses might change)
```

---

### Route 53 — Alias Records Targets ⭐⭐⭐

**Alias record có thể trỏ tới:**

| Target | ✅ |
|--------|---|
| **Elastic Load Balancers** | ✅ |
| **CloudFront Distributions** | ✅ |
| **API Gateway** | ✅ |
| **Elastic Beanstalk environments** | ✅ |
| **S3 Websites** | ✅ |
| **VPC Interface Endpoints** | ✅ |
| **Global Accelerator accelerator** | ✅ |
| **Route 53 record trong CÙNG hosted zone** | ✅ |

### ⚠️⭐⭐ **BẠN KHÔNG THỂ đặt ALIAS record cho một EC2 DNS name!**

> **Đây là bẫy thi kinh điển.** Muốn trỏ domain tới EC2 → dùng **record `A` với Public IP**, hoặc đặt EC2 sau **ALB** rồi Alias tới ALB.

### Mẹo nhớ ⭐

```
Root domain (example.com)     → PHẢI dùng ALIAS
Subdomain (www.example.com)   → dùng CNAME hoặc ALIAS đều được
Trỏ tới tài nguyên AWS        → ưu tiên ALIAS (miễn phí + health check)
Trỏ tới hostname ngoài AWS    → PHẢI dùng CNAME
Trỏ tới EC2 DNS name          → ❌ KHÔNG Alias được → dùng A record với IP
```

---

## 109. Routing Policy - Simple

### Routing Policies là gì? ⭐

- ⭐ **Định nghĩa cách Route 53 PHẢN HỒI các truy vấn DNS**
- ⚠️ ⭐⭐ **ĐỪNG NHẦM LẪN với từ "Routing"**:
  - **KHÔNG giống routing của Load Balancer** (cái đó định tuyến traffic thật)
  - ⭐ **DNS KHÔNG định tuyến traffic nào cả — nó CHỈ PHẢN HỒI các truy vấn DNS**

### Route 53 hỗ trợ 7 Routing Policy ⭐⭐

| # | Policy |
|---|--------|
| 1 | **Simple** |
| 2 | **Weighted** |
| 3 | **Failover** |
| 4 | **Latency based** |
| 5 | **Geolocation** |
| 6 | **Multi-Value Answer** |
| 7 | **Geoproximity** (dùng tính năng **Route 53 Traffic Flow**) |

> *(Bài 117 bổ sung thêm **IP-based routing**.)*

---

### Simple Routing Policy ⭐

- ⭐ **Thường dùng để định tuyến traffic tới MỘT tài nguyên duy nhất**
- ⭐ **Có thể chỉ định NHIỀU GIÁ TRỊ trong CÙNG MỘT record**
- ⭐ **Nếu nhiều giá trị được trả về, CLIENT sẽ chọn NGẪU NHIÊN một giá trị**
- ⭐ **Khi bật Alias, chỉ được chỉ định MỘT tài nguyên AWS**
- ⚠️ ⭐⭐ **KHÔNG THỂ liên kết với Health Checks**

```
   Single Value:
   Client ──► Route 53 ──► foo.example.com  A 11.22.33.44

   Multiple Value:
   Client ──► Route 53 ──► foo.example.com  A 11.22.33.44
                                            A 55.66.77.88
                                            A 99.11.22.33
              ▲ Client tự chọn NGẪU NHIÊN một giá trị
```

> ⭐ **Điểm khác biệt then chốt với Multi-Value:** Simple **không có health check** → có thể trả về IP của server đã chết.

---

## 110. Routing Policy - Weighted

### Đặc điểm ⭐⭐

- ⭐ **Kiểm soát % số request đi tới MỖI tài nguyên cụ thể**
- ⭐ **Gán cho mỗi record một TRỌNG SỐ (weight) tương đối**:

```
              Weight của một record cụ thể
traffic (%) = ─────────────────────────────────  × 100
              Tổng weight của TẤT CẢ các record
```

- ⭐ **Weight KHÔNG cần cộng lại thành 100**
- ⭐⭐ **Các DNS record PHẢI CÓ CÙNG TÊN và CÙNG TYPE**
- ⭐ **CÓ THỂ liên kết với Health Checks**

### Ví dụ (từ slide)

```
                        Weight: 70  →  70%
   Client ──► Route 53 ─ Weight: 20  →  20%
                        Weight: 10  →  10%
```

### Use cases ⭐

- ⭐ **Cân bằng tải giữa các Region**
- ⭐ **Test phiên bản ứng dụng mới** (canary deployment — cho 5% traffic vào version mới)

### Mẹo quan trọng ⭐⭐

- ⭐ **Gán weight = 0 cho một record để NGỪNG gửi traffic tới tài nguyên đó**
- ⭐ **Nếu TẤT CẢ record đều có weight = 0, thì TẤT CẢ record sẽ được trả về ĐỀU NHAU**

> **Bẫy thi:** "Đặt tất cả weight = 0 thì sao?" → **KHÔNG phải là chặn hết traffic**, mà là **chia đều**.

---

## 111. Routing Policy - Latency

### Đặc điểm ⭐⭐

- ⭐ **Chuyển hướng tới tài nguyên có ĐỘ TRỄ THẤP NHẤT so với vị trí của chúng ta**
- ⭐ **Cực kỳ hữu ích khi độ trễ cho người dùng là ưu tiên hàng đầu**
- ⭐⭐ **Độ trễ được tính dựa trên traffic GIỮA NGƯỜI DÙNG và CÁC AWS REGION**
- ⚠️ ⭐ **Người dùng ở Đức CÓ THỂ được chuyển hướng tới Mỹ** (nếu đó là đường có độ trễ thấp nhất)
- ⭐ **CÓ THỂ liên kết với Health Checks** (có khả năng failover)

```
   Users ──► Route 53 ──► ALB (us-east-1)          ← latency thấp nhất cho user Mỹ
                      └─► ALB (ap-southeast-1)      ← latency thấp nhất cho user châu Á
```

### ⭐⭐ Latency vs Geolocation — phân biệt rõ

| | **Latency-based** | **Geolocation** |
|---|---|---|
| **Dựa trên** | ⭐ **ĐỘ TRỄ MẠNG thực đo được** | ⭐ **VỊ TRÍ ĐỊA LÝ của user** |
| **Kết quả** | Region **nhanh nhất** (có thể không phải gần nhất) | Region **đúng theo quy định của bạn** |
| **User Đức** | Có thể bị đưa sang **Mỹ** nếu nhanh hơn | Luôn đi tới **endpoint đã gán cho Đức** |
| **Use case** | **Tối ưu hiệu năng** | **Localization, giới hạn nội dung theo quốc gia** |

> **Bẫy thi:** Đề nói "ưu tiên **performance/latency**" → **Latency-based**. Đề nói "user ở nước X **phải** vào server Y" (lý do pháp lý/bản địa hóa) → **Geolocation**.

---

## 112. Route 53 - Health Checks

### ⚠️⭐ **HTTP Health Checks CHỈ dành cho tài nguyên PUBLIC**

### Health Check => Automated DNS Failover — 3 loại ⭐⭐⭐

| # | Loại Health Check | Mô tả |
|---|-------------------|-------|
| **1** | **Monitor an endpoint** | **Giám sát một endpoint** (application, server, tài nguyên AWS khác) |
| **2** | **Calculated Health Checks** | ⭐ **Health check giám sát CÁC HEALTH CHECK KHÁC** |
| **3** | **Monitor CloudWatch Alarms** | ⭐⭐ **Health check giám sát CloudWatch Alarms (toàn quyền kiểm soát!!)** — ví dụ throttle của DynamoDB, alarm trên RDS, custom metrics… ⭐ **hữu ích cho tài nguyên PRIVATE** |

- ⭐ **Health Checks được tích hợp với CloudWatch metrics**

---

### 1️⃣ Health Checks — Monitor an Endpoint ⭐⭐

Các thông số **rất hay ra thi**:

| Thông số | Giá trị |
|----------|---------|
| **Số health checker toàn cầu** | ⭐ **Khoảng 15** |
| **Status code để PASS** | ⭐⭐ **Chỉ pass khi endpoint trả về 2xx và 3xx** |
| **Kiểm tra theo nội dung** | ⭐ **Có thể set pass/fail dựa trên TEXT trong 5120 BYTE ĐẦU TIÊN của response** |
| **Healthy/Unhealthy Threshold** | ⭐ **3 (mặc định)** |
| **Interval** | ⭐ **30 giây** (có thể đặt **10 giây** — **chi phí cao hơn**) |
| **Giao thức hỗ trợ** | ⭐ **HTTP, HTTPS và TCP** |
| **Ngưỡng đánh giá** | ⭐⭐ **Nếu > 18% health checker báo endpoint là healthy → Route 53 coi là Healthy. Ngược lại là Unhealthy** |
| **Chọn vị trí** | ⭐ **Có thể chọn những location nào bạn muốn Route 53 sử dụng** |

### ⚠️⭐ Yêu cầu firewall:

> **Cấu hình router/firewall của bạn để CHO PHÉP các request đến từ Route 53 Health Checkers.**
> Dải IP: **`https://ip-ranges.amazonaws.com/ip-ranges.json`**

```
   Health Checker (sa-east-1) ──┐
   Health Checker (us-west-1) ──┼── HTTP request to /health ──► ALB (eu-west-1)
   Health Checker (us-east-1) ──┘   ◄────── 200 code ─────────  Auto Scaling group
                                                                  EC2 Instance
                                    ▲ Phải cho phép incoming request
                                      từ dải IP của Route 53 Health Checkers
```

---

### 2️⃣ Route 53 — Calculated Health Checks ⭐⭐

- ⭐ **KẾT HỢP kết quả của NHIỀU Health Check thành MỘT Health Check duy nhất**
- ⭐ **Có thể dùng toán tử OR, AND, hoặc NOT**
- ⭐⭐ **Giám sát được TỐI ĐA 256 Child Health Checks**
- ⭐ **Chỉ định BAO NHIÊU health check cần pass để health check CHA được pass**
- ⭐ **Công dụng: thực hiện BẢO TRÌ website mà KHÔNG làm tất cả health check fail**

```
                Health Check (Parent)
                  ╱       │       ╲     (OR / AND / NOT, tối đa 256 child)
   Health Check(Child) HC(Child) HC(Child)
        │monitor      │monitor    │monitor
      EC2            EC2         EC2
```

---

### 3️⃣ Health Checks — Private Hosted Zones ⭐⭐

### Vấn đề:

- ⭐⭐ **Route 53 health checkers nằm NGOÀI VPC**
- ⭐ **Chúng KHÔNG THỂ truy cập các endpoint PRIVATE** (private VPC hoặc tài nguyên on-premises)

### Giải pháp ⭐:

> **Tạo một CloudWatch Metric và gắn với một CloudWatch Alarm, rồi tạo một Health Check kiểm tra CHÍNH CÁI ALARM ĐÓ.**

```
   ┌────────── VPC ──────────┐
   │    Private subnet        │
   │       EC2 ──monitor──► CloudWatch Alarm
   └──────────────────────────┘         ▲
                                        │ monitor
                     Health Checker (us-east-1)  ← nằm NGOÀI VPC
```

> **Đây là câu trả lời chuẩn cho:** *"Làm sao health check một tài nguyên private/on-premises?"* → **CloudWatch Alarm + Health Check trên alarm đó**.

---

## 113. Route 53 - Health Checks Hands On

### Bước 1 — Tạo Health Check giám sát endpoint

1. Route 53 → menu trái → **Health checks** → **Create health check**.
2. **Configure health check**:
   - **Name**: `EC2-us-east-1`
   - **What to monitor**: ⭐ **Endpoint**
   - **Specify endpoint by**: **IP address** (hoặc Domain name)
   - **IP address**: Public IP của EC2 instance ở `us-east-1`
   - **Protocol**: `HTTP`, **Port**: `80`, **Path**: để trống hoặc `/`
3. **Advanced configuration**:
   - ⭐ **Request interval**: `Standard (30 seconds)` / `Fast (10 seconds — chi phí cao hơn)`
   - ⭐ **Failure threshold**: `3` (mặc định)
   - ⭐ **String matching**: bật để kiểm tra text trong **5120 byte đầu** của response
   - ⭐ **Health checker regions**: `Use recommended` hoặc tự chọn
4. **Next** → **Get notified when health check fails**: chọn `No` (hoặc tạo SNS topic).
5. **Create health check**.

### Bước 2 — Quan sát trạng thái

1. Đợi ~1–2 phút → cột **Status** chuyển thành ⭐ **`Healthy`**.
2. Tab **Health checkers** → thấy **danh sách các checker toàn cầu** và kết quả từng cái.
3. Tab **Monitoring** → biểu đồ CloudWatch `HealthCheckStatus`.

### Bước 3 — Thử làm endpoint fail ⭐

**Cách 1:** Sửa Security Group của EC2, **xóa rule HTTP (80)**.

**Cách 2:** SSH vào instance và dừng web server:
```bash
sudo systemctl stop httpd
```

Quan sát:
- Sau **3 lần fail liên tiếp** (threshold = 3) × **30 giây** → **~90 giây** sau, status chuyển thành ⭐ **`Unhealthy`**.
- Tab **Health checkers** → thấy các checker báo `Failure`.

Khôi phục:
```bash
sudo systemctl start httpd
```
→ Sau ~90 giây, status quay lại **`Healthy`**.

### Bước 4 — Tạo Calculated Health Check ⭐

1. Tạo thêm health check cho các instance ở Region khác (`eu-west-1`, `ap-southeast-1`).
2. **Create health check**:
   - **What to monitor**: ⭐ **Status of other health checks (calculated health check)**
   - **Health checks to monitor**: chọn cả 3 health check con
   - ⭐ **Report healthy when**: `at least` **`2`** `of the selected health checks are healthy`
3. → Health check cha vẫn `Healthy` khi chỉ 1 instance chết — **cho phép bảo trì từng máy**.

### Bước 5 — Tạo Health Check trên CloudWatch Alarm ⭐

1. **Create health check** → **What to monitor**: ⭐ **State of CloudWatch alarm**.
2. Chọn **Region** và **CloudWatch alarm** đã tạo trước.
3. ⭐ Dùng cách này cho **tài nguyên private / on-premises**.

### Bước 6 — Dọn dẹp

- Xóa các health check không dùng (⚠️ **mỗi health check ~$0.50/tháng**).

---

## 114. Routing Policy - Failover

### Failover (Active-Passive) ⭐⭐

```
                    Health Check (BẮT BUỘC)
                            │
   Client ──DNS Requests──► Route 53 ──► EC2 Instance (PRIMARY)
                                │
                                └──Failover──► EC2 Instance
                                               (SECONDARY – Disaster Recovery)
```

### Đặc điểm ⭐⭐

- ⭐⭐ **Health Check là BẮT BUỘC (mandatory) cho record PRIMARY**
- Cấu trúc: **1 record Primary** + **1 record Secondary**
- ⭐ **Khi Primary được đánh giá `Unhealthy` → Route 53 TỰ ĐỘNG trả về record Secondary**
- Đây là mô hình ⭐ **Active-Passive** — dùng cho **Disaster Recovery**

### Cấu hình trên Console

1. **Create record** → **Routing policy**: `Failover`.
2. Record 1:
   - **Failover record type**: ⭐ **Primary**
   - **Health check ID**: ⭐ **chọn health check** (bắt buộc)
   - **Record ID**: `Primary-us-east-1`
3. Record 2:
   - **Failover record type**: ⭐ **Secondary**
   - **Health check ID**: tùy chọn (khuyến nghị có)
   - **Record ID**: `Secondary-eu-west-1`

### Kiểm chứng

```bash
dig myapp.example.com     # trả về IP của Primary

# Làm Primary fail (stop httpd hoặc xóa rule SG)
# Đợi health check chuyển sang Unhealthy (~90 giây)

dig myapp.example.com     # ⭐ giờ trả về IP của Secondary
```

> ⚠️ Nhớ tính cả **TTL của record** vào thời gian failover thực tế mà client cảm nhận.

---

## 115. Routing Policy - Geolocation

### Đặc điểm ⭐⭐

- ⭐⭐ **KHÁC với Latency-based!**
- ⭐ **Routing này dựa trên VỊ TRÍ CỦA NGƯỜI DÙNG**
- ⭐ **Chỉ định vị trí theo Continent (châu lục), Country (quốc gia), hoặc US State (bang của Mỹ)**
  - ⭐ **Nếu có chồng lấn (overlapping), vị trí CHÍNH XÁC NHẤT được chọn**
- ⭐⭐ **NÊN tạo một record "Default"** (phòng khi **không khớp** vị trí nào)
- ⭐ **Use cases: website localization, giới hạn phân phối nội dung, load balancing**, …
- ⭐ **CÓ THỂ liên kết với Health Checks**

```
   User ở Đức    ──► A 55.66.77.88   (record cho Germany)
   User ở Mỹ     ──► A 11.22.33.44   (record cho United States)
   User khác     ──► A 99.11.22.33   (record DEFAULT)
```

### Thứ tự ưu tiên khi chồng lấn ⭐

```
   US State  >  Country  >  Continent  >  Default
   (chính xác nhất)                      (fallback)
```

### Use case điển hình ⭐

| Tình huống | Ví dụ |
|-----------|-------|
| **Website localization** | User Pháp → server tiếng Pháp; user Nhật → server tiếng Nhật |
| **Giới hạn nội dung** | Nội dung chỉ được phép phát ở một số quốc gia (bản quyền) |
| **Tuân thủ pháp lý** | Dữ liệu EU phải được xử lý trong EU (GDPR) |

> ⚠️ **Luôn tạo record Default**, nếu không những user ở khu vực không khớp sẽ **không nhận được phản hồi nào**.

---

## 116. Routing Policy - Geoproximity

### Đặc điểm ⭐⭐

- ⭐ **Định tuyến traffic tới tài nguyên dựa trên VỊ TRÍ ĐỊA LÝ của NGƯỜI DÙNG VÀ TÀI NGUYÊN**
- ⭐⭐ **Có khả năng DỊCH CHUYỂN NHIỀU TRAFFIC HƠN tới tài nguyên dựa trên BIAS đã định nghĩa**

### ⭐⭐ Bias — con số cần nhớ

**Để thay đổi KÍCH THƯỚC của vùng địa lý, chỉ định giá trị bias:**

| Hành động | Giá trị bias |
|-----------|-------------|
| ⭐ **MỞ RỘNG (expand)** — nhiều traffic hơn tới tài nguyên | **1 đến 99** |
| ⭐ **THU HẸP (shrink)** — ít traffic hơn tới tài nguyên | **-1 đến -99** |

### Tài nguyên có thể là ⭐

- ⭐ **AWS resources** — chỉ định **AWS region**
- ⭐ **Non-AWS resources** — chỉ định **Latitude và Longitude**

### ⚠️⭐⭐ **BẠN PHẢI DÙNG Route 53 TRAFFIC FLOW để sử dụng tính năng này**

### Minh họa (từ slide)

**Trường hợp 1 — Bias đều nhau:**
```
   us-west-1              us-east-1
   Bias: 0                Bias: 0
        ← đường ranh giới ở giữa nước Mỹ →
```

**Trường hợp 2 — Tăng bias cho us-east-1:**
```
   us-west-1              us-east-1
   Bias: 0                Bias: 50   ⭐ Higher bias in us-east-1
        ← ranh giới DỊCH SANG TRÁI →
        → us-east-1 phục vụ VÙNG RỘNG HƠN
```

> **Ứng dụng thực tế:** khi một Region sắp bảo trì hoặc quá tải, hạ bias của nó xuống để đẩy bớt traffic sang Region khác.

### ⭐ Geolocation vs Geoproximity — phân biệt

| | **Geolocation** | **Geoproximity** |
|---|---|---|
| **Dựa trên** | **Vị trí user** (continent/country/state) | ⭐ **Khoảng cách giữa user VÀ tài nguyên** |
| **Điều chỉnh vùng** | ❌ Không | ⭐ **CÓ — bằng bias (-99 đến +99)** |
| **Cần Traffic Flow?** | ❌ Không | ⭐⭐ **CÓ — BẮT BUỘC** |
| **Non-AWS resource** | — | ⭐ Có (chỉ định lat/long) |

---

## 117. Routing Policy - IP-based

### Đặc điểm ⭐

- ⭐ **Routing dựa trên ĐỊA CHỈ IP CỦA CLIENT**
- ⭐⭐ **Bạn cung cấp một DANH SÁCH CIDR cho các client và các endpoint/location tương ứng** (**user-IP-to-endpoint mappings**)
- ⭐ **Use cases: tối ưu hiệu năng, GIẢM CHI PHÍ MẠNG**, …
- ⭐ **Ví dụ: định tuyến người dùng cuối từ MỘT ISP CỤ THỂ tới MỘT endpoint cụ thể**

### Cấu trúc (từ slide)

**CIDR Collection:**

| Locations | CIDR blocks |
|-----------|-------------|
| `location-1` | `203.0.113.0/24` |
| `location-2` | `200.5.4.0/24` |

**Records:**

| Record Name | Value | IP-based |
|-------------|-------|----------|
| `example.com` | `1.2.3.4` | `location-1` |
| `example.com` | `5.6.7.8` | `location-2` |

**Kết quả:**

```
   User A (203.0.113.56) ──► khớp location-1 ──► EC2 Instance (1.2.3.4)
   User B (200.5.4.100)  ──► khớp location-2 ──► EC2 Instance (5.6.7.8)
```

> **Mẹo thi:** Đề nhắc **"định tuyến theo ISP cụ thể"**, **"danh sách CIDR của client"**, **"giảm chi phí mạng"** → **IP-based routing**.

---

## 118. Routing Policy - Multi Value

### Đặc điểm ⭐⭐

- ⭐ **Dùng khi định tuyến traffic tới NHIỀU tài nguyên**
- ⭐ **Route 53 trả về NHIỀU giá trị/tài nguyên**
- ⭐⭐ **CÓ THỂ liên kết với Health Checks** (**chỉ trả về giá trị của các tài nguyên HEALTHY**)
- ⭐⭐ **Tối đa 8 record HEALTHY được trả về cho mỗi truy vấn Multi-Value**
- ⚠️ ⭐⭐ **Multi-Value KHÔNG PHẢI là thứ thay thế cho việc có một ELB**

### ⭐⭐ Simple vs Multi-Value — phân biệt (rất hay ra thi)

| | **Simple** | **Multi-Value** |
|---|---|---|
| **Trả về nhiều giá trị** | ✅ Có (trong 1 record) | ✅ Có (nhiều record) |
| **Health Checks** | ❌ ⭐ **KHÔNG hỗ trợ** | ✅ ⭐ **CÓ hỗ trợ** |
| **Chỉ trả về tài nguyên khỏe mạnh** | ❌ Không | ✅ ⭐ **Có** |
| **Số bản ghi trả về tối đa** | — | ⭐ **8 record healthy** |
| **Thay thế được ELB?** | ❌ | ❌ ⭐ **KHÔNG** |

> **Bẫy thi:** Nếu đề mô tả "trả về nhiều IP **và loại bỏ những server đã chết**" → **Multi-Value**, **không phải Simple**.
> Nếu đề hỏi "dùng Multi-Value thay cho Load Balancer được không?" → **KHÔNG**.

---

## 119. 3rd Party Domains & Route 53

### Domain Registrar vs. DNS Service ⭐⭐

- ⭐ **Bạn MUA hoặc ĐĂNG KÝ tên miền với một Domain Registrar**, thường trả **phí hàng năm** (ví dụ **GoDaddy, Amazon Registrar Inc.**, …)
- ⭐ **Domain Registrar thường cung cấp cho bạn một dịch vụ DNS để quản lý các DNS record**
- ⭐⭐ **NHƯNG bạn CÓ THỂ dùng một dịch vụ DNS KHÁC để quản lý DNS record**
- ⭐ **Ví dụ: mua domain từ GoDaddy và dùng Route 53 để quản lý DNS record**

```
   User ──purchase example.com──► GoDaddy (Registrar)
        └──manage DNS records──► Amazon Route 53 (DNS Service)
```

### ⭐⭐ 3rd Party Registrar với Amazon Route 53 — 2 BƯỚC

> **Nếu bạn mua domain ở một registrar bên thứ 3, bạn VẪN có thể dùng Route 53 làm nhà cung cấp dịch vụ DNS:**
>
> **1. Tạo một Hosted Zone trong Route 53**
> **2. CẬP NHẬT các bản ghi NS trên website của bên thứ 3 để trỏ tới Name Servers của Route 53**

### Ghi nhớ ⭐

- ⭐⭐ **Domain Registrar ≠ DNS Service**
- ⭐ **Nhưng mọi Domain Registrar thường đi kèm một vài tính năng DNS**

### Các bước chi tiết

1. **Route 53** → **Hosted zones** → **Create hosted zone**:
   - **Domain name**: `example.com` (đúng tên miền đã mua ở nơi khác)
   - **Type**: `Public hosted zone`
2. Sau khi tạo, mở record type **`NS`** → copy **4 name server** (dạng):
   ```
   ns-123.awsdns-45.com
   ns-678.awsdns-90.net
   ns-1234.awsdns-56.org
   ns-789.awsdns-01.co.uk
   ```
3. Đăng nhập vào **registrar bên thứ 3** (GoDaddy, Namecheap…):
   - Tìm mục **Nameservers** / **DNS Management** → chọn **Custom nameservers**
   - **Dán 4 name server của AWS vào**, xóa name server cũ
4. ⏳ Chờ **DNS propagation** — thường **vài phút đến 48 giờ**.
5. Kiểm chứng:
   ```bash
   dig NS example.com
   # phải trả về 4 name server awsdns-*
   ```

> ⚠️ **Đây là bước hay bị quên.** Tạo hosted zone thôi **chưa đủ** — phải **cập nhật NS ở registrar** thì Route 53 mới thực sự phục vụ domain đó.

---

## 120. Route 53 Resolvers & Hybrid DNS

### Route 53 Resolver là gì? ⭐

**Mặc định, Route 53 Resolver TỰ ĐỘNG trả lời các truy vấn DNS cho:**

- ⭐ **Tên miền cục bộ (local domain names) của các EC2 instance**
- ⭐ **Các record trong Private Hosted Zones**
- ⭐ **Các record trên public Name Servers**

```
   ┌────── Region / VPC ──────┐
   │  Private Hosted Zone      │
   │                           │──► Public Name Server
   │   Route 53 Resolver       │
   │         │                 │
   │      EC2 Instance         │
   │  (ec2-192-0-2-44.compute-1.amazonaws.com)
   └───────────────────────────┘
```

---

### Hybrid DNS ⭐⭐

- ⭐⭐ **Hybrid DNS = phân giải truy vấn DNS GIỮA VPC (Route 53 Resolver) VÀ MẠNG CỦA BẠN (các DNS Resolver khác)**
- ⭐ **Mạng có thể là:**
  - **Chính VPC đó / VPC được peer (Peered VPC)**
  - ⭐ **Mạng On-premises** (kết nối qua **Direct Connect** hoặc **AWS VPN**)

---

### ⭐⭐⭐ Resolver Endpoints — 2 loại (RẤT hay ra thi)

#### 1️⃣ Inbound Endpoint

> ⭐ **Cho phép các DNS Resolver CỦA BẠN phân giải tên miền cho TÀI NGUYÊN AWS** (ví dụ EC2 instance) **và các record trong Private Hosted Zones**

**Hướng: On-premises ──► AWS**

```
   On-Premises Data Center              us-east-1 VPC
   (onpremise.private)                  Private Hosted Zone (aws.private)
                                        Private Subnet
   DNS Resolvers ──DNS Query───────►  Resolver          Route 53
                  app.aws.private?    INBOUND Endpoint ──► Resolver
   Server                              (qua VPN hoặc DX)      │ lookup
   (web.onpremise.private)                                    ▼
                                                     EC2 (app.aws.private)
```

#### 2️⃣ Outbound Endpoint

> ⭐ **Route 53 Resolver CHUYỂN TIẾP (forwards) các truy vấn DNS tới CÁC DNS RESOLVER CỦA BẠN**

**Hướng: AWS ──► On-premises**

```
   us-east-1 VPC                        On-Premises Data Center
   Private Subnet                       (onpremise.private)
   EC2 (app.aws.private)
        │ DNS Query
        │ web.onpremise.private?
        ▼
   Route 53 Resolver ──► Resolver ──DNS Query──► DNS Resolvers
                         OUTBOUND     web.onpremise.private?      │
                         Endpoint     (qua VPN hoặc DX)           ▼
                                                    Server (web.onpremise.private)
```

### ⭐ Bảng ghi nhớ 2 loại Endpoint

| | **Inbound Endpoint** | **Outbound Endpoint** |
|---|---|---|
| **Hướng truy vấn** | ⭐ **On-premises → AWS** | ⭐ **AWS → On-premises** |
| **Ai hỏi** | DNS Resolver **của bạn** (on-prem) | **Route 53 Resolver** (trong VPC) |
| **Phân giải được gì** | ⭐ Tên miền của **tài nguyên AWS** + **Private Hosted Zone** | ⭐ Tên miền **on-premises** |
| **Cần kết nối** | **VPN hoặc Direct Connect** | **VPN hoặc Direct Connect** |
| **Cấu hình thêm** | — | ⭐ **Resolver Rules** (chỉ định domain nào forward đi đâu) |

### Mẹo nhớ ⭐

```
INBOUND  = truy vấn ĐI VÀO AWS   → on-prem muốn biết tên miền AWS
OUTBOUND = truy vấn ĐI RA khỏi AWS → EC2 muốn biết tên miền on-prem
```

> **Bẫy thi:** Đề mô tả *"EC2 trong VPC cần phân giải hostname của server on-premises"* → **Outbound Endpoint**.
> Đề mô tả *"máy chủ on-premises cần phân giải tên trong Private Hosted Zone"* → **Inbound Endpoint**.

---

## 121. Route 53 - Section Cleanup

Dọn dẹp toàn bộ tài nguyên đã tạo trong phần này để tránh phát sinh chi phí.

### Checklist dọn dẹp ⚠️

#### 1. Xóa các DNS record đã tạo

```
Route 53 → Hosted zones → chọn zone → chọn các record (test, myapp, www…) → Delete record
```

⚠️ ⭐ **KHÔNG xóa bản ghi `NS` và `SOA`** — chúng là bắt buộc cho hosted zone.

#### 2. Xóa Health Checks ⭐

```
Route 53 → Health checks → chọn tất cả → Delete health check
```

> 💰 Mỗi health check tốn **~$0.50/tháng** (endpoint AWS) hoặc **$0.75/tháng** (endpoint ngoài AWS).

#### 3. Terminate EC2 instances ở TẤT CẢ Region ⭐

```
EC2 → Instances → (lặp lại cho us-east-1, eu-west-1, ap-southeast-1) → Terminate
```

⚠️ ⭐ **Rất dễ quên các instance ở Region khác** — nhớ chuyển Region để kiểm tra hết.

#### 4. Xóa Load Balancer + Target Group

```
EC2 → Load Balancers → Delete
EC2 → Target Groups → Delete
```

#### 5. Về Hosted Zone và Domain

| Tài nguyên | Nên làm gì |
|-----------|-----------|
| **Hosted Zone** | ⭐ **Giữ lại** nếu vẫn muốn dùng domain (**$0.50/tháng**), hoặc xóa nếu không cần |
| **Registered domain** | ⚠️ ⭐ **KHÔNG hoàn tiền được** — nên **TẮT auto-renew** nếu không muốn gia hạn năm sau |

Tắt auto-renew:
```
Route 53 → Registered domains → chọn domain → Edit → Auto-renew: Off
```

### Bảng chi phí cần lưu ý

| Tài nguyên | Tính tiền khi không dùng? |
|------------|---------------------------|
| **Hosted Zone** | ✅ **$0.50/tháng** |
| **Health Check** | ✅ **~$0.50/tháng mỗi cái** |
| **DNS Records** | ❌ Miễn phí (chỉ tính query) |
| **Alias record trỏ tới tài nguyên AWS** | ❌ ⭐ **Miễn phí** |
| **EC2 instances** | ✅ Có (khi running) |
| **Domain registration** | ✅ Hàng năm (không hoàn tiền) |

---

## Trắc nghiệm 7: Route 53 Quiz

### Các điểm dễ bị bẫy

| Câu hỏi thường gặp | Đáp án đúng |
|--------------------|-------------|
| DNS làm gì? | **Dịch hostname thành IP address** |
| Root DNS Server do ai quản lý? | **ICANN** |
| TLD DNS Server do ai quản lý? | **IANA** (nhánh của ICANN) |
| SLD DNS Server do ai quản lý? | **Domain Registrar** |
| FQDN là gì? | **Fully Qualified Domain Name** — tên miền đầy đủ |
| Vì sao gọi là "Route 53"? | **53 là port DNS truyền thống** |
| Route 53 có SLA bao nhiêu? | ⭐ **100% availability — dịch vụ AWS DUY NHẤT** |
| "Authoritative" nghĩa là gì? | **Khách hàng có thể cập nhật DNS records** |
| Route 53 có phải Domain Registrar? | ✅ **CÓ** |
| 4 record type bắt buộc biết? | **A / AAAA / CNAME / NS** |
| Record `A` ánh xạ tới gì? | **IPv4** |
| Record `AAAA` ánh xạ tới gì? | **IPv6** |
| CNAME tạo được cho Zone Apex không? | ❌ ⭐ **KHÔNG** (không tạo được cho `example.com`) |
| Hosted Zone giá bao nhiêu? | ⭐ **$0.50/tháng** |
| Private Hosted Zone dùng cho gì? | **Phân giải tên miền private TRONG VPC** |
| High TTL có ưu/nhược gì? | Ít traffic Route 53 nhưng **record có thể lỗi thời** |
| TTL bắt buộc cho record nào? | ⭐ **MỌI record NGOẠI TRỪ Alias** |
| Alias trỏ tới root domain được không? | ✅ ⭐ **ĐƯỢC** (CNAME thì không) |
| Alias có tính phí không? | ⭐ **MIỄN PHÍ** |
| Alias có đặt TTL được không? | ❌ **KHÔNG** |
| Alias luôn là record type gì? | ⭐ **A hoặc AAAA** |
| Alias trỏ tới EC2 DNS name được không? | ❌ ⭐⭐ **KHÔNG ĐƯỢC** |
| Alias trỏ được tới những gì? | **ELB, CloudFront, API Gateway, Elastic Beanstalk, S3 Websites, VPC Interface Endpoints, Global Accelerator, record cùng hosted zone** |
| DNS có định tuyến traffic không? | ❌ ⭐ **KHÔNG — chỉ trả lời truy vấn DNS** |
| Route 53 có mấy routing policy? | **7** (+ IP-based) |
| Simple policy có health check không? | ❌ ⭐ **KHÔNG** |
| Simple trả nhiều giá trị thì client làm gì? | **Chọn NGẪU NHIÊN một giá trị** |
| Weighted: weight có cần cộng thành 100? | ❌ **KHÔNG** |
| Weighted: các record phải giống gì? | ⭐ **CÙNG TÊN và CÙNG TYPE** |
| Weighted: đặt weight = 0 cho 1 record? | **Ngừng gửi traffic tới nó** |
| Weighted: TẤT CẢ weight = 0? | ⭐ **Tất cả record được trả về ĐỀU NHAU** |
| Latency-based dựa trên gì? | ⭐ **Traffic giữa user và AWS Region** |
| User Đức có thể bị đưa sang Mỹ không? | ✅ ⭐ **CÓ** (nếu latency thấp nhất) |
| Health check chỉ dùng cho tài nguyên nào? | ⭐ **PUBLIC** (HTTP health check) |
| Có bao nhiêu health checker toàn cầu? | ⭐ **Khoảng 15** |
| Health check pass với status code nào? | ⭐ **2xx và 3xx** |
| Health check đọc bao nhiêu byte để match text? | ⭐ **5120 byte đầu tiên** |
| Healthy/Unhealthy threshold mặc định? | ⭐ **3** |
| Interval mặc định / nhanh? | ⭐ **30 giây / 10 giây (đắt hơn)** |
| Health check hỗ trợ giao thức nào? | ⭐ **HTTP, HTTPS, TCP** |
| Bao nhiêu % checker báo healthy thì coi là Healthy? | ⭐⭐ **> 18%** |
| Calculated Health Check giám sát tối đa bao nhiêu con? | ⭐ **256** |
| Calculated Health Check dùng toán tử nào? | ⭐ **OR, AND, NOT** |
| Health check cho tài nguyên PRIVATE làm sao? | ⭐⭐ **CloudWatch Alarm + Health Check trên alarm đó** |
| Vì sao không health check trực tiếp private resource? | ⭐ **Health checkers nằm NGOÀI VPC** |
| Failover policy: health check có bắt buộc? | ✅ ⭐⭐ **BẮT BUỘC cho record Primary** |
| Failover là mô hình gì? | ⭐ **Active-Passive** |
| Geolocation dựa trên gì? | ⭐ **Vị trí user** (continent/country/US state) |
| Geolocation nên tạo thêm record gì? | ⭐⭐ **Default record** |
| Geolocation khi chồng lấn chọn cái nào? | ⭐ **Vị trí CHÍNH XÁC NHẤT** |
| Geoproximity bias để mở rộng / thu hẹp? | ⭐ **1 đến 99 / -1 đến -99** |
| Geoproximity bắt buộc dùng gì? | ⭐⭐ **Route 53 Traffic Flow** |
| Geoproximity trỏ tới non-AWS resource thế nào? | ⭐ **Chỉ định Latitude và Longitude** |
| IP-based routing dựa trên gì? | ⭐ **CIDR của client** (user-IP-to-endpoint mapping) |
| IP-based use case? | ⭐ **Tối ưu hiệu năng, giảm chi phí mạng, định tuyến theo ISP** |
| Multi-Value trả về tối đa bao nhiêu record? | ⭐⭐ **8 record healthy** |
| Multi-Value có health check không? | ✅ ⭐ **CÓ** |
| Multi-Value thay được ELB không? | ❌ ⭐⭐ **KHÔNG** |
| Mua domain ở GoDaddy, dùng Route 53 làm DNS — làm sao? | ⭐⭐ **1) Tạo Hosted Zone; 2) Cập nhật NS records ở registrar** |
| Domain Registrar có bằng DNS Service không? | ❌ ⭐ **KHÔNG** (nhưng registrar thường kèm tính năng DNS) |
| Route 53 Resolver mặc định trả lời gì? | **Local domain của EC2, Private Hosted Zone records, public Name Servers** |
| On-prem muốn phân giải tên AWS → dùng gì? | ⭐⭐ **Inbound Endpoint** |
| EC2 muốn phân giải tên on-prem → dùng gì? | ⭐⭐ **Outbound Endpoint** |
| Hybrid DNS cần kết nối gì? | ⭐ **Direct Connect hoặc AWS VPN** |

### Checklist tự kiểm tra trước khi làm quiz

- [ ] Nhớ **ICANN (root) / IANA (TLD) / Registrar (SLD)**
- [ ] Nhớ Route 53 là **dịch vụ AWS duy nhất có SLA 100%**
- [ ] Thuộc **bảng CNAME vs Alias** — đặc biệt **Alias dùng được cho root domain, miễn phí, không đặt TTL, KHÔNG trỏ được EC2**
- [ ] Thuộc **danh sách Alias targets** (ELB, CloudFront, API GW, Beanstalk, S3 Website, VPC Interface Endpoint, Global Accelerator)
- [ ] Nhớ **TTL bắt buộc trừ Alias**, và kỹ thuật **hạ TTL trước khi migration**
- [ ] Phân biệt **7 routing policies** và từ khóa nhận diện từng cái
- [ ] Nhớ **Simple KHÔNG có health check**, **Multi-Value CÓ + tối đa 8 record**
- [ ] Nhớ các số Health Check: **~15 checker**, **2xx/3xx**, **5120 byte**, **threshold 3**, **30s/10s**, **> 18%**, **256 child**
- [ ] Nhớ **health check tài nguyên private → CloudWatch Alarm**
- [ ] Nhớ **Failover bắt buộc health check trên Primary**
- [ ] Nhớ **Geoproximity: bias ±1..99, bắt buộc Traffic Flow**
- [ ] Phân biệt **Latency (hiệu năng)** vs **Geolocation (vị trí/pháp lý)** vs **Geoproximity (khoảng cách + bias)**
- [ ] Nhớ **Inbound (on-prem → AWS)** vs **Outbound (AWS → on-prem)**
- [ ] Nhớ **2 bước dùng registrar bên thứ 3**: tạo Hosted Zone + đổi NS records

---

## Thuật ngữ Anh — Việt

| Tiếng Anh | Tiếng Việt |
|-----------|-----------|
| Domain Name System (DNS) | Hệ thống phân giải tên miền |
| Hierarchical naming structure | Cấu trúc đặt tên phân cấp |
| Domain Registrar | Nhà đăng ký tên miền |
| Zone File | Tệp chứa các bản ghi DNS |
| Name Server | Máy chủ phân giải tên |
| Authoritative | Có thẩm quyền (được phép cập nhật record) |
| Top Level Domain (TLD) | Tên miền cấp cao nhất (.com, .org) |
| Second Level Domain (SLD) | Tên miền cấp hai (amazon.com) |
| Sub Domain | Tên miền con |
| FQDN | Tên miền đầy đủ |
| DNS Resolver | Bộ phân giải DNS |
| Hosted Zone | Vùng lưu trữ bản ghi DNS |
| Public / Private Hosted Zone | Vùng công khai / nội bộ |
| Record | Bản ghi DNS |
| Zone Apex | Node gốc của vùng (example.com) |
| TTL (Time To Live) | Thời gian cache bản ghi |
| Outdated record | Bản ghi lỗi thời |
| Alias record | Bản ghi bí danh (trỏ tới tài nguyên AWS) |
| Routing Policy | Chính sách phản hồi truy vấn DNS |
| Simple routing | Định tuyến đơn giản |
| Weighted routing | Định tuyến theo trọng số |
| Weight | Trọng số |
| Latency-based routing | Định tuyến theo độ trễ |
| Failover routing | Định tuyến chuyển dự phòng |
| Active-Passive | Chủ động - Bị động |
| Primary / Secondary | Chính / Dự phòng |
| Geolocation routing | Định tuyến theo vị trí địa lý |
| Geoproximity routing | Định tuyến theo khoảng cách địa lý |
| Bias | Độ lệch (mở rộng/thu hẹp vùng) |
| Traffic Flow | Tính năng thiết kế luồng traffic của Route 53 |
| IP-based routing | Định tuyến theo dải IP client |
| CIDR Collection | Tập hợp các dải CIDR |
| Multi-Value Answer | Trả về nhiều giá trị |
| Health Check | Kiểm tra sức khỏe |
| Health Checker | Máy kiểm tra sức khỏe của Route 53 |
| Calculated Health Check | Health check tổng hợp |
| Child / Parent Health Check | Health check con / cha |
| Threshold | Ngưỡng |
| Interval | Chu kỳ kiểm tra |
| String matching | So khớp chuỗi trong response |
| Route 53 Resolver | Bộ phân giải DNS của VPC |
| Hybrid DNS | DNS lai (giữa AWS và mạng riêng) |
| Inbound / Outbound Endpoint | Điểm cuối vào / ra của Resolver |
| Resolver Rules | Quy tắc chuyển tiếp truy vấn |
| Peered VPC | VPC được kết nối ngang hàng |
| Direct Connect (DX) | Kết nối riêng tới AWS |
| DNS propagation | Quá trình lan truyền thay đổi DNS |

---

*Ghi chú: các phần Hands On được tóm tắt lại các bước thao tác chính trên AWS Console. Giao diện Console có thể thay đổi theo thời gian — logic và khái niệm vẫn giữ nguyên. ⚠️ Phần này cần MUA TÊN MIỀN thật (không nằm trong Free Tier, không hoàn tiền); Hosted Zone tốn $0.50/tháng và mỗi Health Check ~$0.50/tháng — nhớ dọn dẹp theo bài 121.*
