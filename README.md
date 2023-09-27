# On-prem-AWS-Migration
In this project, we will be migrating a workload running in an enterprise data center to AWS. The missing is to migrate the application and the database using EC2 and RDS services.
For this solution, we will implement VPC along with its subnets, an EC2 instance to store the application, and an RDS instance that will store the Database.
Also, we will need to implement an Internet Gateway to enable the apps to be accessible over the Internet. 

# Solution

## Part 1: Deployment "Creating EC2 and RDS instance according to the 'sizing'"

- Creating VPC and the Subnets:

• VPC Only

• Name tag: vpc-bootcamp

• CIDR: 10.0.0.0/16 [ No Overlap with On-p ]


► Creating Public and Privates Subnets:


• Name: Public Subnet

• AZ: us-east-1a

• CIDR: 10.0.0.0/24


• Name: Private Subnet

• AZ: us-east-1a

• CIDR: 10.0.1.0/24



• Name: Private Subnet

• AZ: us-east-1b

• CIDR: 10.0.2.0/24



AWS RDS Subnet Group is a collection of subnets that you can use to deploy your RDS database in a VPC.
Your VPC must have at least 2 subnets and these subnets must be in different Availability Zones


## Creating EC2:

• Name: awsuse1app01

• Ubuntu: 22.04 LTS

• Instance Type: t2.micro

• Key Pair: ec2-ssh        | Create a folder 'c:\aws-mod3' and save you ssh key file

• Network Settings:

VPC (vpc-bootcamp)

Subnet (Public Subnet)

Auto-assign public IP: Enable

Firewall (security groups): app01-sg [ Ports: 22 | 8080 ]


## Creating DB RDS:


• RDS | Create database

• Standard create

• MySql | Version: MySql 5.7.'3x'  | Choose the major MySql 'v5.7.xx'  Keeping '5.7'

• Template: Free Tier


• DB instance identifier: awsuse1db01

• Credentials Settings: admin | admin123456

• DB instance class: db.t2.micro


Connectivity


(*) Don’t connect to an EC2 compute resource

• VPC: vpc-bootcamp

• VPC security group: let's keep the 'default' for while!

• AZ: us-east-1a

• Database port: 3306 (Additional configuration)


## Part 2: Installing and Setting up the Packages and Libraries for the App and DB connection

- Connecting to the VM - EC2
  

► Install Git Bash

https://git-scm.com/downloads


• ssh -i ec2-ssh ubuntu@ip-publico-ec2 | It'll not work, why?!


- Creating an Internet Gateway, attaching it to a VPC and creating a Route

• VPC | Internet Gateway: igw-mod3 | Action: Attach to VPC (vpc-bootcamp)

• VPC

Route Table | Routes | Edit routes

Add route: 0.0.0.0/0 - Target: Internet Gateway (igw-mod3)



- Installing the application's dependencies
- 

"Ubuntu 22.04" prompts ‘pop-ups’ after install/update packages requiring:


“What service should be restarted?”


By default this is set to "interactive" mode which causes the interruption of scripts.
To change this behavior, we can edit the /etc/needrestart/needrestart.conf file, changing lines:


From:
#$nrconf{restart} = 'i';

To:

$nrconf{restart} = 'a';


‘i’ interactive | ‘a’ automatically


- Change by using the ‘sed’ command:
  

sudo sed -i "/#\$nrconf{restart} = 'i';/s/.*/\$nrconf{restart} = 'a';/" /etc/needrestart/needrestart.conf


- Checking the changes made:
  

cat /etc/needrestart/needrestart.conf | grep -i nrconf{restart}


sudo apt update

sudo apt install python3-dev -y

sudo apt install python3-pip -y


sudo apt install build-essential libssl-dev libffi-dev -y

sudo apt install libmysqlclient-dev -y

sudo apt install unzip -y

sudo apt install libpq-dev libxml2-dev libxslt1-dev libldap2-dev -y

sudo apt install libsasl2-dev libffi-dev -y

pip3 install flask


Warning...

Fixing...

export PATH=$PATH:/home/ubuntu/.local/bin/

pip3 install wtforms

sudo apt install pkg-config

pip3 install flask_mysqldb

pip3 install passlib


- MySql Client Installation:


sudo apt-get install mysql-client -y


## Part 3: Go Live


- Creating a Security Group for RDS [ VPC | SG ]


• Name: EC2toRDS-sg

• Description: SG to allow access to MySQL by the application running at EC2.

• VPC: vpc-bootcamp

• Inbound rules | Add rule | Type: MYSQL/Aurora   | Destination: 0.0.0.0/0



- Associating the SG (EC2toRDS-sg) to the RDS instance (awsuse1db01):


• RDS | DB Instances | awsuse1db01 | Modify

• Connectivity | SG: EC2toRDS-sg

• Continue... Apply immediately and "Modify DB instance"


Check if it was associated


- Connecting to the VM - EC2

Let's test the SSH connection using the Windows Command Prompt: CMD

• ssh -i ec2-ssh ubuntu@ip-publico-ec2

WARNING: UNPROTECTED PRIVATE KEY FILE!

Select 'ec2-ssh.pem' file -> Right Click -> Properties

Security > Advanced > Change the "Owner" to your user.

"Disable inheritance" | 'Convert inherited permissions into explicit permissions'
Add > Select a Principal | Add your user

Remove all other Objects/Users


- Downloading the Aplication and the 'Dump' files:

• wget https://aws-mod3.s3.amazonaws.com/wikiapp.zip

• wget https://aws-mod3.s3.amazonaws.com/dump.sql


- Connecting to the MySQL server (AWS RDS)

• Copy the Endpoint of your RDS and replace it below

mysql -h <rds_endpoint> -P 3306 -u admin -p

EXAMPLE: mysql -h awsuse1db01.culdx6558fqq.us-east-1.rds.amazonaws.com -P 3306 -u admin -p

• Password: admin123456


- Creating a DB 'wikidb' and importing data to it:

• show databases;

• create database wikidb;

• show databases;

• use wikidb;

• show tables;

• source dump.sql;

• show tables;

• select * from articles;


- Creating an user 'wiki' in the "wikidb"
- 

• CREATE USER wiki@'%' IDENTIFIED BY 'admin123456';

• GRANT ALL PRIVILEGES ON wikidb.* TO wiki@'%';

• FLUSH PRIVILEGES;

• EXIT;



- Unziping the application's file

• unzip wikiapp.zip


- Editing the file 'wiki.py'

• cd wikiapp/

• vi wiki.py

► session '# Config MySQL'

• 'MYSQL_HOST' = 'awsuse1db01.cfhprbugweo6.us-east-1.rds.amazonaws.com' [EXAMPLE]

• 'MYSQL_USER' = 'wiki'



- Loading the application:

• python3 wiki.py


- Let's validate the migration:

• Copy the Public IP of your EC2 instance and add to it the port "8080":

• <PUBLIC_IP_OF_YOUR_EC2>:8080

• Login: admin / admin


## Post Go Live

- Removing the resources created:

• Deleting RDS: Do not need to create 'Snapshots',

Do not need to retain 'Backups',

Check the option [✔] 'I acknowledge that upon instance deletion, automated backups, including system snapshots and point-in-time recovery, will no longer be available.'

• Terminating EC2: Select the EC2 | Action: Terminate

