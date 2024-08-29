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

Kết nối môi trường phát triển phần mềm với máy chủ cloud giúp tăng cường khả năng mở rộng và linh hoạt của ứng dụng, cho phép các nhà phát triển dễ dàng điều chỉnh tài nguyên theo nhu cầu. 

Ngoài ra, việc sử dụng máy chủ cloud cải thiện khả năng cộng tác, giúp các thành viên trong nhóm có thể làm việc từ xa một cách hiệu quả. Và hơn hết, Cloud cung cấp các công cụ và dịch vụ tự động hóa, giúp tối ưu hóa quy trình phát triển và triển khai ứng dụng.

Bài viết này sẽ giới thiệu các giải pháp để **kết nối môi trường phát triển phần mềm Virtual Studio Code với máy chủ EC2**.

### **MỤC LỤC**
[1. Kết nối VSCode với AWS EC2 bằng SSH](/1-connect-vscode-to-aws-ec2-using-ssh/index.html)
    
[1.1. Giới thiệu kết nối SSH](/1-connect-vscode-to-aws-ec2-using-ssh/index.html)
    
[1.2. Mục đích sử dụng SSH](/1-connect-vscode-to-aws-ec2-using-ssh/index.html)
    
[1.3. Giới thiệu Remote-SSH Extension trên VSCode](/1-connect-vscode-to-aws-ec2-using-ssh/index.html)
    
[1.4. Hướng dẫn khởi tạo Ubuntu EC2](/1-connect-vscode-to-aws-ec2-using-ssh/index.html)
    
[1.5. Hướng dẫn cấu hình kết nối Command Prompt với EC2](/1-connect-vscode-to-aws-ec2-using-ssh/index.html)
    
[1.6. Hướng dẫn cấu hình kết nối VSCode với EC2](/1-connect-vscode-to-aws-ec2-using-ssh/index.html)
    
[2. Kết nối VSCode với AWS EC2 bằng Zerotier](/2-connect-vscode-to-aws-ec2-using-zerotier/index.html)
    
[2.1. SSM connection](/2-connect-vscode-to-aws-ec2-using-zerotier/index.html)
    
[2.2. Zerotier setup](/2-connect-vscode-to-aws-ec2-using-zerotier/index.html)
    
[2.3. Kết nối SSH](/2-connect-vscode-to-aws-ec2-using-zerotier/index.html)
    
[3. Triển khai VS-Code trên EC2 instance từ Amazon Linux 2 bằng CDK](/3-hosting-vs-code-on-ec2-from-amazon-linux-2/index.html)
   
[4. Triển khai thủ công VS-Code trên EC2 instance từ Amazon Linux 2](/4-manually-deploy-vs-code-on-ec2/index.html)
   
[5. Tạo AMI từ instance triển khai VSCode và tạo EC2 instance từ AMI đó](/4-manually-deploy-vs-code-on-ec2/index.html) 
   
[6. Tạo AMI từ EC2 Image Builder và tạo EC2 instance hosting VS Code web app từ AMI đó](/6-generate-vscode-from-ami-build-from-ec2-image-builder/index.html)
   
[7. Kết nối VScode bằng EC2_Instances_Connect và AWS Systems Manager](/7-connect-vscode-using-ec2_instances_connect-and-aws-systems-manager/index.html)