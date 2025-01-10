pipeline {
    agent any
    
    tools {
        maven 'maven'
    }
    
    environment {
        WAR_NAME = 'web-app.war'
        IMAGE_NAME = 'webapp' 
    }

    stages {
        stage('Checkout Code from GitHub') {
            steps {
                // Checkout code from GitHub
                git credentialsId: 'git-cred', url: 'https://github.com/tohidhanfi20/Portfolio-Webapp.git', branch: 'main'
            }
        }
        stage('Build WAR with Maven') {
            steps {
                // Run Maven build to generate WAR file
                script {
                    sh 'mvn clean package'  // This will generate target/web-app.war
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    sh '''
                    docker build -t ${IMAGE_NAME} -f Dockerfile .
                    '''
                }
            }
        }
        stage('Run Docker Container') {
            steps {
                script {
                    sh 'docker run -d --name webapp_container -p 8090:8080 ${IMAGE_NAME}'
                }
            }
        }
    }
}
