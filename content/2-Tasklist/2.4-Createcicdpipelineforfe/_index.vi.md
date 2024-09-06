---
title: "Tạo Pipeline CI/CD để Triển Khai Frontend Tự Động"
date: "`r Sys.Date()`"
weight: 4
chapter: false
pre: " <b> 2.4 </b> "
---

## Giới Thiệu

Trong bước này, chúng ta sẽ thiết lập một pipeline CI/CD sử dụng AWS CodePipeline, CodeBuild và S3. Điều này sẽ cho phép tự động triển khai ứng dụng frontend mỗi khi có một commit mới trong repository.  
Repository: [fcj-ws-fe](https://github.com/dinhhung1598753/fcj-ws-fe)

## Các Bước Thực Hiện

### 1. **Lấy Mã Nguồn Frontend**

Nguồn: [fcj-ws-fe](https://github.com/dinhhung1598753/fcj-ws-fe)

### 2. **Tạo và Cấu Hình Pipeline**

#### a. **Tạo Pipeline trong AWS CodePipeline**

1. **Đi đến AWS Management Console**

   - Điều hướng đến [AWS Management Console](https://aws.amazon.com/console/).
   - Tìm kiếm và chọn **CodePipeline**.

2. **Tạo Một Pipeline**

   - Nhấp vào **Create pipeline**.
     ![code pipeline](/images/2.4/step1.codepipeline.png)
   - Nhập **Tên pipeline**.
   - Nhấp **Next**.

3. **Cấu Hình Giai Đoạn Nguồn**

   - **Nguồn cung cấp**: Chọn `GitHub (Version 1)`.
   - Nhấp **Connect to GitHub** để xác thực và kết nối tài khoản GitHub của bạn.
   - Sau khi kết nối, chọn **Repository** và **Branch** mà bạn muốn sử dụng cho pipeline này.
   - Trong **Tùy chọn phát hiện thay đổi**, chọn `GitHub webhooks` để kích hoạt pipeline mỗi khi có thay đổi được đẩy lên branch.
     ![code pipeline](/images/2.4/step2.codepipeline.png)
   - Nhấp **Next**.

4. **Thêm Giai Đoạn Xây Dựng**

   - **Nhà cung cấp xây dựng**: Chọn `AWS CodeBuild`.
   - **Khu vực**: Chọn cùng khu vực mà bạn muốn triển khai ứng dụng của bạn.
   - **Tên dự án**: Nhấp **Create Project**.

5. **Cấu Hình Dự Án CodeBuild**

   - Nhập **Tên dự án xây dựng**.
   - Trong phần **Môi trường**, chọn môi trường mặc định.
   - Nhập **Tên vai trò CodeBuild**. Nếu bạn không có, bạn có thể tạo một vai trò mới với các quyền cần thiết.
   - **Cấu hình nâng cao**:
     - Thêm biến môi trường:
       - `REACT_APP_API_BASE_URL`: Đặt giá trị này là URL cơ sở API lấy từ đầu ra CloudFormation của backend của bạn. (truy cập CloudFormation, nhấp vào Stacks, chọn Backend stack `fcj-ws-api-dev`, mở tab Outputs và lấy giá trị cần thiết)
       - `REACT_APP_AUTH_URL`: Đặt giá trị này là URL giao diện được lưu trữ của Cognito lấy từ đầu ra CloudFormation của backend của bạn. (truy cập CloudFormation, nhấp vào Stacks, chọn Backend stack `fcj-ws-api-dev`, mở tab Outputs và lấy giá trị cần thiết)
   - **Buildspec**: Chọn `Use a buildspec file`. Điều này sẽ sử dụng file `buildspec.yml` nằm ở thư mục gốc của mã nguồn frontend của bạn.
   - Nhấp **Continue to CodePipeline**.

6. **Hoàn Thành Cấu Hình Giai Đoạn Xây Dựng**

   - Quay lại cấu hình CodePipeline và nhấp **Next**.

7. **Thêm Giai Đoạn Triển Khai**

   - **Nhà cung cấp triển khai**: Chọn `Amazon S3`.
   - **Artifact đầu vào**: Chọn `BuildArtifact`.
   - **Tên bucket**: Nhập tên của bucket S3 mà bạn đã tạo trong các bước trước (từ bước 2.2).
     ![deploy](/images/2.4/deploy.codepipeline.png)
   - Nhấp **Next**.

8. **Xem Xét và Tạo Pipeline**
   - Xem xét tất cả các cấu hình của bạn và nhấp **Create pipeline**.

### 3. **Theo Dõi và Xác Minh Pipeline**

1. **Kiểm Tra Thực Thi Pipeline**

   - Theo dõi pipeline để đảm bảo nó bắt đầu và chạy đúng cách. Bạn có thể thấy trạng thái của từng giai đoạn trong bảng điều khiển AWS CodePipeline.

2. **Xác Minh Triển Khai**
   - Truy cập vào **bucket S3** mà bạn đã chỉ định trong giai đoạn triển khai.
   - Xác minh rằng các artifact xây dựng mới nhất đã được triển khai thành công.

Bằng cách thực hiện các bước này, bạn sẽ thiết lập một pipeline CI/CD tự động xây dựng và triển khai ứng dụng frontend React của bạn lên S3 mỗi khi có thay đổi được thực hiện trong repository GitHub.
