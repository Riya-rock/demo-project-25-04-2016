pipeline {
    agent any

    environment {
        VAULT_ADDR = 'http://127.0.0.1:8200'
    }

    stages {

        stage('Clone') {
            steps {
                echo 'Code cloned'
            }
        }

        stage('Vault Login and Read Secret') {
    steps {
        withCredentials([
            string(credentialsId: 'VAULT_ROLE_ID', variable: 'ROLE_ID'),
            string(credentialsId: 'VAULT_SECRET_ID', variable: 'SECRET_ID')
        ]) {
            sh '''
            vault write auth/approle/login role_id=$ROLE_ID secret_id=$SECRET_ID
            vault kv get dev/testapp/sample
            '''
        }
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
                -Dsonar.host.url=$SONAR_HOST_URL \
                -Dsonar.login=$SONAR_AUTH_TOKEN
                """
            }
        }
    }
}

        stage('Generate SBOM') {
            steps {
                sh '''
                python -m venv sbom-venv
                call sbom-venv\\Scripts\\activate
                pip install cyclonedx-bom
                cyclonedx-py environment -o sbom.xml
                '''
            }
        }

        stage('Trivy Container Scan') {
            steps {
                sh 'trivy image --scanners vuln --severity HIGH,CRITICAL --exit-code 0 --no-progress riya-todo-app'
            }
        }

        stage('Secret scanning (TruffleHog)') {
            steps {
                sh 'docker run --rm -v "%CD%:/repo" trufflesecurity/trufflehog:latest filesystem /repo --exclude-paths=/repo/.git --no-update --only-verified'
            }
        }

        stage('Run Container') {
            steps {
                sh 'docker rm -f riya-container || exit 0'
                sh 'docker run -d -p 8000:8000 --name riya-container riya-todo-app'
            }
        }
    }
}
