---
title : "Lab 1: AWS CodeCommit"
weight : 30
---

AWS CodeCommit is a secure, highly scalable, managed source control service that hosts private Git repositories. CodeCommit eliminates the need for you to manage your own source control system or about scaling its infrastructure.

In this lab we will setup a CodeCommit repository to store our Java code!

![CodeCommit Diagram](/static/progress-diagram-codecommit.png)

### Create a Repository

1. Log into the AWS Console and search for the **CodeCommit** service.

2. Click **Create repository**. Name it **unicorn-web-project** and give it a description. Also add a tag with key **team** and value **devops**.
![CodeCommit Create](/static/codecommit-create.png)

3. Click **Create**.

4. On the next page select **Clone URL** and **Clone HTTPS**. This will copy the repository URL to the clipboard. The URL will have the following format:
:::code{language=md showLineNumbers=false showCopyAction=false}
https://git-codecommit.<region>.amazonaws.com/v1/repos/<project-name>
:::

Keep this URL handy for the next section.

### Commit your Code

1. Back in the Cloud9 environment setup your Git identity:
:::code{language=bash showLineNumbers=false showCopyAction=true}
git config --global user.name "<your name>"
git config --global user.email <your email>
:::

2. Make sure you are in the ~/environment/unicorn-web-project and init the local repo and set the remote origin to the CodeCommit URL you copied earlier:
:::code{language=bash showLineNumbers=false showCopyAction=true}
cd ~/environment/unicorn-web-project
git init -b main
git remote add origin <HTTPS CodeCommit repo URL>
:::

3. Now we can commit and push our code!
:::code{language=bash showLineNumbers=false showCopyAction=true}
git add *
git commit -m "Initial commit"
git push -u origin main
:::

4. You should now be able to refresh the CodeCommit page in the AWS Console and see the newly created files.
![CodeCommit Push](/static/codecommit-push.png)

Nice work! Now we have a working CodeCommit repository to store and version control our code!
Next, we need a way to fetch the packages to produce our Java WAR file. Introducing AWS CodeArtifact!