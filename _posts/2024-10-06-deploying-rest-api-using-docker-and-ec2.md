---
title: "Deploying REST API using Docker and EC2"
date: 2024-10-06T02:06:08-04:00
categories:
  - AWS
  - Amazon EC2
  - Docker
  - REST API

classes: wide
excerpt: There are different methods for deploying APIs or web applications on AWS, including managed and self-hosted services.
---

## Introduction
There are different methods for deploying APIs or web applications on AWS, including managed and self-hosted services. Managed services, such as AWS Elastic Beanstalk, simplify deployment, while self-hosted approaches, like containerization with Elastic Kubernetes Service (EKS), offer more control .

For beginners, it’s essential to first understand how deployment works on a single compute instance before scaling to resources like EKS. In this tutorial, I will use Flask to create a REST API and deploy it on an EC2 instance.

I will walk you through the step-by-step process of deploying a Flask application on EC2 using Docker and making it accessible from the public internet. Let’s get started.

## Setups
We need access to an EC2 instance where Docker will be installed, and a Flask-based API server will be set up. I will use a publicly available Flask API codebase ([link here](https://github.com/RoshanGurungSr/demo-deployment)). If you want to learn more about Flask, refer to the official documentation ([link here](https://flask.palletsprojects.com/en/stable/quickstart/)).

## Launching an EC2 instance
Launching an EC2 instance using the AWS Management Console is straightforward. You can follow the official documentation ([link here](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-launch-instance-wizard.html)) for detailed instructions. Below is a quick summary of the steps:

+ Open the **AWS EC2 console** in your preferred region.
+ Click **“Launch Instances”** from the dashboard.
+ Provide a **name and tags** for the EC2 instance.
+ Select **Ubuntu Amazon Machine Image (AMI)**.
+ Choose **t2.medium** with at least **20 GB of EBS** for installing Docker and running the server.
+ Configure **security group rules:**
+ Allow **SSH access (port 22).**
+ Allow **port 5000**, since our Flask server will run on this port.
+ **Download the key pair** or use an existing one.
+ Set the correct permissions for the key pair using:

`chmod 400 path/to/file.pem`

## Setting up the project
Once the EC2 instance is ready, connect to it using SSH:

`sudo ssh -i “path to file/.pem” ubuntu@”public ip address”`

Now, clone the project from the github using the command:

`git clone https://github.com/RoshanGurungSr/demo-deployment`

## Installing Docker
Follow the given steps to install the docker in the ec2.

+ Update the system packages:

`sudo apt-get update`

+ Install docker:

`sudo apt-get install docker.io -y`

+ Start the docker service:

`sudo systemctl start docker`

+ Enable the docker service:

`sudo systemctl enable docker`

+ Verify the installation:

`docker –version`

## REST API with Flask
In the Flask application we are using, two routes have been created to test the API on the public interface. They are: default route “/” which displays welcoming message and “/home/<user_name>” which displays the strings that you send over the URL.

![rest-api](/images/blog-images/deploying-rest-api-docker-ec2/rest-api-with-flask.png)

## API Containerization and Deployment with Docker
The deployment process involves creating a **Dockerfile**, which contains all the necessary instructions for building a Docker image. This includes defining environment layers, installing required libraries, and starting the Flask server.

A **Dockerfile** has already been created for you in the cloned project, so we will use that file. Navigate to the project directory using:

`cd demo-deployment/`

## Building the Docker Image

To build the Docker image for the Flask application, run:

`sudo docker build -t demo-deployment-python-flask -f Dockerfile.`

## Running the Docker Container

Once the image is built, initialize a container and map the ports:

`sudo docker run -p 5000:5000 -d demo-deployment-python-flask`

## Accessing the Flask Application
After starting the container, open your browser and enter the public IP address of your EC2 instance. You should see the default route of your application.

![my-app](/images/blog-images/deploying-rest-api-docker-ec2/my-app.png)

You can also try using the “public-ip/home/test” route to check if it is displaying a welcome message with the strings.

![welcome](/images/blog-images/deploying-rest-api-docker-ec2/welcome.png)



