# Hosting-WordPress-on-AWS
This project outlines the comprehensive procedure for hosting a fully functional WordPress website on the AWS infrastructure, tailored to meet the demanding standards of production-level performance, security, efficiency, and reliability.

# Cloud Tools Used;
- _Virtual Private Gateway (VPC)_
- _RDS_
- _Elastic File System_
- _NAT Gateway_
- _Application Load Balancer_
- _Route 53_
- _Auto-Scaling Group_
- _AWS Certificate Manager_


# Instructions

* [Create VPC with Public and Private Subnets](create-vpc-with-public-and-private-subnets)
* [Create NAT Gateways](#create-two-nat-gateways)
* [Control Traffic With Security Group](#control-traffic-with-security-group)
* [Create RDS for MYSQL Database](#create-rds-for-mysql-database)
* [Create Elastic File System](#create-elastic-file-system)
* [Install WordPress and Move Files to EFS](#install-wordpress-and-move-files-to-efs)
* [Create Application Load Balancer](#create-application-load-balancer)
* [Register A Domain Name In Route53](#register-a-domain-name-in-route53)
* [Point Domain Name to AWS Application Load Balancer](#point-domain-name-to-aws-application-load-balancer)
* [Launch The WordPress Site](#launch-the-wordpress-site)

![3tier](https://user-images.githubusercontent.com/110143245/221963546-63f1aa6c-b8d1-4601-9c7b-40c1432e83f1.png)


# Create the VPC with Public and Private Subnets 
_The VPC is designed in a three-tier architecture. The first tier which will be the web tier has two public subnets in two different AZs. The Second tier will be the application tier  with two public subnets in two different AZS. The third tier will be the database tier with two private subnets in two different AZs._

![vpc1](https://user-images.githubusercontent.com/110143245/221964506-5a19e568-caa9-4ec3-9c32-b17e641b3730.png)

### Stage 1 – Create the VPC
- Head to the VPC Dashboard 
- Select the appropriate region for the VPC
- Click on **Create VPC**
- Give it a **name;** "WordPress VPC"
- In the **IPV4 CIDR block** field enter "10.0.0.0/16"
- For the **IPV6 CIDR block**, Select "No IPV6 CIDR block"
- Leave **tenancy** to “Default”
- Click **Create VPC**
- Select the newly created VPC, Click **Actions** → Click **Edit DNS hostnames** 
- Select **Enable**
- Click **Save Changes**


### Stage 2 – Create Internet Gateway and Attach it to VPC
- Head to the VPC Dashboard 
- Select Internet Gateways
- Click **Create Internet Gateway**
- Give it a **name**
- Click **Create**
- Select the newly created IGW, Click **Actions** → **Attach to VPC**
- Set **VPC** to the “WordPress VPC” 
- Click **Attach**


### Stage 3 – Create Two Public Subnets in Two AZs
- Head to the VPC Dashboard 
- Select Subnets
- Click Create **Subnet**
- Give it a **name**; “Public Subnet1”
- Set **VPC** to “WordPress VPC” 
- Set the **Availability Zone** to the “first AZ”
- In the **IPV4 CIDR block** field enter “10.0.0.0/24”
- Click **Create subnet**

*_Repeat this process to create the second public subnet. Don’t forget to set the **Availability Zone** to the “second AZ”, for the **IPV4 CIDR block** field enter “10.0.1.0/24”._


### Stage 4 – Modify the IP setting of the Two Public Subnets 
- Select the newly created public subnet, Click **Actions** → **Modify auto-assign IP settings**
- Select **Enable auto-assign public IPV4 address**
- Click **Save**

*_Repeat the same process for the second Public Subnet._

### Stage 5A – Create Public Route Table
Head to the VPC Dashboard 
- Select Route Tables
- Click **Create Route Table** 
- Give it a **name;** “Public Route Table1”
- Set **VPC** to “WordPress VPC”
- Click **Create**

### Stage 5B – Create a Public Route 
- Select the newly created public route table, Click **Routes** → **Edit Route**
- Click **Add Route**
- In the **Destination field** enter “0.0.0.0/0”
- Set the **Target** to **Internet Gateway (IGW)** created in stage 2
- Click **Save**


### Stage 5C – Associate the Public Route Table to the Two Public Subnets
- Select the newly created public route table, **Click Subnet Associations** → **Edit subnet associations**
- Check the Two Public Subnets and Click **Save**

### Stage 6 – Create four Private Subnets in two different AZs
 **1.**
- Head to the VPC Dashboard 
- Select **Subnets**
- Click **Create Subnet**
- Give it a **name;** “Private Subnet1”
- Set **VPC** to “WordPress VPC”
- Set the **Availability Zone**I to the “first AZ”
- In the **IPV4 CIDR block** field enter “10.0.2.0/24”
- Click **Create subnet**

**2.**
- Select Subnets
- Click **Create Subnet**
- Give it a **name;** “Private Subnet2”
- Set **VPC** to “WordPress VPC”
- Set the **Availability Zone** to the “second AZ”
- In the **IPV4 CIDR bloc** field enter “10.0.3.0/24”
- Click **Create subnet**

**3.**
- Select Subnets
- Click **Create Subnet**
- Give it a **name;** “Private Subnet2”
- Set **VPC** to “WordPress VPC”
- Set the **Availability Zone** to the “first AZ”
-  In the **IPV4 CIDR block** field enter “10.0.4.0/24”
- Click **Create subnet**

**4.**
- Select Subnets
- Click **Create Subnet**
- Give it a **name;** “Private Subnet3”
- Set **VPC** to “WordPress VPC”
- Set the **Availability Zone** to the “second AZ”
- In the **IPV4 CIDR block** field enter “10.0.5.0/24”
- Click **Create subnet**

### Stage 6B – Create Route Table for Private Subnet2
- Head to the VPC Dashboard 
- Click on **Route Tables**
- Select the **default Route Table**created with the VPC
- Edit the **name** and label it “Private Route table2”
- Click **Subnet Associations** → **Edit subnet association**
- Check the Two Private Subnets with **CIDR block 10.0.3.0/24** and **10.0.5.0/24**


# Create Two Nat Gateways 
_The two NAT Gateways will be created in the public subnets and attached to the private route tables._

### Stage 1 – Create Elastic IP
- Navigate to the VPC dashboard and Click **Elastic IPs**
- Click **allocate elastic IP address**
Select
- Click **Allocate**


### Stage 2 – Create NAT Gateway
- Navigate to the VPC dashboard and Click **NAT gateways**
- Click **Create NAT Gateway**
- Set the **Subnet** to the “Public Subnet 1”
- Set the **Elastic IP allocation ID** to  Elastic IP from the drop down menu
- Give it a **name;** “NAT Gateway Public Subnet1”
- Click **Create Nat Gateway**
- Click **Edit Route Tables**
- Select **Private Route Tabel1**
- Click **Route**
- Click **Edit Route**
- Click **Add Route**
- In the **Destination** field enter “0.0.0.0/0”
- Set the **Target** to NAT Gateway “NAT Gateway Public Subnet1”
- Click **Save Routes**


### Stage 3 – Create A Second Elastic IP 
- Navigate to the VPC dashboard and Click **Elastic IPs**
- Click **allocate elastic IP address**
Select
- Click **Allocate**


### Stage 4 – Create A Second NAT Gateway
- Navigate to the VPC dashboard and Click **NAT gateways**
- Click **Create NAT Gateway**
- Set the **Subnet** to the “Public Subnet 2”
- Set the **Elastic IP allocation ID** to the "Elastic IP" from the drop down menu
- Give it a **name;** “NAT Gateway Public Subnet2”
- Click **Create Nat Gateway**
- Click **Edit Route Tables**
- Select **Private Route Tabel2**
- Click **Route**
- Click **Edit Route**
- Click **Add Route**
- In the **Destination** field enter “0.0.0.0/0”
- Set the **Target** to NAT Gateway “NAT Gateway Public Subnet2”
- Click **Save Routes**



# Control Traffic with Security Group
### Stage 1 – Create Security Group for the Application Load Balancer
- Navigate to the VPC dashboard and Click   **Security Groups**
- Click **Create Security Group**
- Set the **Security Group name** to “ALB Security Group”
- Set **VPC** to “WordPress VPC”
- On the **Inbound rule,** Click “Add rule”
- For **Type** select “HTTP” from the drop down menu
- Set the **Source** to “0.0.0.0/0”
- Click “Add rule” again
- For **Type** select “HTTPS” from the drop down menu
- Set the **Source** to “0.0.0.0/0”
- Click **Create Security Group**


### Stage 2 – Create Security Group for the Web Servers
- Navigate to the VPC dashboard and Click **Security Groups** 
- Click **Create Security Group**
- Set the **Security Group name** to “Web-Server Security Group”
- Set **VPC** to “WordPress VPC”
- On the **Inbound rule** Click “Add rule”
- For **Type** Select “HTTP” from the drop down menu
- Set the **Source** to “ALB Security Group”
- Click “Add rule” again
- For **Type** Select “HTTPS” from the drop down menu
- Set the **Source** to “ALB Security Group”
- Click **Create Security Group**


### Stage 3 – Create Security Group for the Database
- Navigate to the VPC dashboard and Click **Security Groups** 
- Click **Create Security Group**
- Set the **Security Group name** to “Database Security Group”
- Set **VPC** to “WordPress VPC”
- On the **Inbound rule** Click “Add rule”
- For **Type** Select “MYSQL/Aurora” from the drop down menu
- Set the **Source** to “Web-Server Security Group”
- Click **Create Security Group**


### Stage 4 – Security Group for the Elastic File System
- Navigate to the VPC dashboard and Click **Security Groups**
- Click **Create Security Group**
- Set the **Security Group name** to “EFS Security Group”
- Set **VPC** to “WordPress VPC”
- On the **Inbound rule** Click “Add rule”
- For **Type** select “NFS” from the drop down menu
- Set the **Source** to “Web-Server Security Group”
- Click **Create Security Group**


### Stage 5 – Edit Inbound Rule for the EFS Security Group
- On the **Inbound rule** of the EFS Security Group, Click “Add rule”
For **Type** select “NFS” from the drop down menu
Set the **Source** to “EFS Security Group”
Click **Save Rule**


# Create RDS for MYSQL Database
### Stage 1 – Create Subnet Group
- Navigate to the AWS services dashboard and under Database section Select **RDS**
- On the RDS dashboard, Select **Subnet Groups**
- Click **Create DB Subnet Group**
- Give it a **name;** “DB Subnet Group”
- For the **VPC** Select “WordPress VPC”
- On the **Add subnets**, Click on the dropdown of Availability Zones, Select the **two AZs** where the subnets are created on
- Click on the **Subnets** dropdown menu, Select the database tier which has the **CIDR block** of **10.0.4.0/24** in the first AZ and the **CIDR block** of **10.0.5.0/24** in the second AZ
- Click **Create**


### Stage 2 – Create the RDS Database
- On the RDS dashboard, Select Databases
- Click **Create database**
- Leave **Standard Create** selected
- From the **Engine options** Select “MYSQL”
- From the **versions** dropdown menu select the “latest MYSQL version”
- For the **Templates** Select “Free tier”
- On the **Settings,** Give the database identifier “name tag”
- Fill the Master username and Master password 
- For the **Connectivity**, Select “WordPress VPC”
- Under the **Additional connectivity configuration**, Select “DB Subnet Group”from the dropdown menu
- For the **Existing VPC Security Groups**, -Select the “Database security group” from the dropdown menu
- For the **Availability Zone**, Select any AZ you want to launch your master database on
- On the **Additional Configuration**, Give the **initial database name** any name
- Leave everything as **default**
- Click **Create database**


# Create Elastic File System
- Navigate to the AWS services dashboard and under Storage section Select EFS
- Click **Create file system**
- Click **Customize**
- Give it a **name**
- Leave everything as default
- Click **Nex**
- Under **Network**, Select “WordPress VPC”
- For **Subnet ID**, Select “CIDR block 10.0.4.0/24” and “10.0.5.0/24” respectively in both AZs
- For **Security Group** Select “EFS security group” in both AZs
- Click **Next**
- Leave the File System Policy
- Click **Next**
- Click Create
- Select the newly created elastic file system 
- Click **Attach**
- Select **Mount via DNS**
Copy the one under **Using the NFS client**
- Save it for later use


# Install WordPress and Move Files to EFS
### Stage 1– Create Security Group for SSH
- Navigate to the VPC dashboard created for this project
- Select **Security Groups**
- Click **Create security group**
- Give it a **name**: “SSH Security Group”
- For the **VPC** Select “WordPress VPC” from the dropdown menu
- Click **Add rule**
- For **Type** Select “SSH” from the dropdown menu
- For **Source** Select “My IP” from the dropdown menu
- Click **Create security group**


### Stage 2 – Update the security group for web server and EFS
- Navigate to the VPC dashboard created for this project
- Select **Security Groups**
- Select **EFS Security Group**
- Select **Inbound rules**
- Click **Edit inbound rules**
- Click **Add rule**
- For **Type** Select “SSH” from the dropdown menu
- For **Source** Select “SSH Security Group” from the dropdown menu
- Repeat the above process for the Web-Server Security Group


### Stage 3 – Create EC2 instance for installing WordPress
- Navigate to the VPC dashboard created for this project
- Select **EC2 Dashboard**
- Click **Launch Instance**
- Select **Amazon Linux 2 AMI**
- Select **t2 micro**
- Under **Network** Select “WordPress VPC” 
- For the **Subnet** Select “Public Subnet1” from the dropdown menu
- Leave everything as default
- Click **Next**
- Click **Next**
- Give it a **name**; “Setup-Server”
- Click **Next**
- Click **Select an existing security group**
- Select the “ALB security group, Web-Server security group and SSH security group”
- Click **Next**
- Click **Launch**
- Check the agreement and Click **Launch instance**
- Select the newly created EC2 instance 
- Copy the **Public IPV4 address**
- Open **Putty**
- SSH to the EC2 instance using the "public IPV4 addres" and the "instance key pair"


### Stage 4 – Commands to Install WordPress
1.	### Create the html directory and mount the EFS to it
~~~
sudo su
sudo yum update -y
sudo mkdir -p /var/www/html
~~~
- Navigate to the AWS services dashboard, Select **EFS**
- Select The application EFS created for this project
- Click **Attach**
- Select **Mount via DNS**
- Copy the one under **Using the NFS client**
- Paste it on the environment variable
~~~
# environment variable
~~~
EFS_DNS_NAME=fs-064e9505819af10a4.efs.us-east-1.amazonaws.com
~~~

# mount the efs to the html directory 
~~~
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport "$EFS_DNS_NAME":/ /var/www/html
~~~

2.	### Install apache
~~~ 
sudo yum install -y httpd httpd-tools mod_ssl
sudo systemctl enable httpd 
sudo systemctl start httpd
~~~

3.# install php 8 along with several necessary extensions for wordpress to run
~~~
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
~~~

4.	### Install mysql version 8 community repository
~~~
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm 
~~~

# install the mysql server
~~~
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm 
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server 
~~~

# start and enable the mysql server
~~~
sudo systemctl start mysqld
sudo systemctl enable mysqld
~~~

5.	### Set permissions
~~~
sudo usermod -a -G apache ec2-user
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
sudo find /var/www -type f -exec sudo chmod 0664 {} \;
chown apache:apache -R /var/www/html 
~~~

6.	### Download WordPress files
~~~
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
sudo cp -r wordpress/* /var/www/html/

~~~


7.	### Create the wp-config.php file
~~~
sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php

~~~


8.	### Edit the wp-config.php file
~~~
sudo vi /var/www/html/wp-config.php
Update the Database Credentials using the credentials of the RDS created in this project
~~~

9.	### Restart the webserver
~~~
sudo service httpd restart
~~~


10.	### Navigate to EC2 dashboard, Select the ALB DNS Name and open it on any browser to open WordPress



# Create Application Load Balancer
### Stage 1– Create two EC2 instances in the private subnet
- Navigate to AWS services Dashboard
- Select **EC2** 
- Click **Launch Instance**
- Select **Amazon Linux 2 AMI** 
- Select **t2 micro**
- Under **Network** Select “WordPress VPC” from the dropdown menu
- For the **Subnet** Select “Private Subnet1” (App Tier) from the dropdown menu
- Under **Additional Details**, Copy and Paste the following commands in the **User data**
~~~
                  #!/bin/bash
             yum update -y
             sudo yum install -y httpd httpd-tools mod_ssl
             sudo systemctl enable httpd
             sudo systemctl start httpd
             sudo amazon-linux-extras enable php7.4
             sudo yum clean metadata
             sudo yum install php php-common php-pear -y
             sudo yum install php-{cgi,curl,mbstring,gd,mysqlnd,gettext,json,xml,fpm,intl,zip} -y
             sudo rpm -Uvh https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
             sudo yum install mysql-community-server -y
             sudo systemctl enable mysqld
             sudo systemctl start mysqld
             echo "<Enter The EFS Credential here> /var/www/html nfs4  nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 0 0" >> /etc/fstab
             mount -a
             service httpd restart
~~~

- Click **Next**, Leave everything as default
- Click **Next**, Give it a **name**;
- Click **Next**
- Click **Select an existing security group**
- Select the “Web-Server security group”
- Click **Next**
- Click **Launch**
- Check the agreement and Click **Launch instance**
- Repeat the same process in the second Availability Zone


### Stage 2– Provision the Application Load Balancer 
- Navigate to the EC2 dashboard, Select Load Balancers
- Click **Create Load Balancer**
- Select an **Application Load Balancer**
- Give it a **name** 
- Leave the **Schema** as “Internet facing”
- For **VPC** Select “WordPress VPC”
- For **Availability Zone** Select “both the Availability zones” being used and from the dropdown menu Select **Public Subnet1** and **Public Subnet2** respectively
- Click **Next**
- Click **Select an existing security group**
- Uncheck the **default** and Select **ALB Security Group**
- Click **Next**
- For **Target Group**, Give it a **name**; “AppServer”
- Leave **Target Type** as “Instance”
- Leave everything as default
- Click **Next**
- Under **Register Targets**, Select both EC2 instances “Server1A and Server1B”, Click **Add to registered**
- Click **Next**
- Click **Create**
- Once the ALB is provisioned, Copy the DNS name and open it in the browser to access the WordPress site
Add **“/wp-admin”** to the end of the DNS name, this opens the WordPress login page
Once logged in Select **Setting → General**
Copy the **DNS name** and paste it to the **WordPress Address (URL)** and **Site Address (URL)**
Click **Save Changes**


# Launch The WordPress Site
### Stage 1 - Register a Domain Name in Route53
- Navigate to the AWS services dashboard, Select “Route 53”
- Under **Register domain**, Type the desired Domain name and Click **Check**
- Click **Continue**
- Enter personal information
- Click **Continue**
- Click **Close** 


### Stage 2 - Create a Record Set in Route53
- Navigate to the AWS services dashboard, Select “Route53”
- Under Route53 Dashboard, Click **Hosted zone**
- Select the **registered domain name**
- Click **Create record**
- For **Routing policy**, Select “Simple routing” from the drop down menu
- For **Record name** type “www”
Toggle On “Alias”
- For **Route traffic to**, Select “Alias to Application Load Balancer”
- For **Region**, Select the region where the ALB is from the drop down menu
- Select the ALB created for this project 
- Click **Create records**
- Copy the newly created URL and open it in the browser to access the WordPress site
 Add **“www.”** and **“/wp-admin”** to the beginning and end of the URL respectively, this opens the WordPress login page
- Once logged in Select **Setting → General**
- Copy the **URL** and Paste it to the **WordPress Address (URL)** and **Site Address (URL)**
- Click **Save Changes**

### Stage 3 - Register for an SSL Certificate in AWS Certificate Manager
- Navigate to the AWS services dashboard, Select “Certificate Manager”
- Under Certificate Manager Dashboard, Click **Request certificate**
- Select the **request a public certificate** and Click "Next"
- For the **Domain names**, type the domain name the certificate is requested for
- For "Add another name to this certificate", type "*."
- Leave everything as default and Click "Request"
- Select **Create records in Route53**
- Select the domain name and the wild card and Click "Create records"
- Refresh the page and check if the **status** of the request is "issued"


### Stage 4 - Create an HTTPS Listener
- Navigate to the AWS services dashboard, Select “EC2"
- Under EC2 Dashboard, Click **Load Balancers**
- Click **Listeners** and Select "Add listener"
- For the **Protocol** change it to "HTTPS"
- From the **Default actions** dropdown menu, Select "Forward"
- From the *Target group** dropdown menu, Select the target group created for this project
- Under *Default SSL certificate**, Select the ACM certificate created for this project
- Leave everything else as default and click "Add"
### Stage 5 - Edit HTTP Listener to Redirect Traffic to HTTPS
- On the **Load balancers** dashboard, Select "HTTP:80" and Click "Edit"
- Under the **Default actions** dropdown menu, Select "Redirect"
- For the **port** enter "443"
- Click **Save Changes**
- Modify the wp-config file
- **SSH** to the private server in the private subnet and write the following commands,
~~~
sudo su
nano /var/www/html/wp-config.php

~~~
- Enter the following code in the nano text editor
~~~
/* SSL Settings */
define('FORCE_SSL_ADMIN', true);

// Get true SSL status from AWS load balancer
if(isset($_SERVER['HTTP_X_FORWARDED_PROTO']) && $_SERVER['HTTP_X_FORWARDED_PROTO'] === 'https') {
  $_SERVER['HTTPS'] = '1';
}
~~~
- Open the HTTPS domain name in the browser to access the WordPress site
- Add **“/wp-admin”** to the end of the URL, this opens the WordPress login page
- Once logged in Select **Setting → General**
- Copy the **URL** and Paste it to the **WordPress Address (URL)** and **Site Address (URL)**
- Click **Save Changes**

### Stage 6 - Install WordPress Theme and Template
- log in to the WordPress site as the admin
- Select **Appearance**
- Under *Themes**, Click "Add New"
- Install the "Astra Theme"
- Click "Activate"
- Click **Get Started** to install the template
- Click "Build Your Website"
- Select "Elementor"
- Search for the desired template for your website





















