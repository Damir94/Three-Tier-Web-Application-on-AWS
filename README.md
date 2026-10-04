# How to Deploy a Three-Tier Web Application on AWS

<img width="1517" height="1037" alt="ChatGPT Image Oct 4, 2026, 06_37_13 PM" src="https://github.com/user-attachments/assets/5359238d-4e7b-446f-9b37-24db2627c5a4" />

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

### Create App Servers
- We will now launch two EC2 instances that will serve as our two App servers.

### Create First app server
- Let us created the first app server for App tier. For this we have to create another EC2 instance.
- Click on “Launch Instance”
- We will call the instance “app-server-1”
- On “AMI” select “Amazon Linux”
- Scroll down to “Instance Type” and select “t2.micro”
- Scroll down to “Key Pair”
- Click on the drop down and select the key pair we created previously “three-tier-key”
- Scroll down to “Network Settings”
- Click on “Edit”
- Click on the drop down on “VPC” and select our created VPC
- Then we will use the public subnet “Private-app-subnet-AZ1”
- On “Auto-assign public IP” we will leave it as “Disable”
- Choose “Create security group”
- Let us give the security group a name. We will call it “app-server-sg”
- Then on “Description – required” enter the name of the security group “app-server-sg”
- We will allow the SSH connection to this app server only from the “bastion-host”. So, the source will be the security group of the “bastion-host”, that is bastion-host-sg. Click on the drop down on “Source Type”.
- Select “Custom”
- Click on Source
- Select “bastion-host-sg”.
- Scroll down to the end
- Click on “Launch Instance"
- Click on “Instances”
- We have launched the app server. It is initializing, let us wait for it to pass the “2/2 checks”
- It has passed the “2/2 check”.

<img width="1602" height="361" alt="Screenshot 2026-10-04 at 5 11 32 PM" src="https://github.com/user-attachments/assets/cc463ee1-fcaf-4cca-9bff-374a69d3cbd4" />

### Create second app server
- We have to launch the second EC2 instance that will serve as our second app server.
- Click on “Launch Instance”
- We will call the instance “app-server-2”
- On “AMI” select “Amazon Linux"
- Scroll down to “Instance Type” and select “t2.micro”
- Scroll down to “Key Pair”
- Click on the drop down and select the key pair we created previously “three-tier-key”
- Scroll down to “Network Settings”
- Click on “Edit”
- Click on the drop down on “VPC” and select our created VPC
- Then we will use the private subnet “private-app-subnet-AZ2”
- On “Auto-assign public IP” we will leave it as “Disable”
- Choose “Select Existing security group”
- Click on the drop down on “Common Security Groups” and select “app-server-sg”
- Scroll down to the end
- Click on “Launch Instance”
- Click on “Instances”
- We have launched the “app-server-2”. It is initializing, let us wait for it to pass the “2/2 checks”
- It has passed the “2/2 check”.

<img width="1585" height="209" alt="Screenshot 2026-10-04 at 5 13 30 PM" src="https://github.com/user-attachments/assets/be49fa84-82af-4c6c-b8f8-abd942866abe" />

### Connecting to the servers
- Let us now connect to our servers. We will connect to the Bastion host and through the Bastion host, we will connect to the app servers.

### Connecting to the Bastion Host
- Let us first connect to the “bastion-host
- Select the “bastion-host”
- Copy the “Public IPv4 address”
- Open terminal and navigate to where the key pair file is saved. It is saved in my “Downloads” folder. So, I will run the command:
```bash
cd Downloads
ssh -i three-tier-key.pem ec2-user@3.238.53.122
```
- We are now connected to the bastion host.

<img width="716" height="333" alt="Screenshot 2026-10-04 at 5 14 54 PM" src="https://github.com/user-attachments/assets/204f6c4b-e469-40d4-9856-f9113a683e9f" />

### Connecting “app-server-1” through the Bastion host
- Let us connect to the first app server now. We have to use the private IP address of the first app server.
- Firstly, check if the private key has been copied to your Bastion host using the command:
```bash
cd home/ec2-user
ls
```
- There is no private key found on the Jump Server. If it has not been copied, exit the Jump server using the command:
```bash
exit
```

