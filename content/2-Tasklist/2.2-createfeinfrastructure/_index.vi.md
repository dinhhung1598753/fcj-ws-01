---
title: "Tạo Website Tĩnh S3 và CloudFront"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: " <b> 2.2 </b> "
---

## Giới Thiệu

Trong bước này, chúng ta sẽ thiết lập một bucket S3 để lưu trữ một website tĩnh và cấu hình một phân phối CloudFront để phục vụ nội dung qua CDN.

## Các Bước Thực Hiện

1. **Kéo Mã Nguồn Frontend**  
   Nguồn: [fcj-ws-fe](https://github.com/dinhhung1598753/fcj-ws-fe)

2. **Cài Đặt Serverless Framework và Các Phụ Thuộc**

   ```bash
   npm i -g serverless@4.2.5
   ```

3. **Tạo tài nguyên: S3 Bucket và CloudFront**

   - **Tạo Service name và AWS region**

     Hãy thay thế tên dịch vụ bằng tên bạn muốn

     ```yaml
     service: fcj-ws-fe # Your service name

     provider:
       name: aws
       region: ap-southeast-1 # aws region
       stage: dev
     ```

   - **Tạo S3 Bucket và Configuration**

     Thêm cấu hình sau vào tệp `serverless.yml` của bạn để tạo một S3 bucket và cấu hình nó cho việc lưu trữ website tĩnh:

     ```yaml
     # Define the S3 bucket  resource
     S3Bucket:
       Type: AWS::S3::Bucket
       Properties:
         # The bucket name in this example is "fcj-ws-fe-dev", you need to change Bucket name or service name to make Bucket name is unique
         BucketName: ${self:service}-${self:provider.stage}
         WebsiteConfiguration: # Configure the bucket to serve a static website
           IndexDocument: index.html # Default page to serve
           ErrorDocument: index.html # Error page to serve (useful for single-page apps)
         PublicAccessBlockConfiguration: # Disable public access block for bucket policy
           BlockPublicAcls: true
           IgnorePublicAcls: true
           BlockPublicPolicy: false
           RestrictPublicBuckets: false
     ```

   - **Đặt Chính Sách S3 Bucket Để Cho Phép Truy Cập Đọc Công Khai**

     Thêm cấu hình sau để cho phép truy cập đọc công khai vào S3 bucket:

     ```yaml
     # Set the bucket policy to allow public read access
     S3BucketPolicy:
       Type: AWS::S3::BucketPolicy
       Properties:
         Bucket: !Ref S3Bucket
         PolicyDocument:
           Version: "2012-10-17"
           Statement:
             - Sid: PublicReadGetObject
               Effect: Allow
               Principal: "*"
               Action: "s3:GetObject"
               Resource: !Sub "${S3Bucket.Arn}/*"
     ```

   - **Tạo CloudFront Distribution**

     Thêm cấu hình sau để tạo một CloudFront distribution:

     ```yaml
     # Define the CloudFront distribution  resource
     CloudFrontDistribution:
       Type: AWS::CloudFront::Distribution
       Properties:
         DistributionConfig:
           Enabled: true
           Origins:
             # Static website domain of above bucket
             - DomainName: !Sub "${S3Bucket}.s3-website-${self:provider.region}.amazonaws.com"
               Id: S3Origin # Unique ID for this origin in the distribution
               CustomOriginConfig:
                 HTTPPort: 80
                 HTTPSPort: 443
                 OriginProtocolPolicy: http-only
           DefaultCacheBehavior:
             TargetOriginId: S3Origin # Link the default behavior to the S3 origin
             ViewerProtocolPolicy: redirect-to-https # Redirect HTTP requests to HTTPS
             AllowedMethods:
               - GET # Allow GET requests
               - HEAD # Allow HEAD requests
             CachedMethods:
               - GET # Cache GET requests
               - HEAD # Cache HEAD requests
             ForwardedValues:
               QueryString: false # Do not forward query strings to the origin (S3)
               Cookies:
                 Forward: none # Do not forward cookies to the origin (S3)
           # Serve index.html when accessing the root of the CloudFront domain
           DefaultRootObject: index.html
           ViewerCertificate:
             # Use the default CloudFront certificate (*.cloudfront.net)
             CloudFrontDefaultCertificate: true
           # Limit CloudFront to the lowest-cost edge locations (PriceClass_100, PriceClass_200, PriceClass_All)
           PriceClass: PriceClass_100
           HttpVersion: http2 # Use HTTP/2 for better performance
           Comment: "CloudFront distribution for serving S3 static website"
     ```

   - **Tất Cả Cấu Hình Trong serverless.yml**

     ```yaml
     service: fcj-ws-fe

     provider:
       name: aws
       region: ap-southeast-1
       stage: dev

      resources:
        Resources:
         # Define the S3 bucket  resource
         S3Bucket:
           Type: AWS::S3::Bucket
           Properties:
             # Specify the bucket name with a dynamic name based on the service and stage
             BucketName: ${self:service}-${self:provider.stage}
             WebsiteConfiguration: # Configure the bucket to serve a static website
               IndexDocument: index.html # Default page to serve
               ErrorDocument: index.html # Error page to serve (useful for single-page apps)
             PublicAccessBlockConfiguration: # Disable public access block
               BlockPublicAcls: false
               IgnorePublicAcls: false
               BlockPublicPolicy: false
               RestrictPublicBuckets: false

         # Set the bucket policy to allow public read access
         S3BucketPolicy:
           Type: AWS::S3::BucketPolicy
           Properties:
             Bucket: !Ref S3Bucket
             PolicyDocument:
               Version: "2012-10-17"
               Statement:
                 - Sid: PublicReadGetObject
                   Effect: Allow
                   Principal: "*"
                   Action: "s3:GetObject"
                   Resource : !Sub "${S3Bucket.Arn}/*"

         # Define the CloudFront distribution  resource
         CloudFrontDistribution:
           Type: AWS::CloudFront::Distribution
           Properties:
             DistributionConfig:
               Enabled: true # Enable the CloudFront distribution
               Origins:
                 - DomainName: !Sub "${S3Bucket}.s3-website-${self:provider.region}.amazonaws.com" # Use the S3 website endpoint
                   Id: S3Origin # Unique ID for this origin in the distribution
                   CustomOriginConfig:
                     HTTPPort: 80
                     HTTPSPort: 443
                     OriginProtocolPolicy: http-only
               DefaultCacheBehavior:
                 TargetOriginId: S3Origin # Link the default behavior to the S3 origin
                 ViewerProtocolPolicy: redirect-to-https # Redirect HTTP requests to HTTPS
                 AllowedMethods:
                   - GET # Allow GET requests
                   - HEAD # Allow HEAD requests
                 CachedMethods:
                   - GET # Cache GET requests
                   - HEAD # Cache HEAD requests
                 ForwardedValues:
                   QueryString: false # Do not forward query strings to the origin (S3)
                   Cookies:
                     Forward: none # Do not forward cookies to the origin (S3)
               DefaultRootObject: index.html # Serve index.html when accessing the root of the CloudFront domain
               ViewerCertificate:
                 CloudFrontDefaultCertificate: true # Use the default CloudFront certificate (*.cloudfront.net)
               PriceClass: PriceClass_100 # Limit CloudFront to the lowest-cost edge locations
               HttpVersion: http2 # Use HTTP/2 for better performance
               Comment: "CloudFront distribution for serving S3 static website" # Description of the distribution

       Outputs:
         # Output the CloudFront domain name
         CloudFrontDomain:
           Value:
             Fn::GetAtt: [CloudFrontDistribution, DomainName] # Get the domain name of the CloudFront distribution
     ```

4. **Triển Khai Hạ Tầng**

   Nhớ thêm tệp `.env` chứa `AWS_ACCESS_KEY_ID` và `AWS_SECRET_ACCESS_KEY` mà chúng ta đã tạo trong bước trước vào thư mục gốc của dự án.

   Triển khai các tài nguyên được định nghĩa trong tệp `serverless.yml` bằng cách sử dụng CLI của Serverless:

   ```bash
   serverless deploy
   ```
