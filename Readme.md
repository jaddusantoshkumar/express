# Deploying simple web application on Kubernetes using Docker and Jenkins

This project contains a simple Express.js application that is containerized using Docker and can be deployed to Kubernetes and Jenkins for CI/CD process.
  
## Project Structure

- `Jenkinsfile`: Contains the Jenkins pipeline configuration for continuous integration.
- `Dockerfile`: Contains the Docker configuration to containerize the application.
- `deployment.yaml`: Contains the Kubernetes Deployment configuration to deploy the application on a Kubernetes cluster.
- `service.yaml`: Contains the Kubernetes Service configuration to expose the application to the outside world.
- `/examples/hello-world/index.js`: Contains the Express application code.

### **Step by step guide for the project**

**Step 1: Launch EC2 (Ubuntu 22.04):**

- Provision an EC2 instance on AWS with Ubuntu 22.04.
- Connect to the instance using SSH.
  
**Step 2: Install Docker**

- Set up Docker on the EC2 instance:
    
    ```bash
    
    sudo apt-get update
    sudo apt-get install docker.io -y
    sudo usermod -aG docker $USER  # Replace with your system's username, e.g., 'ubuntu'
    newgrp docker
    sudo chmod 777 /var/run/docker.sock
    ```
**Step 3: Setting up kubectl and its configuration**

- Install kubectl from official documentation and set it to use required cluster

    ```bash
    curl -LO https://dl.k8s.io/release/v1.23.0/bin/linux/amd64/kubectl
    sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
    #updating kube-config
    aws eks --region <region> update-kubeconfig --name <cluster-name>
    
**Step 4: CI/CD Setup**

1. **Install Jenkins for Automation:**
    - Install Jenkins on the EC2 instance to automate deployment:
    Install Java
    
    ```bash
    sudo apt update
    sudo apt install fontconfig openjdk-17-jre
    java -version
    openjdk version "17.0.8" 2023-07-18
    OpenJDK Runtime Environment (build 17.0.8+7-Debian-1deb12u1)
    OpenJDK 64-Bit Server VM (build 17.0.8+7-Debian-1deb12u1, mixed mode, sharing)
    
    #jenkins
    sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
    https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
    echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
    https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
    /etc/apt/sources.list.d/jenkins.list > /dev/null
    sudo apt-get update
    sudo apt-get install jenkins
    sudo systemctl start jenkins
    sudo systemctl enable jenkins
    # add jenkins user to Docker group
    sudo su
    sudo usermod -aG docker jenkins
    sudo systemctl restart jenkins
    ```
    
    - Access Jenkins in a web browser using the public IP of your EC2 instance.
        
        publicIp:8080
2. **Install Necessary Plugins in Jenkins:**

Goto Manage Jenkins →Plugins → Available Plugins →

Install below plugins

1 NodeJs Plugin

2 Docker related plugins

### **Configure Docker and Nodejs in Global Tool Configuration**

Goto Manage Jenkins → Tools → Install Docker and NodeJs(22.2.0)→ Click on Apply and Save
Create a Jenkins webhook


**Add DockerHub Credentials:**

- To securely handle DockerHub credentials in your Jenkins pipeline, follow these steps:
  - Go to "Dashboard" → "Manage Jenkins" → "Manage Credentials."
  - Click on "System" and then "Global credentials (unrestricted)."
  - Click on "Add Credentials" on the left side.
  - Choose "Secret text" as the kind of credentials.
  - Enter your DockerHub credentials (Username and Password) and give the credentials an ID (e.g., "docker").
  - Click "OK" to save your DockerHub credentials.

3. **Configure CI/CD Pipeline in Jenkins:**
- Create a CI/CD pipeline in Jenkins to automate your application deployment.

```groovy
pipeline {
    agent any
    tools {
        nodejs 'node'
    }
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Git Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/jaddusantoshkumar/express.git'
            }
        }
        stage('Build and Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh '''
                        docker build -t jaddusantoshkumar/example:latest .
                        docker push jaddusantoshkumar/example:latest
                        '''
                    }
                }
            }
        }
        stage('Deploy Application') {
            steps {
                sh '''
                kubectl apply -f deployment.yaml
                kubectl apply -f service.yaml
                '''
            }
        }
    }
}
```
**Step 4: Accessing the application**

- Now you can access the application using any NodeIp:31000 as we are exposing application on 31000 port using Nodeport Service.


