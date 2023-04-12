---
title : "Extending the pipeline"
weight : 61
---

In this lab we will look at extending our existing CodePipeline pipeline to include a manual approval step before deploying to a production server. 

Our current pipeline looks like this:
![CodePipeline Extended Orig](/static/extend-pipeline-original.png)

At the end of the lab the pipeline will look like this:
![CodePipeline Extended New](/static/extend-pipeline-new.png)

### Update the CloudFormation stack
First, we will update our CloudFormation stack to include an additional EC2 instance which will act as our production server

1. Log into the AWS Console and go to the CloudFormation console.

2. Download the provided :link[CloudFormation YAML Template]{href="/static/ec2-cfn-cp-ext.yaml" action=download}.

3. In the CloudFormation console click on the **UnicornStack** and click **Update**.

4. Click **Replace current template** and upload the file downloaded from step 2.

5. Proceed with the next steps using **Next** until you arrive at the **Review** page.

6. Confirm the IAM changes and click **Update stack**. This will take a few minutes to complete.

### Add an additional CodeDeploy deployment group
Now we need to add a production server deployment group in CodeDeploy

1. Log into the AWS Console and go to the **CodeDeploy** service.

2. Click on **Applications** and go to the **unicorn-web-deploy** application.

3. Under the deployment groups tab click **Create deployment group**.

4. Configure the following settings:
- Deployment group name = **unicorn-web-deploy-group-prod**
- Service role = **UnicornCodeDeployRole**
- Deployment type = **In-place**
- Environment configuration = **Amazon EC2 instances**
- Tag group 1 Key = **role**
- Tag group 1 Value = **webserverprd**
- Install CodeDeploy Agent = **Now and schedule updates (14 days)**
- Deployment configuration = **CodeDeployDefault.AllAtOnce**
- Load balancer = **Uncheck enable load balancing**

5. Click **Create deployment group**.

### Create SNS topic
We will create an SNS topic to be used for manual approvals in the next section.

1. Log into the AWS Console and go to the **SNS** console.

2. On the left menu bar click on **Topics** then click **Create topic**.

3. Set the type to be **Standard** and the name to be **unicorn-pipeline-notifications**. Leave everything else as default and click **Create topic**.
![CodePipeline Extended SNS](/static/extend-pipeline-sns.png)

4. Under subscriptions click **Create subscription**. Set the protocol to be **Email** and enter your email address. Click **Create subscription**.

5. Log into your emails, where should have an email with the subject **AWS Notification - Subscription Confirmation**. Click the link to confirm the subscription.



### Update CodePipeline
Now we need to add in our manual approval step and the deployment to the production web server

1. Log into the AWS Console and go to the **CodePipeline** console.

2. Click on the **unicorn-web-pipeline** and click **Edit** (you need to view the pipeline details to see the edit button). The following view should become visible.
![CodePipeline Extended Edit](/static/extend-pipeline-edit.png)

3. Under the Deploy stage click **Add stage**. Name the stage **Approval** and click **Add stage**.

4. In the newly created Approval stage click **Add action group**. Set the action name to be **ProductionApproval** and the action provider to be **Manual approval**. Set the SNS topic ARN to be **unicorn-pipeline-notifications** which we created earlier. Leave the other options blank and click **Done**.

5. Now add another Stage after the approval named **DeployProd**.

6. Add an action group in this stage named **DeployProd**. Set the action provider to be **AWS CodeDeploy** and the input artifact to be **BuildArtifact**. For the application name select **unicorn-web-deploy** and **unicorn-web-deploy-prod** for the deployment group.
![CodePipeline Extended Prd](/static/extend-pipeline-prddeploy.png)

7. Click **Done** and then click **Save** to save the pipeline changes. You will need to click **Save** again on the popup screen.

<!-- Below SNS notifications does not work with default EventEngine IAM role -->
<!-- ### Add pipeline notifications

1. In the CodePipeline console click on the **unicorn-web-pipeline** pipeline

2. Click on the **Notify** button and click **Create notification rule**

3. For the notification name enter **Pipeline Status** and select all the options under **Pipeline execution**
![CodePipeline Extended Notification](/static/extend-pipeline-notification.png)

4. Under targets select **SNS topic** and choose the SNS topic created earlier
::alert[You can also choose to send notifications to AWS ChatBot for example to send to a Slack channel]{header="AWS ChatBot"}

5. Click **Submit** to finish -->

### Testing the new pipeline
Now that the pipeline is complete, let's run a test!

1. Click on the **unicorn-web-pipeline** in the CodePipeline console.

2. Click **Release change** and click **Release**.

3. Wait for the pipeline to reach the manual approval stage and then check your emails for an approval link. Note: you can also approve this directly in the AWS Console.

4. Click **Review** under the approval stage and in the review box add some comments and click **Approve**
![CodePipeline Extended Approve](/static/extend-pipeline-approve.png)

Once approved the pipeline will continue onto the production CodeDeploy stage after a few seconds.

5. Ensure the pipeline completes successfully and then check the EC2 instance **UnicornStack::WebServerProd** public IP address is running the web application!
![CodePipeline Verify](/static/codepipeline-verify.png)


Congratulations, you have now successfully extended the CodePipeline by an approval stage that separates development and production deployments.