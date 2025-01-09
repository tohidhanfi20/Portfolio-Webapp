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
                sh 'mvn clean install'
            }
        }

        stage('Deploy WAR using Ansible') {
            steps {
                script {
                    sh """
                    ansible-playbook -i 65.0.101.94, --private-key=/home/ansible/.ssh/id_rsa \
                    /home/ansible/deploy-war.yml --extra-vars 'war_file=/root/Portfolio-webapp/target/web-app.war tomcat_webapps_dir=/usr/share/tomcat/webapps/'

                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t my-app:latest .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USER --password-stdin your-docker-registry-url"
                    sh 'docker push your-docker-registry-url/my-app:latest'
                }
            }
        }

        stage('Deploy to Docker Container') {
            steps {
                sh """
                ssh -i /var/lib/jenkins/.ssh/id_rsa ec2-user@65.0.101.94 'docker run -d --name my-app-container -p 8080:8080 my-app:latest'
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