- We have to copy the private key to your Jump Sever using the command:
```bash
scp -i three-tier-key.pem three-tier-key.pem ec2-user@<BASTION_PUBLIC_IP>:/home/ec2-user/
```
- Then connect to your Bastion host again using the command:
```bash
ssh -i <Name of private>.pem ec2-user@<Public IP address of Bastion Host>
```
- Check again if the private key has been copied to your Bastion host using the command:
```bash
cd /home/ec2-user
ls
```

<img width="1295" height="356" alt="Screenshot 2026-10-04 at 5 17 42 PM" src="https://github.com/user-attachments/assets/41542d97-4784-49a3-9e6b-84041991880e" />

- You can see that the private key has been copied to the Bastion host. The next thing is to change the mode of the private key pair file to read only using the command:
```bash
chmod 400 <Name of private Key>.pem
ls -l
```

<img width="672" height="131" alt="Screenshot 2026-10-04 at 5 18 38 PM" src="https://github.com/user-attachments/assets/56d7c606-b9fa-4ebc-9c25-94a755e6f589" />

- You can see that it is read only. We have to use the private IP address of the first app server.
- Copy the private IPv4 address of the server:
- Run the command to connect to the private EC2 instance
```bash
ssh -i <Name of private>.pem ec2-user@<Private IP address of Private EC2 instance>
```

<img width="825" height="352" alt="Screenshot 2026-10-04 at 5 20 14 PM" src="https://github.com/user-attachments/assets/78e91fee-fda6-4788-95b6-e2e0119eff0f" />

- You can see that we are now connected to the first PHP server.

### Connecting “app-server-2” through the Bastion host
- Let us connect to the second app server now. We have to use the private IP address of the second app server.
- Firstly, we will duplicate the terminal window
- Navigate to where our private key file is saved:
```bash
cd Downloads
```
- Run the command to connect to the Bastion host:
```bash
ssh -i <Name of private>.pem ec2-user@<Public IP address of Bastion Host>
```
- Check again if the private key has been copied to your Bastion host using the command:
```bash
cd /home/ec2-user
ls
```

<img width="819" height="300" alt="Screenshot 2026-10-04 at 5 22 13 PM" src="https://github.com/user-attachments/assets/fef520b9-cdd6-4ccc-a150-f8186f4c278e" />

- The private key found on the Bastion host. We have to use the private IP address of the second app server.
- Copy the private IPv4 address of the server:
- Run the command to connect to the private EC2 instance
```bash
ssh -i <Name of private>.pem ec2-user@<Private IP address of app server 2>
```

<img width="788" height="326" alt="Screenshot 2026-10-04 at 5 23 10 PM" src="https://github.com/user-attachments/assets/9890daa8-de98-4c57-b9f3-b0700ff829dd" />

- You can see that we are now connected to the second app server.

### Install PHP on app servers
- We are going to install PHP on the two app servers.
- Install PHP on First app server
- On the terminal, run the command to update the installed packages on your server:
```bash
sudo dnf upgrade -y
```
- Run the command to Install the lamp-mariadb10.2-phph7.2 and php7.2 Amazon Linux Extras repositories to the latest versions of the LAMP MariaDB and PHP:
```bash
sudo dnf install -y httpd wget php php-fpm php-mysqli php-json php-devel php-mbstring php-xml
```

<img width="1354" height="131" alt="Screenshot 2026-10-04 at 5 25 04 PM" src="https://github.com/user-attachments/assets/89762157-11d7-4d98-813c-895ef7eb9e74" />

- Php has been installed.

### Install Apache on First app server
- We will now run the command to install Apache server:
```bash
sudo dnf install -y httpd
```

<img width="1339" height="221" alt="Screenshot 2026-10-04 at 5 26 11 PM" src="https://github.com/user-attachments/assets/ccb7f92a-cc91-4b0e-8374-511d80a3e8fa" />

