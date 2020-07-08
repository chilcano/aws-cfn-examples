# Deploying sample containers following Networking Architecture Patterns in AWS ECS and CloudFormation

> **Original source:**
> Deploy containers using Elastic Container Service and CloudFormation
> [https://github.com/awslabs/aws-cloudformation-templates/tree/master/aws/services/ECS](https://github.com/awslabs/aws-cloudformation-templates/tree/master/aws/services/ECS)

You can deploy containers with two different networking approaches:

- Public VPC subnet with direct internet access
- Private VPC subnet without direct internet access

You can choose two different options for hosting the containers:

- AWS Fargate for hands-off container execution without managing EC2 instances
- Self managed cluster of EC2 hosts for control over instance type, or to use reserved or spot instances for savings

You can also choose between two different ways of sending traffic to the container:

- A public facing load balancer that accepts traffic from anyone on the internet (ideal for a public facing web service)
- A private, internal load balancer that only accepts traffic from other containers in the cluster (ideal for a private, internal service).

To use these templates launch a cluster template for the launch type and networking stack that you want. Then launch a service template for each service you want to run in the cluster. When launching a service template its important to make sure the "StackName" value is filled in with the same name that you selected for the name of your cluster stack.

Each of the service stacks has default values prefilled for launching a simple Nginx container, but can be adjusted to launch your own container.

## Scenario 1: Fully Public Container

![public subnet public load balancer](images/public-task-public-loadbalancer.png)

This architecture deploys your container into its own VPC, inside a public facing network subnet. The containers are hosted with direct access to the internet, and they are also accessible to other clients on the internet via a public facing appliation load balancer.

### 1a. Run in AWS Fargate

1. Launch the [fully public](FargateLaunchType/clusters/public-vpc.yml) or the [public + private](FargateLaunchType/clusters/private-vpc.yml) cluster template
2. Launch the [public facing service template](FargateLaunchType/services/public-service.yml).

### 1b. Run on EC2

1. Launch the [fully public](EC2LaunchType/clusters/public-vpc.yml) or the [public + private](EC2LaunchType/clusters/private-vpc.yml) cluster template.
2. Launch the [public facing service template](EC2LaunchType/services/public-service.yml).

```sh
$ export AWS_ACCESS_KEY_ID="xxxxxx"; export AWS_SECRET_ACCESS_KEY="yyyyyyy"
$ export AWS_DEFAULT_REGION="us-east-1"

// Creating the ECS stack passing the 'capabilities' parameter 
$ aws cloudformation create-stack \
    --template-body file://EC2LaunchType/clusters/public-vpc.yml \
    --stack-name ecs-scenario1b \
    --parameters ParameterKey=DesiredCapacity,ParameterValue=2 ParameterKey=MaxSize,ParameterValue=3 ParameterKey=InstanceType,ParameterValue=t2.small \
    --capabilities CAPABILITY_IAM 

{
    "StackId": "arn:aws:cloudformation:us-east-1:263455585760:stack/ECSClusterScenario1b/211c14a0-c070-11ea-9a34-120fbf0f0187"
}

// Querying the output variables
$ aws cloudformation describe-stacks --stack-name ecs-scenario1b | jq -c '.Stacks[].Outputs[] | {k: .OutputKey, v:.OutputValue}'

{"k":"ECSRole","v":"arn:aws:iam::263455585760:role/ecs-scenario1b-ECSRole-NGDBCS882EJA"}
{"k":"PublicSubnetOne","v":"subnet-0ce25596551e954ca"}
{"k":"VPCId","v":"vpc-0ad4ccb98543d525a"}
{"k":"PublicSubnetTwo","v":"subnet-01d3cd9afa361c92b"}
{"k":"PublicListener","v":"arn:aws:elasticloadbalancing:us-east-1:263455585760:listener/app/ecs-s-Publi-1GKJLECNCQAVM/0891d9619f437203/3a20e4f92c2e776f"}
{"k":"EcsHostSecurityGroup","v":"sg-0dd59f63865c60777"}
{"k":"ClusterName","v":"ecs-scenario1b-ECSCluster-RR9d5D58kHB5"}
{"k":"ExternalUrl","v":"http://ecs-s-Publi-1GKJLECNCQAVM-5168947.us-east-1.elb.amazonaws.com"}

// Deploying the ECS services to existing stack
$ aws cloudformation update-stack \
    --template-body file://EC2LaunchType/services/public-service.yml \
    --stack-name ecs-scenario1b \
    --parameters ParameterKey=StackName,ParameterValue=ecs-scenario1b 

// Removing ECS cluster
$ aws cloudformation delete-stack --stack-name ecs-scenario1b
```

**Note:**    
> If you have this error `An error occurred (InsufficientCapabilitiesException) when calling the CreateStack operation: Requires capabilities : [CAPABILITY_IAM]`, you can fix it using this parameter `--capabilities`.
> [Explanation here](https://medium.com/@dalumiller/aws-cloudformation-capabilities-parameter-ab73a373278).

## Scenario 2: Publicly Exposed Service with Private Networking

![private subnet public load balancer](images/private-task-public-loadbalancer.png)

This architecture deploys your container into a private subnet. The containers do not have direct internet access, or a public IP address. Their outbound traffic must go out via a NAT gateway, and receipients of requests from the containers will just see the request orginating from the IP address of the NAT gateway. However, inbound traffic from the public can still reach the containers because there is a public facing load balancer that can proxy traffic from the public to the containers in the private subnet.

### 2a. Run in AWS Fargate

1. Launch the [public + private](FargateLaunchType/clusters/private-vpc.yml) cluster template
2. Launch the [public facing, private subnet service template](FargateLaunchType/services/private-subnet-public-service.yml).

### 2b. Run on EC2

1. Launch the [public + private](EC2LaunchType/clusters/private-vpc.yml) cluster template
2. Launch the [public facing, private subnet service template](EC2LaunchType/services/public-service.yml).

&nbsp;

&nbsp;

## Scenario 3: Internal Service with Private Networking

![private subnet private load balancer](images/private-task-private-loadbalancer.png)

This architecture deploys your container in a private subnet, with no direct internet access. Outbound traffic from your container goes through an NAT gateway, and receipients of requests from the containers will just see the request orginating from the IP address of the NAT gateway. There is no acess to the container for the public. Instead there is a private, internal load balancer that only accepts traffic from other containers in the cluster. This is ideal for an internal service that is used by other services, but should not be used directly by the public.

### 3a. Run in AWS Fargate

1. Launch the [public + private](FargateLaunchType/clusters/private-vpc.yml) cluster template
2. Launch the [private service, private subnet template](FargateLaunchType/services/private-subnet-private-service.yml).

### 3b. Run on EC2

1. Launch the [public + private](EC2LaunchType/clusters/private-vpc.yml) cluster template
2. Launch the [private service, private subnet template](EC2LaunchType/services/private-service.yml).

