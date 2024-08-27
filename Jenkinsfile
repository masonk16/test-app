pipeline {
    agent any

    stages {
        stage("Clone Code") {
            steps {
                echo "Cloning the code"
                git url: "https://github.com/masonk16/test-app.git", branch: "main"
            }
        }

        stage("Build") {
            steps {
                echo "Building the Docker image"
                sh "docker build -t test-app ."
            }
        }

        stage("Push to Docker Hub") {
            steps {
                echo "Pushing image to Docker Hub"
                withCredentials([usernamePassword(credentialsId: "dockerHub", passwordVariable: "testing123", usernameVariable: "masondci")]) {
                    sh "docker tag test-app ${env.dockerHubUser}/test-app:latest"
                    sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}"
                    sh "docker push ${env.dockerHubUser}/test-app:latest"
                }
            }
        }

        stage ("Deploy") {
            steps {
                echo "Deploying the container"
                sh "docker-compose down && docker-compose up -d"
            }
        }
    }
}