- Apache has been installed. Run the command to start the Apache server:
```bash
sudo systemctl start httpd
```
- We will run the command to enable it to start automatically after reboot:
```bash
sudo systemctl enable httpd
```
- Run the command to confirm that Apache has been installed:
```bash
sudo systemctl is-enabled httpd
```
- You can see it has been “Enabled”. You can verify that httpd is on by running the following command:
```bash
sudo systemctl status httpd
```
- Let us check whether the httpd service is working:
```bash
curl http://localhost
```

<img width="853" height="219" alt="Screenshot 2026-10-04 at 5 27 21 PM" src="https://github.com/user-attachments/assets/6e59be6a-855e-43f2-8054-dea56c825b09" />

- It is working. Add your user (in this case, ec2-user) to the Apache group:
```bash
sudo usermod -a -G apache ec2-user
```
- Log out and then log in back again to pick u the new group, and then verify your membership. To log out (use the exit command or close the terminal window)
```bash
exit
```
- Then reconnect to our first app server using the command:
```bash
ssh -i three-tier-key.pem ec2-user@10.0.3.145
```
- I have reconnected to our first app server.
- Change the group ownership of /var/www and its contents to the Apache group:
```bash
sudo chown -R ec2-user:apache /var/www
```
- To add group, write permissions and to set the group ID on future subdirectories, change the directory permissions of /var/www and its subdirectories:
```bash
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
```
- To add group, write permissions, recursively change the file permissions of /var/www and its subdirectories:
```bash
find /var/www -type f -exec sudo chmod 0664 {} \;
```

<img width="1023" height="111" alt="Screenshot 2026-10-04 at 5 29 01 PM" src="https://github.com/user-attachments/assets/b17d8ff2-e8e0-4a06-9450-5dbd4eda1217" />

### Install Apache on Second app server
- We will now run the command to install Apache server
```bash
sudo dnf install -y httpd
```
- Apache has been installed. Run the command to start the Apache server:
```bash
sudo systemctl start httpd
```
- We will run the command to enable it to start automatically after reboot:
```bash
sudo systemctl enable httpd
```
- Run the command to confirm that Apache has been installed:
```bash
sudo systemctl is-enabled httpd
```
- You can see it has been “Enabled”. You can verify that httpd is on by running the following command:
```bash
sudo systemctl status httpd
```

<img width="967" height="410" alt="Screenshot 2026-10-04 at 5 30 29 PM" src="https://github.com/user-attachments/assets/053dd066-557d-4041-8bd4-08148914620e" />

- Let us check whether the httpd service is working:
```bash
curl http://localhost
```

<img width="839" height="269" alt="Screenshot 2026-10-04 at 5 31 00 PM" src="https://github.com/user-attachments/assets/d34c753a-d2e8-42c0-8935-c3afcf097fce" />

- It is working. Add your user (in this case, ec2-user) to the Apache group:
```bash
sudo usermod -a -G apache ec2-user
```
- Log out and then log in back again to pick up the new group, and then verify your membership. To log out (use the exit command or close the terminal window)
```bash
exit
```
- We will reconnect to our second app server using the command:
```bash
ssh -i <Name of private>.pem ec2-user@<Private IP address of app server 2>
```
- I have reconnected to our second app server.
- Change the group ownership of /var/www and its contents to the Apache group:
```bash
sudo chown -R ec2-user:apache /var/www
```
- To add group, write permissions and to set the group ID on future subdirectories, change the directory permissions of /var/www and its subdirectories:
```bash
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
```
- To add group, write permissions, recursively change the file permissions of /var/www and its subdirectories:
```bash
find /var/www -type f -exec sudo chmod 0664 {} \;
```
### Install phpMyAdmin on the app servers
- Let us install phpMyAdmin which is a simple web application on the two app servers.

### Install phpMyAdmin on the first app server
- We will start by installing the required dependencies.
```bash
sudo dnf install -y php-mbstring php-xml php-mysqlnd php-json php-gd php-zip
```

<img width="1212" height="371" alt="Screenshot 2026-10-04 at 5 33 44 PM" src="https://github.com/user-attachments/assets/e8611e1a-7a34-4446-82ab-6704705ed145" />

