---
title : "Tạo AMI từ instance và chạy instance từ AMI đó"
date :  "`r Sys.Date()`" 
weight : 5 
chapter : false
pre : " <b> 5. </b> "
---
## 5.1. Tạo AMI

Tham khảo: https://000004.awsstudygroup.com/5-amazonec2basic/5.3-createcustomami/

### Bước 1: Chọn Instance. Ô Action, Chọn Image and template → Create Image

![Untitled](/images/part5/5.1-step1.png)

### Bước 2: Đặt tên rồi chọn Create Image

![Untitled](/images/part5/5.1-step2-1.png)
Chờ vài phút tới khi tình trạng OK:

![Untitled](/images/part5/5.1-step2-2.png)

## 5.2. Tạo instance từ AMI

### Bước 1: Ở tag trái AMIs, chọn AMI vừa tạo, chọn “Launch instance from AMI

![Untitled](/images/part5/5.2-step1.png)

### Bước 2: Đặt tên cho instance tạo từ AMI là `VSCodeFromAMI`

![Untitled](/images/part5/5.2-step2.png)


### Bước 3: Giữ nguyên instance type (muốn nhanh hơn thì để type t3.medium). Mục key pair chọn `Proceed without a key pair` vì ta kết nối bằng SSM.

![Untitled](/images/part5/5.2-step3.png)

### Bước 4: Chọn VPC có private subnet. Phần subnet chọn private subnet.

### Bước 5: Chọn security group tạo ở mục trước, `VSC-SG`, hoặc tạo mới: không có inbound rule; outbound rule mở cho Internet trên mọi giao thức.

![Untitled](/images/part5/5.2-step5.png)

### Bước 6: Gán IAM profile cần thiết cho SSM

![Untitled](/images/part5/5.2-step6.png)

### Bước 7: Chọn Create instance

Sau vài phút, instance sẽ khởi tạo xong

![Untitled](/images/part5/5.2-step7.png)

### Bước 8: Tạo session kết nối instance vừa tạo từ ami qua port forwarding:
```jsx
aws ssm start-session ^
--target instance-id ^
--document-name AWS-StartPortForwardingSessionToRemoteHost ^
--parameters host="private-ip",portNumber="8080",localPortNumber="8080"
```

![Untitled](/images/part5/5.2-step8.png)