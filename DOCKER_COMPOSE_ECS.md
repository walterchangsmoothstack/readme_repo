# DOCKER COMPOSE ECS CONTEXT

Without using the ```ecs-cli```, run docker compose in the root directory of your docker compose file after switching to the ecs context.
> docker create ecs myecs
> docker context use myecs


Using the ```ecs-cli```, include a ecs-params.yml with desired service parameters.
> ecs-cli configure --cluster ecscluster --region us-west-1 --default-launch-type EC2 --config-name ecscluster
> ecs-cli configure profile --access-key [ACCESS_KEY] --secret-key [SECRET ACCESS KEY] --profile-name ecscluster
> ecs-cli up --capability-iam
