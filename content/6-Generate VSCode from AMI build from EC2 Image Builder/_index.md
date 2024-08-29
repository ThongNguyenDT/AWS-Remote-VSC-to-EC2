---
title : "6-Generate VSCode from AMI build from EC2 Image Builder"
date : "`r Sys.Date()`"
weight : 6
chapter : false
pre : " <b> 6. </b> "
---
C2 Image Builder automates the creation, management, and deployment of customized, secure, and up-to-date “golden” ami that is pre-installed and pre-configured with software and settings.

In this section, we will build an AMI similar to part 5, but using EC2 Image Builder to create an image pipeline from infrastructure configuration (OS, VPC, subnet, security group), IAM instance profile, to installing the ami, deploying to an instance and testing the state of that instance, and scanning for security vulnerabilities in the ami.

Reference: [AWS for Microsoft Workloads Immersion Day (workshops.aws)](https://catalog.us-east-1.prod.workshops.aws/workshops/d6c7ecdc-c75f-4ad1-910f-fdd994cc4aed/en-US)

## 6.1. Terminology

### 6.2.1. AMI

The basic unit of deployment in Amazon EC2. An AMI is a preconfigured VM image that contains the operating system and pre-installed software for deploying EC2 instances.

### 6.2.2. Image Pipeline

Automated configuration for building secure operating system images on AWS. The Image Builder pipeline is associated with an **image recipe** that defines the build, validation, and test phases for the image build lifecycle. The image pipeline can be associated with an infrastructure configuration that defines where the image is built. You can define properties such as *version**,** subnet, security groups*, *logging*, and other infrastructure-related configurations.

### 6.2.3. Image Recipe

An image recipe is a document that defines the *source image* and the components that will be applied to the *source image* to create the desired configuration for the output image. You can use an image recipe to replicate builds. Image Builder recipes can be shared, forked, and edited using the console, AWS CLI, or API.

### 6.2.4. Source Image

The selected image and operating system used in your *image recipe document* along with the components. The source image and component definitions combine to create the desired configuration for the *output image.*

### 6.2.5. Build Components

Orchestration documents define the sequence of steps to download, install, and configure software packages. They also define authentication and security steps. A component is defined using a YAML document format.

## 6.2. Deployment ### 6.2.1. Create Component ### Step 1: Access [Components console](https://us-east-1.console.aws.amazon.com/imagebuilder/home?region=us-east-1#/components). Click Create component.

![Untitled](/images/part6/6.2.1-step1.png) ### Step 2: Select Type as `Build`.

![Untitled](/images/part6/6.2.1-step2.png)

### Step 3: Select values ​​from the following list

- OS: Linux
- Compatible OS Versions: Amazon Linux 2023
- Component name: VSC- Component
- Component version: 1.0.0

![Untitled](/images/part6/6.2.1-step3.png)

### Step 4: In the Definition document section, Content section, copy the following content:

```jsx
name: HelloWorldTestingDocument
description: This is hello world testing document.
schemaVersion: 1.0

phases:
  - name: build
    steps:
      - name: HostingVSCode
        action: ExecuteBash
        inputs:
          commands:
            - |
              #!/bin/bash
              sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
              sudo sh -c 'echo -e "[code]
              name=Visual Studio Code
              baseurl=https://packages.microsoft.com/yumrepos/vscode
              enabled=1
              gpgcheck=1
              gpgkey=https://packages.microsoft.com/keys/microsoft.asc" > /etc/yum.repos.d/vscode.repo'
            
              # https://github.com/amazonlinux/amazon-linux-2023/issues/397
              sleep 10
              
              sudo yum install -y code git
              sudo tee /etc/systemd/system/code-server.service <<EOF
              [Unit]
              Description=Start code server
            
              [Service]
              ExecStart=/usr/bin/code serve-web --port 8080 --host 0.0.0.0 --without-connection-token
              Restart=always
              Type=simple
              User=ec2-user
              
              [Install]
              WantedBy = multi-user.target
              EOF
            
              sudo systemctl daemon-reload
              sudo systemctl enable --now code-server
            
              # Install Node.js
              sudo -u ec2-user -i <<EOF
              curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
              source .bashrc
              nvm install 20.11.0
              nvm use 20.11.0
              EOF
  - name: validate
    steps:
      - name: VSCodeValidate
        action: ExecuteBash
        inputs:
          commands:
            - |
              #!/bin/bash
              if ! command -v code &> /dev/null
              then
                echo "code could not be found"
                exit 1
              fi

  - name: validate
    steps:
      - name: NodeJSValidate
        action: ExecuteBash
        inputs:
          commands:
            - |
              #!/bin/bash
              if ! command -v node &> /dev/null
              then
                echo "Node could not be found"
                exit 1
              fi

  - name: test
    steps:
      - name: TestWhetherAppDeploy
        action: ExecuteBash
        inputs:
          commands:
            - |
              #!/bin/bash
              if ! sudo lsof -i -n -P | grep 8080 | grep -w code &> /dev/null
              then
                echo "App could not be found"
                exit 1
              fi
```
This document consists of 3 parts:

- Build: Based on the user-data of the previous part. Install VS Code as a web app listening on port 8080.

- Validation: Check if NodeJS and VS Code have been successfully installed

- Test: Check if the web app exists and listens on port 8080

### Step 5: Select “Create Component”.

![Untitled](/images/part6/6.2.1-step5-1.png)

Result:

![Untitled](/images/part6/6.2.1-step5-2.png)

### Step 6 (Sub): Create a new version by selecting Action → Create new version.

![Untitled](/images/part6/6.2.1-step6.png)

Just change the content of the content section and edit the value of the Component version section and select Create component

### 6.2.2. Create a role for EC2 Image Builder

### Step 1: Select `AWS service`, use case `EC2`

![Untitled](/images/part6/6.2.2-step1.png)

### Step 2: Add policies as shown below

![Untitled](/images/part6/6.2.2-step2.png)

### Step 3: Name the Role `EC2ImageBuilderInfraConfigRole` and describe

![Untitled](/images/part6/6.2.2-step3.png)

### 6.2.3. Create pipeline

### Step 1: Go to [pipeline console](https://us-east-1.console.aws.amazon.com/imagebuilder/home?region=us-east-1#/createPipeline).

### Step 2: Name the pipeline `VSC-Pipeline` . Description optional.

![Untitled](/images/part6/6.2.3-step2.png)

You can also choose the option `Enable EC2 image scanning` to ensure ami security.

### Step 3: In the Build schedule section, select `Manual` and then select Next.

![Untitled](/images/part6/6.2.3-step3.png)

### Step 4: In the Recipe section, select `Create new recipe` . Select `AMI` in Image type

![Untitled](/images/part6/6.2.3-step4.png)

### Step 5: General, name the ami and version

### Step 6: Base image, select `Select managed images` and `Amazon Linux`

![Untitled](/images/part6/6.2.3-step6.png)

### Step 7: At Image origin, select `Quick start` . At Image name, select `Amazon Linux 2023 x86` . Select compatible version, currently select `Use latest available OS version`

![Untitled](/images/part6/6.2.3-step7.png)

### Step 8: In Working directory, type `/home/ec2-user` because it will download nvm into the home directory of the executing user, which is ec2-user.

![Untitled](/images/part6/6.2.3-step8.png)

### Step 9: In `Filter owner` in Components section, select `Owned by me`. Select the created component.

![Untitled](/images/part6/6.2.3-step9-1.png)

In the image is version 1.0.1, because the step to create a new version was added in the previous section (the component content remains unchanged).

In Test components - Amazon Linux section, for demo purposes, select `reboot-test-linux` .

![Untitled](/images/part6/6.2.3-step9-1.png)

### Step 10: Leave the Storage section as default and then Next.

![Untitled](/images/part6/6.2.3-step10.png)

### Step 11: In the Define image creation process section, select Next.

### Step 12: In the Infrastructure configuration section, select `Create a new infrastructure configuration`. Name it `VSC-InfraConfig`. Select the IAM role you created.

![Untitled](/images/part6/6.2.3-step12.png)

### Step 13: Expand the VPC, subnet and security groups section. Select instance type as `t3.medium`, VPC as `VSC` (created in Part 4), select 1 of 2 `private subnets`, select security group with `VSC` (security without inbound rule of previous part) to be able to create and evaluate instance. Select Next.

![Untitled](/images/part6/6.2.3-step13-1.png)

Recall about security group outbound rules:

![Untitled](/images/part6/6.2.3-step13-2.png)

### Step 14: Leave distribution settings as default and then Next.

![Untitled](/images/part6/6.2.3-step14.png)

### Step 15: After reviewing, select Create pipeline.

## 6.3. Create instance from AMI built from pipeline

### 6.3.1. Create AMI from pipeline

### Step 1: Select the newly created pipeline. In the Action box, select Run pipeline:

![Untitled](/images/part6/6.3.1.-step1-1.png)

A green notification appears.

![Untitled](/images/part6/6.3.1.-step1-2.png)

### Step 2: Click `View details` to monitor the details:

![Untitled](/images/part6/6.3.1.-step2.png)

See Image status is Testing.

### Step 3: Select the log stream link to monitor the pipeline's CloudWatch log stream

Test phase, step `TestWhetherAppDeploy`:

![Untitled](/images/part6/6.3.1.-step3-1.png)

RebootTest phase:

![Untitled](/images/part6/6.3.1.-step3-2.png)

If there is an error, exit code 1, if there is no error, exit code 0. Finally, the test instance has no error.

### Step 4: Go back to the running pipeline. When the ami build is finished, the image status changes to Available. Click on the created image version (the image below is 1.0.4/1):

![Untitled](/images/part6/6.3.1.-step4.png)

### Step 5: Observe the image created from EC2 Image Builder. Click on the AMI link:

![Untitled](/images/part6/6.3.1.-step5.png)

### Step 6: Observe the ami from the EC2 page

![Untitled](/images/part6/6.3.1.-6.png)

The AMI has been successfully created, and has passed the tests during the pipeline process without any errors.

### 6.3.2. Create an instance from the newly created AMI

### Step 1: Go to the AMI page of the EC2 console. Select the newly created ami. Click `Launch instance from AMI`

![Untitled](/images/part6/6.3.2.-step1.png)

### Step 2: Select the instance type. To save, choose `t2.micro` .

### Step 3: No key pair required.

![Untitled](/images/part6/6.3.2.-step3.png)

### Step 4: Select VPC and security group with `VSC` , private subnet, make sure there is no public IP

![Untitled](/images/part6/6.3.2.-step4.png)

### Step 5: Provide IAM instance profile with `VSC` to install SSM agent

![Untitled](/images/part6/6.3.2.-step5.png)

### Step 6: Select `Create instance`

### Step 7: Wait until instance status `Available`

![Untitled](/images/part6/6.3.2.-step7.png)

### Step 8: Access VSCode web app using Session Manager via port forwarding ```jsx aws ssm start-session ^ --target i-06ae65824244ff79b ^ --document-name AWS-StartPortForwardingSessionToRemoteHost ^ --parameters host="10.0.132.106",portNumber="8080",localPortNumber="8080" ``` ![Untitled](/images/part6/6.3 .2.-step8.png) Access successful!
