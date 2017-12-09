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
    aws-cli/1.11.36 Python/2.7.13 Darwin/16.4.0 botocore/1.4.93

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


This should return:

     hi!  i'm served via Python + Flask.  i'm a web endpoint.
