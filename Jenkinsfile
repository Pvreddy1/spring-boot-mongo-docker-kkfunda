pipeline {
    agent any

    tools {
        maven 'maven3.9.9'
    }

    environment {
        SNYK_TOKEN = credentials('SNYK_TOKEN')
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Cloning Repository..."
                git branch: 'main',
                    url: 'https://github.com/Pvreddy1/spring-boot-mongo-docker-kkfunda.git'
            }
        }

        stage('Maven Build') {
            steps {
                echo "Building the Maven Project..."
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''
                        mvn sonar:sonar \
                        -Dsonar.projectKey=spring-boot-mongo \
                        -Dsonar.projectName="Spring Boot Mongo Project"
                    '''
                }
            }
        }

        stage('Snyk Repo Scan') {
            steps {
                sh '''
                    echo "Authenticating with Snyk..."
                    snyk auth $SNYK_TOKEN

                    echo "Running Snyk Test..."
                    snyk test --all-projects || true

                    echo "Sending results to Snyk dashboard..."
                    snyk monitor --all-projects || true
                '''
            }
        }

        stage('Build & Tag Docker Image') {
            steps {
                script {
                    echo "Building Docker image..."
                    withDockerRegistry(credentialsId: 'docker') {
                        sh 'docker build -t pvreddy1/mongospring:latest .'
                    }
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh '''
                    echo "Scanning Docker image using Trivy..."
                    trivy image --exit-code 0 --severity HIGH,CRITICAL pvreddy1/mongospring:latest
                    trivy image --exit-code 1 --severity CRITICAL pvreddy1/mongospring:latest || true
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    echo "Pushing Docker image to DockerHub..."
                    withDockerRegistry(credentialsId: 'docker') {
                        sh 'docker push pvreddy1/mongospring:latest'
                    }
                }
            }
        }
    }
}
