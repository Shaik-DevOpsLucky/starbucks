pipeline {
    agent any
    tools {
        nodejs 'node18'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage("Clean Workspace") {
            steps {
                cleanWs()
            }
        }
        stage("Git Checkout") {
            steps {
                git branch: 'main', url: 'https://github.com/Shaik-DevOpsLucky/starbucks.git'
            }
        }
        stage("Install NPM Dependencies") {
            steps {
                sh "npm install"
            }
        }
        stage("SonarQube Analysis") {
            steps {
                withSonarQubeEnv('sonar-scanner') {
                    sh """$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=starbucks -Dsonar.projectKey=starbucks"""
                }
            }
        }
        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false
                }
            }
        }
        stage("Build Docker Image") {
            steps {
                sh "docker build -t starbucks ."
            }
        }
        stage("Tag & Push to DockerHub") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-creds') {
                        sh "docker tag starbucks moulashaik9618/starbucks:latest"
                        sh "docker push moulashaik9618/starbucks:latest"
                    }
                }
            }
        }
        stage("Deploy to Container") {
            steps {
                sh 'docker run -d --name starbucks -p 3000:3000 moulashaik9618/starbucks:latest'
            }
        }
    }
}
