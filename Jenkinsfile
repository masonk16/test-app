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
                withCredentials([usernamePassword(credentialsId: "docker", passwordVariable: "dockerPass", usernameVariable: "dockerUser")]) {
                    sh "docker tag test-app ${env.dockerUser}/test-app:latest"
                    sh "echo ${env.dockerPass} | docker login -u ${env.dockerUser} --password-stdin"
                    sh "docker push ${env.dockerUser}/test-app:latest"
                }
            }
        }

        stage ("Deploy") {
            steps {
                echo "Deploying the container"
                sh "docker compose down && docker compose up -d"
            }
        }
    }
}