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

1. I have created a VPC dedicated for AWS resources. I had set the IPv4 CIDR such that I reserved a usable host range of 10.16.0.1 - 10.16.15.254 for the VPC.
2. I have a total of 2 private subnets dedicated for AWS resources called `sn-aws-private-A` and `sn-aws-private-B`. `sn-aws-private-A` was provisioned in us-east-1a zone while `sn-aws-private-B` was provisioned in us-east-1b zone, increasing the overall avalibility of the AWS resources in the us-east-1 region. I had reserved a usable host range of 10.16.32.1 - 10.16.47.254 for `sn-aws-private-A` and 10.16.96.1 - 10.16.111.254 for `sn-aws-private-B`.
3. A custom route table for the AWS VPC was created under the name of `A4L-AWS-RT`.
4. The transit gateway for the AWS resources was created. The ASN for this transit gateway was set to 64512 with DNS support, VPN ECMP support and Default route table association enabled. This will allow for my AWS VPC to interconnect with the On-premises Network (Simulated using a VPC). I then attached the transit gateway to the two private subnets (Availability Zones of us-east-1a & us-east-1b) in my AWS VPC. Lastly for traffic to flow via the transit gateway, I created a default route for the transit gateway under the custom route table `A4L-AWS-RT`.
5. The AWS private subnets were then associated to the AWS route table. Any traffic targeted to any IP addresses in the range of 10.16.0.0/16 will be handled within the VPC, while traffic targeted outside of this range will be pushed to the Transit Gateway to be allocated outside.
6. The AWS Security Group called `AWSInstanceSG` was created. It will be the default Security Group of the AWS VPC. There are 3 total inbound rules. The 1st allows for SSH IPv4 traffic at port 22, the 2nd allows all traffic from 192.168.8.0/21 which is allowing all traffic from the On-premises Network and the 3rd is a self reference rule which prevents traffic from sources which do not share the same security group as the VPC.
7. A role was created for the setup of the EC2 Instances and VPC endpoints.
8. 3 VPC Endpoints were then setup. The ssm endpoint is the endpoint for the System Manager Service, the ec2messages endpoint is used by Systems Manager to make calls from the SSM Agent to the Systems Manager service and the ssmmessages endpoint is required for connecting to the AWS instances through a secure data channel using Session Manager.
9. Finally 2 EC2 Instances were created in the AWS VPC to simulate AWS resources. `AWS-EC2-A` ran in subnet `sn-aws-private-A` while `AWS-EC2-B` ran in subnet `sn-aws-private`.

## Setting up On-premises Environment

**Creation of On-premises VPC**

1. From `VPC` service, go to `Your VPCs` and select `Create VPC`
2. Set `Name tag` as `ONPREM`, `IPv4 CIDR` as `192.168.8.0/21`
3. Create VPC and toggle `Action` and select `Edit VPC settings`
4. Tick `Enable DNS hostnames` and save

