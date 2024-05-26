<h1>Deploying a WordPress Website on AWS</h1>

I have created a dynamic website using AWS services. The purpose behind this project is to build on what I have learned from the AWS Certified Cloud Practitioner certification and flesh out concepts I will be familiar with for future AWS projects and certification exams. I used a variety of different AWS services to build the website, and I took the website down after building it to save money. I spent about $30 to do this project over the span of a few days. I am using my Windows 10 computer in order to do this project and am using PuTTY to get familiar with using the program. 

_<b>NOTE:</b> This project is done exclusively in the N. Virginia region (us-east-1) in order to ensure all services are available. Not all services are available in multiple regions. If you want to replicate this project, research the services that are used and see if they are all available in a specific region._

<h2>Services and Technologies Used</h2>

- AWS
- PuTTY
- VPC (Public and Private subnets)
- Security groups
- EC2 instances
- Auto Scaling group
- NAT gateways
- RDS
- Application load balancer
- Route 53
- Certificate Manager
- EFS

<h2>High-Level Deployment and Configuration Steps</h2>

- Create a VPC
- Setup NAT gateways
- Create security groups
- Launch an RDS instance
- Enable EFS
- Create a key pair
- Launch a server
- Access public subnet instance with SSH
- Install WordPress
- Create an Application Load Balancer
- Claim a domain name on Route 53
- Create route records on Route 53
- Obtain an SSL certificate
- Launch a bastion host
- SSH into private subnets
- Create an HTTPS listener
- Create an Auto Scaling group

<h2>Step Process</h2>

<h3>&#9312; Create a VPC</h3>

<p align="center">
<img src="https://i.imgur.com/Tqq0xAr.jpg" height="80%" width="80%" alt="VPC Reference"/>
</p>

- A three-tier VPC will serve as the architecture for the project. The first tier will have the public subnets. The public subnets will host resources such as NAT gateways, an application load balancer, and eventually a bastion host. The second tier will host a private subnet. The web servers (EC2 instances) will be hosted there. The third tier will have another private subnet which will host the database necessary to complete the project. The subnets will be duplicated across multiple availability zones to increase fault tolerance and high availability. An internet gateway and route table will also be created to allow resources in the VPC to access the internet.

- The VPC will be created in the <b>N. Virginia region</b>. From the AWS Management console, navigate to the <b>VPC</b> service. In the VPCs menu, click <b>Create VPC</b>.
  - Give a name to the VPC <b>(Dev VPC)</b> and enter the IPv4 CIDR block <b>(10.0.0.0/16)</b>. Leave the rest of the settings as default and click <b>Create VPC</b>.

<p align="center">
<img src="https://i.imgur.com/4bpt43d.png" height="80%" width="80%" alt="Step 1-1"/>
</p>

- Next, DNS host names have to be enabled for the VPC that was created. Under <b>Actions</b>, select <b>Edit VPC settings</b>. Under <b>DNS settings</b>, make sure <b>Enable DNS resolution</b> and <b>Enable DNS hostnames</b> are checked and save the changes.

<p align="center">
<img src="https://i.imgur.com/RXp9haj.png" height="80%" width="80%" alt="Step 1-2"/>
</p>

<p align="center">
<img src="https://i.imgur.com/nnqQFcZ.png" height="80%" width="80%" alt="Step 1-3"/>
</p>

- An internet gateway will now be created for the VPC. On the left-hand menu, select <b>Internet Gateways</b>. Click <b>Create internet gateway</b>.
  - Give a name for the internet gateway <b>(Dev Internet Gateway)</b> and create it.

<p align="center">
<img src="https://i.imgur.com/P984xtj.png" height="80%" width="80%" alt="Step 1-4"/>
</p>

- After creating the internet gateway, it will have to be attached to the VPC. This is to ensure the VPC can communicate with the internet. There will be an option that says to <b>Attach to a VPC</b> after the internet gateway has been created.
  - One thing to note is that you can only attach one internet gateway to one VPC. When you go to attach an internet gateway to a VPC on AWS, you can only select VPCs that do not have internet gateways.

<p align="center">
<img src="https://i.imgur.com/VaRicio.png" height="80%" width="80%" alt="Step 1-5"/>
</p>

- Now that the internet gateway is attached to the VPC, public subnets will be created in two availability zones <b>(us-east-1a and us-east-1b)</b>.
  - Select the <b>Subnets</b> tab on the left-hand menu. Click <b>Create subnet</b>. When creating your public subnets, make sure the <b>Dev VPC</b> is selected. For the first public subnet, name it <b>Public Subnet AZ1</b> and make sure it is in the <b>us-east-1a</b> availability zone. Its IPv4 CIDR block should be <b>10.0.0.0/24</b>. For the second public subnet, name it <b>Public Subnet AZ2</b> and make sure it is in the <b>us-east-1b</b> availability zone. Its IPv4 CIDR block should be <b>10.0.1.0/24</b>.

<p align="center">
<img src="https://i.imgur.com/1QhrXhb.png" height="80%" width="80%" alt="Step 1-6"/>
</p>

<p align="center">
<img src="https://i.imgur.com/toddnWF.png" height="80%" width="80%" alt="Step 1-7"/>
</p>

- After the public subnets are created, the auto enable IP settings need to be enabled for both subnets. This means when an EC2 instance is launched in the subnets, the instances will be assigned an appropriate public IP address in order to communicate with the internet.
  - For each subnet, select them and click on <b>Edit subnet settings</b>. Make sure <b>Enable auto-assign public IPv4 address</b> is turned on for both subnets and save the changes.

<p align="center">
<img src="https://i.imgur.com/YJbkxaN.png" height="80%" width="80%" alt="Step 1-8"/>
</p>

<p align="center">
<img src="https://i.imgur.com/TxhpCUJ.png" height="80%" width="80%" alt="Step 1-9"/>
</p>

- A public route table will now be created.
  - Select the <b>Route Tables</b> tab on the left-hand menu. A route table was already created when the VPC was made. This is referred to as the main route table and is private by default. Click <b>Create route table</b> and name the new route table <b>Public Route Table</b>. It will be attached to the Dev VPC.
 
