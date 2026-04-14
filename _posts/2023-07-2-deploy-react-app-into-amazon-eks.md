---
title: "Deploy React App into Amazon Elastic Kubernetes Service (Amazon EKS)"
date: 2023-06-15T08:00:30-04:00
categories:
  - Elastic Container Service
  - Elastic Kubernetes Service
  - Docker
  - React
  - AWS
classes: wide
excerpt: In today’s fast-paced digital landscape, efficient and seamless deployment processes are crucial for organizations to stay ahead of the curve.
---

## Introduction
In today’s fast-paced digital landscape, efficient and seamless deployment processes are crucial for organizations to stay ahead of the curve. Cloud deployment has emerged as a game-changer, empowering developers to rapidly deploy and scale their applications with ease. Among the myriad of cloud deployment options, Amazon Elastic Container Registry (ECR) and Amazon Elastic Kubernetes Service (EKS) have gained significant popularity. In this blog post, we will explore the power of these two robust services by delving into the process of deploying a React app using ECR and EKS.

**Prerequisites**: ECR and EKS

Before beginning, we need a couple of essentials tools which are listed below with the link to documentation that will help in installations:

+ AWS CLI: [Link](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

After installing AWS CLI, configure the profile using the command **“aws configure”** and enter the appropriate credentials. Find more about this in this [link](https://docs.aws.amazon.com/cli/latest/reference/configure/).

+ Kubectl: [Link](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)
+  Docker: [Link](https://docs.docker.com/engine/install/ubuntu/)

Note: There are also post-installation steps for docker, follow this [link](https://docs.docker.com/engine/install/linux-postinstall/).

+  ReactJS: [Link](https://www.tecmint.com/install-reactjs-on-ubuntu/)

## Setting Up ECR and EKS
From the AWS console, we can create the ECS and EKS clusters. For EKS, select Kubernetes Version: **v1.23.**

While creating an EKS cluster, we need to create an IAM role for the EKS cluster. In the IAM role, you must add “AmazonEKSClusterPolicy” with a given inline policy. This policy will allow the EKS to access the ECR and image.

This policy will allow the EKS to access the ECR and image.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ecr:BatchCheckLayerAvailability",
                "ecr:BatchGetImage",
                "ecr:GetDownloadUrlForLayer",
                "ecr:GetAuthorizationToken"
            ],
            "Resource": "*"
        }
    ]
}
```
Then select “public” for endpoint access control.
We also need to create a “Node Group”. For this, follow the given instructions:


+ Click on manage nodegroup, provide the node group name

+ Create an IAM role and attach the “AmazonEKSWorkerNodePolicy” and “AmazonEC2ContainerRegistryReadOnly” policies. Also, add the “AmazonEKS_CNI_Policy” policy to this role. 

+ Ensure the “Trust Relationship” is matched, otherwise, copy the following content. Then go to the Edit trust policy window, paste the content and choose Update policy.


```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

## Deploying React App
We need to create a React App, so refer to [this](https://create-react-app.dev/docs/getting-started/) documentation to create a new React Project. After this, the project needs to be containerized to ship it to ECR which then can be accessed by the EKS. For this, create a docker file with this command, “**touch Dockerfile**”. Now, we need to put the following content inside the Dockerfile:

```
FROM node:19.6.1-bullseye-slim as base

COPY . /expjenkins
WORKDIR /expjenkins

RUN apt-get -y update
RUN npm install && npm run build
CMD ["npm", "start"]
```

This Dockerfile pulls nodes as base images, copies our code to the directory, runs updates, installs the required installation for React, builds the app and starts the server.

The docker image can be created with the following command:

`docker build -t react-app:latest -f Dockerfile .`

After creating ECR, there are instructions provided when you click the **"View push commands"** button which will guide you on how to push the image to ECR. First, authorization is needed for the docker to access the ECR. The below commands show how to tag the local image and push it to ECR.

+ `aws ecr get-login-password —region us-east-1 | docker login —username AWS —password-stdin` **`<ECR repository url>`**

+ `docker tag jenkins-react:latest` **`<ECR repository url>`** `/<image tag name in ECR>:latest`

+ `docker push` **`<ECR repository url>`** `/<image tag name in ECR>:latest`

![push-command](/images/blog-images/deploy-react-app-into-eks/push-command-for-test.png)

The deployment file is needed to deploy the image into the EKS. It can be created with **"touch deployment.yml"**. In the “deployment.yml”, put these contents:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: react-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: react-app
  template:
    metadata:
      labels:
        app: react-app
    spec:
      containers:
      - name: react-app
        image: <image tag name in EKS>:latest
        resources:
          requests:
            memory: "256Mi"
            cpu: "1000m"
          limits:
            memory: "768Mi"
            cpu: "1500m"
        imagePullPolicy: Always
        ports:
        - containerPort: 3000
---
apiVersion: v1
kind: Service
metadata:
  name: react-app
spec:
  type: NodePort
  selector:
    app: react-app
  ports:
  - port: 3000
    targetPort: 3000
    protocol: TCP
    nodePort: 31000
```

Now, we need to authorize our local machine to access the EKS. To authorize, use the following command:

**`aws eks update-kubeconfig —region <region name> —name <EKS cluster name>`**

We can now deploy our app into EKS with the following command:

`kubectl apply -f deployment.yml`

## Conclusion
In this blog post, we explored the capability of Amazon Elastic Container Registry (ECR) and Amazon Elastic Kubernetes Service (EKS) in streamlining the cloud deployment process for React applications. We created a basic React app, learned about setting up all the tools, built the docker image, pushed it to the ECR and deployed the app into the EKS.