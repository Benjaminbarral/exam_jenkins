pipeline {
    environment {
        DOCKER_ID = "benjaminbarral" // DockerHub username
        MOVIE_IMAGE = "jenkins_movie_service"
        CAST_IMAGE = "jenkins_cast_service"
        DOCKER_TAG = "v.${BUILD_ID}.0" // Tag basé sur le numéro de build
    }
    agent any
    stages {
        stage('Docker Build') {
            steps {
                script {
                    sh '''
                    # Build des images Docker pour les services
                    docker build -t $DOCKER_ID/$MOVIE_IMAGE:$DOCKER_TAG ./movie-service
                    docker build -t $DOCKER_ID/$CAST_IMAGE:$DOCKER_TAG ./cast-service
                    '''
                }
            }
        }
        stage('Docker Push') {
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }
            steps {
                script {
                    sh '''
                    # Push des images Docker vers DockerHub
                    echo $DOCKER_PASS | docker login -u $DOCKER_ID --password-stdin
                    docker push $DOCKER_ID/$MOVIE_IMAGE:$DOCKER_TAG
                    docker push $DOCKER_ID/$CAST_IMAGE:$DOCKER_TAG
                    '''
                }
            }
        }
        stage('Deploy to Kubernetes in dev') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    # Configuration de l'accès Kubernetes
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config

                    # Mise à jour du fichier values.yaml
                    cp charts/values.yaml values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml

                    # Déploiement avec Helm pour dev
                    helm upgrade --install movie ./charts --values=values.yml --namespace dev --set serviceName=movie
                    helm upgrade --install cast ./charts --values=values.yml --namespace dev --set serviceName=cast
                    '''
                }
            }
        }
        stage('Deploy to Kubernetes in qa') {
            steps {
                script {
                    sh '''
                    # Déploiement avec Helm pour qa
                    helm upgrade --install movie ./charts --values=values.yml --namespace qa --set serviceName=movie
                    helm upgrade --install cast ./charts --values=values.yml --namespace qa --set serviceName=cast
                    '''
                }
            }
        }
        stage('Deploy to Kubernetes in staging') {
            steps {
                script {
                    sh '''
                    # Déploiement avec Helm pour staging
                    helm upgrade --install movie ./charts --values=values.yml --namespace staging --set serviceName=movie
                    helm upgrade --install cast ./charts --values=values.yml --namespace staging --set serviceName=cast
                    '''
                }
            }
        }
        stage('Deploy to Kubernetes in prod') {
            steps {
                script {
                    timeout(time: 15, unit: "MINUTES") {
                        input message: 'Do you want to deploy in production?', ok: 'Deploy'
                    }
                    sh '''
                    # Déploiement avec Helm pour prod
                    helm upgrade --install movie ./charts --values=values.yml --namespace prod --set serviceName=movie
                    helm upgrade --install cast ./charts --values=values.yml --namespace prod --set serviceName=cast
                    '''
                }
            }
        }
    }
    post {
        success {
            echo 'Pipeline completed successfully for all namespaces!'
        }
        failure {
            echo 'Pipeline failed. Check logs for more details.'
        }
    }
}