<p align="center">
<img src="https://i.imgur.com/s1gIgpk.png" height="80%" width="80%" alt="Step 1-10"/>
</p>

- A public route will be added to the route table that was made. This public route will route traffic to the internet.
  - Under the <b>Routes</b> tab for the Public Route Table, click <b>Edit Routes</b>. Add a new route where the <b>Destination</b> is <b>0.0.0.0/0</b> (this means all traffic) and the <b>Target</b> is the <b>Dev Internet Gateway</b>. Save the changes.

<p align="center">
<img src="https://i.imgur.com/5Nt9aoP.png" height="80%" width="80%" alt="Step 1-11"/>
</p>

<p align="center">
<img src="https://i.imgur.com/wuOursD.png" height="80%" width="80%" alt="Step 1-12"/>
</p>

- The next thing that needs to be done is to associate the public subnets with the public route table.
  - While under the menu for Public Route Table, open the <b>Subnet associations</b> tab and scroll to <b>Explicit subnet associations</b>. Click on <b>Edit subnet associations</b>. Select both public subnets and save the associations.

 <p align="center">
<img src="https://i.imgur.com/0csGYLF.png" height="80%" width="80%" alt="Step 1-13"/>
</p>

 <p align="center">
<img src="https://i.imgur.com/0zVDZug.png" height="80%" width="80%" alt="Step 1-14"/>
</p>

- In order to finish creating the VPC, the four private subnets need to be created.
  - On the left-hand menu, click on Subnets and create the private subnets for the VPC. When creating your private subnets, make sure the Dev VPC is selected. For the first private subnet, name it <b>Private App Subnet AZ1</b> and make sure it is in the <b>us-east-1a</b> availability zone. Its IPv4 CIDR block should be <b>10.0.2.0/24</b>. For the second private subnet, name it <b>Private App Subnet AZ2</b> and make sure it is in the <b>us-east-1b</b> availability zone. Its IPv4 CIDR block should be <b>10.0.3.0/24</b>. For the third private subnet, name it <b>Private Data Subnet AZ1</b> and make sure it is in the <b>us-east-1a</b> availability zone. Its IPv4 CIDR block should be <b>10.0.4.0/24</b>. For the fourth private subnet, name it <b>Private Data Subnet AZ2</b> and make sure it is in the <b>us-east-1b</b> availability zone. Its IPv4 CIDR block should be <b>10.0.5.0/24</b>.

<p align="center">
<img src="https://i.imgur.com/t5sHdIT.png" height="80%" width="80%" alt="Step 1-15"/>
</p>

<p align="center">
<img src="https://i.imgur.com/Frc068s.png" height="80%" width="80%" alt="Step 1-16"/>
</p>

<p align="center">
<img src="https://i.imgur.com/6UgkdLh.png" height="80%" width="80%" alt="Step 1-17"/>
</p>

<p align="center">
<img src="https://i.imgur.com/3wbbJrt.png" height="80%" width="80%" alt="Step 1-18"/>
</p>

- Before you continue, make sure all 6 subnets are in the correct Availability Zones. The project will rely heavily on all the subnets and all resources and data will flow across the VPC.

_<b>NOTE:</b> When you create a route to a route table, all the subnets associated within the route table will automatically become public. Subnets are private when the route table does not have a route. In the Route Tables tab, check each route table to confirm each subnet are where they belong as shown below. The private subnets should be in the main route table that was automatically created when the VPC was first made. This is because these subnets do not have explicit associations, unlike the public subnets. The main route table is routing traffic locally within the VPC._

<p align="center">
<img src="https://i.imgur.com/8wTlXJy.png" height="80%" width="80%" alt="Step 1-19"/>
</p>

<h3>&#9313; Create NAT gateways</h3>

<p align="center">
<img src="https://i.imgur.com/kFiYDfb.jpg" height="80%" width="80%" alt="Step 2"/>
</p>

- Two NAT gateways will be created within the first and second Availability Zones. One will be in Public Subnet AZ1 and will be tied to a new private route table via a route that will connect the two together. The route table will also be associated with the Private App Subnet AZ1 and Private Data Subnet AZ1 subnets within the VPC. The second NAT gateway wil be created in Public Subnet AZ1 and tied to a new private route table with a route. The second route table will be associated with the Private App Subnet AZ2 and Private Data Subnet AZ2 subnets within the VPC.
- On the AWS management console, navigate to the <b>VPC</b> service. Select <b>NAT Gateways</b> on the VPC Dashboard. Create the first NAT gateway in <b>Public Subnet AZ1</b>. Name it <b>NAT Gateway AZ1</b>. Make sure to click <b>Allocate Elastic IP</b> before creating the NAT gateway.

<p align="center">
<img src="https://i.imgur.com/xy6mj0E.png" height="80%" width="80%" alt="Step 2-1"/>
</p>

- Now that the NAT gateway is created, a private route table and the appropriate route will be created so there will be access to the internet. Call this new route table <b>Private Route Table AZ1</b> and put it in the Dev VPC. For the route, make sure the <b>Destination</b> is <b>0.0.0.0/0</b> and the <b>Target</b> is <b>NAT Gateway AZ1</b>.

<p align="center">
<img src="https://i.imgur.com/ZB8sq4W.png" height="80%" width="80%" alt="Step 2-2"/>
</p>

<p align="center">
<img src="https://i.imgur.com/GsrBCwU.png" height="80%" width="80%" alt="Step 2-3"/>
</p>

- The next step is to associate the route table with Private App Subnet AZ1 and Private Data Subnet AZ1. In Private Route Table AZ1, open the <b>Subnet associations</b> tab and click on <b>Edit subnet associations</b>. Select <b>Private App Subnet AZ1</b> and <b>Private Data Subnet AZ1</b> and save the associations.

<p align="center">
<img src="https://i.imgur.com/VNPTmid.png" height="80%" width="80%" alt="Step 2-4"/>
</p>

- Repeat the previous steps in order to create a NAT gateway in Public Subnet AZ2.
  - Name the second NAT gateway <b>NAT Gateway AZ2</b>.
  - Name the second route table <b>Private Route Table AZ2</b> and put it in the Dev VPC.
  - Add a route where the <b>Destination</b> is <b>0.0.0.0/0</b> and the <b>Target</b> is <b>NAT Gateway AZ2</b>.
  - Associate the route table with <b>Private App Subnet AZ2</b> and <b>Private Data Subnet AZ2</b>.

