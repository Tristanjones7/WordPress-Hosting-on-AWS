🖥️ Scalable WordPress Hosting on AWS

🌐 Architecture Overview

This project hosts a WordPress site on AWS using a secure, highly available, and scalable infrastructure. The architecture includes:

VPC with public/private subnets across two Availability Zones for high availability.
Internet Gateway for external connectivity.
Security Groups to control inbound/outbound traffic.
Public Subnets for the NAT Gateway and Application Load Balancer (ALB). (used 1 nat gateway but for a company i would of used 2 nat gateways)
Private Subnets for EC2 web servers to enhance security.
EC2 Instance Connect Endpoint for secure SSH access.
Application Load Balancer with a target group to distribute web traffic.
Auto Scaling Group (ASG) for dynamic scaling of EC2 instances.
Amazon RDS for a managed MySQL database backend.
Amazon EFS to provide shared file storage across EC2 instances.
AWS Certificate Manager (ACM) for SSL/TLS certificate management.
Amazon SNS to notify on ASG scaling events.
Amazon Route 53 for domain name and DNS management.

⚙️ Deployment Scripts

📝 WordPress EC2 Setup Script
Initial setup script for provisioning WordPress, Apache, PHP, MySQL, and EFS mounting on an EC2 instance.

<details> <summary>Click to expand the setup script</summary>
# Become root user
sudo su

# Update system packages
sudo yum update -y

# Create web root directory
sudo mkdir -p /var/www/html

# Mount EFS
EFS_DNS_NAME=fs-064e9505819af10a4.efs.us-east-1.amazonaws.com
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport "$EFS_DNS_NAME":/ /var/www/html

# Install Apache and start it
sudo yum install -y httpd
sudo systemctl enable httpd 
sudo systemctl start httpd

# Install PHP 8 and extensions
sudo dnf install -y php php-cli php-curl php-mysqlnd php-gd php-json php-xml php-zip php-mbstring php-intl php-fpm

# Install MySQL
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm 
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm 
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf install -y mysql-community-server 
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Set directory permissions
sudo usermod -a -G apache ec2-user
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
sudo find /var/www -type f -exec sudo chmod 0664 {} \;
sudo chown apache:apache -R /var/www/html

# Install WordPress
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
sudo cp -r wordpress/* /var/www/html/
sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php

# Restart Apache
sudo service httpd restart
</details>
📦 Auto Scaling Group Launch Template Script
Script for EC2 instance bootstrapping via Launch Template within Auto Scaling.

<details> <summary>Click to expand the ASG script</summary>
#!/bin/bash
# Update and install Apache
sudo yum update -y
sudo yum install -y httpd
sudo systemctl enable httpd 
sudo systemctl start httpd

# Install PHP 8 and required extensions
sudo dnf install -y php php-cli php-curl php-mysqlnd php-gd php-json php-xml php-zip php-mbstring php-intl php-fpm

# Install MySQL
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm 
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm 
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf install -y mysql-community-server 
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Mount EFS
EFS_DNS_NAME=fs-02d3268559aa2a318.efs.us-east-1.amazonaws.com
echo "$EFS_DNS_NAME:/ /var/www/html nfs4 nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 0 0" >> /etc/fstab
mount -a

# Set directory permissions
sudo chown apache:apache -R /var/www/html

# Restart Apache
sudo service httpd restart
</details>
🚀 How to Use

Clone this repository to your local machine:
git clone https://github.com/your-username/your-repo.git
cd your-repo
Provision AWS infrastructure:
Use the AWS Console, CLI, or Terraform to create the architecture outlined above.
Deploy WordPress:
SSH into an EC2 instance and run the setup script.
Attach it to the Load Balancer target group.
Test the site:
Access the public DNS of the Load Balancer to launch your WordPress setup.
📬 Contributing

Contributions are welcome! Please fork the repository and submit a pull request with any improvements.

📚 AWS Documentation References

For deeper understanding or to replicate components, refer to the following official AWS resources:

Amazon VPC
Subnet Routing & NAT Gateways
Security Groups
Amazon EC2
Amazon RDS
Amazon EFS
Application Load Balancer
Auto Scaling
AWS Certificate Manager
Amazon SNS
Amazon Route 53
Hosting WordPress on AWS (AWS Blog)

