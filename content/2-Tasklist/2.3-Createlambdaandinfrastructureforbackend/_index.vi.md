---
title: "Tạo tài nguyên cho backend "
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: " <b> 2.3 </b> "
---

## Giới Thiệu

Trong bước này, chúng ta sẽ thiết lập DynamoDB, API Gateway, Cognito, và Lambda functions.  
Repository: [fcj-ws-api](https://github.com/dinhhung1598753/fcj-ws-api)

## Các Bước Thực Hiện

1. **Tải mã nguồn Backend**  
   Source: [fcj-ws-api](https://github.com/dinhhung1598753/fcj-ws-api)

2. **Cài đặt Serverless Framework**

   Bạn đã cài đặt ở bước 2.2. Không cần thực hiện lại.

3. **Cấu trúc thư mục dự án**

   Trong phần này, chúng tôi sẽ mô tả cấu trúc của thư mục dự án và giải thích mục đích của từng thư mục và tệp.

   Thư mục dự án `fcj-ws-api` được tổ chức như sau:

   - **` resources/`**: Chứa các mẫu CloudFormation và tệp cấu hình để định nghĩa các tài nguyên AWS.

     - **`iam/`**: Định nghĩa các vai trò IAM.
       - `lambdaRole.yml`: Xác định vai trò IAM và các chính sách cho các hàm Lambda.
     - **`lambda/`**: Định nghĩa các hàm Lambda và cấu hình của chúng.
     - **`cognito.yml`**: Định nghĩa cấu hình cho AWS Cognito User Pool và các thiết lập liên quan.
     - **`dynamo.yml`**: Định nghĩa cấu hình cho các bảng DynamoDB.

   - **`src/`**: Chứa mã nguồn cho các hàm Lambda.

   - **`serverless.yml`**: Tệp cấu hình chính cho Serverless Framework, định nghĩa dịch vụ, nhà cung cấp, các hàm và tài nguyên.

   - **`package.json`**: Chứa các phụ thuộc của dự án, các script và metadata.

   ***

4. **Cấu hình chung của dự án**

   ```yaml
   service: fcj-ws-api # Service name

    custom:
      base: ${self:service}-${self:provider.stage}
      dynamo:
        TodoTable: ${self:service}-todo-${sls:stage}
      iam:
        LambdaRole:
          name: ${self:custom.base}-lambda-role
      cognito:
        UserPoolName: ${self:custom.base}-user-pool
        UserPoolClient: ${self:custom.base}-user-pool-client
        Domain: ${self:custom.base}-auth

    build: # Config to use ES6
        esbuild:
          # Enable or Disable bundling the function code and dependencies. (Default: true)
          bundle: true
          # Enable minifying function code. (Default: false)
          minify: false

    provider:
      name: aws
      runtime: nodejs20.x # Define lambda runtime environment
      region: ap-southeast-1 # Define AWS region
      stage: dev
      httpApi:  # Config API Gateway v2
        cors: # Config CORS
          allowedOrigins: '*'
          allowedHeaders: '*'
          allowedMethods:
            - GET
            - POST
            - PUT
            - DELETE
            - OPTIONS
          maxAge: 6000 #
        authorizers: # Config Authorization
          CognitoAuthorizer:
            type: jwt
            identitySource: $request.header.Authorization
            issuerUrl:
              Fn::Sub: https://cognito-idp.${self:provider.region}.amazonaws.com/${CognitoUserPool}
            audience:
              - Ref: CognitoUserPoolClient
      environment: # Config env variable
        TODO_TABLE: ${self:custom.dynamo.TodoTable}
   ```

5. **Cấu hình quyền cho lambda**

   Trong phần này, chúng ta định nghĩa IAM roles cho các hàm Lambda. Vai trò này cấp quyền cần thiết cho các hàm Lambda để tương tác với các dịch vụ AWS như DynamoDB.

   Dưới đây là định nghĩa tài nguyên cho `LambdaRole` trong ` resources:`.

   ```yaml
   resources:
     Resources:
       LambdaRole:
         Type: AWS::IAM::Role
         Properties:
           RoleName: ${self:custom.iam.LambdaRole.name}
           AssumeRolePolicyDocument:
             Version: "2012-10-17"
             Statement:
               - Effect: Allow
                 Principal:
                   Service:
                     - lambda.amazonaws.com
                 Action: sts:AssumeRole
           ManagedPolicyArns:
             - arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
           Policies:
             - PolicyName: lambda-permissions
               PolicyDocument:
                 Version: "2012-10-17"
                 Statement:
                   - Effect: "Allow"
                     Action:
                       - "dynamodb:PutItem"
                       - "dynamodb:Get*"
                       - "dynamodb:Scan*"
                       - "dynamodb:UpdateItem"
                       - "dynamodb:DeleteItem"
                     Resource:
                       - arn:aws:dynamodb:${aws:region}:${aws:accountId}:table/${self:custom.dynamo.TodoTable}
   ```

6. **DynamoDB Table**

   Trong phần này, chúng tôi định nghĩa một tài nguyên bảng DynamoDB tên là `TodoTable`. Bảng này sẽ được sử dụng để lưu trữ dữ liệu người dùng và sẽ được cấu hình với `id` làm khóa phân vùng và username làm khóa sắp xếp.

   Dưới đây là định nghĩa tài nguyên cho bảng DynamoDB trong ` resources`:.

   ```yaml
   resources:
     Resources:
       TodoTable:
         Type: AWS::DynamoDB::Table
         Properties:
           TableName: ${self:custom.dynamo.TodoTable} # Get table name from custom defined above
           AttributeDefinitions:
             - AttributeName: id
               AttributeType: S
             - AttributeName: username
               AttributeType: S
           KeySchema:
             - AttributeName: id # Define partition key
               KeyType: HASH
             - AttributeName: username # Define sort key
               KeyType: RANGE
           BillingMode: PAY_PER_REQUEST # Define billing mode (on demand)
   ```

7. **Cấu hình Cognito**

   Trong phần này, chúng tôi sẽ cấu hình Amazon Cognito User Pool và User Pool Client, sẽ xử lý xác thực và quản lý người dùng trong ứng dụng. Cognito giúp quản lý đăng ký người dùng, đăng nhập và kiểm soát truy cập.

   Dưới đây là cấu hình cho Cognito:

   ```yaml
    resources:
      Resources:
       # Cognito User Pool: This creates the Cognito User Pool for handling user authentication.
       CognitoUserPool:
         Type: AWS::Cognito::UserPool
         Properties:
           # Name of the User Pool (from custom variables).
           UserPoolName: ${self:custom.cognito.UserPoolName}
           # Users will log in using their email address.
           UsernameAttributes:
             - email
           # Automatically verify users based on their email.
           AutoVerifiedAttributes:
             - email
           # Define recovery settings (e.g., recovering accounts via verified email).
           AccountRecoverySetting:
             RecoveryMechanisms:
               - Priority: 1
                 Name: "verified_email"
           # Define the schema for the User Pool; in this case, we only require an email.
           Schema:
             - Name: email
               Required: true

       # Cognito User Pool Client: This creates a client that will allow the front-end to authenticate with Cognito.
       CognitoUserPoolClient:
         Type: AWS::Cognito::UserPoolClient
         Properties:
           # Name of the User Pool Client (from custom variables).
           ClientName: ${self:custom.cognito.UserPoolClient}
           # The User Pool this client is associated with.
           UserPoolId:
             Ref: CognitoUserPool
           # Allow only admin authentication flow (no SRP authentication method).
           ExplicitAuthFlows:
             - ADMIN_NO_SRP_AUTH
           # No need to generate a secret key for this client.
           GenerateSecret: false
           # Enable OAuth 2.0 implicit flow for authentication.
           AllowedOAuthFlows:
             - implicit
           # Explicitly allow this client to use OAuth flows.
           AllowedOAuthFlowsUserPoolClient: true
           # Define the OAuth scopes that are allowed for this client.
           AllowedOAuthScopes:
             - email
             - openid
           # URL to redirect users to after successful login (from CloudFront distribution output).
           CallbackURLs:
             # CloudFrontDomain is got from CloudFormation outputs of Frontend (2.2)
             # Replace with your app's callback URL if you update service name in serverless.yml of frontend project
             - https://${cf:fcj-ws-fe-${self:provider.stage}.CloudFrontDomain}/callback
           # URL to redirect users to after they log out of the app.
           LogoutURLs:
             # CloudFrontDomain is got from CloudFormation outputs of Frontend (2.2)
             # Replace with your app's signout URL if you update service name in serverless.yml of frontend project
             - https://${cf:fcj-ws-fe-${self:provider.stage}.CloudFrontDomain}/login
           # Enable enhanced error handling to prevent user existence errors during login.
           PreventUserExistenceErrors: ENABLED
           # Supported identity provider for this client (in this case, it's Cognito itself).
           SupportedIdentityProviders:
             - COGNITO

       # Cognito User Pool Domain: Set up a custom domain for the Cognito hosted UI.
       CognitoUserPoolDomain:
         Type: AWS::Cognito::UserPoolDomain
         Properties:
           # Define a custom domain name for the Cognito hosted UI (from custom variables).
           Domain: ${self:custom.cognito.Domain}
           # The User Pool to associate with this domain.
           UserPoolId:
             Ref: CognitoUserPool

     Outputs:
       # Output the User Pool ID.
       cognitoUserPoolId:
         Value:
           Ref: CognitoUserPool

       # Output the User Pool Client ID.
       cognitoUserPoolClientId:
         Value:
           Ref: CognitoUserPoolClient

       # Output the custom domain name for the hosted UI.
       cognitoUserPoolDomain:
         Value:
           Ref: CognitoUserPoolDomain

       # Output the full URL for the Cognito Hosted UI login page.
       cognitoHostedUIUrl:
         Value:
           # This generates the complete URL for the Cognito Hosted UI login page.
           # It includes the User Pool Domain, Client ID, OAuth response type, scopes, and the CloudFront callback URL.
           Fn::Sub: "https://${CognitoUserPoolDomain}.auth.${AWS::Region}.amazoncognito.com/login?client_id=${CognitoUserPoolClient}&response_type=token&scope=email+openid&redirect_uri=https://${cf:fcj-ws-fe-${self:provider.stage}.CloudFrontDomain}/callback"
   ```

8. **Cấu hình API**

   Trong phần này, chúng ta sẽ giải thích chi tiết cấu hình của một hàm Lambda cụ thể, bao gồm handler, tên hàm, IAM role, và các sự kiện kích hoạt.

   ```yaml
   # Code for this lambda function
   handler: src/createTodo/handler.createTodo
   # Lambda function name
   name: ${self:custom.base}-create-todo
   # Lambda function execution role
   role: !GetAtt LambdaRole.Arn
   # Event to trigger function
   events:
     - httpApi: # API gateway v2
         path: /todos
         method: post
         authorizer: # Config authorization to use the cognito authorization which was defined above
           name: CognitoAuthorizer
   ```

   Để xem các hàm khác, vui lòng xem trong repository.

9. **Kết hợp tất cả cấu hình**

   ```yaml
   service: fcj-ws-api

   custom:
     base: ${self:service}-${self:provider.stage}
     dynamo:
       TodoTable: ${self:service}-todo-${sls:stage}
     iam:
       LambdaRole:
         name: ${self:custom.base}-lambda-role
     cognito:
       UserPoolName: ${self:custom.base}-user-pool
       UserPoolClient: ${self:custom.base}-user-pool-client
       Domain: ${self:custom.base}-auth

   build:
     esbuild:
       # Enable or Disable bundling the function code and dependencies. (Default: true)
       bundle: true
       # Enable minifying function code. (Default: false)
       minify: false

   provider:
     name: aws
     runtime: nodejs20.x
     region: ap-southeast-1
     stage: dev
     httpApi:
       cors:
         allowedOrigins: "*"
         allowedHeaders: "*"
         allowedMethods:
           - GET
           - POST
           - PUT
           - DELETE
           - OPTIONS
         maxAge: 6000 #
       authorizers:
         CognitoAuthorizer:
           type: jwt
           identitySource: $request.header.Authorization
           issuerUrl:
             Fn::Sub: https://cognito-idp.${self:provider.region}.amazonaws.com/${CognitoUserPool}
           audience:
             - Ref: CognitoUserPoolClient
     environment:
       TODO_TABLE: ${self:custom.dynamo.TodoTable}

   functions:
     createTodo: ${file(./resources/lambda/createTodo.yml)}
     getTodos: ${file(./resources/lambda/getTodos.yml)}
     getTodo: ${file(./resources/lambda/getTodo.yml)}
     deleteTodo: ${file(./resources/lambda/deleteTodo.yml)}
     updateTodo: ${file(./resources/lambda/updateTodo.yml)}

    resources:
     - ${file(./resources/iam/lambdaRole.yml):resources}
     - ${file(./resources/dynamo.yml):resources}
     - ${file(./resources/cognito.yml):resources}
   ```

10. **Triển khai**
    Hãy nhớ thêm tệp .env chứa `AWS_ACCESS_KEY_ID` và `AWS_SECRET_ACCESS_KEY` mà chúng ta đã tạo ở bước trước vào thư mục gốc của dự án.

    Triển khai các tài nguyên đã được định nghĩa trong tệp `serverless.yml` của bạn bằng cách sử dụng Serverless CLI:

    ```bash
    serverless deploy
    ```
 