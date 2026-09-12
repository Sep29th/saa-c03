# Phần 12 — Amazon S3 Introduction

> Khóa học: *Ultimate AWS Certified Solutions Architect Associate 2026* (Stéphane Maarek) — SAA-C03
> Nguồn tham chiếu: `AWS Certified Solutions Architect Slides v48.pdf` (phần "Amazon S3")
> Code kèm theo: `code_v2025-10-27/s3/` (`index.html`, `coffee.jpg`, `extra-page.html`, `CORS_CONFIG.json`)

---

## Mục lục

| # | Bài giảng | Thời lượng | Loại |
|---|-----------|-----------|------|
| 129 | [S3 Overview](#129-s3-overview) | 6 phút | Video |
| 130 | [S3 Hands On](#130-s3-hands-on) | 7 phút | Video |
| 131 | [S3 Security: Bucket Policy](#131-s3-security-bucket-policy) | 5 phút | Video |
| 132 | [S3 Security: Bucket Policy Hands On](#132-s3-security-bucket-policy-hands-on) | 3 phút | Video |
| 133 | [S3 Website Overview](#133-s3-website-overview) | 1 phút | Video |
| 134 | [S3 Website Hands On](#134-s3-website-hands-on) | 2 phút | Video |
| 135 | [S3 Versioning](#135-s3-versioning) | 1 phút | Video |
| 136 | [S3 Versioning - Hands On](#136-s3-versioning---hands-on) | 4 phút | Video |
| 137 | [S3 Replication](#137-s3-replication) | 1 phút | Video |
| 138 | [S3 Replication Notes](#138-s3-replication-notes) | 1 phút | Video |
| 139 | [S3 Replication - Hands On](#139-s3-replication---hands-on) | 6 phút | Video |
| 140 | [S3 Storage Classes Overview](#140-s3-storage-classes-overview) | 6 phút | Video |
| 141 | [S3 Storage Classes Hands On](#141-s3-storage-classes-hands-on) | 4 phút | Video |
| 142 | [S3 Express One Zone](#142-s3-express-one-zone) | 2 phút | Video |
| — | [Trắc nghiệm 9: Amazon S3 Quiz](#trắc-nghiệm-9-amazon-s3-quiz) | — | Quiz |

---

## 129. S3 Overview

### Section Introduction ⭐

- ⭐ **Amazon S3 là MỘT TRONG NHỮNG KHỐI XÂY DỰNG CHÍNH của AWS**
- ⭐⭐ **Được quảng cáo là lưu trữ "INFINITELY SCALING" (mở rộng vô hạn)**
- ⭐ **Nhiều website dùng Amazon S3 làm xương sống**
- ⭐ **Nhiều dịch vụ AWS cũng tích hợp với Amazon S3**

---

### Amazon S3 Use cases ⭐

| Use case | Ví dụ thực tế |
|----------|---------------|
| **Backup and storage** | Sao lưu dữ liệu |
| **Disaster Recovery** | Khôi phục thảm họa |
| **Archive** | ⭐ **Nasdaq lưu 7 NĂM dữ liệu vào S3 Glacier** |
| **Hybrid Cloud storage** | Kết hợp on-premises + cloud |
| **Application hosting** | Lưu trữ ứng dụng |
| **Media hosting** | Lưu trữ video, ảnh |
| **Data lakes & big data analytics** | ⭐ **Sysco chạy analytics trên dữ liệu để thu được business insights** |
| **Software delivery** | Phân phối phần mềm |
| **Static website** | Website tĩnh |

---

### Amazon S3 — Buckets ⭐⭐

- ⭐ **Amazon S3 cho phép lưu OBJECTS (files) trong "BUCKETS" (directories)**
- ⭐⭐ **Buckets được định nghĩa ở CẤP ĐỘ REGION**
- ⭐⭐ **S3 TRÔNG CÓ VẺ là dịch vụ global NHƯNG bucket được tạo trong MỘT REGION**

### Naming ⭐

| Loại namespace | Ý nghĩa |
|----------------|---------|
| ⭐ **Shared Global Namespace** | **Phải có tên DUY NHẤT TOÀN CẦU** (across all regions all accounts) |
| ⭐ **Account Regional Namespace** | **Cho phép "TÁI SỬ DỤNG" cùng một tên bucket qua các Region khác nhau** |

### ⭐⭐ Naming constraints (ràng buộc đặt tên) — hay ra thi

| # | Ràng buộc |
|---|-----------|
| 1 | ⭐ **KHÔNG chữ HOA, KHÔNG dấu gạch dưới (underscore)** |
| 2 | ⭐ **KHÔNG được là một địa chỉ IP** |
| 3 | ⭐ **PHẢI bắt đầu bằng chữ thường hoặc số** |
| 4 | ⭐ **KHÔNG được bắt đầu bằng prefix `xn--`** |
| 5 | ⭐ **KHÔNG được kết thúc bằng suffix `-s3alias`** |

> Ngoài ra: độ dài **3–63 ký tự**.

---

### Amazon S3 — Objects ⭐⭐

- ⭐ **Objects (files) có một KEY**
- ⭐⭐ **Key là ĐƯỜNG DẪN ĐẦY ĐỦ (FULL path):**
  - `s3://my-bucket/my_file.txt`
  - `s3://my-bucket/my_folder1/another_folder/my_file.txt`

### ⭐⭐ Key = prefix + object name

```
   s3://my-bucket/my_folder1/another_folder/my_file.txt
                  └──────────┬───────────┘ └────┬────┘
                          PREFIX             OBJECT NAME
```

### ⭐⭐ Không có khái niệm "thư mục" trong bucket!

> **KHÔNG có khái niệm "directories" bên trong bucket** (mặc dù giao diện UI sẽ ĐÁNH LỪA bạn nghĩ ngược lại).
> **Chỉ là các KEY với tên rất dài có chứa dấu gạch chéo (`/`)**.

---

### Amazon S3 — Objects (cont.) ⭐⭐

**Object values là nội dung của body:**

| Thuộc tính | Chi tiết |
|-----------|----------|
| ⭐⭐ **Max Object Size** | **50 TB (50,000 GB)** |
| ⭐⭐ **Multi-part upload** | **Nếu upload HƠN 5GB, PHẢI dùng "multi-part upload"** |
| ⭐ **Metadata** | **Danh sách cặp key/value dạng text — system hoặc user metadata** |
| ⭐ **Tags** | **Cặp key/value Unicode — TỐI ĐA 10** — hữu ích cho **security / lifecycle** |
| ⭐ **Version ID** | **Nếu versioning được bật** |

> **Ba con số phải nhớ:** ⭐ **Max object 50TB**, ⭐ **> 5GB phải multi-part upload**, ⭐ **tối đa 10 tags**.

---

## 130. S3 Hands On

### Bước 1 — Tạo Bucket

1. Console → tìm **S3** → **Create bucket**.
2. **General configuration**:
   - ⭐ **AWS Region**: chọn Region (ví dụ `eu-west-1`) — **bucket nằm trong Region này**
   - **Bucket type**: `General purpose` (hoặc `Directory` cho Express One Zone)
   - ⭐ **Bucket name**: phải **duy nhất toàn cầu** (ví dụ `stephane-demo-s3-v3`)
3. **Object Ownership**: `ACLs disabled (recommended)` ⭐
4. ⭐⭐ **Block Public Access settings for this bucket**: **giữ BẬT tất cả** (mặc định)
5. **Bucket Versioning**: `Disable` (sẽ bật ở bài 136)
6. **Default encryption**: `SSE-S3` (mặc định)
7. **Create bucket**.

### Bước 2 — Upload objects

1. Mở bucket → **Upload** → **Add files** → chọn file (ví dụ `coffee.jpg`).
2. **Upload** → xem kết quả **Succeeded**.
3. Bấm vào object → xem:
   - ⭐ **Object URL**: `https://<bucket>.s3.<region>.amazonaws.com/coffee.jpg`
   - **Key**, **Size**, **Type**, **Last modified**, **ETag**

### Bước 3 — Hiểu về "thư mục" ⭐⭐

1. **Create folder** → đặt tên `images` → **Create folder**.
2. Upload một file vào trong `images/`.
3. ⭐ **Quan sát Key của object đó: `images/beach.jpg`**
   → **Không có thư mục thật, chỉ là key chứa dấu `/`!**

### Bước 4 — Thử truy cập public ⭐⭐

| Cách truy cập | Kết quả |
|---------------|---------|
| Bấm nút **Open** trong Console | ✅ **Hoạt động** — vì dùng **pre-signed URL** (có chữ ký tạm) |
| Copy **Object URL** rồi dán vào tab mới | ❌ ⭐ **`403 Forbidden` — Access Denied** |

> ⭐⭐ **Vì sao?** Object **KHÔNG public** theo mặc định. Nút "Open" dùng **credentials của bạn**, còn URL trần thì truy cập ẩn danh → bị từ chối. Đây chính là vấn đề mà **Bucket Policy** (bài 131) giải quyết.

### Bước 5 — Các thao tác khác

| Thao tác | Vị trí |
|----------|--------|
| **Download** | Chọn object → Download |
| **Copy / Move** | Actions → Copy / Move |
| **Delete** | Chọn object → Delete → gõ `permanently delete` |
| **Xóa bucket** | ⚠️ Phải **rỗng** trước → **Empty** rồi mới **Delete** |

---

## 131. S3 Security: Bucket Policy

### Amazon S3 — Security ⭐⭐⭐

S3 có **2 nhóm cơ chế bảo mật**:

#### 🔹 User-Based

| Cơ chế | Mô tả |
|--------|-------|
| ⭐ **IAM Policies** | **Những API call nào được phép cho một user cụ thể từ IAM** |

#### 🔹 Resource-Based

| Cơ chế | Mô tả |
|--------|-------|
| ⭐⭐ **Bucket Policies** | **Quy tắc áp dụng toàn bucket, cấu hình từ S3 console** — ⭐ **CHO PHÉP CROSS ACCOUNT** |
| ⭐ **Object Access Control List (ACL)** | **Chi tiết hơn (finer grain)** — ⭐ **có thể bị disable** |
| ⭐ **Bucket Access Control List (ACL)** | **Ít dùng hơn (less common)** — ⭐ **có thể bị disable** |

---

### ⭐⭐⭐ QUY TẮC ĐÁNH GIÁ QUYỀN (rất hay ra thi)

> **Một IAM principal có thể truy cập một S3 object NẾU:**
>
> **• Quyền IAM của user CHO PHÉP (ALLOW) nó, HOẶC resource policy CHO PHÉP (ALLOW) nó**
> **• VÀ không có DENY tường minh (explicit DENY)**

```
   (IAM permissions ALLOW  OR  Resource policy ALLOW)
                    AND
              NO explicit DENY
                     ↓
              ✅ Được truy cập
```

- ⭐ **Encryption**: mã hóa object trong Amazon S3 bằng encryption keys.

---

### S3 Bucket Policies ⭐⭐

**Cấu trúc (JSON based policies):**

| Thành phần | Ý nghĩa |
|-----------|---------|
| ⭐ **Resources** | **buckets và objects** |
| ⭐ **Effect** | **Allow / Deny** |
| ⭐ **Actions** | **Tập hợp các API để Allow hoặc Deny** |
| ⭐⭐ **Principal** | **Account hoặc user mà policy được áp dụng** |

### ⭐⭐ Dùng S3 bucket policy để:

| # | Mục đích |
|---|----------|
| 1 | ⭐ **Cấp quyền truy cập PUBLIC cho bucket** |
| 2 | ⭐⭐ **BẮT BUỘC object phải được mã hóa khi upload** (force encryption at upload) |
| 3 | ⭐⭐ **Cấp quyền truy cập cho một account KHÁC (Cross Account)** |

### Ví dụ Bucket Policy cấp quyền đọc public

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicRead",
      "Effect": "Allow",
      "Principal": "*",
      "Action": ["s3:GetObject"],
      "Resource": ["arn:aws:s3:::examplebucket/*"]
    }
  ]
}
```

> ⚠️ ⭐ **Chú ý `/*` ở cuối Resource** — nghĩa là áp dụng cho **mọi object trong bucket**, không phải bản thân bucket.

---

### ⭐⭐ 4 kịch bản bảo mật kinh điển (từ slide)

#### 1️⃣ Public Access — Use Bucket Policy

```
   Anonymous www website visitor ──► [S3 Bucket Policy Allows Public Access] ──► S3 Bucket
```

#### 2️⃣ User Access to S3 — IAM permissions

```
   IAM User ──[IAM Policy]──► S3 Bucket
```

#### 3️⃣ EC2 instance access — Use IAM Roles

```
   EC2 Instance ──[EC2 Instance Role + IAM permissions]──► S3 Bucket
```

> ⭐ Nhắc lại từ phần 4: **KHÔNG BAO GIỜ** lưu access key trên EC2 — luôn dùng **IAM Role**.

#### 4️⃣ Cross-Account Access — Use Bucket Policy ⭐⭐

```
   IAM User (Other AWS account) ──► [S3 Bucket Policy Allows Cross-Account] ──► S3 Bucket
```

> ⭐⭐ **Ghi nhớ:** Truy cập **cross-account** → **PHẢI dùng Bucket Policy** (IAM Policy một mình không đủ).

---

### ⭐⭐ Bucket settings for Block Public Access

- ⭐ **Các setting này được tạo ra để NGĂN RÒ RỈ DỮ LIỆU CỦA CÔNG TY (prevent company data leaks)**
- ⭐ **Nếu bạn biết bucket của mình KHÔNG BAO GIỜ nên public, hãy để các setting này BẬT**
- ⭐⭐ **Có thể đặt ở CẤP ĐỘ ACCOUNT**

> ⭐⭐ **Điểm quan trọng:** Block Public Access **GHI ĐÈ (override)** mọi Bucket Policy. Dù bucket policy cho phép public, nếu Block Public Access đang bật thì **vẫn bị chặn**.

---

## 132. S3 Security: Bucket Policy Hands On

### Bước 1 — Xác nhận object chưa public

1. Mở object → copy **Object URL** → dán vào tab mới.
2. ❌ Kết quả: **`403 Forbidden` / AccessDenied`**.

### Bước 2 — Tắt Block Public Access ⭐

1. Bucket → tab **Permissions** → mục **Block public access (bucket settings)** → **Edit**.
2. ⚠️ **Bỏ tick "Block all public access"**.
3. **Save changes** → gõ **`confirm`** để xác nhận.

> ⚠️ AWS sẽ hiển thị cảnh báo màu đỏ — đây là **hành động có chủ đích**, chỉ làm khi bạn thực sự muốn bucket public.

### Bước 3 — Thêm Bucket Policy ⭐⭐

1. Tab **Permissions** → mục **Bucket policy** → **Edit**.
2. Bấm ⭐ **Policy generator** (mở tab mới) để tạo nhanh:
   - **Select Type of Policy**: `S3 Bucket Policy`
   - **Effect**: `Allow`
   - ⭐ **Principal**: `*` (tất cả mọi người)
   - **AWS Service**: `Amazon S3`
   - ⭐ **Actions**: `GetObject`
   - ⭐ **Amazon Resource Name (ARN)**: `arn:aws:s3:::<bucket-name>/*` ← **nhớ dấu `/*`**
   - **Add Statement** → **Generate Policy** → copy JSON
3. Dán JSON vào ô Bucket policy → **Save changes**.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::stephane-demo-s3-v3/*"
    }
  ]
}
```

### Bước 4 — Kiểm chứng ⭐

1. Quay lại object → copy **Object URL** → dán vào tab mới.
2. ✅ **Object hiển thị thành công!**
3. Bucket giờ có nhãn ⭐ **"Publicly accessible"** màu đỏ trong danh sách.

### Lỗi thường gặp ⭐

| Lỗi | Nguyên nhân |
|-----|-------------|
| **`403 Forbidden`** dù đã thêm policy | ⭐⭐ **Chưa tắt Block Public Access** |
| **`403 Forbidden`** dù đã tắt Block Public Access | ⭐ **Thiếu `/*` ở cuối ARN trong Resource** |
| Policy báo lỗi syntax | JSON sai — dùng **Policy generator** |
| Truy cập được object này nhưng object khác thì không | Policy chỉ áp dụng cho prefix cụ thể |

---

## 133. S3 Website Overview

### Amazon S3 — Static Website Hosting ⭐

- ⭐⭐ **S3 CÓ THỂ host các website TĨNH (static websites) và cho chúng truy cập được trên Internet**

### URL của website ⭐⭐

**URL sẽ là (tùy theo Region) — một trong hai dạng:**

```
http://bucket-name.s3-website-aws-region.amazonaws.com
                            ▲ dấu GẠCH NGANG
   HOẶC
http://bucket-name.s3-website.aws-region.amazonaws.com
                            ▲ dấu CHẤM
```

**Ví dụ thực tế:**
```
http://demo-bucket.s3-website-us-west-2.amazonaws.com
http://demo-bucket.s3-website.us-west-2.amazonaws.com
```

### ⚠️⭐⭐ Lưu ý quan trọng:

> **Nếu bạn nhận lỗi `403 Forbidden`, hãy đảm bảo BUCKET POLICY CHO PHÉP PUBLIC READS!**

### Phân biệt 2 loại URL của S3 ⭐⭐

| | **REST API endpoint** | **Website endpoint** |
|---|---|---|
| **Dạng** | `bucket.s3.region.amazonaws.com` | ⭐ `bucket.s3-website-region.amazonaws.com` |
| **Hỗ trợ HTTPS** | ✅ Có | ❌ ⭐ **Chỉ HTTP** |
| **Index document** | ❌ Không | ✅ ⭐ **Có (`index.html`)** |
| **Error document** | ❌ Không | ✅ ⭐ **Có (`error.html`)** |
| **Redirect** | ❌ | ✅ Có |

> ⭐ **Muốn HTTPS cho S3 static website** → phải đặt **CloudFront** phía trước.

---

## 134. S3 Website Hands On

### File của khóa học ⭐

Nội dung `code_v2025-10-27/s3/index.html` (phần cốt lõi):

```html
<html>
    <head>
        <title>My First Webpage</title>
    </head>
    <body>
        <h1>I love coffee</h1>
        <p>Hello world!</p>
    </body>

    <img src="coffee.jpg" width=500/>
</html>
```

> 💡 File `index.html` trong repo còn có thêm đoạn JavaScript `fetch()` để demo **CORS** — phần đó thuộc chương sau (S3 Advanced), bài này chỉ cần phần HTML ở trên.

Các file dùng trong bài: **`index.html`** và **`coffee.jpg`**.

### Bước 1 — Upload file website

1. Upload **`index.html`** và **`coffee.jpg`** vào bucket (ở thư mục gốc).

### Bước 2 — Bật Static Website Hosting ⭐

1. Bucket → tab **Properties** → cuộn xuống cuối → ⭐ **Static website hosting** → **Edit**.
2. Cấu hình:
   - **Static website hosting**: ⭐ **Enable**
   - **Hosting type**: `Host a static website`
   - ⭐ **Index document**: **`index.html`**
   - **Error document** (tùy chọn): `error.html`
3. **Save changes**.
4. Cuộn lại xuống → thấy ⭐ **Bucket website endpoint**:
   ```
   http://<bucket-name>.s3-website-<region>.amazonaws.com
   ```

### Bước 3 — Kiểm chứng ⭐

| Tình huống | Kết quả |
|-----------|---------|
| Chưa có Bucket Policy public | ❌ ⭐ **`403 Forbidden`** |
| Đã có Bucket Policy cho phép `s3:GetObject` public (bài 132) | ✅ **Website hiển thị: "I love coffee" + ảnh cà phê** |

### Bước 4 — Quan sát

1. Truy cập website endpoint → thấy `index.html` được **tự động phục vụ** (không cần gõ `/index.html`).
2. ⭐ So sánh với REST endpoint `https://<bucket>.s3.<region>.amazonaws.com/` → **không tự trả về index.html**.

> ⭐⭐ **Bài học:** Static Website Hosting + Bucket Policy public = **2 điều kiện BẮT BUỘC**. Thiếu một trong hai → `403`.

---

## 135. S3 Versioning

### Đặc điểm ⭐⭐

- ⭐ **Bạn có thể VERSION các file trong Amazon S3**
- ⭐⭐ **Được bật ở CẤP ĐỘ BUCKET (bucket level)**
- ⭐ **Ghi đè cùng một key sẽ thay đổi "version": 1, 2, 3….**
- ⭐⭐ **Đây là BEST PRACTICE — nên version các bucket của bạn:**
  - ⭐ **Bảo vệ khỏi việc XÓA NHẦM (unintended deletes)** — có thể khôi phục một version
  - ⭐ **DỄ DÀNG roll back về version trước**

```
   User ──upload──► S3 Bucket (my-bucket)
                     s3://my-bucket/my-file.docx
                        ├── Version 1
                        ├── Version 2
                        └── Version 3
```

### ⭐⭐ Notes (2 điểm rất hay ra thi)

| # | Ghi chú |
|---|---------|
| 1 | ⭐⭐ **Bất kỳ file nào CHƯA được version TRƯỚC KHI bật versioning sẽ có version là `"null"`** |
| 2 | ⭐⭐ **TẠM DỪNG (suspending) versioning KHÔNG XÓA các version trước đó** |

### Ba trạng thái của Versioning ⭐

| Trạng thái | Ý nghĩa |
|-----------|---------|
| **Unversioned** | Mặc định, chưa từng bật |
| **Enabled** | Đang tạo version mới cho mỗi lần ghi |
| ⭐ **Suspended** | Ngừng tạo version mới, **nhưng version cũ VẪN CÒN** |

> ⚠️ ⭐ **Không thể quay về "Unversioned"** sau khi đã bật — chỉ có thể **Suspend**.

---

## 136. S3 Versioning - Hands On

### Bước 1 — Bật Versioning

1. Bucket → tab **Properties** → **Bucket Versioning** → **Edit**.
2. Chọn ⭐ **Enable** → **Save changes**.

### Bước 2 — Tạo nhiều version ⭐

1. Sửa nội dung `index.html` ở máy (ví dụ đổi `I love coffee` thành `I REALLY love coffee`).
2. Upload lại **cùng tên file** `index.html`.
3. Bucket → bật công tắc ⭐ **Show versions** (góc trên danh sách object).
4. Quan sát: thấy **2 version** của `index.html`, mỗi cái có **Version ID** riêng.
5. Refresh website → thấy nội dung **mới nhất**.

### Bước 3 — Rollback về version cũ ⭐

1. Bật **Show versions** → chọn **version CŨ** (không phải version mới nhất).
2. ⭐ **Actions** → **Delete** version **MỚI** → gõ `permanently delete`.
3. Refresh website → ⭐ **nội dung quay về bản CŨ!**

### Bước 4 — Hiểu về Delete Marker ⭐⭐

1. **Tắt** Show versions → chọn `coffee.jpg` → **Delete** → gõ `delete`.
2. File **biến mất** khỏi danh sách; website bị lỗi ảnh.
3. ⭐ **Bật Show versions** → thấy object vẫn còn, nhưng có thêm một mục:
   ```
   coffee.jpg    Version ID: xxx    Type: ⭐ Delete marker
   ```
4. ⭐⭐ **Khôi phục: chọn Delete marker → Delete (xóa marker) → file XUẤT HIỆN TRỞ LẠI!**

> ⭐⭐⭐ **Đây là cơ chế quan trọng nhất của Versioning:**
> **Xóa một object khi versioning bật KHÔNG thực sự xóa nó — chỉ tạo ra một "Delete Marker" che nó đi.**
> Muốn xóa vĩnh viễn → phải **xóa từng version cụ thể (có Version ID)**.

### Bảng tổng hợp hành vi xóa ⭐⭐

| Thao tác | Versioning TẮT | Versioning BẬT |
|----------|----------------|----------------|
| **Delete object (không có version ID)** | ⚠️ **Xóa vĩnh viễn** | ⭐ **Tạo Delete Marker** — khôi phục được |
| **Delete một version cụ thể (có version ID)** | — | ⚠️ ⭐ **Xóa VĨNH VIỄN version đó** |
| **Khôi phục file đã xóa** | ❌ Không được | ⭐ **Xóa Delete Marker** |

---

## 137. S3 Replication

### Amazon S3 — Replication (CRR & SRR) ⭐⭐

### ⭐⭐ Điều kiện tiên quyết:

> **PHẢI BẬT VERSIONING ở CẢ bucket NGUỒN và bucket ĐÍCH!**

### Hai loại replication ⭐⭐

| Loại | Tên đầy đủ | Mô tả |
|------|-----------|-------|
| ⭐ **CRR** | **Cross-Region Replication** | Sao chép **giữa các Region KHÁC nhau** |
| ⭐ **SRR** | **Same-Region Replication** | Sao chép **trong CÙNG một Region** |

### Đặc điểm ⭐

- ⭐ **Bucket có thể ở các AWS ACCOUNT KHÁC NHAU**
- ⭐⭐ **Việc sao chép là BẤT ĐỒNG BỘ (asynchronous)**
- ⭐⭐ **PHẢI cấp quyền IAM phù hợp cho S3**

```
   S3 Bucket (eu-west-1) ──⭐ asynchronous replication──► S3 Bucket (us-east-2)
```

### ⭐⭐⭐ Use cases (rất hay ra thi)

| Loại | Use cases |
|------|-----------|
| ⭐ **CRR** | **Compliance (tuân thủ)**, **lower latency access (giảm độ trễ truy cập)**, **replication across accounts (sao chép giữa các account)** |
| ⭐ **SRR** | **Log aggregation (gom log)**, **live replication between production and test accounts (sao chép trực tiếp giữa môi trường production và test)** |

### Mẹo nhớ ⭐

```
CRR → khoảng cách ĐỊA LÝ: tuân thủ pháp lý, giảm latency cho user ở xa, DR
SRR → cùng Region: gom log từ nhiều bucket, đồng bộ prod ↔ test
```

---

## 138. S3 Replication Notes

Đây là bài ngắn nhưng chứa **những chi tiết cực hay ra thi**.

### ⭐⭐⭐ 1. Về object hiện có

| Quy tắc | Chi tiết |
|---------|----------|
| ⭐⭐ **Sau khi bật Replication, CHỈ CÁC OBJECT MỚI được sao chép** | Object cũ **KHÔNG** tự động sao chép |
| ⭐⭐ **Tùy chọn: dùng S3 Batch Replication để sao chép object HIỆN CÓ** | **Sao chép cả object hiện có VÀ các object đã sao chép THẤT BẠI** |

### ⭐⭐⭐ 2. Về thao tác DELETE

| Quy tắc | Chi tiết |
|---------|----------|
| ⭐ **Có thể sao chép DELETE MARKER từ nguồn sang đích** | ⭐ **(đây là tùy chọn — optional setting)** |
| ⭐⭐ **Các lần xóa CÓ VERSION ID thì KHÔNG được sao chép** | ⭐ **(để tránh các hành vi xóa độc hại — malicious deletes)** |

### ⭐⭐⭐ 3. KHÔNG có "chaining" (xâu chuỗi) replication

> **Nếu bucket 1 có replication sang bucket 2, và bucket 2 có replication sang bucket 3,**
> **thì các object được tạo trong bucket 1 KHÔNG được sao chép sang bucket 3.**

```
   Bucket 1 ──replication──► Bucket 2 ──replication──► Bucket 3
       └──────────────── ❌ KHÔNG tự động tới Bucket 3 ─────────┘
```

### Bảng tóm tắt để ôn nhanh ⭐⭐

| Câu hỏi | Đáp án |
|---------|--------|
| Object cũ có được replicate không? | ❌ **Không** — phải dùng **S3 Batch Replication** |
| Delete marker có được replicate không? | ⭐ **Có, nếu bật tùy chọn** |
| Xóa một version cụ thể có replicate không? | ❌ ⭐ **KHÔNG** (chống xóa độc hại) |
| Replication có chaining không? | ❌ ⭐ **KHÔNG** |
| Replication đồng bộ hay bất đồng bộ? | ⭐ **Bất đồng bộ (asynchronous)** |

---

## 139. S3 Replication - Hands On

### Bước 1 — Chuẩn bị 2 bucket ⭐

1. Tạo **bucket nguồn** (ví dụ `stephane-v3-origin`) ở `eu-west-1`:
   - ⭐⭐ **BẬT Bucket Versioning** ngay lúc tạo
2. Tạo **bucket đích** (ví dụ `stephane-v3-replica`) ở `us-east-1`:
   - ⭐⭐ **BẬT Bucket Versioning**

> ⚠️ **Nếu quên bật versioning ở một trong hai bucket → không tạo được replication rule.**

### Bước 2 — Upload file vào bucket nguồn

Upload một file bất kỳ (ví dụ `beach.jpg`) **TRƯỚC KHI** tạo rule — để chứng minh object cũ không được sao chép.

### Bước 3 — Tạo Replication Rule ⭐

1. Bucket nguồn → tab **Management** → ⭐ **Replication rules** → **Create replication rule**.
2. Cấu hình:
   - **Replication rule name**: `DemoReplicationRule`
   - **Status**: `Enabled`
   - **Choose a rule scope**: ⭐ `Apply to all objects in the bucket` (hoặc lọc theo prefix/tag)
3. **Destination**:
   - `Choose a bucket in this account` → **Browse S3** → chọn bucket đích
   - (hoặc `Specify a bucket in another account` cho cross-account)
4. ⭐⭐ **IAM role**: chọn **`Create new role`** — S3 cần quyền để đọc nguồn và ghi đích
5. **Additional replication options** (tùy chọn):
   - ⭐ **Replication Time Control (RTC)** — cam kết **99.99% object được sao chép trong 15 phút**
   - ⭐ **Delete marker replication** — bật để sao chép delete marker
   - **Replica modification sync**, **Change object ownership**
6. **Save**.

### Bước 4 — ⭐⭐ Hỏi về object hiện có

Sau khi Save, AWS hiện hộp thoại:

> **"Do you want to replicate existing objects?"**
> - ⭐ **No, do not replicate existing objects** (mặc định)
> - ⭐ **Yes, replicate existing objects** → tạo một **S3 Batch Operations job**

Chọn **No** để kiểm chứng quy tắc "chỉ object mới được sao chép".

### Bước 5 — Kiểm chứng ⭐

1. Mở bucket đích → ❌ **`beach.jpg` KHÔNG có ở đó** (vì upload trước khi tạo rule).
2. Upload một file MỚI (ví dụ `coffee.jpg`) vào bucket nguồn.
3. Đợi vài giây → refresh bucket đích → ✅ ⭐ **`coffee.jpg` đã xuất hiện!**
4. Chọn object ở bucket đích → tab **Properties** → thấy ⭐ **Replication status: `REPLICA`**
5. Object ở bucket nguồn → **Replication status: `COMPLETED`**

### Bước 6 — Thử xóa ⭐⭐

| Thao tác ở nguồn | Kết quả ở đích |
|------------------|----------------|
| Xóa object (tạo **delete marker**) | ⭐ Chỉ replicate **nếu đã bật Delete marker replication** |
| Xóa một **version cụ thể** | ❌ ⭐ **KHÔNG replicate** (chống xóa độc hại) |

### Bước 7 — Dọn dẹp ⚠️

1. Xóa **Replication rule**.
2. **Empty** rồi **Delete** cả hai bucket.
3. ⚠️ Xóa **IAM role** mà S3 đã tạo (nếu không dùng nữa).

---

## 140. S3 Storage Classes Overview

### ⭐⭐ Danh sách 7 Storage Classes

| # | Storage Class |
|---|---------------|
| 1 | ⭐ **Amazon S3 Standard - General Purpose** |
| 2 | ⭐ **Amazon S3 Standard-Infrequent Access (IA)** |
| 3 | ⭐ **Amazon S3 One Zone-Infrequent Access** |
| 4 | ⭐ **Amazon S3 Glacier Instant Retrieval** |
| 5 | ⭐ **Amazon S3 Glacier Flexible Retrieval** |
| 6 | ⭐ **Amazon S3 Glacier Deep Archive** |
| 7 | ⭐ **Amazon S3 Intelligent Tiering** |

- ⭐ **Có thể chuyển đổi giữa các class THỦ CÔNG hoặc dùng S3 Lifecycle configurations**

---

### S3 Durability and Availability ⭐⭐⭐

#### 🔹 Durability (Độ bền)

- ⭐⭐ **Độ bền CAO: 99.999999999% (11 số 9) của object trên NHIỀU AZ**
- ⭐ **Nếu bạn lưu 10,000,000 object với Amazon S3, trung bình bạn có thể mất MỘT object MỘT LẦN MỖI 10,000 NĂM**
- ⭐⭐ **GIỐNG NHAU cho TẤT CẢ storage class!**

#### 🔹 Availability (Tính sẵn sàng)

- ⭐ **Đo lường mức độ dễ dàng truy cập một dịch vụ**
- ⭐⭐ **THAY ĐỔI tùy theo storage class**
- ⭐ **Ví dụ: S3 Standard có 99.99% availability = KHÔNG khả dụng 53 PHÚT một năm**

> ⭐⭐ **Đây là bẫy thi kinh điển:** Hỏi *"storage class nào có durability cao nhất?"* → **TẤT CẢ đều 11 số 9 như nhau**. Chỉ có **availability** là khác nhau.

---

### 1️⃣ S3 Standard – General Purpose ⭐

- ⭐ **99.99% Availability**
- ⭐ **Dùng cho dữ liệu được truy cập THƯỜNG XUYÊN**
- ⭐ **Độ trễ thấp và thông lượng cao (Low latency and high throughput)**
- ⭐ **Chịu được 2 lỗi cơ sở đồng thời (Sustain 2 concurrent facility failures)**
- ⭐ **Use Cases: Big Data analytics, mobile & gaming applications, content distribution…**

---

### 2️⃣ S3 Storage Classes – Infrequent Access ⭐⭐

> **Cho dữ liệu ÍT được truy cập hơn, NHƯNG cần truy cập NHANH khi cần.**
> **Chi phí thấp hơn S3 Standard.**

#### 🔹 Amazon S3 Standard-Infrequent Access (S3 Standard-IA)

- ⭐ **99.9% Availability**
- ⭐ **Use cases: Disaster Recovery, backups**

#### 🔹 Amazon S3 One Zone-Infrequent Access (S3 One Zone-IA)

- ⭐⭐ **Độ bền cao (99.999999999%) TRONG MỘT AZ DUY NHẤT; DỮ LIỆU BỊ MẤT KHI AZ ĐÓ BỊ PHÁ HỦY**
- ⭐ **99.5% Availability**
- ⭐ **Use Cases: lưu BẢN SAO BACKUP THỨ CẤP của dữ liệu on-premises, hoặc dữ liệu bạn CÓ THỂ TẠO LẠI ĐƯỢC**

> ⭐⭐ **Bẫy thi:** One Zone-IA vẫn có **11 số 9 durability**, nhưng **chỉ trong 1 AZ** — mất AZ là **mất dữ liệu**.

---

### 3️⃣ Amazon S3 Glacier Storage Classes ⭐⭐⭐

> **Lưu trữ object CHI PHÍ THẤP dành cho ARCHIVING / BACKUP**
> ⭐ **Pricing: giá lưu trữ + CHI PHÍ TRUY XUẤT object (object retrieval cost)**

#### 🔹 Amazon S3 Glacier Instant Retrieval

- ⭐⭐ **Truy xuất trong MILLISECOND** — tuyệt vời cho dữ liệu truy cập **một lần mỗi quý**
- ⭐⭐ **Thời gian lưu trữ tối thiểu: 90 NGÀY**

#### 🔹 Amazon S3 Glacier Flexible Retrieval (trước đây là Amazon S3 Glacier)

⭐⭐ **3 tùy chọn truy xuất:**

| Tùy chọn | Thời gian |
|----------|-----------|
| ⭐ **Expedited** | **1 đến 5 PHÚT** |
| ⭐ **Standard** | **3 đến 5 GIỜ** |
| ⭐ **Bulk** | **5 đến 12 GIỜ** — ⭐⭐ **MIỄN PHÍ (free)** |

- ⭐⭐ **Thời gian lưu trữ tối thiểu: 90 NGÀY**

#### 🔹 Amazon S3 Glacier Deep Archive — cho lưu trữ DÀI HẠN

⭐⭐ **2 tùy chọn truy xuất:**

| Tùy chọn | Thời gian |
|----------|-----------|
| ⭐ **Standard** | **12 GIỜ** |
| ⭐ **Bulk** | **48 GIỜ** |

- ⭐⭐ **Thời gian lưu trữ tối thiểu: 180 NGÀY**

### ⭐⭐⭐ Bảng con số Glacier phải thuộc lòng

| Class | Retrieval | Min storage duration |
|-------|-----------|---------------------|
| **Glacier Instant Retrieval** | ⭐ **Milliseconds** | ⭐ **90 ngày** |
| **Glacier Flexible Retrieval** | ⭐ **Expedited 1–5 phút / Standard 3–5 giờ / Bulk 5–12 giờ (free)** | ⭐ **90 ngày** |
| **Glacier Deep Archive** | ⭐ **Standard 12 giờ / Bulk 48 giờ** | ⭐ **180 ngày** |

---

### 4️⃣ S3 Intelligent-Tiering ⭐⭐

- ⭐ **Có phí giám sát và auto-tiering hàng tháng NHỎ**
- ⭐⭐ **TỰ ĐỘNG di chuyển object giữa các Access Tier dựa trên MỨC SỬ DỤNG**
- ⭐⭐ **KHÔNG CÓ PHÍ TRUY XUẤT (no retrieval charges) trong S3 Intelligent-Tiering**

### ⭐⭐ 5 Access Tiers

| Tier | Tự động? | Điều kiện |
|------|----------|-----------|
| ⭐ **Frequent Access tier** | **Tự động** | **Tier mặc định** |
| ⭐ **Infrequent Access tier** | **Tự động** | **Object KHÔNG được truy cập trong 30 NGÀY** |
| ⭐ **Archive Instant Access tier** | **Tự động** | **Object KHÔNG được truy cập trong 90 NGÀY** |
| ⭐ **Archive Access tier** | **TÙY CHỌN (optional)** | **Cấu hình từ 90 ngày đến 700+ ngày** |
| ⭐ **Deep Archive Access tier** | **TÙY CHỌN (optional)** | **Cấu hình từ 180 ngày đến 700+ ngày** |

> ⭐⭐ **Mẹo thi:** Đề nói *"access pattern không đoán trước được, không muốn quản lý lifecycle thủ công"* → **S3 Intelligent-Tiering**.

---

### ⭐⭐⭐ S3 Storage Classes Comparison (BẢNG QUAN TRỌNG NHẤT)

| | **Standard** | **Intelligent-Tiering** | **Standard-IA** | **One Zone-IA** | **Glacier Instant Retrieval** | **Glacier Flexible Retrieval** | **Glacier Deep Archive** |
|---|---|---|---|---|---|---|---|
| **Durability** | ⭐ **99.999999999% (11 9's) — GIỐNG NHAU cho tất cả** ||||||| 
| **Availability** | **99.99%** | **99.9%** | **99.9%** | ⭐ **99.5%** | **99.9%** | **99.99%** | **99.99%** |
| **Availability SLA** | **99.9%** | **99%** | **99%** | **99%** | **99%** | **99.9%** | **99.9%** |
| **Availability Zones** | **≥ 3** | **≥ 3** | **≥ 3** | ⭐⭐ **1** | **≥ 3** | **≥ 3** | **≥ 3** |
| **Min. Storage Duration Charge** | **None** | **None** | ⭐ **30 Days** | ⭐ **30 Days** | ⭐ **90 Days** | ⭐ **90 Days** | ⭐⭐ **180 Days** |
| **Min. Billable Object Size** | **None** | **None** | ⭐ **128 KB** | ⭐ **128 KB** | ⭐ **128 KB** | ⭐ **40 KB** | ⭐ **40 KB** |
| **Retrieval Fee** | **None** | ⭐ **None** | Per GB retrieved | Per GB retrieved | Per GB retrieved | Per GB retrieved | Per GB retrieved |

Tham khảo: `https://aws.amazon.com/s3/storage-classes/`

---

### S3 Storage Classes – Price Comparison (ví dụ `us-east-1`) ⭐

| | **Standard** | **Intelligent-Tiering** | **Standard-IA** | **One Zone-IA** | **Glacier Instant** | **Glacier Flexible** | **Glacier Deep Archive** |
|---|---|---|---|---|---|---|---|
| **Storage Cost** (per GB/month) | **$0.023** | **$0.0025 – $0.023** | **$0.0125** | **$0.01** | **$0.004** | **$0.0036** | ⭐ **$0.00099** |
| **Retrieval Cost** (per 1000 req) | GET $0.0004<br>POST $0.005 | GET $0.0004<br>POST $0.005 | GET $0.001<br>POST $0.01 | GET $0.001<br>POST $0.01 | GET $0.01<br>POST $0.02 | GET $0.0004<br>POST $0.03 | GET $0.0004<br>POST $0.05 |
| **Retrieval Time** | ⭐ **Instantaneous** | Instantaneous | Instantaneous | Instantaneous | Instantaneous | Expedited: $10<br>Standard: $0.05<br>Bulk: **free**<br>(1–5 min / 3–5 h / 5–12 h) | Standard: $0.10 (12 h)<br>Bulk: $0.025 (48 h) |
| **Monitoring Cost** (per 1000 obj) | — | ⭐ **$0.0025** | — | — | — | — | — |

Tham khảo: `https://aws.amazon.com/s3/pricing/`

> ⭐ **Nhận xét:** Giá lưu trữ giảm dần từ **Standard ($0.023)** → **Deep Archive ($0.00099)** — **rẻ hơn ~23 lần**, đổi lại thời gian truy xuất lâu hơn và có phí retrieval.

---

### ⭐⭐ Cheat sheet chọn Storage Class

| Từ khóa trong đề | Đáp án |
|------------------|--------|
| **Truy cập thường xuyên**, low latency, high throughput | **S3 Standard** |
| **Access pattern KHÔNG ĐOÁN TRƯỚC**, không muốn quản lý thủ công | ⭐ **Intelligent-Tiering** |
| **Ít truy cập nhưng cần NHANH khi cần**, backup/DR | **Standard-IA** |
| **Dữ liệu có thể TẠO LẠI ĐƯỢC**, bản sao thứ cấp, rẻ hơn | ⭐ **One Zone-IA** |
| Archive, truy cập **1 lần/quý**, cần **milliseconds** | ⭐ **Glacier Instant Retrieval** |
| Archive, chấp nhận chờ **vài phút đến vài giờ** | ⭐ **Glacier Flexible Retrieval** |
| Archive **DÀI HẠN** (7–10 năm), **RẺ NHẤT**, chờ được **12–48 giờ** | ⭐⭐ **Glacier Deep Archive** |
| Cần **truy xuất miễn phí** từ archive | ⭐ **Glacier Flexible — Bulk (free)** |

---

## 141. S3 Storage Classes Hands On

### Bước 1 — Chọn storage class khi UPLOAD ⭐

1. Bucket → **Upload** → **Add files** → chọn file.
2. Mở rộng ⭐ **Properties** → mục **Storage class**.
3. Thấy danh sách đầy đủ với mô tả và **"Designed for"**:

| Lựa chọn | Mô tả trên Console |
|----------|--------------------|
| **Standard** | Frequently accessed data |
| **Intelligent-Tiering** | Data with changing or unknown access patterns |
| **Standard-IA** | Infrequently accessed data |
| **One Zone-IA** | Re-creatable, infrequently accessed data |
| **Glacier Instant Retrieval** | Long-lived archive data accessed once a quarter |
| **Glacier Flexible Retrieval** | Long-lived archive data accessed once a year |
| **Glacier Deep Archive** | Long-lived archive data accessed less than once a year |
| **Reduced Redundancy** | ⚠️ **Không khuyến nghị (deprecated)** |

4. Chọn một class (ví dụ **Standard-IA**) → **Upload**.

### Bước 2 — Thay đổi storage class của object đã có ⭐

1. Chọn object → tab **Properties** → mục **Storage class** → **Edit**.
2. Chọn class mới (ví dụ **One Zone-IA**) → **Save changes**.
3. Quan sát cột **Storage class** trong danh sách object đã đổi.

### Bước 3 — Xem Lifecycle Rules ⭐

1. Bucket → tab **Management** → ⭐ **Lifecycle rules** → **Create lifecycle rule**.
2. Xem các loại action:
   - ⭐ **Transition current versions of objects between storage classes**
   - ⭐ **Transition noncurrent versions of objects between storage classes**
   - **Expire current versions of objects**
   - **Permanently delete noncurrent versions of objects**
   - **Delete expired object delete markers or incomplete multipart uploads**

> ⭐ **Lifecycle Rules** được học chi tiết ở chương sau (**S3 Advanced**) — ở đây chỉ xem qua giao diện.

### Lưu ý thực hành ⚠️

| Lưu ý | Chi tiết |
|-------|----------|
| ⭐ **Min storage duration** | Chuyển sang IA/Glacier rồi xóa sớm vẫn **bị tính phí đủ 30/90/180 ngày** |
| ⭐ **Min billable object size** | File nhỏ hơn **128 KB** (IA) vẫn tính như 128 KB → **không nên** đưa file nhỏ vào IA |
| **Đổi class có mất phí** | Mỗi lần transition tính như một **request** |

---

## 142. S3 Express One Zone

### Đặc điểm ⭐⭐

- ⭐⭐ **Storage class HIỆU NĂNG CAO, MỘT Availability Zone duy nhất**
- ⭐⭐ **Object được lưu trong DIRECTORY BUCKET** (bucket trong một AZ duy nhất)
- ⭐⭐ **Xử lý HÀNG TRĂM NGHÌN (100,000s) request mỗi giây với độ trễ MỘT CHỮ SỐ MILLISECOND (single-digit millisecond latency)**
- ⭐⭐ **Hiệu năng TỐT HƠN TỚI 10 LẦN so với S3 Standard** (⭐ **chi phí thấp hơn 50%**)
- ⭐ **Độ bền cao (99.999999999%) và Availability 99.95%**
- ⭐⭐ **ĐẶT CÙNG CHỖ (co-locate) storage và compute trong CÙNG MỘT AZ** → **giảm độ trễ**

### Use cases ⭐

- ⭐ **Ứng dụng nhạy cảm với độ trễ (latency-sensitive apps)**
- ⭐ **Ứng dụng thâm dụng dữ liệu (data-intensive apps)**
- ⭐⭐ **AI & ML training**
- ⭐ **Financial modeling**
- ⭐ **Media processing**
- ⭐ **HPC**

### Tích hợp tốt nhất với ⭐

> **SageMaker Model Training, Athena, EMR, Glue…**

### Tên Directory Bucket ⭐

```
   Region (us-east-1)
      └── Availability Zone (AZ 4)
             └── stephane--use1-az4--x-s3
                          └────┬────┘ └─┬─┘
                           mã AZ      suffix bắt buộc
```

> ⭐ Tên directory bucket có dạng đặc biệt: `<base-name>--<az-id>--x-s3`

### ⭐⭐ So sánh S3 Express One Zone vs S3 Standard

| | **S3 Standard** | **S3 Express One Zone** |
|---|---|---|
| **Số AZ** | **≥ 3** | ⭐ **1 (Directory Bucket)** |
| **Latency** | Milliseconds | ⭐⭐ **Single-digit milliseconds** |
| **Hiệu năng** | Chuẩn | ⭐ **Tốt hơn tới 10 lần** |
| **Chi phí request** | Chuẩn | ⭐ **Thấp hơn 50%** |
| **Availability** | **99.99%** | ⭐ **99.95%** |
| **Durability** | 11 số 9 | 11 số 9 (nhưng **1 AZ**) |
| **Use case** | Mục đích chung | ⭐ **AI/ML training, HPC, latency-sensitive** |

> ⭐⭐ **Mẹo thi:** Đề nhắc **"single-digit millisecond"**, **"10x faster"**, **"ML training"**, **"co-locate compute and storage"** → **S3 Express One Zone**.
> ⚠️ Nhưng nhớ: **1 AZ** → **mất AZ là mất dữ liệu**, không dùng cho dữ liệu quan trọng không tái tạo được.

---

## Trắc nghiệm 9: Amazon S3 Quiz

### Các điểm dễ bị bẫy

| Câu hỏi thường gặp | Đáp án đúng |
|--------------------|-------------|
| S3 là dịch vụ global hay regional? | ⭐⭐ **Trông như global nhưng BUCKET được tạo trong một REGION** |
| Tên bucket phải duy nhất ở phạm vi nào? | ⭐ **Duy nhất TOÀN CẦU** (mọi region, mọi account) |
| Tên bucket có được chữ hoa/gạch dưới không? | ❌ ⭐ **KHÔNG** |
| Tên bucket không được bắt đầu/kết thúc bằng gì? | ⭐ **Không bắt đầu `xn--`, không kết thúc `-s3alias`** |
| Object key là gì? | ⭐⭐ **ĐƯỜNG DẪN ĐẦY ĐỦ** = prefix + object name |
| S3 có thư mục thật không? | ❌ ⭐⭐ **KHÔNG — chỉ là key chứa dấu `/`** |
| Object tối đa bao nhiêu? | ⭐⭐ **50 TB (50,000 GB)** |
| Upload hơn bao nhiêu thì phải multi-part? | ⭐⭐ **5 GB** |
| Object tối đa bao nhiêu tag? | ⭐ **10** |
| Quy tắc đánh giá quyền truy cập S3? | ⭐⭐ **(IAM ALLOW **OR** Resource policy ALLOW) **AND** không có explicit DENY** |
| Cross-account access dùng gì? | ⭐⭐ **Bucket Policy** |
| EC2 truy cập S3 dùng gì? | ⭐ **IAM Role** (không dùng access key) |
| Bucket Policy dùng để làm gì? | ⭐ **Public access, force encryption at upload, cross-account** |
| Block Public Access có thể đặt ở cấp nào? | ⭐ **Cấp ACCOUNT** |
| Bucket Policy cho public nhưng vẫn 403? | ⭐⭐ **Chưa tắt Block Public Access** |
| S3 static website URL có HTTPS không? | ❌ ⭐ **Chỉ HTTP** — muốn HTTPS phải dùng **CloudFront** |
| S3 website báo 403 thì kiểm tra gì? | ⭐ **Bucket policy có cho phép public reads không** |
| Versioning bật ở cấp nào? | ⭐⭐ **Bucket level** |
| File có trước khi bật versioning có version gì? | ⭐⭐ **`null`** |
| Suspend versioning có xóa version cũ không? | ❌ ⭐⭐ **KHÔNG** |
| Xóa object khi versioning bật thì sao? | ⭐⭐ **Tạo Delete Marker** — khôi phục được |
| Khôi phục file bị xóa (versioning bật)? | ⭐ **Xóa Delete Marker** |
| Xóa vĩnh viễn một object? | ⭐ **Xóa version cụ thể (có Version ID)** |
| Replication cần điều kiện gì? | ⭐⭐ **BẬT versioning ở CẢ nguồn VÀ đích** |
| Replication đồng bộ hay bất đồng bộ? | ⭐ **Bất đồng bộ (asynchronous)** |
| CRR dùng cho gì? | ⭐ **Compliance, lower latency access, cross-account** |
| SRR dùng cho gì? | ⭐ **Log aggregation, live replication prod ↔ test** |
| Bật replication, object CŨ có được sao chép? | ❌ ⭐⭐ **KHÔNG — phải dùng S3 Batch Replication** |
| Delete marker có replicate không? | ⭐ **Có, nếu bật tùy chọn** |
| Xóa có version ID có replicate không? | ❌ ⭐⭐ **KHÔNG** (chống malicious deletes) |
| Replication có chaining (bucket 1→2→3)? | ❌ ⭐⭐ **KHÔNG** |
| Bucket replication có được khác account? | ✅ **CÓ** |
| Storage class nào có durability cao nhất? | ⭐⭐ **TẤT CẢ đều 11 số 9 như nhau** |
| Availability của S3 Standard? | ⭐ **99.99%** |
| Availability của One Zone-IA? | ⭐ **99.5%** (thấp nhất) |
| Storage class nào chỉ có 1 AZ? | ⭐⭐ **One Zone-IA** (và **Express One Zone**) |
| Min storage duration của Standard-IA / One Zone-IA? | ⭐ **30 ngày** |
| Min storage duration của Glacier Instant/Flexible? | ⭐ **90 ngày** |
| Min storage duration của Glacier Deep Archive? | ⭐⭐ **180 ngày** |
| Min billable object size của IA / Glacier? | ⭐ **128 KB / 40 KB** |
| Glacier Instant Retrieval truy xuất bao lâu? | ⭐ **Milliseconds** |
| Glacier Flexible có mấy tùy chọn truy xuất? | ⭐ **3: Expedited (1–5 phút), Standard (3–5 giờ), Bulk (5–12 giờ, FREE)** |
| Glacier Deep Archive truy xuất bao lâu? | ⭐⭐ **Standard 12 giờ, Bulk 48 giờ** |
| Tùy chọn truy xuất nào MIỄN PHÍ? | ⭐ **Bulk của Glacier Flexible Retrieval** |
| Intelligent-Tiering có phí truy xuất không? | ❌ ⭐⭐ **KHÔNG có retrieval charges** |
| Intelligent-Tiering có phí gì? | ⭐ **Phí monitoring và auto-tiering hàng tháng** |
| IA tier tự động sau bao nhiêu ngày không truy cập? | ⭐ **30 ngày** |
| Archive Instant Access tier sau bao nhiêu ngày? | ⭐ **90 ngày** |
| Tier nào là optional trong Intelligent-Tiering? | ⭐ **Archive Access (90→700+ ngày)** và **Deep Archive Access (180→700+ ngày)** |
| Dữ liệu có thể tạo lại được, muốn rẻ → chọn gì? | ⭐ **One Zone-IA** |
| Access pattern không đoán trước → chọn gì? | ⭐ **Intelligent-Tiering** |
| Archive rẻ nhất, chờ 12–48 giờ → chọn gì? | ⭐ **Glacier Deep Archive** |
| S3 Express One Zone latency? | ⭐⭐ **Single-digit millisecond** |
| S3 Express One Zone nhanh hơn Standard bao nhiêu? | ⭐ **Tới 10 lần**, chi phí request **thấp hơn 50%** |
| S3 Express One Zone lưu object ở đâu? | ⭐⭐ **Directory Bucket** (một AZ) |
| S3 Express One Zone availability? | ⭐ **99.95%** |
| S3 Express One Zone hợp với use case nào? | ⭐ **AI/ML training, HPC, latency-sensitive, financial modeling** |

### Checklist tự kiểm tra trước khi làm quiz

- [ ] Nhớ **bucket = region level**, tên **duy nhất toàn cầu**, **5 ràng buộc đặt tên**
- [ ] Nhớ **key = full path**, **không có thư mục thật**
- [ ] Thuộc **3 con số object**: **50TB max**, **>5GB → multi-part**, **10 tags**
- [ ] Thuộc **quy tắc đánh giá quyền**: `(IAM ALLOW OR Resource ALLOW) AND no explicit DENY`
- [ ] Nhớ **cross-account → Bucket Policy**, **EC2 → IAM Role**
- [ ] Nhớ **Block Public Access GHI ĐÈ Bucket Policy**
- [ ] Nhớ **S3 website chỉ HTTP**, cần **CloudFront** cho HTTPS
- [ ] Nhớ **Versioning: bucket level, version `null`, suspend không xóa, Delete Marker**
- [ ] Thuộc **4 Replication Notes**: chỉ object mới, Batch Replication, delete marker optional, **không chaining**, xóa có version ID **không replicate**
- [ ] Nhớ **CRR (compliance/latency/cross-account)** vs **SRR (log aggregation/prod-test)**
- [ ] Nhớ **durability 11 số 9 GIỐNG NHAU**, chỉ **availability khác nhau**
- [ ] Thuộc bảng **min storage duration**: **30 / 30 / 90 / 90 / 180 ngày**
- [ ] Thuộc bảng **min billable size**: **128 KB (IA, Glacier Instant)** / **40 KB (Glacier Flexible, Deep Archive)**
- [ ] Thuộc **thời gian truy xuất Glacier**: Instant (ms), Flexible (1–5min / 3–5h / 5–12h free), Deep (12h / 48h)
- [ ] Nhớ **Intelligent-Tiering KHÔNG có retrieval fee**, có **5 tier** (3 tự động + 2 optional)
- [ ] Nhớ **Express One Zone: Directory Bucket, single-digit ms, 10x nhanh, 50% rẻ hơn, 99.95%**

---

## Thuật ngữ Anh — Việt

| Tiếng Anh | Tiếng Việt |
|-----------|-----------|
| Infinitely scaling | Mở rộng vô hạn |
| Object storage | Lưu trữ dạng đối tượng |
| Bucket | Thùng chứa (tương đương thư mục gốc) |
| Object | Đối tượng (file) |
| Key | Khóa — đường dẫn đầy đủ của object |
| Prefix | Tiền tố (phần đường dẫn trước tên file) |
| Metadata | Siêu dữ liệu |
| Tags | Thẻ gán nhãn |
| Multi-part upload | Tải lên nhiều phần |
| Globally unique | Duy nhất toàn cầu |
| Namespace | Không gian tên |
| User-Based / Resource-Based | Dựa trên người dùng / Dựa trên tài nguyên |
| Bucket Policy | Chính sách cấp bucket |
| Access Control List (ACL) | Danh sách kiểm soát truy cập |
| Principal | Chủ thể được áp dụng policy |
| Explicit DENY | Từ chối tường minh |
| Cross Account | Liên tài khoản |
| Block Public Access | Chặn truy cập công khai |
| Data leak | Rò rỉ dữ liệu |
| Static Website Hosting | Lưu trữ website tĩnh |
| Index document / Error document | Trang mặc định / Trang lỗi |
| Versioning | Quản lý phiên bản |
| Version ID | Mã định danh phiên bản |
| Delete Marker | Dấu đánh dấu đã xóa |
| Suspend | Tạm dừng |
| Roll back | Quay lại phiên bản trước |
| Replication | Sao chép |
| CRR (Cross-Region Replication) | Sao chép liên vùng |
| SRR (Same-Region Replication) | Sao chép cùng vùng |
| Asynchronous | Bất đồng bộ |
| Batch Replication | Sao chép hàng loạt |
| Chaining | Xâu chuỗi (replication nối tiếp) |
| Malicious deletes | Xóa với mục đích phá hoại |
| Log aggregation | Gom nhật ký |
| Storage Class | Lớp lưu trữ |
| Durability | Độ bền dữ liệu |
| Availability | Tính sẵn sàng |
| Availability SLA | Cam kết mức sẵn sàng |
| Frequently / Infrequently accessed | Truy cập thường xuyên / không thường xuyên |
| Concurrent facility failures | Lỗi cơ sở hạ tầng đồng thời |
| Archiving | Lưu trữ dài hạn |
| Retrieval cost / time | Chi phí / thời gian truy xuất |
| Expedited / Standard / Bulk | Cấp tốc / Tiêu chuẩn / Số lượng lớn |
| Min. Storage Duration Charge | Phí thời gian lưu trữ tối thiểu |
| Min. Billable Object Size | Kích thước object tối thiểu bị tính phí |
| Intelligent-Tiering | Phân tầng thông minh |
| Access Tier | Tầng truy cập |
| Auto-tiering | Tự động chuyển tầng |
| Lifecycle configuration | Cấu hình vòng đời |
| Directory Bucket | Bucket trong một AZ duy nhất |
| Single-digit millisecond | Mili-giây một chữ số |
| Co-locate | Đặt cùng vị trí (storage + compute) |
| Latency-sensitive | Nhạy cảm với độ trễ |
| Data-intensive | Thâm dụng dữ liệu |

---

*Ghi chú: các phần Hands On được tóm tắt lại các bước thao tác chính trên AWS Console. Giao diện Console có thể thay đổi theo thời gian — logic và khái niệm vẫn giữ nguyên. ⚠️ Khi dọn dẹp, nhớ **Empty bucket trước khi Delete**; với bucket đã bật Versioning phải **xóa cả các version cũ và delete marker** thì bucket mới thực sự rỗng.*
