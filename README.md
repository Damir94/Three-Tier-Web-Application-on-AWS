# How to Deploy a Three-Tier Web Application on AWS

- In this demo, we are going to deploy a three-tier web application in AWS. We will walk through the complete setup of a highly available three-tier architecture on AWS, perfect for modern web applications. The implementation includes creating a VPC with structured subnets, setting up public and private routing with IGW and NAT, deploying Application Load Balancers (ALBs), and launching scalable frontend and backend server groups. We will also configure a multi-AZ RDS database layer for data durability.

- Services used:
  • VPC
  • ALB
  • EC2
  • RDS

- Before we start deploying the application components, we need to create the networking base for the demo. In this case we will create a VPC, three public subnets that is one subnet in each availability zone. These three subnets with have our internet facing load balancers web servers and jump servers.

- You will then create a private app subnet in each availability zone for application deployment. And finally, we will create private subnets for database (db) instances.

- Once the VPC and subnets are done we will require gateways to configure incoming and outgoing traffic. Firstly, internet gateway needs to be configured with the VPC. Then a NAT gateway needs to be created in any of the public subnets. As the best practice, it is recommended to create multiple NAT gateways for redundancy. But in this demo, we will stick with a single NAT gateway.

- People often make the mistake of creating the NAT gateway in the private subnet. Although the NAT gateway is used to facilitate outgoing internet traffic for private subnets. It needs to reside in the public subnet.

- After the gateways are created, we will need to create the corresponding route tables for public, app and db subnets. Once the underlying networking infrastructure to host the three-tier application is created, we can start with the application components.

- For this use case, Route53 is optional and we can directly access the application via application load balancer. Then, we will create two EC2 instances on both availability zones and install php and Apache on it.

- And finally, we will create the RDS instances. The multi-AZ configuration is optional.

- This is the 3-tier architecture we are going to implement in a particular region. In the architecture we have a VPC, this is our custom VPC we are going to create. After creating the VPC, we will deploy VPC components like Subnets, Internet Gateway, NAT Gateway.

- We will create a public subnet for the Web tier, we will create private subnet for the application tier and we will have another private subnet for the data tier. We have divided our application across three availability zones namely Availability Zone 1, Availability Zone 2, and Availability Zone 3.

- In the public subnet, we will be using a load balancer. We need the load balancer because it will actually help to balance traffic across our servers. What will be happening the application tier’s private subnet is that, we will be running two application servers and we do not want users to access these two servers directly, that is why we are going through a load balancer.

- We have divided our subnet into three segments namely public-subnet-AZx, “Private-app-subnetAZx”, and “Private-db-subnet-AZx”. Where x=1,2,3.

- The Private-app-subnet-AZx will have the Frontend servers in the autoscaling group. The “Privateapp-subnet-AZx” will have the Backend Application Load Balancer (ALB) as well as the Backend servers which will be the autoscaling group. And finally, the “Private-db-subnet-AZx” will have databases.

- When a user tries to access our application, it will go through the load balancer and the load balancer will be the one to route traffic or distribute traffic across the servers in the application tier.

- There will be a communication between the application tier and the data tier

### Create VPC and Enable Hostname

- We are going to implement the architecture above. We will start by creating the Virtual Private Cloud (VPC) and enable hostname.
- Let us start by creating the VPC. Go to AWS Management console.
- Search for “VPC”
- Click on “VPC

<img width="1417" height="309" alt="Screenshot 2026-10-04 at 11 58 47 AM" src="https://github.com/user-attachments/assets/cdbf1fcd-5e70-4d38-aad1-37034e7ca197" />

- Click on “Your VPCs”
- Click on “Create VPC”

<img width="1233" height="212" alt="Screenshot 2026-10-04 at 12 00 22 PM" src="https://github.com/user-attachments/assets/05724750-92ce-4ff7-a68c-2a35b6a643b3" />

- Let us give the “VPC” a name, we will call it “three-tier-vpc”
- Then, we have to enter the “IPv4 CIDR Block”, we will use “10.0.0.0/16”
- We will leave the rest of the things as default and click on “Create VPC”

<img width="1311" height="768" alt="Screenshot 2026-10-04 at 12 02 32 PM" src="https://github.com/user-attachments/assets/36b4e511-3eec-470a-8935-4b43cb2c2159" />

