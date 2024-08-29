---
title : "REMOTE DEVELOPMENT WITH VS VSCODE AND EC2"
date :  "`r Sys.Date()`" 
# weight : 5
chapter : false
---
# REMOTE DEVELOPMENT WITH VS VSCODE AND EC2
![image](/images/NEN-REMOTE-VSCODE-TO-EC2.png)
![image](/images/aws-remote/image.png)
![image](/images/aws-remote/remote-achirtected.png)

Connecting your software development environment to a cloud server increases the scalability and flexibility of your application, allowing developers to easily scale resources as needed.

In addition, using a cloud server improves collaboration, allowing team members to work remotely efficiently. And most of all, the Cloud provides automation tools and services that help optimize the application development and deployment process.

This article will introduce solutions to **connect your Virtual Studio Code software development environment to an EC2 server**.

### **TABLE OF CONTENTS**
[1. Connect VSCode to AWS EC2 using SSH](/1-connect-vscode-to-aws-ec2-using-ssh/index.html)

[1.1. Introduction to SSH connection](/1-connect-vscode-to-aws-ec2-using-ssh/index.html)

[1.2. Purpose of using SSH](/1-connect-vscode-to-aws-ec2-using-ssh/index.html)

[1.3. Introducing Remote-SSH Extension on VSCode](/1-connect-vscode-to-aws-ec2-using-ssh/index.html)

[1.4. Instructions for initializing Ubuntu EC2](/1-connect-vscode-to-aws-ec2-using-ssh/index.html)

[1.5. Instructions for configuring Command Prompt connection with EC2](/1-connect-vscode-to-aws-ec2-using-ssh/index.html)

[1.6. Instructions for configuring VSCode connection to EC2](/1-connect-vscode-to-aws-ec2-using-ssh/index.html)

[2. Connect VSCode to AWS EC2 using Zerotier](/2-connect-vscode-to-aws-ec2-using-zerotier/index.html)

[2 .1. SSM connection](/2-connect-vscode-to-aws-ec2-using-zerotier/index.html)

[2.2. Zerotier setup](/2-connect-vscode-to-aws-ec2-using-zerotier/index.html)

[2.3. SSH connection](/2-connect-vscode-to-aws-ec2-using-zerotier/index.html)

[3. Deploy VS-Code on EC2 instance from Amazon Linux 2 using CDK](/3-hosting-vs-code-on-ec2-from-amazon-linux-2/index.html)
   
[4. Manually deploy VS-Code on EC2 instance from Amazon Linux 2](/4-manually-deploy-vs-code-on-ec2/index.html)
   
[5. Create AMI from VSCode deployment instance and create EC2 instance from AMI that](/5-extended-create-an-ami-from-an-instance-and-run-an-instance-from-that-ami/index.html)
   
[6. Create AMI from EC2 Image Builder and create EC2 instance hosting VS Code web app from that AMI](/5-extended-create-an-ami-from-an-instance-and-run-an-instance-from-that-ami/index.html)

[7. Connect VScode using EC2_Instances_Connect and AWS Systems Manager](/7-connect-vscode-using-ec2_instances_connect-and-aws-systems-manager/index.html)