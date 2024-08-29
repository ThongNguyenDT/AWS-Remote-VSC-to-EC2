---
title : "Connect VScode using EC2_Instances_Connect and AWS Systems Manager"
date : "`r Sys.Date()`"
weight : 7
chapter : false
pre : " <b> 7. </b> "
---
In this section we will connect vscode remote ssh to EC2 in private subnet

through two different methods:

<aside>
❗ If you do not have a VPC, you can refer to the steps to create [here](https://www.notion.so/REMOTE-DEVELOPMENT-WITH-VS-VSCODE-AND-EC2-aec58040853d4f229f7e1d1f930b417f?pvs=21)

</aside>

## EC2_Instances_Connect

## AWS Systems Manager

### Step 1 Set up the necessary endpoints

To connect ssh to ec2 in private connect, we need to connect through an intermediary device: it can be a bastion host, or endpoints

Use **EC2 Instance Connect Endpoint**

Similar to the endpoint settings in the above article, we need to create endpoint as follows

![Untitled](/images/part7/7.step1-l-1.png)

Use **3 ssm endpoint connect**

Similar to the endpoint settings in the above article, we need to create endpoint as follows

![Untitled](/images/part7/7.step1-r-1.png)

![Untitled](/images/part7/7.step1-r-2.png)

![Untitled](/images/part7/7.step1-r-3.png)

Notes in the setup section:

- We need to point correctly to the VPC, subnet of EC2 that needs to be connected
- Need to open security group so that EC2 and endpoint can connect to each other

Note:

- We need to point correctly to the VPC, subnet of EC2 that needs to be connected
- Need to open security group so that EC2 and endpoint can connect to each other
- Need to create 3 endpoints including:

- com.amazonaws.us-east-1.ssm

- com.amazonaws.us-east-1.ssmmessages

- com.amazonaws.us-east-1.ec2messages

**EC2 Instance Connect** allows us to directly create VPC **endpoint** in the **connection** section of ec2.

EC2 Instance Connect does not require additional plugins

EC2 Instance Connect does not require access to connect ec2 vs management service because it is a direct connection so no management service is needed

Can only connect to 1 EC2, Set up each machine separately

Easier configuration.

You must set up these endpoints in the VPC Endpoint section

Must set up the plugin for AWS CLI on the source device

Must have SSM agent on EC2, note:

- Some default **AMI** already have agent configuration
- [You can refer to more here](https://docs.aws.amazon.com/systems-manager/latest/userguide/ami-preinstalled-agent.html)

SSM is essentially a management service, so you **need to grant permission** to allow EC2 to interact with this service. You can refer to [permission granting steps](https://www.notion.so/REMOTE-DEVELOPMENT-WITH-VS-VSCODE-AND-EC2-aec58040853d4f229f7e1d1f930b417f?pvs=21), [permission details](https://www.notion.so/REMOTE-DEVELOPMENT-WITH-VS-VSCODE-AND-EC2-aec58040853d4f229f7e1d1f930b417f?pvs=21)

Allows you to configure multiple devices at the same time, this is the content of the service [**AWS Systems Manager**](https://us-east-1.console.aws.amazon.com/systems-manager/home)

---

Both of these methods do not require:

- Setting up the ssh port for connection
- Public EC2 you go to the internet
- Install VScode user data packages

---

### Step 2: Configure EC2

<aside>
❗ If you do not have EC2, you can refer to [here](https://www.notion.so/REMOTE-DEVELOPMENT-WITH-VS-VSCODE-AND-EC2-aec58040853d4f229f7e1d1f930b417f?pvs=21)
You do not need to do step 7
To connect session manager, you must have assigned role rights to your EC2, which can be [self-created from Json](https://www.notion.so/REMOTE-DEVELOPMENT-WITH-VS-VSCODE-AND-EC2-aec58040853d4f229f7e1d1f930b417f?pvs=21) or in the way already in [step 1](https://www.notion.so/REMOTE-DEVELOPMENT-WITH-VS-VSCODE-AND-EC2-aec58040853d4f229f7e1d1f930b417f?pvs=21)

</aside>

<aside>
❗ The security groups in this article, I can leave the default security group to connect to each other, the details will have the best practice diagram later

</aside>

Create the **EC2 Instance Connect** connection directly in the **connect** tab of ec2

![Untitled](/images/part7/7.step2.png)

### Step 3: Try to connect

### EC2_Instances_Connect

![Untitled](/images/part7/7.step3-l-1.png)

If you see the above 3 warnings and cannot connect to your ec2 in the EC2 Instance Connect tab, it means that your VPC and subnet settings are currently on the right track.

Select by image

![Untitled](/images/part7/7.step3-l-2.png)

if you get a yellow connect button result your sg and endpoint settings are correct

![Untitled](/images/part7/7.step3-l-3.png)

if not read the troubleshooting steps on the side

- Select Connect

![Untitled](/images/part7/7.step3-l-4.png)
### AWS Systems Manager

![Untitled](/images/part7/7.step3-r-1.png)

In the Session Manager tab, if you see the yellow connection button as shown in the image, it means you have successfully established your connection

If not, you can check the following settings:

- Is your security group set up correctly, it is recommended that if you are not sure, you should set sg default for both your endpoint and EC2
- Check that your endpoints and subnets point to the same private subnet. Note that selecting the subnet for the endpoint in part one is setting the target for the endpoint to connect to, not the location of the endpoint.

- No need to set an endpoint for the public subnet
- Check if your AMI supports Agent ssm. If you reboot the instance, if not, you can delete it, and recreate it or proceed to connect EC2_Instances_Connect and install the agent

- Select Connect

![Untitled](/images/part7/7.step3-r-2.png)

### Step 4: connect using CLI

The next steps require that you have set up AWS CLI v2

If you don't have it, you can refer to [here](https://www.notion.so/REMOTE-DEVELOPMENT-WITH-VS-VSCODE-AND-EC2-aec58040853d4f229f7e1d1f930b417f?pvs=21)

### EC2_Instances_Connect

No other installation package required

### AWS Systems Manager

requires a plugin to connect the session on cli, if not [you can refer to here](https://www.notion.so/REMOTE-DEVELOPMENT-WITH-VS-VSCODE-AND-EC2-aec58040853d4f229f7e1d1f930b417f?pvs=21)

```powershell
aws ec2-instance-connect ssh --instance-id <instance_id>
```

![Untitled](/images/part7/7.step4-l-1.png)

```powershell
aws ssm start-session --target <instance_id>
```

![Untitled](/images/part7/7.step4-r-1.png)

### Step 5: Configure VSCode

So we have successfully connected ec2 via cli, now we need to set them up to connect via vscode

[We perform the same steps as connecting to EC2 on public subnet](https://www.notion.so/REMOTE-DEVELOPMENT-WITH-VS-VSCODE-AND-EC2-aec58040853d4f229f7e1d1f930b417f?pvs=21)

In the edit host step, we will create tunnels for ssh instead of using public ip

The job is very simple, just configure your host file as follows