- The VPC has been created. Click on “Your VPCs”
- You can see the VPC we have created.
- We have created the VPC, the next thing is to enable host name on the VPC.
- Select the VPC
- Click on the drop down on “Actions”
- Select “Edit VPC Settings
- Select “Enable DNS Hostnames”
- Click on “Save”
- We have enabled the hostname on the VPC.

<img width="1519" height="186" alt="Screenshot 2026-10-04 at 12 04 36 PM" src="https://github.com/user-attachments/assets/d93acf55-aa48-4b15-9bca-1296604b65ce" />

### Create Internet Gateway and attached it to the VPC

- We need to establish connectivity, for that we will require internet gateway for public web subnet.
- We will create an internet gateway and attached it to the VPC.
- Let us create the internet gateway.
- Click on “Internet Gateways”
- Click on “Create Internet Gateway”
- Let us give the internet gateway a name. We will call it “My-IGW”
- Click on “Create Internet Gateway”
- We have created the internet gateway.

<img width="1609" height="362" alt="Screenshot 2026-10-04 at 12 07 00 PM" src="https://github.com/user-attachments/assets/01c46741-0b7a-4c98-aa5e-20d6e41e7245" />

### Attach the Internet Gateway to the VPC

- We have to attach the created internet gateway to the VPC
- Click on the drop down on “Actions”
- Select “Attach VPC”
- Click on the field under “Available VPCs”
- Select the our VPC for this demo “three-tier-vpc”
- Click on “Attach Internet Gateway”
- We have attached the Internet Gateway to the VPC.

<img width="1599" height="317" alt="Screenshot 2026-10-04 at 12 08 27 PM" src="https://github.com/user-attachments/assets/2f302634-4f66-4e57-af30-3766ea9c850c" />

### Create Subnets

- We will now create the subnets. We are going to create nine subnets. That is three public subnets for the Web tier, three private subnets for the app tier and three private subnets for the data tier.

### Create Public Subnets for Web
- We have to create three public subnets for the web tier in three different availability zones.
- Click on “Subnets”
- Click on “Create Subnet”
- Click on the drop down on “VPC ID” and select our VPC named “three-tier-vpc”
- We have to give the subnet a name. We will call it “Public-subnet-AZ1”
- On “Availability Zone”, click on the drop down and select “us-east-1a”
- For the “IPv4 Subnet CIDR Block”, we will enter “10.0.0.0/24”

- Let us add the second public subnet called “Public-subnet-AZ2”. Click on “Add new Subnet”
- Let us give the subnet a name. We will call it “Public-subnet-AZ2”
- Click on the drop down on “Availability Zone” and select “us-east-1b”
- For the “IPv4 Subnet CIDR Block”, we will enter “10.0.1.0/24”

- Let us add the third public subnet called “Public-subnet-AZ3”. Click on “Add new subnet”
- We have to give the subnet a name. We will call it “Public-subnet-AZ3”
- Click on the drop down on “Availability Zone” and select “us-east-1c”
- For the “IPv4 Subnet CIDR Block”, we will enter “10.0.2.0/24”
- Click on “Create Subnet”
- The three public subnets for our web tier have been created

<img width="1604" height="180" alt="Screenshot 2026-10-04 at 12 14 09 PM" src="https://github.com/user-attachments/assets/7a9fe6c2-1b72-491e-a452-f97a44131499" />

### Create Private Subnets for App Tier

- We are going to create three private subnets for the App tier in three different availability zones.
- Click on “Create Subnet”
- Click on the drop down on “VPC ID” and select our VPC
- We have to give the subnet a name. We will call it “Private-app-subnet-AZ1”
- On “Availability Zone”, click on the drop down and select “us-east-1a”
- For the “IPv4 Subnet CIDR Block”, we will enter “10.0.3.0/24”

- Let us add the second private subnet called “Private-app-subnet-AZ2”. Click on “Add new subnet”
- We have to give the subnet a name. We will call it “Private-app-subnet-AZ2”
- On “Availability Zone”, click on the drop down and select “us-east-1b”
- For the “IPv4 Subnet CIDR Block”, we will enter “10.0.4.0/24”

- Let us add the third private subnet for app called “Private-app-subnet-AZ3”. Click on “Add new Subnet”
- We have to give the subnet a name. We will call it “Private-app-subnet-AZ3”
- On “Availability Zone”, click on the drop down and select “us-east-1c”
- For the “IPv4 Subnet CIDR Block”, we will enter “10.0.5.0/24”
- Click on “Create Subnet”
- We have created the three private subnets for app tier.

