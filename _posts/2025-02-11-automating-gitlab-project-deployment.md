---
title: "Automating GitLab project deployment with GitLab pipelines and AWS ECR"
date: 2025-02-11T01:04:06-09:00
categories:
  - AWS
  - CICD
  - ECR
  - GitLab
  - Pipeline
classes: wide
excerpt: In software development phases, getting a project from code to deployment involves a lot of teamwork.
---

## Introduction
In software development phases, getting a project from code to deployment involves a lot of teamwork. When new features are rolled out frequently, the deployment process can start to feel like a never-ending task for DevOps teams. That’s where Continuous Integration and Continuous Deployment (CI/CD) come in handy. By automating the deployment process, we can save time and effort, allowing us to build, test, provision resources, and deploy to the cloud automatically whenever there’s a change in the code.

In this blog, we will explore how GitLab pipelines can be leveraged to automate project deployment. Specifically, we will focus on using GitLab pipelines to monitor code changes, build projects using Docker, and push Docker images to Amazon Elastic Container Registry (ECR).

## Project Setups
For this blog, I’ll be using my existing REST API codebase, which will be transferred to GitLab. To set up the GitLab project, you can follow these steps:

+ Click on “New Project”: Locate and click the “New Project” button on the right-hand side of your GitLab dashboard.
+ Select “Create a Blank Project”: Choose the option to create a blank project to start from scratch.
+ Provide a Unique Project Name: Enter a unique name for your project to easily identify it.
+ Select Project Deployment Target (Optional): If needed, choose a deployment target for your project. This step is optional.
+ Choose Visibility Level: Decide on the visibility level for your project (In my case, I have set it to public).
+ Configure Project Settings: Set up any additional project configurations as needed to suit your requirements.

![create-new-project](/images/blog-images/automating-gitlab-project/create-new-project.png)

<p style="text-align: center;">Repository Creation Page in GitLab</p>


![demo-api](/images/blog-images/automating-gitlab-project/demo-api.png)

<p style="text-align: center;">Creating a new respoitory</p>

Once your new project is set up in GitLab, the next step is to clone this empty repository to your local machine using HTTPS. This will create a local copy of the project where you can work. After cloning, you can easily transfer the files from your existing codebase into this newly created project directory. Once all the necessary files are in place, it’s time to commit these changes and push to the remote repository.

![clone-with-ssh](/images/blog-images/automating-gitlab-project/clone-with-ssh.png)

![base](/images/blog-images/automating-gitlab-project/base.png)

<p style="text-align: center;">Clone empty respository</p>


## Setting up ECR
To get started with Amazon Elastic Container Registry (ECR), goto to the AWS Management Console. Once there, navigate to the ECR service to create a new repository. I will be creating a repository in the N.Virginia region. This will serve as the destination for our Docker images, allowing us to integrate with our GitLab pipeline for automated deployments.

![private-repositories](/images/blog-images/automating-gitlab-project/private-repositories.png)

<p style="text-align: center;">Creating ECR repository</p>


## Creating Access Credentials
To enable GitLab pipelines to interact with AWS ECR, you’ll need to set up access credentials. We will start by creating a new IAM user in the AWS Management Console, specifically for accessing ECR. Once the user is created, attach the predefined policy **“AmazonEC2ContainerRegistryPowerUser”** to grant the necessary permissions for managing ECR resources. After setting up the user, generate an Access Key ID and a Secret Access Key, which will be used to authenticate GitLab pipelines with AWS.

Note: You can restrict the access in the policy based on your requirement. In this blog, I am considering Full Access for demonstration purposes only.

![test-ecr-user](/images/blog-images/automating-gitlab-project/test-ecr-user.png)

<p style="text-align: center;">Creating Access Credentials</p>


## GitLab Pipelines
To configure AWS credentials in your GitLab project, go to the project settings and select “CI/CD.” Under the “Variables” section, add new variables for your AWS credentials, such as “AWS_ACCESS_KEY_ID”, “AWS_SECRET_ACCESS_KEY, “”AWS_DEFAULT_REGION” and “GITLAB_AWS_ACCOUNT_ID”, with their respective values.

![save-changes](/images/blog-images/automating-gitlab-project/save-changes.png)

<p style="text-align: center;">Setting Variables for pipelines</p>


To set up your GitLab CI/CD pipeline, create a .gitlab-ci.yml file in the root directory of your project. This file will define the pipeline configuration with the following components:

```
variables:
   DOCKER_REGISTRY: 422244203993.dkr.ecr.us-east-1.amazonaws.com
   APP_NAME: marvel-repo
   DOCKER_HOST: tcp://docker:2375
   TAG: "latest"

publish:
    image:
        name: amazon/aws-cli
        entrypoint: [""]
    services:
        - docker:dind
    before_script:
        - amazon-linux-extras install docker
        - aws --version
        - docker --version
    script:
        - docker build -t $DOCKER_REGISTRY/$APP_NAME:"$TAG" -f Dockerfile .
        - aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin $DOCKER_REGISTRY
        - docker push $DOCKER_REGISTRY/$APP_NAME:"$TAG"
```

This configuration sets up a pipeline that builds a Docker image from your project, logs into AWS ECR, and pushes the image to your specified ECR repository. The variables section defines key parameters like the Docker registry URL, application name, and tag. The publish job uses the AWS CLI Docker image and Docker-in-Docker service to perform these tasks.

With this configuration in place, any changes made to the code will automatically trigger the pipeline. This process will build a new Docker image and push it to your specified AWS ECR repository, which reduces the overhead of building and pushing the image to ECR, whenever there are any new updates in code.