- Run the command to install wget:
```bash
sudo dnf install -y wget
```
- Run the command to Restart Apache:
```bash
sudo systemctl restart httpd
```
- Run the command to Restart php-fpm:
```bash
sudo systemctl restart php-fpm
```
- Run the command to navigate to the Apache document root at /var/www/html
```bash
cd /var/www/html
ls
```
- You can see that it is empty.
- To download the file directly to your instance, copy the link and paste it into a wget command, as in this example:
```bash
wget https://www.phpmyadmin.net/downloads/phpMyAdmin-latest-all-languages.tar.gz
```
- Run the command to check the content:
```bash
ls -lh
```
- Create a phpMyAdmin folder and extract the package into it with the following command.
```bash
mkdir phpMyAdmin && tar -xvzf phpMyAdmin-latest-all-languages.tar.gz -C phpMyAdmin --strip-components 1
```
- Run the command:
```bash
ls
```
- You can see the folder we have created. Run the command to Delete the phpMyAdmin-latest-alllanguages.tar.gz tarball:
```bash
rm phpMyAdmin-latest-all-languages.tar.gz
```
- Run the command:
```bash
ls
```

<img width="727" height="116" alt="Screenshot 2026-10-04 at 5 35 54 PM" src="https://github.com/user-attachments/assets/28c588cd-7b9c-45fd-9bd7-2180e6727a0c" />

- You can see that the tar.gz file has been deleted
- Now, we are done with the configuration of phpMyAdmin for the first app server. At a later stage when RDS is created, we will make certain changes to the config file.

### Install phpMyAdmin on the second app server
- Let us install phpMyAdmin which is a simple web application following the steps below
- - We will start by installing the required dependencies.
```bash
sudo dnf install -y php-mbstring php-xml php-mysqlnd php-json php-gd php-zip
```

<img width="1135" height="318" alt="Screenshot 2026-10-04 at 5 36 54 PM" src="https://github.com/user-attachments/assets/af33b9a9-98e8-469d-953c-339ee2374811" />

- Run the command to install wget:
```bash
sudo dnf install -y wget
```
- Run the command to Restart Apache:
```bash
sudo systemctl restart httpd
```
- Run the command to Restart php-fpm:
```bash
sudo systemctl restart php-fpm
```
- Run the command to navigate to the Apache document root at /var/www/html
```bash
cd /var/www/html
ls
```
- You can see that it is empty.
- To download the file directly to your instance, copy the link and paste it into a wget command, as in this example:
```bash
wget https://www.phpmyadmin.net/downloads/phpMyAdmin-latest-all-languages.tar.gz
```
- Run the command to check the content:
```bash
ls -lh
```
- Create a phpMyAdmin folder and extract the package into it with the following command.
```bash
mkdir phpMyAdmin && tar -xvzf phpMyAdmin-latest-all-languages.tar.gz -C phpMyAdmin --strip-components 1
```
- Run the command:
```bash
ls
```
- You can see the folder we have created. Run the command to Delete the phpMyAdmin-latest-alllanguages.tar.gz tarball:
```bash
rm phpMyAdmin-latest-all-languages.tar.gz
```
- Run the command:
```bash
ls
```

<img width="779" height="149" alt="Screenshot 2026-10-04 at 5 38 22 PM" src="https://github.com/user-attachments/assets/0f075ab2-b78f-41f1-8c2b-b0bcbf2ad285" />

- You can see that the tar.gz file has been deleted
- Now, we are done with the configuration of phpMyAdmin for the second app server. At a later stage when RDS is created, we will make certain changes to the config file.

### Create and configure an application load balancer
- We will now create the load balancer. We need a load balancer because we have two app servers running, so load balancer will help distribute traffic across these two servers running. The load balancer will also help in scalability.
- We will create load balancer for the public subnet and the private subnets.

