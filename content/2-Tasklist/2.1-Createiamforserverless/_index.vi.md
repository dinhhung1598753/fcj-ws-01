---
title: "Chuẩn Bị IAM user cho Serverless"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: " <b> 2.1 </b> "
---


Trong bước này, chúng ta sẽ tạo một IAM user và tạo secret access key để Serverless Framework có thể tương tác với các tài nguyên AWS.

## Các bước thực hiện:

1. **Đăng nhập vào AWS Management Console**  
   Truy cập [AWS Management Console](https://aws.amazon.com/console/) và đăng nhập bằng thông tin xác thực của bạn.

2. **Đi đến Bảng điều khiển IAM**  
   Trong AWS Management Console, tìm kiếm "IAM" trong thanh tìm kiếm và mở Bảng điều khiển IAM.

3. **Nhấp vào Users**  
   Từ menu bên trái, chọn **Users**. Tại đây, bạn sẽ quản lý tất cả các Users trong tài khoản AWS của bạn.

4. **Tạo User**

   - Nhấp vào nút **Create User**.
   - Nhập **User name** (ví dụ: `serverless-deployer`).
   - Chọn **Access type** là "Programmatic access" để tạo **Access key** và **Secret access key** mà Serverless Framework sẽ sử dụng.

5. **Cấp quyền cho User**

   - Trong phần "Set permissions", chọn **Attach Policies Directly**.
   - Để kiểm soát chi tiết hơn, chúng ta sẽ tạo một chính sách tùy chỉnh (Custom policy) trong bước tiếp theo.

6. **Tạo Chính sách**

   - Trong phần "Attach policies", nhấp vào **Create policy**.

7. **Chọn JSON trong trình chỉnh sửa Policy**

   - Trong trình hướng dẫn tạo chính sách, chuyển sang tab **JSON** trong trình chỉnh sửa chính sách.

8. **Thêm quyền cho chính sách tuỳ chỉnh**  
    Sao chép và dán đoạn JSON sau vào trình chỉnh sửa, điều này cấp quyền cần thiết để quản lý các tài nguyên. Bạn có thể tùy chỉnh chính sách này dựa trên các dịch vụ bạn sẽ sử dụng.

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Sid": "Statement1",
         "Effect": "Allow",
         "Action": [
           "cloudformation:*",
           "lambda:*",
           "s3:*",
           "logs:*",
           "dynamodb:*",
           "events:*",
           "apigateway:*",
           "iam:GetRole",
           "iam:CreateRole",
           "iam:*",
           "cognito-idp:*",
           "cloudfront:*"
         ],
         "Resource ": "*"
       }
     ]
   }
   ```

9. **Xem lại và Tạo chính sách**

   - Nhấp vào **Next: Review**.
   - Cung cấp một **Tên** cho chính sách của bạn (ví dụ: `ServerlessDeploymentPolicy`) và xem lại các quyền.
   - Nhấp vào **Create policy** để hoàn tất.

10. **Gán chính sách**

- Sau khi tạo chính sách, quay lại màn hình tạo người dùng.
- Trong phần "Attach the New Policy", tìm kiếm chính sách mà bạn vừa tạo (ví dụ: `ServerlessDeploymentPolicy`).
- Chọn chính sách từ danh sách và nhấp vào **Next**.

11. **Hoàn tất tạo User**

- Xem lại các chi tiết và quyền đã gán cho IAM user.
- Nhấp vào **Create User** để hoàn tất việc tạo người dùng.

12. **Tải về các Khóa Truy Cập**

- Sau khi tạo thành công người dùng IAM, bạn cần tạo và tải về các khóa truy cập:

  1.  **Nhấp vào IAM User**  
      Trong bảng điều khiển IAM, nhấp vào người dùng bạn vừa tạo từ danh sách người dùng.

  2.  **Chuyển đến Tab Security Credentials**  
      Đi đến tab **Thông tin bảo mật**.

  3.  **Tạo khóa truy cập**

      - Cuộn xuống phần **Access key**.
      - Nhấp vào nút **Create access key**.

  4.  **Chọn Application Type**

      - Trong "Create access key", chọn **Application running outside AWS**.
      - Nhấp vào **Next** để tiếp tục.

  5.  **Tải về khóa truy cập**

      - Khi khóa truy cập được tạo, bạn sẽ thấy **Access key ID** và **Secret access key**.
      - **Download** thông tin đăng nhập bằng cách nhấp vào nút **Download .csv**, hoặc sao chép thủ công **Access key ID** và **Secret access key**.

      **Quan trọng**: Đảm bảo lưu trữ các thông tin đăng nhập này một cách an toàn vì bạn sẽ cần chúng để cấu hình Serverless Framework. **Secret access key** chỉ hiển thị một lần, vì vậy hãy chắc chắn lưu nó ngay lập tức.

- Bạn sẽ sử dụng các thông tin đăng nhập này trong các bước tiếp theo để cấu hình Serverless Framework.

13. **Cấu hình Serverless Framework với thông tin đăng nhập IAM**

- Với phiên bản Serverless Framework 4, bạn không cần cấu hình thông tin đăng nhập qua dòng lệnh nữa. Thay vào đó, bạn có thể thêm chúng trực tiếp vào tệp `.env` của dự án:

  1.  **Mở thư mục dự án của bạn**  
      Điều hướng đến thư mục gốc của dự án Serverless của bạn.

  2.  **Tạo hoặc chỉnh sửa tệp `.env`**  
      Nếu bạn chưa có tệp `.env`, hãy tạo một tệp mới trong thư mục gốc của dự án. Nếu tệp đã tồn tại, mở nó để chỉnh sửa.

  3.  **Thêm thông tin dăng nhập của bạn**  
      Thêm các dòng sau vào tệp `.env`, thay thế `<Your_Access_Key_ID>` và `<Your_Secret_Access_Key>` bằng giá trị thực tế:

      ```plaintext
      AWS_ACCESS_KEY_ID=<Your_Access_Key_ID>
      AWS_SECRET_ACCESS_KEY=<Your_Secret_Access_Key>
      ```

  4.  **Lưu tệp `.env`**  
      Đảm bảo rằng tệp được lưu với thông tin đăng nhập chính xác.

- Serverless Framework sẽ tự động sử dụng các thông tin đăng nhập này cho việc triển khai và các hoạt động khác.
