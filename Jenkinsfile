pipeline {
    agent any

    options {
        disableConcurrentBuilds()
    }

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

        stage('Push Image') {
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

        stage('Update GitOps') {
            steps {
                sh '''
                    rm -rf gitops

                    git clone \
                      git@github-gitops:888lilili/cicd-demo-gitops.git \
                      gitops

                    cd gitops

                    git config user.name "jenkins"
                    git config user.email "jenkins@master"

                    sed -i -E \
                      "s#image: ghcr.io/888lilili/cicd-demo:[^[:space:]]+#image: ${IMAGE}:${BUILD_NUMBER}#" \
                      k8s/deploy.yaml

                    echo "修改后的镜像："
                    grep 'image:' k8s/deploy.yaml

                    git add k8s/deploy.yaml
                    git commit -m "Deploy cicd-demo:${BUILD_NUMBER}"
                    git push origin main
                '''
            }
        }
    }
}
