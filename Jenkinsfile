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
                    echo "Building Docker images for movie and cast services"
                    sh '''
                    docker build -t $DOCKER_ID/$MOVIE_IMAGE:$DOCKER_TAG ./movie-service
                    docker build -t $DOCKER_ID/$CAST_IMAGE:$DOCKER_TAG ./cast-service
                    '''
                }
            }
        }
        stage('Docker Push') {
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // Utilisation du credential Jenkins
            }
            steps {
                script {
                    echo "Pushing Docker images to DockerHub"
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_ID --password-stdin
                    docker push $DOCKER_ID/$MOVIE_IMAGE:$DOCKER_TAG
                    docker push $DOCKER_ID/$CAST_IMAGE:$DOCKER_TAG
                    '''
                }
            }
        }
        stage('Prepare Kubernetes Config') {
            environment {
                KUBECONFIG = credentials("config") // Fichier kubeconfig depuis les credentials Jenkins
            }
            steps {
                script {
                    echo "Configuring Kubernetes access"
                    sh '''
                    mkdir -p ~/.kube
                    chmod 700 ~/.kube
                    echo "$KUBECONFIG" > ~/.kube/config
                    chmod 600 ~/.kube/config
                    '''
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Liste des namespaces cibles
                    def namespaces = ['dev', 'qa', 'staging', 'prod']
                    for (namespace in namespaces) {
                        echo "Deploying to ${namespace} namespace"
                        deployHelm(namespace)
                    }
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

def deployHelm(namespace) {
    sh """
    # Vérification et création du namespace si nécessaire
    kubectl create namespace ${namespace} || true

    # Mise à jour du fichier values.yml avec le tag Docker
    cp charts/values.yaml values.yml
    sed -i "s+tag.*+tag: ${env.DOCKER_TAG}+g" values.yml

    # Déploiement avec Helm
    helm upgrade --install movie ./charts --values=values.yml --namespace ${namespace} --set serviceName=movie
    helm upgrade --install cast ./charts --values=values.yml --namespace ${namespace} --set serviceName=cast
    """
}
