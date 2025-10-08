pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "syamks8/myflaskapp"       // Docker Hub image name
        DOCKER_CREDENTIALS_ID = "dockerhub-creds" // DockerHub credentials in Jenkins
        KUBE_NAMESPACE = "default"               // Kubernetes namespace
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-creds',
                    url: 'https://github.com/<your-github-username>/k8s-flask-jenkins-demo.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKER_IMAGE}:${BUILD_NUMBER}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS_ID}") {
                        dockerImage.push()
                        dockerImage.push("latest")
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Replace IMAGE_PLACEHOLDER in deployment.yaml with current Docker image
                    sh """
                    sed -i 's|IMAGE_PLACEHOLDER|${DOCKER_IMAGE}:${BUILD_NUMBER}|g' k8s/deployment.yaml
                    kubectl apply -f k8s/deployment.yaml -n ${KUBE_NAMESPACE}
                    kubectl rollout status deployment/my-flask-app -n ${KUBE_NAMESPACE}
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment Successful! Access at NodePort 30080"
        }
        failure {
            echo "❌ Deployment Failed!"
        }
    }
}
