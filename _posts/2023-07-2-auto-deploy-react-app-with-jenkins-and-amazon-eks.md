---
title: "Auto Deploy React app with Jenkins and Amazon Elastic Kubernetes Service (Amazon EKS)"
date: 2023-07-02T08:00:35-08:00
categories:
  - Elastic Container Service
  - Elastic Kubernetes Service
  - Jenkins
  - React
  - Bitbucket
classes: wide
excerpt: CI/CD is the process of automating the software development life cycle where new codes are built, tested and merged, then they are deployed to the target environment.

---
## Introduction
CI/CD is the process of automating the software development life cycle where new codes are built, tested and merged, then they are deployed to the target environment. One of the tools providing such a feature is Jenkins and Kubernetes is a very popular deployment management software. In this post, I will provide a step-by-step guide to deploying React applications in the Elastic Kubernetes Service (EKS) and setting up CI/CD with Jenkins. Bitbucket will be used as a git repository and Jenkins will be hosted on the EC2. Let’s Begin.

**Prerequisites**: Bitbucket, EC2, ECR and EKS

## Setup
### Setting Up Jenkins
To install Jenkins, we will first create an EC2 instance with Ubuntu AMI, where for feasibility, we will create an IAM role with **AdministratorAccess**. But feel free to have finer control over the role and allow only necessary permissions.

For the Jenkins installation, follow the following steps:

1. **Connect to the ec2**: `ssh -i <.pem file> ubuntu@<public ip>`

2. `curl -fsSL` [https://pkg.jenkins.io/debian-stable/jenkins.io.key](https://pkg.jenkins.io/debian-stable/jenkins.io.key) `| sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null`

3. `echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]` [https://pkg.jenkins.io/debian-stable](https://pkg.jenkins.io/debian-stable/) `binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null`

4. sudo apt-get update
5. sudo apt-get install fontconfig openjdk-11-jre
6. sudo apt-get install jenkins
7. Check installations: `ps -ef | grep jenkins`

Now, after the installation, you will also need to set up users to get access to the dashboard. First, go to <ip_addres:8080> in your browser where 8080 is the default port of Jenkins, you will see the following page.

![unlock-jenkins](/images/blog-images/auto-deploy-react-app-with-jenkins-and-eks/unlock-jenkins.png)

You will see that the administrator password is saved in a log file in host ec2 and the path to the file is written on the page. Execute the following command in the shell of the host machine.

`sudo cat /var/lib/jenkins/secrets/initialAdminPassword`

![install-admin-pass](/images/blog-images/auto-deploy-react-app-with-jenkins-and-eks/install-admin-pass.png)

The admin password will be displayed, then copy the password to the password field of the page in the browser. After clicking the Continue button, you will be asked to install the plugins. Choose “Install suggested plugins”.

![unlock-jenkins-with-pass](/images/blog-images/auto-deploy-react-app-with-jenkins-and-eks/unlock-jenkins-with-pass.png)

Now, you will be redirected to the account creation form where you can create the admin user, so fill in all the details in it.

![create-first-user-admin](/images/blog-images/auto-deploy-react-app-with-jenkins-and-eks/create-first-user-admin.png)

After this, click on **“Save and Finish”**. The Jenkins setup is complete and you can start using it for your projects.

### Setting Up React Project, ECR and EKS
Refer to my previous blog to set up ECR and EKS in this [link](https://medium.com/@sasmigrg93/deploy-react-app-into-amazon-elastic-kubernetes-service-amazon-eks-a2d7d383fbee).

## Plugins Installation and Setting up Variables in Jenkins
When you have Jenkins installed and the setup is completed, we need to install the necessary plugins for our automation. For installation, follow the following steps:

1. Go to **Manage Jenkins** and select **Manage Plugins**.
2. Choose **Available plugins** then search these plugins: Docker Plugin, Docker Pipeline, NodeJS Plugin, Kubernetes, Bitbucket WebHook
3. Click **Install without restart** or **Download now and Install after restart**.

Jenkins needs access to the code repository in order to pull the code changes. In our case, we are using BitBucket, so we can set an **APP Password** which then can be used by Jenkins to access the BitBucket. For this, Login into a bitbucket account and go to Personal settings then choose APP Password. Click on Create app password then provide Label name and select all the necessary permissions and Click on Create.

**NOTE**: Copy and paste the password into a secure place.

![bitbucket](/images/blog-images/auto-deploy-react-app-with-jenkins-and-eks/bitbucket.png)

Now, in the Jenkins Dashboard, Go to **Manage Jenkins**, and choose **Manage Credentials**. Click on global then choose **Add Credentials**.

1. In the **“Kind”** option, select **“Username with Password”**
2. In the **“Scope”** option, select **“Global”**
3. In the **“Username”** and **“Password”**, put the credentials set in APP Password in BitBucket.

![new-credentials](/images/blog-images/auto-deploy-react-app-with-jenkins-and-eks/new-credentials.png)

To deploy codes into the EKS, Jenkins needs access to the EKS and credentials of aws are added to the environment variables. To add environment variables, go to Manage Jenkins again and click on Configure System. Here select Environment variables then click on Add and add these variables:

1. AWS_DEFAULT_REGION = “ ”
2. AWS_ACCESS_KEY_ID = “ ”
3. AWS_SECRECT_ACCESS_KEY =” ”

After adding these variables, Click on Save and your variables will be added.

To create a new project in Jenkins, click on **“New Item”** and select **“Freestyle Project”**.

![enter-an-item](/images/blog-images/auto-deploy-react-app-with-jenkins-and-eks/enter-an-item.png)

After this, click on the **“Configure”** and select **“Source Code Management”**. In here, select “Git” and perform the following steps:

1. Provide the **“Repository URL”** of the BitBucket
2. Chose the **“Credentials”** that we previously created
3. Select the branch of the project in the BitBucket.

![configure](/images/blog-images/auto-deploy-react-app-with-jenkins-and-eks/configure.png)

After this, we need to configure trigger mechanism. So, for this, select **“Build Trigggers”** and Select **“Build when a change is pushed to BitBucket”**. Now, goto **“Build Steps”**, then in the **“Add build step”**, select **“Execute Shell”**. Add the following command which will trigger our build script. Then click on apply:

**“sh build.sh”**

![build-steps](/images/blog-images/auto-deploy-react-app-with-jenkins-and-eks/build-steps.png)

We need to create a webhook in BitBucket which will trigger our Jenkins whenever there are any changes in the code in the repository. While creating the webhook, we need to put **`“http://<ip address>:8080/bitbucket-hook”`** in the URL section and select **"Push”** in the Trigger section.

![edit-webhook](/images/blog-images/auto-deploy-react-app-with-jenkins-and-eks/edit-webhook.png)

Now the setup is complete and every time when changes are pushed to the BitBucket, it will automatically deploy it into the EKS.

## Conclusion
In this post, we learned about how to deploy the React App with CI/CD pipeline. We created a simple React App and looked into setting up all the tools. We used Jenkins to manage our CI/CD pipeline and our scripts were placed into the BitBucket. The processing of building the React app, creating images, and pushing to ECR is automated using Jenkins. Also, whenever there are some changes, these steps in Jenkins are triggered and the app is deployed to EKS.