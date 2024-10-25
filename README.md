Node.js Application Deployment to Kubernetes with ArgoCD and Jenkins
Overview
This project demonstrates the deployment of a Node.js application from the official Node.js GitHub repository to a local Kubernetes cluster using ArgoCD for continuous delivery. The entire process is fully automated through a Jenkins pipeline, which handles building, testing, Dockerizing, and pushing the application to Docker Hub.

Project Structure
Jenkins: Automates the CI/CD pipeline for building and deploying the Node.js application.
Docker: Containerizes the Node.js application.
Minikube: Hosts the local Kubernetes cluster.
ArgoCD: Manages continuous delivery for the Kubernetes cluster.
Prerequisites
Before starting, ensure the following setup:

Virtual Machine 1 (Jenkins)
Jenkins installed.
Docker installed.
Virtual Machine 2 (Minikube and ArgoCD)
Minikube installed.
kubectl installed.
ArgoCD installed.
Accounts
GitHub Account: Ghobashy-Cloud
Docker Hub Account: khaled55
Part 1: Jenkins Setup and Dockerization
Step 1: Fork the Repository
Navigate to the Node.js GitHub repository.
Click the "Fork" button to create a copy of the repository in Khaled's GitHub account.
Step 2: Clone the Repository
Clone the forked repository to the Jenkins VM:

bash
Copy code
git clone https://github.com/Ghobashy-Cloud/nodejs.org.git
Step 3: Build and Run Unit Tests Locally
Ensure that Node.js is installed on the VM. Install Node.js version 20 or higher:

bash
Copy code
nvm install 20
nvm use 20
To build and test the application locally:

bash
Copy code
npm install
npm test
Step 4: Create and Push the Dockerfile
Create a Dockerfile for the Node.js application:

dockerfile
Copy code
# Use Node.js official image
FROM node:20

# Set the working directory
WORKDIR /usr/src/app

# Copy package.json and install dependencies
COPY package*.json ./
RUN npm install

# Copy the application code
COPY . .

# Expose the application port
EXPOSE 8000

# Command to start the application
CMD ["npm", "start"]
Push this Dockerfile to Khaled's GitHub repository.

Step 5: Set Up Jenkins Pipeline
Configure a multibranch pipeline in Jenkins to automate the following stages:

Checkout the repository: Pull the latest code from GitHub.
Install dependencies: Use npm to install required packages.
Run unit tests: Execute npm test to ensure the code passes all tests.
Dockerize the application: Build the Docker image using the Dockerfile.
Push Docker image: Push the Docker image to Khaled's Docker Hub account (khaled55/nodejs-app).
Sample Jenkinsfile Structure:
groovy
Copy code
pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/Ghobashy-Cloud/nodejs.org.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        stage('Run Unit Tests') {
            steps {
                sh 'npm test'
            }
        }
        stage('Dockerize') {
            steps {
                sh 'docker build -t khaled55/nodejs-app .'
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push khaled55/nodejs-app'
                }
            }
        }
    }
}
Part 2: Kubernetes Deployment
Step 1: Set Up Minikube and ArgoCD
On the second VM:

Install Minikube.
Install kubectl.
Install ArgoCD.
Step 2: Create Kubernetes Deployment YAML
Create a deployment.yaml file for the Node.js application and push it to Khaled's GitHub repository:

yaml
Copy code
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nodejs-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nodejs-app
  template:
    metadata:
      labels:
        app: nodejs-app
    spec:
      containers:
      - name: nodejs-app
        image: khaled55/nodejs-app:latest
        ports:
        - containerPort: 8000
---
apiVersion: v1
kind: Service
metadata:
  name: nodejs-app-service
spec:
  type: NodePort
  ports:
    - port: 8000
      nodePort: 30007
  selector:
    app: nodejs-app
    
Step 3: Deploy with ArgoCD
Add Khaled's GitHub repository as a Git source in ArgoCD.
Create an ArgoCD application to manage the Node.js deployment.
Sync ArgoCD to deploy the application to the Minikube cluster.
Step 4: Verify the Deployment
After deploying with ArgoCD, ensure that the application is running successfully:

Get the list of pods to check their status:
bash
Copy code
kubectl get pods
Access the application via the Minikube IP and the NodePort:
Retrieve the Minikube IP:
bash
Copy code
minikube ip
Open a web browser and navigate to http://<MINIKUBE_IP>:30007. Khaled should see the Node.js application running.
Conclusion
This project illustrates a complete CI/CD pipeline for a Node.js application, encompassing building, testing, containerization, and deployment using Kubernetes and ArgoCD. The Jenkins pipeline automates these processes, ensuring that every new code change is tested, Dockerized, and deployed without manual intervention.
