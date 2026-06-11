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
<<<<<<< HEAD
        withCredentials([string(credentialsId: 'SONAR_AUTH_TOKEN', variable: 'SONAR_TOKEN')]) {
            sh '''
            /opt/sonar-scanner/bin/sonar-scanner \
            -Dsonar.projectKey=riya-todo-app \
            -Dsonar.projectName=riya-todo-app \
            -Dsonar.sources=. \
            -Dsonar.host.url=http://localhost:9000 \
            -Dsonar.token=$SONAR_TOKEN
            '''
        }
=======
        writeFile file: 'sonar-project.properties', text: '''
stage('Generate SBOM') {
    steps {
        sh '''
        python3 -m pip install --user cyclonedx-bom
        python3 -m cyclonedx_py environment -o sbom.xml
        '''
    }
}

sonar.projectKey=riya-todo-app
sonar.projectName=riya-todo-app
sonar.sources=.
sonar.host.url=http://localhost:9000
'''
>>>>>>> 11e1bed (Fix SonarQube stage)
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
                sh 'trivy image --scanners vuln --severity HIGH,CRITICAL --exit-code 0 --no-progress riya-todo-app'
            }
        }

        stage('Secret scanning (TruffleHog)') {
            steps {
                sh 'docker run --rm -v "$PWD:/repo" trufflesecurity/trufflehog:latest filesystem /repo --exclude-paths=/repo/.git --no-update --only-verified'
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
