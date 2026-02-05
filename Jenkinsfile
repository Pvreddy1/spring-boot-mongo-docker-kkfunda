pipeline {
    agent any

    tools {
        maven 'maven3.9.9'
    }

    environment {
        SNYK_TOKEN = credentials('SNYK_TOKEN') // Snyk API token
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

        stage('Snyk Repo Scan') {
            steps {
                echo "Running Snyk scan for dependency vulnerabilities..."
                sh '''
                    echo "Authenticating with Snyk..."
                    snyk auth $SNYK_TOKEN
                    echo "Running Snyk Test..."
                    snyk test --all-projects || true
                    snyk monitor --all-projects || true
                '''
            }
        }

    }
}
