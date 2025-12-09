pipeline {
agent any

environment {  
    DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')  
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
                sh """  
                    rm -rf manifests || true  
                    git clone ${MANIFESTS_REPO} manifests  
                    cd manifests  

                    sed -i "s|image: .*|image: ${IMAGE_NAME}:${IMAGE_TAG}|g" deployment.yaml  

                    git config user.name "jenkins"  
                    git config user.email "jenkins@ci.com"  

                    git add deployment.yaml  
                    git commit -m "Update image tag to ${IMAGE_TAG}" || true  
                    git push origin main  
                """  
            }  
        }  
    }  
}

}
