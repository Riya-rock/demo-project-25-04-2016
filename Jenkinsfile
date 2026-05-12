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

        stage('Create Sonar Properties') {
    steps {
        writeFile file: 'sonar-project.properties', text: '''
sonar.projectKey=riya-todo-app
sonar.projectName=riya-todo-app
sonar.sources=.
sonar.host.url=http://localhost:9000
'''
    }
}

        stage('Generate SBOM') {
            steps {
                sh 'cyclonedx-py environment -o sbom.xml'
            }
        }

        stage('Trivy Container Scan') {
            steps {
                sh '''
                trivy image \
                --scanners vuln \
                --severity HIGH,CRITICAL \
                --exit-code 0 \
                --no-progress \
                riya-todo-app
                '''
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
