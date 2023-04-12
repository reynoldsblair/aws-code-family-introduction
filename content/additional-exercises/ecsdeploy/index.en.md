---
title : "Container build"
weight : 81
---

In the previous labs, we have focused on a simple Java web application being deployed directly on an EC2 instance.

We will now look at modifying our pipeline to build a container image using CodeBuild and push this to Amazon Elastic Container Registry (ECR).

![Container Diagram](/static/container-diagram.png)

### Add files for docker build
First, we need to add a Dockerfile which will contain instructions on how to build our container image. 

1. Log back into the Cloud9 IDE.

2. Create a new file named **Dockerfile** in the **~/environment/unicorn-web-project/** folder and add the following lines:
:::code{language=docker showLineNumbers=false showCopyAction=true}
FROM public.ecr.aws/bitnami/tomcat:9.0

COPY ./target/unicorn-web-project.war /usr/local/tomcat/webapps/ROOT.war
:::

This is a very simple install as we are using the Bitnami [Apache Tomcat image](https://gallery.ecr.aws/bitnami/tomcat) from the Amazon ECR Public Gallery.

3. Create a new file named **buildspec_docker.yml** in the same folder and add the following lines:
:::code{language=yaml showLineNumbers=false showCopyAction=true}
version: 0.2

phases:
  pre_build:
    commands:
      - echo Logging in to Amazon ECR...
      - aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com
  build:
    commands:
      - echo Build started on `date`
      - echo Building the Docker image...          
      - docker build -t $IMAGE_REPO_NAME:$IMAGE_TAG .
      - docker tag $IMAGE_REPO_NAME:$IMAGE_TAG $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG      
  post_build:
    commands:
      - echo Build completed on `date`
      - echo Pushing the Docker image...
      - docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG  
:::

This new buildspec file will build our container image and push it to an ECR repository.

4. Modify the existing **buildspec.yml** file to include the additional artifact files (**Dockerfile** and **buildspec_docker.yml**). This CodeBuild job will run before the buildspec-docker job, so we need to pass the files downstream.
:::code{language=yaml showLineNumbers=false showCopyAction=false}
version: 0.2

phases:
  [..]
  
artifacts:
  files:
    - target/unicorn-web-project.war
    - appspec.yml
    - scripts/**/*
    - Dockerfile
    - buildspec_docker.yml
  discard-paths: no
:::

5. Now commit the changes to CodeCommit
:::code{language=bash showLineNumbers=false showCopyAction=true}
git add *
git commit -m "Adding container support"
git push -u origin main
:::

### Create an Amazon ECR repository
We need somewhere to store our container images and this is what we will use Amazon ECR (Elastic Container Registry) for.

1. Log into the AWS Console and search for the **Elastic Container Registry** console.

2. Under Private repositories click **Create repository**. Note: if you see a Getting Started page instead, click **Get started**.

3. Name the repository **unicorn-web-images** leaving the other settings as default. Click **Create repository**

### Create a new CodeBuild build project
Now we need to create a new CodeBuild project which will use our buildspec_docker.yml file to build our container image and push to ECR.

1. Log into the AWS Console and search for the **CodeBuild** console.

2. Under Build projects select **Create build project**.

3. Configure the following build settings:
*Project Configuration*
- Project name = **unicorn-web-build-container**
- Description = **Build docker container image**

*Source*
- Source provider = **Amazon S3**
- Bucket = **\<your artifact bucket name\>**
- S3 object key or S3 folder = **unicorn-web-build.zip**

*Environment*
- Environment image = **Managed image**
- Operating system = **Amazon Linux 2**
- Runtime = **Standard**
- Image = **aws/codebuild/amazonlinux2-x86_64-standard:3.0**
- Image version = **Always use the latest image for this runtime version**
- Environment Type = **Linux**
- Privileged = **Tick to enable flag (required for docker builds)**
- Service role = **New service role (leave name as default)**
- Additional configuration Environment variables:
    - Name = AWS_DEFAULT_REGION; Value = **\<your-region e.g. eu-west-1\>**
    - Name = AWS_ACCOUNT_ID; Value = **\<your account id\>**
    - Name = IMAGE_REPO_NAME; Value = **unicorn-web-images**
    - Name = IMAGE_TAG; Value = **latest**
::alert[You can get you AWS Account ID by clicking on the username to the left of the region in the top banner bar in the AWS Console. You can get the region name by click on the region in the top banner.]{header="What is my AWS account ID and region?"}
![Container CodeBuild Variables](/static/container-codebuild-env.png)

*Buildspec*
- Buildspec name = **buildspec_docker.yml**

*Artifacts*
- Type = **No artifacts**

*Logs*
- Group name = **unicorn-build-logs**
- Stream name = **container**

4. Click **Create build project**.

### Set build project IAM role permission
Before we can test the build, we first need to add some additional permissions to the CodeBuild project IAM role. This is to provide CodeBuild permissions to use Amazon ECR.

1. In the AWS Console, search for **IAM** and select **Roles** in the menu on the left.

2. Search for **codebuild-unicorn-web-build-container-service-role** to locate the auto-generated role and click it.

3. Click the **Add permissions** button and select **Create inline policy** in the drop-down menu.

4. Click the **JSON** tab and insert the following policy code:
:::code{language=json showLineNumbers=false showCopyAction=true}
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ecr:BatchCheckLayerAvailability",
                "ecr:CompleteLayerUpload",
                "ecr:GetAuthorizationToken",
                "ecr:InitiateLayerUpload",
                "ecr:PutImage",
                "ecr:UploadLayerPart"
            ],
            "Resource": "*"
        }
    ]
}
:::

4. Click **Review policy** and name the policy **UnicornECRPush**. Then confirm the changes with **Create policy**.

### Test the CodeBuild project
Before we run the test, we first need an updated version of our artifact on S3 containing the Dockerfile. To do this, we will re-run the original **unicorn-web-build** project to create this zip file.

1. Log into the AWS Console and search for the **CodeBuild** console.

2. Under Build projects select **unicorn-web-build** and select **Start build > Start now** and ensure the build completes successfully (you should have a zip file in the S3 artifact bucket with the latest timestamp).

3. Now go back to Build projects and select **unicorn-web-build-container** and click **Start build > Start now**.

4. Ensure the build completes successfully. If you browse to the **unicorn-web-images** repository with the Amazon ECR console, you should find there is now one image tagged **latest**.
![Container ECR Complete](/static/container-ecr-complete.png)

Congratulations, you now have a containerized version of the Unicorn Wild Rydes application ready to deploy onto your container platform of choice!
For example, you could now configure CodeDeploy to deploy to an Amazon ECS Fargate cluster. This is however not covered in this workshop.