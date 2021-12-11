# TERRAFORM

### Goal:
Set up an application (using Fargate) that uses an application load balancer and is connected to a private RDS instance. Possible reference https://section411.com/2019/07/hello-world/

### Project Layout
![image](https://user-images.githubusercontent.com/90656351/145691195-38e94c8b-90b9-4dfd-8a89-2c90ce05019a.png)


## Networking Module
- VPC with CIDR block 10.0.0.0/16
- (4) Subnets with CIDR blocks 10.0.x.0/24
- Private Subnets:
  - Create a subnet group with two subnets attached to it (these will be designated as our private subnets)
- Public Subnets:
  - Create an internet gateway and a route table
  - Attach the internet gateway to the route table and associate your other two subnets (public subnets) with the route table.  
  - **Set auto-assign IPv4 addresses on subnets and enable DNS hostnames on VPC**

## Database Module
- Create an RDS instance with the subnet group of unexposed subnets from earlier
- Create a security group with (2) ingress rules: 
  - Allow MYSQL/Aurora port 3306 from bastion host IP (to be created later) OR from anywhere.
  - Allow all TCP from load balancer (to be created later) OR from anywhere.
- Create a bastion host to connect to the database:
  - Create an EC2 instance with a **public** subnet from earlier specified.
  - Create a security group for the bastion host to allow for SSH (port 22 from your IP)
  - Create a template file to inject shell scripts to create the MYSQL database upon launch.
```
      #!/bin/bash

      yum update -y
      yum install mysql -y

      mysql -h ${RDS_MYSQL_ENDPOINT} -u ${RDS_MYSQL_USER} -p${RDS_MYSQL_PASS} -D=${RDS_MYSQL_BASE} << EOF 
 
      CREATE SCHEMA IF NOT EXISTS \`utopia\` DEFAULT CHARACTER SET utf8 ;
      USE \`utopia\` ;

      --------------------------------------------------------
      -- Table \`utopia\`.\`airport\`
      --------------------------------------------------------
      CREATE TABLE IF NOT EXISTS \`utopia\`.\`airport\` (  
        \`iata_id\` CHAR(3) NOT NULL,  
        \`city\` VARCHAR(45) NOT NULL,  
        PRIMARY KEY (\`iata_id\`),  
        UNIQUE INDEX \`iata_id_UNIQUE\` (\`iata_id\` ASC) VISIBLE)  
      ENGINE = InnoDB;  
      EOF  
```
## Cluster Module
- Create a cluster, task definitions, and services
- Create an ecsTaskExecutionRole for the task definition to function
- Example of task definition inside a for each mapping:
```
        resource "aws_ecs_task_definition" "task_definitions" {
        for_each                 =  var.task_definitions
        family                   = each.value["family"] 
        task_role_arn            = aws_iam_role.api_ecs_task_execution_role.arn
        execution_role_arn       = aws_iam_role.api_ecs_task_execution_role.arn
        network_mode             = each.value["network_mode"]
        cpu                      = each.value["cpu"]
        memory                   = each.value["memory"]
        requires_compatibilities = ["FARGATE"]
        container_definitions    = jsonencode([
       {
          name      = each.value["container_name"]
          image     = each.value["image"]
          cpu       = 10
          memory    = 512
         network_mode             = each.value["network_mode"]
          environment = var.environment
          portMappings = [
            {
              containerPort = var.app_port
              hostPort      = var.app_port
            }
            ]
          },
         ]) 
      }
```
- Example of a service inside a for each mapping:
```    
        resource "aws_ecs_service" "ecs_services" {
        for_each          = var.ecs_services
        name              = each.value["name"]
        cluster           = aws_ecs_cluster.utopia-cluster.id
        task_definition   = aws_ecs_task_definition.task_definitions[ each.value["task_name"] ].id
        desired_count     = each.value["desired_count"]
        launch_type       = "FARGATE"

        network_configuration {
          security_groups  = [aws_security_group.service_sg.id]
          subnets          = var.public_subnet_ids
          assign_public_ip = true
        }

        load_balancer {
          target_group_arn = each.value["target_group_arn"]
          container_name   = each.value["container_name"]
          container_port   = var.app_port
        }
    }
```
- Referenced resources such as container uri's will come from the ECR (create a data object that depends on the given repository). Target groups will come from the Load Balancer Module speaking of which...

## Load Balancer Module
- Create a load balancer, target groups, and listener rules.
- Create a new security group for the load balancer that accepts outside traffic. (Unsure whether port 80 or all TCP is more appropriate)
- Create an ```aws_alb_listener``` resource, and a target group for each task defintion.
- Create an ```aws_alb_listener```. This will route default traffic on port 80 to the target group referenced. Subsequent target groups will be attached to the listener using ```aws_lb_listener_rule``` resources.