<img width="1602" height="378" alt="Screenshot 2026-10-04 at 12 18 11 PM" src="https://github.com/user-attachments/assets/1bd5b99d-57c4-49ef-942f-b804ffd13c23" />

### Create Private Subnets for Database

- Let us create three private subnets for the database in three different availability zones.
- Click on “Create Subnet”
- Click on the drop down on “VPC ID” and select our VPC
- We have to give the subnet a name. We will call it “Private-db-subnet-AZ1”
- On “Availability Zone”, click on the drop down and select “us-east-1a”
- For the “IPv4 Subnet CIDR Block”, we will enter “10.0.6.0/24”

- Let us add the second db private subnet. Click on “Add new subnet”
- We have to give the subnet a name. We will call it “Private-db-subnet-AZ2”
- On “Availability Zone”, click on the drop down and select “us-east-1b”
- For the “IPv4 Subnet CIDR Block”, we will enter “10.0.7.0/24”

- Let us add the third private db subnet for app called “Private-db-subnet-AZ3”. Click on “Add new Subnet”
- We have to give the subnet a name. We will call it “Private-db-subnet-AZ3”
- On “Availability Zone”, click on the drop down and select “us-east-1c”
- For the “IPv4 Subnet CIDR Block”, we will enter “10.0.8.0/24”
- Click on “Create Subnet”
- We have created the three private subnets for data tier. Click on “Subnets”

<img width="1606" height="343" alt="Screenshot 2026-10-04 at 12 23 54 PM" src="https://github.com/user-attachments/assets/143778af-7362-493a-834e-be489bdbb91e" />

### Enable Auto-Assign Public IP on Public Subnets

- We have to enable auto-assign public IP on the public subnets.
- Enable Auto-Assign Public IP on “Public-subnet-AZ1”
- Let us enable auto-assign public IP on the first public subnet called “Public-subnet-AZ1”
- Select “Public-subnet-AZ1”
- Click on the drop down on “Actions”
- Select “Edit Subnet Settings”
- Check the box on “Enable auto-assign public IPv4 address”
- Click on “Save”
- We have enabled the auto-assign public IP on the first public subnet.

<img width="1869" height="387" alt="Screenshot 2026-10-04 at 12 26 23 PM" src="https://github.com/user-attachments/assets/1ea94c57-8eab-4a7b-83f5-564ddf2295e1" />

- Enable Auto-Assign Public IP on “Public-subnet-AZ2”
- Let us enable auto-assign public IP on the first public subnet called “Public-subnet-AZ2”
- Select “Public-subnet-AZ2”
- Click on the drop down on “Actions”
- Select “Edit Subnet Settings”
- Check the box on “Enable auto-assign public IPv4 address”
- Click on “Save”
- We have enabled the auto-assign public IP on the second public subnet.

<img width="1324" height="380" alt="Screenshot 2026-10-04 at 12 28 42 PM" src="https://github.com/user-attachments/assets/6eff332f-343f-4421-a8da-5c819cada0f5" />

- Enable Auto-Assign Public IP on “Public-subnet-AZ3”
- Let us enable auto-assign public IP on the first public subnet called “Public-subnet-AZ3”
- Select “Public-subnet-AZ3”
- Click on the drop down on “Actions”
- Select “Edit Subnet Settings”
- Check the box on “Enable auto-assign public IPv4 address”
- Click on “Save”
- We have enabled the auto-assign public IP on the third public subnet.

<img width="1568" height="401" alt="Screenshot 2026-10-04 at 12 29 51 PM" src="https://github.com/user-attachments/assets/6781d82b-7e06-4ac8-b549-c96a96349d05" />

### Create Route Tables
- In this part, we will be creating the route tables for the public and private subnets.

### Create Route Table for Public Subnet for Web Tier
- Click on “Route Tables”
- Click on “Create Route Table”
- We have to give the route table a name. We will call it “Public-RT”
- Click on the drop down on “VPC” to select our VPC called “three-tier-vpc”
- Click on “Create Route Table”
- We have created the route table for the public subnets

<img width="1588" height="577" alt="Screenshot 2026-10-04 at 12 34 04 PM" src="https://github.com/user-attachments/assets/91b48441-35d9-4482-98b9-221839562fe0" />

### Create the Route Table for Private Subnet for App Tier
- We have to create the route table for the private subnet of app tier.
- Click on “Route Tables”
- Click on “Create Route Table”
- We have to give the subnet a name. We will call it “Private-app-RT”
- Click on the drop down on “VPC” to select our VPC
- Click on “Create Route Table”
- We have created the route table of the private subnet for app tier

