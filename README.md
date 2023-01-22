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

Creation of AWS VPC

1. Go into the `VPC` service in the `AWS Managment Console`
2. Find `Your VPCs` under `Virtual private cloud` and select `Create VPC`
3. Set `Name tag` as `A4L-AWS`
4. Set `IPv4 CIDR` as `10.16.0.0/16`
5. Press `Create VPC`
6. From `A4L-AWS`, toggle `Action` and select `Edit VPC Settings`
7. Enable `DNS hostnames` and press `Save`
![image](https://user-images.githubusercontent.com/123274310/213908713-77b980e4-c3f8-4cd8-872b-daa01bdef4d9.png)

Creation of AWS Private Subnet A & B

Creation of AWS Custom Route Table

Creation of AWS Transit Default Route

Association of AWS Private Subnets into AWS Route Table

Creation of AWS Instance Secruity Group

Creation of AWS EC2 Resource Instances

Creation of AWS EC2 Role & Setting up IAM Instance Profile

Setting AWS VPC Endpoints

Creation of AWS Transit Gateway & Attachment

## Setting up On-premises Environment

Creation of On-premises VPC

Creation of On-premises Internet Gateway & Attachment

Creation of On-premises Public Subnet

Creation of On-premises Private Subnet 1 & 2

Creation of On-premises Route Tables

Setting up On-premises Routes

Creation of On-premises Instances & Elastic IPs

Creation of On-premises IAM Role & Setting up IAM Instance Profile

Creation of On-premises Security Group & VPC Endpoints
