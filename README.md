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
    - [understanding-docker](https://raw.githubusercontent.com/AbdElRhmanArafa/Building-Microservices-and-a-CI-CD-Pipeline-with-AWS/44ab769a3010edc6a9db4ea0b67d4f8529255800/Docs/Architecture%20diagram.png)
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

