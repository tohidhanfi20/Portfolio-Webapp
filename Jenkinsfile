pipeline {
    agent any

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
                    // Run Ansible to deploy the WAR file
                    sh """
                    ansible-playbook -i 65.0.101.94, --private-key=/home/ansible/.ssh/id_rsa \
                    /home/ansible/deploy-war.yml --extra-vars 'war_file=/root/Portfolio-webapp/target/web-app.war tomcat_webapps_dir=/usr/share/tomcat/webapps/'
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                // Build the Docker image
                sh 'docker build -t web-app:latest .'
            }
        }

        stage('Push Docker Image') {
            steps {
                // Push the Docker image to the Docker registry
                withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh """
                    echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USER --password-stdin your-docker-registry-url
                    docker push your-docker-registry-url/web-app:latest
                    """
                }
            }
        }

        stage('Deploy to Docker Container') {
    steps {
        sh """
        ssh -i /var/lib/jenkins/.ssh/id_rsa ec2-user@65.0.101.94 \
        'docker run -d --name web-app-container -p 8081:8080 web-app:latest'
        """
                }
             }

        }

    post {
        always {
            // Clean up Docker system
            sh 'docker system prune -f'
        }
    }
}
