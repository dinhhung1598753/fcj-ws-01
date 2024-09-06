---
title: "Preparing IAM user for Serverless"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: " <b> 2.1 </b> "
---

# Preparing IAM User for Serverless

In this step, we will create an IAM user and generate secret access keys for the Serverless Framework to interact with AWS resources.

## Steps to follow:

1. **Login to the AWS Management Console**  
   Go to [AWS Management Console](https://aws.amazon.com/console/) and sign in with your credentials.

2. **Go to IAM Dashboard**  
   In the AWS Management Console, search for "IAM" in the search bar and open the IAM Dashboard.

3. **Click on Users**  
   From the left-hand side menu, select **Users**. Here, you'll manage all the users in your AWS account.

4. **Create User**

   - Click on the **Add user** button.
   - Enter a **User name** (e.g., `serverless-deployer`).
   - Select the **Access type** as "Programmatic access" to generate the **Access key** and **Secret access key** that the Serverless Framework will use.

5. **Attach Policies Directly**

   - In the "Set permissions" section, choose **Attach policies directly**.
   - For more granular control, we'll create a custom policy in the next step.

6. **Create Policy**

   - In the "Attach policies" section, click on **Create policy**.

7. **Choose JSON in the Policy Editor**

   - In the policy creation wizard, switch to the **JSON** tab in the policy editor.

8. **Add Policy Permissions**  
    Copy and paste the following JSON into the editor, which grants the necessary permissions to manage needed resources. You can customize this policy based on the services you'll be using.

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

9. **Review and Create Policy**

   - Click **Next: Review**.
   - Provide a **Name** for your policy (e.g., `ServerlessDeploymentPolicy`) and review the permissions.
   - Click **Create policy** to finalize.

10. **Attach the New Policy**

- After creating the policy, return to the user creation screen.
- In the "Attach policies" section, search for the policy you just created (e.g., `ServerlessDeploymentPolicy`).
- Select the policy from the list and click **Next**.

11. **Complete User Creation**

- Review the IAM user’s details and attached permissions.
- Click **Create user** to finish creating the user.

12. **Download the Access Keys**

- After successfully creating the IAM user, you need to generate and download the access keys:

  1.  **Click on the IAM User**  
      In the IAM Dashboard, click on the user you just created from the list of users.

  2.  **Navigate to the Security Credentials Tab**  
      Go to the **Security credentials** tab.

  3.  **Create an Access Key**

      - Scroll down to the **Access keys** section.
      - Click on the **Create access key** button.

  4.  **Choose Application Type**

      - In the "Create access key" wizard, select **Application running outside AWS**.
      - Click **Next** to proceed.

  5.  **Download the Access Key**

      - Once the access key is created, you will see the **Access key ID** and **Secret access key**.
      - **Download** the credentials by clicking the **Download .csv** button, or manually copy the **Access key ID** and **Secret access key**.

      **Important**: Make sure to store these credentials securely as you will need them to configure the Serverless Framework. The **Secret access key** will only be shown once, so be sure to save it immediately.

- You will use these credentials in the next steps to configure the Serverless Framework.

13. **Configure Serverless Framework with IAM User Credentials**

- With Serverless Framework version 4, you no longer need to configure credentials via command line. Instead, you can add them directly to your project’s `.env` file:

  1.  **Open Your Project Directory**  
      Navigate to the root directory of your Serverless project.

  2.  **Create or Edit the `.env` File**  
      If you don’t have a `.env` file already, create one in the root directory of your project. If it exists, open it for editing.

  3.  **Add Your Credentials**  
      Add the following lines to the `.env` file, replacing `<Your_Access_Key_ID>` and `<Your_Secret_Access_Key>` with the actual values:

      ```plaintext
      AWS_ACCESS_KEY_ID=<Your_Access_Key_ID>
      AWS_SECRET_ACCESS_KEY=<Your_Secret_Access_Key>
      ```

  4.  **Save the `.env` File**  
      Ensure the file is saved with the correct credentials.

- The Serverless Framework will automatically use these credentials for deployments and other operations.
