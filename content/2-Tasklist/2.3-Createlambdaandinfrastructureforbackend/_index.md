---
title: "Create resource and lambda functions for backend"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: " <b> 2.3 </b> "
---

## Introduction

In this step, we will set up DynamoDB, API Gateway, Cognito, and Lambda functions.  
Repository: [fcj-ws-api](https://github.com/dinhhung1598753/fcj-ws-api)

## Steps to Follow

1. **Pull Backend Source**  
   Source: [fcj-ws-api](https://github.com/dinhhung1598753/fcj-ws-api)

2. **Install serverless framework and dependencies**

   You have installed in step 2.2. No need to perform again.

3. **Project Directory Structure**

   In this section, we'll outline the structure of the project directory and explain the purpose of each folder and file.

   The project directory `fcj-ws-api` is organized as follows:

   - **` resources/`**: Contains CloudFormation templates and configuration files for defining AWS resources.

     - **`iam/`**: Defines IAM roles.
       - `lambdaRole.yml`: Specifies the IAM role and policies for Lambda functions.
     - **`lambda/`**: Defines Lambda functions and their configurations.
     - **`cognito.yml`**: Defines the configuration for AWS Cognito User Pool and related settings.
     - **`dynamo.yml`**: Defines the configuration for DynamoDB tables.

   - **`src/`**: Contains the source code for Lambda functions.

   - **`serverless.yml`**: The main configuration file for the Serverless Framework, defining the service, provider, functions, and resources.

   - **`package.json`**: Contains the project's dependencies, scripts, and metadata.

   ***

4. **Project Config**

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

5. **Config lambda role**

   In this section, we define the IAM Role for Lambda functions. This role gives the Lambda functions the necessary permissions to interact with AWS services like DynamoDB.

   The following is the resource definition for the `LambdaRole` in ` resources:`.

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

   In this section, we define a DynamoDB table resource named `TodoTable`. This table will be used to store user data and will be configured with `id` as the partition key and `username` as the sort key.

   The following is the resource definition for the DynamoDB table in ` resources:`.

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

7. **Cognito Configuration**

   In this section, we will configure an Amazon Cognito User Pool and User Pool Client, which will handle authentication and user management in the application. Cognito helps manage user sign-up, sign-in, and access control.

   The following is the configuration for Cognito:

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

8. **API Configuration**

   In this section, we will explain the detailed configuration of a specific Lambda function, including the handler, function name, IAM role, and event triggers.

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

   For other functions, please look into repository.

9. **Combine all config**

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

10. **Deploy**
    Remember add the .env AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY we create in previous step with file to root level of the project

    Deploy the resources defined in your `serverless.yml` file using the Serverless CLI:

    ```bash
    serverless deploy
    ```