<h3>&#9314; Create Security Groups</h3>

<p align="center">
<img src="https://i.imgur.com/yw8HU3r.jpg" height="80%" width="80%" alt="Step 3"/>
</p>

- The above image details all the security groups that need to be created to continue with the project. The application load balancer will have a security group to allow internet traffic (HTTP and HTTPS). One security group will be dedicated to allow SSH access to EC2 instances using your IP address. (Any time an SSH security group is created, it is always best practice to limit the source to your IP address for safety.) A security group will be created for web servers in the Private App Subnets. The sources for this security group will be limited to the ALB and SSH security groups respectively. A security group will be created for the RDS database that will be hosted on the Private Data Subnets and the source will be from the Webserver security group. An EFS security group will be made for elastic file system and use previous security groups for the sources.
- On the AWS management console, navigate to the <b>VPC</b> service. On the VPC Dashboard, open the <b>Security Groups</b> tab. The first security group that will be created is the <b>ALB Security Group</b>. Click on <b>Create security group</b> to get started. Make sure the security group is in the Dev VPC. For Inbound rules, there will be two rules that will be added. For the <b>Type</b>, select <b>HTTP and HTTPS</b>. The <b>Sources</b> will come from <b>Anywhere</b>. To have this setting, input the CIDR block <b>0.0.0.0/0</b>. Click <b>Create security group</b> to confirm the settings.

<p align="center">
<img src="https://i.imgur.com/RHjr9gP.png" height="80%" width="80%" alt="Step 3-1"/>
</p>

<p align="center">
<img src="https://i.imgur.com/Bafkoaa.png" height="80%" width="80%" alt="Step 3-2"/>
</p>

- Create the rest of the security groups with the following settings:
  - <b>SSH Security Group</b> - VPC: Dev VPC, Inbound rules: SSH, Source: My IP
  - <b>Webserver Security Group</b> - VPC: Dev VPC, Inbound rules: HTTP, Source: ALB Security Group, Inbound rules: HTTPS, Source: ALB Security Group, Inbound rules: SSH, Source: SSH Security Group.
  - <b>Database Security Group</b> - VPC: Dev VPC, Inbound rules: MySQL/Aurora, Source: Webserver Security Group.
  - <b>EFS Security Group</b> - VPC: Dev VPC, Inbound rules: NFS, Source: Webserver Security Group, Inbound rules: SSH, Source: SSH Security Group.
- After the EFS Security Group is created, click on <b>Edit inbound rules</b> to add one more important rule:
  - Add an additional NFS rule where the source is from the EFS Security Group. This rule could not be added unless the security group was already created.

<p align="center">
<img src="https://i.imgur.com/LF15HvK.png" height="80%" width="80%" alt="Step 3-3"/>
</p>

<h3>&#9315; Create the RDS Instance</h3>

<p align="center">
<img src="https://i.imgur.com/mx6xtMG.jpg" height="80%" width="80%" alt="Step 4"/>
</p>

- The next step is to create a RDS database in the Private Data Subnets. On the AWS management console, navigate to the <b>RDS</b> service to get started. Before creating the RDS instance, subnet groups need to be created. They specify which subnets the RDS database will be created in. Select <b>Subnet groups</b> on the RDS Dashboard and click <b>Create DB subnet group</b>.
  - Name the group <b>database subnets</b> and place it in the Dev VPC. Under the <b>Add subnets</b> section, select the <b>us-east-1a</b> and <b>us-east-1b</b> Availability Zones. For <b>Subnets</b>, select the subnets with the CIDR blocks <b>10.0.4.0/24</b> and <b>10.0.5.0/24</b>. Click <b>Create</b> to make the subnet group.

<p align="center">
<img src="https://i.imgur.com/3N0vEt9.png" height="80%" width="80%" alt="Step 4-1"/>
</p>

- Now that the subnet group is created, it is time to make the database itself. Click on <b>Databases</b> on the left-hand menu and click on <b>Create database</b>. Use the following parameters to create the database:
  - <b>Creation method</b>: Standard create
  - <b>Engine options</b>: MySQL
  - <b>Engine Version</b>: MySQL 5.7 (The latest version of 5.7 as in the future, more updated versions will be released beyond when I created the website.)
  - <b>Templates</b>: Dev/Test
  - <b>DB instance identifier</b>: dev-rds-db
  - <b>Master username</b>: (Whatever you choose, in my case it is ernesto.)
  - <b>Master password</b>: (Whatever you choose, in my case it is Password1. Make sure you remember this password as there will be no way to retrieve it afterward.)
  - <b>DB instance class</b>: Burstable classes (db.t2.micro)
  - <b>VPC</b>: Dev VPC
  - <b>Subnet group</b>: database subnets
  - <b>VPC security group</b>: Choose existing (Database Security Group)
  - <b>Availability Zone</b>: us-east-1b
  - <b>Database authentication</b>: Password authentication
  - <b>Initial database name</b>: applicationdb (Make sure you expand Additional configuration to see this parameter, you must specify a name or else RDS will not make the database.)
- After the database is created (it will take a few minutes for AWS to create), click on the database indentifier name. Under the <b>Connectivity & security</b> tab, take note of the <b>Endpoint</b> of the database. This information will be used later when connecting to the database using an EC2 instance. Under the <b>Configuration</b> tab, take note of the <b>DB instance ID</b> and <b>DB name</b> as they will also be used to connect to the database.

<p align="center">
<img src="https://i.imgur.com/PGn58sg.png" height="80%" width="80%" alt="Step 4-2"/>
</p>

<p align="center">
<img src="https://i.imgur.com/IIeUG0w.png" height="80%" width="80%" alt="Step 4-3"/>
</p>

<h3>&#9316; Create the Elastic File System (EFS)</h3>

