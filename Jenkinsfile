pipeline {
    agent any

    environment {
        WAR_FILE = '/var/lib/jenkins/workspace/Portfolio-webapp/target/web-app.war'
        TOMCAT_CONTAINER_NAME = 'web-app-container'
        DOCKER_REGISTRY = 'docker.io/tohidaws'
    }

    stages {
        stage('Pull Code from GitHub') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [
                        [url: 'https://github.com/tohidhanfi20/Portfolio-Webapp.git', credentialsId: 'github-cred']
                    ]
                ])
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Deploy WAR using Ansible') {
            steps {
                script {
                    sh """
                    ansible-playbook -i 65.0.101.94, --private-key=/home/ansible/.ssh/id_rsa \
                    /home/ansible/deploy-war.yml --extra-vars 'war_file=${WAR_FILE} tomcat_container_name=${TOMCAT_CONTAINER_NAME}'
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t web-app:latest .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USER --password-stdin ${DOCKER_REGISTRY}"
                    sh 'docker push ${DOCKER_REGISTRY}/web-app:latest'
                }
            }
        }

        stage('Deploy to Docker Container') {
            steps {
                sh """
                docker run -d --name ${TOMCAT_CONTAINER_NAME} -p 8081:8080 web-app:latest
                """
            }
        }
    }

    post {
        always {
            sh 'docker system prune -f'
        }
    }
}