### Create Load Balancer for Web Tier
- Let us create the load balancer for the public subnet. Go back to AWS Management console
- Click on “Load Balancers”
- Click on “Create Load Balancer”
- Click on “Create” under “Application Load Balancer”
- We will give the load balancer the name “my-alb”
- For the “Scheme”, we will use “Internet Facing” because we want to expose this load balancer to outside world (internet).
- Scroll down to “Network Mapping”
- Click on the drop down on “VPC” and select our created VPC
- Then select all the three availability zones. That is “us-east-1a”, “us-east-1b” and “us-east-1c”
- Make sure “Public-subnet-AZ1”, “Public-AZ2” and “Public-AZ3” as shown above.

<img width="1899" height="685" alt="Screenshot 2026-10-04 at 5 41 55 PM" src="https://github.com/user-attachments/assets/b3d43ac4-ce47-41dc-9f40-a6b11a5c8d17" />

- Scroll down to “Security Groups”
- Remove the default security group and select the security group we created for the Frontend ALB.
- Click on “Create security group”
- Give the security group a name, I will call it “my-alb-sg"
- Then for “Description” enter the name of the security group
- Click on the drop down on “VPC” and select our VPC.
- Then, click on “Add Rule”
- On “Port Range” enter “80”
- On “Source”, click on the search
- Select “0.0.0.0/0”
- Scroll down to the end
- Click on “Create Security Group”
- The security group has been created. Head back to the creation of the load balancer

<img width="1598" height="585" alt="Screenshot 2026-10-04 at 5 41 06 PM" src="https://github.com/user-attachments/assets/e710881f-c1a0-46b2-a624-352a294e4182" />

- Remove the default security group
- Then, click on the drop down
- Select “my-alb-sg” we just created
- Scroll down to “Listeners and Routing”
- Next, we are going to create a target group. Click on “Create Target Group”

### Create Target Group for Load Balancer
- We have to create a target group that will house our target instances.
- Click on “Create Target Group” and a new window will open
- For “Target Type” will be “Instances”
- We will name the Target group “my-alb-app-tg”

<img width="1273" height="759" alt="Screenshot 2026-10-04 at 5 43 40 PM" src="https://github.com/user-attachments/assets/c21868ab-75a8-4592-94ec-740de5b41661" />

- Scroll down to the end
- Click on “Next”
- We have to select the targets we want to register. Select “app-server-1” and “app-server-2”

<img width="1530" height="460" alt="Screenshot 2026-10-04 at 5 44 22 PM" src="https://github.com/user-attachments/assets/c0750471-b2b7-4007-b3ba-a5fd372c46e6" />

- Click on “Include as pending below”

<img width="1488" height="438" alt="Screenshot 2026-10-04 at 5 45 08 PM" src="https://github.com/user-attachments/assets/c27d8a24-de73-4850-a42f-1aba6f180784" />

- Scroll down to the end
- Click on “Next” again
- Scroll down
- Then click on “Create Target Group”
- We have created the Target Group.

<img width="1590" height="714" alt="Screenshot 2026-10-04 at 5 45 43 PM" src="https://github.com/user-attachments/assets/afc1a56c-2c76-43b0-b7da-0791b7598363" />

### Add the Target Group to Application Load Balancer
- Head back to our application load balancer page
- Click on “Refresh”
- Then click on the drop down on “Target Group
- Select the target group we just created.
- Then scroll down to the end
- Click on “create load balancer”
- We have created the load balancer for the App.

<img width="1580" height="780" alt="Screenshot 2026-10-04 at 5 46 25 PM" src="https://github.com/user-attachments/assets/9cb4c301-eb7d-459f-806c-f7e7ed85ab4c" />

### Allow Load Balancer Security Group in App Server
- By the time our instances are getting registered in the target group, we need to allow the load balancer security group in our App server security group. So, let us make the changes. Go back to our EC2 instances.
- Select “app-server-1”
- Click on “Security” tab
- Click on the security group url
- Click on “Edit Inbound Rules”
- We have to add a new rule to establish connection between the load balancer and “app-server-1”. Click on “Add Rule”
- On “Port Range”, enter “80”
- On “Source”, click on the search
- Select the security group of the load balancer, that is “my-alb-sg”
- Click on “Save rules
- This will establish connectivity between the load balancer and the instances.

