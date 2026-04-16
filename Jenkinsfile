pipeline {
    agent any

    environment {
        IMAGE = "sravyachinthakunta/my-app"
        TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Check Tools') {
            steps {
                sh '''
                echo "Checking tools..."
                docker version || exit 1
                helm version || echo "Helm missing"
                kubectl version --client || echo "kubectl missing"
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                echo "Building Docker image..."
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
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Push Image') {
            steps {
                sh '''
                echo "Pushing Docker image..."
                docker push $IMAGE:$TAG
                docker push $IMAGE:latest
                '''
            }
        }

        stage('Helm Deploy') {
            steps {
                sh '''
                echo "Deploying with Helm..."

                helm upgrade --install my-app ./helm-chart \
                  --set image.repository=$IMAGE \
                  --set image.tag=$TAG
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                kubectl get pods || true
                kubectl get svc || true
                kubectl get ingress || true
                '''
            }
        }
    }

    post {
        failure {
            echo "Pipeline Failed ❌"
        }
        success {
            echo "Pipeline Success 🚀"
        }
    }
}
