# Jenkins Pipeline with Webhook Setup

This guide provides step-by-step instructions for setting up a **Jenkins pipeline with a GitHub webhook** to trigger builds automatically.

## Prerequisites

Ensure you have the following:
- **Jenkins installed and running** (on-premise or in Docker)
- **GitHub repository** with the project source code
- **GitHub Webhook Configuration Access**
- **Jenkins plugins**:
  - [Git Plugin](https://plugins.jenkins.io/git/)
  - [Pipeline Plugin](https://plugins.jenkins.io/workflow-aggregator/)
  - [GitHub Plugin](https://plugins.jenkins.io/github/)

## Step 1: Install Required Plugins

Go to **Manage Jenkins** → **Manage Plugins** → **Available Plugins**, search for and install:
- Git Plugin
- Pipeline Plugin
- GitHub Plugin

## Step 2: Configure Jenkins Job

1. Go to **Jenkins Dashboard** → **New Item**.
2. Select **Pipeline** and name your job.
3. Click **OK**.
4. In **Pipeline Definition**, select **Pipeline script from SCM**.
5. Choose **Git** and enter your GitHub repository URL.
6. Set **Branch Specifier** to the branch you want to build (e.g., `*/main`).
7. In **Script Path**, enter `Jenkinsfile` (if using a separate file for pipeline definition).
8. Click **Save**.

## Step 3: Configure GitHub Webhook

To trigger builds automatically from GitHub:

1. Go to your **GitHub repository**.
2. Click on **Settings** → **Webhooks**.
3. Click **Add webhook**.
4. In the **Payload URL** field, enter:
   ```
   http://<JENKINS_URL>/github-webhook/
   ```
   Example: `http://your-jenkins-server.com/github-webhook/`
5. Set **Content type** to `application/json`.
6. Choose **Just the push event**.
7. Click **Add webhook**.

## Step 4: Create a Jenkinsfile

If your project uses a pipeline, ensure there is a `Jenkinsfile` in the root directory:

```groovy
pipeline {
    agent any  // Runs on any available agent
 
    environment {
        IMAGE_NAME = "flask_app"  // Docker image name
        CONTAINER_NAME = "flask_container"
    }
 
    stages {
        stage('Clean Workspace') {
            steps {
                deleteDir()  // Ensures old files are removed
            }
        }
        stage('Clone Repository') {
            steps {
                script {
                    git branch: 'main', url: 'https://github.com/snath/testsatya.git'
                    sh 'git config --global --add safe.directory `pwd`'
                    // Enable sparse checkout to pull only a specific folder
                    sh '''
                    git sparse-checkout init --cone
                    echo "docker_flask" > .git/info/sparse-checkout
                    git checkout main
                    git pull origin main
                    '''
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    sh '''
                    cd docker_flask
                    docker build -t ${IMAGE_NAME} .
                    '''
                }
            }
        }
        stage('Run Docker Container') {
            steps {
                script {
                    // Stop and remove any existing container with the same name
                    sh '''
                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true
                    docker run -d --name ${CONTAINER_NAME} -p 5000:5000 ${IMAGE_NAME}
                    '''
                }
            }
        }
    }
}


```

## Step 5: Enable Build Trigger in Jenkins

1. Open your Jenkins job.
2. Click **Configure**.
3. Scroll down to **Build Triggers**.
4. Check **GitHub hook trigger for GITScm polling**.
5. Click **Save**.

## Step 6: Test the Webhook

1. Make a change in the GitHub repository and push it.
2. Go to **Jenkins Dashboard** → **Your Job**.
3. Click **Build History** to see if the build was triggered automatically.

## Conclusion

Your Jenkins pipeline is now configured to trigger builds automatically when a change is pushed to GitHub. 🎉

For more details, refer to the [Jenkins Documentation](https://www.jenkins.io/doc/).

