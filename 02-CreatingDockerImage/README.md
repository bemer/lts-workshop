# Creating your docker image

## Overview

This tutorial is going to drive you through the process of creating your first docker image, running a docker image locally and pushing it to a image repository.

In this tutorial, we assume that you completed the "Setup Environment" tutorial and:

* [Have a working AWS account](<https://aws.amazon.com>)
* [Have a working Github account](<https://www.github.com>)
* [Have the AWS CLI installed](<http://docs.aws.amazon.com/cli/latest/userguide/installing.html>)

To check if you have the AWS CLI installed and configured:

    $ aws --version

This should return something like:

    $ aws --version
    aws-cli/1.14.7 Python/2.7.12 Darwin/15.6.0 botocore/1.8.11

> Note that to run this tutorial, you need to have the most recent version of AWS CLI installed.

To check if you have Docker installed:

    $  which docker

This should return something like:

    $ which docker
    /usr/bin/docker

If you have completed these steps, you are good to go!

## Creating your first image

Clone this repository:

    $ git clone git@github.com:bemer/lts-workshop.git

Now we are going to build and test our containers locally.  If you've never worked with Docker before, there are a few basic commands that we'll use in this workshop, but you can find a more thorough list in the [Docker "Getting Started" documentation](https://docs.docker.com/engine/getstarted/).

To start your first container, go to the `app` directory in the project:

    $ cd <path/to/project>/02-CreatingDockerImage/app

And run the following command to build your image:

    $ docker build -t lts-demo-app .

This should output steps that look something like this:

    $ docker build -t lts-demo-app .
    Sending build context to Docker daemon  4.608kB
    Step 1/9 : FROM ubuntu:latest
     ---> 20c44cd7596f
    Step 2/9 : MAINTAINER brunemer@amazon.com
     ---> Running in 2d3745e79c4f
     ---> 06781c0980fb
    Removing intermediate container 2d3745e79c4f
    Step 3/9 : RUN apt-get update -y && apt-get install -y python-pip python-dev build-essential
     ---> Running in 7e3bf79f03a2

If the container builds successfully, the output should end with something like this:

     Removing intermediate container d2cd523c946a
     Successfully built ec59b8b825de

To run your container:

     $  docker run -d -p 3000:3000 lts-demo-app

To check if your container is running:

     $ docker ps

This should return a list of all the currently running containers.  In this example,  it should just return a single container, the one that we just started:

    CONTAINER ID        IMAGE                 COMMAND             CREATED              STATUS              PORTS                              NAMES
    fa922a2376d5        lts-demo-app:latest   "python app.py"     About a minute ago   Up About a minute   3000/tcp,    0.0.0.0:3000->3000/tcp   clever_shockley   

To test the actual container output, access the following URL in your web browser:

     http://localhost:3000/app

## Uploading your image to ECR

### Setting up your IAM roles

In order to work with the AWS CLI, you'll need an IAM role with the proper permissions set up.  To do this, we'll create both an IAM Group, and an IAM user.

To create the group, navigate to the IAM console, and select **Groups** > **Create New Group**.  Name the group "**lts-workshop**".  From the list of managed policies, add the following policies:

![add IAM group](https://github.com/bemer/lts-workshop/blob/master/02-CreatingDockerImage/images/iam-group-permissions.png)

Once you've created your group, you need to attach it to a user.  If you already have an existing user, you can add it to the lts-workshop group.  If you don't have a user, or need to create a new one, you can do so by going to **Users** > **Add User**.

If you are creating a new user, name it to something like "**lts-workshop-user**", so it is going to be easy delete all the assets used in this lab later. In the wizard, add your user to the "**lts-workshop**" group that we created in the previous step:

![add user to group](https://github.com/bemer/lts-workshop/blob/master/02-CreatingDockerImage/images/add_user_to_group.png)


When the wizard finishes, make sure to copy or download your access key and secret key.  You'll need them in the next step.

### Configuring the AWS CLI

If you've never configured the AWS CLI, the easiest way is by running:

    $ aws configure

This should drop you into a setup wizard:

    $ aws configure
    AWS Access Key ID [****************K2JA]:
    AWS Secret Access Key [****************Oqx+]:
    Default region name [us-east-1]:
    Default output format [json]:

If you already have a profile setup with the AWS CLI, you can also add a new profile to your credentials file. In order to add another profile, edit your credentials (usually located in *~/.aws/credentials*) and add a new profile called "**lts-workshop**". After adding this new profile, your credentials file will be like this:

    [default]
    aws_access_key_id = AKIAKSJF4ICN787JSCYA
    aws_secret_access_key = BQBy2lJ/p8s4lPKjoa7vYhTAh3lmUM59pf1zyokx

    [lts-workshop]
    aws_access_key_id = AKIAJKSJ2SXZFV7ABLDQ
    aws_secret_access_key = YCFxyhMI57fOb9wokfBR3TqxNu4h29L1P6gMdKTp


You can test that your IAM user has the correct permissions, and that your CLI is setup to connect to your AWS account by running the command to obtain an ECR authentication token.  This will allow us to pull our registries in the next step:

    $ aws ecr get-login --region us-east-1 --no-include-email --profile lts-workshop

This should output something like:

    $ docker login -u AWS -p AQECAHhwm0YaISJeRtJm5n1G6uqeekXuoXXPe5UFce9Rq8/14wAAAy0wggMpBgkqhkiG9w0BBwagggMaMIIDFgIBADCCAw8GCSqGSIb3DQEHATAeBglghkgBZQMEAS4wEQQM+76slnFaYrrZwLJyAgEQgIIC4LJKIDmvEDtJyr7jO661//6sX6cb2jeD/RP0IA03wh62YxFKqwRMk8gjOAc89ICxlNxQ6+cvwjewi+8/W+9xbv5+PPWfwGSAXQJSHx3IWfrbca4WSLXQf2BDq0CTtDc0+payiDdsXdR8gzvyM7YWIcKzgcRVjOjjoLJpXemQ9liPWe4HKp+D57zCcBvgUk131xCiwPzbmGTZ+xtE1GPK0tgNH3t9N5+XA2BYYhXQzkTGISVGGL6Wo1tiERz+WA2aRKE+Sb+FQ7YDDRDtOGj4MwZ3/uMnOZDcwu3uUfrURXdJVddTEdS3jfo3d7yVWhmXPet+3qwkISstIxG+V6IIzQyhtq3BXW/I7pwZB9ln/mDNlJVRh9Ps2jqoXUXg/j/shZxBPm33LV+MvUqiEBhkXa9cz3AaqIpc2gXyXYN3xgJUV7OupLVq2wrGQZWPVoBvHPwrt/DKsNs28oJ67L4kTiRoufye1KjZQAi3FIPtMLcUGjFf+ytxzEPuTvUk4Xfoc4A29qp9v2j98090Qx0CHD4ZKyj7bIL53jSpeeFDh9EXubeqp6idIwG9SpIL9AJfKxY7essZdk/0i/e4C+481XIM/IjiVkh/ZsJzuAPDIpa8fPRa5Gc8i9h0bioSHgYIpMlRkVmaAqH/Fmk+K00yG8USOAYtP6BmsFUvkBqmRtCJ/Sj+MHs+BrSP7VqPbO1ppTWZ6avl43DM0blG6W9uIxKC9SKBAqvPwr/CKz2LrOhyqn1WgtTXzaLFEd3ybilqhrcNtS16I5SFVI2ihmNbP3RRjmBeA6/QbreQsewQOfSk1u35YmwFxloqH3w/lPQrY1OD+kySrlGvXA3wupq6qlphGLEWeMC6CEQQKSiWbbQnLdFJazuwRUjSQlRvHDbe7XQTXdMzBZoBcC1Y99Kk4/nKprty2IeBvxPg+NRzg+1e0lkkqUu31oZ/AgdUcD8Db3qFjhXz4QhIZMGFogiJcmo= -e none https://<account_id>.dkr.ecr.us-east-1.amazonaws.com


> Specify if the '-e' flag should be included in the 'docker login' command. The '-e' option has been deprecated and is removed in docker version 17.06 and later. You must specify --no-include-email if you're using docker version 17.06 or later. The default behavior is to include the '-e' flag in the 'docker login' output.

To login to ECR, copy and paste that output or just run `` `aws ecr get-login --region us-east-1 --no-include-email --profile lts-workshop` `` which will tell your shell to execute the output of that command.  That should return something like:

    WARNING! Using --password via the CLI is insecure. Use --password-stdin.
    Login Succeeded

Note:  if you are running Ubuntu, it is possible that you will need to preface your Docker commands with `sudo`.  For more information on this, see the [Docker documentation](https://docs.docker.com/engine/installation/linux/ubuntu/).

If you are unable to login to ECR, check your IAM user group permissions.