<img width="1581" height="404" alt="Screenshot 2026-10-04 at 5 48 18 PM" src="https://github.com/user-attachments/assets/16ac535f-91fc-4155-ab55-faf67746f4ae" />

### Test the Load Balancer
- We have to create an index.html file in the www.html folder in both app servers to validate if the requests are going to both app servers.

## For app server 2
- Let is start with the second app server.
- Head back to our terminal window where we are connected to app-server-2.
- Run the command:
```bash
cd /var/www/html
ls
```
- Then put a sample text in the index.html file using the command:
```bash
echo "My Server 2 is running" > index.html
ls
```

<img width="698" height="188" alt="Screenshot 2026-10-04 at 5 49 15 PM" src="https://github.com/user-attachments/assets/dde231f4-b375-450d-a468-7e85c235bc16" />

### For app server 1
- Let us continue with the first app server.
- Head back to our terminal window where we are connected to app-server-2.
- Exit the terminal to take us to the connected Bastion host:
```bash
exit
```
- We are back to the connected Bastion host. Let us now connect to our app-server-1. Run the command:
```bash
ssh -i three-tier-key.pem ec2-user@10.0.3.145
```
- We are now connected to app-server-1. Run the command:
```bash
cd /var/www/html
ls
```
- Then put a sample text in the index.html file using the command:
```bash
echo "My Server 1 is running" > index.html
ls
```

<img width="735" height="175" alt="Screenshot 2026-10-04 at 5 50 23 PM" src="https://github.com/user-attachments/assets/56b20b92-16ea-496d-bb58-7958558ac88d" />

- We have to test the load balancer using the DNS name
- Click on “Load Balancers”
- Select the load balancer
- Copy the DNS name:
- Then, paste this on your browser:
- We are able to see that our application load balancer is working. Then refresh the page and see if you will see the messages on both app server.
- We can see that it is going to app server 1 and app server 2. So, the load balancer is working it is able to route the traffic between the two app servers.

<img width="1128" height="133" alt="Screenshot 2026-10-04 at 5 51 23 PM" src="https://github.com/user-attachments/assets/c6ec588c-1cf1-45e9-a020-a261007042be" />

<img width="993" height="148" alt="Screenshot 2026-10-04 at 5 53 15 PM" src="https://github.com/user-attachments/assets/896ddef3-6e39-4827-b44c-319eb029e550" />

- So, now we have created two layers of the architecture. We will now create the third layer which is the database.

### Create RDS instance
- We are going to create the third layer of the architecture which is the database.

### Create Database Subnet Group
- We have to create the subnet group for RDS database. Go to AWS Management Console.
- Search for “RDS”
- Click on “RDS”
- Click on “Subnet Groups"
- Click on “Create DB subnet group”
- Let us give the Subnet group a name. We will call it “db-subnet-group”
- In the description, we will use the name of the subnet group details. That is “db-subnet-group”
- Click on the drop down on “VPC” ad select our created VPC
- Click on the drop down on “Availability Zones”
- Select “us-east-1a”, “us-east-1b” and “us-east-1c”
- Then click on the drop down on “Subnets”
- Select the three database subnets
- Scroll down
- Click on “Create”
- We have created the subnet group.

<img width="1517" height="723" alt="Screenshot 2026-10-04 at 5 55 27 PM" src="https://github.com/user-attachments/assets/20329da5-06aa-4bff-9bc7-246a7c6a9d80" />

### Create database
- Here we have to create the actual database.
- Click on “Databases”
- Click on “create database”
- On “creation Method”, choose “Standard Create” and on “Engine Type”, choose “MySQL”

<img width="1904" height="616" alt="Screenshot 2026-10-04 at 5 58 48 PM" src="https://github.com/user-attachments/assets/1f9ec6e2-2a84-421e-9b33-0183c84b5649" />

- Scroll down to “Templates” and select “Dev/Test”
- Scroll down to “Availability and durability”, select “multi-AZ DB instance deployment (2 instances)” since in our architecture, we are using Multi-AZ.

