---
title : "Lab 3: AWS CodeBuild"
weight : 40
---

AWS CodeBuild is a fully managed continuous integration service that compiles source code, runs tests, and produces software packages that are ready to deploy. You can get started quickly with prepackaged build environments, or you can create custom build environments that use your own build tools.

In this lab we will setup a CodeBuild project to package our application code into a Java Web Application Archive (WAR) file.

![CodeBuild Diagram](/static/progress-diagram-codebuild.png)

### Create an S3 bucket
We first need to create an S3 bucket which will be used to store the output from CodeBuild i.e. our WAR file!

1. Log into the AWS Console and search for the **S3** service.

2. Click **Create Bucket** and give the bucket a unique name e.g. *unicorn-build-artifacts-12345*.

3. Leave all other options as default and click **Create bucket**.

### Create a CodeBuild build project

1. Log into the AWS Console and search for the **CodeBuild** service.

2. Under build projects select **Create build project**.

3. Name the project **unicorn-web-build** and set a helpful description. Below **Additional configuration**, add a tag with key **team** and value **devops**.

4. Under source select **AWS CodeCommit** as the source provider and select the **unicorn-web-project** as the repository. The branch should be **main** with no Commit ID.
![CodeBuild Source](/static/codebuild-source.png)

::alert[Take a moment to explore the options available for the source. You can use Amazon S3, GitHub, Bitbucket as providers. You can also reference a specific Git tag, or Commit ID rather than a branch name.]{header="Browse the other options"}

5. Under environment choose to use a **Managed image** and select the following:
- Operating System = **Amazon Linux 2**
- Runtime = **Standard**
- Image = **aws/codebuild/amazonlinux2-x86_64-standard:3.0**
- Image version = **Always use the latest image for this runtime version**
- Environment Type = **Linux**

Choose to create a **New service role** and leave the Role name as default.
![CodeBuild Environment](/static/codebuild-env.png)

::alert[You can find the full list of runtime versions at https://docs.aws.amazon.com/codebuild/latest/userguide/available-runtimes.html. Should you require a runtime not provided by CodeBuild you can provide your own custom docker container image. For this example, we only need Amazon Corretto support which is an Amazon distro of the OpenJDK.]{header="Available Runtimes"}

::alert[The service role grants CodeBuild permission to access other AWS resources require for your build.]{header="Service Role"}

6. Under Buildspec leave the default option to **Use a buildspec file** which will look for a config file called buildspec.yml (we will create this later).

7. Under Artifacts select **Amazon S3** and choose the bucket name created earlier. Set the name to **unicorn-web-build.zip**. Leave the other options as default ensuring the artifact packaging is set to **Zip**.
::alert[Artifacts refers to the files that are produced as part of the build process. Currently CodeBuild (for the build process) and CodeDeploy (for the deployment) don't yet integrate with CodeArtifact directly, which is why we're using an S3 bucket here instead.]{header="Artifacts = CodeArtifact?"}
![CodeBuild Artifact](/static/codebuild-artifact.png)

8. Finally, under Logs enable **CloudWatch logs** if it's not enabled yet. Set the group name to **unicorn-build-logs** and the stream name to **webapp**. This will allow us to track the output of our build in CloudWatch Logs.

9. Click **create build project** to finish!

### Create the buildspec.yml file
Now we have our build project setup we need to give it some instructions on how to build our application. To do this we will create a buildspec.yml (YAML) file in the root of the code repository.

1. Log back into the Cloud9 IDE.

2. Under the **~/environment/unicorn-web-project/** folder create a new file called **buildspec.yml** (naming must be exact!) and copy in the below contents. Make sure to replace the **domain-owner** account ID with your own account ID.
:::code{language=yaml showLineNumbers=false showCopyAction=true}
version: 0.2

phases:
  install:
    runtime-versions:
      java: corretto8
  pre_build:
    commands:
      - echo Initializing environment
      - export CODEARTIFACT_AUTH_TOKEN=`aws codeartifact get-authorization-token --domain unicorns --domain-owner 123456789012 --query authorizationToken --output text`
  build:
    commands:
      - echo Build started on `date`
      - mvn -s settings.xml compile
  post_build:
    commands:
      - echo Build completed on `date`
      - mvn -s settings.xml package
artifacts:
  files:
    - target/unicorn-web-project.war
  discard-paths: no
:::

::alert[You can find the full buildspec.yml reference documentation here https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html]{header="Buildspec Reference"}

3. Save the buildspec.yml file. Then commit and push it to CodeCommit.
:::code{language=bash showLineNumbers=false showCopyAction=true}
cd ~/environment/unicorn-web-project
git add *
git commit -m "Adding buildspec.yml file"
git push -u origin main
:::

### Modifying the IAM role
As we're using CodeArtifact during the build phase, there is a small change required in the previously auto-generated IAM role to ensure it has permissions to use CodeArtifact. For this, we will use the IAM policy that was created in the earlier lab.

1. In the AWS Console, search for **IAM** and select **Roles** in the menu on the left.

2. Search for **codebuild-unicorn-web-build-service-role** to locate the auto-generated role and click it.

3. Click the **Add permissions** button and select **Attach policies** in the drop-down menu.

4. Search for **codeartifact-unicorn-consumer-policy**, select the item, and click **Attach policies**.


### Testing the build project
Now we have everything in place to run our first build using CodeBuild!

1. In the AWS Console search for **CodeBuild** service.

2. Select the unicorn-web-build project and select **Start build** > **Start now**.

3. Monitor the logs and wait for the build status to complete (this should take no more than 5 minutes):
![CodeBuild Complete](/static/codebuild-complete.png)

4. Finally browse to your artifact S3 bucket to verify you have a packaged WAR file inside a zip named **unicorn-web-project.zip**:
![CodeBuild S3](/static/codebuild-s3.png)

Amazing work! We now have a code and artifact repository, and a solution to compile our application. It's a manually triggered build at the moment which will fix later. But at least we know our builds are completed in a consistent manner.

Next, we will look into how to host and deploy our packaged WAR file with AWS CodeDeploy!