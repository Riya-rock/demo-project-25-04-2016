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
        script {
            def scannerHome = tool 'sonar-scanner'
            withSonarQubeEnv('sonarqube') {
                sh """
                ${scannerHome}/bin/sonar-scanner \
                -Dsonar.projectKey=riya-todo-app \
                -Dsonar.sources=. \
                -Dsonar.projectName=riya-todo-app
                """
            }
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
