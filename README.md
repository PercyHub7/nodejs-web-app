# README

This documentation outlines the steps to set up a CI/CD pipeline using GitHub Actions to automate the deployment of a Node.js web application to an AWS EC2 instance. It includes instructions for setting up the GitHub repository, configuring the EC2 instance, installing Docker, and creating the GitHub Actions workflow.

## Table of Contents

- [README](#readme)
  - [Table of Contents](#table-of-contents)
  - [Prerequisites](#prerequisites)
  - [Step 1: Setting Up the GitHub Repository](#step-1-setting-up-the-github-repository)
  - [Step 2: Preparing the AWS EC2 Instance](#step-2-preparing-the-aws-ec2-instance)
  - [Step 3: Configuring GitHub Actions Runner on EC2](#step-3-configuring-github-actions-runner-on-ec2)

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
