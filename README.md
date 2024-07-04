# Deploying a Python Application on AWS private instance

This project demonstrates how to deploy a simple Python application on a private subnet server within an AWS VPC. The setup includes:

- One VPC spanning two Availability Zones.
- One private subnet and one public subnet in each Availability Zone.
- NAT gateways in public subnets for outbound traffic from private subnets.
- A jump server in the public subnet for SSH access to the private subnet servers.
- Auto Scaling for application servers in private subnets.
- An Application Load Balancer (ALB) in the public subnets.

## Table of Contents

- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Setup](#setup)
  - [Step 1: Create a VPC](#step-1-create-a-vpc)
  - [Step 2: Create Subnets](#step-2-create-subnets)
  - [Step 3: Create an Internet Gateway and Attach to VPC](#step-3-create-an-internet-gateway-and-attach-to-vpc)
  - [Step 4: Create Route Tables](#step-4-create-route-tables)
  - [Step 5: Create NAT Gateways](#step-5-create-nat-gateways)
  - [Step 6: Launch EC2 Instances](#step-6-launch-ec2-instances)
  - [Step 7: Set Up Auto Scaling](#step-7-set-up-auto-scaling)
  - [Step 8: Configure Load Balancer](#step-8-configure-load-balancer)
  - [Step 9: Deploy the Python Application](#step-9-deploy-the-python-application)
  - [Step 10: Test the Setup](#step-10-test-the-setup)
- [IAM Roles and Policies](#iam-roles-and-policies)
- [Monitoring and Logging](#monitoring-and-logging)
- [Contributing](#contributing)
- [License](#license)

## Architecture

![Architecture Diagram](path/to/architecture-diagram.png)

## Prerequisites

- AWS account with administrative access.
- AWS CLI installed and configured.
- SSH key pair for accessing EC2 instances.
- Basic knowledge of AWS services (VPC, EC2, NAT Gateway, Auto Scaling, ALB).

## Setup

### Step 1: Create a VPC

1. Login to the AWS Management Console.
2. Navigate to the VPC Dashboard.
3. Create a new VPC with the following settings:
   - Name: `MyVPC`
   - IPv4 CIDR block: `10.0.0.0/16`

### Step 2: Create Subnets

1. Create two public subnets:
   - `PublicSubnet1`: `10.0.1.0/24` in `us-east-1a`
   - `PublicSubnet2`: `10.0.2.0/24` in `us-east-1b`
2. Enable auto-assign public IP for both public subnets.
3. Create two private subnets:
   - `PrivateSubnet1`: `10.0.3.0/24` in `us-east-1a`
   - `PrivateSubnet2`: `10.0.4.0/24` in `us-east-1b`

### Step 3: Create an Internet Gateway and Attach to VPC

1. Create an Internet Gateway named `MyIGW`.
2. Attach `MyIGW` to `MyVPC`.

### Step 4: Create Route Tables

1. Create a public route table named `PublicRouteTable`.
2. Add a route to `PublicRouteTable` for `0.0.0.0/0` pointing to `MyIGW`.
3. Associate `PublicSubnet1` and `PublicSubnet2` with `PublicRouteTable`.
4. Create a private route table named `PrivateRouteTable`.
5. Associate `PrivateSubnet1` and `PrivateSubnet2` with `PrivateRouteTable`.

### Step 5: Create NAT Gateways

1. Allocate two Elastic IPs.
2. Create a NAT Gateway in `PublicSubnet1` using one Elastic IP.
3. Create another NAT Gateway in `PublicSubnet2` using the second Elastic IP.
4. Update `PrivateRouteTable` to route `0.0.0.0/0` traffic through the NAT Gateways.

### Step 6: Launch EC2 Instances

1. Create a security group named `PublicSG`:
   - Allow SSH (port 22) from your IP.
   - Allow HTTP (port 80) and HTTPS (port 443) from anywhere.
2. Create another security group named `PrivateSG`:
   - Allow all traffic from `PublicSG`.
   - Allow HTTP (port 80) and HTTPS (port 443) from `PublicSG`.
3. Launch a jump server in `PublicSubnet1` with `PublicSG`.
4. Launch application servers in `PrivateSubnet1` and `PrivateSubnet2` with `PrivateSG`.

### Step 7: Set Up Auto Scaling

1. Create a launch template named `MyAppLaunchTemplate` with the same configuration as the application servers.
2. Create an Auto Scaling Group using `MyAppLaunchTemplate`, spanning `PrivateSubnet1` and `PrivateSubnet2`.

### Step 8: Configure Load Balancer

1. Create an Application Load Balancer named `MyAppALB` in `PublicSubnet1` and `PublicSubnet2`.
2. Attach `PublicSG` to `MyAppALB`.
3. Create a target group for HTTP and register the application servers.
4. Add listeners to `MyAppALB` to forward traffic to the target group.

### Step 9: Deploy the Python Application

1. SSH into the jump server using the key pair.
2. From the jump server, SSH into the private instances.
3. Install Python and dependencies on the private instances:
   ```sh
   sudo yum update -y
   sudo yum install python3 -y
   ```
4. Deploy and run your Python application:
   ```sh
   python3 my_app.py
   ```

### Step 10: Test the Setup

1. Access the application via the ALB DNS name.
2. Ensure the application is running correctly.
3. Verify the NAT Gateway allows private instances to reach the Internet.
4. Check the Auto Scaling group to ensure proper scaling of instances.

## IAM Roles and Policies

Ensure the EC2 instances have the necessary IAM roles and policies to interact with other AWS services.

## Monitoring and Logging

Set up CloudWatch for monitoring instance performance and logging application outputs.

## Contributing

Contributions are welcome! Please open an issue or submit a pull request.

## License

This project is licensed under the MIT License.

---

Replace `path/to/architecture-diagram.png` with the actual path to your architecture diagram if you have one.
