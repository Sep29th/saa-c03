# Phần 7 — EC2 Instance Storage

> Khóa học: *Ultimate AWS Certified Solutions Architect Associate 2026* (Stéphane Maarek) — SAA-C03
> Nguồn tham chiếu: `AWS Certified Solutions Architect Slides v48.pdf` (phần "Amazon EC2 – Instance Storage")
> Code kèm theo: `code_v2025-10-27/efs/efs.sh`, `code_v2025-10-27/ebs/commands.txt`

---

## Mục lục

| # | Bài giảng | Thời lượng | Loại |
|---|-----------|-----------|------|
| 56 | [EBS Overview](#56-ebs-overview) | 5 phút | Video |
| 57 | [EBS Hands On](#57-ebs-hands-on) | 6 phút | Video |
| 58 | [EBS Snapshots](#58-ebs-snapshots) | 2 phút | Video |
| 59 | [EBS Snapshots - Hands On](#59-ebs-snapshots---hands-on) | 4 phút | Video |
| 60 | [AMI Overview](#60-ami-overview) | 3 phút | Video |
| 61 | [AMI Hands On](#61-ami-hands-on) | 5 phút | Video |
| 62 | [EC2 Instance Store](#62-ec2-instance-store) | 3 phút | Video |
| 63 | [EBS Volume Types](#63-ebs-volume-types) | 5 phút | Video |
| 64 | [EBS Multi-Attach](#64-ebs-multi-attach) | 2 phút | Video |
| 65 | [EBS Encryption](#65-ebs-encryption) | 4 phút | Video |
| 66 | [Amazon EFS](#66-amazon-efs) | 5 phút | Video |
| 67 | [Amazon EFS - Hands On](#67-amazon-efs---hands-on) | 13 phút | Video |
| 68 | [EFS vs EBS](#68-efs-vs-ebs) | 2 phút | Video |
| 69 | [EBS & EFS - Section Cleanup](#69-ebs--efs---section-cleanup) | 2 phút | Video |
| — | [Trắc nghiệm 4: EC2 Data Management Quiz](#trắc-nghiệm-4-ec2-data-management-quiz) | — | Quiz |

---

## 56. EBS Overview

### EBS Volume là gì?

- **EBS = Elastic Block Store**.
- Là một **ổ đĩa mạng (network drive)** bạn có thể **gắn (attach) vào instance trong lúc chúng đang chạy**.
- Cho phép instance **lưu dữ liệu bền vững, kể cả sau khi instance bị terminate**.
- **Chỉ mount được vào MỘT instance tại một thời điểm** (ở cấp độ CCP — Cloud Practitioner).
- **Bị ràng buộc vào một Availability Zone cụ thể**.
- **Ví von: hãy nghĩ về nó như một "chiếc USB qua mạng" (network USB stick)** 🔌

### Đặc điểm chi tiết ⭐

#### 1. Là ổ đĩa MẠNG (không phải ổ vật lý)

- Dùng **mạng để giao tiếp với instance** → **có thể có một chút độ trễ (latency)**.
- **Có thể detach khỏi một EC2 instance và attach sang instance khác rất nhanh**.

#### 2. Bị khóa vào một Availability Zone (AZ) ⭐⭐

- **Một EBS Volume ở `us-east-1a` KHÔNG THỂ attach vào instance ở `us-east-1b`**.
- **Để di chuyển volume sang AZ khác, trước tiên phải SNAPSHOT nó**.

#### 3. Có capacity được cấp phát trước (provisioned capacity)

- Cấp phát theo **size (GB)** và **IOPS**.
- **Bạn bị tính tiền cho TOÀN BỘ capacity đã cấp phát** — dù có dùng hết hay không.
- **Có thể tăng dung lượng ổ đĩa theo thời gian**.

### Sơ đồ ví dụ (từ slide)

```
        US-EAST-1A                      US-EAST-1B
   ┌──────────────────┐            ┌──────────────────┐
   │  EC2 ── EBS 10GB │            │  EC2 ── EBS 50GB │
   │      └─ EBS 100GB│            │                  │
   │                  │            │      EBS 10GB    │
   │      EBS 50GB    │            │    (unattached)  │
   └──────────────────┘            └──────────────────┘
```

Nhận xét từ sơ đồ:
- Một instance **gắn được NHIỀU EBS volume** (10GB + 100GB).
- Một EBS volume **có thể tồn tại mà không gắn vào instance nào** (`unattached`) — vẫn bị tính tiền.
- Volume **không đi qua ranh giới AZ**.

### EBS – Delete on Termination attribute ⭐⭐

- Thuộc tính **kiểm soát hành vi của EBS khi EC2 instance bị terminate**.

| Loại volume | Mặc định | Ý nghĩa |
|-------------|----------|---------|
| **Root EBS volume** | **BỊ XÓA** (attribute **enabled**) | Terminate instance → mất root volume |
| **Mọi EBS volume khác đã attach** | **KHÔNG bị xóa** (attribute **disabled**) | Terminate instance → volume vẫn còn |

- Có thể điều khiển qua **AWS Console / AWS CLI**.
- **Use case**: **giữ lại root volume khi instance bị terminate** (ví dụ để điều tra sự cố hoặc khôi phục dữ liệu).

> **Ghi nhớ thi:** Câu hỏi "làm sao giữ dữ liệu root volume sau khi terminate?" → **tắt (disable) thuộc tính Delete on Termination**.

---

## 57. EBS Hands On

### 1. Xem EBS volume của instance

1. EC2 → menu trái → **Elastic Block Store** → **Volumes**.
2. Thấy volume root được tạo tự động cùng instance, các cột:
   - **Volume ID** (`vol-0a1b2c3d…`)
   - **Size** (`8 GiB`)
   - **Volume type** (`gp3` / `gp2`)
   - **IOPS**, **Throughput**
   - **Availability Zone** ⭐ (ví dụ `eu-west-3a`)
   - **State**: `in-use` / `available`
   - **Attached resources**: instance ID + device (`/dev/xvda`)

### 2. Tạo EBS volume mới

1. **Create volume**:
   - **Volume type**: `gp2` (General Purpose SSD)
   - **Size**: `2 GiB`
   - **Availability Zone**: ⚠️ **PHẢI chọn đúng AZ mà instance của bạn đang chạy**
   - **Encryption**: có thể bật (tùy chọn)
2. **Create volume** → volume mới có state **`available`**.

### 3. Attach volume vào instance

1. Chọn volume → **Actions** → **Attach volume**.
2. **Instance**: chọn instance (⚠️ **chỉ hiện instance trong CÙNG AZ**).
3. **Device name**: `/dev/sdf` (Linux sẽ thấy là `/dev/xvdf`).
4. **Attach volume** → state chuyển sang **`in-use`**.

### 4. Chứng minh ràng buộc AZ ⭐

- Tạo một volume ở AZ **khác** (ví dụ `eu-west-3b`), rồi thử attach vào instance ở `eu-west-3a`
  → **instance đó KHÔNG xuất hiện trong danh sách** → chứng minh **EBS bị khóa vào một AZ**.

### 5. Dùng volume mới trong OS

SSH vào instance:

```bash
lsblk                          # liệt kê block device: thấy xvda (root) và xvdf (volume mới)
sudo file -s /dev/xvdf         # nếu in ra "data" → chưa có filesystem
sudo mkfs -t xfs /dev/xvdf     # tạo filesystem XFS
sudo mkdir /data               # tạo thư mục mount
sudo mount /dev/xvdf /data     # mount volume
df -h                          # xác nhận đã mount
echo "hello EBS" | sudo tee /data/test.txt
```

Tài liệu tham khảo (từ `code_v2025-10-27/ebs/commands.txt`):
- `https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-using-volumes.html`
- Đo hiệu năng đĩa với `fio` và `ioping`: `https://www.unixmen.com/how-to-measure-disk-performance-with-fio-and-ioping/`

### 6. Kiểm chứng Delete on Termination

1. EC2 → Instances → chọn instance → tab **Storage**.
2. Xem cột **Delete on termination**:
   - Root volume (`/dev/xvda`) → **`Yes`**
   - Volume bạn tự attach (`/dev/sdf`) → **`No`**
3. **Terminate instance** → quan sát trong **Volumes**:
   - Root volume **biến mất**
   - Volume bạn tự tạo vẫn còn với state **`available`** ⭐

### 7. Dọn dẹp

- Chọn volume `available` → **Actions** → **Delete volume**.

> ⚠️ Volume ở trạng thái `available` **vẫn bị tính tiền** — nhớ xóa.

---

## 58. EBS Snapshots

### Snapshot là gì?

- **Tạo bản backup (snapshot) của EBS volume tại một thời điểm** (point in time).
- **Không bắt buộc phải detach volume để snapshot, nhưng ĐƯỢC KHUYẾN NGHỊ** (để đảm bảo tính toàn vẹn dữ liệu).
- **Có thể copy snapshot qua AZ khác hoặc Region khác** ⭐

### Sơ đồ

```
    US-EAST-1A                              US-EAST-1B
  ┌──────────┐                            ┌──────────┐
  │   EBS    │  ──snapshot──►  EBS        │   EBS    │
  │  (50 GB) │                Snapshot ───│  (50 GB) │
  └──────────┘                  restore──►└──────────┘
```

> **Đây chính là cách di chuyển EBS volume sang AZ hoặc Region khác:** snapshot → copy → restore.

### EBS Snapshots Features ⭐⭐

Ba tính năng quan trọng (rất hay ra thi):

#### 1. EBS Snapshot Archive

- **Chuyển snapshot sang một "archive tier" RẺ HƠN 75%**.
- **Mất từ 24 đến 72 GIỜ để khôi phục (restore) từ archive** ⭐

> Dùng cho snapshot cần lưu lâu dài vì lý do tuân thủ, hiếm khi phải khôi phục.

#### 2. Recycle Bin for EBS Snapshots (Thùng rác)

- **Thiết lập rule để giữ lại các snapshot đã bị xóa**, giúp **khôi phục sau khi lỡ tay xóa nhầm** (accidental deletion).
- **Chỉ định thời gian lưu giữ (retention): từ 1 NGÀY đến 1 NĂM** ⭐

#### 3. Fast Snapshot Restore (FSR)

- **Buộc khởi tạo đầy đủ (full initialization) snapshot để KHÔNG có độ trễ ở lần dùng đầu tiên**.
- **RẤT ĐẮT ($$$)** ⭐

> **Vì sao cần FSR?** Bình thường, volume khôi phục từ snapshot được nạp dữ liệu **lazy loading** từ S3 → lần đọc đầu tiên của mỗi block bị chậm. FSR nạp trước toàn bộ.

### Sơ đồ tính năng

```
                    ┌──── archive ────►  EBS Snapshot Archive  (rẻ hơn 75%,
                    │                                            restore 24-72h)
    EBS Snapshot ───┤
                    │
                    └──── delete ─────►  Recycle Bin  (giữ 1 ngày → 1 năm)
```

### Bảng tổng hợp 3 tính năng ⭐

| Tính năng | Mục đích | Con số cần nhớ |
|-----------|----------|----------------|
| **Snapshot Archive** | Lưu trữ lâu dài, giá rẻ | **Rẻ hơn 75%**, restore **24–72 giờ** |
| **Recycle Bin** | Chống xóa nhầm | Retention **1 ngày → 1 năm** |
| **Fast Snapshot Restore (FSR)** | Không có latency lần đọc đầu | **Rất đắt ($$$)** |

### Đặc điểm khác của Snapshot

- Snapshot là **incremental** — chỉ lưu **các block đã thay đổi** so với snapshot trước.
- Snapshot được lưu trong **Amazon S3** (nhưng bạn không thấy trực tiếp trong S3 console).
- **Snapshot của volume đã mã hóa thì cũng được mã hóa**.
- Có thể **chia sẻ snapshot** với account AWS khác (nếu chưa mã hóa, hoặc dùng CMK và chia sẻ key).

---

## 59. EBS Snapshots - Hands On

### 1. Tạo snapshot

1. EC2 → **Volumes** → chọn volume → **Actions** → **Create snapshot**.
2. **Description**: ví dụ `DemoSnapshot`.
3. **Create snapshot**.
4. EC2 → menu trái → **Snapshots** → thấy snapshot với:
   - **Status**: `pending` → `completed`
   - **Progress**: `0%` → `100%`
   - **Volume size**, **Encryption**, **Started**

### 2. Copy snapshot sang Region khác ⭐

1. Chọn snapshot → **Actions** → **Copy snapshot**.
2. **Destination Region**: chọn Region khác (ví dụ `us-east-1`).
3. Tùy chọn: bật **Encryption**.
4. **Copy snapshot**.
5. Chuyển Region ở góc trên → vào **Snapshots** → thấy bản copy.

> **Đây là cách chuẩn để backup dữ liệu qua Region — phục vụ Disaster Recovery.**

### 3. Khôi phục snapshot thành volume mới

1. Chọn snapshot → **Actions** → **Create volume from snapshot**.
2. Chọn:
   - **Volume type**, **Size** (có thể tăng, **không thể giảm**)
   - **Availability Zone**: ⭐ **có thể chọn AZ KHÁC với volume gốc** → đây là cách "di chuyển" EBS qua AZ
3. **Create volume** → attach vào instance như bình thường.

### 4. Recycle Bin

1. Console → tìm **Recycle Bin** (thuộc EC2).
2. **Create retention rule**:
   - **Retention rule name**
   - **Resource type**: `EBS snapshots`
   - **Apply to all resources** hoặc theo **tag**
   - **Retention period**: ví dụ `7 days`
3. Sau khi có rule, snapshot bị xóa sẽ vào Recycle Bin và **khôi phục được** trong thời hạn đã đặt.

### 5. Snapshot Archive

1. Chọn snapshot → **Actions** → **Archive snapshot**.
2. Cảnh báo: **restore mất 24–72 giờ**.
3. Storage tier chuyển từ `standard` → `archive`.
4. Để khôi phục: **Actions** → **Restore snapshot from archive** (chọn tạm thời hoặc vĩnh viễn).

> ⚠️ Snapshot phải **≥ 24 giờ tuổi** mới archive được, và có phí tối thiểu 90 ngày.

### 6. Dọn dẹp

- Xóa snapshot ở cả Region gốc và Region đã copy.
- Xóa các volume tạo từ snapshot.

---

## 60. AMI Overview

### AMI là gì?

- **AMI = Amazon Machine Image**.
- **AMI là bản tùy biến (customization) của một EC2 instance**.
- Bạn **thêm phần mềm, cấu hình, hệ điều hành, monitoring… của riêng mình**.
- **Thời gian boot / cấu hình NHANH HƠN** vì **toàn bộ phần mềm đã được đóng gói sẵn (pre-packaged)** ⭐
- **AMI được xây dựng cho một REGION cụ thể** — và **có thể copy qua các Region khác** ⭐

### Ba nguồn AMI để launch EC2 instance ⭐

| Nguồn | Mô tả |
|-------|-------|
| **A Public AMI** | **AWS cung cấp** (Amazon Linux, Ubuntu, Windows Server…) |
| **Your own AMI** | **Bạn tự tạo và tự bảo trì** |
| **An AWS Marketplace AMI** | **AMI do người khác tạo** (và có thể **bán**) |

> **Mẹo kiếm tiền:** Bạn có thể tạo AMI chuyên dụng và **bán trên AWS Marketplace**.

### AMI Process (từ một EC2 instance) ⭐

Quy trình 4 bước (nguyên văn slide):

1. **Start an EC2 instance and customize it** — khởi chạy instance và tùy biến nó
2. **Stop the instance (for data integrity)** — **DỪNG instance để đảm bảo toàn vẹn dữ liệu** ⭐
3. **Build an AMI — this will also create EBS snapshots** — tạo AMI, **thao tác này cũng tạo ra EBS snapshots**
4. **Launch instances from other AMIs** — khởi chạy instance mới từ AMI đó

### Sơ đồ

```
    US-EAST-1A                                    US-EAST-1B
  ┌───────────┐                                 ┌───────────┐
  │    EC2    │ ──Create AMI──►  Custom AMI ──► │    EC2    │
  │(tùy biến) │                              Launch        │
  └───────────┘                              from AMI      │
                                                └───────────┘
```

> **Điểm quan trọng:** AMI cho phép launch instance ở **AZ khác** (và Region khác nếu copy AMI) với **cấu hình y hệt**.

### Vì sao AMI quan trọng?

| Lợi ích | Giải thích |
|---------|-----------|
| **Boot nhanh** | Không phải chạy User Data cài đặt phần mềm mỗi lần |
| **Nhất quán** | Mọi instance đều giống hệt nhau — tránh "configuration drift" |
| **Nền tảng cho Auto Scaling** | Launch Template dùng AMI → scale nhanh |
| **Golden image** | Chuẩn hóa bảo mật, patch, agent giám sát |

### AMI vs EBS Snapshot ⭐

| | **AMI** | **EBS Snapshot** |
|---|---|---|
| Bản chất | **Template để launch instance** | **Backup của một volume** |
| Bao gồm | Thông tin OS + **1 hoặc nhiều snapshot** + metadata (kiến trúc, kernel, block device mapping) | Chỉ dữ liệu của **một volume** |
| Dùng để | **Launch EC2 instance** | **Tạo lại volume** |
| Phạm vi | Region (copy được) | Region (copy được) |

---

## 61. AMI Hands On

### 1. Tùy biến instance nguồn

1. Launch một EC2 instance với User Data cài Apache (như phần 5).
2. SSH vào, cài thêm phần mềm / thay đổi cấu hình:
   ```bash
   sudo yum install -y htop
   echo "<h1>My Custom AMI</h1>" | sudo tee /var/www/html/index.html
   ```

### 2. Tạo AMI

1. EC2 → Instances → chọn instance → **Actions** → **Image and templates** → **Create image**.
2. Điền:
   - **Image name**: ví dụ `my-custom-ami`
   - **Image description**
   - **No reboot**:
     - **Không tick** (mặc định) → AWS **reboot instance** để đảm bảo **toàn vẹn dữ liệu** ⭐ **(khuyến nghị)**
     - **Tick** → không reboot, nhưng **rủi ro dữ liệu không nhất quán**
3. **Create image**.
4. EC2 → menu trái → **Images** → **AMIs** → thấy AMI với status `pending` → `available`.
5. Vào **Snapshots** → thấy **snapshot được tạo tự động** kèm theo AMI ⭐

### 3. Launch instance từ AMI tùy chỉnh

1. *Launch instances* → mục **Application and OS Images** → tab **My AMIs** → chọn AMI vừa tạo.
2. Chọn instance type, key pair, security group (**nhớ mở port 80**).
3. **Launch instance**.
4. Truy cập `http://<public-ip>` → thấy ngay `My Custom AMI` — **không cần User Data, không cần cài lại gì**.
5. SSH vào → chạy `htop` → phần mềm đã có sẵn.

### 4. Copy AMI sang Region khác

1. **AMIs** → chọn AMI → **Actions** → **Copy AMI**.
2. **Destination Region**: chọn Region đích.
3. **Copy AMI** → AMI xuất hiện ở Region mới (kèm snapshot mới).

### 5. Chia sẻ AMI

- **Actions** → **Edit AMI permissions**:
  - **Private** (mặc định) → thêm **AWS Account ID** cụ thể để chia sẻ
  - **Public** → ai cũng dùng được ⚠️ (cẩn thận rò rỉ dữ liệu nhạy cảm)

### 6. Dọn dẹp ⚠️

Xóa AMI đúng cách gồm **2 bước**:

1. **AMIs** → chọn → **Actions** → **Deregister AMI**.
2. **Snapshots** → xóa **snapshot còn sót lại** (deregister AMI **không tự xóa snapshot**) ⭐
3. Terminate các instance đã launch từ AMI.

> ⚠️ Đây là lỗi tốn tiền phổ biến: deregister AMI nhưng quên xóa snapshot → **vẫn bị tính phí lưu trữ**.

---

## 62. EC2 Instance Store

### Vấn đề với EBS

- **EBS volume là ổ đĩa MẠNG với hiệu năng tốt nhưng "GIỚI HẠN" (limited)**.
- **Nếu cần ổ đĩa PHẦN CỨNG hiệu năng cao → dùng EC2 Instance Store** ⭐

### EC2 Instance Store là gì?

Đặc điểm (nguyên văn slide):

- **Hiệu năng I/O tốt hơn** (Better I/O performance)
- **EC2 Instance Store MẤT TOÀN BỘ dữ liệu nếu instance bị STOP (ephemeral — phù du)** ⭐⭐
- **Tốt cho: buffer / cache / scratch data / temporary content** (dữ liệu đệm, tạm)
- **Rủi ro mất dữ liệu nếu phần cứng hỏng**
- **Backup và Replication là TRÁCH NHIỆM CỦA BẠN** ⭐

### Vì sao nhanh hơn?

```
EBS:            EC2 ──── MẠNG ────► EBS Volume  (có latency mạng)
Instance Store: EC2 ──── gắn TRỰC TIẾP ───► Ổ đĩa vật lý trên máy chủ  (không qua mạng)
```

→ **Very high IOPS** (IOPS rất cao — theo slide, có thể lên tới hàng triệu IOPS).

### Bảng so sánh EBS vs Instance Store ⭐⭐

| | **EBS Volume** | **EC2 Instance Store** |
|---|---|---|
| Bản chất | **Ổ đĩa mạng** (network drive) | **Ổ đĩa vật lý gắn trực tiếp** trên host |
| Hiệu năng I/O | Tốt nhưng **giới hạn** | ⭐ **Rất cao (very high IOPS)** |
| Độ bền dữ liệu | ✅ **Bền vững (persistent)** | ❌ **Ephemeral — mất khi stop/terminate** |
| Khi **Stop** instance | ✅ Dữ liệu còn | ❌ **Dữ liệu MẤT** |
| Khi **Reboot** instance | ✅ Dữ liệu còn | ✅ Dữ liệu còn |
| Khi phần cứng hỏng | ✅ Dữ liệu còn | ❌ **Dữ liệu MẤT** |
| Detach/Attach sang instance khác | ✅ Được | ❌ **Không được** |
| Snapshot | ✅ Có | ❌ **Không** |
| Thay đổi dung lượng | ✅ Tăng được | ❌ Cố định theo instance type |
| Backup & Replication | AWS lo phần độ bền | ⭐ **Trách nhiệm của BẠN** |
| Use case | Root volume, database, dữ liệu quan trọng | **Buffer, cache, scratch data, temporary content** |

### Mẹo nhớ ⭐

| Từ khóa trong đề | Đáp án |
|------------------|--------|
| "**maximum / highest IOPS**", "hiệu năng đĩa cao nhất" | **EC2 Instance Store** |
| "**ephemeral**", "temporary", "buffer/cache/scratch" | **EC2 Instance Store** |
| "dữ liệu phải tồn tại sau khi stop instance" | **EBS** |
| "cần backup/snapshot" | **EBS** |
| "mất dữ liệu khi stop" | **EC2 Instance Store** |

> **Lưu ý:** Instance Store **chỉ có trên một số instance type** (có chữ **`d`** trong tên như `i3`, `c5d`, `m5d`, `r5d`).

---

## 63. EBS Volume Types

### 6 loại EBS Volume ⭐

| Loại | Công nghệ | Mô tả (nguyên văn slide) |
|------|-----------|--------------------------|
| **gp2 / gp3** | **SSD** | **General purpose SSD** — cân bằng giữa **giá và hiệu năng** cho nhiều loại workload |
| **io1 / io2 Block Express** | **SSD** | **SSD hiệu năng CAO NHẤT** cho workload **mission-critical, low-latency hoặc high-throughput** |
| **st1** | **HDD** | **HDD chi phí thấp** cho workload **truy cập thường xuyên, thiên về throughput** |
| **sc1** | **HDD** | **HDD chi phí THẤP NHẤT** cho workload **ít được truy cập** |

### Ba đặc tính của EBS Volume

**EBS Volumes được đặc trưng bởi: `Size | Throughput | IOPS (I/O Operations Per Second)`**

> **Khi phân vân, luôn tra tài liệu AWS — nó rất tốt!** (lời khuyên từ slide)

### ⭐⭐ QUY TẮC BOOT VOLUME (rất hay ra thi)

> **CHỈ `gp2`/`gp3` và `io1`/`io2 Block Express` mới dùng được làm BOOT VOLUME.**
> → **HDD (`st1`, `sc1`) KHÔNG THỂ làm boot volume.**

---

### 1️⃣ General Purpose SSD (gp2 / gp3)

**Đặc điểm:**
- **Lưu trữ tiết kiệm chi phí, độ trễ thấp**.
- **Use cases**: **System boot volumes**, **Virtual desktops**, **Development and test environments**.
- **Dung lượng: 1 GiB – 16 TiB**.

**gp3 (thế hệ mới):**
- **Baseline 3,000 IOPS** và **throughput 125 MiB/s**.
- **Có thể tăng IOPS lên tới 16,000 và throughput lên tới 1,000 MiB/s MỘT CÁCH ĐỘC LẬP** ⭐
  (tăng IOPS mà không cần tăng dung lượng)

**gp2 (thế hệ cũ):**
- Volume **gp2 nhỏ có thể "burst" IOPS lên 3,000**.
- **Kích thước volume và IOPS bị RÀNG BUỘC với nhau**, **max IOPS = 16,000** ⭐
- **3 IOPS mỗi GiB** → **ở 5,334 GiB thì đạt max IOPS** (5,334 × 3 = 16,002 ≈ 16,000)

> **So sánh gp2 vs gp3:** gp3 **rẻ hơn ~20%** và cho phép **tách rời IOPS khỏi dung lượng** → **gp3 luôn là lựa chọn tốt hơn gp2**.

---

### 2️⃣ Provisioned IOPS (PIOPS) SSD (io1 / io2 Block Express)

**Use cases:**
- **Ứng dụng kinh doanh quan trọng cần hiệu năng IOPS bền vững (sustained)**.
- **Hoặc ứng dụng cần HƠN 16,000 IOPS** ⭐ (vượt giới hạn của gp2/gp3)
- **Rất tốt cho workload DATABASE** (nhạy cảm với hiệu năng và tính nhất quán của lưu trữ)

**io1 (4 GiB – 16 TiB):**
- **Max PIOPS: 64,000 cho instance EC2 dòng Nitro** & **32,000 cho các dòng khác** ⭐
- **Có thể tăng PIOPS ĐỘC LẬP với dung lượng lưu trữ**

**io2 Block Express (4 GiB – 64 TiB):**
- **Độ trễ dưới mili-giây (sub-millisecond latency)** ⭐
- **Max PIOPS: 256,000** với tỉ lệ **IOPS:GiB = 1,000:1**

**⭐ Hỗ trợ EBS Multi-Attach** (xem bài 64)

---

### 3️⃣ Hard Disk Drives (HDD) — st1 / sc1

**Đặc điểm chung:**
- ⭐ **KHÔNG THỂ làm boot volume**
- **Dung lượng: 125 GiB – 16 TiB**

**Throughput Optimized HDD (st1):**
- **Use cases**: **Big Data, Data Warehouses, Log Processing**
- **Max throughput 500 MiB/s** — **max IOPS 500** ⭐

**Cold HDD (sc1):**
- **Cho dữ liệu ÍT được truy cập (infrequently accessed)**
- **Kịch bản mà CHI PHÍ THẤP NHẤT là quan trọng**
- **Max throughput 250 MiB/s** — **max IOPS 250** ⭐

---

### Bảng tổng hợp EBS Volume Types ⭐⭐

| | **gp3** | **gp2** | **io1** | **io2 Block Express** | **st1** | **sc1** |
|---|---|---|---|---|---|---|
| **Loại** | SSD | SSD | SSD | SSD | HDD | HDD |
| **Dung lượng** | 1 GiB–16 TiB | 1 GiB–16 TiB | 4 GiB–16 TiB | 4 GiB–64 TiB | 125 GiB–16 TiB | 125 GiB–16 TiB |
| **Max IOPS** | **16,000** | **16,000** | **64,000** (Nitro) / 32,000 | **256,000** | **500** | **250** |
| **Max Throughput** | **1,000 MiB/s** | 250 MiB/s | 1,000 MiB/s | 4,000 MiB/s | **500 MiB/s** | **250 MiB/s** |
| **Boot volume** | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ |
| **Multi-Attach** | ❌ | ❌ | ✅ | ✅ | ❌ | ❌ |
| **IOPS độc lập với size** | ✅ | ❌ (3 IOPS/GiB) | ✅ | ✅ | — | — |
| **Use case chính** | Boot, dev/test, virtual desktop | (cũ, nên chuyển gp3) | Database, mission-critical | Database cực lớn, sub-ms latency | Big Data, Data Warehouse, Log Processing | Archive, truy cập hiếm, **rẻ nhất** |

Tài liệu tham chiếu (từ slide):
`https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html#solid-state-drives`

### Cheat sheet chọn volume type ⭐

| Từ khóa trong đề | Đáp án |
|------------------|--------|
| Boot volume, general purpose, cân bằng giá/hiệu năng | **gp3** (hoặc gp2) |
| Cần **> 16,000 IOPS** | **io1 / io2** |
| **Database** mission-critical, cần IOPS bền vững | **io1 / io2** |
| **Sub-millisecond latency**, cần tới 256,000 IOPS | **io2 Block Express** |
| Gắn **một volume vào nhiều instance** | **io1 / io2** (Multi-Attach) |
| **Big Data, Data Warehouse, Log Processing** (throughput cao, giá rẻ) | **st1** |
| **Chi phí thấp nhất**, dữ liệu ít truy cập | **sc1** |
| Cần làm boot volume nhưng đề đưa st1/sc1 | ❌ **Loại ngay** — HDD không boot được |

---

## 64. EBS Multi-Attach

### Khái niệm ⭐

**EBS Multi-Attach — chỉ dành cho dòng `io1` / `io2`**

- **Attach CÙNG MỘT EBS volume vào NHIỀU EC2 instance trong CÙNG MỘT AZ** ⭐
- **Mỗi instance có toàn quyền đọc & ghi (full read & write)** trên volume hiệu năng cao đó.

### Sơ đồ

```
┌────────────── Availability Zone 1 ──────────────┐
│                                                  │
│   ┌─────┐    ┌─────┐    ┌─────┐                 │
│   │ EC2 │    │ EC2 │    │ EC2 │                 │
│   └──┬──┘    └──┬──┘    └──┬──┘                 │
│      └──────────┼──────────┘                    │
│                 ▼                                │
│      ┌────────────────────────┐                 │
│      │ io2 volume             │                 │
│      │ with Multi-Attach      │                 │
│      └────────────────────────┘                 │
└──────────────────────────────────────────────────┘
```

### Use case (nguyên văn slide)

- **Đạt được tính sẵn sàng cao hơn cho các ứng dụng Linux dạng cluster** (ví dụ: **Teradata**).
- **Ứng dụng PHẢI TỰ QUẢN LÝ các thao tác ghi đồng thời (concurrent write operations)** ⭐

### Ràng buộc quan trọng ⭐⭐

| Ràng buộc | Chi tiết |
|-----------|----------|
| **Loại volume** | **CHỈ `io1` / `io2`** |
| **Số instance tối đa** | **Tối đa 16 EC2 instance cùng lúc** ⭐ |
| **Phạm vi AZ** | **PHẢI trong CÙNG MỘT Availability Zone** ⭐ |
| **File system** | **PHẢI dùng file system nhận biết cluster (cluster-aware)** — **KHÔNG dùng XFS, EXT4, v.v.** ⭐⭐ |

> **Bẫy thi kinh điển:** Đề nói "attach một EBS volume vào nhiều instance với XFS/EXT4" → **SAI**. Phải dùng cluster-aware file system như **GFS2**, **OCFS2**, hoặc **Veritas CFS**.

### Multi-Attach vs EFS ⭐

| | **EBS Multi-Attach** | **Amazon EFS** |
|---|---|---|
| Số instance | **Tối đa 16** | **Hàng nghìn** |
| Phạm vi | **1 AZ duy nhất** | **Nhiều AZ** |
| Loại lưu trữ | Block storage | File storage (NFS) |
| File system | Phải cluster-aware | Tự động (NFS) |
| Ứng dụng phải quản lý concurrency | ✅ **Có** | ❌ Không |
| Use case | Cluster Linux (Teradata) | Chia sẻ file, WordPress, CMS |

---

## 65. EBS Encryption

### Khi tạo một EBS volume được mã hóa, bạn nhận được ⭐

- **Dữ liệu at rest (khi lưu trữ) được mã hóa bên trong volume**
- **Toàn bộ dữ liệu in flight (đang di chuyển) giữa instance và volume được mã hóa**
- **Tất cả snapshot đều được mã hóa**
- **Tất cả volume được tạo từ snapshot đó cũng được mã hóa**

### Đặc điểm quan trọng ⭐

- **Mã hóa và giải mã được xử lý HOÀN TOÀN TRONG SUỐT (transparently) — bạn KHÔNG phải làm gì cả** ⭐
- **Mã hóa có tác động TỐI THIỂU tới độ trễ (minimal impact on latency)**
- **EBS Encryption tận dụng key từ KMS (thuật toán AES-256)** ⭐
- **Copy một snapshot CHƯA mã hóa cho phép bật mã hóa** ⭐
- **Snapshot của volume đã mã hóa thì cũng được mã hóa**

### ⭐⭐ Quy trình mã hóa một EBS volume CHƯA được mã hóa

Đây là quy trình **rất hay ra thi** (nguyên văn 4 bước từ slide):

```
Bước 1: Create an EBS snapshot of the volume
        → Tạo snapshot của volume gốc (chưa mã hóa)

Bước 2: Encrypt the EBS snapshot (using COPY)
        → MÃ HÓA snapshot bằng cách COPY nó (tick Encrypt khi copy)

Bước 3: Create new EBS volume from the snapshot
        (the volume will also be encrypted)
        → Tạo volume MỚI từ snapshot đã mã hóa → volume này tự động được mã hóa

Bước 4: Now you can attach the encrypted volume to the original instance
        → Attach volume đã mã hóa vào instance ban đầu
```

> **Điểm mấu chốt:** Bạn **KHÔNG THỂ mã hóa trực tiếp** một volume đang tồn tại. Phải đi đường vòng qua **snapshot → copy có mã hóa → tạo volume mới**.

### Sơ đồ quy trình

```
   EBS Volume            EBS Snapshot         EBS Snapshot        EBS Volume
  (chưa mã hóa)  ──►    (chưa mã hóa)  ──►    (ĐÃ mã hóa)  ──►   (ĐÃ mã hóa)
                snapshot            COPY + Encrypt        create volume
                                                                     │
                                                                     ▼
                                                          attach vào instance
```

### Bảng tổng hợp hành vi mã hóa ⭐

| Thao tác | Kết quả |
|----------|---------|
| Snapshot của volume **đã mã hóa** | **Đã mã hóa** |
| Snapshot của volume **chưa mã hóa** | **Chưa mã hóa** |
| Volume tạo từ snapshot **đã mã hóa** | **Đã mã hóa** |
| **Copy** snapshot chưa mã hóa (tick Encrypt) | ⭐ **Trở thành đã mã hóa** |
| Copy snapshot đã mã hóa | Vẫn mã hóa (có thể đổi KMS key) |
| Chia sẻ snapshot **đã mã hóa** | Phải chia sẻ cả **KMS key** |

### Ghi chú thêm

- Có thể bật **"Encryption by default"** ở cấp Region: EC2 → **EBS encryption** → **Manage** → tick **Enable**.
- Key mặc định là `aws/ebs` (AWS managed key); có thể dùng **Customer Managed Key (CMK)** để kiểm soát chặt hơn.
- **Root volume** cũng mã hóa được ngay lúc launch instance.

---

## 66. Amazon EFS

### EFS là gì?

- **EFS = Elastic File System**.
- Là **NFS (network file system) được quản lý (managed)**, **có thể mount lên NHIỀU EC2 instance cùng lúc** ⭐
- **EFS hoạt động với các EC2 instance ở NHIỀU AZ (multi-AZ)** ⭐
- **Highly available, scalable, ĐẮT (gấp 3 lần gp2), trả tiền theo mức dùng (pay per use)** ⭐

### Sơ đồ

```
     us-east-1a          us-east-1b          us-east-1c
   ┌────────────┐      ┌────────────┐      ┌────────────┐
   │ EC2        │      │ EC2        │      │ EC2        │
   │ Instances  │      │ Instances  │      │ Instances  │
   └─────┬──────┘      └─────┬──────┘      └─────┬──────┘
         │                   │                   │
         └───────────┬───────┴───────────────────┘
                     ▼
            ┌─────────────────┐
            │ Security Group  │
            ├─────────────────┤
            │  EFS FileSystem │
            └─────────────────┘
```

### Đặc điểm chi tiết ⭐

- **Use cases**: **content management, web serving, data sharing, WordPress**
- **Dùng giao thức NFSv4.1** ⭐
- **Dùng Security Group để kiểm soát truy cập vào EFS** ⭐
- **Chỉ tương thích với AMI dựa trên Linux — KHÔNG hỗ trợ Windows** ⭐⭐
- **Mã hóa at rest bằng KMS**
- Là **POSIX file system (~Linux)** với **API file chuẩn**
- **File system TỰ ĐỘNG SCALE, trả tiền theo mức dùng, KHÔNG CẦN capacity planning!** ⭐

> **Bẫy thi:** Câu hỏi có Windows instance cần shared file system → **KHÔNG phải EFS**, mà là **Amazon FSx for Windows File Server**.

---

### EFS – Performance & Storage Classes

#### EFS Scale

- **Hàng nghìn (1000s) NFS client đồng thời**, **throughput 10 GB+/s**
- **Tự động phát triển tới quy mô Petabyte**

#### Performance Mode (đặt tại thời điểm TẠO EFS) ⭐

| Mode | Mô tả |
|------|-------|
| **General Purpose** (mặc định) | Cho use case **nhạy cảm với độ trễ** (web server, CMS…) |
| **Max I/O** | **Độ trễ cao hơn**, **throughput cao hơn**, **song song cao (highly parallel)** — cho **big data, media processing** |

> ⚠️ **Performance Mode CHỈ đặt được lúc tạo EFS**, không đổi sau đó.

#### Throughput Mode ⭐

| Mode | Mô tả |
|------|-------|
| **Bursting** | **1 TB = 50 MiB/s** + **burst lên tới 100 MiB/s** |
| **Provisioned** | **Đặt throughput bất kể dung lượng lưu trữ** — ví dụ **1 GiB/s cho 1 TB storage** |
| **Elastic** | **Tự động scale throughput lên/xuống theo workload**<br>• **Tới 3 GiB/s cho đọc** và **1 GiB/s cho ghi**<br>• Dùng cho **workload không đoán trước được (unpredictable)** ⭐ |

---

### EFS – Storage Classes ⭐

#### Storage Tiers (tính năng lifecycle management — chuyển file sau N ngày)

| Tier | Mô tả |
|------|-------|
| **Standard** | Cho **file được truy cập thường xuyên** |
| **Infrequent Access (EFS-IA)** | **Có phí khi truy xuất file**, nhưng **giá lưu trữ thấp hơn** ⭐ |
| **Archive** | Cho **dữ liệu hiếm khi truy cập** (vài lần mỗi năm), **rẻ hơn 50%** ⭐ |

- **Triển khai lifecycle policies để tự động chuyển file giữa các storage tier**.

```
                    no access for 60 days
    EFS Standard  ──────────────────────►  EFS IA
                     (Lifecycle Policy)
```

#### Availability and Durability ⭐

| Class | Mô tả |
|-------|-------|
| **Standard** | **Multi-AZ** — **tuyệt vời cho PRODUCTION** ⭐ |
| **One Zone** | **Một AZ duy nhất** — **tuyệt vời cho DEV**, **backup được bật mặc định**, tương thích với IA (**EFS One Zone-IA**) |

- **Tiết kiệm hơn 90% chi phí** khi kết hợp One Zone + IA ⭐

### Cheat sheet EFS ⭐

| Từ khóa trong đề | Đáp án |
|------------------|--------|
| "shared file system cho nhiều EC2 across AZ" | **EFS** |
| "**WordPress**", "content management", "web serving" | **EFS** |
| "NFS", "POSIX" | **EFS** |
| "Windows file share" | ❌ **KHÔNG phải EFS** → **FSx for Windows** |
| "workload không đoán trước được" | **Elastic Throughput** |
| "big data, media processing, highly parallel" | **Max I/O Performance Mode** |
| "môi trường dev, tiết kiệm chi phí" | **EFS One Zone** |
| "file ít truy cập, tự động chuyển sau N ngày" | **EFS-IA + Lifecycle Policy** |

---

## 67. Amazon EFS - Hands On

### Script chính thức của khóa học

Nội dung file `code_v2025-10-27/efs/efs.sh`:

```bash
# on both instances:
sudo yum install -y amazon-efs-utils
sudo mkdir /efs
sudo mount -t efs fs-yourid:/ /efs

# you can now write files into /efs and they'll be available on both your ec2 instances!
```

Giải thích:

| Lệnh | Ý nghĩa |
|------|---------|
| `sudo yum install -y amazon-efs-utils` | Cài bộ công cụ EFS của Amazon (cung cấp loại mount `efs`) |
| `sudo mkdir /efs` | Tạo thư mục làm điểm mount |
| `sudo mount -t efs fs-yourid:/ /efs` | Mount EFS file system (**thay `fs-yourid` bằng File system ID thật của bạn**) |

---

### Bước 1 — Tạo EFS File System

1. Console → tìm **EFS** → **Create file system**.
2. Bấm **Customize** (thay vì Create nhanh) để thấy đầy đủ tùy chọn:
   - **Name**: `my-efs`
   - **Storage class**: **Standard (Multi-AZ)** hoặc **One Zone**
   - **Automatic backups**: bật/tắt
   - **Lifecycle management**:
     - *Transition into IA*: `30 days since last access`
     - *Transition into Archive*
     - *Transition out of IA*: `On first access`
   - **Encryption**: bật (khuyến nghị)
   - **Performance mode**: `General Purpose`
   - **Throughput mode**: `Elastic` / `Bursting` / `Provisioned`
3. **Next** → **Network access**:
   - **VPC**: chọn default VPC
   - **Mount targets**: EFS tạo **một mount target cho MỖI AZ** ⭐
   - **Security groups**: ⚠️ **tạo/chọn một SG riêng cho EFS**
4. **Next** → bỏ qua File system policy → **Create**.

### Bước 2 — Cấu hình Security Group ⭐⭐

Đây là bước **hay bị sai nhất**:

1. EC2 → **Security Groups** → tạo SG mới, ví dụ `efs-demo-sg`.
2. **Inbound rule**:
   - **Type**: **NFS**
   - **Port**: **2049** ⭐
   - **Source**: **Security Group của EC2 instances** (tham chiếu SG, không dùng IP)
3. Quay lại EFS → **Network** → **Manage** → gán `efs-demo-sg` cho **tất cả mount target**.

> **Nếu quên bước này:** lệnh `mount` sẽ **treo rồi timeout** — đúng như quy tắc "timeout = lỗi Security Group".

### Bước 3 — Launch 2 EC2 instance ở 2 AZ khác nhau

1. Launch instance thứ nhất ở `eu-west-3a`, instance thứ hai ở `eu-west-3b`.
2. Cả hai dùng chung một Security Group (SG này được EFS SG tham chiếu).
3. Mở port **22 (SSH)** để kết nối.

> Mẹo: trong wizard launch instance có mục **File systems** → **Add shared file system** → chọn EFS → AWS **tự động cấu hình SG và mount** giúp bạn.

### Bước 4 — Mount EFS trên cả hai instance

SSH vào **từng** instance và chạy:

```bash
sudo yum install -y amazon-efs-utils
sudo mkdir /efs
sudo mount -t efs fs-0123456789abcdef0:/ /efs
df -h                      # xác nhận: thấy dòng 127.0.0.1:/ mount vào /efs
```

Lấy **File system ID** ở EFS Console (dạng `fs-xxxxxxxxx`), hoặc bấm **Attach** để AWS hiển thị sẵn lệnh mount.

### Bước 5 — Kiểm chứng chia sẻ dữ liệu ⭐

**Trên instance 1:**

```bash
cd /efs
sudo touch hello-from-instance-1.txt
echo "Xin chao tu instance 1" | sudo tee /efs/message.txt
ls -l /efs
```

**Trên instance 2:**

```bash
ls -l /efs                  # ⭐ THẤY NGAY file vừa tạo từ instance 1
cat /efs/message.txt        # Xin chao tu instance 1
sudo touch hello-from-instance-2.txt
```

**Quay lại instance 1:**

```bash
ls -l /efs                  # thấy cả file của instance 2
```

> **Đây chính là điểm khác biệt cốt lõi so với EBS:** cùng một file system, **hai instance ở HAI AZ KHÁC NHAU**, đọc/ghi đồng thời.

### Bước 6 — Mount tự động sau reboot (tùy chọn)

```bash
echo 'fs-0123456789abcdef0:/ /efs efs _netdev,tls 0 0' | sudo tee -a /etc/fstab
sudo mount -a               # kiểm tra cấu hình fstab không lỗi
```

### Lỗi thường gặp

| Lỗi | Nguyên nhân |
|-----|-------------|
| `mount` **treo rồi timeout** | **Security Group của EFS chưa mở port 2049** cho SG của EC2 |
| `mount: unknown filesystem type 'efs'` | Chưa cài `amazon-efs-utils` |
| `Failed to resolve` | Instance ở AZ **không có mount target** |
| Mount được nhưng **không ghi được** | Quyền thư mục — dùng `sudo` hoặc `sudo chown ec2-user:ec2-user /efs` |

---

## 68. EFS vs EBS

### EBS — Elastic Block Storage (từ slide)

**EBS volumes…**

- Phục vụ **MỘT instance** (**ngoại trừ multi-attach `io1`/`io2`**)
- **Bị khóa ở cấp độ Availability Zone (AZ)**
- **`gp2`: IO tăng nếu dung lượng đĩa tăng**
- **`gp3` & `io1`: có thể tăng IO ĐỘC LẬP**

**Để migrate một EBS volume qua AZ khác:**

1. **Take a snapshot** (tạo snapshot)
2. **Restore the snapshot to another AZ** (khôi phục snapshot sang AZ khác)

**Lưu ý:** **EBS backup dùng IO và bạn KHÔNG NÊN chạy chúng khi ứng dụng đang xử lý nhiều traffic** ⭐

**Root EBS Volume** của instance **bị xóa mặc định** khi EC2 instance bị terminate (**có thể tắt hành vi này**).

```
  Availability Zone 1              Availability Zone 2
     ┌──────┐                          ┌──────┐
     │ EBS  │──snapshot──► EBS ────────│ EBS  │
     └──────┘             Snapshot     └──────┘
                              restore──►
```

---

### EFS — Elastic File System (từ slide)

- **Mount được lên HÀNG TRĂM instance XUYÊN QUA CÁC AZ** ⭐
- **EFS chia sẻ file website (WordPress)**
- **CHỈ dành cho Linux instance (POSIX)** ⭐
- **EFS có mức giá CAO HƠN EBS** ⭐
- **Có thể tận dụng Storage Tiers để tiết kiệm chi phí**

```
  Availability Zone 1              Availability Zone 2
    ┌─────────┐                       ┌─────────┐
    │  Linux  │                       │  Linux  │
    └────┬────┘                       └────┬────┘
         │                                 │
    ┌────▼─────┐                      ┌────▼─────┐
    │   EFS    │                      │   EFS    │
    │  Mount   │                      │  Mount   │
    │  Target  │                      │  Target  │
    └────┬─────┘                      └────┬─────┘
         └──────────────┬─────────────────┘
                        ▼
                     ┌─────┐
                     │ EFS │
                     └─────┘
```

> **Nhớ: EFS vs EBS vs Instance Store** — đây là bộ ba luôn được so sánh trong đề thi.

---

### ⭐⭐ Bảng so sánh 3 loại lưu trữ (BẢNG QUAN TRỌNG NHẤT PHẦN NÀY)

| | **EBS** | **EFS** | **EC2 Instance Store** |
|---|---|---|---|
| **Loại lưu trữ** | **Block storage** | **File storage (NFS)** | **Block storage (vật lý)** |
| **Số instance gắn được** | **1** (trừ Multi-Attach io1/io2: tối đa 16) | **Hàng trăm / hàng nghìn** | **1** (gắn cứng vào host) |
| **Phạm vi** | **1 AZ** | **Multi-AZ** ⭐ | **1 host vật lý** |
| **Hệ điều hành** | Linux & Windows | **CHỈ Linux (POSIX)** ⭐ | Linux & Windows |
| **Độ bền** | ✅ Bền vững | ✅ Bền vững | ❌ **Ephemeral** |
| **Dữ liệu khi Stop** | ✅ Còn | ✅ Còn | ❌ **MẤT** |
| **Tự động scale dung lượng** | ❌ (phải tự tăng) | ✅ **Tự động** ⭐ | ❌ Cố định |
| **Capacity planning** | ✅ Cần | ❌ **Không cần** | ✅ Cần (theo instance type) |
| **Hiệu năng** | Tốt (giới hạn) | Tốt, scale được | ⭐ **Cao nhất (very high IOPS)** |
| **Giá** | Trung bình | **Cao (~3x gp2)** ⭐ | Đã tính trong giá instance |
| **Snapshot / Backup** | ✅ EBS Snapshot | ✅ AWS Backup | ❌ **Tự lo** |
| **Di chuyển qua AZ** | Snapshot → restore | Có sẵn multi-AZ | ❌ Không |
| **Use case** | Root volume, database, dữ liệu 1 instance | **Shared files, WordPress, CMS, data sharing** | **Buffer, cache, scratch, temp** |

### Cheat sheet chọn loại lưu trữ ⭐

| Từ khóa trong đề | Đáp án |
|------------------|--------|
| "nhiều EC2 **across AZ** cùng đọc/ghi một file system" | **EFS** |
| "**WordPress**", "shared content", "CMS" | **EFS** |
| "**Windows** file share" | **FSx for Windows** (không phải EFS) |
| "database trên EC2", "root volume" | **EBS** |
| "cần **IOPS cao nhất**", "temporary / scratch data" | **Instance Store** |
| "dữ liệu **mất khi stop instance**" | **Instance Store** |
| "gắn 1 volume vào **nhiều instance trong CÙNG AZ**" | **EBS Multi-Attach (io1/io2)** |
| "**không cần capacity planning**", "tự động scale" | **EFS** |
| "chi phí thấp nhất cho shared file system" | **EFS One Zone-IA** |

---

## 69. EBS & EFS - Section Cleanup

Bài này giảng viên hướng dẫn **dọn dẹp toàn bộ tài nguyên** đã tạo trong phần này để **không phát sinh chi phí**.

### Checklist dọn dẹp đầy đủ ⚠️

#### 1. Terminate EC2 instances

```
EC2 → Instances → chọn tất cả → Instance state → Terminate instance
```

#### 2. Xóa EFS File System

```
EFS → chọn file system → Delete → gõ File system ID để xác nhận
```

> EFS tính tiền theo **GB-tháng** — dù không dùng vẫn mất phí.

#### 3. Xóa EBS Volumes còn sót

```
EC2 → Volumes → lọc State = "available" → Actions → Delete volume
```

> ⚠️ Volume `available` (không gắn vào instance nào) **vẫn bị tính tiền đầy đủ**.

#### 4. Xóa EBS Snapshots

```
EC2 → Snapshots → chọn → Actions → Delete snapshot
```

Nhớ kiểm tra **cả các Region khác** nếu bạn đã copy snapshot sang Region khác.

#### 5. Deregister AMIs + xóa snapshot của AMI ⭐

```
Bước 1: EC2 → AMIs → Actions → Deregister AMI
Bước 2: EC2 → Snapshots → xóa snapshot còn lại của AMI đó
```

> ⚠️ **Deregister AMI KHÔNG tự xóa snapshot** — đây là lỗi tốn tiền phổ biến nhất.

#### 6. Release Elastic IPs (nếu có từ phần trước)

```
EC2 → Elastic IPs → Actions → Release Elastic IP addresses
```

#### 7. Xóa Security Groups tự tạo (tùy chọn)

Security Group không tốn tiền, nhưng nên dọn cho gọn.

### Bảng kiểm tra nhanh

| Tài nguyên | Vị trí trong Console | Tính tiền khi không dùng? |
|------------|----------------------|---------------------------|
| EC2 Instance | EC2 → Instances | ✅ Có (khi running) |
| EBS Volume | EC2 → Volumes | ✅ **Có, kể cả `available`** |
| EBS Snapshot | EC2 → Snapshots | ✅ Có |
| AMI (snapshot đằng sau) | EC2 → AMIs + Snapshots | ✅ **Có** |
| EFS File System | EFS | ✅ Có |
| Elastic IP | EC2 → Elastic IPs | ✅ **Có, đặc biệt khi không gắn** |
| Security Group | EC2 → Security Groups | ❌ Không |
| Placement Group | EC2 → Placement Groups | ❌ Không |

### Kiểm tra cuối cùng

1. **Billing Dashboard** → **Bills** → xem chi phí phát sinh theo dịch vụ.
2. Kiểm tra **tất cả Region** bạn đã từng dùng (dùng **Resource Explorer** hoặc **Tag Editor** để quét toàn bộ Region).
3. Đợi 24h rồi kiểm tra lại Budget alert đã thiết lập ở bài 31.

---

## Trắc nghiệm 4: EC2 Data Management Quiz

### Các điểm dễ bị bẫy

| Câu hỏi thường gặp | Đáp án đúng |
|--------------------|-------------|
| EBS volume gắn được vào mấy instance? | **Một** (trừ Multi-Attach `io1`/`io2`) |
| EBS bị khóa vào đâu? | **Một Availability Zone** |
| Làm sao chuyển EBS volume sang AZ khác? | **Snapshot → restore ở AZ mới** |
| Root volume mặc định khi terminate instance? | **BỊ XÓA** (Delete on Termination = enabled) |
| Volume phụ (non-root) mặc định khi terminate? | **KHÔNG bị xóa** (disabled) |
| Có phải detach volume mới snapshot được không? | **Không bắt buộc**, nhưng **được khuyến nghị** |
| Snapshot copy được qua Region không? | **CÓ** |
| Snapshot Archive rẻ hơn bao nhiêu? | **75%** |
| Restore từ Snapshot Archive mất bao lâu? | **24–72 giờ** |
| Recycle Bin giữ snapshot được bao lâu? | **1 ngày đến 1 năm** |
| Tính năng nào loại bỏ latency lần đọc đầu tiên? | **Fast Snapshot Restore (FSR)** — rất đắt |
| AMI được xây cho phạm vi nào? | **Một Region** (copy qua Region được) |
| 3 nguồn AMI? | **Public AMI (AWS)**, **Your own AMI**, **AWS Marketplace AMI** |
| Trước khi tạo AMI nên làm gì? | **Stop instance** (để đảm bảo toàn vẹn dữ liệu) |
| Tạo AMI có tạo ra gì kèm theo? | **EBS snapshots** |
| Deregister AMI có xóa snapshot không? | **KHÔNG** — phải xóa thủ công |
| Instance Store mất dữ liệu khi nào? | **Khi STOP instance** (ephemeral) hoặc phần cứng hỏng |
| Instance Store dùng cho gì? | **Buffer / cache / scratch data / temporary content** |
| Ai chịu trách nhiệm backup Instance Store? | **BẠN** |
| Cần IOPS cực cao trên đĩa cục bộ? | **EC2 Instance Store** |
| EBS có mấy loại volume? | **6** (gp2, gp3, io1, io2 Block Express, st1, sc1) |
| Loại nào dùng làm **boot volume** được? | **CHỈ gp2/gp3 và io1/io2 Block Express** |
| `st1` và `sc1` làm boot volume được không? | **KHÔNG** |
| gp2 max IOPS? | **16,000** (3 IOPS/GiB, đạt max ở **5,334 GiB**) |
| gp3 baseline? | **3,000 IOPS** + **125 MiB/s**, tăng độc lập tới **16,000 IOPS** / **1,000 MiB/s** |
| io1 max PIOPS? | **64,000** (Nitro) / **32,000** (khác) |
| io2 Block Express max PIOPS + latency? | **256,000 IOPS**, **sub-millisecond latency**, tới **64 TiB** |
| Cần **hơn 16,000 IOPS** → chọn gì? | **io1 / io2** |
| Big Data, Data Warehouse, Log Processing? | **st1** (Throughput Optimized HDD) |
| Chi phí thấp nhất, dữ liệu ít truy cập? | **sc1** (Cold HDD) |
| st1 max throughput / IOPS? | **500 MiB/s / 500 IOPS** |
| sc1 max throughput / IOPS? | **250 MiB/s / 250 IOPS** |
| EBS Multi-Attach hỗ trợ loại nào? | **CHỈ io1 / io2** |
| Multi-Attach tối đa bao nhiêu instance? | **16** |
| Multi-Attach có qua được nhiều AZ không? | **KHÔNG** — phải cùng 1 AZ |
| Multi-Attach cần file system nào? | **Cluster-aware** — **KHÔNG dùng XFS, EXT4** |
| EBS Encryption dùng gì? | **KMS, AES-256** |
| Mã hóa volume chưa mã hóa — làm sao? | **Snapshot → COPY có mã hóa → tạo volume mới → attach** |
| Snapshot của volume đã mã hóa? | **Cũng được mã hóa** |
| EFS dùng giao thức gì? | **NFSv4.1** |
| EFS port bao nhiêu? | **2049** |
| EFS hỗ trợ Windows không? | **KHÔNG** — chỉ Linux (POSIX) |
| EFS đắt gấp mấy lần gp2? | **~3 lần** |
| EFS có cần capacity planning không? | **KHÔNG** — tự động scale |
| Performance mode cho big data / highly parallel? | **Max I/O** |
| Throughput mode cho workload không đoán trước? | **Elastic** |
| EFS Archive tier rẻ hơn bao nhiêu? | **50%** |
| EFS One Zone tiết kiệm bao nhiêu? | **Hơn 90%** |
| EFS Standard vs One Zone? | **Standard = Multi-AZ (prod)**, **One Zone = 1 AZ (dev)** |
| EFS mount được lên bao nhiêu instance? | **Hàng trăm / hàng nghìn, xuyên AZ** |

### Checklist tự kiểm tra trước khi làm quiz

- [ ] Nắm chắc bảng so sánh **EBS vs EFS vs Instance Store**
- [ ] Nhớ **EBS = 1 AZ**, **EFS = multi-AZ**, **Instance Store = ephemeral**
- [ ] Thuộc **6 loại EBS volume** + **quy tắc boot volume (chỉ SSD)**
- [ ] Nhớ các con số: **gp2/gp3 = 16,000 IOPS**, **io1 = 64,000**, **io2 BE = 256,000**, **st1 = 500**, **sc1 = 250**
- [ ] Thuộc **quy trình 4 bước mã hóa volume chưa mã hóa**
- [ ] Nhớ **Multi-Attach: io1/io2, 16 instance, cùng AZ, cluster-aware FS**
- [ ] Nhớ **3 tính năng Snapshot**: Archive (75%, 24–72h), Recycle Bin (1 ngày–1 năm), FSR ($$$)
- [ ] Nhớ **EFS chỉ Linux**, dùng **NFSv4.1**, port **2049**, Security Group kiểm soát truy cập
- [ ] Nhớ **Delete on Termination**: root = enabled, non-root = disabled

---

## Thuật ngữ Anh — Việt

| Tiếng Anh | Tiếng Việt |
|-----------|-----------|
| Elastic Block Store (EBS) | Lưu trữ khối co giãn |
| Block storage / File storage | Lưu trữ dạng khối / dạng tệp |
| Network drive | Ổ đĩa mạng |
| Persist data | Lưu dữ liệu bền vững |
| Provisioned capacity | Dung lượng được cấp phát |
| Attach / Detach / Mount | Gắn / Tháo / Gắn kết (vào hệ thống tệp) |
| Delete on Termination | Xóa khi hủy instance |
| Snapshot | Bản chụp (backup tại một thời điểm) |
| Incremental | Tăng dần (chỉ lưu phần thay đổi) |
| Snapshot Archive | Lưu trữ nguội snapshot |
| Recycle Bin | Thùng rác (khôi phục snapshot xóa nhầm) |
| Fast Snapshot Restore (FSR) | Khôi phục snapshot tức thì |
| Retention period | Thời hạn lưu giữ |
| Amazon Machine Image (AMI) | Ảnh máy (template khởi tạo instance) |
| Pre-packaged | Đóng gói sẵn |
| Deregister AMI | Hủy đăng ký AMI |
| Golden image | Ảnh chuẩn dùng chung |
| Instance Store | Ổ đĩa cục bộ của instance |
| Ephemeral | Phù du, tạm thời (mất khi stop) |
| Scratch data | Dữ liệu nháp/tạm |
| IOPS (I/O Operations Per Second) | Số thao tác đọc/ghi mỗi giây |
| Throughput | Thông lượng (MiB/s) |
| Provisioned IOPS (PIOPS) | IOPS được cấp phát cố định |
| Boot volume | Ổ đĩa khởi động |
| Burst | Bùng phát hiệu năng tạm thời |
| Sub-millisecond latency | Độ trễ dưới mili-giây |
| Multi-Attach | Gắn một volume vào nhiều instance |
| Cluster-aware file system | Hệ thống tệp nhận biết cụm |
| Concurrent write | Ghi đồng thời |
| Encryption at rest / in flight | Mã hóa khi lưu trữ / khi truyền |
| KMS (Key Management Service) | Dịch vụ quản lý khóa |
| Transparently | Trong suốt (người dùng không phải làm gì) |
| Elastic File System (EFS) | Hệ thống tệp co giãn |
| NFS (Network File System) | Hệ thống tệp qua mạng |
| Mount target | Điểm mount của EFS trong mỗi AZ |
| POSIX | Chuẩn giao diện hệ điều hành kiểu Unix |
| Capacity planning | Hoạch định dung lượng |
| Performance mode / Throughput mode | Chế độ hiệu năng / chế độ thông lượng |
| Storage tier / Storage class | Tầng lưu trữ / lớp lưu trữ |
| Infrequent Access (IA) | Truy cập không thường xuyên |
| Lifecycle policy | Chính sách vòng đời (tự chuyển tầng) |

---

*Ghi chú: các phần Hands On được tóm tắt lại các bước thao tác chính trên AWS Console. Giao diện Console có thể thay đổi theo thời gian — logic và khái niệm vẫn giữ nguyên. Đặc biệt chú ý bài 69 (Section Cleanup): EBS volume `available`, snapshot và AMI đều bị tính tiền dù không sử dụng.*
