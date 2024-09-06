+++
title = "Clean Up  resources"
date = "`r Sys.Date()`"
weight = 4
chapter = false
pre = "<b>4. </b>"
+++

## Clean Up resources

To ensure that you do not incur unnecessary charges and to maintain a clean and organized AWS environment, it's important to properly clean up the resources created during this exercise. Follow these steps to delete the resources:

### 1. Empty S3 Bucket

Go to the AWS Management Console, navigate to S3, and select the bucket used for hosting your frontend. To empty the bucket:

- Select the bucket.
- Choose "Empty"
- Confirm the action to delete all objects within the bucket.

### 2. Clean Up Backend resources

In your terminal, navigate to the directory containing your backend project. Run the following command to remove the backend resources:

```bash
serverless remove
```

### 3. Clean Up Frontend resources

In your terminal, navigate to the directory containing your frontend project. Run the following command to remove the frontend resources:

```bash
serverless remove
```

Make sure these commands run successfully.
