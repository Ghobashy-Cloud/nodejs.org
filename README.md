
# Node.js Application Kubernetes Deployment

## Overview

This project demonstrates the deployment of a Node.js application from the Node.js GitHub repository to a local Kubernetes cluster. The deployment uses **ArgoCD** for continuous delivery. Additionally, a **Jenkins** pipeline is set up to automate the build and deployment process, including Dockerization and testing.

## Prerequisites

Before proceeding, ensure you have the following setup:

- **Two Virtual Machines (VMs)**
  - One VM for **Jenkins** with **Docker** installed.
  - One VM for **Minikube** and **kubectl**.
- **Minikube** installed on the second VM.
- **ArgoCD** installed and configured on Minikube.
- **GitHub account** to fork and store the repository.
- **Docker Hub account** for storing the Docker image.

## Part 1: Jenkins Setup and Dockerization

### Step 1: Fork the Repository

1. Go to the Node.js GitHub repository: [https://github.com/nodejs/nodejs.org.git](https://github.com/nodejs/nodejs.org.git)
2. Click on the "Fork" button in the upper-right corner to create a copy of the repository in your GitHub account.

### Step 2: Clone the Repository on Your VM

On the VM where Jenkins is installed, clone the forked repository:
```bash
git clone git@github.com:<Ghobashy-Cloud>/nodejs.org.git
```

### Step 3: Build the Application and Run Unit Tests Locally

1. Ensure your VM has **Node.js** installed (version 18 is required). If not, install it:
   ```bash
   sudo apt update
   sudo apt install -y nodejs npm
   nvm install 18
   ```
2. Navigate to the project directory:
   ```bash
   cd nodejs.org
   ```
3. Install the dependencies:
   ```bash
   npm install
   ```
4. Run the unit tests to ensure the application is working:
   ```bash
   npm test
   ```

### Step 4: Create a Dockerfile and Push it to GitHub

1. Create a `Dockerfile` in the root of your project directory. Here's an example Dockerfile:
   ```dockerfile
   FROM node:18-alpine
   WORKDIR /app
   COPY package*.json ./
   RUN npm install
   COPY . .
   EXPOSE 3000
   CMD ["npm", "start"]
   ```
2. Commit the `Dockerfile` to your repository:
   ```bash
   git add Dockerfile
   git commit -m "Add Dockerfile"
   git push origin main
   ```

### Step 5: Set Up Jenkins Pipeline

You will now configure Jenkins to automate the build and deployment process. The pipeline will:

1. **Fetch the code** from the GitHub repository.
2. **Install dependencies**.
3. **Run unit tests**.
4. **Dockerize the application**.
5. **Push the Docker image to Docker Hub**.

Here is a basic structure of the Jenkins pipeline (`Jenkinsfile`):

```groovy
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git 'git@github.com:<Ghobashy-Cloud>/nodejs.org.git'
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
                script {
                    docker.build('khaled55/nodejs-app')
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
                        docker.image('khaled55/nodejs-app').push('latest')
                    }
                }
            }
        }
    }
}
```


## Part 2: Kubernetes Deployment

### Step 1: Setup Minikube and ArgoCD

1. **Install Minikube** on the second VM:
   ```bash
   curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
   sudo install minikube-linux-amd64 /usr/local/bin/minikube
   ```
   Start Minikube:
   ```bash
   minikube start
   ```

2. **Install kubectl**:
   ```bash
   sudo apt-get update
   sudo apt-get install -y apt-transport-https
   sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
   sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
   sudo apt-get install -y kubectl
   ```

3. **Install ArgoCD**:
   - Install ArgoCD on Minikube following [ArgoCD official instructions](https://argo-cd.readthedocs.io/en/stable/getting_started/).

### Step 2: Deploy the Node.js App to Kubernetes

1. **Create a Kubernetes Deployment YAML** file (`deployment.yaml`):
   ```yaml
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
           - containerPort: 3000
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: nodejs-app-service
   spec:
     type: NodePort
     selector:
       app: nodejs-app
     ports:
     - protocol: TCP
       port: 80
       targetPort: 3000
       nodePort: 30080
   ```

2. **Apply the Kubernetes configuration** to deploy the app:
   ```bash
   kubectl apply -f deployment.yaml
   ```

3. **Monitor the deployment** to ensure it’s running:
   ```bash
   kubectl get pods
   ```

4. **Access the Node.js application**:
   Find the Minikube IP and access the app on `http://<minikube-ip>:30080`.

## Conclusion

This project demonstrates a full CI/CD pipeline for a Node.js application using Jenkins, Docker, Minikube, and ArgoCD. It automates the build, test, and deployment processes, allowing seamless integration and continuous delivery of the application.

