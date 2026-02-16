![Starbucks Clone Deployment](https://github.com/user-attachments/assets/6b654f47-9537-4b88-9584-41c760fc49ac)

# Deploy Starbucks Clone Application AWS using DevSecOps Approach


# **Install AWS CLI**
```
sudo apt install unzip -y
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```
 
# **Install Jenkins on Ubuntu:**

```
#!/bin/bash
sudo apt update -y
wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | sudo tee /etc/apt/keyrings/adoptium.asc
echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | sudo tee /etc/apt/sources.list.d/adoptium.list
sudo apt update -y
sudo apt install temurin-17-jdk -y
/usr/bin/java --version
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update -y
sudo apt-get install jenkins -y
sudo systemctl start jenkins
sudo systemctl status jenkins
```


# **Install Docker on Ubuntu:**
```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
sudo usermod -aG docker ubuntu
sudo chmod 777 /var/run/docker.sock
newgrp docker
sudo systemctl status docker
```

# **Install Trivy on Ubuntu:**

Reference Doc: https://aquasecurity.github.io/trivy/v0.55/getting-started/installation/
```
sudo apt-get install wget apt-transport-https gnupg
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb generic main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy
```


# **Install Docker Scout:**
```
docker login       `Give Dockerhub credentials here`
```
```
curl -sSfL https://raw.githubusercontent.com/docker/scout-cli/main/install.sh | sh -s -- -b /usr/local/bin
```
# Deployment Stages:
<img width="966" alt="Screenshot 2024-09-15 at 7 20 49 AM" src="https://github.com/user-attachments/assets/ddb5e618-79ab-49b3-8f13-b5114824eec3">

# Install sonarqube
```
docker run -d --name sonarqube -p 9000:9000 sonarqube:lts-community
```
# Install plugins
```
Eclipse Temurin installer
SonarQube Scanner
NodeJS
Docker
Docker Commons
Docker Pipeline
Docker API
docker-build-step
OWASP Dependency-CheckVersion
Pipeline: Stage View
Email Extension Template
Blue ocean
```

# In Sonarqube:
```
login sonarqube --> goto administration --> security --> users --> update token --> name of token --> create token.
```
# credentials setup for sonar and dockerhub
```
Goto manage jenkins --> credentials --> type:secrete token --> name:sonar-token --> create
```
```
Goto manage jenkins --> credentials --> type:username and password --> name:dockerhub-username --> password of dockerhub --> description:docker --> create
```
# Webhook configuration in sonarqube
```
Goto manage admin --> users --> configurations --> webhooks --> create --> name:jenkins --> URL:http://54.146.189.148:8080/sonarqube-webhook/ --> create
```
# Tools setup in Jenkins
```
1. Install java: name:jdk17 --> Install automatically --> add installer --> adoptium dot net --> jdk-17.0.8.1+1
2. Install Sonaqube: SonarQube Scanner --> name:sonar-scanner--> keep default as it is
3. Install nodejs: name:nodejs16 -->version: nodejs 16.20.0
4. Install Dependency-Check: name:DP-Check --> Install automatically --> Install from github.com.
5. Install Docker: name:docker --> Install automatically --> Install from dockerhub.com --> version:latest
```

# Sonarqube configuration in Jenkins system:
```
Manage Jenkins --> Systems --> SonarQube installations --> name:soar-scanner --> URL: (provide sonarqube url) http://54.146.189.148:9000 --> Server authentication token: choose the token which we created previously --> apply and save
```
# email token:
```
goto gmail manager --> search for --> app password --> provide name / jenkins --> create token
```
zxhhflppiybstggevnymi

# Setup Email creds in jenkins:
```
Goto manage jenkins --> credentials --> type:username and password --> name:enter your email id --> password= password/secretekey of email --> description:mail-creds --> create
```
```
Goto manage jenkins --> systems --> search for email --> Extended E-mail Notification --> SMTP server=smtp.gmail.com --> port=465 --> click on advance --> credentials=mail-cred --> select the checkboxes (Use SSL, Use OAuth 2.0) --> scroll down --> search for Email-notification --> SMTP server=smtp.gmail.com --> click on advances --> select "Use SMTP Authentication" --> User Name=moula.cloud5@gmail.com --> password= enter the previosly generated password --> SMTP port=465 --> select the checkbox "Use SSL" --> select "Test configuration by sending test e-mail" --> enter your email id --> click on Test email --> apply --> before save, goto "default triggers" --> choose (Failure any, Always and Sucess) --> apply and save.
```

# Jenkins Complete pipeline
```
pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage ("clean workspace") {
            steps {
                cleanWs()
            }
        }
        stage ("Git checkout") {
            steps {
                git branch: 'main', url: 'https://github.com/Shaik-DevOpsLucky/starbucks.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=starbucks \
                    -Dsonar.projectKey=starbucks '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
                }
            } 
        }
        stage("Install NPM Dependencies") {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage ("Trivy File Scan") {
            steps {
                sh "trivy fs . > trivy.txt"
            }
        }
        stage ("Build Docker Image") {
            steps {
                sh "docker build -t starbucks ."
            }
        }
        stage ("Tag & Push to DockerHub") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker') {
                        sh "docker tag starbucks moulashaik9618/starbucks:latest "
                        sh "docker push moulashaik9618/starbucks:latest "
                    }
                }
            }
        }
        stage('Docker Scout Image') {
            steps {
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       sh 'docker-scout quickview moulashaik9618/starbucks:latest'
                       sh 'docker-scout cves moulashaik9618/starbucks:latest'
                       sh 'docker-scout recommendations moulashaik9618/starbucks:latest'
                   }
                }
            }
        }
        stage ("Deploy to Conatiner") {
            steps {
                sh 'docker run -d --name starbucks -p 3000:3000 moulashaik9618/starbucks:latest'
            }
        }
    }
    post {
    always {
        emailext attachLog: true,
            subject: "'${currentBuild.result}'",
            body: """
                <html>
                <body>
                    <div style="background-color: #FFA07A; padding: 10px; margin-bottom: 10px;">
                        <p style="color: white; font-weight: bold;">Project: ${env.JOB_NAME}</p>
                    </div>
                    <div style="background-color: #90EE90; padding: 10px; margin-bottom: 10px;">
                        <p style="color: white; font-weight: bold;">Build Number: ${env.BUILD_NUMBER}</p>
                    </div>
                    <div style="background-color: #87CEEB; padding: 10px; margin-bottom: 10px;">
                        <p style="color: white; font-weight: bold;">URL: ${env.BUILD_URL}</p>
                    </div>
                </body>
                </html>
            """,
            to: 'provide_your_Email_id_here',
            mimeType: 'text/html',
            attachmentsPattern: 'trivy.txt'
        }
    }
}

```
