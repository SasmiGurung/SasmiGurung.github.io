---
title: "Setup Guide for Site-to-Site VPN using OpenSwan"
date: 2024-02-04T08:05:03-02:00
categories:
  - AWS
  - Openswan
  - VPN
  - EC2 Instance
  - Linux

classes: wide
excerpt: Virtual Private Network (VPN) can help to achieve private access and establish encrypted communications.
---

## Introduction
With the rising cyber threats, demands for encrypted and private communication for resource access and transfers have increased. Virtual Private Network (VPN) can help to achieve private access and establish encrypted communications.

Site-to-Site VPN is a VPN connection that establishes secure connections between two or more networks across different locations. It enables the creation of a virtual encrypted tunnel between the networks, typically over the internet, ensuring the data transmitted remains private and accessible to authorized users only.

In this step-by-step tutorial, I will show the setup process to create a site-to-site VPN connection between an AWS EC2 instance and OpenSwan (an open-source implementation of IPsec) for Linux-based systems.

## Prerequisites

+ AWS Account

## Creating the VPC

+ Go to the Amazon VPC console at: [https://console.aws.amazon.com/vpc/home?region=us-east-1](https://console.aws.amazon.com/vpc/home?region=us-east-1)
+ Click on **Your VPCs**, then Select **Create VPC**
+ Under VPC settings, select **VPC Only**, provide **Name tag** and **IPv4 CIDR**

![create-vpc](/images/blog-images/site-to-site-vpn-using-openswan/create-vpc.png)

+ Click on **Create VPC**

![your-vpc](/images/blog-images/site-to-site-vpn-using-openswan/your-vpc.png)

## Creating the Subnet

+ Choose **Subnets** from the VPC dashboard, click on **Create subnet**
+ Under **Create subnet**, select **VPC ID**, provide **Subnet name** and **IPv4 CIDR block**

![create-subnet](/images/blog-images/site-to-site-vpn-using-openswan/create-subnet.png)

+ Click on **Create subnet**

![subnets](/images/blog-images/site-to-site-vpn-using-openswan/subnets.png)

## Creating an Internet Gateway
+ Now again, Go to the Amazon **VPC Dashboard**
+ Choose **Internet gateways**, click on **Create internet gateway**
+ Under **Internet gateway settings**, provide a **Name tag**

![create-internet-gateway](/images/blog-images/site-to-site-vpn-using-openswan/create-internet-gateway.png)

+ Click on **Create internet gateway**

![detached-internet-gateway](/images/blog-images/site-to-site-vpn-using-openswan/detached-internet-gateway.png)

## Attaching internet gateway with VPC
+ Select the newly created internet gateway, Choose **Actions**
+ Select **Attach to VPC**
+ Under VPC, select **Available VPCs**

![attach-to-vpc](/images/blog-images/site-to-site-vpn-using-openswan/attach-to-vpc.png)

+ Click on **Attach Internet gateway**

![attached-internet-gateway](/images/blog-images/site-to-site-vpn-using-openswan/attached-internet-gateway.png)

## Creating Route Table
+ From the **VPC Dashboard**, Select **Route table**
+ To create a route table, click on **Create route table**
+ Under **Route table settings**, Enter **Name** (optional) and Select **VPC**

![create-route-table](/images/blog-images/site-to-site-vpn-using-openswan/create-route-table.png)

+ Click on the **Create route table** button

## Editing the Route Table
+ Now from the Route Tables, choose Routes, then click on **Action** and select **Edit Routes**

+ Under **Edit routes**, click on **Add route**, and select **0.0.0.0/0**

![edit-route](/images/blog-images/site-to-site-vpn-using-openswan/edit-route.png)

+ Click on **Save changes**

## Creating ec2 instance
+ Go to **EC2 Dashboard**
+ Click on **Launch instances**
+ Give a name for your instance in **Name**, Select **AMI, Instance Type**, and **Key pair**
+ Under **Configure Instance**, Select **Network** and **Subnet**

![configure-instance-details](/images/blog-images/site-to-site-vpn-using-openswan/configure-instance-details.png)

+ Configure the **Security group** as shown in the following screenshot

![configure-security-group](/images/blog-images/site-to-site-vpn-using-openswan/configure-security-group.png)

+ Choose **Launch instances**

Now, we need another ec2 instance with a similar setup as above. We will repeat the same process from the previous steps:

+ Creating the VPC
+ Creating the Subnet
+ Creating an Internet Gateway
+ Attaching internet gateway with VPC
+ Creating Route Table
+ Editing the Route Table
+ Creating ec2 instance

## Creating a Virtual Private Network Resources
+ Go to the **VPC dashboard**
+ On the left side pane, go to **Virtual Private Network** and Select Virtual private gateways. Click on **Create Virtual private gateway**
+ Enter **Name tag**

![create-virtual-private-gateway](/images/blog-images/site-to-site-vpn-using-openswan/create-virtual-private-gateway.png)

+ Click on Create Virtual Private Gateway

## Attaching Virtual Private Gateway toVPC

+ Select virtual private gateway, Choose **Actions**, and click on **Attach to VPC**
+ Under the **Available VPC**, choose your created vpc

![attached-to-vpc](/images/blog-images/site-to-site-vpn-using-openswan/attached-to-vpc.png)

+ Click on the **Attach to VPC**

## Creating a Customer Gateway
+ On the VPN section of the VPC dashboard, click on **Customer Gateways**
+ Click on **Create Customer Gateway**. Enter **Name** and **IP Address**

![create-customer-gateway](/images/blog-images/site-to-site-vpn-using-openswan/create-customer-gateway.png)

+ Click on **Create Customer Gateway**

## Changing the Source or Destination
+ Go to the ec2 dashboard, select the launched ec2 instance, and click on **Actions**
+ Select **Networking**, click on **Change Source/Dest check**
+ Click the **stop** check box

![source-check](/images/blog-images/site-to-site-vpn-using-openswan/source-check.png)

+ Click on the **Save** button

## Creating Site-to-Site VPN Connections
+ Go to the VPC console
+ On the **Site-to-Site VPN Connections**, Select **Create VPN Connection**
+ Under **Create VPN Connection**, Enter **Name tag**, Select **Virtual Private Gateway**, Select **Customer Gateway ID**, Choose **Routing Options**(static) and Enter Static IP Prefixes (20.0.0.0/16)

![create-vpn-connection](/images/blog-images/site-to-site-vpn-using-openswan/create-vpn-connection.png)

+ Click on **Create VPN Connection**
+ Select **Download Configuration**, Select **Generic**

![download-configuration](/images/blog-images/site-to-site-vpn-using-openswan/download-configuration.png)

+ Click on the Download

## Enabling route propagation
+ Go to **Route Table** and click on **Actions**
+ Select **Edit route propagation** and **Enable** Propagation
+ Click on the **Save** button

**Installing openswan**

+ ssh into the ec2 instance with the IP address and the key pair file
+ To install openswan in Amazon Linux, use the command: `yum install openswan`

![install-openswan](/images/blog-images/site-to-site-vpn-using-openswan/install-openswan.png)

+ Configure ipsec.conf file by using the command `nano /etc/ipsec.conf` and uncomment the line `include/etc/ipsec.d/*.conf`
+ Run the command to configure the file: `nano/etc/ipsec.conf`

![ipsec-conf](/images/blog-images/site-to-site-vpn-using-openswan/ipsec-conf.png)

+ Configure /etc/sysctl.conf by adding code lines as shown in the figure below:

![include-ipsec](/images/blog-images/site-to-site-vpn-using-openswan/include-ipsec.png)

+ Run the command below to configure the file /etc/ipsec.d/aws.conf : `sudo nano /etc/ipsec.d/aws.conf`

+ And edit leftid with (Customer Gateway) and rightid with (Virtual Private Gateway)

![gnu-nano](/images/blog-images/site-to-site-vpn-using-openswan/gnu-nano.png)

+ Configure and edit /etc/ipsec.d/aws.secrets

![aws-secrets](/images/blog-images/site-to-site-vpn-using-openswan/aws-secrets.png)

+ Configure system level /etc/sysctl.conf file: `nano/etc/sysctl.conf`- nano /etc/sysctl.conf

![sysctl-conf](/images/blog-images/site-to-site-vpn-using-openswan/sysctl-conf.png)

+ To Restart the network service, run the following steps:

![restart-network](/images/blog-images/site-to-site-vpn-using-openswan/restart-network.png)

+ Restart the network service by executing the command: `systemctl restart network`s
+ To Start ipsec service, run the command: `systemctl start ipsec`