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
