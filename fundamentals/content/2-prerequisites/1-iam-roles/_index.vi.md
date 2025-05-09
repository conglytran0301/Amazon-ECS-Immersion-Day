---
title: "IAM Roles – Phân quyền truy cập dịch vụ"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: "<b> 2.1. </b>"
---

### Giới thiệu


[IAM Role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) là tính năng giúp nâng cao tính bảo mật trên AWS. Một IAM Role có thể được gán tạm thời cho các IAM User và các tài nguyên AWS trong nội bộ (internal) hoặc bên ngoài (external) tài khoản của bạn. Giả sử, khi một IAM User tiếp nhận (assume) một IAM Role, IAM User đó sẽ tạm thời có được những quyền hạn của IAM Role đó. Bạn nên sử dụng IAM Role khi bạn muốn cung cấp quyền truy cập ngắn hạn cho một IAM User hoặc tài nguyên AWS.

Để một IAM User có thể tiếp nhận (assume) IAM Role, thì bản thân IAM Role cần cho phép User thực hiện thao tác tiếp nhận (trust policy).

Một đặc tính quan trọng của IAM Role là không có thông tin chứng thực (credentials) nên bạn sẽ không thể đăng nhập trực tiếp vào tài khoản AWS bằng IAM Role.

![IAM ROLE](/images/2-prerequisites/1-iam-roles/ECS-Lab-IAM-Role.png)


---
#### 1. Tạo IAM Role cho ECS Task Execution

Role này cho phép ECS thực hiện các tác vụ hệ thống như pull container images và ghi logs.

1. Truy cập [IAM Dashboard](https://console.aws.amazon.com/iam/home)
2. Chọn **Access Management** > **Roles** > **Create Role**

![IAM Role Dashboard](/images/2-prerequisites/1-iam-roles/image.png)

3. Cấu hình role:
   - **Trusted Entity Type**: `AWS Service`
   - **Use Case**: `Elastic Container Service`
   - **Service or Use Case**: `Elastic Container Task`
   - Nhấn **Next**

![Chọn ECS Task](/images/2-prerequisites/1-iam-roles/image-1.png)

4. Thêm permissions:
   - Tìm và chọn `AmazonECSTaskExecutionRolePolicy`

![Chọn Policy](/images/2-prerequisites/1-iam-roles/image-2.png)

5. Đặt tên role: `retailStoreECSTaskExecutionRole`

{{% notice warning %}}
Tên role phải chính xác là `retailStoreECSTaskExecutionRole` để tương thích với các câu lệnh CLI!
{{% /notice %}}

![Xác nhận Role](/images/2-prerequisites/1-iam-roles/image-3.png)

Kết quả sau khi tạo thành công:

![Role đã tạo](/images/2-prerequisites/1-iam-roles/image-4.png)

#### 2. Tạo IAM Role cho ECS Task (Application Role)

Role này cho phép ứng dụng trong container truy cập các dịch vụ AWS và thực thi lệnh.

1. Tại [IAM Dashboard](https://console.aws.amazon.com/iam/home):
   - Chọn **Roles** > **Create Role**
   - **Trusted Entity Type**: `AWS Service`
   - **Use case**: `Elastic Container Service`
   - **Service or Use Case**: `Elastic Container Task`

![Chọn ECS Task](/images/2-prerequisites/1-iam-roles/image-1.png)

2. Bỏ qua bước Add permissions (sẽ thêm inline policy sau)

3. Đặt tên role: `retailStoreEcsTaskRole`

{{% notice warning %}}
Tên role phải chính xác là `retailStoreEcsTaskRole` để tương thích với các câu lệnh CLI!
{{% /notice %}}

![Tạo Role ECS Task](/images/2-prerequisites/1-iam-roles/image-5.png)

![Hoàn tất tạo Role](/images/2-prerequisites/1-iam-roles/image-7.png)

#### Thêm Inline Policy cho ECS Task Role

Để cho phép ECS Task thực thi lệnh từ xa qua Systems Manager và tương tác với ECS API:

1. Chọn role > **Add permissions** > **Create inline policy**

![Role Detail](/images/2-prerequisites/1-iam-roles/image-6.png)

2. Chuyển sang tab **JSON**, thêm policy sau:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ssmmessages:CreateControlChannel",
        "ssmmessages:CreateDataChannel", 
        "ssmmessages:OpenControlChannel",
        "ssmmessages:OpenDataChannel"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow", 
      "Action": [
        "ecs:ExecuteCommand",
        "ecs:DescribeTasks"
      ],
      "Resource": [
        "arn:aws:ecs:${var.aws_region}:${var.aws_account_id}:task/retail-store-ecs-cluster/*", // Thay thế biến môi trường
        "arn:aws:ecs:${var.aws_region}:${var.aws_account_id}:cluster/*" // Thay thế biến môi trường
      ]
    }
  ]
}
```

![Paste JSON Policy](/images/2-prerequisites/1-iam-roles/image-8.png)

3. Review và tạo policy:

![Đặt tên Policy](/images/2-prerequisites/1-iam-roles/image-9.png)

Xác nhận policy đã được gán:

![Xác nhận policy](/images/2-prerequisites/1-iam-roles/image-10.png)

---
### Tổng kết

Bạn đã hoàn thành:
- Tạo IAM Role cho EC2 
- Tạo Execution Role cho ECS Task
- Tạo Application Role cho ECS Task với inline policy
- Thiết lập quyền truy cập an toàn cho các thành phần trong hệ thống