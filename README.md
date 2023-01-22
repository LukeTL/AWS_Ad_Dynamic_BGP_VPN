# AWS - Advanced Dynamic, BGP Based, Highly-Available VPN (AWS & Simulated On-premises)

This project is the implemetation of a Dynamic BGP Based, Highly-Available Site-To-Site VPN on a simulated Hybrid AWS and On-premise environment

The simulated AWS environment consists of

- 2 Subnets
- 2 EC2 Instances
- A TGW attachement
- A VPC attachment
- Default route towards the TGW

The simulated On-premises environment consists of

- 1 public subnet
- 2 private subnets
- 4 EC2 Instances
- The public subnet has 2 Ubuntu + strongSwan + FRRouting endpoints

## Setting up AWS Environment

**Creation of AWS VPC**

1. Go into the `VPC` service in the `AWS Managment Console`
2. Find `Your VPCs` under `Virtual private cloud` and select `Create VPC`
3. Set `Name tag` as `A4L-AWS`
4. Set `IPv4 CIDR` as `10.16.0.0/16`
5. Press `Create VPC`
6. From `A4L-AWS`, toggle `Action` and select `Edit VPC Settings`
7. Enable `DNS hostnames` and press `Save`

![image](https://user-images.githubusercontent.com/123274310/213908713-77b980e4-c3f8-4cd8-872b-daa01bdef4d9.png)

**Creation of AWS Private Subnet A & B**

1. From `VPC` service, go into `Subnets` and select `Create subnet`
2. Set VPC ID as `A4L-AWS`, `Subnet name` as `sn-aws-private-A`, `Availability Zone` as `us-east-1a` and `IPv4 CIDR block` as `10.16.32.0/20`
3. Create `sn-aws-private-A` and press `Create subnet` again for the second private subnet
4. Set VPC ID as `A4L-AWS`, `Subnet name` as `sn-aws-private-B`, `Availability Zone` as `us-east-1b` and `IPv4 CIDR block` as `10.16.96.0/20`
5. Create `sn-aws-private-B`

![image](https://user-images.githubusercontent.com/123274310/213909174-2eb87980-deb0-4c9f-b20a-3de35c2cb9a0.png)

**Creation of AWS Custom Route Table**

1. From `VPC` service, go into `Route tables` and select `Create Route Table`
2. Set `Name` as `A4L-AWS-RT` and `VPC` as `A4L-AWS` 

**Creation of AWS Transit Gateway & Attachment**

1. From `VPC` service, go into `Transit gateways` and select `Create transit gateway`
2. Set `ASN` as `64512` and ensure `DNS support`, `VPN ECMP support` and `Default route table association` are enabled
3. Create transit gateway

![image](https://user-images.githubusercontent.com/123274310/213909848-f2665964-be9c-470d-8978-ce50d6fc9c59.png)

4. From `VPC` service, go into `Transit gateway attachmenets` and select `Create transit gateway attachment`
5. Set `Name tag` as `A4LTGWATTACHMENT`, `Transit gateway ID` as `A4LTGW`, `VPC ID` as `A4L-AWS` and ensure `Subnet IDs` are set to `sn-aws-private-A` and `sn-aws-private-B`
6. Create attachment

![image](https://user-images.githubusercontent.com/123274310/213910188-6c4040c8-de86-4522-8e3a-712fe96acea5.png)

**Creation of AWS Transit Default Route**

1. From `VPC` service, go into `Route tables` and select `A4L-AWS-RT`
2. Select `Edit routes` and press `Add route`
3. Set `Destination` as `0.0.0.0/0` and `Target` as `Transit Gateway`
4. `A4LTGATTACHMENT` should appear. Go and select it and create route

![image](https://user-images.githubusercontent.com/123274310/213910585-39893b74-9fbd-46a2-8279-3dab8df5f94a.png)

**Association of AWS Private Subnets into AWS Route Table**

1. From `VPC` service, go into `Subnets` and select `sn-aws-private-A`
2. Select `Route table` and press `Edit route table association`
3. Set `Route table ID` as `A4L-AWS-RT` and select `Save`
4. Now exit and select `sn-aws-private-B`, repeating steps 2 and 3

![image](https://user-images.githubusercontent.com/123274310/213910820-e25fa13b-6a9c-4f8d-bd23-118423594d49.png)

**Creation of AWS Instance Secruity Group**

1. From `VPC` service, go into `Security Groups` and select `Create security group`
2. Set `Description` as `Default A4L AWS SG` and `VPC` to `A4L-AWS`
3. Add the 1st Inbound Rule, Set `Type` as `Custom TCP`, `Port range` as `22` and `Source` as `0.0.0.0/0`. The description is set as `Allow SSH IPv4 IN`
4. Add the 2nd Inbound Rule, Set `Type` as `All traffic` and `Source` as `192.168.8.0/21`. The desciption is set as `Allow All from ONPREM Networks`
5. Create Security Group
6. Edit Inbound rules and add a self reference rule. Set `Type` as `All Traffic` and Source as `AWSInstanceSG`
7. Save rules

![image](https://user-images.githubusercontent.com/123274310/213918823-d4c07938-1c03-499e-aede-298e0ada3d26.png)

**Creation of AWS EC2 Role & Setting up IAM Instance Profile**

1. From `IAM` service, go to `Roles` and select `Create role`
2. Choose `AWS Service` and select the `EC2` service
3. Choose `Create policy` and input this code

```
{
    "Version": "2012-10-17",
    "Statement": [
    {
        "Effect":"Allow",
        "Action":[
            "ssm:DescribeAssociation",
            "ssm:GetDeployablePatchSnapshotForInstance",
            "ssm:GetDocument",
            "ssm:DescribeDocument",
            "ssm:GetManifest",
            "ssm:GetParameter",
            "ssm:GetParameters",
            "ssm:ListAssociations",
            "ssm:ListInstanceAssociations",
            "ssm:PutInventory",
            "ssm:PutComplianceItems",
            "ssm:PutConfigurePackageResult",
            "ssm:UpdateAssociationStatus",
            "ssm:UpdateInstanceAssociationStatus",
            "ssm:UpdateInstanceInformation"
        ],
        "Resource":"*"
    },
    {
        "Effect":"Allow",
        "Action":[
            "ssmmessages:CreateControlChannel",
            "ssmmessages:CreateDataChannel",
            "ssmmessages:OpenControlChannel",
            "ssmmessages:OpenDataChannel"
        ],
        "Resource":"*"
    },
    {
        "Effect":"Allow",
        "Action":[
            "ec2messages:AcknowledgeMessage",
            "ec2messages:DeleteMessage",
            "ec2messages:FailMessage",
            "ec2messages:GetEndpoint",
            "ec2messages:GetMessages",
            "ec2messages:SendReply"
        ],
        "Resource":"*"
    },
    {
        "Effect":"Allow",
        "Action":"s3:*",
        "Resource":"*"
    },
    {
        "Effect":"Allow",
        "Action":"sns:*",
        "Resource":"*"
    }
    ]
}
```
4. Move to next section and name policy `root`
5. Save policy and move back to IAM role menu. Append new policy to role.
6. Name role `AWSEC2Role`

![image](https://user-images.githubusercontent.com/123274310/213919907-01f3bc5c-b255-4c04-b6a1-72284fa8932c.png)

**Setting AWS VPC Endpoints**

1. From `VPC` service, go into `Endpoints` and select `Create endpoint`
2. Set `Name Tag` as `AWSssmec2messagesinterfaceendpoint`, `Service Category` as `AWS Services`, `Services` as `com.amazonaws.us-east-1.ec2messages`, `VPC` as `A4L-AWS`, `Subnets` as `sn-aws-private-A` and `sn-aws-private-B` and `Security group` as `AWSInstanceSG`.
3. Create Endpoint and do the 2nd one
4. Set `Name Tag` as `AWSssminterfaceendpoint`, `Service Category` as `AWS Services`, `Services` as `com.amazonaws.us-east-1.ssm`, `VPC` as `A4L-AWS`, `Subnets` as `sn-aws-private-A` and `sn-aws-private-B` and `Security group` as `AWSInstanceSG`.
5. Create Endpoint and do the last one
6. Set `Name Tag` as `AWSssmmessagesinterfaceendpoint`, `Service Category` as `AWS Services`, `Services` as `com.amazonaws.us-east-1.ssmmessages`, `VPC` as `A4L-AWS`, `Subnets` as `sn-aws-private-A` and `sn-aws-private-B` and `Security group` as `AWSInstanceSG`.
7. Create last endpoint

![image](https://user-images.githubusercontent.com/123274310/213921434-9b06746d-31b4-42fe-8027-1352867c4efb.png)

**Creation of AWS EC2 Resource Instances**

1. From `EC2` service, go to `Instances` and select `Launch Instance`
2. Set `Name` as `AWS-EC2-A`, `AMI` as `Amazon Linux`, `Instance type` as `t2.micro`, `VPC` as `A4L-AWS`, `Subnet` as `sn-aws-private-A`, `Firewall` as `AWSInstanceSG` and `IAM instance profile` as `AWSEC2Role
3. Create instance and move on to the 2nd instance
4. Set `Name` as `AWS-EC2-B`, `AMI` as `Amazon Linux`, `Instance type` as `t2.micro`, `VPC` as `A4L-AWS`, `Subnet` as `sn-aws-private-B`, `Firewall` as `AWSInstanceSG` and `IAM instance profile` as `AWSEC2Role
5. Create Instance

![image](https://user-images.githubusercontent.com/123274310/213922481-00792db0-ac55-4ff9-b198-907a7c940a68.png)

**Summary of AWS environment setup**


## Setting up On-premises Environment

**Creation of On-premises VPC**

**Creation of On-premises Internet Gateway & Attachment**

**Creation of On-premises Public Subnet**

**Creation of On-premises Private Subnet 1 & 2**

**Creation of On-premises Route Tables**

**Setting up On-premises Routes**

**Creation of On-premises Instances & Elastic IPs**

**Creation of On-premises IAM Role & Setting up IAM Instance Profile**

**Creation of On-premises Security Group & VPC Endpoints**
