# DevOps Full-Stack Application CI/CD Pipeline

## Introduction
Welcome to this DevOps and Cloud project! This project focuses on building a full-stack application from scratch, deploying it, and monitoring it in a production-like environment.


## Project Overview
### **Objective**
- Develop a full-stack application with features like user login, registration, posting content, and viewing posts.
- Deploy the application using a CI/CD pipeline.
- Monitor the application using modern DevOps tools.

### **Tools and Technologies**
- **GitHub**: Source code management.
- **Jenkins**: CI/CD pipeline automation.
- **SonarQube**: Code quality and security analysis.
- **Nexus**: Artifact repository for storing build artifacts.
- **EKS (Elastic Kubernetes Service)**: Deployment platform for the application.
- **Prometheus & Grafana**: Monitoring and visualization.
- **Blackbox Exporter**: Monitoring the application's availability.

---
## Project Architecture
### **Developer Workflow**
1. Developer writes code and pushes it to GitHub.
2. A Jenkins pipeline is triggered automatically to build, test, and deploy the application.

### **CI/CD Pipeline**
1. Code is checked out from GitHub.
2. Code is compiled, tested, and analyzed for vulnerabilities using SonarQube.
3. Build artifacts are stored in Nexus.
4. The application is deployed to EKS.

### **Monitoring**
- **Prometheus** collects metrics from the application and infrastructure.
- **Grafana** visualizes the metrics.
- **Blackbox Exporter** monitors the application's HTTP endpoints.

---
## Steps to Implement the Project

### **1. Set Up the GitHub Repository**
```bash
git init
git remote add origin https://github.com/your-username/your-repo-name.git
git add .
git commit -m "Initial commit"
git push -u origin main
```

### **2. Create Virtual Machines on AWS**
- Launch 3 EC2 instances for **Jenkins, SonarQube, and Nexus**.
- Use **Amazon Linux 2** or **Ubuntu** as the OS.
- Configure Security Groups:
  - **Jenkins**: Allow ports `22 (SSH)`, `8080 (Jenkins UI)`.
  - **SonarQube**: Allow ports `22 (SSH)`, `9000 (SonarQube UI)`.
  - **Nexus**: Allow ports `22 (SSH)`, `8081 (Nexus UI)`.

### **3. Install Required Tools**
#### **Install Java (Required for Jenkins, SonarQube, Nexus)**
```bash
sudo apt update
sudo apt install openjdk-11-jdk -y
```
#### **Install Jenkins**
```bash
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install jenkins -y
sudo systemctl start jenkins
sudo systemctl enable jenkins
```
#### **Install Docker (Required for SonarQube and Nexus)**
```bash
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
```
#### **Install SonarQube**
```bash
docker run -d --name sonarqube -p 9000:9000 sonarqube:lts
```
#### **Install Nexus**
```bash
docker run -d --name nexus -p 8081:8081 sonatype/nexus3
```

### **4. Configure Jenkins, SonarQube, and Nexus**
#### **Jenkins**
- Access Jenkins at `http://<jenkins-server-ip>:8080`.
- Install required plugins: **Git, Pipeline, SonarQube Scanner, Nexus Artifact Uploader**.
- Configure credentials for GitHub, SonarQube, and Nexus.

#### **SonarQube**
- Access SonarQube at `http://<sonarqube-server-ip>:9000`.
- Log in with the default credentials (**admin/admin**).
- Generate a token for Jenkins integration.

#### **Nexus**
- Access Nexus at `http://<nexus-server-ip>:8081`.
- Log in with the default credentials (**admin/admin123**).
- Create a repository for storing build artifacts.

### **5. Write the Jenkins Pipeline**
Create a `Jenkinsfile` in your GitHub repository:

```groovy
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/your-username/your-repo-name.git'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean package'  // Replace with your build command
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }
        stage('Upload to Nexus') {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: '<nexus-server-ip>:8081',
                    groupId: 'com.example',
                    version: '1.0',
                    repository: 'your-repo',
                    credentialsId: 'nexus-credentials',
                    artifacts: [
                        [artifactId: 'your-artifact',
                         classifier: '',
                         file: 'target/your-app.jar',
                         type: 'jar']
                    ]
                )
            }
        }
        stage('Deploy to EKS') {
            steps {
                sh 'kubectl apply -f k8s-deployment.yaml'
            }
        }
    }
}
```

### **6. Set Up Monitoring**
#### **Install Prometheus and Grafana**
```bash
docker run -d --name prometheus -p 9090:9090 prom/prometheus
docker run -d --name grafana -p 3000:3000 grafana/grafana
```
#### **Install Blackbox Exporter**
```bash
docker run -d --name blackbox-exporter -p 9115:9115 prom/blackbox-exporter
```
- Configure Prometheus to scrape metrics from the application and Blackbox Exporter.

---

