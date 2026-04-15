---
title: "Automatic emptying s3 bucket monthly based on first specific day"
date: 2023-07-03T08:00:24-09:00
categories:
  - AWS Lambda
  - AWS Eventbridge
  - Simple Storage Service
  - Automation
classes: wide
excerpt: In this post, I will guide you in setting up an automated pipeline to remove data from the bucket on the first Tuesday of the month.

---
## Introduction
AWS Simple Storage Service (S3) is a cost effective object storage service which allows to store any amount of data. The pricing is charged based on the size of data and data requests/retrievals frequencies. When there is only a need for short retention of data, storing all the data longer is a waste of resources. But removing the data during an interval is also time consuming and manual work. So, in this post, I will guide you in setting up an automated pipeline to remove data from the bucket on the first Tuesday of the month.

## Prerequisites: S3, Lambda function, EventBridge
We can create a s3 bucket from aws console. After this, create an IAM Role with policies: **“AmazonS3FullAccess”** and **“CloudWatchFullAccess”**. Now we can create a Lambda function with **“Python 3.8”** as runtime and attach the previously created Role. Then, follow the following steps for setting up Lambda:

+ Select bucket name where you want to tigger
+ Select All object create events

Using your choice of IDE, create a filename “function.py” and write the codes shown in the screenshots:

```
import json
import boto3

#delete multiple objects
import os
#glob method returns list of files or folders that matches the path specified in the pathname argument
import glob

s3_resource = boto3.client("s3")

def lambda_handler(event, context):
    # TODO implement
    
    #finds all the objects from the bucket 
    objects = s3_resource.list_objects(Bucket="expemptylambda-function")["Contents"]
    len(objects)
    
    
    #delete all the objects form the bucket
    for objects in objects:
        s3_resource.delete_object(Bucket='expemptylambda-function', Key = objects["Key"])
```

Now, we will set up an Amazon EventBridge which will help us to schedule a cron job. When the scheduled datetime is occurred, it will trigger the lambda and lambda function will remove the data from s3. Follow the following steps to create a EventBridge from AWS console:

+ Provide a name for EventBridge
+ Select **“Recurring schedule”** in Occurrence
+ Select **“Corn-based schedule”** in Schedule type
+ In **“Cron expression”**, fill the boxes as shown in the below screenshot. In here, **“3#1”** for Day of the week represents 3rd day of week i.e Tuesday and 1st week of any month. To know more about cron expressions, checkout [this documentation](https://docs.oracle.com/cd/E12058_01/doc/doc.1014/e12030/cron_expressions.htm).

![schedule](/images/blog-images/automatic-emptying-s3-bucket/schedule.png)

+ Click **Next** and then choose **“AWS Lambda”** for **“target”**.
+ Choose your **“Lambda function”** that was previously created.

![invoke](/images/blog-images/automatic-emptying-s3-bucket/invoke.png)

+ Select **“Create Role”**
+ Again Click Next and then Click on Create Schedule. Now the EventBridge rule is successfully created.

![eventbridge](/images/blog-images/automatic-emptying-s3-bucket/eventbridge.png)

The setup is complete and now you can check cloudwatch logs as well your s3 bucket to verify whether its working or not. For this, you can change the trigger schedule in **“Cron expression”**.