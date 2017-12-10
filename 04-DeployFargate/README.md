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

Now that we already have our infrastructure provisioned, we need to create a new IAM Policy and apply it to a new IAM Role that will allow our application containers to access this services.

Go to the IAM console, select **Policies** and click in **Create policy**. Select **JSON** and add the following Policy document (remember to replace your account ID):

    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "SNSRole",
                "Effect": "Allow",
                "Action": "sns:\*",
                "Resource": "arn:aws:sns:us-east-1:996278879643:scorekeep-notifications"
            },
            {
                "Sid": "DynamoDbRole",
                "Effect": "Allow",
                "Action": "dynamodb:\*",
                "Resource": [
                    "arn:aws:dynamodb:us-east-1:996278879643:table/scorekeep-game\*",
                    "arn:aws:dynamodb:us-east-1:996278879643:table/scorekeep-move\*",
                    "arn:aws:dynamodb:us-east-1:996278879643:table/scorekeep-session\*",
                    "arn:aws:dynamodb:us-east-1:996278879643:table/scorekeep-state\*",
                    "arn:aws:dynamodb:us-east-1:996278879643:table/scorekeep-user\*"
                ]
            }
        ]
    }

Here is a screenshot of this screen:

![create taks policy](https://github.com/bemer/lts-workshop/blob/master/04-DeployFargate/images/task_policy_creation.png)


Click in **Review policy**, name your policy `lts-scorekeep-policy` and click in **Create policy**:

![finish policy creation ](https://github.com/bemer/lts-workshop/blob/master/04-DeployFargate/images/finish_policy_creation.png)

Now, go to the IAM console, select **Roles** in the left corner of the page and click in **Create role**. In this screen, select **EC2 Container Service** and under **use case** select **EC2 Container Service Task** and click in **Next: Permissions**:

![create taks role](https://github.com/bemer/lts-workshop/blob/master/04-DeployFargate/images/task_role_creation.png)

In the permissions screen, select the `lts-scorekeep-policy` which we just created and click in **Next: Review**

![associate role policy](https://github.com/bemer/lts-workshop/blob/master/04-DeployFargate/images/associate_role_policy.png)

Name your Role `lts-scorekeep-role` and click in **Create role**:

![role creation](https://github.com/bemer/lts-workshop/blob/master/04-DeployFargate/images/role_creation.png)



## 3. Building the Docker images

Now that we have the infrastructure, let's create the scorekeeper Docker images. As mentioned earlier, this application depends on two images: one with the frontend and another with the backend API. Let's start creating our backend container.

Go to the `scorekeep-api` folder and run the following command to build your image:

    docker build -t scorekeep-api .

Take a minute to look at the `Dockerfile` used to build this image. Note that it is using a base image called `openjdk:alpine`. The `alpine linux` is a minimal Docker image based on Alpine Linux with a complete package index and only 5 MB in size.

Just after this, we are adding the `scorekeep-api-1.0.0.jar` to our container. The source code of this file is available under the orifinal repository, but we already builded it for you.

We are them setting a few environment variables, exposing the port 5000 and informing the entrypoint for this container.

Let's now create our frontend image. Go to the `scorekeep-frontend` directory and run the following command:

    docker build -t scorekeep-frontend .

After building the image, you can test it to see if it runs locally. In order to execute this test, use the following command:

    docker run -p 80:8080 scorekeepe-frontend

Them, using your browser, access the url http://localhost. You will see the scorekeep frontend interface:

![scorekeep front local](https://github.com/bemer/lts-workshop/blob/master/04-DeployFargate/images/scorekeep-frontend-local.png)


## 4. Pushing your images to ECR

After creating your images, go to the ECS console, under **Repositories** and create a new one called `scorekeep-api`:

![scorekeep api repository](https://github.com/bemer/lts-workshop/blob/master/04-DeployFargate/images/scorekeep-api-repository.png)

Use the `aws-ecr` api call to get your repository credentials and execute your authentication, running the command:

    `aws ecr get-login --no-include-email --region us-east-1`

You should get the following result:

    WARNING! Using --password via the CLI is insecure. Use --password-stdin.
    Login Succeeded

Now, that your repository is created, tag your Docker image with the command:

    docker tag scorekeep-api:latest <account_id>.dkr.ecr.us-east-1.amazonaws.com/scorekeep-api:latest

And them push your image using the command:

    docker push <account_id>.dkr.ecr.us-east-1.amazonaws.com/scorekeep-api:latest

This will start the upload of your image to the ECR. You should see something similar to this in your terminal:

    $ docker push 996278879643.dkr.ecr.us-east-1.amazonaws.com/scorekeep-api:latest
    The push refers to a repository [996278879643.dkr.ecr.us-east-1.amazonaws.com/scorekeep-api]
    4e2b459f47ea: Pushing [==============================>                    ]  12.62MB/20.91MB
    69cc5717c281: Pushing [==========>                                        ]  20.35MB/97.4MB
    5b1e27e74327: Pushed
    04a094fe844e: Pushed

When completed, you will something like:

    The push refers to a repository [996278879643.dkr.ecr.us-east-1.amazonaws.com/scorekeep-api]
    4e2b459f47ea: Pushed
    69cc5717c281: Pushed
    5b1e27e74327: Pushed
    04a094fe844e: Pushed
    latest: digest: sha256:9caa0d1508ea59ed1e13eb52ea90fd988c1dd5e0fec46ebda34c4eddc3679120 size: 1159

Now, follow these same steps to your `scorekeep-frontend` image, creating a new repository, tagging and uploading the image.

## 5. Creating the Cluster

Let's create a new cluster to run our containers. In your AWS account dashboard, navigate to the [ECS console](https://console.aws.amazon.com/ecs/home?region=us-east-1#/clusters).

Click in the button **Create cluster** and in the following screen select the **Networking only (Powered by AWS Fargate)** cluster template:

![cluster template](https://github.com/bemer/lts-workshop/blob/master/04-DeployFargate/images/cluster_template.png)

In the **Cluster configuration** screen, add the name `lts-scorekeep-app` and click in create:

![cluster configuration](https://github.com/bemer/lts-workshop/blob/master/04-DeployFargate/images/cluster_configuration.png)

## 6. Creating the Task Definition


To create a Task Definition, choose **Task Definitions** from the ECS console menu.  Then, choose **Create new Task Definition**. Select `FARGATE` as the *Launch type compatibility*:

![type compatibility](https://github.com/bemer/lts-workshop/blob/master/04-DeployFargate/images/task_compatibility.png)

Click in **Next Step** and name your task **lts-scorekeep-app**. In the **Task Role**, select the role we created before `lts-scorekeep-role` and in the **Task execution role** select `ecsTaskExecutionRole`:

![task configuration](https://github.com/bemer/lts-workshop/blob/master/04-DeployFargate/images/task_configuration.png)

In **Task size** select `2GBI` in **Task memory (GB)** and `1vCPU` in **Task CPU (vCPU)** click in **Add container**:

![task size](https://github.com/bemer/lts-workshop/blob/master/04-DeployFargate/images/task_size.png)

Let's first add our API container. In **Container name** insert `scorekeep-api` and add the URL for the scorekeep-api from your ERC registry with the tag `latest`. Set the memory **Soft limit** to `512` and in the port `5000` in **Port mappings**:

![scorekeep-api standard configurations](https://github.com/bemer/lts-workshop/blob/master/04-DeployFargate/images/api_standard_configurations.png)

Under **Advanced container configuration** add `768` in **CPU Units** and create a new environment variable called `NOTIFICATION_TOPIC` where the value must be the ARN of the SNS Topic created by the CloudFormation template:

![scorekeep-api advanced configurations](https://github.com/bemer/lts-workshop/blob/master/04-DeployFargate/images/api_advanced_configurations.png)

Now, click again in **Add container** to add the scorekeep-frontend container.

In **Container name** insert `scorekeep-frontend`, in **Image** the URL of your scorekeep-frontend in the ECR repository with the tag `latest` and `512` under **Soft limit**. This container exposes the port `8080`, so add it to the **Port mappings** field. Under **Advanced container configuration** add `256` **CPU Units** and click in **Add**:

![scorekeep-frontend configurations](https://github.com/bemer/lts-workshop/blob/master/04-DeployFargate/images/frontend_configurations.png)

After adding both containers, click in **Create**:

![creating task](https://github.com/bemer/lts-workshop/blob/master/04-DeployFargate/images/creating_task.png)

## 7. Deploying the application

In the ECS console, click in **Clusters** and them click in the **lts-scorekeep-app** cluster that we created earlier. Under the **Services** tab, click in **Create**:

![service create](https://github.com/bemer/lts-workshop/blob/master/04-DeployFargate/images/service_create.png)

Select **FARGATE** as the **Launch type** and select the **Task definition** `lts-scorekeep-app:1` that we just created. Under **Platform version** select `LATEST`. Note that the cluster `lts-scorekeep-app` will be selected under **Cluster**. Don't change it. Add `lts-scorekeep-app` in **Service name** and set the number of tasks to `1`, them click in **Next step**:

![configure service](https://github.com/bemer/lts-workshop/blob/master/04-DeployFargate/images/configure_service.png)

Select the VPC where you want to run your containers and the subnets that you want to use. After, click in **Edit** to edit your security group rules:

![configure network](https://github.com/bemer/lts-workshop/blob/master/04-DeployFargate/images/configure_network.png)

Change the type of the rule to `Custom TCP` and add `8080` in **Port range**. This is because the frontend container is going to expose the port 8080 instead of 80. Click in **Save**:

![configure security group](https://github.com/bemer/lts-workshop/blob/master/04-DeployFargate/images/configure_security_group.png)

Leave the **Auto-assign public IP** as `ENABLED` and click in **Next step**. In this lab, we will not be using a load balancer.

In the **Set Auto Scaling (optional)** scree, just click in **Next step** and finally, click in **Create service** on the **Review** screen:

![service review](https://github.com/bemer/lts-workshop/blob/master/04-DeployFargate/images/service_review.png)

## 8. Acessing the application

After creating your service, wait a few seconds, so the ENI will be created and your containers will be executed. When finished, you will be able to see the number of running tasks in the `lts-scorekeep-app` Cluster screen:

![running tasks](https://github.com/bemer/lts-workshop/blob/master/04-DeployFargate/images/running_tasks.png)

Click in **Tasks** and them click in the Task ID. You will be redirected to a screen that shows all the infomation about your running task:

![task information](https://github.com/bemer/lts-workshop/blob/master/04-DeployFargate/images/task_information.png)

In this screen, click in the **ENI Id**. You will be redirected to the ENI screen. Here, get the **IPv4 Public IP**. This is the IP Address that you will use to access your application:

![eni information](https://github.com/bemer/lts-workshop/blob/master/04-DeployFargate/images/eni_information.png)

Open your Firefox browser and navigate to this IP in the port 8080. You should be able to access and interact with your application:

![application access](https://github.com/bemer/lts-workshop/blob/master/04-DeployFargate/images/application_access.png)

> For some reason this application is not working when accessed from Google Chrome. since we don't had enough time to fix it, let's use the Firefox browser. When fixing the problem, we will update this tutorial.

Click in **Create** them give a name to your game, select **Tic Tac Toe** under **Rules** and them click in **Create**. A new button will appear. Click in **Play** and start playing:

![game creation](https://github.com/bemer/lts-workshop/blob/master/04-DeployFargate/images/game_creation.png)

After starting your game and making a few movements in it, you should be able to see some information about your games in your DynamoDB tables. 