<img width="1852" height="819" alt="Screenshot 2026-10-04 at 5 59 19 PM" src="https://github.com/user-attachments/assets/06a267e3-d58a-427b-8ab9-c2818d8dfdd5" />

- Scroll down to “Settings”
- On “DB Instance Identifier”, we will give it the name “my-db”

<img width="1779" height="721" alt="Screenshot 2026-10-04 at 5 59 50 PM" src="https://github.com/user-attachments/assets/eea67316-c0fc-4167-8989-d5477e371ffe" />

- Scroll down
- On “Master username”, we will leave it as “admin”,
- And on “Credentials Management”, select “Self-Managed"
- Then on “Master Password”, enter a password and confirm the password. I will use “IloveTexas1234” as password.

<img width="1812" height="596" alt="Screenshot 2026-10-04 at 6 00 19 PM" src="https://github.com/user-attachments/assets/afeb7c04-5c5e-41ff-acef-90c75503b207" />

- Scroll down to “Instance Configuration"
- Select “Burstable classes (includes t classes)” and also select “db.t3.micro”

<img width="1538" height="569" alt="Screenshot 2026-10-04 at 6 00 47 PM" src="https://github.com/user-attachments/assets/660fc3e8-98f7-4c9d-931b-32debc1518c7" />

- Scroll down to “Storage”.
- On “Storage Type”, select “General Purpose SSD (gp3)” and on “Allocated storage”, use “20” GiB.
- Click on “Additional Storage Configuration”
- Uncheck the box on “Enable storage autoscaling” since this is just a demo. But it is advisable to check this box when working on production.

<img width="1723" height="796" alt="Screenshot 2026-10-04 at 6 01 13 PM" src="https://github.com/user-attachments/assets/d3bee9bc-8f50-4b3c-a163-6e22fa7331cc" />

- Scroll down to “Connectivity”.
- Remember we have created a security group and allowed access from the Backend server (app server) to the database server. So, we will leave “Compute Source” as default, that is “Don’t connect to an EC2 compute resource”
- Click on the drop down on “Virtual Private Cloud (VPC)”, and select our VPC “three-tier-vpc”
- Click on the drop down on “DB Subnet Group” and select our created subnet group “db-subnet-group”
- or “Public Access”, we will select “No”

<img width="1760" height="727" alt="Screenshot 2026-10-04 at 6 02 20 PM" src="https://github.com/user-attachments/assets/8b81d247-1479-4e84-88e7-91d92618746a" />

- Scroll down to “VPC Security Group (Firewall)”
- On “VPC Security group (firewall)”, select “Create New”
- We have to create a security group for the database. We will call it “my-db-sg”
- Click on “Additional Configuration”

<img width="1730" height="706" alt="Screenshot 2026-10-04 at 6 03 19 PM" src="https://github.com/user-attachments/assets/c34b3944-f3fc-4c39-aefe-c9bd727210e2" />

- On “Tag - Optional”, leave everything as default and scroll down to “Monitoring”.
- Leave everything as default and scroll down to the end.
- Click on “Create Database”
- The database is being created. Wait for it to be created.

<img width="1565" height="312" alt="Screenshot 2026-10-04 at 6 04 28 PM" src="https://github.com/user-attachments/assets/0f6b6f8a-96b2-437b-9894-e4c336ccf4b7" />

### Enable Connectivity between App Tier and Data Tier
- Once the database is created, we have to enable connectivity between the app server and the database server. So, we have to go to the security group of the database.
- The database has been created and its status is “modifying”. Wait for the status to be “Available”
- The database is now available. Click on the database
- Select “Endpoints”
- Click on the VPC Security Group url
- Click on the security group ID
- Click on “Edit Inbound Rules”
- We have to add one entry for the app server. Click on “Add Rule”
- On “Port Range”, enter “3306”
- Click on the search field near “Custom”
- Remember that the App Tier will be connecting to the Data Tier. So, we will select the security group of the App tier here. Select “app-server-sg” to enable communication between the database and the App servers.
- And delete the default rule that was created.
- Click on “Save Rules”

