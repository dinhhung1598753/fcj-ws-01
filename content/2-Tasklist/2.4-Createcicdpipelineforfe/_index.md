---
title: "Create CI/CD Pipeline to Automatically Deploy Frontend"
date: "`r Sys.Date()`"
weight: 4
chapter: false
pre: " <b> 2.4 </b> "
---

## Introduction

In this step, we will set up a CI/CD pipeline using AWS CodePipeline, CodeBuild, and S3. This will enable automatic deployment of the frontend application whenever there is a new commit in the repository.  
Repository: [fcj-ws-fe](https://github.com/dinhhung1598753/fcj-ws-fe)

## Steps to Follow

### 1. **Pull Frontend Source**  
   Source: [fcj-ws-fe](https://github.com/dinhhung1598753/fcj-ws-fe)

### 2. **Create and Configure the Pipeline**

#### a. **Create Pipeline in AWS CodePipeline**

1. **Go to AWS Management Console**  
   - Navigate to the [AWS Management Console](https://aws.amazon.com/console/).
   - Search for and select **CodePipeline**.

2. **Create a Pipeline**  
   - Click on **Create pipeline**.
    ![code pipeline](/images/2.4/step1.codepipeline.png)
   - Enter a **Pipeline name**.
   - Click **Next**.

3. **Configure Source Stage**  
   - **Source provider**: Select `GitHub (Version 1)`.
   - Click **Connect to GitHub** to authenticate and connect your GitHub account.
   - Once connected, select the **Repository** and **Branch** that you want to use for this pipeline.
   - In **Change detection options**, choose `GitHub webhooks` to trigger the pipeline whenever changes are pushed to the branch.
    ![code pipeline](/images/2.4/step2.codepipeline.png)
   - Click **Next**.

4. **Add Build Stage**  
   - **Build provider**: Select `AWS CodeBuild`.
   - **Region**: Choose the same region where you want to deploy your application.
   - **Project name**: Click **Create Project**.

5. **Configure CodeBuild Project**  
   - Enter a **Build project name**.
   - In the **Environment** section, select the default environment.
   - Enter the **CodeBuild role name**. If you don't have one, you can create a new role with the necessary permissions.
   - **Advanced configuration**:
     - Add environment variables:
       - `REACT_APP_API_BASE_URL`: Set this to the API base URL obtained from your backend CloudFormation output. (go to CloudFormation, click Stacks, choose Backend stack `fcj-ws-api-dev`, open Outputs tab and get the needed value)
       - `REACT_APP_AUTH_URL`: Set this to the Cognito hosted UI URL obtained from your backend CloudFormation output. (go to CloudFormation, click Stacks, choose Backend stack `fcj-ws-api-dev`, open Outputs tab and get the needed value)
   - **Buildspec**: Choose `Use a buildspec file`. This will use the `buildspec.yml` file located in the root of your frontend source directory.
   - Click **Continue to CodePipeline**.

6. **Complete Build Stage Configuration**  
   - Go back to the CodePipeline configuration and click **Next**.

7. **Add Deploy Stage**  
   - **Deploy provider**: Select `Amazon S3`.
   - **Input artifact**: Choose `BuildArtifact`.
   - **Bucket name**: Enter the name of the S3 bucket you created in the previous steps (from step 2.2).
    ![deploy](/images/2.4/deploy.codepipeline.png)
   - Click **Next**.

8. **Review and Create Pipeline**  
   - Review all your configurations and click **Create pipeline**.

### 3. **Monitor and Verify Pipeline**

1. **Check Pipeline Execution**  
   - Monitor the pipeline to ensure it starts and runs correctly. You can see the status of each stage in the AWS CodePipeline console.

2. **Verify Deployment**  
   - Go to the **S3 bucket** you specified in the deploy stage.
   - Verify that the latest build artifacts have been deployed successfully.

By following these steps, you will set up a CI/CD pipeline that automatically builds and deploys your React frontend application to S3 whenever changes are made to the GitHub repository.
