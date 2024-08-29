---
title : "kết nối VSCode với AWS EC2 bằng Zerotier"
date :  "`r Sys.Date()`" 
weight : 2 
chapter : false
pre : " <b> 2. </b> "
---
Hiện mình đang có một EC2 **Instance** chạy trong private subnet

![Untitled](/images/part2/2-0-1.png)

**Instance** này kết nối internet thông qua nat gateway

![Untitled](/images/part2/2-0-2.png)

Để tiến hành setup các phần mềm cần thiết cho **Instance** mình dùng **SSM**

## 2.1. SSM connection

### Bước 1: Tạo role

- Search trên khung tìm kiếm **IAM**
- Chọn dịch vụ **IAM**
- Chọn tab **Role**
- Chọn **Create Role**

![Untitled](/images/part2/2-2.1-step1-1.png)


- Chọn **AWS Service**
- Chọn **EC2**
- Chọn **EC2 Role for AWS Systems Manager**
- Next

![Untitled](/images/part2/2-2.1-step1-2.png)

- **Next**

![Untitled](/images/part2/2-2.1-step1-3.png)


- Đặt tên
- **Create Role**

![Untitled](/images/part2/2-2.1-step1-4.png)

### Bước 2: Attack role

- Trở về tab **Instances**
- Chọn **Instance**
- Chọn **Action → Security → Modify IAM role**

![Untitled](/images/part2/2-2.1-step2-1.png)

- Chọn role
- Chọn **Update IAM Role**

![Untitled](/images/part2/2-2.1-step2-2.png)
## Bước 3 Create SSM endpoint

Bước này chỉ cần thực hiện khi bạn muốn kết nồi **Instance** nằm ở **private subnet**

- Search dịch vụ ***VPC***
- Chọn **VPC**
- Chọn tab **Endpoints**
- Chọn **Create endpoints**

![Untitled](/images/part2/2-2.1-step3-1.png)
- Đặt tên
- search ***ssm***
- chọn theo hình

![Untitled](/images/part2/2-2.1-step3-2.png)

- Chọn ssm
- Chọn VPC
- Chọn subnet
- Chọn Security group bạn dùng cho instace

![Untitled](/images/part2/2-2.1-step3-3.png)

- Chọn **Create**

Tương tự ta cần phải tạo 3 endpoint bao gồm:

- com.amazonaws.us-east-1.ssm
- com.amazonaws.us-east-1.ssmmessages
- com.amazonaws.us-east-1.ec2messages

### Bước 4 test connection

- Tại Instance của bạn chọn **Connect**

![Untitled](/images/part2/2-2.1-step4-1.png)

- chọn tab **Session Manager**
- Chọn **Connect**

![Untitled](/images/part2/2-2.1-step4-2.png)

## 2.2. Zerotier setup

### Bước 1: tạo tài khoản zerotier

- Truy cập vào trang web

[ZeroTier | Global Networking Solution for IoT, SD-WAN, and VPN](https://www.zerotier.com/)

https://www.zerotier.com/

- Tiến hành tạo tài khoản nếu chưa có hoặc đăng nhập nếu đã có

### Bước 2: Tạo mạng ảo

- chọn như hình

![Untitled](/images/part2/2-2.2-step2-1.png)


bạn sẽ thấy xuất hiện một mạng ảo ms

![Untitled](/images/part2/2-2.2-step2-2.png)

- chọn mạng này và đổi tên cho nó
- ghi nhớ net ID để cho step kết tiếp

![Untitled](/images/part2/2-2.2-step2-3.png)

### Bước 3 cài zerotier lên instance của bạn

- Dán lệnh sau vào Session manage connect của instant để tiến hành download

```bash
curl -s [https://install.zerotier.com](https://install.zerotier.com/) | sudo bash
```

![Untitled](/images/part2/2-2.2-step2-4.png)

sau khi tải xong tiến hành kết nôi đến mạng ảo riêng

- Dán lệnh sau vào Session manage connect của instant

```bash
sudo zerotier-cli join YOUR_NETWORD_ID
```

Trở lại trang web

![Untitled](/images/part2/2-2.2-step2-5.png)

bạn thấy có một ip ms xin vào mạng của bạn

- Chấp nhận nó

![Untitled](/images/part2/2-2.2-step2-6.png)

### Bước 4 cài lên zerotier lên thiết bị của bạn

tại trang

[Download - ZeroTier](https://www.zerotier.com/download/)

tiến hành tải và cài đặt phiên bản phù hợp vs hệ điều hành của bạn

### Ví dụ sau khi cài đặt hoàn tất trên window của mình

- Join network như hình
- nhập network id của bạn
- và tiến hành chấp nhận như Bước 3 trên

## 2.3. Kết nối SSH

tiến hành tương tư như các bước kết nối bằng địa chỉ ip public ở bài 1

bạn chỉ việc thay địa chỉ ip public bằng pivate ip được cấp ở bước 3. 

![Untitled](/images/part2/2-2.3-1.png)

sau khi đợi vài giây 

![Untitled](/images/part2/2-2.3-2.png)

<aside>
❗ sau khi tải các packet cần thiết bạn nên xóa ***nat gateway*** để giảm chi phí cho hệ thống của mình

</aside>