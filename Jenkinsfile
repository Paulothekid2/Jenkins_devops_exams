pipeline {
    agent any

    environment {
        DOCKER_USER = 'paulothekid'
        CAST_IMAGE  = "${DOCKER_USER}/cast-service"
        MOVIE_IMAGE = "${DOCKER_USER}/movie-service"
        BUILD_TAG   = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Push Docker Images') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
                        sh "echo ${DH_PASS} | docker login -u ${DH_USER} --password-stdin"
                        
                        sh "docker build -t ${CAST_IMAGE}:${BUILD_TAG} -t ${CAST_IMAGE}:latest ./cast-service"
                        sh "docker push ${CAST_IMAGE}:${BUILD_TAG}"
                        sh "docker push ${CAST_IMAGE}:latest"

                        sh "docker build -t ${MOVIE_IMAGE}:${BUILD_TAG} -t ${MOVIE_IMAGE}:latest ./movie-service"
                        sh "docker push ${MOVIE_IMAGE}:${BUILD_TAG}"
                        sh "docker push ${MOVIE_IMAGE}:latest"
                    }
                }
            }
        }

        stage('Deploy to Dev') {
            steps {
                script { deployHelm('dev') }
            }
        }

        stage('Deploy to QA') {
            steps {
                script { deployHelm('qa') }
            }
        }

        stage('Deploy to Staging') {
            steps {
                script { deployHelm('staging') }
            }
        }

        stage('Manual Approval for Production') {
            when { branch 'master' }
            steps {
                input message: "Approuver le déploiement en PRODUCTION ?", ok: "Déployer"
            }
        }

        stage('Deploy to Prod') {
            when { branch 'master' }
            steps {
                script { deployHelm('prod') }
            }
        }
    }

    post {
        always {
            sh "docker logout"
        }
    }
}

def deployHelm(String targetNamespace) {
    sh """
        helm upgrade --install api-release ./charts \
            --namespace ${targetNamespace} \
            --set castService.image.repository=${CAST_IMAGE} \
            --set castService.image.tag=${BUILD_TAG} \
            --set movieService.image.repository=${MOVIE_IMAGE} \
            --set movieService.image.tag=${BUILD_TAG}
    """
}
