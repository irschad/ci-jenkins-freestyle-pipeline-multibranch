# CI Pipeline for Java Maven Application

## Project Overview
This project demonstrates the creation of a Continuous Integration (CI) pipeline for a Java Maven application using Jenkins. It includes building a JAR file, creating a Docker image, and pushing it to a private DockerHub repository. The project explores different Jenkins job types: Freestyle, Pipeline, and Multibranch Pipeline.

## Technologies Used
- Jenkins
- Docker
- Linux
- Git
- Java
- Maven

## Project Features
- Install build tools (Maven, Node.js) in Jenkins.
- Configure Docker for Jenkins.
- Create Jenkins credentials for connecting to a Git repository and DockerHub.
- Implement Freestyle, Pipeline, and Multibranch Pipeline jobs for:
  - Connecting to the Git repository.
  - Building a JAR file.
  - Creating a Docker image.
  - Pushing the Docker image to a private DockerHub repository.

---

## Prerequisites
1. **Jenkins Setup:** Ensure Jenkins is installed and running.
2. **Docker Installation:** Docker must be installed and configured on the Jenkins server.
3. **Git Repository:** Clone or fork the repository: [java-maven-app](https://github.com/irschad/java-maven-app.git).
4. **Credentials:**
   - GitHub Personal Access Token for accessing the repository.
   - DockerHub credentials for pushing images.

---

## Steps to Configure the Pipeline

### 1. Install Build Tools in Jenkins
#### Install Maven
1. Go to `Manage Jenkins > Global Tool Configuration`.
2. Under "Maven", click "Add Maven" and provide a name (e.g., `maven-3.9`).
3. Save the configuration.

#### Install Node.js
1. Enter the Jenkins Docker container as root:
   ```bash
   docker exec -u 0 -it <container-id> bash
   ```
2. Install Node.js and npm:
   ```bash
   apt update
   apt install -y curl
   curl -sL https://deb.nodesource.com/setup_20.x -o nodesource_setup.sh
   apt install -y nodejs
   ```
3. Verify installation:
   ```bash
   node --version
   v18.19.0
   npm --version
   9.2.0
   ```

### 2. Configure Docker in Jenkins
1. Restart the Jenkins container with Docker socket and binary mounted:
   ```bash
   docker run -p 8080:8080 -p 50000:50000 -d \
     -v jenkins_home:/var/jenkins_home \
     -v /var/run/docker.sock:/var/run/docker.sock \
     -v $(which docker):/usr/bin/docker \
     jenkins/jenkins:lts
   ```
2. Provide Docker permissions to the Jenkins user:
   ```bash
   docker exec -u 0 -it <container-id> bash
   chmod 666 /var/run/docker.sock
   exit
   ```

### 3. Create Jenkins Credentials
1. Generate a GitHub Personal Access Token with "repo" scope.
2. Add the token to Jenkins:
   - Go to `Manage Jenkins > Manage Credentials > Global Credentials`.
   - Add a "Username and password" credential with the GitHub username and token.
3. Add DockerHub credentials:
   - Use the DockerHub username and password.

### 4. Freestyle Job
1. Create a new Freestyle Job.
2. Configure the job to connect to the Git repository.
3. Add build steps for:
   - Packaging the application using Maven.
   - Building a Docker image.
   - Pushing the image to DockerHub.

### 5. Pipeline Job
1. Create a new Pipeline Job.
2. Use "Pipeline script from SCM" and point to the repository.
3. Add a Jenkinsfile in the repository with stages for building the JAR, Docker image, and pushing to DockerHub.

### 6. Multibranch Pipeline Setup
#### Step 1: Create a Multibranch Pipeline
1. Go to `Dashboard > New Item`.
2. Enter a name (e.g., `java-maven-multibranch`) and select "Multibranch Pipeline".
3. Click "OK".

#### Step 2: Configure Branch Sources
1. In the configuration page, go to the "Branch Sources" section.
2. Add a Git source and provide the repository URL: `https://github.com/irschad/java-maven-app.git`.
3. Select the credentials created earlier for GitHub.
4. Under "Discover Branches", click "Add" and select "Filter by name (with regular expression)".
5. Enter `.*` to discover all branches containing a Jenkinsfile.

#### Step 3: Scan Repository
1. Save the configuration.
2. Jenkins will scan the repository and identify branches containing a Jenkinsfile.
3. For each branch, it will create a pipeline and start builds automatically.

#### Step 4: Multibranch Pipeline Jenkinsfile
Add the following Jenkinsfile to the repository:
```groovy
pipeline {
    agent any
    tools {
        maven 'maven-3.9'
    }
    stages {
        stage('Build Jar') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t irschad/java-app:1.0 .'
            }
        }
        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'DockerHub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push irschad/java-app:1.0'
                }
            }
        }
    }
}
```

This Jenkinsfile defines stages for building the JAR, building the Docker image, and pushing it to DockerHub.

---

## Conclusion
This project demonstrates how to set up a comprehensive CI pipeline for a Java Maven application using Jenkins. By utilizing Freestyle, Pipeline, and Multibranch Pipeline jobs, the project showcases different approaches to automating builds and deployments in a DevOps workflow.
