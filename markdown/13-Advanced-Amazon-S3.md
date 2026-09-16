# Phần 13 — Advanced Amazon S3

> Khóa học: *Ultimate AWS Certified Solutions Architect Associate 2026* (Stéphane Maarek) — SAA-C03
> Nguồn tham chiếu: `AWS Certified Solutions Architect Slides v48.pdf` (phần "Amazon S3 – Advanced")

---

## Mục lục

| # | Bài giảng | Thời lượng | Loại |
|---|-----------|-----------|------|
| 143 | [S3 Lifecycle Rules (with S3 Analytics)](#143-s3-lifecycle-rules-with-s3-analytics) | 4 phút | Video |
| 144 | [S3 Lifecycle Rules - Hands On](#144-s3-lifecycle-rules---hands-on) | 2 phút | Video |
| 145 | [S3 Requester Pays](#145-s3-requester-pays) | 2 phút | Video |
| 146 | [S3 Event Notifications](#146-s3-event-notifications) | 4 phút | Video |
| 147 | [S3 Event Notifications - Hands On](#147-s3-event-notifications---hands-on) | 6 phút | Video |
| 148 | [S3 Performance](#148-s3-performance) | 5 phút | Video |
| 149 | [S3 Batch Operations](#149-s3-batch-operations) | 2 phút | Video |
| 150 | [S3 Storage Lens](#150-s3-storage-lens) | 6 phút | Video |
| — | [Trắc nghiệm 10: Amazon S3 Advanced Quiz](#trắc-nghiệm-10-amazon-s3-advanced-quiz) | — | Quiz |

---

## 143. S3 Lifecycle Rules (with S3 Analytics)

### Amazon S3 — Moving between Storage Classes ⭐

- ⭐ **Bạn có thể CHUYỂN ĐỔI (transition) object giữa các storage class**
- ⭐ **Với object ít được truy cập → chuyển sang Standard IA**
- ⭐ **Với object archive mà bạn không cần truy cập nhanh → chuyển sang Glacier hoặc Glacier Deep Archive**
- ⭐⭐ **Việc di chuyển object có thể được TỰ ĐỘNG HÓA bằng Lifecycle Rules**

### Sơ đồ luồng chuyển đổi ⭐

```
   Standard
      ├──► Standard IA
      ├──► Intelligent Tiering
      ├──► One-Zone IA
      ├──► Glacier Instant Retrieval
      ├──► Glacier Flexible Retrieval
      └──► Glacier Deep Archive
              (càng xuống dưới càng RẺ, truy xuất càng CHẬM)
```

---

### ⭐⭐⭐ Amazon S3 — Lifecycle Rules

#### 🔹 Transition Actions ⭐

> **Cấu hình object để CHUYỂN sang một storage class khác**

- ⭐ **Chuyển object sang Standard IA class 60 NGÀY sau khi tạo**
- ⭐ **Chuyển sang Glacier để archive sau 6 THÁNG**

#### 🔹 Expiration Actions ⭐⭐

> **Cấu hình object để HẾT HẠN (XÓA) sau một thời gian**

- ⭐ **File access log có thể được set để XÓA sau 365 NGÀY**
- ⭐⭐ **Có thể dùng để XÓA CÁC VERSION CŨ của file** (nếu versioning được bật)
- ⭐⭐ **Có thể dùng để XÓA CÁC MULTI-PART UPLOAD CHƯA HOÀN THÀNH (incomplete Multi-Part uploads)**

#### 🔹 Phạm vi áp dụng Rule ⭐

| Cách lọc | Ví dụ |
|----------|-------|
| ⭐ **Theo PREFIX** | `s3://mybucket/mp3/*` |
| ⭐ **Theo TAGS của object** | `Department: Finance` |

> ⭐⭐ **Ghi nhớ:** Lifecycle Rules là câu trả lời cho **mọi** câu hỏi kiểu *"tự động chuyển/xóa object sau N ngày"*.

---

### ⭐⭐⭐ Scenario 1 — Thumbnails (đề bài mẫu)

> **Đề bài:** Ứng dụng của bạn trên EC2 tạo **thumbnail ảnh** sau khi ảnh đại diện được upload lên S3.
> Các thumbnail này **có thể dễ dàng tạo lại**, và **chỉ cần giữ 60 ngày**.
> **Ảnh gốc** phải **truy xuất NGAY LẬP TỨC trong 60 ngày đó**, và **sau đó user có thể chờ TỚI 6 GIỜ**.
> **Bạn thiết kế thế nào?**

### ✅ Đáp án:

| Đối tượng | Giải pháp |
|-----------|-----------|
| ⭐ **S3 source images (ảnh gốc)** | **Để ở Standard**, với **lifecycle configuration chuyển sang GLACIER sau 60 ngày** |
| ⭐⭐ **S3 thumbnails** | **Để ở ONE-ZONE IA** (vì **tạo lại được**), với **lifecycle configuration EXPIRE (xóa) chúng sau 60 ngày** |

> ⭐ **Điểm mấu chốt:** "có thể tạo lại được" → **One-Zone IA**. "Chờ được tới 6 giờ" → **Glacier Flexible Retrieval** (Standard 3–5 giờ, Bulk 5–12 giờ).

---

### ⭐⭐⭐ Scenario 2 — Recover deleted objects (đề bài mẫu)

> **Đề bài:** Một quy định trong công ty nói rằng bạn phải có thể **khôi phục các object S3 đã xóa NGAY LẬP TỨC trong 30 ngày**, dù điều này **hiếm khi xảy ra**.
> Sau thời gian đó, và **tới 365 ngày**, các object đã xóa phải **khôi phục được trong vòng 48 GIỜ**.

### ✅ Đáp án — 3 bước:

| # | Bước |
|---|------|
| 1 | ⭐⭐ **Bật S3 VERSIONING** để có các version của object, nhờ đó **"object đã xóa" thực chất bị CHE bởi một "delete marker"** và **khôi phục được** |
| 2 | ⭐ **Chuyển các "NONCURRENT VERSIONS" của object sang Standard IA** |
| 3 | ⭐⭐ **Sau đó chuyển các "noncurrent versions" sang GLACIER DEEP ARCHIVE** |

> ⭐ **Vì sao Deep Archive?** Vì yêu cầu *"khôi phục trong 48 giờ"* → khớp chính xác với **Glacier Deep Archive — Bulk (48 giờ)**.

### ⭐⭐ Khái niệm cần phân biệt

| Thuật ngữ | Ý nghĩa |
|-----------|---------|
| ⭐ **Current version** | **Version MỚI NHẤT** của object |
| ⭐ **Noncurrent version** | **Các version CŨ** (bị version mới đè lên) |

> Lifecycle Rules **có thể cấu hình RIÊNG** cho current version và noncurrent version — đây là điểm rất hay ra thi.

---

### ⭐⭐ Amazon S3 Analytics — Storage Class Analysis

- ⭐ **Giúp bạn QUYẾT ĐỊNH KHI NÀO chuyển object sang storage class phù hợp**
- ⭐⭐ **Đưa ra khuyến nghị cho Standard và Standard IA**
- ⚠️ ⭐⭐ **KHÔNG hoạt động cho One-Zone IA hoặc Glacier**
- ⭐ **Report được cập nhật HÀNG NGÀY**
- ⭐⭐ **Mất 24 ĐẾN 48 GIỜ để bắt đầu thấy dữ liệu phân tích**
- ⭐⭐ **Là BƯỚC ĐẦU TIÊN tốt để xây dựng (hoặc cải thiện) Lifecycle Rules!**

### Ví dụ báo cáo `.csv` (từ slide)

| Date | StorageClass | ObjectAge |
|------|--------------|-----------|
| 8/22/2022 | STANDARD | 000-014 |
| 8/25/2022 | STANDARD | 030-044 |
| 9/6/2022 | STANDARD | 120-149 |

```
   S3 Bucket ──► S3 Analytics ──► .csv report ──► xây dựng Lifecycle Rules
```

> ⭐⭐ **Mẹo thi:** Đề hỏi *"làm sao biết nên đặt Lifecycle Rule như thế nào?"* → **S3 Analytics — Storage Class Analysis**. Nhớ kèm 2 giới hạn: **chỉ Standard/Standard-IA**, **24–48 giờ mới có dữ liệu**.

---

## 144. S3 Lifecycle Rules - Hands On

### Bước 1 — Tạo Lifecycle Rule

1. Bucket → tab **Management** → **Lifecycle rules** → **Create lifecycle rule**.
2. **Lifecycle rule configuration**:
   - **Lifecycle rule name**: `DemoRule`
   - ⭐ **Choose a rule scope**:
     - **Limit the scope to specific prefixes or tags** → nhập **prefix** (ví dụ `mp3/`) hoặc **tag** (ví dụ `Department = Finance`)
     - **Apply to all objects in the bucket** (phải tick xác nhận)

### Bước 2 — Chọn Lifecycle rule actions ⭐⭐

Console cho phép chọn **5 loại action**:

| # | Action | Ý nghĩa |
|---|--------|---------|
| 1 | ⭐ **Move current versions of objects between storage classes** | Chuyển version hiện tại |
| 2 | ⭐ **Move noncurrent versions of objects between storage classes** | Chuyển version cũ |
| 3 | ⭐ **Expire current versions of objects** | Xóa version hiện tại (tạo delete marker) |
| 4 | ⭐ **Permanently delete noncurrent versions of objects** | Xóa vĩnh viễn version cũ |
| 5 | ⭐⭐ **Delete expired object delete markers or incomplete multipart uploads** | Dọn delete marker mồ côi & multipart dở dang |

### Bước 3 — Cấu hình chi tiết ⭐

**Ví dụ cho action 1 (Move current versions):**

| Storage class transitions | Days after object creation |
|---------------------------|---------------------------|
| **Standard-IA** | `30` |
| **Glacier Flexible Retrieval** | `90` |
| **Glacier Deep Archive** | `180` |

⚠️ Console sẽ cảnh báo về **minimum storage duration** và **chi phí transition request**.

**Ví dụ cho action 5:**
- ⭐ **Delete incomplete multipart uploads**: sau `7` ngày

> ⭐⭐ **Đây là best practice tiết kiệm chi phí:** multipart upload dở dang **vẫn chiếm dung lượng và tính tiền** nhưng **không hiện trong danh sách object**.

### Bước 4 — Review & Create

1. Console hiển thị **Timeline summary** — sơ đồ trực quan object sẽ đi qua các class nào theo thời gian.
2. **Create rule**.

### Bước 5 — Xem S3 Analytics ⭐

1. Bucket → tab **Metrics** → mục **Storage Class Analysis** → **Create analytics configuration**.
2. Cấu hình name, scope (prefix/tag), và **Export CSV** tới một bucket đích.
3. ⚠️ ⭐ **Chờ 24–48 giờ** mới có dữ liệu.

### Lưu ý ⚠️

| Lưu ý | Chi tiết |
|-------|----------|
| ⭐ **Lifecycle chạy 1 lần/ngày** | Không tức thì — chạy theo lịch nội bộ của S3 (UTC) |
| ⭐ **Không tính phí cho việc chạy rule** | Nhưng **transition request có phí** |
| ⭐ **Min storage duration vẫn áp dụng** | Chuyển sang IA rồi xóa sớm vẫn tính đủ 30 ngày |

---

## 145. S3 Requester Pays

### Mô hình thanh toán mặc định ⭐

> ⭐ **Nhìn chung, CHỦ BUCKET (bucket owner) trả TẤT CẢ chi phí lưu trữ Amazon S3 và chi phí truyền dữ liệu liên quan tới bucket của họ**

```
   ┌──────── Standard Bucket ────────┐
   │  Owner $$ Storage Cost           │
   │  Owner $$ Networking Cost        │──download──► Requester (trả $0)
   └──────────────────────────────────┘
```

---

### ⭐⭐ Với Requester Pays buckets

> ⭐⭐ **NGƯỜI YÊU CẦU (requester) THAY VÌ chủ bucket sẽ trả chi phí của REQUEST và việc TẢI DỮ LIỆU từ bucket**

```
   ┌──────── Requester Pays Bucket ────────┐
   │  Owner     $$ Storage Cost             │
   │  Requester $$ Networking Cost          │──download──► Requester (TRẢ TIỀN)
   └────────────────────────────────────────┘
```

### Bảng phân chia chi phí ⭐⭐

| Chi phí | **Standard Bucket** | **Requester Pays Bucket** |
|---------|---------------------|---------------------------|
| ⭐ **Storage Cost** (lưu trữ) | **Owner** | ⭐ **Owner** (vẫn là chủ bucket!) |
| ⭐ **Networking Cost** (tải xuống) | **Owner** | ⭐⭐ **REQUESTER** |

> ⭐⭐ **Bẫy thi:** Requester Pays **CHỈ chuyển phí NETWORKING/REQUEST** sang người tải. **Phí LƯU TRỮ vẫn do chủ bucket trả.**

### Use case ⭐

- ⭐⭐ **Hữu ích khi bạn muốn CHIA SẺ CÁC TẬP DỮ LIỆU LỚN với các account khác**

### ⚠️⭐⭐ Ràng buộc quan trọng:

> **Requester PHẢI được xác thực trong AWS (authenticated) — KHÔNG THỂ ẩn danh (anonymous)**

Lý do: AWS phải biết **tính tiền cho ai** → không thể cho phép truy cập ẩn danh.

### Cách bật

```
Bucket → Properties → Requester pays → Edit → Enable
```

Khi gọi API, requester phải thêm header:
```
x-amz-request-payer: requester
```

---

## 146. S3 Event Notifications

### Các loại event ⭐⭐

> **`S3:ObjectCreated`, `S3:ObjectRemoved`, `S3:ObjectRestore`, `S3:Replication`…**

### Đặc điểm ⭐

- ⭐ **Có thể LỌC THEO TÊN OBJECT** (object name filtering) — ví dụ **`*.jpg`**
- ⭐⭐ **Use case: tạo THUMBNAIL của ảnh được upload lên S3**
- ⭐ **Có thể tạo BAO NHIÊU "S3 events" tùy thích**
- ⭐⭐ **S3 event notifications thường gửi event TRONG VÀI GIÂY nhưng ĐÔI KHI có thể mất MỘT PHÚT hoặc LÂU HƠN**

### 3 đích đến truyền thống ⭐⭐

```
                    ┌──► SNS
   Amazon S3 ──events──► SQS
                    └──► Lambda Function
```

| Đích | Dùng khi |
|------|----------|
| ⭐ **SNS** | Gửi thông báo tới nhiều subscriber (email, HTTP…) |
| ⭐ **SQS** | Đưa vào hàng đợi để xử lý nền |
| ⭐ **Lambda Function** | Chạy code xử lý ngay (ví dụ tạo thumbnail) |

---

### ⭐⭐⭐ S3 Event Notifications — IAM Permissions

Đây là điểm **cực kỳ hay ra thi**:

```
                    ┌──► SNS     ← ⭐ SNS Resource (Access) Policy
   Amazon S3 ──events──► SQS     ← ⭐ SQS Resource (Access) Policy
                    └──► Lambda  ← ⭐ Lambda Resource Policy
```

### ⚠️⭐⭐ Quy tắc vàng:

> **KHÔNG dùng IAM Role gắn vào S3!**
> **Phải gắn RESOURCE (ACCESS) POLICY vào chính SNS / SQS / Lambda để CHO PHÉP S3 gửi event tới chúng.**

| Đích | Cần cấu hình |
|------|-------------|
| **SNS** | ⭐ **SNS Resource (Access) Policy** |
| **SQS** | ⭐ **SQS Resource (Access) Policy** |
| **Lambda** | ⭐ **Lambda Resource Policy** |

> **Bẫy thi kinh điển:** Đề hỏi *"S3 không gửi được event tới SQS, sửa thế nào?"* → **Sửa SQS Access Policy** (không phải IAM Role của S3).

---

### ⭐⭐ S3 Event Notifications với Amazon EventBridge

```
   Amazon S3 bucket ──All events──► Amazon EventBridge ──rules──► ⭐ Over 18 AWS services
                                                                     as destinations
```

### Ba nhóm lợi ích ⭐⭐

| Lợi ích | Chi tiết |
|---------|----------|
| ⭐⭐ **Advanced filtering options** | **Với JSON rules** (metadata, **object size**, name...) |
| ⭐⭐ **Multiple Destinations** | Ví dụ: **Step Functions, Kinesis Streams / Firehose**… |
| ⭐⭐ **EventBridge Capabilities** | **Archive, Replay Events, Reliable delivery** |

### So sánh 2 cách ⭐

| | **S3 Event Notifications (truyền thống)** | **Qua EventBridge** |
|---|---|---|
| **Đích** | ⭐ **3: SNS, SQS, Lambda** | ⭐⭐ **Hơn 18 dịch vụ AWS** |
| **Lọc** | Theo **prefix / suffix** tên object | ⭐ **JSON rules nâng cao** (metadata, **kích thước**, tên…) |
| **Archive / Replay events** | ❌ Không | ✅ ⭐ **Có** |
| **Reliable delivery** | Cơ bản | ✅ ⭐ **Có** |
| **Gửi tới nhiều đích cùng lúc** | Hạn chế | ✅ ⭐ **Có** |

> ⭐ **Mẹo thi:** Đề nhắc **"lọc theo kích thước object"**, **"replay events"**, **"gửi tới Step Functions/Kinesis"** → **EventBridge**.

---

## 147. S3 Event Notifications - Hands On

### Bước 1 — Tạo SQS Queue

1. Console → **SQS** → **Create queue**:
   - **Type**: `Standard`
   - **Name**: `DemoS3Notification`
2. **Create queue**.

### Bước 2 — ⭐⭐ Sửa Access Policy của SQS (bước quan trọng nhất)

1. Chọn queue → tab **Access policy** → **Edit**.
2. Dán policy cho phép S3 gửi message:

```json
{
  "Version": "2012-10-17",
  "Id": "example-ID",
  "Statement": [
    {
      "Sid": "example-statement-ID",
      "Effect": "Allow",
      "Principal": {
        "Service": "s3.amazonaws.com"
      },
      "Action": "SQS:SendMessage",
      "Resource": "arn:aws:sqs:eu-west-1:123456789012:DemoS3Notification",
      "Condition": {
        "ArnLike": {
          "aws:SourceArn": "arn:aws:s3:::my-bucket-name"
        },
        "StringEquals": {
          "aws:SourceAccount": "123456789012"
        }
      }
    }
  ]
}
```

> ⚠️ ⭐⭐ **Nếu bỏ qua bước này**, khi tạo Event Notification S3 sẽ báo lỗi:
> `Unable to validate the following destination configurations`

### Bước 3 — Tạo Event Notification trên S3 ⭐

1. Bucket → tab **Properties** → mục ⭐ **Event notifications** → **Create event notification**.
2. Cấu hình:
   - **Event name**: `DemoEventNotification`
   - ⭐ **Prefix / Suffix** (tùy chọn): ví dụ suffix `.jpg` để chỉ bắt file ảnh
   - ⭐ **Event types**: tick **All object create events** (`s3:ObjectCreated:*`)
   - ⭐ **Destination**: **SQS queue** → chọn `DemoS3Notification`
3. **Save changes**.

### Bước 4 — Kiểm chứng ⭐

1. Upload một file vào bucket.
2. SQS → chọn queue → **Send and receive messages** → **Poll for messages**.
3. ✅ Thấy message JSON chứa thông tin event:

```json
{
  "Records": [{
    "eventVersion": "2.1",
    "eventSource": "aws:s3",
    "awsRegion": "eu-west-1",
    "eventName": "ObjectCreated:Put",
    "s3": {
      "bucket": { "name": "my-bucket-name" },
      "object": { "key": "coffee.jpg", "size": 12345 }
    }
  }]
}
```

### Bước 5 — Bật EventBridge ⭐

1. Bucket → **Properties** → **Amazon EventBridge** → **Edit**.
2. ⭐ **Send notifications to Amazon EventBridge for all events in this bucket**: **On** → **Save**.
3. Console → **EventBridge** → **Rules** → **Create rule**:
   - **Event source**: `AWS events`
   - **Event pattern**: `AWS services` → `Simple Storage Service (S3)` → chọn event type
   - ⭐ Xem danh sách **Targets** — hơn 18 dịch vụ (Lambda, Step Functions, Kinesis, SNS, SQS, ECS…)

### Dọn dẹp ⚠️

- Xóa Event notification, EventBridge rule, và SQS queue.

---

## 148. S3 Performance

### ⭐⭐⭐ S3 — Baseline Performance (con số PHẢI thuộc)

- ⭐ **Amazon S3 TỰ ĐỘNG SCALE tới tốc độ request cao, độ trễ 100-200 ms**
- ⭐⭐ **Ứng dụng của bạn có thể đạt ÍT NHẤT:**
  - ⭐⭐ **3,500 PUT/COPY/POST/DELETE** mỗi giây **TRÊN MỖI PREFIX** trong bucket
  - ⭐⭐ **5,500 GET/HEAD** mỗi giây **TRÊN MỖI PREFIX** trong bucket
- ⭐⭐ **KHÔNG CÓ GIỚI HẠN về số lượng prefix trong một bucket**

### ⭐⭐ Ví dụ về prefix (object path => prefix)

| Object path | Prefix |
|-------------|--------|
| `bucket/folder1/sub1/file` | ⭐ `/folder1/sub1/` |
| `bucket/folder1/sub2/file` | ⭐ `/folder1/sub2/` |
| `bucket/1/file` | ⭐ `/1/` |
| `bucket/2/file` | ⭐ `/2/` |

### ⭐⭐⭐ Phép tính kinh điển:

> **Nếu bạn TRẢI ĐỀU các lần đọc trên CẢ BỐN prefix, bạn có thể đạt 22,000 request/giây cho GET và HEAD**

```
   4 prefix × 5,500 GET/s = 22,000 GET/s
```

> ⭐⭐ **Đây là dạng câu hỏi tính toán hay gặp.** Nhớ: **3,500 ghi / 5,500 đọc — MỖI PREFIX**, và **prefix không giới hạn**.

---

### ⭐⭐ S3 Performance — các kỹ thuật tăng tốc

#### 1️⃣ Multi-Part upload ⭐⭐

- ⭐⭐ **ĐƯỢC KHUYẾN NGHỊ cho file > 100MB**
- ⭐⭐ **BẮT BUỘC dùng cho file > 5GB**
- ⭐ **Giúp SONG SONG HÓA việc upload (tăng tốc truyền)**

```
   BIG file ──Divide in parts──► [Part 1] [Part 2] … [Part N]
                                     └── Parallel uploads ──► Amazon S3
```

> ⭐ **Hai con số phải nhớ: khuyến nghị > 100MB, BẮT BUỘC > 5GB.**

#### 2️⃣ S3 Transfer Acceleration ⭐⭐⭐

- ⭐⭐ **Tăng tốc độ truyền bằng cách chuyển file tới một AWS EDGE LOCATION, nơi sẽ CHUYỂN TIẾP dữ liệu tới S3 bucket ở Region đích**
- ⭐ **TƯƠNG THÍCH với multi-part upload**

```
   File in USA ──Fast (public www)──► Edge Location USA ──Fast (private AWS)──► S3 Bucket Australia
```

> ⭐⭐ **Ý tưởng:** giảm tối đa quãng đường đi trên **internet công cộng (chậm)**, tối đa hóa quãng đường đi trên **mạng riêng của AWS (nhanh)**.

#### 3️⃣ S3 Byte-Range Fetches ⭐⭐

- ⭐⭐ **SONG SONG HÓA các lệnh GET bằng cách yêu cầu các DẢI BYTE (byte ranges) cụ thể**
- ⭐ **Khả năng chống lỗi tốt hơn khi có sự cố**

**Hai công dụng:**

| Công dụng | Mô tả |
|-----------|-------|
| ⭐⭐ **Tăng tốc download** | **Chia file thành nhiều phần, request SONG SONG** |
| ⭐⭐ **Lấy MỘT PHẦN dữ liệu** | **Ví dụ: chỉ lấy PHẦN ĐẦU (header) của file** |

```
   Tăng tốc download:
   File in S3 ──► [Part 1] [Part 2] … [Part N] ──► Requests in parallel

   Lấy header:
   File in S3 ──► Byte-range request for header (first XX bytes) ──► header
```

### ⭐⭐ Bảng tổng hợp 3 kỹ thuật

| Kỹ thuật | Dùng cho | Điểm nhớ |
|----------|----------|----------|
| ⭐ **Multi-Part Upload** | **UPLOAD** file lớn | **> 100MB khuyến nghị, > 5GB bắt buộc** |
| ⭐ **Transfer Acceleration** | **UPLOAD & DOWNLOAD** qua khoảng cách địa lý xa | **Đi qua Edge Location** |
| ⭐ **Byte-Range Fetches** | **DOWNLOAD** song song / lấy một phần | **Yêu cầu dải byte cụ thể** |

> ⭐ **Mẹo thi:**
> - "upload file 20GB nhanh hơn" → **Multi-Part Upload**
> - "user ở Úc upload lên bucket ở Mỹ chậm" → **Transfer Acceleration**
> - "chỉ cần đọc metadata/header của file lớn" → **Byte-Range Fetches**

---

## 149. S3 Batch Operations

### S3 Batch Operations là gì? ⭐⭐

> ⭐⭐ **Thực hiện các thao tác HÀNG LOẠT (bulk operations) trên các object S3 HIỆN CÓ với MỘT REQUEST DUY NHẤT**

### ⭐⭐ Các thao tác ví dụ

| # | Thao tác |
|---|----------|
| 1 | ⭐ **Modify object metadata & properties** (sửa metadata và thuộc tính) |
| 2 | ⭐ **Copy objects between S3 buckets** (sao chép giữa các bucket) |
| 3 | ⭐⭐ **Encrypt un-encrypted objects** (mã hóa các object chưa mã hóa) |
| 4 | ⭐ **Modify ACLs, tags** |
| 5 | ⭐ **Restore objects from S3 Glacier** |
| 6 | ⭐⭐ **Invoke Lambda function để thực hiện hành động tùy chỉnh trên MỖI object** |

### Cấu trúc một Job ⭐

> ⭐ **Một job bao gồm: DANH SÁCH OBJECT, HÀNH ĐỘNG cần thực hiện, và các THAM SỐ TÙY CHỌN**

### Những gì S3 Batch Operations quản lý giúp bạn ⭐⭐

> ⭐ **S3 Batch Operations quản lý RETRIES, theo dõi TIẾN ĐỘ, gửi THÔNG BÁO HOÀN THÀNH, tạo BÁO CÁO…**

### ⭐⭐⭐ Cách lấy danh sách object

> ⭐⭐ **Bạn có thể dùng S3 INVENTORY để lấy danh sách object và dùng ATHENA để truy vấn và lọc các object của bạn**

### Sơ đồ luồng hoàn chỉnh ⭐

```
   S3 Inventory ──► Objects List Report
                            │
                            ▼
                         Athena ──filter──► filtered list
                            │                    │
   User ──operation + parameters──────────────────┤
                                                  ▼
                                      ⭐ S3 Batch Operations
                                                  │
                                                  ▼
                                          Processed Objects
```

### ⭐⭐ Bộ ba cần nhớ

| Công cụ | Vai trò |
|---------|---------|
| ⭐ **S3 Inventory** | **Tạo DANH SÁCH object** (báo cáo định kỳ, CSV/ORC/Parquet) |
| ⭐ **Athena** | **TRUY VẤN và LỌC** danh sách đó bằng SQL |
| ⭐ **S3 Batch Operations** | **THỰC THI hành động** trên danh sách đã lọc |

> ⭐⭐ **Mẹo thi:** Đề nói *"mã hóa hàng triệu object chưa mã hóa"* hoặc *"chạy Lambda trên mọi object trong bucket"* → **S3 Batch Operations**. Nếu đề hỏi *"làm sao lấy danh sách object để xử lý?"* → **S3 Inventory + Athena**.

---

## 150. S3 Storage Lens

### S3 Storage Lens là gì? ⭐⭐

- ⭐⭐ **HIỂU, PHÂN TÍCH và TỐI ƯU HÓA storage trên TOÀN BỘ AWS ORGANIZATION**
- ⭐⭐ **Phát hiện BẤT THƯỜNG (anomalies), xác định các điểm TIẾT KIỆM CHI PHÍ, và áp dụng BEST PRACTICES về bảo vệ dữ liệu trên toàn bộ AWS Organization** (⭐ **30 ngày dữ liệu usage & activity metrics**)
- ⭐⭐ **TỔNG HỢP (aggregate) dữ liệu cho: Organization, các account cụ thể, regions, buckets, hoặc prefixes**
- ⭐ **Dashboard mặc định hoặc TỰ TẠO dashboard riêng**
- ⭐⭐ **Có thể cấu hình EXPORT metrics HÀNG NGÀY tới một S3 bucket (CSV, Parquet)**

### Sơ đồ ⭐

```
   Organization ──┐
   Accounts     ──┤                                   ┌── Summary Insights
   Regions      ──┼──► ⭐ S3 Storage Lens ──Analyze──►├── Data Protection
   Buckets      ──┘         (Aggregate)   (Dashboard)  └── Cost Efficiency
                                                              │
                                                              ▼
                                                          Optimize
```

---

### ⭐⭐ Storage Lens — Default Dashboard

- ⭐ **Trực quan hóa các insight và xu hướng đã tổng hợp cho CẢ free metrics VÀ advanced metrics**
- ⭐⭐ **Dashboard mặc định hiển thị dữ liệu MULTI-REGION và MULTI-ACCOUNT**
- ⭐ **Được Amazon S3 CẤU HÌNH SẴN (preconfigured)**
- ⭐⭐ **KHÔNG THỂ XÓA, nhưng CÓ THỂ VÔ HIỆU HÓA (disabled)**

---

### ⭐⭐⭐ Storage Lens — Metrics (7 nhóm)

#### 1️⃣ Summary Metrics ⭐

- **Insight tổng quát về S3 storage của bạn**
- **Metrics: `StorageBytes`, `ObjectCount`…**
- ⭐ **Use cases: xác định bucket và prefix TĂNG TRƯỞNG NHANH NHẤT (hoặc KHÔNG ĐƯỢC DÙNG)**

#### 2️⃣ Cost-Optimization Metrics ⭐⭐

- **Cung cấp insight để quản lý và tối ưu chi phí storage**
- **Metrics: `NonCurrentVersionStorageBytes`, `IncompleteMultipartUploadStorageBytes`…**
- ⭐⭐ **Use cases: xác định bucket có INCOMPLETE MULTIPART UPLOAD CŨ HƠN 7 NGÀY; xác định object nào có thể chuyển sang storage class rẻ hơn**

#### 3️⃣ Data-Protection Metrics ⭐⭐

- **Cung cấp insight cho các tính năng bảo vệ dữ liệu**
- **Metrics: `VersioningEnabledBucketCount`, `MFADeleteEnabledBucketCount`, `SSEKMSEnabledBucketCount`, `CrossRegionReplicationRuleCount`…**
- ⭐ **Use cases: xác định bucket KHÔNG tuân theo best practices về bảo vệ dữ liệu**

#### 4️⃣ Access-management Metrics ⭐

- **Cung cấp insight cho S3 Object Ownership**
- **Metrics: `ObjectOwnershipBucketOwnerEnforcedBucketCount`…**
- ⭐ **Use cases: xác định bucket của bạn dùng thiết lập Object Ownership nào**

#### 5️⃣ Event Metrics ⭐

- **Cung cấp insight cho S3 Event Notifications**
- **Metrics: `EventNotificationEnabledBucketCount`**
- ⭐ **Use case: xác định bucket nào đã cấu hình S3 Event Notifications**

#### 6️⃣ Performance Metrics ⭐

- **Cung cấp insight cho S3 Transfer Acceleration**
- **Metrics: `TransferAccelerationEnabledBucketCount`**
- ⭐ **Use case: xác định bucket nào đã bật S3 Transfer Acceleration**

#### 7️⃣ Activity Metrics ⭐

- **Cung cấp insight về cách storage của bạn được yêu cầu**
- **Metrics: `AllRequests`, `GetRequests`, `PutRequests`, `ListRequests`, `BytesDownloaded`…**

#### 8️⃣ Detailed Status Code Metrics ⭐

- **Cung cấp insight cho các HTTP status code**
- **Metrics: `200OKStatusCount`, `403ForbiddenErrorCount`, `404NotFoundErrorCount`…**

### Bảng tra nhanh ⭐⭐

| Nhóm metrics | Trả lời câu hỏi |
|--------------|-----------------|
| ⭐ **Summary** | "Bucket nào lớn nhất / tăng nhanh nhất?" |
| ⭐⭐ **Cost-Optimization** | "Tôi đang lãng phí tiền ở đâu?" (**version cũ, multipart dở dang**) |
| ⭐ **Data-Protection** | "Bucket nào chưa bật versioning / MFA Delete / mã hóa / replication?" |
| ⭐ **Access-management** | "Bucket dùng Object Ownership nào?" |
| ⭐ **Event** | "Bucket nào có Event Notifications?" |
| ⭐ **Performance** | "Bucket nào bật Transfer Acceleration?" |
| ⭐ **Activity** | "Ai đang request bao nhiêu?" |
| ⭐ **Status Code** | "Có bao nhiêu lỗi 403/404?" |

---

### ⭐⭐⭐ Storage Lens — Free vs. Paid

| | ⭐ **Free Metrics** | ⭐⭐ **Advanced Metrics and Recommendations** |
|---|---|---|
| **Khả dụng** | ⭐ **TỰ ĐỘNG có cho MỌI khách hàng** | **Tính phí thêm (paid)** |
| **Số metrics** | ⭐⭐ **Khoảng 28 usage metrics** | **Nhiều hơn** |
| **Thời gian lưu dữ liệu** | ⭐⭐ **14 NGÀY** | ⭐⭐ **15 THÁNG** |
| **Các nhóm metrics nâng cao** | — | ⭐ **Activity, Advanced Cost Optimization, Advanced Data Protection, Status Code** |
| **CloudWatch Publishing** | ❌ | ⭐ **Truy cập metrics trong CloudWatch KHÔNG mất phí thêm** |
| **Prefix Aggregation** | ❌ | ⭐ **Thu thập metrics ở CẤP ĐỘ PREFIX** |

> ⭐⭐ **Hai con số phải nhớ: Free = 14 NGÀY, Paid = 15 THÁNG.** Và **Free có ~28 usage metrics**.

---

## Trắc nghiệm 10: Amazon S3 Advanced Quiz

### Các điểm dễ bị bẫy

| Câu hỏi thường gặp | Đáp án đúng |
|--------------------|-------------|
| Tự động chuyển object giữa các storage class dùng gì? | ⭐ **Lifecycle Rules** |
| Lifecycle Rules có mấy loại action chính? | ⭐ **Transition Actions** và **Expiration Actions** |
| Xóa incomplete multipart uploads dùng gì? | ⭐⭐ **Lifecycle Rule (Expiration action)** |
| Lifecycle Rule lọc được theo gì? | ⭐ **Prefix** và **Object Tags** |
| Lifecycle Rule áp dụng riêng cho version cũ được không? | ✅ ⭐ **Có — noncurrent versions** |
| Ảnh thumbnail tạo lại được, giữ 60 ngày → chọn gì? | ⭐⭐ **One-Zone IA + lifecycle expire sau 60 ngày** |
| Ảnh gốc cần truy xuất ngay 60 ngày, sau đó chờ 6 giờ? | ⭐ **Standard → transition sang Glacier sau 60 ngày** |
| Khôi phục object đã xóa trong 48 giờ → dùng gì? | ⭐⭐ **Versioning + noncurrent versions → Glacier Deep Archive** |
| Công cụ nào khuyến nghị khi nào nên transition? | ⭐⭐ **S3 Analytics — Storage Class Analysis** |
| S3 Analytics hỗ trợ class nào? | ⭐⭐ **Standard và Standard IA** |
| S3 Analytics KHÔNG hỗ trợ class nào? | ⭐⭐ **One-Zone IA và Glacier** |
| S3 Analytics mất bao lâu mới có dữ liệu? | ⭐⭐ **24 đến 48 giờ** |
| S3 Analytics cập nhật bao lâu một lần? | ⭐ **Hàng ngày** |
| Mặc định ai trả phí storage và networking của bucket? | ⭐ **Bucket owner** |
| Requester Pays: ai trả phí STORAGE? | ⭐⭐ **Vẫn là OWNER** |
| Requester Pays: ai trả phí NETWORKING? | ⭐⭐ **REQUESTER** |
| Requester Pays có cho phép truy cập ẩn danh? | ❌ ⭐⭐ **KHÔNG — requester phải được xác thực trong AWS** |
| Requester Pays dùng khi nào? | ⭐ **Chia sẻ tập dữ liệu LỚN với account khác** |
| S3 Event Notifications có những event nào? | ⭐ **`S3:ObjectCreated`, `S3:ObjectRemoved`, `S3:ObjectRestore`, `S3:Replication`…** |
| 3 đích truyền thống của S3 Event Notifications? | ⭐⭐ **SNS, SQS, Lambda Function** |
| Cấp quyền cho S3 gửi event bằng gì? | ⭐⭐⭐ **Resource (Access) Policy trên SNS/SQS/Lambda — KHÔNG phải IAM Role** |
| S3 Event gửi mất bao lâu? | ⭐ **Thường vài giây, đôi khi một phút hoặc lâu hơn** |
| Event Notifications lọc được theo gì? | ⭐ **Tên object (ví dụ `*.jpg`)** |
| EventBridge hỗ trợ bao nhiêu đích? | ⭐⭐ **Hơn 18 dịch vụ AWS** |
| EventBridge có lợi thế gì? | ⭐⭐ **Advanced filtering (JSON), multiple destinations, Archive/Replay, reliable delivery** |
| Cần lọc event theo KÍCH THƯỚC object → dùng gì? | ⭐⭐ **EventBridge** |
| S3 latency bao nhiêu? | ⭐ **100–200 ms** |
| Bao nhiêu PUT/COPY/POST/DELETE mỗi giây mỗi prefix? | ⭐⭐ **3,500** |
| Bao nhiêu GET/HEAD mỗi giây mỗi prefix? | ⭐⭐ **5,500** |
| Số prefix trong một bucket bị giới hạn không? | ❌ ⭐⭐ **KHÔNG giới hạn** |
| Trải đều trên 4 prefix được bao nhiêu GET/s? | ⭐⭐ **22,000** (4 × 5,500) |
| Multi-Part upload khuyến nghị / bắt buộc từ kích thước nào? | ⭐⭐ **> 100MB khuyến nghị, > 5GB BẮT BUỘC** |
| Transfer Acceleration hoạt động thế nào? | ⭐⭐ **Truyền tới Edge Location, rồi đi qua mạng riêng AWS tới bucket** |
| Transfer Acceleration có dùng chung với multi-part? | ✅ ⭐ **Có — tương thích** |
| Byte-Range Fetches dùng để làm gì? | ⭐⭐ **Song song hóa GET để tăng tốc download, hoặc lấy một phần dữ liệu (header)** |
| User ở Úc upload lên bucket ở Mỹ chậm → dùng gì? | ⭐ **S3 Transfer Acceleration** |
| Mã hóa hàng triệu object chưa mã hóa → dùng gì? | ⭐⭐ **S3 Batch Operations** |
| S3 Batch Operations làm được những gì? | ⭐ **Sửa metadata/properties, copy giữa bucket, mã hóa, sửa ACL/tags, restore từ Glacier, gọi Lambda** |
| Một Batch Operations job gồm những gì? | ⭐ **Danh sách object + hành động + tham số tùy chọn** |
| Lấy danh sách object cho Batch Operations bằng gì? | ⭐⭐ **S3 Inventory**, lọc bằng **Athena** |
| S3 Batch Operations tự quản lý những gì? | ⭐ **Retries, tiến độ, thông báo hoàn thành, báo cáo** |
| Storage Lens phân tích phạm vi nào? | ⭐⭐ **Toàn bộ AWS Organization** |
| Storage Lens lưu bao nhiêu ngày dữ liệu usage/activity? | ⭐ **30 ngày** |
| Storage Lens tổng hợp theo cấp nào? | ⭐ **Organization, accounts, regions, buckets, prefixes** |
| Storage Lens export được định dạng gì? | ⭐ **CSV, Parquet** (hàng ngày, tới S3 bucket) |
| Default dashboard có xóa được không? | ❌ ⭐⭐ **KHÔNG xóa được, chỉ DISABLE được** |
| Metrics nào giúp tìm multipart upload cũ hơn 7 ngày? | ⭐⭐ **Cost-Optimization Metrics** |
| Metrics nào cho biết bucket nào chưa bật versioning? | ⭐ **Data-Protection Metrics** |
| Metrics nào cho biết lỗi 403/404? | ⭐ **Detailed Status Code Metrics** |
| Free metrics có bao nhiêu metric, lưu bao lâu? | ⭐⭐ **~28 usage metrics, lưu 14 NGÀY** |
| Advanced metrics lưu dữ liệu bao lâu? | ⭐⭐ **15 THÁNG** |
| Advanced metrics có tính năng gì thêm? | ⭐ **CloudWatch Publishing (không phí thêm), Prefix Aggregation** |

### Checklist tự kiểm tra trước khi làm quiz

- [ ] Nhớ **Transition vs Expiration actions**, lọc theo **prefix/tags**, áp dụng cho **current & noncurrent versions**
- [ ] Thuộc **2 scenario mẫu** (thumbnail → One-Zone IA + expire; recover 48h → Deep Archive)
- [ ] Nhớ **S3 Analytics: chỉ Standard/Standard-IA, KHÔNG One-Zone IA/Glacier, 24–48 giờ**
- [ ] Nhớ **Requester Pays: owner trả STORAGE, requester trả NETWORKING, không ẩn danh**
- [ ] Nhớ **3 đích event (SNS/SQS/Lambda)** và **phải dùng Resource Policy, không phải IAM Role**
- [ ] Nhớ **EventBridge: >18 đích, JSON filtering, Archive/Replay**
- [ ] Thuộc con số performance: **3,500 ghi / 5,500 đọc mỗi prefix**, **không giới hạn prefix**, **4 prefix = 22,000 GET/s**
- [ ] Nhớ **Multi-Part: >100MB khuyến nghị, >5GB bắt buộc**
- [ ] Phân biệt **Multi-Part (upload)** / **Transfer Acceleration (khoảng cách)** / **Byte-Range (download song song, lấy header)**
- [ ] Nhớ **S3 Batch Operations + S3 Inventory + Athena** là bộ ba
- [ ] Nhớ **Storage Lens: toàn Organization, 30 ngày metrics, default dashboard không xóa được**
- [ ] Thuộc **Free (28 metrics, 14 ngày)** vs **Paid (15 tháng, CloudWatch Publishing, Prefix Aggregation)**

---

## Thuật ngữ Anh — Việt

| Tiếng Anh | Tiếng Việt |
|-----------|-----------|
| Lifecycle Rules | Quy tắc vòng đời |
| Transition Action | Hành động chuyển đổi lớp lưu trữ |
| Expiration Action | Hành động hết hạn (xóa) |
| Current version | Phiên bản hiện hành |
| Noncurrent version | Phiên bản cũ (không còn hiện hành) |
| Incomplete Multi-Part upload | Tải lên nhiều phần chưa hoàn tất |
| Prefix | Tiền tố đường dẫn |
| Object Tags | Thẻ gán cho object |
| Storage Class Analysis | Phân tích lớp lưu trữ |
| Recommendations | Khuyến nghị |
| Requester Pays | Người yêu cầu trả phí |
| Bucket owner | Chủ sở hữu bucket |
| Storage Cost / Networking Cost | Chi phí lưu trữ / Chi phí mạng |
| Authenticated / Anonymous | Đã xác thực / Ẩn danh |
| Event Notifications | Thông báo sự kiện |
| Object name filtering | Lọc theo tên object |
| Resource (Access) Policy | Chính sách truy cập gắn trên tài nguyên |
| Destination | Đích đến |
| EventBridge | Dịch vụ điều phối sự kiện của AWS |
| Archive / Replay Events | Lưu trữ / Phát lại sự kiện |
| Reliable delivery | Gửi tin cậy |
| Baseline Performance | Hiệu năng nền |
| Requests per second per prefix | Số request mỗi giây trên mỗi prefix |
| Multi-Part upload | Tải lên nhiều phần |
| Parallelize | Song song hóa |
| Transfer Acceleration | Tăng tốc truyền tải |
| Edge Location | Điểm biên của AWS |
| Byte-Range Fetches | Lấy theo dải byte |
| Resilience | Khả năng chống chịu lỗi |
| Batch Operations | Thao tác hàng loạt |
| Bulk operations | Thao tác số lượng lớn |
| Retries | Thử lại |
| Completion notifications | Thông báo hoàn thành |
| S3 Inventory | Báo cáo kiểm kê object |
| Athena | Dịch vụ truy vấn SQL trên S3 |
| Storage Lens | Công cụ phân tích storage toàn tổ chức |
| AWS Organization | Tổ chức AWS (nhiều account) |
| Anomalies | Bất thường |
| Cost efficiencies | Hiệu quả chi phí |
| Aggregate | Tổng hợp |
| Dashboard | Bảng điều khiển trực quan |
| Summary Metrics | Chỉ số tổng quan |
| Cost-Optimization Metrics | Chỉ số tối ưu chi phí |
| Data-Protection Metrics | Chỉ số bảo vệ dữ liệu |
| Access-management Metrics | Chỉ số quản lý truy cập |
| Activity Metrics | Chỉ số hoạt động |
| Detailed Status Code Metrics | Chỉ số mã trạng thái HTTP |
| Prefix Aggregation | Tổng hợp ở cấp tiền tố |
| CloudWatch Publishing | Xuất chỉ số sang CloudWatch |

---

*Ghi chú: các phần Hands On được tóm tắt lại các bước thao tác chính trên AWS Console. Giao diện Console có thể thay đổi theo thời gian — logic và khái niệm vẫn giữ nguyên. ⚠️ Ở bài 147, bước dễ sai nhất là **quên gắn Access Policy cho SQS** trước khi tạo Event Notification — S3 sẽ báo lỗi `Unable to validate the following destination configurations`.*
