pipeline {
    agent any

    environment {
    IMAGE = "sravyachinthakunta/myapp"
    TAG = "${BUILD_NUMBER}"
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
                    
                    docker build -t $IMAGE:$TAG -f Dockerfile .
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
                --namespace dev --create-namespace \
                --set image.repository=$IMAGE \
                --set image.tag=$TAG
                '''
            }
        }

        stage('Verify') {
            steps {
                sh '''
                kubectl get pods -n dev
                kubectl get svc -n dev
                kubectl get ingress -n dev
                '''
            }
        }
    }
}