<img width="1574" height="320" alt="Screenshot 2026-10-04 at 6 06 26 PM" src="https://github.com/user-attachments/assets/010bc80a-d526-4719-b7c0-bbbd4ab2a14c" />

### Configure phpMyAdmin with RDS
- We have to configure the phpMyAdmin with the database. To do this, we have to go to the database.
- Copy the “Endpoint”
- Go to connected first app server. If the connection has expired, we have to reconnect again.
- To do this, we have to first connect to our Bastion host using the command:
```bash
ssh -i three-tier-key.pem ec2-user@3.238.53.122
```
- We are now connected to our Bastion host. Let us connect to the first App server using the command:
```bash
ssh -i three-tier-key.pem ec2-user@10.0.3.145
```
- We are now connected to our first App server. Run the command:
```bash
cd /var/www/html
ls
```
- Then, switch directory to the phpMyAdmin folder using the command:
```bash
cd phpMyAdmin
ls
```
- Let us rename the file “config.sample.inc.php” to “config.inc.php” by using the command:
```bash
mv config.sample.inc.php config.inc.php
ls
```
- You can see that the file has been renamed. Now, let us open the file using the command:
```bash
vi config.inc.php
```
- Search for “host” in the file
- Replace “localhost” with the hostname (endpoint) of the RDS instance we just created. That is:
```bash
my-db.cu726k462mpf.us-east-1.rds.amazonaws.com
```
- Save the file by pressing “ESC” followed by “:wq” and press “enter”

<img width="845" height="254" alt="Screenshot 2026-10-04 at 6 14 45 PM" src="https://github.com/user-attachments/assets/b8176ec6-1915-465a-a7d3-affcffeb7347" />

- Then, go to the browser and paste the DNS name of the load balancer followed by phpMyAdmin:
```bash
my-alb-130448560.us-east-1.elb.amazonaws.com/phpMyAdmin
```

<img width="1738" height="612" alt="Screenshot 2026-10-04 at 6 16 21 PM" src="https://github.com/user-attachments/assets/b8da1678-0da0-45f9-81f1-d6b89fb853c6" />

- We can successfully access the sample PHP app

### Configure session stickiness
- Before we go ahead to log in, we should enable stickiness. This means that when a client sends a request, and later come back to make the same request, stickiness will make the request to be sent to one particular target. So, all subsequent requests from that client will be routed to the same target unless the target becomes unavailable. Stickiness will be enabled from the target group. Go to “Target Groups” on AWS Console
- Click on “Target Groups”
- Select the target group
- Click on the “Attributes” tab
- Click on “Edit"
- Scroll down to “Target selection configuration”
- Check the box on “Turn on Stickiness”
- Click on “Save Changes”
- We have configured the session stickiness.

<img width="1282" height="478" alt="Screenshot 2026-10-04 at 6 17 59 PM" src="https://github.com/user-attachments/assets/5a2f3765-8281-4cbe-9551-f39202bcbd3e" />

### Final Test
- Now, we will try to log in to the sample PHP App using the username and password used when creating the Database.
```bash
http://my-alb-130448560.us-east-1.elb.amazonaws.com/phpMyAdmin/index.php
```
- Enter the Username: admin
- Enter the Password: IloveTexas1234
- Click on “Log in”

<img width="669" height="390" alt="Screenshot 2026-10-04 at 6 21 12 PM" src="https://github.com/user-attachments/assets/8617e76b-f88a-408c-b681-8adcd4aa925b" />

- We are able to log in to the sample application. The load balancer will be able to distribute incoming traffic evenly to the application servers. This will make the system to be able to handle a large volume of requests without overloading any individual server.
- The application servers are responsible for running the PHP codes and communicating with the database server to fetch and manipulate data.
- The database server stores the data and provides a way for the application servers to retrieve and modify data.

### Services recap
- We have made used of Bastion host and VPC that helps to limit access to our environment and provide secured entry pints for administrators. Overall, the architecture is created because it allows for scalability.
