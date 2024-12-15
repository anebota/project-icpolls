pipeline {
    agent { label 'agent1' }  // Replace 'agent1' with the label of your agent

    environment {
        GITHUB_REPO_URL = 'https://github.com/anebota/project-icpolls.git'
        BRANCH_NAME = 'main'  // Replace with your branch name if it's not 'main'
        GITHUB_CREDENTIALS_ID = 'github_access'  // Replace with your Jenkins GitHub credentials ID
        DOCKERHUB_CREDENTIALS_ID = 'dockerhub_cred'  // Replace with your Jenkins Docker Hub credentials ID
        DOCKERHUB_REPO = 'megastellas/zenithit-app'  // Replace with your Docker Hub repository
        UNIQUE_CONTAINER_NAME = "techvista-app-${BUILD_ID}"  // Generate unique container name
        CONTAINER_PORT = '8080'  // Change this as per the application requirements
        BASE_PORT = 8000  // Base port to avoid conflicts
        HOST_PORT = "${BASE_PORT + (BUILD_NUMBER % 1000)}"  // Ensure valid port within the range 8000-8999
    }

    stages {
        stage('Agent Details') {
            steps {
                echo "Running on agent: ${env.NODE_NAME}"
                sh 'uname -a'  // Print system information
                sh 'whoami'    // Print the current user
            }
        }

        stage('Clone Repository') {
            steps {
                git branch: "${env.BRANCH_NAME}", url: "${env.GITHUB_REPO_URL}", credentialsId: "${env.GITHUB_CREDENTIALS_ID}"
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'  // Simple Maven build
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    sh 'sudo docker --version'  // Verify Docker installation
                    sh "sudo docker build -t ${env.DOCKERHUB_REPO}:latest ."  // Build Docker image
                }
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "${env.DOCKERHUB_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh 'sudo docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD'
                        sh "sudo docker push ${env.DOCKERHUB_REPO}:latest"
                        sh 'sudo docker logout'
                    }
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    sh "sudo docker run --name ${env.UNIQUE_CONTAINER_NAME} --rm -d -p ${env.HOST_PORT}:${env.CONTAINER_PORT} ${env.DOCKERHUB_REPO}:latest"
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up Docker containers and images...'
            sh 'sudo docker ps -a --filter "name=techvista-app" -q | xargs --no-run-if-empty docker stop || true'
            sh 'sudo docker ps -a --filter "name=techvista-app" -q | xargs --no-run-if-empty docker rm || true'
            sh 'sudo docker rmi $(docker images -q) || true'
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
