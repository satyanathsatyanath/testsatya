# Jenkins in Docker

This guide provides step-by-step instructions for setting up **Jenkins in Docker**.

## Prerequisites

Ensure you have the following installed:
- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)

## Step 1: Pull Jenkins Docker Image

```sh
docker pull jenkins/jenkins:lts
```

## Step 2: Create a Docker Network (Optional but Recommended)

```sh
docker network create jenkins
```

## Step 3: Run Jenkins Container

```sh
docker run -d \
  --name jenkins \
  --network jenkins \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts
```

- `-p 8080:8080` → Exposes Jenkins UI on port 8080.
- `-p 50000:50000` → For connecting agents.
- `-v jenkins_home:/var/jenkins_home` → Persistent volume for Jenkins data.

## Step 4: Get Initial Admin Password

Run the following command to retrieve the initial password:

```sh
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

Copy the password and use it for the first login.

## Step 5: Access Jenkins UI

- Open your browser and go to: **http://localhost:8080**
- Enter the **initial admin password**.
- Follow the setup wizard to install recommended plugins and create an admin user.

## Step 6: Install Docker in Jenkins (For Running Docker inside Jenkins)

1. **Access Jenkins Container Shell**
   ```sh
   docker exec -it jenkins bash
   ```
2. **Install Docker inside Jenkins container**
   ```sh
   apt-get update && apt-get install -y docker.io
   ```
3. **Add Jenkins User to Docker Group**
   ```sh
   usermod -aG docker jenkins
   ```
4. **Restart Jenkins**
   ```sh
   exit
   docker restart jenkins
   ```

## Step 7: (Optional) Run Jenkins with Docker Compose

Create a `docker-compose.yml` file:

```yaml
version: '3.8'
services:
  jenkins:
    image: jenkins/jenkins:lts
    container_name: jenkins
    ports:
      - "8080:8080"
      - "50000:50000"
    volumes:
      - jenkins_home:/var/jenkins_home
    networks:
      - jenkins

volumes:
  jenkins_home:

networks:
  jenkins:
```

Run the following command:
```sh
docker-compose up -d
```

## Step 8: Creating Jenkins Container Using Dockerfile with Python

You can also create a Jenkins container using a `Dockerfile` with Python pre-installed. Create a `Dockerfile` with the following content:

```Dockerfile
FROM jenkins/jenkins:lts
USER root
RUN apt-get update && apt-get install -y python3 python3-pip
USER jenkins
```

Then, build and run the container:

```sh
docker build -t jenkins-python .
docker run -d --name jenkins-docker   -v /var/run/docker.sock:/var/run/docker.sock   -v jenkins_home:/var/jenkins_home   --group-add $(getent group docker | cut -d: -f3)   -p 8080:8080 -p 50000:50000 <image_name>
```

## Step 9: Login to Jenkins Without Credentials

If you need to bypass authentication, login to the Jenkins container and remove the user configuration:

```sh
docker exec -it jenkins bash
rm -rf /var/jenkins_home/users
rm /var/jenkins_home/config.xml
exit
```

Then restart Jenkins:

```sh
docker restart jenkins
```

## Step 10: Cleanup (If Needed)

To stop and remove Jenkins container:
```sh
docker stop jenkins && docker rm jenkins
```
To remove the volume:
```sh
docker volume rm jenkins_home
```
To remove the network:
```sh
docker network rm jenkins
```

## Conclusion

You have successfully set up Jenkins in Docker! 🎉 Now you can start configuring jobs and pipelines.

For more details, refer to the [official Jenkins documentation](https://www.jenkins.io/doc/).

