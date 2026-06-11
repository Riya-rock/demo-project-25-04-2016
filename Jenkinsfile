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
                    bat '''
                    vault write auth/approle/login role_id=%ROLE_ID% secret_id=%SECRET_ID%
                    vault kv get dev/testapp/sample
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t riya-todo-app .'
            }
        }

        stage('SonarQube Scan') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner'
                    withSonarQubeEnv('SonarQube') {
                        bat """
                        "${scannerHome}\\bin\\sonar-scanner.bat" ^
                        -Dsonar.projectKey=riya-todo-app ^
                        -Dsonar.projectName=riya-todo-app ^
                        -Dsonar.sources=. ^
                        -Dsonar.host.url=http://localhost:9000 ^
                        -Dsonar.token=sqa_76f586da93c23ab7a396fe99b9c6e3b57433d494
                        """
                    }
                }
            }
        }

        stage('Generate SBOM') {
            steps {
                bat '''
                python -m venv sbom-venv
                call sbom-venv\\Scripts\\activate
                pip install cyclonedx-bom
                cyclonedx-py environment -o sbom.xml
                '''
            }
        }

        stage('Trivy Container Scan') {
            steps {
                bat 'trivy image --scanners vuln --severity HIGH,CRITICAL --exit-code 0 --no-progress riya-todo-app'
            }
        }

        stage('Secret scanning (TruffleHog)') {
            steps {
                bat 'docker run --rm -v "%CD%:/repo" trufflesecurity/trufflehog:latest filesystem /repo --exclude-paths=/repo/.git --no-update --only-verified'
            }
        }

        stage('Run Container') {
            steps {
                bat 'docker rm -f riya-container || exit 0'
                bat 'docker run -d -p 8000:8000 --name riya-container riya-todo-app'
            }
        }
    }
}
