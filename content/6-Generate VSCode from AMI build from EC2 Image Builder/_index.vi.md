---
title : "Tạo VSCode từ bản dựng AMI từ EC2 Image Builder"
date :  "`r Sys.Date()`" 
weight : 6 
chapter : false
pre : " <b> 6. </b> "
---
C2 Image Builder giúp tự động hóa việc tạo, quản lý và triển khai các ami “golden” được tùy chỉnh, an toàn và cập nhật được cài đặt sẵn và cấu hình sẵn bằng phần mềm và cài đặt. 

Phần này ta sẽ xây dựng một AMI cũng như phần 5, nhưng bằng cách dùng EC2 Image Builder để tạo ra một quy trình image pipeline từ khâu cấu hình hạ tầng (OS, VPC, subnet, security group), IAM instance profile, tới việc cài đặt cho ami, triển khai thành instance và test trạng thái instance đó, đồng thời scan lỗ hổng bảo mật trong ami. 

Tham khảo: [AWS for Microsoft Workloads Immersion Day (workshops.aws)](https://catalog.us-east-1.prod.workshops.aws/workshops/d6c7ecdc-c75f-4ad1-910f-fdd994cc4aed/en-US)

## 6.1. Thuật ngữ

### 6.2.1. AMI

Đơn vị triển khai cơ bản trong Amazon EC2. AMI là VM image được cấu hình sẵn có chứa hệ điều hành và phần mềm được cài đặt sẵn để triển khai các phiên bản EC2.

### 6.2.2. Image Pipeline

Cấu hình tự động để xây dựng hình ảnh hệ điều hành an toàn trên AWS. Image Builder pipeline được liên kết với một **image recipe** xác định các giai đoạn xây dựng (build), xác thực (validation) và thử nghiệm (test) cho vòng đời xây dựng image. Image pipeline có thể được liên kết với infrastructure configuration nhằm định nghĩa nơi image được xây dựng (build). Bạn có thể xác định các thuộc tính, chẳng hạn như *version**,** subnet, security groups*, *logging* và các cấu hình khác liên quan đến cơ sở hạ tầng.

### 6.2.3. Image Recipe

Image recipe là một tài liệu (document) xác định *source image* và các thành phần sẽ được áp dụng cho *source image* để tạo cấu hình mong muốn cho hình ảnh đầu ra. Bạn có thể sử dụng image recipe để sao chép các bản dựng (build). Image Builder recipe có thể được chia sẻ, phân nhánh và chỉnh sửa bằng console, AWS CLI hoặc API. 

### 6.2.4. Source Image

Image đã chọn và hệ điều hành được sử dụng trong *image recipe document* của bạn cùng với các thành phần. Ảnh nguồn và định nghĩa thành phần kết hợp tạo ra cấu hình mong muốn cho *output image.*

### 6.2.5. Build Components

Tài liệu dàn dựng (orchestration documents) xác định trình tự các bước để tải xuống, cài đặt và cấu hình các gói phần mềm. Chúng cũng xác định các bước xác thực và tăng cường bảo mật. Một thành phần được xác định bằng định dạng tài liệu YAML.

## 6.2. Triển khai

### 6.2.1. Tạo Component

### Bước 1: Truy cập [Components console](https://us-east-1.console.aws.amazon.com/imagebuilder/home?region=us-east-1#/components). Click Create component.

![Untitled](/images/part6/6.2.1-step1.png)

### Bước 2: Chọn Type là `Build`.

![Untitled](/images/part6/6.2.1-step2.png)

### Bước 3: Chọn các giá trị theo danh sách sau

- OS: Linux
- Compatible OS Versions: Amazon Linux 2023
- Component name: VSC-Component
- Component version: 1.0.0

![Untitled](/images/part6/6.2.1-step3.png)

### Bước 4: Mục Definition document, phần Content, copy nội dung sau vào:

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

Nội dung document này gồm 3 phần:

- Build: Dựa trên user-data của phần trước. Cài đặt VS Code dạng web app lắng nghe ở port 8080.
- Validation: Kiểm tra liệu đã cài thành công NodeJS và VS Code
- Test: Kiểm tra liệu web app có tồn tại và lắng nghe ở port 8080

### Bước 5: Chọn “Create Component”.

![Untitled](/images/part6/6.2.1-step5-1.png)

Kết quả:

![Untitled](/images/part6/6.2.1-step5-2.png)

### Bước 6 (Phụ): Tạo version mới bằng cách chọn Action → Create new version.

![Untitled](/images/part6/6.2.1-step6.png)

Chỉ cần đổi nội dung mục content và sửa lại giá trị mục Component version rồi chọn Create component

### 6.2.2. Tạo role cho EC2 Image Builder

### Bước 1: Chọn `AWS service` , use case `EC2`

![Untitled](/images/part6/6.2.2-step1.png)

### Bước 2: Thêm các policy như hình sau

![Untitled](/images/part6/6.2.2-step2.png)

### Bước 3: Đặt tên Role là `EC2ImageBuilderInfraConfigRole` và mô tả

![Untitled](/images/part6/6.2.2-step3.png)


### 6.2.3. Tạo pipeline

### Bước 1: Truy cập [pipeline console](https://us-east-1.console.aws.amazon.com/imagebuilder/home?region=us-east-1#/createPipeline).

### Bước 2: Đặt tên cho pipeline là `VSC-Pipeline` . Mô tả tùy ý.

![Untitled](/images/part6/6.2.3-step2.png)

Có thể chọn thêm các option `Enable EC2 image scanning` để đảm bảo ami bảo mật.

### Bước 3: Mục Build schedule, chọn `Manual` rồi chọn Next.

![Untitled](/images/part6/6.2.3-step3.png)

### Bước 4: Mục Recipe, chọn `Create new recipe` . Mục Image type chọn `AMI`

![Untitled](/images/part6/6.2.3-step4.png)

### Bước 5: Mục General, đặt tên cho ami và phiên bản

### Bước 6: Mục Base image, chọn `Select managed images` và `Amazon Linux`

![Untitled](/images/part6/6.2.3-step6.png)

### Bước 7: Tại Image origin, chọn `Quick start` . Tại Image name, chọn `Amazon Linux 2023 x86` . Chọn version tương thích, hiện tại chọn `Use latest available OS version`

![Untitled](/images/part6/6.2.3-step7.png)

### Bước 8: Tại Working directory, gõ `/home/ec2-user` vì sẽ tải nvm vào thư mục home của user thực thi là ec2-user.

![Untitled](/images/part6/6.2.3-step8.png)

### Bước 9: Tại `Filter owner` mục Components, chọn ô `Owned by me`. Chọn component đã tạo.

![Untitled](/images/part6/6.2.3-step9-1.png)

Trong hình là phiên bản 1.0.1, vì đã làm thêm bước tạo version mới ở phần trước (nội dung component không đổi).

Mục Test components - Amazon Linux, với mục đích demo, chọn `reboot-test-linux` .

![Untitled](/images/part6/6.2.3-step9-1.png)

### Bước 10: Để mặc định phần Storage rồi Next.

![Untitled](/images/part6/6.2.3-step10.png)

### Bước 11: Mục Define image creation process, chọn Next.

### Bước 12: Mục Infrastructure configuration, chọn `Create a new infrastructure configuration`. Đặt tên là `VSC-InfraConfig` . Chọn IAM role đã tạo.

![Untitled](/images/part6/6.2.3-step12.png)

### Bước 13: Mở rộng mục VPC, subnet and security groups. Chọn instance type là `t3.medium` , VPC là `VSC` (đã tạo ở Phần 4), chọn 1 trong 2 `private subnet`, chọn security group có từ `VSC` (security không inbound rule của phần trước) để có thể tạo và đánh giá instance. Chọn Next.

![Untitled](/images/part6/6.2.3-step13-1.png)

Nhắc lại về security group outbound rules:

![Untitled](/images/part6/6.2.3-step13-2.png)


### Bước 14: Mục distribution settings để default rồi Next.

![Untitled](/images/part6/6.2.3-step14.png)

### Bước 15: Sau khi review, chọn Create pipeline.

## 6.3. Tạo instance từ AMI dựng từ pipeline

### 6.3.1. Tạo AMI từ pipline

### Bước 1: Chọn pipeline vừa tạo. Ô Action, chọn Run pipeline:

![Untitled](/images/part6/6.3.1.-step1-1.png)

Thông báo màu xanh hiện ra. 

![Untitled](/images/part6/6.3.1.-step1-2.png)

### Bước 2: Bấm `View details` để theo dõi chi tiết:

![Untitled](/images/part6/6.3.1.-step2.png)

Thấy Image status là Testing. 

### Bước 3: Chọn link log stream để theo dõi CloudWatch log stream của pipeline

Phase test, step `TestWhetherAppDeploy`:

![Untitled](/images/part6/6.3.1.-step3-1.png)

Phase `RebootTest` :

![Untitled](/images/part6/6.3.1.-step3-2.png)

Nếu lỗi thì exit code 1, không lỗi thì exit code 0. Sau cùng, instance test không lỗi.

### Bước 4: Quay lại pipeline đang chạy. Khi build ami xong, image status đổi thành Available. Click vào phiên bản image đã tạo (hình dưới là 1.0.4/1):

![Untitled](/images/part6/6.3.1.-step4.png)

### Bước 5: Quan sát image đã tạo từ EC2 Image Builder. Bấm vào link AMI:

![Untitled](/images/part6/6.3.1.-step5.png)

### Bước 6: Quan sát ami từ trang EC2

![Untitled](/images/part6/6.3.1.-6.png)


AMI đã tạo thành công, và đã trải qua các test trong quá trình pipeline mà không phát sinh lỗi. 

### 6.3.2. Tạo instance từ AMI vừa tạo

### Bước 1: Vào trang AMI của EC2 console. Chọn ami vừa tạo. Click `Launch instance from AMI`

![Untitled](/images/part6/6.3.2.-step1.png)


### Bước 2: Chọn instance type. Để tiết kiệm thì chọn `t2.micro` .

### Bước 3: Không cần key pair.

![Untitled](/images/part6/6.3.2.-step3.png)

### Bước 4: Chọn VPC và security group có từ `VSC` , private subnet, đảm bảo không có public IP

![Untitled](/images/part6/6.3.2.-step4.png)

### Bước 5: Cung cấp IAM instance profile có từ `VSC` để cài đặt SSM agent

![Untitled](/images/part6/6.3.2.-step5.png)

### Bước 6: Chọn `Create instance`

### Bước 7: Đợi tới khi instance status `Available`

![Untitled](/images/part6/6.3.2.-step7.png)

### Bước 8: Truy cập web app VSCode bằng Session Manager qua port forwarding

```jsx
aws ssm start-session ^
--target i-06ae65824244ff79b ^
--document-name AWS-StartPortForwardingSessionToRemoteHost ^
--parameters host="10.0.132.106",portNumber="8080",localPortNumber="8080"
```

![Untitled](/images/part6/6.3.2.-step8.png)

Truy cập thành công!