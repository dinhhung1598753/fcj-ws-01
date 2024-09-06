+++
title = "Dọn Dẹp Tài Nguyên"
date = "`r Sys.Date()`"
weight = 4
chapter = false
pre = "<b>4. </b>"
+++

## Dọn Dẹp Tài Nguyên

Để đảm bảo rằng bạn không phát sinh các khoản phí không cần thiết và duy trì một môi trường AWS sạch sẽ và có tổ chức, việc dọn dẹp tài nguyên đã tạo ra trong quá trình này là rất quan trọng. Hãy làm theo các bước sau để xóa các tài nguyên:

### 1. Xóa S3 Bucket

Đi đến AWS Management Console, điều hướng đến S3, và chọn bucket được sử dụng để lưu trữ frontend của bạn. Để xóa bucket:

- Chọn bucket.
- Vào menu "Actions" và chọn "Empty"
- Xác nhận hành động để xóa tất cả các đối tượng trong bucket.

### 2. Dọn Dẹp Tài Nguyên Backend

Trong terminal, điều hướng đến thư mục chứa dự án backend của bạn. Chạy lệnh sau để xóa các tài nguyên backend:

```bash
serverless remove
```

### 3. Dọn Dẹp Tài Nguyên Frontend

Trong terminal, điều hướng đến thư mục chứa dự án frontend của bạn. Chạy lệnh sau để xóa các tài nguyên frontend:

```bash
serverless remove
```

Đảm bảo rằng các lệnh này chạy thành công.
