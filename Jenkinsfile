pipeline {
    agent any

    environment {
        IMAGE = "sravyachinthakunta/my-app"
        TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Build') {
            steps {
                sh 'cd app && docker build -t $IMAGE:$TAG .'
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
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                helm upgrade --install my-app ./helm-chart \
                --set image.repository=$IMAGE \
                --set image.tag=$TAG
                '''
            }
        }

        stage('Verify') {
            steps {
                sh 'kubectl get pods'
                sh 'kubectl get svc'
                sh 'kubectl get ingress'
            }
        }
    }
}
