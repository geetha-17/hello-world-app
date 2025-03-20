pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('41380629-0631-44da-ba4d-add2b73c5926')  // DockerHub credentials
        GIT_CREDENTIALS = credentials('b213f9bd-9d94-4318-aec9-aabb24118bfd')             // Git credentials for manifests repo
        IMAGE_NAME = "geetha8500/hello-world"
        MANIFESTS_REPO = "https://github.com/geetha-17/k8s-manifests.git"             // e.g., https://github.com/<username>/k8s-manifests.git
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
