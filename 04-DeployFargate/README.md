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

In this project folder, go to the `cloudformation` directory inside this project and run the following command to start a deployment of a new cloudformation stack in your account:

    aws --region us-east-1 cloudformation create-stack --stack-name lts-scorekeep --capabilities "CAPABILITY_NAMED_IAM" --template-body file://cf-resources.yaml

You can check the CloudFormation creating process in your console, on the CloudFormation service screen:

![cloudformation provisioning](https://github.com/bemer/lts-workshop/blob/master/04-DeployFargate/images/cloudformation_creation.png)

This cloudformation will create a few DynamoDB Tables, a Cloudwatch LogGroup and a SNS Topic.

Now that we already have our infrastructure provisioned, we need to create a new IAM Role that will allow our application containers to access this services.

Go to the IAM console, select **Roles** in the left corner of the page and click in **Create role**. In this screen, select **EC2 Container Service** and under **use case** select **EC2 Container Service Task** and click in **Next: Permissions**:

![create taks role](https://github.com/bemer/lts-workshop/blob/master/04-DeployFargate/images/task_role_creation.png)

In the next screen, click in **Create policy**, them select **JSON** and add the following Policy document (remember to replace your account ID):

    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "SNSRole",
                "Effect": "Allow",
                "Action": "sns:\*",
                "Resource": "arn:aws:sns:us-east-1:<your_account_id>:scorekeep-notifications"
            },
            {
                "Sid": "DynamoDbRole",
                "Effect": "Allow",
                "Action": "dynamodb:\*",
                "Resource": [
                    "arn:aws:dynamodb:us-east-1:<your_account_id>:table/scorekeep-game",
                    "arn:aws:dynamodb:us-east-1:<your_account_id>:table/scorekeep-move",
                    "arn:aws:dynamodb:us-east-1:<your_account_id>:table/scorekeep-session",
                    "arn:aws:dynamodb:us-east-1:<your_account_id>:table/scorekeep-state",
                    "arn:aws:dynamodb:us-east-1:<your_account_id>:table/scorekeep-user"
                ]
            }
        ]
    }

![create taks policy](https://github.com/bemer/lts-workshop/blob/master/04-DeployFargate/images/task_policy_creation.png)

Click in **Review policy**, name your policy `lts-scorekeep-policy` and click in **Create policy**:

![finish policy creation ](https://github.com/bemer/lts-workshop/blob/master/04-DeployFargate/images/finish_policy_creation.png)


## 3. Building the Docker images

Now that we have the infrastructure, let's create the scorekeeper docker containers. As mentioned earlier, this application depends on two containers: one with the frontend and another with the backend API. Let's start creating our backend container.

Go to the `scorekeep-api` folder and run the following command to build your image:

    docker build -t scorekeep-api .

Take a minute to look at the `Dockerfile` used to build this image. Note that it is using a base image called `openjdk:alpine`. The `alpine linux` is a minimal Docker image based on Alpine Linux with a complete package index and only 5 MB in size.

Just after this, we are adding the `scorekeep-api-1.0.0.jar` to our container. The source code of this file is available under the orifinal repository, but we already builded it for you.

We are them setting a few environment variables, exposing the port 5000 and informing the entrypoint for this container.

After provisioning this container, go to the ECS console, under **Repositories** and create a new one called `scorekeep-api` and follow the procedures to push your image `scorekeep-api`. If you don't remember how to upload your image, follow the same steps described in the [`02-CreatingDockerImage`](https://github.com/bemer/lts-workshop/tree/master/02-CreatingDockerImage#pushing-our-tested-images-to-ecr) tutorial. Remember to use the `scorekeep-api` as your image name.

Let's now create our frontend image.

Go to the `scorekeep-frontend` directory and run the following command:

    docker build -t scorekeep-frontend .

After building the image, you can test it to see if it runs locally. In order to execute this test, use the following command:

    docker run -p 80:8080 scorekeepe-frontend

Them, using your browser, access the url http://localhost. You will see the scorekeep frontend interface:

![scorekeep front local](https://github.com/bemer/lts-workshop/blob/master/04-DeployFargate/images/scorekeep-frontend-local.png)
