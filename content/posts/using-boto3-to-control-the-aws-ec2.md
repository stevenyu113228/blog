---
title: Using boto3 to control the AWS EC2
date: 2022-03-23T14:39:17+08:00
draft: false
url: "/2022/03/23/using-boto3-to-control-the-aws-ec2/"
categories:
  - Cloud
showToc: true
TocOpen: false
---

## Install awscli and boto3

```bash
sudo apt install awscli
pip3 install boto3
```

## Add a new AWS user

Go to https://console.aws.amazon.com/iam/home  
Add User, pick a username, and set the AWS credential type to `Access key`

In the "Set permissions" section, select the "Attach existing policies directly" and check the `AmazonEC2FullAccess`

And it will receive an `Access Key ID` and a `Secret Access key`

## Configure the aws-cli

Run the command

```bash
aws configure
```

Input the `AWS Access Key ID` and `AWS Secret Access key`, other leave blanks.

And now, you can use the boto3 to control the AWS now.

## Example

### Create Instances

```python
import boto3

client = boto3.client('ec2','ap-northeast-3')

ami_id = 'ami-096c4b6e0792d8c16' # Ubuntu Server 20.04 On ap-northeast-3
instance_type = 't2.micro' # https://aws.amazon.com/tw/ec2/instance-types/
response  = client.run_instances(ImageId=ami_id, 
                    MinCount=1, 
                    MaxCount=1,
                    InstanceType=instance_type,
                    ) 

print(response)
```

### Stop Instances

```python
import boto3

client = boto3.client('ec2','ap-northeast-3')

instance_id = "i-0c1cfc6b6eb28e993"
response = client.stop_instances(
    InstanceIds = [instance_id],
    Force = True
)

print(response)
```

### Terminate Instances

```python
import boto3

client = boto3.client('ec2','ap-northeast-3')

instance_id = "i-0c1cfc6b6eb28e993"
response = client.terminate_instances(
    InstanceIds=[
        instance_id,
    ]
)

print(response)
```

## Reference

[https://boto3.amazonaws.com/v1/documentation/api/latest/guide/quickstart.html](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/quickstart.html)

[https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/ec2.html](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/ec2.html)
