---
title : "Triển khai thủ công VS Code trên EC2"
date :  "`r Sys.Date()`" 
weight : 4 
chapter : false
pre : " <b> 4. </b> "
---
(mở rộng của mục 3, không cần dùng CDK)

## 4.1. Cài đặt môi trường

### 4.1.1. VPC

Yêu cầu: 

- Private subnet để đặt EC2 instance (tối thiểu 1)
- 1 NAT Gateway để instance truy cập internet
- Tối thiểu 1 Public subnet để đặt Internet Gateway

Để tiện thì mình dùng tính năng “VPC and more” trong “Create VPC”:

![Untitled](/images/part4/4.1.1-1.png)


Resource map:

![Untitled](/images/part4/4.1.1-2.png)

Kết quả:

![Untitled](/images/part4/4.1.1-3.png)

### 4.1.2. Tạo Role và Policy cho SSM

### Bước 1: Tạo policy

Tạo Policy dạng JSON, có thể đặt tên là VSCodeBastionHostInstanceRoleDefaultPolicy

```jsx
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "*",
                "ec2messages:*",
                "ssm:UpdateInstanceInformation",
                "ssmmessages:*"
            ],
            "Resource": "*",
            "Effect": "Allow"
        }
    ]
}
```

Kết quả:

![Untitled](/images/part4/4.1.2-step1.png)

### Bước 2: Tạo role và thêm policy vừa tạo vào role

Đặt tên role là VscodeOnEc2ForPrototyping

![Untitled](/images/part4/4.1.2-step2.png)
Role được dùng cho IAM instance profile

### 4.1.3. Tạo security group cho EC2

Nếu EC2 instance của bạn không phải một ứng dụng công khai, thì theo best practice, không cần inbound rule, vì ta chỉ truy cập tới EC2 instance qua port forwarding của Session Manager.

Đặt tên security group là VSC-SG

![Untitled](/images/part4/4.1.3-step1.png)

![Untitled](/images/part4/4.1.3-step2.png)

## 4.2 Triển khai

### 4.2.1. Tạo EC2 instance

### Bước 1: Chọn AMI là Amazon Linux 2023, instance type là t3.medium

![Untitled](/images/part4/4.2.1-step1.png)

### Bước 2: Chọn VPC vừa tạo → private subnet bất kỳ

### Bước 3: Đảm bảo không có public IP

### Bước 4: Chọn security group VSC-SG

![Untitled](/images/part4/4.2.1-step4.png)

### Bước 5: Cấu hình volume 100GB

![Untitled](/images/part4/4.2.1-step6.png)

### Bước 6: Chọn IAM instance profile tên “VscodeOnEc2ForPrototyping” đã tạo từ trước

![Untitled](/images/part4/4.2.1-step6.png)

### Bước 7: Mục user-data, dán đoạn code sau:

```jsx
#!/bin/bash
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
```

Mục đích: triển khai VS-Code trên web port 8080 của EC2 instance

### Bước 8: Cuối cùng, chọn “Create instance”.

Sau vài phút, đảm bảo rằng instance đã được khởi tạo thành công

![Untitled](/images/part4/4.2.1-step8.png)

### Bước 9: Kết nối instance bằng việc chọn Connect → Session Manager

Đảm bảo SSM agent đã được cài đặt bằng cách kết nối vào instance qua Session Manager

![Untitled](/images/part4/4.2.1-step9.png)

Khi cấu hình thành công:

![Untitled](/images/part4/4.2.1-step9-1.png)

### Bước 10: Kiểm tra web app đã triển khai thành công bằng lệnh

```jsx
curl localhost:8080
```

Khi web đã triển khai, ta có thể lấy được tài nguyên của trang đang mở trên port 8080:

![Untitled](/images/part4/4.2.1-step9-2.png)

### 4.2.2. Truy cập VS Code triển khai trên EC2 từ SSM qua port forwarding

Thay <instance-ID> và <Private-ID> trong đoạn script dưới tương ứng với private IP và instance ID của EC2 instance

### Kết nối từ Linux (lưu ý phải có GUI trên máy Linux):

```jsx
aws ssm start-session \
    --target <instance-ID> \
    --document-name AWS-StartPortForwardingSessionToRemoteHost \
    --parameters '{"host":["<Private-IP>"],"portNumber":["8080"], "localPortNumber":["8080"]}'
```

![Untitled](/images/part4/4.2.2-1.png)

### Kết nối từ Windows:

```jsx
aws ssm start-session ^
    --target <instance-ID> ^
    --document-name AWS-StartPortForwardingSessionToRemoteHost ^
    --parameters host="<Private-IP>",portNumber="8080",localPortNumber="8080"
```

![Untitled](/images/part4/4.2.2-2.png)

Khi này, truy cập [localhost:8080](http://localhost:8080) trên browser:

![Untitled](/images/part4/4.2.3-3.png)

## 4.2. Kiểm tra performace:

Link youtube: https://youtu.be/6DxtEsX6gvY
