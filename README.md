# AWS MariaDB Lab: Secure EC2 Bastion and RDS Setup

This repository provides a comprehensive guide to setting up a secure AWS environment with a bastion host on EC2 for accessing a private RDS MariaDB instance. It's ideal for learning VPC networking, secure SSH access, and database management in the cloud.

## Overview

In this lab, you'll:
- Create a custom VPC with public and private subnets.
- Launch a bastion EC2 instance in the public subnet.
- Deploy a MariaDB RDS instance in the private subnet.
- Connect securely from the bastion to RDS and perform database operations.
- Clean up resources to avoid unnecessary costs.

## Prerequisites

- An AWS account with permissions for VPC, EC2, and RDS.
- SSH client on your local machine.
- A key pair for EC2 (e.g., generate one named `key.pem` in the AWS console).

## Step 1: Create VPC and Subnets

1. In the VPC console, create a VPC:
   - Name: MyVPC
   - IPv4 CIDR: 10.0.0.0/16

2. Create subnets:
   - Public Subnet: MyPublicSubnet (CIDR: 10.0.1.0/24, Availability Zone: e.g., ap-south-1a)
   - Private Subnet: MyPrivateSubnet (CIDR: 10.0.2.0/24, Availability Zone: e.g., ap-south-1b)

3. Attach an Internet Gateway to MyVPC.
4. Create a route table for the public subnet and add a route (0.0.0.0/0) to the Internet Gateway.

## Step 2: Launch Bastion EC2 Instance in Public Subnet

1. In the EC2 console, launch an instance:
   - Name: BastionHost
   - AMI: Amazon Linux 2023
   - Instance Type: t3.micro
   - Key Pair: Select your key pair (e.g., associated with key.pem)
   - Network: MyVPC, MyPublicSubnet
   - Security Group: Create a new group allowing SSH (port 22) from your IP only
   - Enable Auto-assign Public IP

2. SSH into the bastion:
   ```
   ssh -i "key.pem" ec2-user@<Bastion-Public-IP>
   ```

3. Update the instance and install MariaDB:
   ```
   sudo dnf update -y
   sudo dnf install mariadb105-server -y
   sudo systemctl start mariadb
   sudo systemctl enable mariadb
   ```

## Step 3: Launch RDS MariaDB Instance in Private Subnet

1. In the RDS console, create a database:
   - Engine: MariaDB
   - DB Instance Identifier: my-rds-mariadb
   - Instance Class: db.t3.micro
   - Storage: 20 GiB General Purpose SSD
   - VPC: MyVPC
   - Subnet Group: Create one including MyPrivateSubnet
   - Public Access: No
   - Security Group: Allow port 3306 from the bastion's security group
   - Master Username: admin
   - Master Password: Set a secure password

2. Note the RDS endpoint (e.g., my-rds-mariadb.<region>.rds.amazonaws.com).

## Step 4: Connect to RDS from Bastion Host

From the bastion instance:
```
mysql -h my-rds-mariadb.<region>.rds.amazonaws.com -u admin -p
```
Enter the master password.

## Step 5: Run MariaDB SQL Commands on RDS

Once connected:

- Create a database:
  ```
  CREATE DATABASE testdb;
  USE testdb;
  SHOW DATABASES;
  ```

- Create a table:
  ```
  CREATE TABLE benchmark (
      id INT AUTO_INCREMENT PRIMARY KEY,
      data VARCHAR(255)
  );
  ```

- Insert data using a stored procedure:
  ```
  DELIMITER $$
  CREATE PROCEDURE insert_data()
  BEGIN
      DECLARE i INT DEFAULT 1;
      WHILE i <= 10000 DO
          INSERT INTO benchmark (data) VALUES (UUID());
          SET i = i + 1;
      END WHILE;
  END$$
  DELIMITER ;
  CALL insert_data();
  ```

- Verify data:
  ```
  SELECT COUNT(*) FROM benchmark;
  ```

- Cleanup:
  ```
  DELETE FROM benchmark;
  DROP TABLE IF EXISTS benchmark;
  DROP DATABASE testdb;
  ```

## Step 6: Cleanup (Teardown)

To avoid charges:
1. **Delete RDS Instance:** RDS > Databases > Select my-rds-mariadb > Actions > Delete.
2. **Terminate EC2 Bastion:** EC2 > Instances > Select BastionHost > Actions > Instance State > Terminate.
3. **Network Cleanup:**
   - Detach and delete Internet Gateway.
   - Delete route tables (disassociate subnets first).
   - Delete subnets (MyPublicSubnet, MyPrivateSubnet).
   - Delete VPC (MyVPC).
4. **Delete Security Groups:** EC2 > Security Groups > Delete custom groups.

## Best Practices

- Restrict security groups to necessary IPs/ports.
- Use IAM roles for enhanced security.
- Enable RDS encryption and backups for production.
- Monitor with CloudWatch.


## Connect with Me

<a href="https://www.linkedin.com/in/hiranmaya-biswas-505a1823a/" target="_blank"> <img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white" alt="LinkedIn Button"> </a> 
<a href="https://github.com/Harry-404" target="_blank"> <img src="https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white" alt="GitHub Button"> </a>

