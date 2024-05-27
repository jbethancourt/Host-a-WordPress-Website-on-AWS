Hosting a Static HTML Web App on AWS
This repository contains the resources and scripts used to deploy a static HTML web app on Amazon Web Services (AWS). The project utilizes various AWS services to ensure high availability, scalability, and security for the web application.

Architecture Overview
The static HTML web app is hosted on EC2 instances within a highly available and secure architecture that includes:

Virtual Private Cloud (VPC): with public and private subnets across two Availability Zones (AZs) for fault tolerance and high availability.
Internet Gateway: allowing communication between instances in the VPC and the internet.
Security Groups: acting as a virtual firewall to control inbound and outbound traffic.
Public Subnets: used for the NAT Gateway and Application Load Balancer, facilitating external access and load balancing.
Private Subnets: for web servers to enhance security.
EC2 Instance Connect Endpoint: for secure SSH access.
Application Load Balancer (ALB): with a target group to distribute incoming web traffic across multiple EC2 instances.
Auto Scaling Group (ASG): automatically adjusts the number of EC2 instances based on traffic, ensuring scalability and resilience.
Amazon RDS: for a managed relational database service.
Amazon EFS: for a scalable, elastic file storage system.
AWS Certificate Manager (ACM): for managing SSL/TLS certificates.
AWS Simple Notification Service (SNS): for notifications related to the Auto Scaling Group activities.
Amazon Route 53: for domain name registration and DNS management.
Deployment Scripts
Initial Setup Script
This script is used for the initial setup of the web application on an EC2 instance. It includes steps for installing Apache, PHP, and mounting the Amazon EFS to the instance.

bash
Copy code
# create to root user
sudo su

# update the software packages on the EC2 instance 
sudo yum update -y

# create an HTML directory 
sudo mkdir -p /var/www/html

# environment variable
EFS_DNS_NAME=fs-064e9505819af10a4.efs.us-east-1.amazonaws.com

# mount the EFS to the HTML directory 
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport "$EFS_DNS_NAME":/ /var/www/html

# install the Apache web server, enable it to start on boot, and then start the server immediately
sudo yum install -y httpd
sudo systemctl enable httpd 
sudo systemctl start httpd

# set permissions
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
sudo find /var/www -type f -exec sudo chmod 0664 {} \;

# restart the webserver
sudo service httpd restart
How to Use
Clone this repository to your local machine.
Follow the AWS documentation to create the required resources (VPC, subnets, Internet Gateway, etc.) as outlined in the architecture overview.
Use the provided scripts to set up the static HTML web app on EC2 instances within the VPC.
Configure the Auto Scaling Group, Load Balancer, and other services as per the architecture.
Access the web app through the Load Balancer's DNS name.
License
This project is licensed under the MIT License - see the LICENSE file for details.
