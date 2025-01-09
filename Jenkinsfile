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
                // Checkout the source code from GitHub
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [
                        [url: 'https://github.com/tohidhanfi20/Portfolio-Webapp.git', 
                         credentialsId: 'github-cred']
                    ]
                ])
            }
        }

        stage('Build with Maven') {
            steps {
                // Build the application with Maven
                sh 'mvn clean install'
            }
        }

        stage('Deploy WAR using Ansible') {
            steps {
                script {
                    // Ensure Ansible deploys the WAR file correctly
                    sh """
                    ansible-playbook -i 65.0.101.94, --private-key=/home/ansible/.ssh/id_rsa \
                    /home/ansible/deploy-war.yml --extra-vars 'war_file=${WAR_FILE} tomcat_webapps_dir=/usr/local/tomcat/webapps/'
                    """
                }
            }
        }

        stage('Build Docker Image for Tomcat') {
            steps {
                script {
                    // Build the Docker image using the Dockerfile in the repository
                    sh 'docker build -t web-app:latest .'
                }
            }
        }

        stage('Push Docker Image to Registry') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASSWORD')]) {
                    // Log in to Docker registry and push the image
                    sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USER --password-stdin ${DOCKER_REGISTRY}"
                    sh 'docker push ${DOCKER_REGISTRY}/web-app:latest'
                }
            }
        }

        stage('Deploy to Docker Container') {
            steps {
                script {
                    // Deploy the Docker container on the remote EC2 instance
                    sh """
                    ssh -i /var/lib/jenkins/.ssh/id_rsa ec2-user@65.0.101.94 \
                    'docker run -d --name ${TOMCAT_CONTAINER_NAME} -p 8080:8080 ${DOCKER_REGISTRY}/web-app:latest'
                    """
                }
            }
        }
    }

    post {
        always {
            // Clean up unused Docker images
            sh 'docker system prune -f'
        }
    }
}
