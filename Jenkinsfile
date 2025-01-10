pipeline {
    agent any
    environment {
        WAR_NAME = "web-app.war"
        ANSIBLE_SERVER = "localhost"
    }
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/your-username/your-repo.git'
            }
        }

        stage('Build WAR') {
            steps {
                sh 'mvn clean package'
            }
            post {
                success {
                    archiveArtifacts artifacts: 'target/*.war', fingerprint: true
                }
            }
        }

        stage('Copy WAR to Docker Context') {
            steps {
                sshPublisher(publishers: [
                    sshPublisherDesc(
                        configName: ANSIBLE_SERVER,
                        transfers: [
                            sshTransfer(
                                sourceFiles: "target/${WAR_NAME}",
                                removePrefix: "target",
                                remoteDirectory: "/tmp"
                            )
                        ]
                    )
                ])
                sh '''
                ansible-playbook deploy-war.yml -i localhost,
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t web-app-image .
                '''
            }
        }

        stage('Run Docker Container') {
            steps {
                sh '''
                docker run -d -p 8081:8080 --name web-app-container web-app-image
                '''
            }
        }
    }
}
