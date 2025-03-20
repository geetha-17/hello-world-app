pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')  // DockerHub credentials
        GIT_CREDENTIALS = credentials('git-credentials')             // Git credentials for manifests repo
        IMAGE_NAME = "geetha-17/hello-world"                         // Replace with your DockerHub username
        MANIFESTS_REPO = "https://github.com/geetha-17/k8s-manifests.git"  // Your manifests repo
    }
    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .'
                    sh 'docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:latest'
                }
            }
        }
        stage('Push to DockerHub') {
            steps {
                script {
                    sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                    sh 'docker push ${IMAGE_NAME}:${BUILD_NUMBER}'
                    sh 'docker push ${IMAGE_NAME}:latest'
                }
            }
        }
        stage('Update Manifests') {
            steps {
                script {
                    // Clone the manifests repo
                    sh 'git clone ${MANIFESTS_REPO} manifests'
                    dir('manifests') {
                        // Update the image tag in deployment.yaml
                        sh '''
                        sed -i "s|image: ${IMAGE_NAME}:.*|image: ${IMAGE_NAME}:${BUILD_NUMBER}|g" deployment.yaml
                        git config user.email "jenkins@example.com"
                        git config user.name "Jenkins"
                        git add deployment.yaml
                        git commit -m "Update image to ${IMAGE_NAME}:${BUILD_NUMBER}" || echo "No changes to commit"
                        git push https://${GIT_CREDENTIALS_USR}:${GIT_CREDENTIALS_PSW}@github.com/geetha-17/k8s-manifests.git HEAD:main
                        '''
                    }
                }
            }
        }
    }
    post {
        always {
            sh 'docker logout'
            cleanWs()  // Clean workspace after run
        }
    }
}
