# Setting up the CI/CD Pipeline with Jenkins

- EC2
- Docker
- Jenkins
- GitHub
- Django

1. Initialize the GitHub repository
    - pip freeze > requirements.txt
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

        ```shell
        sudo apt-get update -y

        # Install docker
        sudo apt-get install docker.io
        
        # add docker user to the same user group as root user
        sudo usermod -aG docker $USER

        # Install docker compose
        sudo apt-get install docker-compose

        # Create username and password for DockerHub
        docker login
        ```

    - Create Dockerfile on local repository

        ```
        # Use Python 3.9 as the base image
        FROM python:3:9

        # Set the working directory within the container
        WORKDIR /app/test-app

        # Copy the requirements.txt file to the container
        COPY requirements.txt /app/test-app

        # Install dependencies using pip
        RUN pip install -r requirements.txt

        # Copy the application to the container
        COPY . /app/test-app

        # Expose port 8000
        EXPOSE 8000

        # Apply migrations to set up the database (SQLite)
        RUN python manage.py migrate

        # Run the Django application
        CMD python /app/test-app/manage.py runserver 0.0.0.0:8000
        ```

    - Clone the app repository to EC2

    - Navigate into the app directory
    - build a docker container
        ```
        docker build -t test-app .
        ```

    - Local test
        ```
        docker ps

        docker run -d -p 8000:8000 test-app:latest
        ```