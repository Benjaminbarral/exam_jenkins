pipeline {
    environment {
        DOCKER_ID = "benjaminbarral"
        MOVIE_IMAGE = "jenkins_movie_service"
        CAST_IMAGE = "jenkins_cast_service"
        DOCKER_TAG = "v.${BUILD_ID}.0"
        KUBECONFIG = credentials("config") // Fichier kubeconfig depuis les credentials
    }
    agent any
    stages {
        stage('Prepare Kubernetes Config') {
            steps {
                script {
                    echo "Configuring Kubernetes access"
                    sh '''
                    mkdir -p ~/.kube
                    chmod 700 ~/.kube
                    cp ${KUBECONFIG} ~/.kube/config
                    chmod 600 ~/.kube/config
                    export KUBECONFIG=~/.kube/config
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
