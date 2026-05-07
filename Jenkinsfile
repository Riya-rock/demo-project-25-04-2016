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
                    def scannerHome = tool 'SonarScanner'
                    withSonarQubeEnv('SonarQube') {
                        sh """
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=riya-todo-app \
                        -Dsonar.sources=. \
                        -Dsonar.login=$SONAR_AUTH_TOKEN
                        """
                    }
                }
            }
        }

        stage('Generate SBOM') {
    steps {
        sh 'export PATH=$PATH:/home/riyamhatre/.local/bin && cyclonedx-py environment -o sbom.xml'
    }
}
 stage('Trivy Container Scan') {
            steps {
               sh 'trivy image --scanners vuln --severity HIGH,CRITICAL --exit-code 0 --no-progress dockeriya03/riya-todo-app:latest'
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
