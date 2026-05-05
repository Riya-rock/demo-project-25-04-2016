pipeline {
    agent any

    stages {
        stage('Clone') {
            steps {
                echo 'Code cloned'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t riya-todo-app .'
            }
        }

        stage('SonarQube Scan') {
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    sh '''
                    sonar-scanner \
                      -Dsonar.projectKey=riya-todo-app \
                      -Dsonar.sources=. \
                      -Dsonar.host.url=http://192.168.153.128:9000 \
                      -Dsonar.login=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage('Run Container') {
            steps {
                sh 'docker rm -f riya-container || true'
                sh 'docker run -d -p 8000:8000 --name riya-container riya-todo-app'
            }
        }
    }
}
