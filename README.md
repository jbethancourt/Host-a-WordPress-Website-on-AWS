# Host a WordPress Website on AWS

This repository contains the resources and scripts used to deploy a static HTML web app on Amazon Web Services (AWS). The project utilizes various AWS services to ensure high availability, scalability, and security for the web application.

## Architecture Overview

The static HTML web app is hosted on EC2 instances within a highly available and secure architecture that includes:

1. **Virtual Private Cloud (VPC)**: with public and private subnets across two Availability Zones (AZs) for fault tolerance and high availability.
2. **Internet Gateway**: allowing communication between instances in the VPC and the internet.
3. **Security Groups**: acting as a virtual firewall to control inbound and outbound traffic.
4. **Public Subnets**: used for the NAT Gateway and Application Load Balancer, facilitating external access and load balancing.
5. **Private Subnets**: for web servers to enhance security.
6. **EC2 Instance Connect Endpoint**: for secure SSH access.
7. **Application Load Balancer (ALB)**: with a target group to distribute incoming web traffic across multiple EC2 instances.
8. **Auto Scaling Group (ASG)**: automatically adjusts the number of EC2 instances based on traffic, ensuring scalability and resilience.
9. **Amazon RDS**: for a managed relational database service.
10. **Amazon EFS**: for a scalable, elastic file storage system.
11. **AWS Certificate Manager (ACM)**: for managing SSL/TLS certificates.
12. **AWS Simple Notification Service (SNS)**: for notifications related to the Auto Scaling Group activities.
13. **Amazon Route 53**: for domain name registration and DNS management.

## Repository Structure

* **Deployment Script**: Contains the script for setting up the web application on the EC2 instance.
* **Architectural Diagram**: Visual respresentation of the AWS architecture.
  
## Deployment Scripts

### Initial Setup Script

This script is used for the initial setup of the web application on an EC2 instance. It includes steps for installing Apache, PHP, and mounting the Amazon EFS to the instance.

```bash
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

# install php 8 along with several necessary extensions for wordpress to run
sudo dnf install -y \
php \
php-cli \
php-cgi \
php-curl \
php-mbstring \
php-gd \
php-mysqlnd \
php-gettext \
php-json \
php-xml \
php-fpm \
php-intl \
php-zip \
php-bcmath \
php-ctype \
php-fileinfo \
php-openssl \
php-pdo \
php-tokenizer

# install the mysql version 8 community repository
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm 
#
# install the mysql server
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm 
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server 
#
# start and enable the mysql server
sudo systemctl start mysqld
sudo systemctl enable mysqld

# set permissions
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
sudo find /var/www -type f -exec sudo chmod 0664 {} \;
chown apache:apache -R /var/www/html

# download wordpress files
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
sudo cp -r wordpress/* /var/www/html/

# create the wp-config.php file
sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php

# edit the wp-config.php file
sudo vi /var/www/html/wp-config.php

# restart the webserver
sudo service httpd restart
```
### Auto Scaling Group Launch Template Script

This script is included in the launch template for the Auto Scaling Group, ensuring that new instances are configured correctly with the necessary software and settings.

```bash
#!/bin/bash
# update the software packages on the ec2 instance 
sudo yum update -y

# install the apache web server, enable it to start on boot, and then start the server immediately
sudo yum install -y http

d
sudo systemctl enable httpd 
sudo systemctl start httpd

# install php 8 along with several necessary extensions for wordpress to run
sudo dnf install -y \
php \
php-cli \
php-cgi \
php-curl \
php-mbstring \
php-gd \
php-mysqlnd \
php-gettext \
php-json \
php-xml \
php-fpm \
php-intl \
php-zip \
php-bcmath \
php-ctype \
php-fileinfo \
php-openssl \
php-pdo \
php-tokenizer

# install the mysql version 8 community repository
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm 
#
# install the mysql server
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm 
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server 
#
# start and enable the mysql server
sudo systemctl start mysqld
sudo systemctl enable mysqld

# environment variable
EFS_DNS_NAME=fs-02d3268559aa2a318.efs.us-east-1.amazonaws.com

# mount the efs to the html directory 
echo "$EFS_DNS_NAME:/ /var/www/html nfs4 nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 0 0" >> /etc/fstab
mount -a

# set permissions
chown apache:apache -R /var/www/html

# restart the webserver
sudo service httpd restart
```

## How to Use

1. Clone this repository to your local machine.
2. Follow the AWS documentation to create the required resources (VPC, subnets, Internet Gateway, etc.) as outlined in the architecture overview.
3. Use the provided scripts to set up the static HTML web app on EC2 instances within the VPC.
4. Configure the Auto Scaling Group, Load Balancer, and other services as per the architecture.
5. Access the web app through the Load Balancer's DNS name.

## Additional Resources
* AWS Documentation
* Git-Hub Repository for Web files:

This project is licensed under the MIT License - see the LICENSE file for details.
