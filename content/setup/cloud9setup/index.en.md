---
title : "Setup AWS Cloud9 IDE"
weight : 11
---

:::alert{header="Important" type="warning"}
If you are running this workshop as part of an AWS event then a Cloud9 environment has been pre-created in your account named `UnicornIDE`.
:::

AWS Cloud9 is a cloud-based integrated development environment (IDE) that lets you to write, run and debug code with just a browser. It includes a code editor, debugger, and terminal. Cloud9 comes prepackaged with essential tools for popular programming languages, so you don’t need to install files or configure your development machine to start new projects.

Throughout this workshop we will be using AWS Cloud9 to develop our application code and interact with Git.


### Launch a new Cloud9 IDE

1. Log onto the AWS Console.

2. Search for **Cloud9** and then click **Create environment**.

3. Name the environment **UnicornIDE** and give a helpful description. Click **Next step**.

4. On the configure settings page select **Create a new EC2 instance for environment**. For the instance type select **t2.micro** and for the platform **Amazon Linux 2**. The remaining settings can be left as default.
![Cloud9 Create](/static/cloud9-create.png)

5. Click **Next step** and then **Create environment**.

Cloud9 automatically creates a new EC2 instance in your AWS Account running the Cloud9 IDE software. 

Once complete you should be presented with the Cloud9 IDE and a terminal prompt as shown below.

![Cloud9 Complete](/static/cloud9-complete.png)

On the left-hand side is the file explorer and at the bottom of the screen is a terminal prompt. The Cloud9 environment already has tools such as Docker and Git installed. The awscli is also pre-configured with [AWS managed temporary credentials](https://docs.aws.amazon.com/cloud9/latest/user-guide/security-iam.html#auth-and-access-control-temporary-managed-credentials).

Congratulations, you have setup your development environment! Let's create our application in the next lab!