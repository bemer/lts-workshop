# Deployng an application with Fargate

**Quick jump:**

* [1. Tutorial overview](https://github.com/bemer/lts-workshop/tree/master/02-CreatingDockerImage#tutorial-overview)
* [2. Creating your first image](https://github.com/bemer/lts-workshop/tree/master/02-CreatingDockerImage#creating-your-first-image)
* [3. Setting up the IAM Roles](https://github.com/bemer/lts-workshop/tree/master/02-CreatingDockerImage#setting-up-the-iam-roles)
* [4. Configuring the AWS CLI](https://github.com/bemer/lts-workshop/tree/master/02-CreatingDockerImage#configuring-the-aws-cli)
* [5. Creating the Container registries with ECR](https://github.com/bemer/lts-workshop/tree/master/02-CreatingDockerImage#creating-the-container-registries-with-ecr)
* [6. Pushing our tested images to ECR](https://github.com/bemer/lts-workshop/tree/master/02-CreatingDockerImage#pushing-our-tested-images-to-ecr)

## 1. Tutorial overview

This tutorial aims to help you to deploy an web application using AWS Fargate. This application is called scorekeep, which is  a RESTful web API implemented in Java that uses Spring to provide an HTTP interface for creating and managing game sessions and users. This project includes the Scorekeep API and a front-end web app that consumes it.

> We didn't developed this application. If you want to get more information about it, you can see the original repository at this URL: https://github.com/awslabs/eb-java-scorekeep/.

After running this tutorial you will have a serverless application running in AWS Fargate.

## 2. Provisioning the infrastructure

To deploy the *Scorekeep** application, you will need to create some infrastructure. This includes DynamoDB tables and a SNS topic that will receive messages. Since this tutorial is more focused in the AWS Fargate usage, we have a CloudFormation template that will provision everything for you.

In this project folder, go to the directory `cloudformation` and run the following command to start a deployment of a new cloudformation stack in your account:

    aws --region us-east-1 cloudformation create-stack --stack-name lts-scorekeep --capabilities "CAPABILITY_NAMED_IAM" --template-body file://cf-resources.yaml

You can check the CloudFormation creating process in your console, on the CloudFormation service screen:

![cloudformation provisioning](https://github.com/bemer/lts-workshop/blob/master/04-DeployFargate/images/cloudformation_creation.png)
