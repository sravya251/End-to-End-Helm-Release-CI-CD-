pipeline {
    agent any

    environment {
        IMAGE = "sravyachinthakunta/myapp"
        TAG = "${BUILD_NUMBER}"
        NAMESPACE = "dev"
    }

    stages {

        stage('Build') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-cred',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin

                    docker build -t $IMAGE:$TAG .
                    docker tag $IMAGE:$TAG $IMAGE:latest
                    '''
                }
            }
        }

        stage('Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-cred',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin

                    docker push $IMAGE:$TAG
                    docker push $IMAGE:latest
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                aws eks update-kubeconfig --region ap-south-1 --name eks-cluster

                helm upgrade --install myapp ./helm-chart \
                --namespace $NAMESPACE --create-namespace \
                --set image.repository=$IMAGE \
                --set image.tag=$TAG
                '''
            }
        }

        stage('Verify') {
            steps {
                sh '''
                kubectl get pods -n $NAMESPACE
                kubectl get svc -n $NAMESPACE
                kubectl get ingress -n $NAMESPACE
                kubectl get hpa -n $NAMESPACE
                '''
            }
        }
    }
}
