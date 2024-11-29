# CI/CD Pipeline Setup for Node.js Backend

This repository contains the CI/CD pipeline setup for deploying a Node.js backend to an EC2 instance using GitHub Actions and Docker. The CI/CD pipeline automates the process of building, testing, and deploying your application whenever changes are pushed to the `main` branch.

## Contents
- [CI/CD Overview](#cicd-overview)
- [Setup Steps](#setup-steps)
- [GitHub Actions Workflow](#github-actions-workflow)
- [Environment Variables](#environment-variables)
- [Deploying to EC2](#deploying-to-ec2)
- [Testing the Pipeline](#testing-the-pipeline)

## CI/CD Overview
This setup automates the following tasks:

1. **Build**: Builds the Docker image for your Node.js backend.
2. **Push**: Pushes the Docker image to Docker Hub.
3. **Deploy**: Deploys the Docker container to an EC2 instance.

## Setup Steps
Follow these steps to set up the CI/CD pipeline:

1. **Create an EC2 Instance**: Set up an EC2 instance and install Docker on it.
2. **Configure AWS and Docker Hub Credentials**: Set up the necessary credentials in your GitHub repository's Secrets (e.g., `DOCKER_USERNAME`, `DOCKER_PASSWORD`, `EC2_HOST`, etc.).
3. **Create GitHub Actions Workflow**: Create a `.github/workflows` directory in your repository and add the GitHub Actions YAML file.

## GitHub Actions Workflow
The GitHub Actions workflow automates the CI/CD process. Here's an example workflow file (`deploy.yml`):

```yaml
name: Deploy Node.js Backend to EC2 with Docker

on:
  push:
    branches:
      - main  # Trigger this workflow on pushes to the main branch

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2

    - name: Login to Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Build Docker image
      run: |
        IMAGE_NAME="${{ secrets.DOCKER_USERNAME }}/node-backend:latest"
        docker build -t $IMAGE_NAME .

    - name: Push Docker image to Docker Hub
      run: |
        IMAGE_NAME="${{ secrets.DOCKER_USERNAME }}/node-backend:latest"
        docker push $IMAGE_NAME

  deploy:
    runs-on: ubuntu-latest
    needs: build
    steps:
    - name: SSH into EC2 and deploy Docker image
      uses: appleboy/ssh-action@v0.1.7
      with:
        host: ${{ secrets.EC2_HOST }}
        username: ubuntu  # Modify if needed based on your EC2 setup
        key: ${{ secrets.EC2_SSH_KEY }}
        port: 22
        script: |
          # Define the Docker image name
          IMAGE_NAME="${{ secrets.DOCKER_USERNAME }}/node-backend:latest"
          echo "Docker image to be pulled: $IMAGE_NAME"
          
          # Stop and remove any existing containers (with sudo)
          sudo docker stop node-backend || true
          sudo docker rm node-backend || true

          # Pull the latest Docker image (with sudo)
          sudo docker pull $IMAGE_NAME
          
          # Run the container (with sudo)
          sudo docker run -d --name node-backend -p 5000:5000 $IMAGE_NAME