- Now that the RDS database is in place, it is time to create an EFS file system with mount targets in the Private Data Subnets in both Availability Zones. This is to ensure the web servers can have access to shared files.
- On the AWS management console, navigate to the <b>EFS</b> service and click <b>Create file system</b> and <b>Customize</b>. Use the following parameters to create the file system:
  - <b>Name</b>: Dev-EFS
  - <b>Encryption</b>: Check off Enable encryption of data at rest (This is to ensure we do not get charged for the encryption.)
  - <b>Tag key</b>: Name, Tag value: Dev-EFS
  - <b>VPC</b>: Dev VPC
  - <b>Mount targets</b>: us-east-1a, Private Data Subnet AZ1, EFS Security Group and us-east-1b, Private Data Subnet AZ2, EFS Security Group
  - <b>File system policy</b>: Leave everything as default

<p align="center">
<img src="https://i.imgur.com/8gXgWA4.png" height="80%" width="80%" alt="Step 5-1"/>
</p>

<p align="center">
<img src="https://i.imgur.com/T6798l6.png" height="80%" width="80%" alt="Step 5-2"/>
</p>

- Now that the elastic file system is created, click on the File system ID and click on <b>Attach</b>. _This information will be used later in the project to mount the file system._

<p align="center">
<img src="https://i.imgur.com/9XEGzAk.png" height="80%" width="80%" alt="Step 5-3"/>
</p>

<p align="center">
<img src="https://i.imgur.com/2ISmlXF.png" height="80%" width="80%" alt="Step 5-4"/>
</p>

<h3>&#9317; Create a Key Pair</h3>

- A key pair will now have to be created in order to progress further with the project. On the AWS management console, navigate to the <b>EC2</b> service. On the left-hand menu, click on <b>Key Pairs</b> and click <b>Create key pair</b>.
  - Name the key pair <b>(myec2key)</b> and make sure the Key pair type is <b>RSA</b>. The file format will be kept as .ppk because I will be using the key pair for use with PuTTY.

<p align="center">
<img src="https://i.imgur.com/NHsrLTe.png" height="80%" width="80%" alt="Step 6-1"/>
</p>

<p align="center">
<img src="https://i.imgur.com/iXgObty.png" height="80%" width="80%" alt="Step 6-2"/>
</p>

- When a key pair is made, two keys are generated: a public key and a private key. The key on the AWS console is the public key and it will be used in the EC2 instance when it is launched. The key that is downloaded on the computer is the private key and it will be used whenever SSH is used to access an instance.

<h3>&#9318; Launching a Setup Server</h3>

- An EC2 instance will be launched in Public Subnet AZ1 in order to install the website and move files to the EFS. On the AWS management console, navigate to the <b>EC2</b> service and select <b>Instances (running)</b>. Click on <b>Launch instances</b> to get started. Use the following parameters for the instance:
  - <b>Name</b>: Setup Server
  - <b>Application and OS Images</b>: Amazon Linux
  - <b>AMI</b>: Amazon Linux 2 AMI (Free tier eligible)
  - <b>Instance type</b>: t2.micro
  - <b>Key pair (login)</b>: myec2key
  - <b>VPC</b>: Dev VPC
  - <b>Subnet</b>: Public Subnet AZ1
  - <b>Firewall (security groups)</b>: SSH Security Group, ALB Security Group, Webserver Security Group

<p align="center">
<img src="https://i.imgur.com/0ZSMbeb.png" height="80%" width="80%" alt="Step 7-1"/>
</p>

<p align="center">
<img src="https://i.imgur.com/X87q45d.png" height="80%" width="80%" alt="Step 7-2"/>
</p>

<p align="center">
<img src="https://i.imgur.com/ut8LC58.png" height="80%" width="80%" alt="Step 7-3"/>
</p>

<h3>&#9319; Accessing the Public Subnet EC2 Instance</h3>

- Because I am using a Windows computer, I will be using PuTTY to SSH into the instance that was created. While it is possible to not use PuTTY because I am using a Windows 10 computer, I will still use PuTTY for practice.
- To SSH into the instance, copy the instance's <b>Public IPv4 address</b>. Within the <b>Session</b> tab of PuTTY, enter the <b>Host Name</b> ec2-user@(Public IPv4 address). In the <b>Connection</b> tab, expand <b>SSH</b> and expand <b>Auth</b>. Select <b>Credentials</b> under the <b>Auth</b> tab. Enter the private key that was downloaded to the computer when the key pair was created earlier in the project. After you click <b>Open</b>, you will successfully access the EC2 instance.

<p align="center">
<img src="https://i.imgur.com/P3r8ZZR.png" height="80%" width="80%" alt="Step 8-1"/>
</p>

<p align="center">
<img src="https://i.imgur.com/0UaATYQ.png" height="80%" width="80%" alt="Step 8-2"/>
</p>

<p align="center">
<img src="https://i.imgur.com/a7V4UkB.png" height="80%" width="80%" alt="Step 8-3"/>
</p>

<h3>&#9320; Installing WordPress</h3>

- Once the EC2 instance has been accessed through SSH, commands will have to be run in order to install the WordPress website. Before continuing, make sure that the relevant EFS mount data has been copied from a previous step in the project. In the EFS that was created earlier, the <b>Attach</b> menu will show the code that is necessary to mount the EFS. Make sure to copy the highlighted section in the image below.

<p align="center">
<img src="https://i.imgur.com/snqtoNi.png" height="80%" width="80%" alt="Step 9-1"/>
</p>

- Within the PuTTY session, run the following commands (and make sure to place the EFS code where specified and remove the parentheses around it):
  - sudo su
  - yum update -y
  - mkdir -p /var/www/html
  - sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport (EFS code):/ /var/www/html

- Now that the EFS has been mounted, Apache will have to be installed. Run the following commands:
  - sudo yum install -y httpd httpd-tools mod_ssl
  - sudo systemctl enable httpd
  - sudo systemctl start httpd

- Next, PHP 7.4 will be installed with the following commands:
  - sudo amazon-linux-extras enable php7.4
  - sudo yum clean metadata
  - sudo yum install php php-common php-pear -y
  - sudo yum install php-{cgi,curl,mbstring,gd,mysqlnd,gettext,json,xml,fpm,intl,zip} -y

- MySQL 5.7 will be installed with these commands:
  - sudo rpm -Uvh https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
  - sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
  - sudo yum install mysql-community-server -y
  - sudo systemctl enable mysqld
  - sudo systemctl start mysqld

