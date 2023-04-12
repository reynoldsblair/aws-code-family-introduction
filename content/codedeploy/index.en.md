---
title : "Lab 4: AWS CodeDeploy"
weight : 50
---

AWS CodeDeploy is a fully managed deployment service that automates software deployments to a variety of compute services such as Amazon EC2, AWS Fargate, AWS Lambda, and even on-premise services. You can use AWS CodeDeploy to automate software deployments, eliminating the need for error-prone manual operations.

In this lab, we will use CodeDeploy to install our Java WAR package onto an Amazon EC2 instance running [Apache Tomcat](http://tomcat.apache.org/).

::alert[For simplicity this lab will use a single EC2 instance in a single Availability Zone (AZ). In production environment, you should use instances spread across multiple AZs behind an Elastic Load Balancer (ELB).]{header="Best practice for production scenarios"}


![CodeDeploy Diagram](/static/progress-diagram-codedeploy.png)



### Create an EC2 instance
We are going to use AWS CloudFormation to provision a VPC and an EC2 instance to deploy our application to. 

1. Log into the AWS Console and search for **CloudFormation**.

2. Download the provided :link[CloudFormation YAML Template]{href="/static/ec2-cfn.yaml" action=download}.

3. In the CloudFormation Console, click **Create stack > with new resources (standard)**.

4. Select **Upload a template file** and click **Choose file**. Select the yaml file downloaded in step 2 and click **Next**.

5. Name the stack **UnicornStack** and provide your IP address from http://checkip.amazonaws.com/ in the format 1.2.3.4/32 when prompted. Click through next accepting all the remaining defaults. Remember to acknowledge the IAM resources checkbox before clicking **Create stack**.

6. Wait for the stack to complete. This should take no longer than 5 minutes.

7. Once successful, search for "EC2" in the AWS Console and click on **Instances (running)**. You should see once instance named **UnicornStack::WebServer**.

::alert[If you wish to connect to the instance at any point you can do so using Session Manager which has been configured for you. Select the instance click **Connect** and follow the instructions on the **Session Manager** tab.]{header="Connecting to the instance"}

### Create scripts to run the application
Next, we need to create some bash scripts in our Git repository. CodeDeploy uses these scripts to setup and deploy the application on the target EC2 instance.

1. Log into the Cloud9 IDE.

2. Create a new folder **scripts** under **~/environment/unicorn-web-project/** .

3. Create a file **install_dependencies.sh** file in the **scripts** folder and add the following lines:
:::code{language=bash showLineNumbers=false showCopyAction=true}
#!/bin/bash
sudo yum install tomcat -y
sudo yum -y install httpd
sudo cat << EOF > /etc/httpd/conf.d/tomcat_manager.conf
<VirtualHost *:80>
    ServerAdmin root@localhost
    ServerName app.wildrydes.com
    DefaultType text/html
    ProxyRequests off
    ProxyPreserveHost On
    ProxyPass / http://localhost:8080/unicorn-web-project/
    ProxyPassReverse / http://localhost:8080/unicorn-web-project/
</VirtualHost>
EOF
:::

4. Create a **start_server.sh** file in the **scripts** folder and add the following lines:
:::code{language=bash showLineNumbers=false showCopyAction=true}
#!/bin/bash
sudo systemctl start tomcat.service
sudo systemctl enable tomcat.service
sudo systemctl start httpd.service
sudo systemctl enable httpd.service
:::

5. Create a **stop_server.sh** file in the **scripts** folder and add the following lines:
:::code{language=bash showLineNumbers=false showCopyAction=true}
#!/bin/bash
isExistApp="$(pgrep httpd)"
if [[ -n $isExistApp ]]; then
sudo systemctl stop httpd.service
fi
isExistApp="$(pgrep tomcat)"
if [[ -n $isExistApp ]]; then
sudo systemctl stop tomcat.service
fi
:::

6. CodeDeploy uses an application specification (AppSpec) file in YAML to specify what actions to take during a deployment, and to define which files from the source are placed where at the target destination. The AppSpec file must be named **appspec.yml** and placed in the root directory of the source code.

Create a new file **appspec.yml** in the **~/environment/unicorn-web-project/** folder and add the following lines:
:::code{language=yaml showLineNumbers=false showCopyAction=true}
version: 0.0
os: linux
files:
  - source: /target/unicorn-web-project.war
    destination: /usr/share/tomcat/webapps/
hooks:
  BeforeInstall:
    - location: scripts/install_dependencies.sh
      timeout: 300
      runas: root
  ApplicationStart:
    - location: scripts/start_server.sh
      timeout: 300
      runas: root
  ApplicationStop:
    - location: scripts/stop_server.sh
      timeout: 300
      runas: root

:::

7. To ensure that that the newly added scripts folder and appspec.yml file are available to CodeDeploy, we need to add them to the zip file that CodeBuild creates. This is done by modifying the **artifacts** section in the **buildspec.yml** like shown below:

:::code{language=yaml showLineNumbers=false showCopyAction=false}
phases:
  [..]
  
artifacts:
  files:
    - target/unicorn-web-project.war
    - appspec.yml
    - scripts/**/*
  discard-paths: no
:::

8. Now commit all the changes to CodeCommit:
:::code{language=bash showLineNumbers=false showCopyAction=true}
cd ~/environment/unicorn-web-project
git add *
git commit -m "Adding CodeDeploy files"
git push -u origin main
:::

9. Log into the CodeBuild Console and click on **Start build** to run the unicorn-web-build project again which will include our newly added artifacts in the zip package.

### Create CodeBuild service IAM Role
CodeDeploy requires a service role to grant it permissions to the desired compute platform. For EC2/On-Premises deployments you can use the AWS Managed **AWSCodeDeployRole** policy.

1. Log into the **AWS Console** and open the **IAM** console.

2. Choose **Roles** and then click **Create role**.

3. Choose **CodeDeploy** as the service and then select **CodeDeploy** for the use case. Click **Next**.
![CodeDeploy IAM Role](/static/iam-usecase-codedeploy.png)

4. Accept the **AWSCodeDeployRole** default policy. Don't forget to take a look at the permissions this grants - in production you will want to be more granular!

5. Click **Next** and name the role **UnicornCodeDeployRole**. Click **Create role** to finish.

### Create a CodeDeploy application
Now that we have our required files in place, let's create a CodeDeploy application. An application is simply a name or container used by CodeDeploy to ensure that the correct revision, deployment configuration, and deployment group are referenced during a deployment. 

1. Log into the AWS Console and search for **CodeDeploy**.

2. Click on **Applications** on the left-hand menu and select **Create application**.

3. Name the application **unicorn-web-deploy** and select **EC2/On-premises** as the Compute platform. Note the other options for AWS Lambda and Amazon ECS. Click **Create application**.
![CodeDeploy Create](/static/codedeploy-create.png)



### Create a deployment group
Next, let's create a deployment group, which contains settings and configurations used during the deployment. It defines for example that our deployment shall target any EC2 instances with a specific tag.

1. Under the **unicorn-web-deploy** application in the **Deployment groups** tab click **Create deployment group**.

2. Configure the following options:
- Name = **unicorn-web-deploy-group**
- Service role = **arn:aws:iam::\<aws-account-id\>\:role/UnicornCodeDeployRole**
- Deployment type = **In-place**
- Environment configuration = **Amazon EC2 instances**
- Tag group 1 Key = **role**
- Tag group 1 Value = **webserver**
- Install AWS CodeDeploy Agent = **Now and schedule updates (14 days)**
- Deployment settings = **CodeDeployDefault.AllAtOnce**
- Load balancer = **Uncheck Enable load balancing (we just have one server)**
![CodeDeploy Type](/static/codedeploy-type.png)

::alert[When you deployed the CloudFormation template earlier an EC2 instance role was automatically applied giving permissions to AWS SSM for the CodeDeploy agent install and read access to S3 to download the deployment artifact.]{header="EC2 Instance Role"}

::alert[The webserver role tag was applied to the EC2 instance from the CloudFormation template]{header="Where did the tag come from?"}

3. Click **Create deployment group**.

### Create deployment
After creating our deployment group, i.e. defining the resources that we want to deploy, we can now create a deployment!

1. In the **unicorn-web-deploy-group** click **Create deployment**.

2. For the revision location use the S3 bucket created earlier:
:::code{language=bash showLineNumbers=false showCopyAction=true}
s3://<my-artifact-bucket-name>/unicorn-web-build.zip
:::

3. Select the revision file type as **.zip**.
![CodeDeploy Deployment](/static/codedeploy-deployment.png)

4. Leave the other settings as default and click **Create deployment**.

5. The deployment will now begin. Keep an eye on the deployment lifecycle events and check it completes successfully.
![CodeDeploy Complete](/static/codedeploy-complete.png)

6. Finally check that the web application is working by browsing to **http://\<public-ip-address\>**. You can get the public IP address from the instance details Networking tab. Remember that if you click the open address link this will default to https and needs to be changed to http.
![CodeDeploy WebApp](/static/codedeploy-webapp.png)
::alert[Make sure you're accessing the address using http (not https!) and check if your own IP address hasn't changed. If your IP has changed, you'll have to modify the Security Group of the EC2 instance.]{header="Seeing an error when opening the website?"}

Nice! We have a working web application! Don't worry, we will jazz the content up later on.

We now have a way to build and deploy our application, but currently both jobs are triggered manually. It would be much better if we automated the whole process triggered by a change to CodeCommit. Introducing AWS CodePipeline!