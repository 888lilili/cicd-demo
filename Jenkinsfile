pipeline {
    agent any

    stages {
        stage('Build Image') {
            steps {
                sh 'docker build -t cicd-demo:${BUILD_NUMBER} .'
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                    docker rm -f cicd-demo || true
                    docker run -d \
                      --name cicd-demo \
                      -p 18080:80 \
                      cicd-demo:${BUILD_NUMBER}
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
