# AWS CloudFormation templates

The set of templates used by [AWS CloudFormation](https://aws.amazon.com/cloudformation/)
to provision cloud resources (EC2 instance and ECS cluster with services).

## ECR repository

A simple template to create a ECR repository with default settings (private
access).

```.bash
aws cloudformation deploy \
  --stack-name ECR_STACK_NAME \
  --template-file ./ecr.yaml \
  --parameter-overrides "Name=repository"
```

Parameters:

* `Name`: (required) a name of ECR repository.

## VPC

A template used to set up a VPC with 2 subnets, each in own availability
zone. Both subnets have the Internet Gateway attached.

```.bash
aws cloudformation deploy \
  --stack-name VPC_STACK_NAME \
  --template-file ./vpc.yaml
```

Parameters:

* `CIDR`: The CIDR block for the VPC. If not specified, then `10.0.0.0/16`
  will be used.
* `SubnetACidr`: The CIDR block for subnet in availability zone "A". If not
  specified, then `10.0.0.0/17` will be used.
* `SubnetBCidr`: The CIDR block for subnet in availability zone "B". If not
  specified, then `10.0.128.0/17` will be used.

Exports (replace `VPC_STACK_NAME` with name of your stack):

* `VPC_STACK_NAME:VPC`: the identifier of created VPC.
* `VPC_STACK_NAME:SubnetA`: the identifier of subnet in availability zone "A".
* `VPC_STACK_NAME:SubnetB`: the identifier of subnet in availability zone "B".

## EC2

A template used to create EC2 instance. Depends on the [VPC](#VPC) template.
The instance will be attached to subnet in availability zone "A" and will
have public IP.

This template is very basic and doesn't let you configure most of EC2 instance
properties.

```.bash
aws cloudformation deploy \
  --stack-name EC2_STACK_NAME \
  --template-file ./vpc.yaml \
  --parameter-overrides "VpcStackName=VPC_STACK_NAME" "KeyName=EC2_KEY_PAIR"
```

Parameters:

* `InstanceType`: EC2 instance type. Default is `t3.micro`.
* `VpcStackName`: (required) A name of stack created by using a [VPC](#VPC)
  template.
* `KeyName`: (required) A name of SSH key pair.
* `AllowSshFrom`: A CIDR block of IP addresses allowed to access SSH port.
  By default all IP addresses can access the SSH port of instance.
* `LatestAmiId`: AMI identifier. By default uses the latest official image
  of Amazon Linux.

## ECS Cluster

A template used to create a ECS cluster backed by AWS Fargate. It also
creates Internet-facing load balancer. Depends on [VPC](#VPC) template.

```.bash
aws cloudformation deploy \
  --stack-name ECS_CLUSTER_STACK_NAME \
  --template-file ./ecs_cluster.yaml \
  --capabilities CAPABILITY_IAM \
  --parameter-overrides "Name=CLUSTER_NAME" "VpcStackName=VPC_STACK_NAME"
```

Parameters:

* `Name`: (required) A name of ECS cluster.
* `VpcStackName`: (required) A name of stack created by using a [VPC](#VPC)
  template.

Exports (replace `ECS_CLUSTER_STACK_NAME` with name of your stack):

* `ECS_CLUSTER_STACK_NAME:ClusterName`: A name of cluster.
* `ECS_CLUSTER_STACK_NAME:LoadBalancerHost`: A host of load balancer.
* `ECS_CLUSTER_STACK_NAME:ECSRole`: ARN of ECS IAM role.
* `ECS_CLUSTER_STACK_NAME:ECSTaskExecutionRole`: ARN of task execution IAM role.
* `ECS_CLUSTER_STACK_NAME:VPC`: Identifier of VPC used for cluster.
* `ECS_CLUSTER_STACK_NAME:VPCSubnetA`: Identifier of VPC subnet for availability zone "A".
* `ECS_CLUSTER_STACK_NAME:VPCSubnetB`: Identifier of VPC subnet for availability zone "B".
* `ECS_CLUSTER_STACK_NAME:LoadBalancerListener`: ARN of load balancer listener.
* `ECS_CLUSTER_STACK_NAME:FargateSecurityGroup`: identifier of Fargate container security group.

## ECS service

A template used to create a ECS service. Depends on [ECS Cluster](#ECS-Cluster)
template.

```.bash
aws cloudformation deploy \
  --stack-name ECS_SERVICE_STACK_NAME \
  --template-file ./ecs_service.yaml \
  --parameter-overrides "Name=helloworld" "EcsClusterStackName=ECS_CLUSTER_STACK_NAME"
```

Parameters:

* `Name`: (required) A name of service.
* `Image`: Docker image to deploy. If not specified, a default one will deploy
  "hello world" web application.
* `ContainerPort`: A port on which application listens inside the container.
  Default is `5000` which is used by "hello world" application default Docker
  image.
* `ContainerCpu`: Amount of CPU shares for the container (1024 shares = 1 CPU).
  Must be a value defined by Fargate configurations. Default is `256`.
* `ContainerMemory`: Amount of memory for the container in megabytes.
  Must be a value defined by Fargate configurations. Default is `512`.
* `DesiredCount`: Amount of service replicas. Default is `1`.
* `Role`: IAM role for the service. Default is no role.
* `ServiceUrlPath`: An URL path that will trigger routing of traffic to the
  service. If specified as '*', then load balancer will redirect all traffic.
  Default is `*`.
* `Priority`: The priority for the routing rule added to the load balancer.
  Default is `1`.
* `EcsClusterStackNam`: (required) A name of stack created by using a
  [ECS Cluster](#ECS-Cluster) template.

## RDS

A template used to create RDS database instance. Supported engine types are:
mysql, mariadb, postgres.

```.bash
aws cloudformation deploy \
  --stack-name RDS_STACK_NAME \
  --template-file ./rds.yaml \
  --parameter-overrides 'Name=dbname' 'AdminUserName=USERNAME' 'AdminUserPassword=PASSWORD'
```

Parameters:

* `Name`: (required) Database name.
* `Engine`: The engine of RDS instance. Default is `mysql`.
* `AdminUserName`: (required) The admin user name.
* `AdminUserPass`: (required) The admin user password.
* `AllocatedStorage`: How much storage RDS instance should have in gigabytes.
   Default is `5`.
* `InstanceType`: The type of RDS instance. Default is `db.t3.small`.
* `AllowedSecurityGroup`: Resources of which security group should have access
   to the database. Default is `default`.
* `MultiAZ`: Whether the RDS instance should be multi-AZ (true/false). Default
  is `false`.

Exports (replace `RDS_STACK_NAME` with name of your stack):

* `RDS_STACK_NAME:EndpointAddress`: database endpoint host.
* `RDS_STACK_NAME:EndpointPort`: database endpoint port.
