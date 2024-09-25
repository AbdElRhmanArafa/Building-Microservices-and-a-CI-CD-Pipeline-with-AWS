# Microservice Project: Docker + AWS ECR + ECS

## Description
This project demonstrates the use of Docker to containerize a microservice and deploy it to AWS using ECS and ECR. and using codepipeline to automate the deployment process.

## Prerequisites
- AWS Account
- Docker
- AWS CLI configured with proper permissions.

## Architecture
![Architecture](https://raw.githubusercontent.com/AbdElRhmanArafa/Building-Microservices-and-a-CI-CD-Pipeline-with-AWS/44ab769a3010edc6a9db4ea0b67d4f8529255800/Docs/Architecture%20diagram.png)
## Setup and Installation
1. Create a cloud9 environment in your AWS account for development user.
    - choose the environment type as `EC2` and the instance type as `t2.micro` and  Network settings as `ssh` in `public subnet`.
2. Clone the repository to your cloud9 environment.
    ```bash
    git clone https://github.com/AbdElRhmanArafa/microservices-pop-Cafe.git
    ```
3. Build two docker images for the two microservices.
    - Navigate to the `customer` or `employee` directory and build the docker image.
    - Run the following command to build the docker image.
        ```bash
        docker build -t <image-name> .
        ```
4. test the docker image locally.
    - make sure you open inbound traffic on port 8080 and 80 in the security group of the cloud9 environment.
    - Run the following command to test the docker image.
        ```bash
        docker run -p 8080:8080 <image-name:customer>
        docker run -p 80:8080 <image-name:employee>
        ```
5. Create an ECR repository for each microservice using AWScli.
    - Run the following command to create the ECR repository.
        ```bash
        aws ecr create-repository --repository-name <repository-name>
        ```
6. Tag the docker image and push it to the ECR repository.
    - login to the ECR repository.
        ```bash
        aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <aws-account-id>.dkr.ecr.<region>.amazonaws.com
        ```
    - Run the following command to tag the docker image.
        ```bash
        docker tag <image-name> <aws-account-id>.dkr.ecr.<region>.amazonaws.com/<repository-name>:<tag>
        ```
    - Run the following command to push the docker image to the ECR repository.
        ```bash
        docker push <aws-account-id>.dkr.ecr.<region>.amazonaws.com/<repository-name>:<tag>
        ```
    - ![Understanding Docker](https://raw.githubusercontent.com/AbdElRhmanArafa/Building-Microservices-and-a-CI-CD-Pipeline-with-AWS/715a755ab3196759fc217a3a9e214e5529e1a91b/Docs/understanding-docker.png)
7. Create an ECS cluster
    - Run the following command to create the ECS cluster.
        ```bash
        aws ecs create-cluster \
        --cluster-name <microservices-serverlesscluster> \
        --capacity-providers FARGATE \
        --region us-east-1
        ```
8. Create a task definition for each microservice.
    - Navigate to the `.aws` directory and create a task definition for each microservice.
    - Run the following command to create the task definition.
        ```bash
        aws ecs register-task-definition --cli-input-json "$(cat <task-definition-file>)"
        ```
    - Note that you can create log groups for each microservice in the cloudwatch logs manually or using the task definition example in my taskdef: 
        ```bash
        awslogs-create-group: true: # This means that the log group awslogs-capstone will be automatically created if it doesn't exist
        ```
### Edit the taskdef-customer.json file After Creating the Task Definition

1. Modify line 5 to match the following:
    ```json
    "image": "<IMAGE1_NAME>",
    ```
**Analysis**: `<IMAGE1_NAME>` is a placeholder and not a valid image name, which is why you initially set the image name to `customer` when registering the first revision with Amazon ECS. Later in the project, `CodePipeline` will dynamically update the correct image name at runtime.


9. Create an AppSpec file for each microservice.
     > **Important**: DON'T modify `<TASK_DEFINITION>`. This setting will be updated automatically when the pipeline runs.

10. Create a target group for each microservice.
    - Run the following command to create the target group.
        ```bash
        aws elbv2 create-target-group \
        --name <target-group-name> \
        --protocol HTTP \
        --port 80 \
        --target-type ip \
        --vpc-id <vpc-id> \
        --health-check-protocol HTTP \
        --health-check-path / 
        ```
11. Create a load balancer and securty group.
    - Create a new EC2 security group named `<Name-sg>` to use in `same vpc that you used in target group`.
    - Add inbound rules that allow TCP traffic from any IPv4 address on ports 80 and 8080.
    - In the Amazon EC2 console, create an Application Load Balancer named <Name-LB>
    - Use the same VPC and two public subnets.
    - Configure two listeners on it. The first should listen on HTTP:80 and forward traffic to customer-tg-two by default. 
    - The second should listen on HTTP:8080 and forward traffic to customer-tg-one by default.
    - Add new rules to the target group to route traffic to the correct target group based on the path. if IF Path is `/admin/*` THEN Forward to employee-tg-<>.
12. Creating two Amazon ECS services
    - Create two services in the ECS cluster, one for each microservice.
    - make sure you have the correct values for the following fields `By replacing it from sevice file.json`:
        - `cluster`
        - `service-name`
        - `task-definition`
        - `load-balancers`
        - `target-group-arn`
        - `public-subnets`
        - `security-groups`
    - Run the following command to create the service.
        - make sure that all traget groups associated with the load balancer.
        ```bash
        aws ecs create-service --servic-name `<name>` --cli-input-json "$(cat <PATH_TO_SERVICE_DEFINITION>)"
        ```



