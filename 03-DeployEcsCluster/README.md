# Running an ECS Cluster

**Quick jump:**

* [Tutorial overview](https://github.com/bemer/lts-workshop/tree/master/02-CreatingDockerImage#tutorial-overview)
* [Creating your first image](https://github.com/bemer/lts-workshop/tree/master/02-CreatingDockerImage#creating-your-first-image)
* [Setting up the IAM Roles](https://github.com/bemer/lts-workshop/tree/master/02-CreatingDockerImage#setting-up-the-iam-roles)
* [Configuring the AWS CLI](https://github.com/bemer/lts-workshop/tree/master/02-CreatingDockerImage#configuring-the-aws-cli)
* [Creating the Container registries with ECR](https://github.com/bemer/lts-workshop/tree/master/02-CreatingDockerImage#creating-the-container-registries-with-ecr)
* [Pushing our tested images to ECR](https://github.com/bemer/lts-workshop/tree/master/02-CreatingDockerImage#pushing-our-tested-images-to-ecr)

## Tutorial overview

This tutorial will guide you through the creation of an AWS ECS Cluster and the deployment of a container with a simple Python application created in the previous lab.

In order to run this tutorial, you must have completed the following steps:

* [Setup Environment](https://github.com/bemer/lts-workshop/tree/master/01-SetupEnvironment)
* [Creating your Docker image](https://github.com/bemer/lts-workshop/tree/master/02-CreatingDockerImage)


## Setting up the Cluster

Once you've signed into your AWS account, navigate to the [ECS console](https://console.aws.amazon.com/ecs/home?region=us-east-1#/clusters). This URL will redirect you to the ECS interface. If this is your fist time using this service, you will see the "*Clusters*" screen without any clusters in it:

![clusters screen](https://github.com/bemer/lts-workshop/blob/master/03-DeployEcsCluster/images/clusters_screen.png)

Let's them create our first ECS Cluster. Click in the button "*Create cluster*" and in the following screen select the "*EC2 Linux + Networking*" cluster template:

![cluster template](https://github.com/bemer/lts-workshop/blob/master/03-DeployEcsCluster/images/cluster_template.png)

You will them be asked to input information about your new cluster. Fill the fields in the *Configure cluster* screen and note that here you can create a new VPC or select a existing one. We recommend you to create a new VPC using the wizard, so it is going to be easy clean up your account after the event.

Remember to select your keypair, so you will be able to access the EC2 instances in your cluster in order to perform any kind of troubleshooting later.

Note that this wizard is also going to create a security group for you allowing access in the port 80 (TCP).

Click in "*Create*".

When the creation process finishes, you will see the following screen:

![cluster created](https://github.com/bemer/lts-workshop/blob/master/03-DeployEcsCluster/images/cluster_created.png)
