---
title : "Hosting VS Code on EC2 from Amazon Linux 2"
date : "`r Sys.Date()`"
weight : 3
chapter : false
pre : " <b> 3. </b> "
---
In this part, we will deploy a web application for VS Code on EC2 from an Amazon Linux 2 virtual machine (from now on called AL2), then connect to that website securely via port forwarding of AWS Session Manager System Manager. References:

1. https://github.com/aws-samples/vscode-on-ec2-for-prototyping

2. https://github.com/peteragility/ssm-port-forward

![Untitled](/images/part3/3-start.png)

## 3.0. Configure SSH on the AL2 virtual machine

For convenience in typing commands, I configure SSH for the AL2 virtual machine to access from the real computer Terminal.

### Step 1: Open the file /etc/ssh/sshd_config

`sudo nano /etc/ssh/sshd_config`

### Step 2: Make sure there is a line “PasswordAuthentication yes”

![Untitled](/images/part3/3-3.0-step2.png)

### Step 3: Reset sshd

`sudo systemctl restart sshd`

## 3.1. Install AWS CLI

Reference: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

### Linux:

```jsx
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

### Windows:

Download and install AWS CLI from [`https://awscli.amazonaws.com/AWSCLIV2.msi`](https://awscli.amazonaws.com/AWSCLIV2.msi)

### Check version

```jsx
aws --version
```

## 3.2. Install NodeJS

- see [Tutorial: Setting Up Node.js on an Amazon EC2 Instance - AWS SDK for JavaScript](https://docs.aws.amazon.com/sdk-for-javascript/v2/developer-guide/setting-up-node-on-ec2-instance.html)

### Step 1: Install nvm

![Untitled](/images/part3/3-3.2-step1.png)

### Step 2: Install NodeJS LTS

![Untitled](/images/part3/3-3.2-step2.png)

The installed NodeJS version is 20.16.0.

### Step 3: Determine the version by command `node -v` then the library is not found

![Untitled](/images/part3/3-3.2-step3.png)

To fix this problem, I refer to the link [node.js - GLIBC_2.27 not found while installing Node on Amazon EC2 instance - Stack Overflow](https://stackoverflow.com/questions/72022527/glibc-2-27-not-found-while-installing-node-on-amazon-ec2-instance), there are 2 solutions:

1. Downgrade the version of NodeJS (currently 20.16.0) to 16.0.0.

```jsx
nvm install 16.0.0
```

1. Install the GLBIC_2.27 library version or higher

The stack overflow link above will lead to the link [glibc 2.27+ on Amazon Linux 2 | AWS re:Post (repost.aws)](https://repost.aws/questions/QUrXOioL46RcCnFGyELJWKLw/glibc-2-27-on-amazon-linux-2). Accordingly, we know that upgrading the version is not possible, and downgrading NodeJS will be at risk of missing supporting libraries whenever the web app deployed behind is upgraded.

## 3.3. Install session-manager plugin reference [Install the Session Manager plugin on Amazon Linux 2 and Red Hat Enterprise Linux distributions - AWS Systems Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/install-plugin-linux.html)) ### Step 1: Install session-manager-plugin ```jsx sudo yum install -y https://s3.amazonaws.com/session-manager-downloads/plug in/latest/linux_64bit/session-manager-plugin.rpm ``` ![Untitled](/images/part3/3-3.3-step1.png) ### Step 2: Install project ```jsx npm ci ``` ![Untitled](/images/part3/3-3.3-step2.png)
### Step 3: If you have never used CDK before, the Bootstrap process is only necessary for the first time. The following command is not necessary if you have already bootstrapped.

```jsx
npx cdk bootstrap
```

![Untitled](/images/part3/3-3.3-step3.png)

### Step 4: Deploy to AWS:

```jsx
npx cdk deploy
```

![Untitled](/images/part3/3-3.3-step4-1.png)

If asked “Do you wish to deploy these changes?” Enter y ![Untitled](/images/part3/3-3.3-step4-2.png) Wait a few minutes for installation: ![Untitled](/images/part3/3-3.3-step4-3.png) ### (secondary): Proceed to observe ![Untitled](/images/part3/3-3.3-step4-4.png) ### stack VscodeOnEc2ForPrototyping Stack Tag Output: ![Untitled](/images/part3/3-3.3-step4-5.png) Tag Resources: ![Untitled](/images/part3/3-3.3-step4-6.png) ![Untitled](/images/part3/3-3.3-step4-7.png) ### CDKToolkit stack ### Tag Outputs ![Untitled](/images/part3/3-3.3-step4-8.png) ### Tag Resources ![Untitled](/images/part3/3-3.3-step4-9.png) Check instance ![Untitled](/images/part3/3-3.3-step4-10.png) ![Untitled](/images/part3/3- 3.3-step4-10-1.png) ![Untitled](/images/part3/3-3.3-step4-11.png) ### Step 5: Connecting but error ![Untitled](/images/part3/3-3.3-step5-1.png) Cannot connect web app because it only opens at localhost. But if you create a session to connect EC2 using Windows (refer to https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-sessions-start.html#sessions-remote-port-forwarding):

![Untitled](/images/part3/3-3.3-step5-2.png)

Tada:

![Untitled](/images/part3/3-3.3-step5-3.png)

## 3.4. Demo network performance:

https://www.youtube.com/watch?v=Pe1Whz-Q0TQ

## 3.5. Cleanup:

![Untitled](/images/part3/3-3.5.png)
Wait a few minutes:

![Untitled](/images/part3/3-3.6.png)
Go to S3 Console to delete the created S3 bucket (name starts with cdk-hnb)