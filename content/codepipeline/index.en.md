---
title : "Lab 5: AWS CodePipeline"
weight : 60
---

AWS CodePipeline is a fully managed continuous delivery service that helps you automate your release pipelines for fast and reliable application and infrastructure updates. You only pay for what you use.

In this lab we use CodePipeline to create an automated pipeline using the CodeCommit, CodeBuild and CodeDeploy components created earlier. The pipeline will be triggered when a new commit is pushed to the main branch of our Git repo.

![CodePipeline Diagram](/static/progress-diagram-codepipeline.png)

### Create the pipeline

1. Log into the **AWS Console** and search for the **CodePipeline** service.

2. Under **Pipelines** choose **Create pipeline**.

3. Enter **unicorn-web-pipeline** as the pipeline name. Choose to create a new service role and use the auto generated name. Leave other settings as default and click **Next**.
![CodePipeline Config](/static/codepipeline-config.png)


4. Under the source provider select **AWS CodeCommit** and select the **unicorn-web-project** as the repository. Set the branch name to be **main**. Leave the detection option as **Amazon CloudWatch Events** and the output artifact format to be **CodePipeline default**. Click **Next**.
![CodePipeline Source](/static/codepipeline-source.png)


5. On the build stage screen select **AWS CodeBuild** as the build provider and **unicorn-web-build** as the project name. Leave the build type as **Single build**. Click **Next**.
![CodePipeline Build](/static/codepipeline-build.png)

6. On the deploy stage screen select **AWS CodeDeploy** as the deploy provider and **unicorn-web-deploy** as the application name. Select **unicorn-web-deploy-group** as the deployment group. Click **Next**.
![CodePipeline Deploy](/static/codepipeline-deploy.png)

::alert[Take a moment to browse the other supported deploy providers including CloudFormation, ECS, Elastic Beanstalk and Service Catalog.]{header="Other deploy providers"}

7. Review the pipeline settings and click **Create pipeline**. Once you click create, the whole pipeline will run for the first time. Ensure it completes successfully (this may take a few minutes).

### Release a change
Congratulations, you now have a fully managed CI/CD pipeline! Let's test if everything is working.

1. Log back into your Cloud9 environment.

2. Update the **index.jsp** with the below html:
:::code{language=html showLineNumbers=false showCopyAction=true}
<!doctype html>

<html lang="en">
<head>
  <meta charset="utf-8">
  <style>
    body{
        font-family:'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    }
    .site-header .title{
        background: url(images/wr-home-top.jpg) no-repeat top;
        background-size: cover;
        padding-bottom: 70.2753441802%;
        margin: 0;
        text-indent: -999em;
        position: relative;
    }
    .home-about {
        background: #f50856;
        color: #fff;
        padding: 5rem 0;
        text-align: center;
    }
    </style>
  <title>Wild Rydes</title>
</head>

<body>
    <header class="site-header">
        <h1 class="title">Wild Rydes</h1>
    </header>
    <section class="home-about">
        
        <h2 class="section-title">How Does This Work?</h2>
        <p class="content">
            In today's fast paced world, you've got places you need to be but not enough time in your jam packed schedule. Wouldn't it be nice if there were a transportation service that changed the way you get around daily? Introducing Wild Rydes, an innovative transportation service that helps people get to their destination faster and hassle-free. Getting started is as easy as tapping a button in our app.
        </p>
        <h2 class="section-title">Our Story</h2>
      <p class="content">
        Wild Rydes was started by a former hedge fund analyst and a software developer. The two long-time friends happened upon the Wild Rydes idea after attending a silent yoga retreat in Nevada. After gazing upon the majestic herds of unicorns prancing across a surreal Nevada sunset, they witnessed firsthand the poverty and unemployment endemic to that once proud race. Whether it was modern society's reliance on science over magic or not, we'll never know the cause of their Ozymandian downfall and fade to obscurity. Moved by empathy, romance, and free enterprise, they saw an opportunity to marry society's demand for faster, more flexible transportation to underutilized beasts of labor through an on-demand market making transportation app. Using the founders' respective expertise in animal husbandry and software engineering, Wild Rydes was formed and has since raised untold amounts of venture capital. Today, Wild Rydes has thousands of unicorns in its network fulfilling hundreds of rydes each day.
      </p>
    </section>
    

</body>
</html>
:::

3. Download the :link[background image]{href="/static/wr-home-top.jpg" action=download} and save it to your local machine. Then create a new folder **images** below **unicorn-web-project/src/main/webapp/images/** and upload the file via Cloud9 using **File > Upload Local File...**

4. Commit the changes using the command below:
:::code{language=bash showLineNumbers=false showCopyAction=true}
cd ~/environment/unicorn-web-project/
git add *
git commit -m "Visual improvements to homepage"
git push -u origin main
:::

5. Check back in the CodePipeline console. The pipeline should be triggered by the push automatically. Wait for the pipeline to complete successfully (this should take no longer than 5 mins).

6. Once the pipeline has finished, browse to the EC2 public IP address **http://\<ip-address\>/** to see the changes.
![CodePipeline Verify](/static/codepipeline-verify.png)

Congratulations! You now have a working CI/CD pipeline using the AWS Code services.