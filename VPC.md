
# VPC
Within AWS, you have something called a VPC (Virtual Private Cloud).
A VPC is a virtual private cloud which works like a private network to isolate the resources within it (https://docs.aws.amazon.com/quicksight/latest/user/vpc-terminology.html  |  https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html). 

- It is a virtual network that you can create (by default, you get one when you create an AWS account).
- Logically isolated from other virtual networks in the AWS cloud. (like a fenced area)
- You can launch your AWS resources, such as Amazon EC2 instances, into your VPC.
- You can configure your VPC; you can select its IP address range, create subnets, and configure route tables, network gateways, and security settings.

![VPC Architecture](https://github.com/user-attachments/assets/12a8b17d-4650-4ded-afbe-2f6197f1f75a)


## VPC Components
- **Subnets**: A range of IP addresses in your VPC, used to increase security and efficiency of network communications. A subnet must reside within a single Availability Zone. You can think of them like postal codes, used for routing packages from one location to another.  A public subnet is exposed to the public, while a private subnet is not. You have to create a subnet within a VPC to launch instances.

- **Route Tables**: Use route tables to determine where network traffic from your subnet or gateway is directed. Contains a set of rules, called routes, that are used to determine where network traffic is directed.

- **Security Groups**: Acts as a virtual firewall for your instance to control incoming and outgoing traffic. When you launch an instance, you can assign it to one or more security groups. You can add rules to each security group that allow traffic to or from its associated instances.

- **Internet Gateway**: A gateway enables communication between resources in your VPC and the internet.There are internet gateways, which allow communication between instances in your VPC and the internet, and virtual private gateways, which allow communication between your VPC and your own network.

## Creating a VPC 
- On Amazon VPC dashboard, choose 'Create VPC'
- Choose 'VPC only' for Resources to create (this will create a VPC without any subnets)
- Give your VPC a name (optional)
- Specify the CIDR block (IP address range) for your VPC. This is a range of IP addresses that can be assigned to hosts, routers, and networks. Given in the form of x.x.x.x/y, where x.x.x.x is the IP address and y is the number of bits in the subnet mask. if y=24, then the address range is x.x.x.0 to x.x.x.255, i.e. you can have 256 addresses in the range., if y=16, then the address range is x.x.0.0 to x.x.255.255, i.e. you can have 65536 addresses in the range.

- Set IPv4 CIDR (e.g. 10.0.0.0/16)
- Click 'Create VPC'

## Creating a Subnet
- On Amazon VPC dashboard, choose 'Subnets'
- Click 'Create Subnet'
- Choose the VPC you created earlier
- Give your subnet a name (optional) (e.g. PublicSubnet, PrivateSubnet). A convention is to create two public subnets and two private subnets with each in a different availability zone for high availability.
- Choose an availability zone
- Choose an IPv4 CIDR block within the range of the VPC CIDR block (a common convention is to give a lot of range to private subnets and less to public subnets since more resources are not made public)
- To add the private subnet, click 'Add new subnet' and repeat the process for the private subnet.
An example of a CIDR block for a VPC with one public and one private subnet is: 
VPC CIDR: 10.0.0.0/16  (The first 16 bits are fixed, the last 16 bits are available, 65536 addresses)
Public Subnet CIDR: 10.0.0.0/24 (The first 24 bits are fixed, the last 8 bits are available, 256 addresses for public subnet)
Private Subnet CIDR: 10.0.1.0/24 (The first 24 bits are fixed, the last 8 bits are available, 256 addresses for private subnet)

## Creating an Internet Gateway
Until you create an internet gateway and attach it to your VPC, your instances will not be able to connect to the internet, and the internet will not be able to connect to your instances.. 
- On Amazon VPC dashboard, choose 'Internet Gateways'
- Click 'Create internet gateway'

## Attaching an Internet Gateway to a VPC
The internet gateway you created is not yet attached to your VPC. You need to attach it to your VPC to enable internet access.
- On the Internet Gateways page, select the internet gateway you created, under Actions, click 'Attach to VPC'

## Creating a Route Table
Until you create a route table and associate it with your subnet, your instances will not be able to connect to the internet. Every VPC comes with a default route table, but you can create additional route tables, altering the rules to enalbe internet access from your public subnet. The route table by default allows communication within the VPC subnets (including between public and private subnets), but not to the internet.

- On Amazon VPC dashboard, choose 'Route Tables'
- Click 'Create Route Table'
- Give your route table a name (optional) (e.g. PublicRouteTable)
- Choose the VPC you created earlier
- Click 'Create'
- Also create a route table for the private subnet (e.g. PrivateRouteTable)

## Associating a Route Table with a Subnet  
- On the Route Tables page, select the route table you created, under Actions, click 'Edit subnet associations'
- Do this for both the public and private route tables, associating the public route table with the public subnet and the private route table with the private subnet.

## Adding a Route to the Route Table
To make the public route table public, you need to add a route to the internet gateway.
- On the Route Tables page, select the public route table, under the Routes tab, click 'Edit routes'
- Click 'Add route'
- Destination: 0.0.0.0/0 (this is a wildcard for all IP addresses)
- Target: Select the internet gateway you created earlier


## Create EC2 Instance in your VPC
- On the EC2 dashboard, click 'Launch Instance'
- Choose an Amazon Machine Image (AMI) (e.g. Amazon Linux 2 AMI)
- Choose an instance type (e.g. t2.micro)
- Configure Instance Details
  - Network: Choose the VPC you created earlier
  - Subnet: Choose the public subnet you created earlier
  - Auto-assign Public IP: Enable
- firewall: Create a new security group
- Review and Launch

## Lauch an EC2 instance in a private subnet
- On the EC2 creation, follow same steps, but choose the private subnet, disable auto-assign public IP and create a new security group (e.g. sgprivate), leave the type aas SSH and source type as Anywhere


### To connect to the private instance from the public instance
Although, you cannot access a private server on the internet or outside the VPC, but you can access the public server within the same VPC and SSH into the privater server from there.
- Use an existing key pair or create a new key pair 
    - A key pair, consisting of a public key and a private key, is a set of security credentials that you use to prove your identity when connecting to an Amazon EC2 instance. For Linux instances, the private key allows you to securely SSH into your instance. For Windows instances, the private key is required to decrypt the administrator password, which you then use to connect to your instance. Amazon EC2 stores the public key on your instance, and you store the private key. 
    - To Create a key pair:
        - On the EC2 dashboard, click 'Key Pairs' under 'Network & Security'
        - Click 'Create Key Pair'
        - Give your key pair a name (e.g. key)
        - Click 'Create'
        - Download the key pair and store it in a secure location (the key pair must be stored in a secure location, as it can be used to access your instances and you cannot download it again!)

- On your local computer, run the following command to copy the key pair to the public instance
    - `sudo scp -i key.pem key.pem ec2-user@[public-instance-ip]:/home/ec2-user`   (replace key.pem with your key pair and public-instance-ip with the public instance IP)
        - If you do not have scp installed, you can install it with `apt-get update && apt-get install -y openssh-client`
        - If you get Unprotected private key file error, run `chmod 600 key.pem` to change the permissions of the key pair file
    -  `ssh -i key.pem ec2-user@[private-instance-ip]` (replace key.pem with your key pair and private-instance-ip with the private instance IP)
Even though we can access the private server from the public server, you can't access internet from within the private server (so you can't run command like `apt-get update` or `curl google.com` from the private server), unless you use a NAT Gateway.

## NAT Gateway
NAT Gateway (Network Address Translation) can be used to enable instances in a private subnet to connect to services outside the VPC (to access the internet), but prevent external services to initiate a connection to the instances.
### Creating a NAT Gateway in the public subnet (because the public subnet has internet access)
- On the VPC dashboard, choose 'NAT Gateways'
- Click 'Create NAT Gateway'
- Give your NAT Gateway a name (optional) and choose the public subnet
- Select Public for Connectivity type
- Click 'Allocate Elastic IP' and click 'Create NAT Gateway'
### Update the route table for the private subnet to use the NAT Gateway
- On the Route Tables page, select the private route table, under the Routes tab, click 'Edit routes'
- Click 'Add route'
    - Destination: 0.0.0.0/0
    - Target: Select the NAT Gateway you created earlier


## NACLs (Network Access Control List) 
NACLs is like a virtual firewall that protects the subnet. It is another layer of protection around the subnet. It is stateless (meaning if you allow an incoming request, you have to also have an outbound rule to allow it out, it does not remember the state of the request). 
- A network access control list (ACL) is an optional layer of security for your VPC that acts as a firewall for controlling traffic in and out of one or more subnets. You might set up network ACLs with rules similar to your security groups in order to add an additional layer of security to your VPC.
- By default, network ACLs allow all inbound and outbound traffic. You can create custom network ACLs with rules to allow or deny traffic based on protocol, port, and source or destination IP address.  But most of the time, people use security groups to control traffic in and out of the instances, instead of NACLs to control traffic in and out of the subnet. A security group is stateful, meaning if you allow an incoming request, you do not have to have an outbound rule to allow it out, it remembers the state of the request.
