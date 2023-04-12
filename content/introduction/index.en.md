---
title : "Introduction"
weight : 5
---

### What you will learn

During this workshop you will experience the AWS Code services hands-on, including:
- AWS CodeCommit as a Git repository
- AWS CodeArtifact as a managed artifact repository
- AWS CodeBuild as a way to run tests and produce software packages
- AWS CodeDeploy as a software deployment service
- AWS CodePipeline to create an automated CI/CD pipeline

You will experience the process of creating a CI/CD pipeline for a Java application deployed onto an EC2 Linux instance. Later you will containerize the application and publish to Amazon ECR before finally looking at the SAM CLI for Serverless CI/CD.

As a bonus, you will get hands-on experience using AWS Cloud9, a cloud-based integrated development environment (IDE) that lets you write, run, and debug your code with just a browser.

### Target audience
This workshop is aimed at developers, devops engineers, and technical architects looking to learn more about AWS Code services.

### Expected duration

This workshop will take approx 3 hours to complete. You do not need to complete all the labs, however, they should be completed in order.

### Running in your own AWS Account

This workshop is designed to be run at either an AWS event or in your own AWS account. No pre-created resources are required.

Remember to run through the cleanup section once you have completed the workshop to avoid any undesired costs.

### Estimated costs

The services used in the workshop are mostly covered by the [AWS Free Tier](https://aws.amazon.com/free/)

- AWS CodeCommit includes 5 actives users per month for free
- AWS CodeArtifact includes 2GB of storage and 100,000 requests per month for free
- AWS CodeBuild includes 100 build minutes of build.general1.small per month for free
- AWS CodeDeploy is free to use for EC2, Lambda and ECS deployments
- AWS CodePipeline pipelines are free for the first 30 days after creation

EC2 instances used in the workshop are **t2.micro** which are covered for 750 hours under the AWS Free Tier each month for one year. If you are outside the first year the cost should be no more than $2 for the workshop.

Please ensure all resources are deleted at the end of the workshop using the cleanup instructions to avoid charges.

### Required skills

A basic understand of AWS is required including Amazon S3, Amazon EC2, AWS IAM and AWS CloudFormation.

You should also be familiar with Git, and basic Linux commands.
