pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'niharikarao15/department-portal'
        DOCKER_EXE = 'C:\\Users\\Niharika\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe'
        KUBECTL_EXE = 'C:\\Users\\Niharika\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\kubectl.exe'
        KUBECONFIG = 'C:\\Users\\Niharika\\.kube\\config'
    }

    stages {
        stage('Clone Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                bat '"%DOCKER_EXE%" build -t "%DOCKER_IMAGE%:%BUILD_NUMBER%" .'
            }
        }

        stage('Push Image to Docker Hub') {
    steps {
        script {
            docker.withRegistry(
                'https://index.docker.io/v1/',
                'dockerhub-credentials'
            ) {
                docker.image("${DOCKER_IMAGE}:${BUILD_NUMBER}").push()
                docker.image("${DOCKER_IMAGE}:${BUILD_NUMBER}").push('latest')
            }
        }
    }
}

        stage('Deploy to Kubernetes') {
            steps {
                bat '''
                    "%KUBECTL_EXE%" set image deployment/department-portal department-portal=%DOCKER_IMAGE%:%BUILD_NUMBER% -n default
                    "%KUBECTL_EXE%" rollout status deployment/department-portal -n default --timeout=180s
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Review the console output.'
        }
    }
}
