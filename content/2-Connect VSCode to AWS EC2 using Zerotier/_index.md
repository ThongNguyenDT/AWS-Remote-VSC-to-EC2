---
title : "Connect VSCode to AWS EC2 using Zerotier"
date :  "`r Sys.Date()`" 
weight : 2 
chapter : false
pre : " <b> 2. </b> "
---
I currently have an EC2 **Instance** running in a private subnet

![Untitled](/images/part2/2-0-1.png)

This **Instance** connects to the internet via nat gateway

![Untitled](/images/part2/2-0-2.png)

To set up the necessary software for the **Instance**, I use **SSM**

## 2.1. SSM connection

### Step 1: Create role

- Search on the search box **IAM**
- Select the **IAM** service

- Select the **Role** tab

- Select **Create Role**

![Untitled](/images/part2/2-2.1-step1-1.png)

- Select **AWS Service**
- Select **EC2**
- Select **EC2 Role for AWS Systems Manager**
- Next

![Untitled](/images/part2/2-2.1-step1-2.png)

- **Next**

![Untitled](/images/part2/2-2.1-step1-3.png)

- Name

- **Create Role**

![Untitled](/images/part2/2-2.1-step1-4.png)

### Step 2: Attack role

- Return to the **Instances** tab

- Select **Instance**
- Select **Action → Security → Modify IAM role**

![Untitled](/images/part2/2-2.1-step2-1.png)

- Select role
- Select **Update IAM Role**

![Untitled](/images/part2/2-2.1-step2-2.png)
## Step 3 Create SSM endpoint

This step is only needed when you want to connect **Instance** located in **private subnet**

- Search for ***VPC*** service
- Select **VPC**
- Select **Endpoints** tab
- Select **Create endpoints**

![Untitled](/images/part2/2-2.1-step3-1.png)
- Name
- search ***ssm***
- select by image

![Untitled](/images/part2/2-2.1-step3-2.png)

- Select ssm
- Select VPC
- Select subnet
- Select the Security group you use for the instance

![Untitled](/images/part2/2-2.1-step3-3.png)

- Select **Create**

Similarly, we need to create 3 endpoints including:

- com.amazonaws.us-east-1.ssm
- com.amazonaws.us-east-1.ssmmessages
- com.amazonaws.us-east-1.ec2messages

### Step 4 test connection

- In your Instance, select **Connect**

![Untitled](/images/part2/2-2.1-step4-1.png)

- select the **Session Manager** tab
- Select **Connect** ![Untitled](/images/part2/2-2.1-step4-2.png)
## 2.2. Zerotier setup

### Step 1: create zerotier account

- Go to website

[ZeroTier | Global Networking Solution for IoT, SD-WAN, and VPN](https://www.zerotier.com/)

https://www.zerotier.com/

- Create an account if you don't have one or log in if you already have one

### Step 2: Create a virtual network

- select as shown

![Untitled](/images/part2/2-2.2-step2-1.png)

you will see a virtual network ms appear

![Untitled](/images/part2/2-2.2-step2-2.png)

- select this network and rename it
- remember the net ID for the next step

![Untitled](/images/part2/2-2.2-step2-3.png)

### Step 3 install zerotier on your instance

- Paste the following command into the instant manage connect session to download

```bash
curl -s [https://install.zerotier.com](https://install.zerotier.com/) | sudo bash
```

![Untitled](/images/part2/2-2.2-step2-4.png)

after downloading, connect to the private virtual network

- Paste the following command into the Session manage connect of instant

```bash
sudo zerotier-cli join YOUR_NETWORD_ID
```

Return to the website

![Untitled](/images/part2/2-2.2-step2-5.png)

you see an ip ms asking to join your network

- Accept it

![Untitled](/images/part2/2-2.2-step2-6.png)

### Step 4 install zerotier on your device

on the page

[Download - ZeroTier](https://www.zerotier.com/download/)

download and install the version suitable for your operating system

### For example, after the installation is complete on your window

- Join the network as shown
- enter your network id
- and proceed to accept as in Step 3 above

## 2.3. SSH connection

proceed similarly to the steps to connect with the public ip address in lesson 1

you just need to replace the public ip address with the private ip provided in step 3.

![Untitled](/images/part2/2-2.3-1.png)

after waiting a few seconds

![Untitled](/images/part2/2-2.3-2.png)

<aside>
❗ after downloading the necessary packages, you should delete ***nat gateway*** to reduce costs for your system

</aside>
