pipeline {
    agent any

    tools {
        maven 'maven3'
        jdk 'jdk17'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/Abhijadhav03/fullstack-blog-cicd.git'
            }
        }

        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh '''
                    # Run Trivy FS Scan
                    trivy fs --format table -o fs.html .
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh """
                        $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Blogging-app -Dsonar.projectKey=Blogging-app \
                        -Dsonar.java.binaries=target
                    """
                }
            }
        }

        stage('Build') {
            steps {
                sh "mvn package"
            }
        }

        stage('Docker Build and Tag') {
            steps {
                script{
                withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                    sh "docker build -t abhijadhav03/bloggingapp:latest ."
                }
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh '''
                    # Run Trivy Image Scan
                    trivy image --format table -o image.html abhijadhav03/bloggingapp:latest
                '''
            }
        }

        stage('Docker Push Image') {
            steps {
                script{
                withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                    sh "docker push abhijadhav03/bloggingapp:latest"
                }
                }
            }
        }

       stage('Publish Artifacts') {
            steps {
                withMaven(globalMavenSettingsConfig: 'maven-settings', jdk: 'jdk17', maven: 'maven3', traceability: true) {
                    sh "mvn deploy"
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
        
        stage('Package') {
            steps {
                sh "mvn package"
            }
        }
    }
}
