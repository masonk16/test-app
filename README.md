# Setting up the CI/CD Pipeline with Jenkins

- EC2
- Docker
- Jenkins
- GitHub
- Django

1. Initialize the remote repo with a `.gitignore` file

2. Clone to local:

    ```shell
    git clone <link to repository>
    ```

3. Create and activate virtual environment:

    ```shell
    # Linux
    # Create virtual environment
    python3 -m venv .venv --prompt web-app

    # Activate virtual environment
    source .venv/bin/activate
    ```

    ```shell
    # Windows/Powershell
    python -m venv .venv

    .venv\Scripts\activate
    ```

4. Install Django

    ```shell
    pip install Django
    ```

5. Create requirements file:

    ```shell
    pip freeze > requirements.txt
    ```

6. Create a django project:

    ```shell
    django-admin startproject config .
    ```

7. Setup EC2:
    - Create EC2 with Ubuntu
    - Allow traffic to port 8080 & 8000 (Inbound rules)
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
        - Navigate to `http://<your-ec2-public-ip-address>:8080`
        - Run `sudo cat /var/lib/jenkins/secrets/initialAdminPassword` to get the admin password
        - Follow the prompts to a new admin user

    - Install Docker

        ```shell

        # Add Docker's official GPG key:
        sudo apt-get update
        sudo apt-get install ca-certificates curl
        sudo install -m 0755 -d /etc/apt/keyrings
        sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
        sudo chmod a+r /etc/apt/keyrings/docker.asc

        # Add the repository to Apt sources:
        echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
        sudo apt-get update 

        # Install docker
        sudo apt-get install docker.io

        # Install docker packages
        sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
        
        sudo systemctl enable docker

        sudo systemctl start docker

        sudo systemctl status docker

        # add docker user to the same user group as root user
        sudo usermod -aG docker $USER

        # Create username and password for DockerHub
        docker login -u <username>

        # Give permissions for docker socket file
        sudo chmod 666 /var/run/docker.sock
        sudo usermod -aG docker $USER
        ```

    - In the `config/settings.py` file on your local repo edit the `ALLOWED_HOSTS` setting to add the IP address for your EC2 instance.

    - Create Dockerfile on local repository

        ```
        # Use Python 3.11 as the base image
        FROM python:3.11

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
        CMD ["python", "/app/test-app/manage.py", "runserver", "0.0.0.0:8000"]
        ```
        - Push changes to remote repo

    - Clone the app repository to EC2
      ```shell
      git clone <link to your repo>
      ```

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

8. Configure Jenkins Pipeline

    - add docker user to the jenkins group
        ```shell
        sudo usermod -aG docker jenkins
        ```
   - Create Docker Hub Personal Access Token:
       - Sign in to your Docker account.

       - Select your avatar in the top-right corner and from the drop-down menu select Account settings.

       - In the Security section, select Personal access tokens.

       - Select Generate new token.

       - Add a description for your token. Use something that indicates the use case or purpose of the token.

       - Set the access permissions. The access permissions are scopes that set restrictions in your repositories. For example, for Read & Write permissions, an automation pipeline can build an image and then push it to a repository. However, it can't delete the repository.

       - Select Generate and then copy the token that appears on the screen and save it. You won't be able to retrieve the token once you close this prompt.
         
    - In the Jenkins dashboard:
        - select "Manage Jenkins"
        - Under the Security section select "Credentials"
        - Click on the "system" credentials
        - Click on "Global credentials (unrestricted)"
        - Click Add credentials:
            - Kind: Username with password
            - Enter the username for your Docker account
            - For password enter the Personal Access Token you created earlier
            - Enter ID as "dockerHub"

    - Create a jenkins job
        - Click on "Create a job"
        - Give the job a name
        - Select the "Pipeline"
        - Configure General Settings:
            - Choose Discard old builds
            - Choose github project and enter the repository URL
            - Set build triggers - GitHub hook trigger for GITscm polling
            - Under pipeline:
                - select "Pipeline script from SCM" for definition
                - Select Git under SCM
                - Add the repository URL
                - Ensure Branch specifier has the correct value
            - Create `Jenkinsfile` in local repository and add the following code:
                ```
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
                                sh "sudo docker build -t test-app ."
                            }
                        }

                        stage("Push to Docker Hub") {
                            steps {
                                echo "Pushing image to Docker Hub"
                                withCredentials([usernamePassword(credentialsId: "dockerHub", passwordVariable: "dockerHubPass", usernameVariable: "dockerHubUser")]) {
                                    sh "sudo docker tag test-app ${env.dockerHubUser}/test-app:latest"
                                    sh "sudo docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}"
                                    sh "sudo docker push ${env.dockerHubUser}/test-app:latest"
                                }
                            }
                        }

                        stage ("Deploy") {
                            steps {
                                echo "Deploying the container"
                                sh "sudo docker compose down && docker compose up -d"
                            }
                        }
                    }
                }
                ```

9. Create the docker-compose.yml

    ```yaml
    version: "3.3"
    services:
      web: 
        image: <your docker username>/test-app:latest
        ports:
          - "8000:8000" 
    ```

10. Create a github webhook
    - under repository settings
    - select "Webhooks"
    - Enter payload URL `http://<your EC2 ip address>:8080/github-webhook/`
    - Under content type select "application/x-www-form-urlencoded"
    - Select "Send me everything"
    - Ensure that "Active" is checked
    - Click on "Add Webhook"

11. Test the CI/CD Pipeline
