# AWS CloudFormation templates

The set of templates used by [AWS CloudFormation](https://aws.amazon.com/cloudformation/)
to provision cloud resources.

## ECR repository

A simple template to create a ECR repository with default settings (private
access).

```.bash
aws cloudformation deploy \
  --stack-name my-ecr-stack \
  --template-file ./ecr.yaml \
  --parameter-overrides "Name=<reponame>"
```

Parameters:

* `Name`: (required) a name of ECR repository.

## VPC

A template used to set up a VPC with 2 subnets, each in own availability
zone. Both subnets have the Internet Gateway attached.

```.bash
aws cloudformation deploy \
  --stack-name my-vpc-stack \
  --template-file ./vpc.yaml
```

Parameters:

* `CIDR`: The CIDR block for the VPC. If not specified, then `10.0.0.0/16`
  will be used.
* `SubnetACidr`: The CIDR block for subnet in availability zone "A". If not
  specified, then `10.0.0.0/17` will be used.
* `SubnetBCidr`: The CIDR block for subnet in availability zone "B". If not
  specified, then `10.0.128.0/17` will be used.
