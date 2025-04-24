# Dockerized Jenkins with Django Application

This repository contains a Dockerized Django application with a Jenkins pipeline for CI/CD. The project also integrates with SonarQube for static code analysis.

## Requirements

- Jenkins (runs in a Docker container)
- Docker
- Docker Compose
- SonarQube (optional, for code quality analysis)

---

## Implementation Steps

### **Step 1: Jenkins Setup**

1. **Install Jenkins in Docker on Your Server**:
   Run Jenkins in a Docker container:
   ```bash
   docker run -d --name jenkins -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts
   ```
   - Access Jenkins at `http://<your-server-ip>:8080`.

2. **Unlock Jenkins**:
   - Get the initial admin password from the container logs:
     ```bash
     docker logs jenkins
     ```
   - Use the password to unlock Jenkins in the web interface.

3. **Install Plugins**:
   - Install the following plugins:
     - **Git**
     - **Pipeline**
     - **SonarQube Scanner**
     - **Docker**
     - **Docker Pipeline**

4. **Configure Tools**:
   - **Add SonarQube**:
     - In Jenkins, go to `Manage Jenkins > Configure System > SonarQube servers`.
     - Add your SonarQube server URL and authentication token.
   - **Add Docker Host**:
     - Ensure Docker is installed on the same server as Jenkins.
     - Install the **Docker Pipeline** plugin.

---

### **Step 2: Create a Jenkins Pipeline Job**

1. **Create a New Pipeline Job**:
   - In Jenkins, click `New Item`, enter a name (e.g., "Django CI/CD"), and select `Pipeline`.

2. **Configure the Pipeline**:
   - Choose `Pipeline script from SCM`.
   - Set the SCM to `Git` and provide the repository URL:
     ```
     https://github.com/blackkolly/dockerizing-jenkins.git
     ```
   - Specify the branch (e.g., `main`).

---

### **Step 3: Jenkins Pipeline Script**

The `Jenkinsfile` is already included in the repository. It handles:
   - Cloning the repository from GitHub.
   - Building the Django application and applying migrations.
   - Running tests.
   - Performing SonarQube analysis.
   - Building a Docker image for the Django app and pushing it to DockerHub.
   - Deploying the application using `docker-compose` on the Jenkins server.

---

### **Step 4: Configure Docker in Jenkins**

Ensure that Docker is installed on the same server as Jenkins and is accessible to the Jenkins user:

1. Add the Jenkins user to the Docker group:
   ```bash
   sudo usermod -aG docker jenkins
   ```
   Restart Jenkins after making this change.

2. Install the **Docker Pipeline Plugin** in Jenkins.

3. Add Docker credentials to Jenkins for pushing images to DockerHub:
   - Go to `Manage Jenkins > Credentials > System > Global credentials`.
   - Add a new credential for DockerHub with your username and password.
   - Replace `'docker-hub-credentials-id'` in the `Jenkinsfile` with the ID of your credentials.

---

### **Step 5: Run the Pipeline**

1. Trigger a build of the pipeline in Jenkins.
2. The pipeline will perform the following:
   - Clone the repository from GitHub.
   - Build the Django application and apply migrations.
   - Run tests.
   - Perform SonarQube analysis (if configured).
   - Build a Docker image for the Django app and push it to DockerHub.
   - Deploy the application using `docker-compose` on the Jenkins server.

---

### **Step 6: Access the Application**

1. After the pipeline runs successfully, access the Django application in your browser:
   - URL: `http://<your-server-ip>:8000`

---

### **Step 7: Monitor and Improve**

1. **Monitor Jenkins**:
   - Use the Jenkins dashboard to monitor pipeline runs and view logs for debugging.
2. **Improve Code Quality**:
   - Review SonarQube reports and fix any issues.
3. **Scale Services**:
   - Modify `docker-compose.yml` to scale the Django application (e.g., adding a load balancer).