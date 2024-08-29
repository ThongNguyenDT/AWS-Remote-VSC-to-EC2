---
title : "Kết nối VScode bằng EC2_Instances_Connect và AWS Systems Manager"
date :  "`r Sys.Date()`" 
weight : 7 
chapter : false
pre : " <b> 7. </b> "
---
Trong phần này chúng ta sẽ thực hiện kết  nối vscode remote ssh với EC2  nằm trong private subnet

thông qua hai phương pháp khác nhau:

<aside>
❗ Nếu bạn chưa có VPC, có thể tham khảo các bước tạo [ở đây](https://www.notion.so/REMOTE-DEVELOPMENT-WITH-VS-VSCODE-AND-EC2-aec58040853d4f229f7e1d1f930b417f?pvs=21)

</aside>

## EC2_Instances_Connect

## AWS Systems Manager

### Bước 1 Setup các endpoint cần thiết

để kết nối ssh với ec2 nằm trong private connect ta cần phải kết nối thông qua một thiết bị trung gian: có thể là bastion host, hoặc các endpoint 

Dùng **EC2 Instance Connect Endpoint**

Tương tự như các thiết lập endpoint ở bài trên ta cần tạo endpoint như sau

![Untitled](/images/part7/7.step1-l-1.png)

Dùng **bộ 3 ssm endpoint connect** 

Tương tự như các thiết lập endpoint ở bài trên ta cần tạo endpoint như sau

![Untitled](/images/part7/7.step1-r-1.png)

![Untitled](/images/part7/7.step1-r-2.png)

![Untitled](/images/part7/7.step1-r-3.png)

Lưu ý trong phần thiết lập:

- Chúng ta cần trỏ đúng đến VPC, subnet của EC2 cần kết nối
- Cần mở security group để EC2 và endpoint có thể kết nối được với nhau

Lưu ý:

- Chúng ta cần trỏ đúng đến VPC, subnet của EC2 cần kết nối
- Cần mở security group để EC2 và endpoint có thể kết nối được với nhau
- Cần phải tạo 3 endpoint bao gồm:
    - com.amazonaws.us-east-1.ssm
    - com.amazonaws.us-east-1.ssmmessages
    - com.amazonaws.us-east-1.ec2messages

**EC2 Instance Connect** cho phép chúng ta tạo trực tiếp VPC **endpoint** trong phần **kết nối** của ec2.

EC2 Instance Connect không yêu cầu thiết lập thêm các plugin phụ

EC2 Instance Connect không yêu cầu cấp quyền truy cập để kết nối ec2 vs dịch vụ quản lý vì bản chất nó là kết nối trực tiếp nên không cần dịch vụ quản lý

Chỉ có thể kết nối đến 1 EC2, Thiết lập riêng từng máy

Cấu hình dễ dàng hơn.

Bản phải thiết lập các endpoint này trong phần VPC Endpoint

Phải thiết lập plugin cho AWS CLI trên thiết bị nguồn

Phải có SSM agent trên EC2, lưu ý:

- Một số **AMI** mặt định đã có cấu hình agent
- [Bạn có thể tham khảo thêm ở đây](https://docs.aws.amazon.com/systems-manager/latest/userguide/ami-preinstalled-agent.html)

Bản chất SSM là một dịch vụ quản lý nên, bạn **cần cấp quyền** cho phép EC2 Tương tác với dịch vụ này. Bạn có thể tham khảo [các bước cấp quyền](https://www.notion.so/REMOTE-DEVELOPMENT-WITH-VS-VSCODE-AND-EC2-aec58040853d4f229f7e1d1f930b417f?pvs=21), [chi tiết quyền](https://www.notion.so/REMOTE-DEVELOPMENT-WITH-VS-VSCODE-AND-EC2-aec58040853d4f229f7e1d1f930b417f?pvs=21)

Cho phép bạn cấu hình cùng lúc nhiều thiết bị, đây là nội dung của dịch vụ [**AWS Systems Manager**](https://us-east-1.console.aws.amazon.com/systems-manager/home)

---

Cả 2 phương pháp này đều không yêu cầu:

- Thiết lập các port ssh cho quá trình kết nối
- Public EC2 bạn ra internet
- Cài đặt các gói user data của VScode

---

### Bước 2: Cấu hình EC2

<aside>
❗ Nếu bạn chưa có EC2 có thể tham khảo [ở đây](https://www.notion.so/REMOTE-DEVELOPMENT-WITH-VS-VSCODE-AND-EC2-aec58040853d4f229f7e1d1f930b417f?pvs=21) 
Bạn không cần thực hiện bước 7
Để kết nối session manager bạn phải có gán quyền role cho EC2 của mình có thể [tự tạo từ Json](https://www.notion.so/REMOTE-DEVELOPMENT-WITH-VS-VSCODE-AND-EC2-aec58040853d4f229f7e1d1f930b417f?pvs=21) hoặc theo cách đã có ở [bước 1](https://www.notion.so/REMOTE-DEVELOPMENT-WITH-VS-VSCODE-AND-EC2-aec58040853d4f229f7e1d1f930b417f?pvs=21)

</aside>

<aside>
❗ Các security group trong bài này mình có thể để security group mặt định có thể kết nối với nhau chi tiết sẽ có sơ đồ best practice sau

</aside>

Tạo kết nối **EC2 Instance Connect** trực tiếp trong tab **connect** của ec2

![Untitled](/images/part7/7.step2.png)

### Bước 3: Thử kết nối

### EC2_Instances_Connect

![Untitled](/images/part7/7.step3-l-1.png)


Nếu bạn nhìn thấy 3 cảnh báo trên và không thể kết nối được với ec2 của mình trong tab EC2 Instance Connect chứng tỏ các thiết lập về VPC, subnet của bạn hiện tại đang đi đúng hướng.

Chọn theo hình

![Untitled](/images/part7/7.step3-l-2.png)

nếu bạn nhận được kết quả nút kết nối màu vàng các thiết lập về sg và endpoint của bạn đã chính xác

![Untitled](/images/part7/7.step3-l-3.png)

nếu không được đọc các bước khắc phục ở bên

- Chọn Connect

![Untitled](/images/part7/7.step3-l-4.png)

### AWS Systems Manager

![Untitled](/images/part7/7.step3-r-1.png)

Tại tab Session Manager nếu bạn nhìn thấy nút kết nối có màu vàng như trong hình có nghĩa là bạn đã thiết lập thành công kết nối của mình

Nếu không được bạn có thể kiểm tra các thiết lập sau:

- Security group của bạn có thiết lập đúng không, recommend nếu không chắc chắn bạn nên thiết lập sg default cho cho cả endpoint và EC2 của mình
- Kiểm tra các endpoint và các subnet của mình có trỏ đúng đến cùng một subnet private không. Lưu ý rằng việc chọn subnet cho endpoint ở phần một là thiết lập target để endpoint kết nối đến chứ không phải là vị trí đặt endpoint.
- Không cần thiết lập endpoint cho public subnet
- Kiểm tra xem AMI của bạn có hổ trợ Agent ssm hay không. Nếu có thực hiện  reboot lại instance, nếu không có thể xóa đi, và tạo lại hoặc tiến hành kết nối EC2_Instances_Connect và cài đặt agent

- Chọn Connect

![Untitled](/images/part7/7.step3-r-2.png)

### Bước 4: kết nối bằng CLI

Các bước tiếp theo yêu cầu mấy phải được thiết lập AWS CLI v2

Nếu chưa có bạn có thể tham khảo [ở đây](https://www.notion.so/REMOTE-DEVELOPMENT-WITH-VS-VSCODE-AND-EC2-aec58040853d4f229f7e1d1f930b417f?pvs=21)

### EC2_Instances_Connect

Không yêu cầu cái gói cài đặt khác

### AWS Systems Manager

yêu cầu phải có plugin để thực hiện kết nối session trên cli, nếu chưa [có thể tham khảo tại đây](https://www.notion.so/REMOTE-DEVELOPMENT-WITH-VS-VSCODE-AND-EC2-aec58040853d4f229f7e1d1f930b417f?pvs=21)

```powershell
aws ec2-instance-connect ssh  --instance-id <instance_id>
```

![Untitled](/images/part7/7.step4-l-1.png)

```powershell
aws ssm start-session --target <instance_id>
```

![Untitled](/images/part7/7.step4-r-1.png)


### Bước 5: Cấu hình VSCode

Như vậy ta đã thành công kết nối ec2 thông qua cli, bây giờ ta cần  thiết lập chúng kết nối thông qua vscode

[Ta thực hiện các bước tương tự như kết nối với EC2 ở public subnet](https://www.notion.so/REMOTE-DEVELOPMENT-WITH-VS-VSCODE-AND-EC2-aec58040853d4f229f7e1d1f930b417f?pvs=21)

Tại bước edit host ta sẽ thực hiện việc tạo các tunnel để ssh thay vì dùng public ip

Công việc rất đơn giản, chỉ việc cấu hình file host của bạn như sau