pipeline {
    agent any

    environment {
        IMAGE = "sravyachinthakunta/my-app"
        TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $IMAGE:$TAG .
                docker tag $IMAGE:$TAG $IMAGE:latest
                '''
            }
        }

        stage('Login to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-cred',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    '''
                }
            }
        }

        stage('Push Image') {
            steps {
                sh '''
                docker push $IMAGE:$TAG
                docker push $IMAGE:latest
                '''
            }
        }
        stage('Check Helm') {
            steps {
                sh '''
                export PATH=$PATH:/var/jenkins_home/bin

                echo "Checking Helm installation..."
                which helm
                helm version
                '''
            }
        }


        stage('Helm Deploy') {
            steps {
                sh '''
                export PATH=$PATH:/var/jenkins_home/bin

                echo "Starting Helm deployment..."

                helm upgrade --install my-app ./helm-chart \
                  --set image.repository=sravyachinthakunta/my-app \
                  --set image.tag=20
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                kubectl get pods
                kubectl get svc
                kubectl get ingress
                '''
            }
        }
    }
}
