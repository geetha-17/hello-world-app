pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')  // DockerHub credentials
        GIT_CREDENTIALS = credentials('git-credentials')             // Git credentials for manifests repo
        IMAGE_NAME = "<your-dockerhub-username>/hello-world"
        MANIFESTS_REPO = "<your-manifests-repo-url>.git"             // e.g., https://github.com/<username>/k8s-manifests.git
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
                        sh """
                        sed -i 's|image: ${IMAGE_NAME}:.*|image: ${IMAGE_NAME}:${BUILD_NUMBER}|g' deployment.yaml
                        git config user.email "jenkins@example

.com"
                        git config user.name "Jenkins"
                        git add deployment.yaml
                        git commit -m "Update image to ${IMAGE_NAME}:${BUILD_NUMBER}"
                        git push https://$GIT_CREDENTIALS_USR:$GIT_CREDENTIALS_PSW@${MANIFESTS_REPO#https://} HEAD:main
                        """
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
