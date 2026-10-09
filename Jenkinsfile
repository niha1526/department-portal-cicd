pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'niharikarao15/department-portal'
        K8S_DEPLOYMENT = 'department-portal'
        K8S_CONTAINER = 'department-portal'
        K8S_NAMESPACE = 'default'
    }

    stages {
        stage('Clone Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${BUILD_NUMBER}")
                }
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
                withCredentials([file(
                    credentialsId: 'kubeconfig',
                    variable: 'KUBECONFIG_FILE'
                )]) {
                    withEnv(["KUBECONFIG=${KUBECONFIG_FILE}"]) {
                        bat '''
                            kubectl set image deployment/%K8S_DEPLOYMENT% %K8S_CONTAINER%=%DOCKER_IMAGE%:%BUILD_NUMBER% -n %K8S_NAMESPACE%
                            kubectl rollout status deployment/%K8S_DEPLOYMENT% -n %K8S_NAMESPACE% --timeout=180s
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the stage logs for details.'
        }
    }
}