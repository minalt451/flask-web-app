pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
        GITHUB_CREDENTIALS = credentials('github-creds')
        IMAGE_NAME = "tambeminal/flask-app"
        APP_REPO = "https://github.com/minalt451/flask-web-app.git"
        MANIFESTS_REPO = "https://github.com/minalt451/flask-web-manifests.git"
    }

    stages {

        stage('Clone Application Repository') {
            steps {
                git branch: 'main', url: "${APP_REPO}"
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                script {
                    env.IMAGE_TAG = "${BUILD_NUMBER}"

                    sh """
                        docker login -u ${DOCKERHUB_CREDENTIALS_USR} -p ${DOCKERHUB_CREDENTIALS_PSW}
                        docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Update Manifests Repo') {
            steps {
                script {
                    // Clone manifests repo
                    sh """
                        rm -rf manifests || true
                        git clone ${MANIFESTS_REPO} manifests
                    """

                    dir("manifests") {

                        // Update image tag in deployment.yaml
                        sh """
                            sed -i "s|image: .*|image: ${IMAGE_NAME}:${IMAGE_TAG}|g" deployment.yaml
                        """

                        // Configure Git and push with credentials
                        withCredentials([usernamePassword(credentialsId: 'github-creds', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
                            sh """
                                git config user.name "jenkins"
                                git config user.email "jenkins@ci.com"

                                git add deployment.yaml
                                git commit -m "Update image tag to ${IMAGE_TAG}" || true

                                git remote set-url origin https://${GIT_USER}:${GIT_PASS}@github.com/minalt451/flask-web-manifests.git
                                git push origin main
                            """
                        }
                    }
                }
            }
        }
    }
}
