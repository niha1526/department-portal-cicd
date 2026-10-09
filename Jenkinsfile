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
        withCredentials([usernamePassword(
            credentialsId: 'dockerhub-credentials',
            usernameVariable: 'DOCKER_USER',
            passwordVariable: 'DOCKER_TOKEN'
        )]) {
            bat '''
                @echo off
                echo %DOCKER_TOKEN% | "%DOCKER_EXE%" login -u "%DOCKER_USER%" --password-stdin
                if errorlevel 1 exit /b 1

                "%DOCKER_EXE%" push "%DOCKER_IMAGE%:%BUILD_NUMBER%"
                if errorlevel 1 exit /b 1

                "%DOCKER_EXE%" tag "%DOCKER_IMAGE%:%BUILD_NUMBER%" "%DOCKER_IMAGE%:latest"
                if errorlevel 1 exit /b 1

                "%DOCKER_EXE%" push "%DOCKER_IMAGE%:latest"
                if errorlevel 1 exit /b 1
            '''
        }
    }
}

        stage('Deploy to Kubernetes') {
    steps {
        bat '''
            "%KUBECTL_EXE%" set image deployment/department-portal department-portal=%DOCKER_IMAGE%:%BUILD_NUMBER% -n default
            if errorlevel 1 exit /b 1

            "%KUBECTL_EXE%" rollout status deployment/department-portal -n default --timeout=180s
            if errorlevel 1 exit /b 1

            "%KUBECTL_EXE%" get deployment department-portal -n default
        '''
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
