# aws-fargate-playground
Getting to grips and distributing basic apps with AWS Fargate

# TODO:

- Follow guidance from (Rapid Docker on AWS)[https://github.com/jpignata/fargate]
- Gain more knowledge with CloudFormation, ECS, ECR and AWS architecture.
- Practice with both AWS cli and fargate cli.
- With AdonisJs test project...
    - Dockerize and clean
    - Upload to ECR
    - Setup load balancer
    - Use ECR containers in Fargate
    - Clean up projects

### Update AWS CLI

```
pip3 install awscli --upgrade --user
```

### Configure User

Ensure that your AWS's security polices have been checked and updated.

Create a IAM user which has permission for ECS and only provides an access key for CLI access, ignore the AWS portal usage.

Use with the newly created **AWS Access Key ID** and **AWS Secret Access Key**.

Set the **region name**, currently using **us-east-1**.

```
aws configure
```

### Create Cluster

```
aws ecs create-cluster --cluster-name fargate-cluster
```

### Register Task Definition

See: **basic-task/sample-fargate-task**.

The **file://** usage is required for the JSON to be correctly passed from the file.

```
aws ecs register-task-definition --cli-input-json file://basic-task/sample-fargate-task.json
```

### List Task Definitions

```
aws ecs list-task-definitions
```

### Create Service

Look at, to get a **subnets** and **securityGroups** value:
- https://console.aws.amazon.com/vpc/home?region=us-east-1#subnets:sort=SubnetId
- https://console.aws.amazon.com/vpc/home?region=us-east-1#SecurityGroups:sort=groupId

#### No Public IP

```
aws ecs create-service --cluster fargate-cluster --service-name fargate-service --task-definition sample-fargate:1 --desired-count 2 --launch-type "FARGATE" --network-configuration "awsvpcConfiguration={subnets=[subnet-0597a417aac87d6de],securityGroups=[sg-0a4b02cf0dadc87d4]}"
```

#### Service With Public IP

```
aws ecs create-service --cluster fargate-cluster --service-name fargate-service --task-definition sample-fargate:1 --desired-count 2 --launch-type "FARGATE" --network-configuration "awsvpcConfiguration={subnets=[subnet-0597a417aac87d6de],securityGroups=[sg-0a4b02cf0dadc87d4],assignPublicIp=ENABLED}"
```

###  List Services

```
aws ecs list-services --cluster fargate-cluster
```

###  Update Service - scale down for deletion

```
aws ecs update-service --cluster fargate-cluster --service fargate-service --desired-count 0
```
###  Delete Service

```
aws ecs delete-service --cluster fargate-cluster --service fargate-service
```

### Describe the Running Service

```
aws ecs describe-services --cluster fargate-cluster --services fargate-service
```

### Updating a Task

```
aws ecs register-task-definition --cli-input-json file://basic-task/updated-sample-fargate-task.json
```

### Deploying the new change

Requires a script via https://github.com/aws/aws-cli/issues/3064

```
aws ecs update-service --cluster fargate-cluster \
                       --service fargate-service \
                       --task-definition sample-fargate:2
```

# DNS

Set up an **A** record for mapping a given task service IP to your domain.

- http://nginx-demo.remotesoftwaredevelopment.com/

# Resources

- https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html
- https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_AWSCLI_Fargate.html
- https://medium.com/@daniel.revie1/deploying-react-docker-image-to-aws-fargate-bf551128cb88
- https://cloudonaut.io/rapid-docker-on-aws/
- https://github.com/jpignata/fargate
