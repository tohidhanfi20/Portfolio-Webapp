pipeline {
    agent any
    
    environment {
        WAR_NAME = 'web-app.war'  // Name of your WAR file
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

        stage('Copy WAR to Root Directory') {
            steps {
                // Copy WAR file to root directory for Ansible deployment
                script {
                    sh "cp target/${WAR_NAME} /root/"
                }
            }
        }

        stage('Deploy WAR to Tomcat with Ansible') {
            steps {
                // Run Ansible playbook to deploy WAR file to Tomcat
                script {
                    sh '''
                    ansible-playbook /opt/ansible/deploy-war.yml -i localhost,
                    '''
                }
            }
        }

        stage('Docker Deployment') {
            steps {
                // Use Docker to deploy the WAR file (assuming your Dockerfile is at the root)
                script {
                    sh '''
                    docker build -t tomcat-app .
                    docker run -d -p 8082:8080 --name tomcat-container tomcat-app
                    '''
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
