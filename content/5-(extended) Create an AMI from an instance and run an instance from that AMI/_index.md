---
title : "Appendix"
date : "`r Sys.Date()`"
weight : 5
chapter : false
pre : " <b> 5. </b> "
---
## 5.1. Create AMI

Reference: https://000004.awsstudygroup.com/5-amazonec2basic/5.3-createcustomami/

### Step 1: Select Instance. Action box, Select Image and template → Create Image

![Untitled](/images/part5/5.1-step1.png)

### Step 2: Name and select Create Image

![Untitled](/images/part5/5.1-step2-1.png)
Wait a few minutes until the status is OK:

![Untitled](/images/part5/5.1-step2-2.png)

## 5.2. Create instance from AMI

### Step 1: In the left AMIs tag, select the AMI just created, select “Launch instance from AMI

![Untitled](/images/part5/5.2-step1.png)

### Step 2: Name the instance created from AMI as `VSCodeFromAMI`

![Untitled](/images/part5/5.2-step2.png)

### Step 3: Keep the instance type the same (if you want it faster, leave the type t3.medium). In the key pair section, select `Proceed without a key pair` because we connect using SSM.

![Untitled](/images/part5/5.2-step3.png)

### Step 4: Select a VPC with a private subnet. In the subnet section, select private subnet.

### Step 5: Select the security group created in the previous section, `VSC-SG`, or create a new one: no inbound rule; outbound rule open for Internet on all protocols.

![Untitled](/images/part5/5.2-step5.png)

### Step 6: Assign the necessary IAM profile to SSM

![Untitled](/images/part5/5.2-step6.png)

### Step 7: Select Create instance

After a few minutes, the instance will be initialized

![Untitled](/images/part5/5.2-step7.png)

### Step 8: Create a session to connect the newly created instance from ami via port forwarding:
```jsx
aws ssm start-session ^
--target instance-id ^
--document-name AWS-StartPortForwardingSessionToRemoteHost ^
--parameters host="private-ip",portNumber="8080",localPortNumber="8080" ``` ![Untitled](/images/part5/5.2-step8.png)