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
                -Dsonar.projectName=riya-todo-app \
                -Dsonar.sources=. \
                -Dsonar.host.url=http://localhost:9000 \
                -Dsonar.token=sqa_76f586da93c23ab7a396fe99b9c6e3b57433d494
                """
            }
        }
    }
}

        stage('Generate SBOM') {
    steps {
        sh '''
        python3 -m venv sbom-venv
        . sbom-venv/bin/activate
        pip install cyclonedx-bom
        cyclonedx-py environment -o sbom.xml
        '''
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
        stage('Secret scanning (TruffleHog)') {
    steps {
        sh '''
        docker run --rm -v "$PWD:/repo" trufflesecurity/trufflehog:latest filesystem /repo --exclude-paths=/repo/.git --no-update --only-verified
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
