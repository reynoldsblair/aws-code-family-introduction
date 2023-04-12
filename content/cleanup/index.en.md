---
title : "Cleanup"
weight : 900
---

Follow the below steps to cleanup up any resources created in your AWS account. If you running at an AWS event then you do not need to follow this as accounts will be terminated automatically.

### Delete CloudFormation stack
1. Log into the AWS Console and browse to the CloudFormation service.

2. Select the **UnicornStack** and click the **Delete** button followed by **Delete stack**. This will delete the VPC, subnets and EC2 instances.

### Delete the Cloud9 environment
1. Log into the AWS Console and browse to the Cloud9 service.

2. Click the **UnicornIDE** and click the **Delete** button and confirm the deletion.


### Delete CodePipeline
1. Log into the AWS Console and browse to the CodePipeline service.

2. Select **unicorn-web-pipeline**, click the **Delete pipeline** button and confirm the deletion.

### Delete the CodeCommit repository
1. Log into the AWS Console and browse to the CodeCommit service.

2. Select repositories, click the **unicorn-web-project**, click the **Delete repository** button and confirm the deletion.

### Delete the CodeArtifact repository and domain
1. Log into the AWS Console and browse to the CodeArtifact service.

2. Select **Repositories**, select **maven-central-store**, click the **Delete** button and confirm the deletion.

3. Repeat the last step for the **unicorn-packages** repository.

4. Select **Domains**, select **unicorn**, click the **Delete** button and confirm the deletion.

### Delete CodeBuild projects
1. Log into the AWS console and browse to the CodeBuild service.

2. Select **Build projects** and delete the **unicorn-web-build** project.

### Delete CodeDeploy application
1. Log into the AWS console and browse to the CodeDeploy service.

2. Select **Applications** and view details on the **unicorn-web-deploy** application.

3. Click the **Delete application** button and confirm the deletion.

### Delete SNS topic and subscription
1. Log into the AWS Console and browse to the SNS service.

2. Select **Topics** and then delete the **unicorn-pipeline-notifications** topic.

3. Select **Subscriptions** on the left-hand menu. Select your email address, click **Delete** and confirm the deletion.

### Delete artifact S3 bucket
1. Log into the AWS Console and browse to the S3 service.

2. Select your **unicorn-build-artifacts** bucket, click **Empty** and confirm the dialog.

3. Select the bucket again, click **Delete** and confirm once more.

### Empty bucket created by CodePipeline
1. Still in the S3 console, search for a bucket with a name similar to **codepipeline-\<region\>-\<accountid/>**.

2. Select this bucket, click **Empty** and confirm. You do not need to delete this bucket.

### Delete IAM Policies
1. Log into the AWS Console and browse to the IAM service.

2. Select **Policies** and then filter for **unicorn**.

3. Select **codeartifact-unicorn-consumer-policy**, click **Actions > Delete** and confirm the deletion.

4. Do the same for all policies that have **unicorn-web** in their name.

### Delete IAM Roles
1. Still in the IAM console, select **Roles** in the menu.

2. Filter for **unicorn**.

3. Select **UnicornCodeDeployRole**, click **Delete** and confirm the deletion.

4. Do the same for all policies that have **unicorn-web** in their name.

### Delete CloudWatch Logs group
1. Log into the AWS Console and browse to the CloudWatch Service.

2. Select **Log groups** from the left-hand link panel.

3. Select the **unicorn-build-logs** group, select **Actions > Delete log group(s)** and confirm the deletion.

# Cleanup for Additional Exercises

If you completed any of the [additional, optional exercises](../additional-exercises/), please follow the instructions below to clean up the related resources.

### Clean up for [Container build](../additional-exercises/ecsdeploy/)

1. Log into the AWS Console and browse to the Amazon ECR service.

2. Select the **unicorn-web-images** repository, select **Delete** and confirm the deletion.

3. Browse to the CodeBuild service.

2. Select **Build projects** and delete the **unicorn-web-build-container** project.

### Clean up for [Serverless deploy](../additional-exercises/serverlessdeploy/)

#### Delete CloudFormation Stacks

1. Log into the AWS Console and browse to the Amazon CloudFormation service.

2. Delete the following stacks. Do so in the correct order, waiting for each Stack deletion to complete before continuing.

- serverless-unicorns-prod
- serverless-unicorns-uat
- serverless-pipeline
- aws-sam-cli-managed-prod-pipeline-resources
- aws-sam-cli-managed-uat-pipeline-resources
- serverless-unicorns
- aws-sam-cli-managed-default

#### Delete the CodeCommit repository

1. Log into the AWS Console and browse to the CodeCommit service.

2. Select repositories, click the **serverless-unicorns**, click the **Delete repository** button and confirm the deletion.