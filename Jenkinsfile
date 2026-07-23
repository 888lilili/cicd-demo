pipeline {
    agent any

    environment {
        IMAGE = 'ghcr.io/888lilili/cicd-demo'
    }

    stages {
        stage('Build Image') {
            steps {
                sh '''
                    docker build \
                      -t ${IMAGE}:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Login and Push Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'ghcr-login',
                        usernameVariable: 'GHCR_USER',
                        passwordVariable: 'GHCR_TOKEN'
                    )
                ]) {
                    sh '''
                        printf '%s' "$GHCR_TOKEN" | \
                          docker login ghcr.io \
                          -u "$GHCR_USER" \
                          --password-stdin

                        docker push ${IMAGE}:${BUILD_NUMBER}

                        docker logout ghcr.io
                    '''
                }
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                    docker rm -f cicd-demo || true

                    docker run -d \
                      --name cicd-demo \
                      -p 18080:80 \
                      ${IMAGE}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Verify') {
            steps {
                sh '''
                    sleep 2
                    curl -fsS http://127.0.0.1:18080
                '''
            }
        }
    }
}
