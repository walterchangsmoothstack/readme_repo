# TERRAFORM

### Goal:
Set up an application (using Fargate) that uses an application load balancer and is connected to a private RDS instance.

## Networking Module
- VPC with CIDR block 10.0.0.0/16
- (4) Subnets with CIDR blocks 10.0.x.0/24
- Private Subnets:
  - Create a subnet group with both attached for later use in the database section
- Public Subnets:
  - Create an internet gateway and a route table
  - Attach the internet gateway to the route table and associate your designated public subnets to the route table  
  - **Set auto-assign IPv4 addresses on subnets and enable DNS hostnames on VPC**

## Database Module
- Create an RDS instance with the subnet group of unexposed subnets from earlier
- Create a security group with (2) ingress rules: 
  - Allow MYSQL/Aurora port 3306 from bastion host IP (to be created later) OR from anywhere.
  - Allow all TCP from load balancer (to be created later) OR from anywhere.
- Create a bastion host to connect to the database:
  - Create an EC2 instance with a **public** subnet from earlier specified.
  - Create a template file to inject shell scripts to create the MYSQL database upon launch.
> #!/bin/bash
>
>yum update -y
>yum install mysql -y
>
> mysql -h ${RDS_MYSQL_ENDPOINT} -u ${RDS_MYSQL_USER} -p${RDS_MYSQL_PASS} -D=${RDS_MYSQL_BASE} << EOF 
> 
> CREATE SCHEMA IF NOT EXISTS \`utopia\` DEFAULT CHARACTER SET utf8 ;
> USE \`utopia\` ;
>
>--------------------------------------------------------
>-- Table \`utopia\`.\`airport\`
>--------------------------------------------------------
>CREATE TABLE IF NOT EXISTS \`utopia\`.\`airport\` (  
>  \`iata_id\` CHAR(3) NOT NULL,  
>  \`city\` VARCHAR(45) NOT NULL,  
>  PRIMARY KEY (\`iata_id\`),  
>  UNIQUE INDEX \`iata_id_UNIQUE\` (\`iata_id\` ASC) VISIBLE)  
>ENGINE = InnoDB;  
>EOF  