![image](https://user-images.githubusercontent.com/123274310/213933598-b073e1a3-9542-4619-a757-50bdef453991.png)

**Creation of On-premises Internet Gateway & Attachment**

1. From `VPC` service, go to `Internet gateways` and select `Create Internet gateway`
2. Set `Name tag` as `IGW-ONPREM` and create internet gateway
3. Toggle `Actions` and select `Attach to VPC`
4. Set `Available VPCs` as `ONPREM` and attach internet gateway

![image](https://user-images.githubusercontent.com/123274310/213933772-65e351ff-f9fd-4d83-8cf0-caa065c7c55a.png)

**Creation of On-premises Public Subnet**

1. From `VPC` service, go to `Subnets` and select `Create subnet`
2. Set `VPC ID` as ONPREM, `Subnet name` as `ONPREM-PUBLIC`, `Availability Zone` as `us-east-1a` and `IPv4 CIDR block` as '192.168.12.0/24'
3. Create subnet and toggle `Actions` and select `Edit subnet settings`.
4. Enable `Enable auto-assign public IPv4 address` and save

![image](https://user-images.githubusercontent.com/123274310/213934108-724d632f-e7b6-4202-a00f-05d6de5d9bdf.png)

**Creation of On-premises Private Subnet 1 & 2**

1. From 'VPC' service, go to `Subnets` and select `Create subnet`
2. Set `VPC ID` as ONPREM, `Subnet name` as `ONPREM-PRIVATE-1`, `Availability Zone` as `us-east-1a` and `IPv4 CIDR block` as '192.168.10.0/24'
3. Create subnet and move on to create the 2nd private subnet
4. Set `VPC ID` as ONPREM, `Subnet name` as `ONPREM-PRIVATE-2`, `Availability Zone` as `us-east-1a` and `IPv4 CIDR block` as '192.168.11.0/24'
5. Create 2nd subnet

![image](https://user-images.githubusercontent.com/123274310/213934303-f5991f91-a26c-4260-a1fb-18c77e309b1a.png)

**Creation of On-premises Route Tables**

1. From `VPC` service, go to `Route tables` and select `Create route table`
2. Set `Name` as `ONPREM-PRIVATE-RT1` and `VPC` as `ONPREM`
3. Create route table and move to creating the 2nd route table
4. Set `Name` as `ONPREM-PRIVATE-RT2` and `VPC` as `ONPREM`
5. Create route table and move to creating the last route table
6. Set `Name` as `ONPREM-PUBLIC-RT` and `VPC` as `ONPREM`
7. Create last route table

![image](https://user-images.githubusercontent.com/123274310/213934550-f99d1187-4f29-41a0-8d3e-b4d8c58483c1.png)

**Subnet Association for On-premises**

1. From `VPC` service, go to `Route tables` and select `ONPREM-PRIVATE-RT1`
2. Select `subnet associations` and add `ONPREM-PRIVATE-1`. Move to next route table
3. From `ONPREM-PRIVATE-RT2`, select `subnet associations` and add `ONPREM-PRIVATE-2`. Move to last route table
4. From `ONPREM-PUBLIC-RT`, select `subnet associations` and add `ONPREM-PUBLIC`.

![image](https://user-images.githubusercontent.com/123274310/213938725-a898a4c6-0d4d-480b-aad8-7b441a983ffd.png)

**Creating of On-premises Security Group**

1. From `EC2` service, go to `Security Groups` and select `Create security group`
2. Set `Security group name` to `ONPREMInstanceSG`, `Description` as `Default ONPREM SG`, `VPC` as `ONPREM`.
3. Add inbound rule with `Type` as `All traffic` and `Source` as `10.16.0.0/16`
4. Create Security Group
5. Select `Edit inbound rules` and add self referencing rule with `Type` as `All traffic` and `Source` as `ONPREMInstanceSG`.
6. Save Security group

![image](https://user-images.githubusercontent.com/123274310/213939269-a9e71839-f3e9-4211-81b7-cb7c4af4fc46.png)

**Creating On-premises Network Interfaces**

1. From `EC2` service, go to `Network Interfaces` and select `Create network interface`
2. Set `Description` as `Router1 PRIVATE INTERFACE`, `Subnet` as `ONPREM-PRIVATE-1`, `Security Groups` as `ONPREMInstanceSG` and `Name` as `ONPREM-R1-PRIVATE`. Disable source/destination check and move to next network interface.
3. Set `Description` as `Router1 PUBLIC INTERFACE`, `Subnet` as `ONPREM-PUBLIC`, `Security Groups` as `ONPREMInstanceSG` and `Name` as `ONPREM-R1-PUBLIC`. Disable source/destination check and move to next network interface.
4. Set `Description` as `Router2 PRIVATE INTERFACE`, `Subnet` as `ONPREM-PRIVATE-2`, `Security Groups` as `ONPREMInstanceSG` and `Name` as `ONPREM-R2-PRIVATE`. Disable source/destination check and move to next network interface.
5. Set `Description` as `Router2 PUBLIC INTERFACE`, `Subnet` as `ONPREM-PUBLIC`, `Security Groups` as `ONPREMInstanceSG` and `Name` as `ONPREM-R2-PUBLIC`. Disable source/destination check and finish up.

![image](https://user-images.githubusercontent.com/123274310/213940171-15fc6e7b-951e-415c-b968-76fb09c19812.png)

**Setting up On-premises Routes**

1. From `VPC` service, go to `Route table` and edit routes in `ONPREM-PUBLIC-RT`
2. Add route with `0.0.0.0/0` and set `Target` to `IGW-ONPREM`
3. Now edit routes in `ONPREM-PRIVATE-RT1`, adding `10.16.0.0/16` and set `Target` as `ONPREM-R1-PRIVATE`
4. Now edit routes in `ONPREM-PRIVATE-RT2`, adding `10.16.0.0/16` and set `Target` as `ONPREM-R2-PRIVATE`

![image](https://user-images.githubusercontent.com/123274310/213940430-ab45c0cb-574f-447a-baab-2507edb358ca.png)

![image](https://user-images.githubusercontent.com/123274310/213940441-ebe9d392-0fc1-4b05-b391-514c18401ba6.png)

![image](https://user-images.githubusercontent.com/123274310/213940451-6b4cb7a5-8a5c-42f9-b77f-72392232c07a.png)

**Creation of On-premises Elastic IPs and their Association**

1. From `EC2` service, go to `Elastic IPs` and select `Allocate Elastic IP address`
2. Allocate Elastic IP
3. Toggle `Actions` and press `Associate Elastic IP address`
4. Select `Network Interface` and choose `ONPREM-R1-PUBLIC`
5. Repeat steps 2 to 4 for the 2nd elastic IP but select `ONPREM-R2-PUBLIC` instead

![image](https://user-images.githubusercontent.com/123274310/214002604-2b91b850-65a7-43e7-9d49-ece6b8cc713a.png)

**Creation of On-premises IAM Role & Setting up IAM Instance Profile**

1. From `IAM` service, go to `Roles` and select `Create role`
2. Choose `AWS Service` and select the `EC2` service
3. Select `root` policy and append it
6. Name role `ONPREMEC2Role`

![image](https://user-images.githubusercontent.com/123274310/214003291-e6614579-2c16-4735-9ec2-ed4339eff1ce.png)

**Creation of On-premises VPC Endpoints**

1. From `VPC` service, go to `endpoints` and select `Create endpoint`
2. Set `Name tag` as `ONPREMssmVPCe`, `Service category` as `AWS services`, `Services` as `com.amazonaws.us-east-1.ssm`, `VPC` as `ONPREM`, `Subnets` as `ONPREM-PUBLIC` and `Security groups` as `ONPREMInstanceSG`. Move to next endpoint
3. Set `Name tag` as `ONPREMssmec2messagesVPCe`, `Service category` as `AWS services`, `Services` as `com.amazonaws.us-east-1.ec2messages`, `VPC` as `ONPREM`, `Subnets` as `ONPREM-PUBLIC` and `Security groups` as `ONPREMInstanceSG`. Move to next endpoint
4. Set `Name tag` as `ONPREMssmmessagesVPCe`, `Service category` as `AWS services`, `Services` as `com.amazonaws.us-east-1.ssmmessages`, `VPC` as `ONPREM`, `Subnets` as `ONPREM-PUBLIC` and `Security groups` as `ONPREMInstanceSG`. Move to last endpoint
5. Set `Name tag` as `ONPREMs3VPCe`, `Services` as `com.amazonaws.us-east-1.s3 (Gateway)`, `VPC` as `ONPREM`, `Route tables` as `ONPREM-PRIVATE-RT1`, `ONPREM-PRIVATE-RT2` and `ONPREM-PUBLIC-RT`.

![image](https://user-images.githubusercontent.com/123274310/214021999-52eee18d-c67b-4648-95f9-3b409b7b7975.png)

**Creation of On-premises EC2 Instances**

1. From 'EC2', go to `Instances` and select `Launch instance`
2. Set `Name` as `ONPREM-ROUTER1`, `AMI` to `ami-0ac80df6eff0e70b5` with `t3.small`, `Keypair` to `None`, `VPC` as `ONPREM`, `Subnet` as `ONPREM-PUBLIC`, `Network interface` as `ONPREM-R1-PUBLIC` & `ONPREM-R1-PRIVATE`, `Instance profile` as `ONPREMEC2Role` and add metadata as listed. Create Instance and move to the next one.

```
#!/bin/bash -xe
apt-get update && apt-get install -y strongswan wget
mkdir /home/ubuntu/demo_assets
cd /home/ubuntu/demo_assets
wget https://raw.githubusercontent.com/acantril/learn-cantrill-io-labs/master/aws-hybrid-bgpvpn/OnPremRouter1/ipsec-vti.sh
wget https://raw.githubusercontent.com/acantril/learn-cantrill-io-labs/master/aws-hybrid-bgpvpn/OnPremRouter1/ipsec.conf
wget https://raw.githubusercontent.com/acantril/learn-cantrill-io-labs/master/aws-hybrid-bgpvpn/OnPremRouter1/ipsec.secrets
wget https://raw.githubusercontent.com/acantril/learn-cantrill-io-labs/master/aws-hybrid-bgpvpn/OnPremRouter1/51-eth1.yaml
wget https://raw.githubusercontent.com/acantril/learn-cantrill-io-labs/master/aws-hybrid-bgpvpn/OnPremRouter1/ffrouting-install.sh
chown ubuntu:ubuntu /home/ubuntu/demo_assets -R
cp /home/ubuntu/demo_assets/51-eth1.yaml /etc/netplan
netplan --debug apply
```

3. Set `Name` as `ONPREM-ROUTER2`, `AMI` to `ami-0ac80df6eff0e70b5` with `t3.small`, `Keypair` to `None`, `VPC` as `ONPREM`, `Subnet` as `ONPREM-PUBLIC`, `Network interface` as `ONPREM-R2-PUBLIC` & `ONPREM-R2-PRIVATE`, `Instance profile` as `ONPREMEC2Role` and add metadata as listed. Create Instance and move to the next one.

```
#!/bin/bash -xe
apt-get update && apt-get install -y strongswan wget
mkdir /home/ubuntu/demo_assets
cd /home/ubuntu/demo_assets
wget https://raw.githubusercontent.com/acantril/learn-cantrill-io-labs/master/aws-hybrid-bgpvpn/OnPremRouter2/ipsec-vti.sh
wget https://raw.githubusercontent.com/acantril/learn-cantrill-io-labs/master/aws-hybrid-bgpvpn/OnPremRouter2/ipsec.conf
wget https://raw.githubusercontent.com/acantril/learn-cantrill-io-labs/master/aws-hybrid-bgpvpn/OnPremRouter2/ipsec.secrets
wget https://raw.githubusercontent.com/acantril/learn-cantrill-io-labs/master/aws-hybrid-bgpvpn/OnPremRouter2/51-eth1.yaml
wget https://raw.githubusercontent.com/acantril/learn-cantrill-io-labs/master/aws-hybrid-bgpvpn/OnPremRouter2/ffrouting-install.sh
chown ubuntu:ubuntu /home/ubuntu/demo_assets -R
cp /home/ubuntu/demo_assets/51-eth1.yaml /etc/netplan
netplan --debug apply
```
4. Set `Name` as `ONPREM-SERVER1`, `AMI` to `Amazon Linux` with `t2.micro`, `Keypair` to `None`, `VPC` as `ONPREM`, `Subnet` as `ONPREM-PRIVATE-1`, `Security groups` as `ONPREMInstanceSG` and `Instance profile` as `ONPREMEC2Role`. Create and move to last instance
5. Set `Name` as `ONPREM-SERVER2`, `AMI` to `Amazon Linux` with `t2.micro`, `Keypair` to `None`, `VPC` as `ONPREM`, `Subnet` as `ONPREM-PRIVATE-2`, `Security groups` as `ONPREMInstanceSG` and `Instance profile` as `ONPREMEC2Role`.

![image](https://user-images.githubusercontent.com/123274310/214038500-3a3938f8-1e84-4907-8049-98d612d33f65.png)

**Summary of On-premises environment setup**

## Starting Project! Setting up 2 VPN Connections linking AWS VPC & On-premises VPC via Transit Gateway


