# Setting up the CI/CD Pipeline with Jenkins

- EC2
- Docker
- Jenkins
- GitHub
- Django

1. Initialize the GitHub repository
    - git init .
    - create the github repository
    - git remote add origin <remote_repository_url>
    - git pull origin main
    - git push -u origin main

2. Setup EC2:
    - Create EC2 with Ubuntu
    - Allow traffic to port 8080 (Inbound rules)
    - Install Jenkins
        ```shell
        sudo apt update

        sudo apt install fontconfig openjdk-21-jre

        sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian/jenkins.io-2023.key

        echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

        sudo apt-get update

        sudo apt-get install jenkins

        sudo systemctl enable jenkins

        sudo systemctl start jenkins

        sudo systemctl status jenkins
        ```
        - Configure the Jenkins Server
    - Install Docker
    - Create Dockerfile on local repository
    - Clone the app to EC2