---
title : "Lab 2: AWS CodeArtifact"
weight : 35
---

AWS CodeArtifact is a fully managed artifact repository service that makes it easy for organizations of any size to securely fetch, store, publish, and share software packages used in their software development process.

In this lab we will setup a CodeArtifact repository that we will be using during the build phase with CodeBuild to fetch Maven packages from a public package repository (the "Maven Central Repository"). Using CodeArtifact rather than the public repository directly has several advantages including improved security, as you can strictly define which packages can be used. To see other advantages of using CodeArtifact please refer to the [AWS CodeArtifact features](https://aws.amazon.com/codeartifact/features/) web page.

Within this workshop, we will use CodeArtifact as a simple package cache. This way, even if the public package repository would become unavailable, we could still build our application. In real-world scenarios this can be an important requirement to mitigate the risk that an outage of the public repository can break the complete CI/CD pipeline. Furthermore, it helps to ensure that packages, which your project depends on, and which are (accidentally, or on purpose) being removed from the public package repository, don't break the CI/CD pipeline (as they are still available via CodeArtifact in that case).

![CodeArtifact Diagram](/static/progress-diagram-codeartifact.png)

### Create Domain and Repository

1. Log into the AWS Console and search for the **CodeArtifact** service.

2. Click **Create repository**. Name it **unicorn-packages** and give it a description. Select **maven-central-store** as public upstream repository and click **Next**.
![CodeArtifact Create Repository](/static/codeartifact-create.png)

3. Choose **This AWS account** as the owner of the domain. Enter **unicorns** as domain name, then click **Next**.
![CodeArtifact Select Domain](/static/codeartifact-select-domain.png)

4. Review the settings. Especially keep note of the **Package flow** section that visualizes how there will be two repositories created as part of the process: the actual *unicorn-packages* repository, as well as an upstream repository (*maven-central-store*), which serves as an intermediate between the public repository. Click **Create repository** to finish the process.
![CodeArtifact Package Flow](/static/codeartifact-package-flow.png)

### Connect the CodeArtifact repository

1. On the next page, click **View connection instructions** and select **mvn** in the dialog.

2. Copy the command for the authorization token and run it in your Cloud9 command prompt. This will look similar to the below, just the account ID will be a different one:
:::code{language=bash showLineNumbers=false showCopyAction=false}
export CODEARTIFACT_AUTH_TOKEN=`aws codeartifact get-authorization-token --domain unicorns --domain-owner 123456789012 --query authorizationToken --output text`
:::

3. For the next steps, we'll have to update the **settings.xml**. As this doesn't exist yet, let's create it first:
:::code{language=bash showLineNumbers=false showCopyAction=true}
cd ~/environment/unicorn-web-project
echo $'<settings>\n</settings>' > settings.xml 
:::

3. Open the newly created **settings.xml** in the Cloud9 directory tree and follow the remaining steps in the **Connection instructions** dialog in the **CodeArtifact** console including the mirror section. The complete file will look similar to the one below. Close the dialog when finished by clicking **Done**.
::alert[**Important**: Do **NOT** copy/paste the content below into your settings.xml as this is just to illustrate the structure - your own settings.xml will use different URLs, which you need to copy from the CodeArtifact dialog.]{type="warning"}
:::code{language=xml showLineNumbers=false showCopyAction=false}
<settings>
    <profiles>
        <profile>
            <id>unicorns-unicorn-packages</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <repositories>
                <repository>
                    <id>unicorns-unicorn-packages</id>
                    <url>https://unicorns-123456789012.d.codeartifact.us-east-2.amazonaws.com/maven/unicorn-packages/</url>
                </repository>
            </repositories>
        </profile>
    </profiles>
    <servers>
        <server>
            <id>unicorns-unicorn-packages</id>
            <username>aws</username>
            <password>${env.CODEARTIFACT_AUTH_TOKEN}</password>
        </server>
    </servers>
    <mirrors>
        <mirror>
            <id>unicorns-unicorn-packages</id>
            <name>unicorns-unicorn-packages</name>
            <url>https://unicorns-123456789012.d.codeartifact.us-east-2.amazonaws.com/maven/unicorn-packages/</url>
            <mirrorOf>*</mirrorOf>
        </mirror>
    </mirrors>
</settings>
:::


### Testing via Cloud9

1. Let's verify if the application can be compiled successfully locally in Cloud9 using the settings file:

:::code{language=bash showLineNumbers=false showCopyAction=true}
mvn -s settings.xml compile
:::

2. If the build was successful, go back to the CodeArtifact console and refresh the page of the **unicorn-packages** repository. You should now see the packages that were used during the build in the artifact repository. This means that they were downloaded from the public repository and are now available as a copy inside CodeArtifact.

![CodeArtifact Packages](/static/codeartifact-packages.png)

### IAM Policy for consuming CodeArtifact

Before moving on to the next lab, let's define an IAM policy so that other services can consume our newly created CodeArtifact repository.

1. In the AWS Console, search for **IAM**, select it, and click **Policies** in the menu on the left.

2. Click **Create policy** and select the **JSON** tab on top to view the raw JSON code of the IAM policy, then copy/paste the policy code below (source: [Using Maven packages in CodeBuild](https://docs.aws.amazon.com/codeartifact/latest/ug/using-maven-packages-in-codebuild.html)). This will make sure that other services such as CodeBuild will be able to read the packages in our CodeArtifact repository.

:::code{language=xml showLineNumbers=false showCopyAction=true}
{
  "Version": "2012-10-17",
  "Statement": [
      {
          "Effect": "Allow",
          "Action": [ "codeartifact:GetAuthorizationToken",
                      "codeartifact:GetRepositoryEndpoint",
                      "codeartifact:ReadFromRepository"
                      ],
          "Resource": "*"
      },
      {       
          "Effect": "Allow",
          "Action": "sts:GetServiceBearerToken",
          "Resource": "*",
          "Condition": {
              "StringEquals": {
                  "sts:AWSServiceName": "codeartifact.amazonaws.com"
              }
          }
      }
  ]
}
:::

3. Click **Next: Tags** and **Next: Review**. 

4. Name the policy **codeartifact-unicorn-consumer-policy** and provide a meaningful description such as "Provides permissions to read from CodeArtifact".

5. Click **Create policy**.

Nice work! We now have our working repositories which can be consumed by other services.
Next, we need a way to compile our code from within AWS to produce our Java WAR file. Introducing AWS CodeBuild!
