# DOCKER COMPOSE ECS CONTEXT

Without using the ```ecs-cli```, run docker compose in the root directory of your docker compose file after switching to the ecs context.
> docker context create ecs myecs  
> docker context use myecs


Using the ```ecs-cli```, include a ecs-params.yml with desired service parameters.
> ecs-cli configure --cluster ecscluster --region us-west-1 --default-launch-type EC2 --config-name ecscluster  
> ecs-cli configure profile --access-key [ACCESS_KEY] --secret-key [SECRET ACCESS KEY] --profile-name ecscluster  
> ecs-cli up --vpc vpc-010e28d6ac8abed72 --capability-iam --size 2 --instance-type t2.micro --key-pair MyKeySS --cluster-config ecscluster --subnets subnet-05a7fc58460a893b9,subnet-02ccd9af12fcac561  
> ecs-cli compose up --create-log-groups --cluster-config ecscluster

### Notes on ecs-params.yml
1). Values are strings (no need for quotes)  
2). Provide **task_role_arn** and **task_execution_role** to use secrets manager  
3). Role requires permissions to access **Secrets Manager** and **Parameter Store**

```{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ssm:GetParameters",
                "secretsmanager:GetSecretValue",
                "kms:Decrypt"
            ],
            "Resource": [
                "arn:aws:ssm:us-west-1:182310506613:parameter/*",
                "arn:aws:secretsmanager:us-west-1:182310506613:secret:utopia-secrets",
                "arn:aws:kms:us-west-1:182310506613:key/*"
            ]
        }
    ]
}
```  
4). Create AWS Parameters and reference them in the following manner:
> arn:aws:ssm:us-west-1:{*account_id*}:parameter/{*parameter_path*}  

5). Edit EC2 security group to accept TCP for specific port.