<img width="1608" height="650" alt="Screenshot 2026-10-04 at 12 35 36 PM" src="https://github.com/user-attachments/assets/5134fb2a-ccff-47cc-b3a2-dbfe63c2445f" />

### Create the Route Table for Private Subnet for Data Tier
- We have to create the route table for the private subnet of the database.
- Click on “Route Tables
- Click on “Create Route Table”
- We have to give the subnet a name. We will call it “Private-db-RT”
- Click on the drop down on “VPC” to select our VPC
- Click on “Create Route Table”
- We have created the route table for the private subnet of the database.

<img width="1610" height="579" alt="Screenshot 2026-10-04 at 12 36 59 PM" src="https://github.com/user-attachments/assets/d9c26722-a92f-420c-a3fb-01f7a7be4779" />

### Create NAT Gateway in Public Subnets
- We will now create three NAT gateways in the three public subnets. It is important to note that the NAT gateway is created in the public subnet.

### Create NAT Gateway in “Public-subnet-AZ1”
- We have to create our first NAT Gateway in the public subnet “Public-subnet-AZ1”. We will call the NAT gateway “NAT-gateway-AZ1”.
- Click on “NAT Gateways”
- Click on “Create NAT Gateway”
- We will now give the NAT gateway a name. We will call it “NAT-gateway-AZ1”.
- For “Availability Mode”, select “Zonal"
- For the “Subnet”, we will select one of the public subnets for this demo. But in the best practice you should create multiple NAT gateways to ensure redundancy. So, click on the drop down and select “Public-subnet-AZ1”
- For “Connectivity Type”, select “Public”
- Also allocate Elastic IP to the NAT gateway by clicking on “Allocate Elastic IP”
- Then, click on “Create NAT Gateway”
- Click on “NAT gateways”
- The NAT gateway is being created. Wait for it to be created.
- The NAT gateway has been created.

<img width="1606" height="192" alt="Screenshot 2026-10-04 at 12 44 04 PM" src="https://github.com/user-attachments/assets/6b8443dc-6268-4854-b154-409ec1e78158" />

### Route Tables Subnets Associations
- We have to associate the route tables with the subnets

### Associate Public Subnets with its Route Table
- We have to associate the web subnet with its route table.
- Select the public web route table called “Public-RT”.
- Click on the “Subnet Associations” tab
- Click on “Edit Subnet Associations”
- Select the three public subnets.
- Click on “Save Associations”
- You can see that we have associated three public subnets with the public route table.

<img width="1598" height="562" alt="Screenshot 2026-10-04 at 12 45 16 PM" src="https://github.com/user-attachments/assets/5e37048b-fdbe-4a63-93aa-9c94f5d502c5" />

### Associate Web Subnets with its Route Table
- We have to associate the web subnet with its route table.
- Select the public web route table called “Private-app-RT”.
- Click on the “Subnet Associations” tab
- Click on “Edit Subnet Associations”
- Select the three private app subnets.
- Click on “Save Associations”
- You can see that we have associated three private app subnets with the app route table.

<img width="1599" height="434" alt="Screenshot 2026-10-04 at 12 49 40 PM" src="https://github.com/user-attachments/assets/6659d309-fb4d-4dbe-b79a-e4e6bf5c55dc" />

### Associate Database Subnets with its Route Table
- We have to associate the db subnet with its route table.
- Select the public web route table called “Private-db-RT”.
- Click on the “Subnet Associations” tab
- Click on “Edit Subnet Associations”
- Select the three database subnets.
- Click on “Save Associations”
- You can see that we have associated three database subnets with the db route table.

<img width="1588" height="391" alt="Screenshot 2026-10-04 at 12 51 48 PM" src="https://github.com/user-attachments/assets/bc2de666-9eab-450a-9c75-e4936aa9a7c8" />

### Add Route through the Internet Gateway and NAT Gateway
- We have to make changes in the route table so that the connectivity to the internet will be established from your internet gateway for the public subnets and the NAT gateway in the private subnets

