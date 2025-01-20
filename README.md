jenkins-sonarqube
------------------------------
first create Docker containers 
take one new VM and execute below commands to install docker and run Jenkins and Sonar Quebe containers

sudo apt-get update
sudo apt-get update -y
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
sudo apt-get update -y
sudo apt-get install docker-ce docker-ce-cli containerd.io -y
sudo docker version
sudo docker pull jenkins/jenkins:lts
sudo docker run --name jenkins -d -p 8080:8080 -p 50000:50000 jenkins/jenkins:lts
sudo docker ps
sudo docker logs jenkins
sudo docker pull sonarqube
sudo docker run -d --name sonarqube -p 9000:9000 sonarqube

Jenkins Pipeline File
------------------------
pipeline {
    agent any
    tools{
        jdk 'jdk11'
        maven 'maven3'
    }

    environment {
        SONARQUBE_SERVER = 'sonar-scanner'  // Name of your SonarQube server configuration in Jenkins
        SONAR_TOKEN = credentials('sonar-token')  // SonarQube token stored in Jenkins credentials
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/praveen89j/testwebsite.git'
            }
        }

         
        stage('SonarQube Analysis') {
            steps {
                script {
                    // Running the SonarQube analysis using the SonarScanner
                    withSonarQubeEnv(SONARQUBE_SERVER) {
                        sh 'mvn sonar:sonar -Dsonar.url=http://20.51.231.88:9000/ -Dsonar.projectKey=my-html-project -Dsonar.sources=.'  // Adjust project key and sources as needed
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    // Wait for SonarQube analysis to complete and check the quality gate
                    def qualityGate = waitForQualityGate()
                    if (qualityGate.status != 'OK') {
                        error "Quality Gate failed. Analysis status: ${qualityGate.status}"
                    }
                }
            }
        }
    }

    post {
        always {
            // Clean up or other steps if necessary
            echo 'Pipeline completed'
        }
    }
}
----------------------------------------------
another pipeline file
---------------------------------------------
pipeline {
   agent any
    
    tools{
        jdk 'jdk11'
        maven 'maven3'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/praveen89j/testwebsite.git'
            }
        }
       
        stage("Sonarqube Analysis"){
            steps{
                withSonarQubeEnv('sonar-scanner') {
    			  sh "mvn clean compile"
                   sh '''mvn sonar:sonar -Dsonar.url=http://40.76.243.153:9000/ -Dsonar.projectName=Testwebsite \
                   -Dsonar.sources=src \
                   -Dsonar.login=squ_839ad9a2a626c9f9cd078c13748774a31743dd9c \
                   -Dsonar.java.binaries=. \
                   -Dsonar.projectKey=Testwebsite'''
    
                }
            }
        }
    }
    

}
