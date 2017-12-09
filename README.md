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