### Add a Route through the Internet Gateway to Public subnet’s Route Table
- Let us add a route through the internet gateway to public subnet’s route table “Public-RT”.
- Click on “Route Tables”
- Select the public subnet “Public-RT”
- Click on the “Routes” tab
- Click on “Edit Routes”
- Click on “Add Route”
- Select “0.0.0.0./0”
- Click on the drop down on “Target”
- Select “Internet Gateway”
- Click on “igw-“
- Select “My-IGW”
- Click on “Save Changes”
- We have added routes through the Internet gateway to the public subnet’s route table

<img width="1587" height="316" alt="Screenshot 2026-10-04 at 12 54 55 PM" src="https://github.com/user-attachments/assets/8ad7ae6c-4f3f-4add-988b-1502426f2897" />

### Add a Route through the NAT Gateway to App Subnet’s Route Table
- Similarly, we have to add a route through the internet gateway to web subnet’s route table “Privateapp-RT”.
- To do this, go to the route tables.
- Click on “Route Tables”
- Select the app route table “Private-app-RT”
- Click on the “Routes” tab
- Click on “Edit Routes”
- Click on “Add Route”
- Select “0.0.0.0/0” for the “destination”
- Click on the drop down on “Target”
- Select “NAT Gateway”
- Click on “nat-“
- Select the NAT gateway
- Click on “Save Changes”
- We have added routes through the NAT gateway to app subnet’s route table.

<img width="1580" height="378" alt="Screenshot 2026-10-04 at 12 57 38 PM" src="https://github.com/user-attachments/assets/30f796fe-717d-427b-8a8e-94b7ff043da9" />

### dd a Route through the NAT Gateway to DB subnet’s Route Table
- Finally, let us add a route through the internet gateway to web subnet’s route table “Private-db-RT”
- Go to the route tables.
- Click on “Route Tables”
- Uncheck “Private-app-RT"
- Select the database route table “Private-db-RT”
- Click on “Routes” tab
- Click on “Edit Routes”
- Click on “Add Route”
- Select “0.0.0.0/0”
- Click on the drop down on “Target”
- Select “NAT Gateway”
- Click on “nat-”
- Select “NAT-gateway-AZ1”
- Click on “Save Changes”
- We have added routes through the NAT gateway to DB subnet’s route table

<img width="1582" height="606" alt="Screenshot 2026-10-04 at 12 59 59 PM" src="https://github.com/user-attachments/assets/4359e3a9-ad20-431d-a3c6-bd9a70485ad8" />

- We have completed our networking configuration. We have created VPC, route tables, subnets, internet gateway, NAT gateway and we have associated them. We can now move to deploying our application components.

### Create the Bastion Host
- We will be creating an EC2 instance for our Bastion host to serve as our Web server. We will be making use of a Bastion host that will provide secured access to other servers that are not directly accessible from the internet or other networks. We don’t want our application server to be accessed directly.
- Go AWS Management console.
- Search for “EC2”
- Click on “EC2”
- Click on “Launch Instance”
- We will call the instance “bastion-host”
- On “AMI” select “Amazon Linux”
- Scroll down to “Instance Type” and select “t3.micro”
- Scroll down to “Key Pair”
- Enter the name “three-tier-key”
- Click on “Create key pair”

<img width="1242" height="808" alt="Screenshot 2026-10-04 at 1 05 10 PM" src="https://github.com/user-attachments/assets/fc7d2678-86a3-4089-8c21-62ee6ab42aef" />

- Scroll down to “Network Settings”
- Click on “Edit”
- Click on the drop down on “VPC” and select our created VPC
- Then we will use the public subnet “Public-subnet-AZ1”
- Let us “enable” the Public IP. Click on the drop down and select “Enable”

<img width="1202" height="356" alt="Screenshot 2026-10-04 at 1 06 03 PM" src="https://github.com/user-attachments/assets/71e5152a-7dfe-408b-9e8e-2f6708a7a6a4" />

- Choose “Create security group”
- Let us give the security group a name. We will call it “bastion-host-sg”
- Then on “Description – required” enter the name of the security group “bastion-host-sg”

<img width="1209" height="322" alt="Screenshot 2026-10-04 at 1 07 13 PM" src="https://github.com/user-attachments/assets/19667e33-34a7-45a5-a29b-f80f395b7478" />

- Scroll down to the end
- Click on “Launch Instance”
- Click on “Instances”
- We have launched the bastion host. It is initializing, let us wait for it to pass the “2/2 checks”

<img width="1614" height="304" alt="Screenshot 2026-10-04 at 1 07 45 PM" src="https://github.com/user-attachments/assets/ab4780c1-bd2e-463e-ba45-bd54a70d1894" />
