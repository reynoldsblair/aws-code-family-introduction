---
title: "Serverless deploy"
weight: 82
---

In this lab, we will look at how to deploy serverless applications on AWS using the [Serverless Application Model](https://aws.amazon.com/serverless/sam/) CLI.

The AWS Serverless Application Model (SAM) is an open-source framework for building serverless applications. It provides shorthand syntax to express functions, APIs, databases, and event source mappings. With just a few lines per resource, you can define the application you want and model it using YAML. During deployment, SAM transforms and expands the SAM syntax into AWS CloudFormation syntax, enabling you to build serverless applications faster.

We will use the built-in [sam pipeline bootstrap](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-pipeline-bootstrap.html) functionality to create the following pipeline:
![SAM Pipeline Diagram](/static/sam-pipeline-diagram.png)

::alert[Notice how CodeBuild is being used for deployment in this case. Because we can run any commands we like in CodeBuild, it makes it a great tool to perform the **sam deploy** command!]{header="CodeBuild used for deployment"}

The Serverless application we will deploy is a simple API Gateway which triggers a Lambda function:
![SAM Pipeline Application](/static/sam-pipeline-app.png)

### Getting Setup

For this lab we will be using the existing UnicornIDE Cloud9 environment which has the SAM CLI already installed.

::alert[If you wish to install the SAM CLI on your own machine, rather than Cloud9, follow the instructions [here](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html)]{header="AWS Access Keys"}

### Creating a Serverless application

With everything set up, it's time to create our first serverless application.

Log into the UnicornIDE Cloud9 environment created earlier. In the terminal, run the following:
:::code{language=bash showLineNumbers=false showCopyAction=true}
cd ~/environment/
sam init
:::

Follow the guided initialization process, specifying the following options when prompted:

* Template: **AWS Quick Start Templates**
* Package type: **Zip**
* Runtime: nodejs14.x
* Project name: **serverless-unicorns**
* Quick start template: **Web Backend**

This will create a new templated serverless NodeJS web application with an API Gateway, a Lambda Function, and a DynamoDB table. Take a moment to explore the resources created, particularly the **template.yml** file, which contains several "Serverless" resources that will be created.

### Create the Git repo for the serverless app

1. Navigate to the **CodeCommit** console and create a new repository.

2. Name the repository **serverless-unicorns**. Once created, copy the HTTPS URL.

3. Back in the Cloud9 IDE, initialize the Git repo:
:::code{language=bash showLineNumbers=false showCopyAction=true}
cd ~/environment/serverless-unicorns
git init -b main
git remote add origin <HTTPS CodeCommit repo URL>
:::

Push the new application to the repository by running the following commands:

:::code{language=bash showLineNumbers=false showCopyAction=true}
git add *
git commit -m "Initial commit"
git push -u origin main
:::

::alert[If you have any issues with the CodeCommit section of this lab, refer back to [Lab 1](../../codecommit) for a refresher]{header="CodeCommit Help"}

### Deploying the app

Now we have our basic serverless web app, so it's time to deploy it. In the terminal, run the following commands:

:::code{language=bash showLineNumbers=false showCopyAction=true}
sam build
sam deploy --guided
:::

Follow the guided stack creation process, inputting the following variables when prompted, replacing the region with the region in which the workshop is being run (e.g. eu-west-1):

- Stack Name [sam-app]: **serverless-unicorns**
- AWS Region [us-east-1]: **REGION WHERE YOU ARE RUNNING THIS WORKSHOP**
- Confirm changes before deploy [y/N]: **y**
- Allow SAM CLI IAM role creation [Y/n]: **y**
- getAllItemsFunction may not have authorization defined, Is this okay? [y/N]: y
- getByIdFunction may not have authorization defined, Is this okay? [y/N]: y
- putItemFunction may not have authorization defined, Is this okay? [y/N]: y 
- Save arguments to configuration file [Y/n]: **y**
- SAM configuration file [samconfig.toml]: _confirm default_
- SAM configuration environment [default]: _confirm default_

When prompted whether to "**Deploy this changeset?**", confirm with **y**. This will then deploy the expanded CloudFormation template, and provision the underlying resources within your AWS account (this will take a few minutes).

Navigate to the **CloudFormation** console in the AWS Management Console and you will see the stack has been created. Here you can view the resources created by the SAM deployment.

### Creating the pipeline

This is great, but we don't want every developer to have to manually push from their laptops every time. We want to create a centralized CI/CD pipeline that we can invoke when changes are made to our serverless app repository.

Luckily, SAM has some tools built in to help us with this. We will now use the SAM CLI to create the pipeline for AWS CodePipeline that will deploy our serverless application.

In the terminal, run the following command:

:::code{language=bash showLineNumbers=false showCopyAction=true}
sam pipeline init --bootstrap
:::

Specify the following values when prompted by the guided initialization:

- Template: **AWS Quick Start Pipeline Templates**
- CI/CD system: **AWS CodePipeline**
- _[..] go through stage setup process [..]?_: **y**
- Stage name: **uat**
- Credential source: **default (named profile)**
- Region: "REGION WHERE YOU ARE RUNNING THIS WORKSHOP"
- Pipeline user: _confirm default_
- Pipeline execution role: _confirm default_
- CloudFormation execution role: _confirm default_
- Artifacts bucket: _confirm default_
- Does your application contain any IMAGE type Lambda functions? [y/N]: _confirm default_

Press enter to confirm everything looks ok. Then press **Y** to confirm to proceed creating the resources.

Next, we will get prompted to create the second stage. Confirm with **Y** that you would like to go through the stage setup process again. The repeat the process above again starting with the stage name where you specify **prod**. The other values can stay the same as in the previous stage.

When prompted for the Git provider, choose the following.

* Git provider: **CodeCommit**
* Repository name: **serverless-unicorns**.
* Branch: **main**
* Template file path: **template.yml**

For the next prompt, choose the uat stage (**1**) and enter **serverless-unicorns-uat** as the SAM stack name. Then choose prod stage (**2**) with name **serverless-unicorns-prod**.

SAM will now create the CodePipeline configuration file. Push the new configuration files to the repository by running the following commands:

:::code{language=bash showLineNumbers=false showCopyAction=true}
git add .
git commit -m "Add CodePipeline config"
git push origin main
:::

Run the following command to deploy the pipeline into your AWS account. Confirm with **y** when being asked whether to deploy the changes.

:::code{language=bash showLineNumbers=false showCopyAction=true}
sam deploy -t codepipeline.yaml --stack-name serverless-pipeline --capabilities=CAPABILITY_IAM
:::

This will take a minute to deploy the CloudFormation stack and create the underlying resources. Once completed, navigate to the **CloudFormation** console where you will notice three new CloudFormation stacks that have been created:
* **serverless-pipeline** stack for the CI resources
* **serverless-unicorns-uat** stack for the testing environment 
* **serverless-unicorns-prod** stack for the production environment

Open the **CodePipeline** console and see that a new pipeline has been created as an initial deployment of our project. Wait for this pipeline to complete before continuing.

![Serverless Changeset](/static/serverless-changeset.png)

### Testing the pipeline

Navigate back to the **CloudFormation**. Select the **serverless-unicorns-uat** stack and click the **Resources** tab. Follow the link for the **ServerlessRestApi**.

![Serverless Stack](/static/serverless-stack.png)

Click the **POST** resource and click **Test**.

![Serverless API](/static/serverless-api.png)

Paste the following into the **Request Body** box and press **Test**:

:::code{language=json showLineNumbers=false showCopyAction=true}
{ "id": "1" }
:::

See that a new resource has been created:

![Serverless POST](/static/serverless-api-post.png)

Now try the same for the **GET** method and see that our id is returned.

![Serverless GET](/static/serverless-api-get.png)

We'll now make a change locally and deploy this to see our CI/CD in action.

Open the **get-all-items.js** file and find the following line of JavaScript:

:::code{language=javascript showLineNumbers=false showCopyAction=true}
const items = data.Items;
:::

Replace this with the following:

:::code{language=javascript showLineNumbers=false showCopyAction=true}
const items = data.Items;
items.push({ "unicorns-enabled": true });
:::

Save and close the file. Then commit and push our changes to the origin:

:::code{language=bash showLineNumbers=false showCopyAction=true}
git add .
git commit -m "Add unicorn power"
git push origin main
:::

Navigate to the **CodePipeline** console. After a few moments you will see a new pipeline running for our changes:

![Serverless Pipeline](/static/serverless-pipeline.png)

Click to view the pipeline as it executes. Once it has completed, navigate back to the **API Gateway** console.

Test the **GET** method on the API again to see our changes have been deployed successfully. Now our serverless microservice has unicorn power!

![Serverless Testing](/static/serverless-get-2.png)
