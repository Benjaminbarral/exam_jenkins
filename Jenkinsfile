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
                    echo $DOCKER_PASS | docker login -u $DOCKER_ID --password-stdin
                    docker push $DOCKER_ID/$MOVIE_IMAGE:$DOCKER_TAG
                    docker push $DOCKER_ID/$CAST_IMAGE:$DOCKER_TAG
                    '''
                }
            }
        }
        stage('Deploy to Kubernetes') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    cp charts/values.yaml values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install movie charts --values=values.yml --namespace dev
                    helm upgrade --install cast charts --values=values.yml --namespace dev
                    '''
                }
            }
        }
    }
    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs for more details.'
        }
    }
}