- Some web files will need to have their permissions changed. Run these commands to set the permissions:
  - sudo usermod -a -G apache ec2-user
  - sudo chown -R ec2-user:apache /var/www
  - sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
  - sudo find /var/www -type f -exec sudo chmod 0664 {} \;
  - chown apache:apache -R /var/www/html

- The WordPress files will now be downloaded and moved to the html directory with the following commands:
  - wget https://wordpress.org/latest.tar.gz
  - tar -xzf latest.tar.gz
  - cp -r wordpress/* /var/www/html/
 
- A WordPress configuration file will have to be created and modified. Run these commands:
  - cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php
  - nano /var/www/html/wp-config.php

<p align="center">
<img src="https://i.imgur.com/oWHtG8G.png" height="80%" width="80%" alt="Step 9-2"/>
</p>

- Within the text editor for the configuration file, some information needs to be inserted from the RDS instance that was created earlier in the project. Go to the <b>RDS</b> console from AWS to get this information. In the database that was created, open the <b>Configuration</b> tab to get the necessary information.
  - Copy the <b>DB name</b> from the <b>Configuration</b> tab and replace it where database_name_here is.

_<b>NOTE:</b> Make sure to copy the DB name and <b>NOT</b> the DB instance ID. They refer to different things and are not the same. Make sure what you are copying is the DB name. Refer to the image below. The Database instance ID is highlighted here. DB name is located underneath it._

<p align="center">
<img src="https://i.imgur.com/ikK6jvP.png" height="80%" width="80%" alt="Step 9-3"/>
</p>

- The next things to change in the file are the username and password for the RDS database. Enter the master username and password for the database when it was created. Replace username_here and password_here respectively.
- The next thing to change is the database hostname in the file. The database hostname will be the endpoint of the RDS instance. Return to the RDS console and open the <b>Connectivity & security</b> tab. Copy the endpoint and replace localhost within the configuration file.

<p align="center">
<img src="https://i.imgur.com/SzI29kR.png" height="80%" width="80%" alt="Step 9-4"/>
</p>

- Now that the necessary information is inserted in the configuration file, the EC2 instance will now be able to connect to the RDS instance. Save all the changes and run the last command to restart the Apache web server:
  - service httpd restart
- Return to the EC2 console and copy the <b>Public IPv4 address</b> of the Setup Server. Open a new tab in the web browser and paste the IPv4 address. When everything has been configured correctly, a WordPress welcome page will be shown. Enter the necessary information to create the admin account and website. The Setup Server cannot be deleted yet as the next step is to create the application load balancer.

<p align="center">
<img src="https://i.imgur.com/TFawYpa.png" height="80%" width="80%" alt="Step 9-5"/>
</p>

<p align="center">
<img src="https://i.imgur.com/xW5Phri.png" height="80%" width="80%" alt="Step 9-6"/>
</p>

<h3>&#9321; Create the Application Load Balancer</h3>

- An application load balancer will be created to distribute web traffic across EC2 instances in the Private App Subnets in the VPC. Before creating the application load balancer, new EC2 instances will be launched in the Private App Subnets. Navigate to the <b>EC2</b> service to get started. Launch an instance with the following configurations:
  - <b>Name and Tags</b>: Name, Webserver AZ1
  - <b>Application and OS Images</b>: Amazon Linux 2 AMI (free tier eligible)
  - <b>Instance type</b>: t2.micro
  - <b>Key pair</b>: myec2key (the key pair that you created earlier)
  - <b>VPC</b>: Dev VPC
  - <b>Subnet</b>: Private App Subnet AZ1
  - <b>Firewall (security groups)</b>: Web Server Security Group
- For the user data, some commands will be pasted in. This means that the commands will be run whenever the instance is booting up. Before pasting the commands in the user data, return to the <b>EFS</b> console and obtain the mount data that was previously used to install WordPress earlier in the project.

<p align="center">
<img src="https://i.imgur.com/mnUdGeu.png" height="80%" width="80%" alt="Step 10-1"/>
</p>

- Paste the following script into the user data section of the EC2 instance creation menu (and replace the EFS data where specified):
  - #!/bin/bash
  - yum update -y
  - sudo yum install -y httpd httpd-tools mod_ssl
  - sudo systemctl enable httpd
  - sudo systemctl start httpd
  - sudo amazon-linux-extras enable php7.4
  - sudo yum clean metadata
  - sudo yum install php php-common php-pear -y
  - sudo yum install php-{cgi,curl,mbstring,gd,mysqlnd,gettext,json,xml,fpm,intl,zip} -y
  - sudo rpm -Uvh https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
  - sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
  - sudo yum install mysql-community-server -y
  - sudo systemctl enable mysqld
  - sudo systemctl start mysqld
  - echo "(EFS data):/ /var/www/html nfs4 nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 0 0" >> /etc/fstab
  - mount -a
  - chown apache:apache -R /var/www/html
  - sudo service httpd restart
 
<p align="center">
<img src="https://i.imgur.com/rWC269n.png" height="80%" width="80%" alt="Step 10-2"/>
</p>

- Launch a second EC2 instance while the first one is being made and use the following configurations:
  - <b>Name and Tags</b>: Key - Name, Value - Webserver AZ2
  - <b>Application and OS Images</b>: Amazon Linux 2 AMI (free tier eligible)
  - <b>Instance type</b>: t2.micro
  - <b>Key pair</b>: myec2key (the key pair that you created earlier)
  - <b>VPC</b>: Dev VPC
  - <b>Subnet</b>: Private App Subnet AZ2
  - <b>Firewall (security groups)</b>: Web Server Security Group
  - <b>User data</b>: the same user data script that was used in the first instance

- After creating the two EC2 instances, the next step is to create the target group and put the instances in the target group to allow the application load balancer to route traffic to them. On the left-hand menu, open the <b>Target Groups</b> tab and click on <b>Create target group</b>. Use the following configurations to make the target group:
  - <b>Target type</b>: Instances
  - <b>Name</b>: Dev-TG
  - <b>Protocol</b>: HTTP
  - <b>VPC</b>: Dev VPC
  - <b>Advanced health check settings - Success codes</b>: 200,301,302
  - <b>Register targets</b>: Webserver AZ1 and Webserver AZ2 (click on Include as pending below to confirm the choices)

<p align="center">
<img src="https://i.imgur.com/NtCmQyg.png" height="80%" width="80%" alt="Step 10-3"/>
</p>

- The next step is to create the application load balancer. Select <b>Load Balancers</b> on the left-hand menu and click on <b>Create load balancer</b>. Use these configurations to create the application load balancer:
  - <b>Load balancer name</b>: Dev-ALB
  - <b>Scheme</b>: Internet-facing
  - <b>IP address type</b>: IPv4
  - <b>VPC</b>: Dev VPC
  - <b>Mappings</b>: us-east-1a - Public Subnet AZ, us-east-1b - Public Subnet AZ2
  - <b>Security groups</b>: ALB Security Group
  - <b>Listener HTTP 80 Default Action</b>: Forward to Dev-TG

- After the application load balancer is active, copy the DNS name and paste it in a new browser tab. The website can now be accessed using the DNS name of the application load balancer.

<p align="center">
<img src="https://i.imgur.com/D2plyij.png" height="80%" width="80%" alt="Step 10-4"/>
</p>

<p align="center">
<img src="https://i.imgur.com/vC2fNyf.png" height="80%" width="80%" alt="Step 10-5"/>
</p>

- Any time the address is changed, it is necessary to go into the WordPress settings as an admin and change the domain address there. Before accessing the settings, copy the domain name of the application load balancer. After the domain name, type /wp-admin and press Enter. You will be prompted to log in as the admin using the WordPress crendentials when the website was first made. Click on <b>Settings</b> and paste the domain address in the <b>WordPress Address</b> and <b>Site Address</b> boxes (remove the / at the end of the address if it is retained).

<p align="center">
<img src="https://i.imgur.com/NSlCbst.png" height="80%" width="80%" alt="Step 10-6"/>
</p>

<p align="center">
<img src="https://i.imgur.com/p3LzW2V.png" height="80%" width="80%" alt="Step 10-7"/>
</p>

- Now that the instances are launched in the Private App Subnets and the website can be accessed via the DNS name of the application load balancer, there is no need to have the Setup Server running. Terminate the Setup Server on the EC2 console.

<p align="center">
<img src="https://i.imgur.com/rVWN8te.png" height="80%" width="80%" alt="Step 10-8"/>
</p>

<h3>&#9322; Register a Domain Name</h3>

- A domain name will be registered with Route 53 to be used as the url for the WordPress website. This domain name will be used instead of the DNS name of the application load balancer. Navigate to the <b>Route 53</b> service on AWS to get started. Click on <b>Registered domains</b> to get started.
  - I am registering ernestoawswebsitelab.com for the purposes of the project. It will cost $13 to register the domain name. Enter the contact information to complete the transaction and make sure privacy protection is enabled. Give at least 15 minutes for the domain name to be registered. It may take longer for the registration to go through, just be patient.
 
_<b>NOTE:</b> This project can use any domain that you own, even ones that are regsitered with other providers such as GoDaddy. Route 53 is used for the purposes of the project._
 
<p align="center">
<img src="https://i.imgur.com/axFpFkN.png" height="80%" width="80%" alt="Step 11-1"/>
</p>

<p align="center">
<img src="https://i.imgur.com/yaxmGAz.png" height="80%" width="80%" alt="Step 11-2"/>
</p>

<p align="center">
<img src="https://i.imgur.com/2pBLEFk.png" height="80%" width="80%" alt="Step 11-3"/>
</p>

<h3>&#9323; Create a Record Set</h3>

- After getting a registered domain name, a record set will be created in Route 53 to access the website with the domain name. Navigate to the <b>Route 53</b> service and click on <b>Hosted zones</b> to get started. Select the domain name and click on <b>Create record</b>. Use the following configurations to create the record:
  - <b>Record name</b>: www
  - <b>Record type</b>: A
  - Toggle <b>Alias</b> next to Route Traffic to
  - <b>Route Traffic to</b>: Alias to Application and Classic Load Balancer
  - <b>Region</b>: US East (N. Virginia) [us-east-1]
  - <b>Load balancer</b>: The application load balancer created earlier

<p align="center">
<img src="https://i.imgur.com/NNt8evg.png" height="80%" width="80%" alt="Step 12-1"/>
</p>

- Now that the record set is made, the website can now be accessed using the domain name. Select the record that was created and copy the record name. Paste the record name into a new browser tab and the website will be accessed.

<p align="center">
<img src="https://i.imgur.com/RBD6WfA.png" height="80%" width="80%" alt="Step 12-2"/>
</p>

<p align="center">
<img src="https://i.imgur.com/ecG8l6L.png" height="80%" width="80%" alt="Step 12-3"/>
</p>

- Since the domain name has changed once again, it is time to update the WordPress URL settings to reflect this. Repeat the steps from updating the URL name after creating the application load balancer.

<p align="center">
<img src="https://i.imgur.com/IKBY67i.png" height="80%" width="80%" alt="Step 12-4"/>
</p>

<p align="center">
<img src="https://i.imgur.com/okdHD17.png" height="80%" width="80%" alt="Step 12-5"/>
</p>

<p align="center">
<img src="https://i.imgur.com/MfEgxox.png" height="80%" width="80%" alt="Step 12-6"/>
</p>

- The site URL will now be the domain name!

<h3>&#9324; Register an SSL Certificate</h3>

- SSL certificates are necessary to encrypt traffic between the web servers and web browser. This is a concept referred to as encryption in transit. All traffic from the website is currently not secure. The website will now have an appropriate SSL certificate using the Certificate Manager service on AWS. Request a public certificate from <b>Certificate Manager</b> to get started.

<p align="center">
<img src="https://i.imgur.com/Q5qIP9u.png" height="80%" width="80%" alt="Step 13-1"/>
</p>

- For domain names, enter the domain name that you have. Enter a second domain name and include the *. wildcard before the domain name again. Refer to the image below to see how to input the domain names. Make sure to select <b>DNS validation</b> and the <b>RSA 2048</b> key algorithm before requesting the certificate. 

<p align="center">
<img src="https://i.imgur.com/jYpOVNq.png" height="80%" width="80%" alt="Step 13-2"/>
</p>

- When the certificate is pending validation, record sets need to be created in Route 53. This is to validate that the domain name belongs to the rightful owner. Click <b>Create records</b> in Route 53 and select the two domain names (this includes the wildcard that was created) to create the records. Wait a few minutes and refresh the page to see that the certificate has been issued.

<p align="center">
<img src="https://i.imgur.com/Iw6W4Px.png" height="80%" width="80%" alt="Step 13-3"/>
</p>

<p align="center">
<img src="https://i.imgur.com/lcyJQpC.png" height="80%" width="80%" alt="Step 13-4"/>
</p>

<p align="center">
<img src="https://i.imgur.com/08pcLbA.png" height="80%" width="80%" alt="Step 13-5"/>
</p>

<h3>&#9325; Launch a Bastion Host</h3>

- In order to SSH into the instances in the private subnets, an EC2 instance needs to be launched in a public subnet. This instance is called a bastion host. First, the instance in the public subnet needs to be accessed with SSH. From the public subnet instance, SSH into the private subnet. Navigate to the <b>EC2</b> service and create a new instance to get started. Use the following configurations to make the bastion host:
  - <b>Name</b>: Bastion Host
  - <b>Application and OS Images</b>: Amazon Linux
  - <b>Amazon Machine Image</b>: Amazon Linux 2 AMI (free tier eligible)
  - <b>Instance type</b>: t2.micro
  - <b>Key pair</b>: myec2key
  - <b>VPC</b>: Dev VPC
  - <b>Subnet</b>: Public Subnet AZ1
  - <b>Auto-assign Public IP</b>: Enable
  - <b>Firewall (security groups)</b>: SSH Security Group
 
<h3>&#9326; SSH into Private Subnets</h3>

- Now that the bastion host has been created, it is possible to SSH into the private subnets on the VPC. In order to do so, a PuTTY authentication agent known as Pageant needs to be installed. Search for PuTTY and select the search result that has greenend.org in the URL. Scroll down the Alternative binary files list and download and install Pageant from there.

<p align="center">
<img src="https://i.imgur.com/QDLNyEa.png" height="80%" width="80%" alt="Step 15-1"/>
</p>

- When you run Pageant, it will be a hidden icon on the bottom right. Click on its icon to open it. Click<b>Add Key</b> to add the private key on the computer and close the application. Once the key has been added to Pageant, it is possible to SSH into the private subnets.

<p align="center">
<img src="https://i.imgur.com/CD7l7TO.png" height="80%" width="80%" alt="Step 15-2"/>
</p>

- SSH into the bastion host to get started. Copy the bastion host's <b>Public IPv4 address</b> and open PuTTY. For the <b>host name</b>, enter ec2-user@(IPv4 address). Expand the <b>SSH</b> tab and select <b>Auth</b>. Check <b>Allow agent forwarding</b> and click <b>Open</b> to access the bastion host. You will know if you have accessed the bastion host if the IP address on PuTTY matches the <b>Private IPv4 address</b> of the bastion host on the AWS console.

<p align="center">
<img src="https://i.imgur.com/s6X4Mjv.png" height="80%" width="80%" alt="Step 15-3"/>
</p>

<p align="center">
<img src="https://i.imgur.com/L8WFnQR.png" height="80%" width="80%" alt="Step 15-4"/>
</p>

<p align="center">
<img src="https://i.imgur.com/BODeBt4.png" height="80%" width="80%" alt="Step 15-5"/>
</p>

- It is now possible to SSH into a Private App Subnet. On the AWS console, select <b>Webserver AZ1</b> and copy its <b>Private IPv4 address</b>. On PuTTY, enter the following command:
  - ssh ec2-user@(Private IP address)

<p align="center">
<img src="https://i.imgur.com/ZqIRIF8.png" height="80%" width="80%" alt="Step 15-6"/>
</p>

<h3>&#9327; Create an HTTPS Listener</h3>

- The application load balancer will need an HTTPS listener now that the SSL certificate has been issued. It is needed in order to secure the website. Navigate to the <b>Load Balancers</b> tab in the <b>EC2</b> service. Select <b>Dev-ALB</b> and scroll to the <b>Listeners and rules</b> section.

<p align="center">
<img src="https://i.imgur.com/tTRVKCu.png" height="80%" width="80%" alt="Step 16-1"/>
</p>

- Click <b>Add listener</b> and use the following configurations:
  - <b>Protocol</b>: HTTPS
  - <b>Action types</b>: Forward to target groups
  - <b>Target group</b>: Dev-TG
  - <b>Default SSL/TLS certificate</b>: The certificate that you created earlier
 
<p align="center">
<img src="https://i.imgur.com/DVnn84n.png" height="80%" width="80%" alt="Step 16-2"/>
</p>

<p align="center">
<img src="https://i.imgur.com/OWraWri.png" height="80%" width="80%" alt="Step 16-3"/>
</p>

- The HTTPS listener will be edited after it has been created. This will allow HTTP traffic to be redirected to HTTPS. Select the <b>HTTP listener</b> and click <b>Edit listener</b>.

<p align="center">
<img src="https://i.imgur.com/BiRNsbl.png" height="80%" width="80%" alt="Step 16-4"/>
</p>

- Under <b>Default actions</b>, select <b>Redirect to URL</b>. The <b>Protocol</b> should be <b>HTTPS</b>. Save the changes.

<p align="center">
<img src="https://i.imgur.com/JYviuw6.png" height="80%" width="80%" alt="Step 16-5"/>
</p>

- The next step is to SSH into one of the Private App Subnets. <b>Webserver AZ1</b> will accessed via the bastion host.
  - Escalate to root privileges with the command <b>sudo su</b>. Now that you are the root user, enter this command:
  - nano /var/www/html/wp-config.php

<p align="center">
<img src="https://i.imgur.com/nJGR5Kw.png" height="80%" width="80%" alt="Step 16-6"/>
</p>

- While in the text editor, paste the following code into the text editor in the location specified within the image.

<p align="center">
<img src="https://i.imgur.com/A3w4TLE.png" height="80%" width="80%" alt="Step 16-7"/>
</p>

<p align="center">
<img src="https://i.imgur.com/biow1us.png" height="80%" width="80%" alt="Step 16-8"/>
</p>

- Now that the configuration file has been modified, access the website in a new tab. Enter the domain name with <b>HTTPS</b> in the URL. When the website is accessed, the connection is now secure. Because the URL has changed again, update the WordPress settings as an admin to reflect the change.

<p align="center">
<img src="https://i.imgur.com/shuvAaU.png" height="80%" width="80%" alt="Step 16-9"/>
</p>

<p align="center">
<img src="https://i.imgur.com/dqhp8xe.png" height="80%" width="80%" alt="Step 16-10"/>
</p>

<p align="center">
<img src="https://i.imgur.com/4tadVoA.png" height="80%" width="80%" alt="Step 16-11"/>
</p>

<h3>&#9328; Create an Auto Scaling Group</h3>

- An auto scaling group will be made to dynamically create and scale web servers in the Private App Subnets. This is to make the website highly available, scalable, fault-tolerant, and elastic. Before making the auto scaling group, <b>terminate Webserver AZ1 and AZ2</b> from the <b>EC2</b> console.

<p align="center">
<img src="https://i.imgur.com/FoXfHP1.png" height="80%" width="80%" alt="Step 17-1"/>
</p>

- A launch template will be made to contain the configurations of the EC2 instances that are created in the auto scaling gorup. Select <b>Launch Templates</b> from the land-hand menu and click <b>Create launch template</b>. Use the following configurations to create the launch template:
  - <b>Launch template name</b>: Dev-Launch-Template
  - <b>Description</b>: Launch Template for ASG
  - <b>Enable Auto Scaling guidance</b>
  - <b>Application and OS Images</b>: Amazon Linux
  - <b>Amazon Machine Image</b>: Amazon Linux 2 AMI (free tier eligible)
  - <b>Instance type</b>: t2.micro
  - <b>Key pair</b>: myec2key
  - <b>Firewall (security groups)</b>: Web Server Security Group
- Before creating the launch template, insert the same script used to create the EC2 instance for the application load balancer (and make sure the EFS data is correct) into the user data.

<p align="center">
<img src="https://i.imgur.com/jDeOgMq.png" height="80%" width="80%" alt="Step 17-2"/>
</p>

<p align="center">
<img src="https://i.imgur.com/7Ba46zW.png" height="80%" width="80%" alt="Step 17-3"/>
</p>

- Now that the launch template is made, the auto scaling group can now be created. Select <b>Auto Scaling Groups</b> on the left-hand menu and click <b>Create Auto Scaling group</b>. Use the following configurations to create the Auto Scaling group:
  - <b>Auto Scaling gorup name</b>: Dev-ASG
  - <b>Launch template</b> (_<b>NOT</b>_ Launch configuration): Dev-Launch-Template
  - <b>VPC</b>: Dev VPC
  - <b>Availabilty Zones and subnets</b>: Private App Subnet AZ1 and Private App Subnet AZ2
  - <b>Load balancing</b>: Attach to an existing load balancer
  - <b>Attach to an existing load balancer</b>: Choose from your load balancer target groups
  - <b>Existing load balancer target groups</b>: Dev-TG | HTTP
  - <b>Additional health check types</b>: Turn on Elastic Load Balancing health checks
  - <b>Monitoring</b>: Enable group metrics collection within CloudWatch
  - <b>Group size</b>: Desired capacity - 2, Minimum capacity - 1, Maximum capacity - 4
  - <b>Add notifications</b>: Create a topic named Default_CloudWatch_Alarms_Topic with your email as the recipient
  - <b>Tags</b>: Key - Name, Value - ASG-Webserver
- After creating the Auto Scaling group, the group will now make two new instances based on the launch template.

<p align="center">
<img src="https://i.imgur.com/kiL3Pao.png" height="80%" width="80%" alt="Step 17-4"/>
</p>

<p align="center">
<img src="https://i.imgur.com/DcUxO0T.png" height="80%" width="80%" alt="Step 17-5"/>
</p>

- The website will take a few minutes to become online once again because the instances need to be created. Their status can be checked in the <b>Target Groups</b> tab. The <b>Health status</b> will show whether or not an instance is healthy. If the health status is unhealthy, wait a few minutes to allow all the user data script to run and install the software. Refreshing the page will update the status to healthy after time has passed.

<p align="center">
<img src="https://i.imgur.com/VJxwuUQ.png" height="80%" width="80%" alt="Step 17-6"/>
</p>

- All that is left is to log in to the website as an admin and customize the website. The <b>Appearance</b> tab on WordPress will allow different themes and other customization items to be used to make the website more presentable.

<p align="center">
<img src="https://i.imgur.com/zisDtwJ.png" height="80%" width="80%" alt="Step 17-7"/>
</p>

- The website is now complete!

<h3>&#9329; Clean Up</h3>

- I took down the website in order to save money because AWS charges based on waht resources are used and for how long. Resources will have to be deleted in a specific order because some resources are reliant on other resources and they won't be deleted otherwise. If you would like to save on money and take down the website, delete the resources in this order:
  - Auto Scaling group
  - Launch template
  - Application load balancer
  - Target Group
  - RDS instance
  - Bastion host
  - EFS
  - Security groups (except for the default security group)
  - NAT gateways
  - VPC
  - Elastic IPs (AWS will charge elastic IPs that are unusued and are not associated with a resource)
  - Record Set (A record)
 
<h2>Lessons Learned </h2>

This project has made me become more familiar with the services offered in AWS. The website is reliant on many different components that work together to keep it afloat. I was already familiar with some of the services because I studied for the AWS Certified Cloud Practitioner. I previously studied for the Solutions Architect Associate exam, but I have not taken the exam. While doing this project, I refered to some of the notes I took while studying for the Solutions Architect Associate exam. I did not have any problems making the website because I tackled each task one step at a time. At a glance, there is a lot of work that needs to be done to make a dynamic website on AWS. I thought I would be overwhelemed, but that was far from the truth. Now, I have a much deeper understanding of how to handle certain tasks when using services on AWS in order to make future projects work.
