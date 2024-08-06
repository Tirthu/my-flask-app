# My Flask App CI/CD Pipeline

## Overview

This project is a simple Flask web application that demonstrates automated deployment using GitHub Actions and AWS ECS (Elastic Container Service). The CI/CD pipeline is configured to automate the build, deployment, and rollback processes for the application.

## Getting Started

Follow these instructions to set up and run the project locally, as well as to understand the CI/CD pipeline configuration.

### Prerequisites

- **Docker**: Ensure Docker is installed and running on your local machine.
- **AWS CLI**: AWS Command Line Interface to interact with AWS services.
- **GitHub Account**: For accessing and managing the repository.
- **AWS Account**: To deploy resources on AWS.

### Installation

1. **Clone the Repository**

   ```bash
   git clone https://github.com/your-username/my-flask-app.git
 cd my-flask-app
## Set up AWS CLI with your credentials
aws configure

## 

## Configure AWS CLI and create an ECR repository
aws ecr create-repository --repository-name my-flask-app

## Authenticate Docker with ECR
aws ecr get-login-password --region <your-region> | docker login --username AWS --password-stdin <your-account-id>.dkr.ecr.<your-region>.amazonaws.com

## Create an ECS Cluster and Service
aws ecs create-cluster --cluster-name my-flask-app-cluster
aws ecs create-service \
  --cluster default \
  --service-name my-flask-app-service \
  --task-definition my-flask-app \
  --desired-count 1 \
  --launch-type EC2

## CI/CD Pipeline
The pipeline is set up using GitHub Actions to automate the following steps:

1.Checkout Code

The pipeline checks out the code from the GitHub repository.

2.Build Docker Image

The Docker image is built using a multistage build process to optimize the final image. The image is then tagged and pushed to AWS ECR (Elastic Container Registry).

3.Push Docker Image to ECR

The image is pushed to the AWS ECR repository.

4.Deploy to ECS

The Docker image is deployed to AWS ECS. If the deployment fails, the pipeline performs a rollback to the previous stable version.

5.Perform Integration Tests

Integration tests are run to ensure the deployment is successful. If tests fail, the pipeline triggers a rollback to the previous stable version.
## GitHub Actions Workflow
name: Deploy to ECS

on:
  push:
    branches:
      - master

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Login to Amazon ECR
      run: |
        aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 024848477002.dkr.ecr.ap-south-1.amazonaws.com

    - name: Build Docker image
      run: docker build -t 024848477002.dkr.ecr.ap-south-1.amazonaws.com/my-flask-app:latest .

    - name: Push Docker image
      run: docker push 024848477002.dkr.ecr.ap-south-1.amazonaws.com/my-flask-app:latest

    - name: Deploy to ECS
      run: |
        aws ecs update-service --cluster default --service my-flask-app-service --force-new-deployment

    - name: Run Integration Tests
      run: |
        # Add commands to run integration tests here
        # For example, you might use curl or a testing framework to verify the deployment
        curl -f http://<your-ecs-service-url>/ || exit 1

    - name: Rollback on Failure
      if: failure()
      run: |
        aws ecs update-service --cluster default --service my-flask-app-service --force-new-deployment --desired-count 0

## Testing
To test the CI/CD pipeline:

1.Push Changes to Repository

Any push to the master branch will trigger the GitHub Actions workflow.

2.Check Workflow Status

Monitor the status of the workflow in the "Actions" tab of your GitHub repository.

3.Verify Deployment

Ensure the application is correctly deployed and accessible at the provided ECS service URL
## License
This project is licensed under the MIT License - see the LICENSE file for details.
## Repository URL
https://github.com/Tirthu/my-flask-app.git