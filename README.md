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



**Creation of AWS EC2 Role & Setting up IAM Instance Profile**

**Setting AWS VPC Endpoints**

**Creation of AWS EC2 Resource Instances**

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
