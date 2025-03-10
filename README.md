# Documentation: Automating Deployment of a NodeJS Web Application to AWS EC2 using GitHub Actions

This documentation outlines the steps to set up a CI/CD pipeline using GitHub Actions to automate the deployment of a Node.js web application to an AWS EC2 instance. It includes instructions for setting up the GitHub repository, configuring the EC2 instance, installing Docker, and creating the GitHub Actions workflow.

## Table of Contents

- [Documentation: Automating Deployment of a NodeJS Web Application to AWS EC2 using GitHub Actions](#documentation-automating-deployment-of-a-nodejs-web-application-to-aws-ec2-using-github-actions)
  - [Table of Contents](#table-of-contents)
  - [Prerequisites](#prerequisites)
  - [Step 1: Setting Up the GitHub Repository](#step-1-setting-up-the-github-repository)
  - [Step 2: Preparing the AWS EC2 Instance](#step-2-preparing-the-aws-ec2-instance)
  - [Step 3: Configuring GitHub Actions Runner on EC2](#step-3-configuring-github-actions-runner-on-ec2)
  - [Step 4: Installing Docker on EC2](#step-4-installing-docker-on-ec2)
  - [Step 5: Creating GitHub Actions Workflow](#step-5-creating-github-actions-workflow)
  - [Step 6: Testing the Deployment](#step-6-testing-the-deployment)
  - [Conclusion](#conclusion)
  - [Additional Notes:](#additional-notes)

---

## Prerequisites

Before you begin, ensure that you have the following:
- A GitHub account.
- An AWS account with permissions to create and manage EC2 instances.
- A `.pem` key file for SSH access to your EC2 instance.
- Basic knowledge of Git, GitHub, and AWS EC2.

---

## Step 1: Setting Up the GitHub Repository

1. **Create a New GitHub Repository**:
   - Go to [GitHub](https://github.com) and create a new repository.
   - Name it something relevant like `nodejs-web-app`.

2. **Fork or Clone a Simple NodeJS Web App**:
   - You can fork an existing simple NodeJS web app or clone one from a public repository.

---

## Step 2: Preparing the AWS EC2 Instance

1. **Launch an EC2 Instance**:
   - Log in to the AWS Management Console.
   - Navigate to the EC2 Dashboard and launch a new instance.
   - Choose an Ubuntu Server AMI (e.g., Ubuntu 24.04 LTS).

2. **Configure Security Groups**:
   - Ensure the security group allows inbound traffic on port `8080` (for the NodeJS app) and port `22` (for SSH).

3. **Download the `.pem` Key File**:
   - Save the `.pem` key file to your local machine (e.g., in the `Downloads` directory).

4. **SSH into the EC2 Instance**:
    cd Downloads
    chmod 400 "nodejs-web-app-key.pem"
    ssh -i "nodejs-web-app-key.pem" ubuntu@ec2-54-89-144-102.compute-1.amazonaws.com

5. **Update and Upgrade the System**:
    sudo apt update
    sudo apt-get upgrade


## Step 3: Configuring GitHub Actions Runner on EC2

1. **Install GitHub Actions Runner**:
    mkdir actions-runner && cd actions-runner
    curl -o actions-runner-linux-x64-2.322.0.tar.gz -L https://github.com/actions/runner/releases/download/v2.322.0 actions-runner-linux-x64-2.322.0.tar.gz
    echo "b13b784808359f31bc79b08a191f5f83757852957dd8fe3dbfcc38202ccf5768  actions-runner-linux-x64-2.322.0.tar.gz" | shasum -a 256 -c
    tar xzf ./actions-runner-linux-x64-2.322.0.tar.gz

2. **Configure the Runner**:
    ./config.sh --url https://github.com/PercyHub7/nodejs-web-app --token BDJ2HYRWKI4KDEKKYW7UDZLHZX6CG
    - Follow the prompts to configure the runner.
    - Accept default options unless you have specific requirements.
  
3. **Start the Runner**:
    ./run.sh &

## Step 4: Installing Docker on EC2

1. **Install Docker Dependencies**:
    sudo apt-get install ca-certificates curl
    sudo install -m 0755 -d /etc/apt/keyrings
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
    sudo chmod a+r /etc/apt/keyrings/docker.asc

2. **Add Docker Repository**:

    echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

3. **Install Docker**:
    sudo apt-get update
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

4. **Verify Docker Installation**:
    sudo docker run hello-world

## Step 5: Creating GitHub Actions Workflow
1. **Create Workflow File** :
    - In your GitHub repository, navigate to .github/workflows.
    - Create a new file named ci-cd.yml.
  
2. **Define the Workflow**:
    name: CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: self-hosted
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Build Docker image
        run: |
          docker build -t percy7/nodejs-web-app .

      - name: Login to Docker Hub
        run: |
          echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u percy7 --password-stdin

      - name: Push Docker image
        run: |
          docker push percy7/nodejs-web-app

      - name: Deploy to EC2
        run: |
          docker pull percy7/nodejs-web-app
          docker stop nodejs-web-app-container || true
          docker rm nodejs-web-app-container || true
          docker run -d -p 8080:8080 --name nodejs-web-app-container percy7/nodejs-web-app

3. **Commit and Push the Workflow** :
     - Commit the workflow file and push it to the main branch.
  
## Step 6: Testing the Deployment

1. **Access the Deployed Application** :
     - After the workflow completes successfully, access the deployed application by navigating to http://<EC2_PUBLIC_IP>:8080 in your browser.

2. **Verify the Response** :
     - You should see the response message configured in your NodeJS app (e.g., {"status":200,"message":"hello world changing"}).


## Conclusion
    You have successfully set up a CI/CD pipeline using GitHub Actions to automate the deployment of a NodeJS web application to an AWS EC2 instance. This setup ensures that every push to the main branch triggers a build and deployment process, streamlining the development workflow.

## Additional Notes:
    - Ensure your GitHub Secrets are properly configured (DOCKER_PASSWORD, etc.).
    - Regularly monitor and maintain your EC2 instance and GitHub Actions runner for optimal performance.
    - Consider adding additional steps such as running tests before deployment for enhanced reliability.
