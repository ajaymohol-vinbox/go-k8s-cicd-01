pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'ajaymoholvinbox/go-k8s-app'
        DOCKER_TAG   = 'latest'
        KUBE_NAMESPACE = 'default'
    }

    triggers {
        githubPush()
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
                  echo "Building Docker image..."
                  docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withDockerRegistry(
                    credentialsId: 'dockerhub-credentials',
                    url: 'https://index.docker.io/v1/'
                ) {
                    sh '''
                      echo "Pushing Docker image..."
                      docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
    steps {
        sh '''
          echo "Deploying to Kubernetes..."
          kubectl apply -f k8s/
          kubectl rollout restart deployment/go-app
          kubectl rollout status deployment/go-app --timeout=120s
        '''
    }
}

    }

    post {
        success {
            echo "🎉 Deployed successfully to kubeadm cluster!"
        }
        failure {
            echo "❌ Deployment failed. Check logs."
        }
    }
}
