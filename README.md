## Docker and Jenkins Setup

1. **Install Docker in Ubuntu machine**  
   Run the `user_data.sh` script to install Docker.

2. **Run Jenkins as a Docker Container**
   - Pull the Jenkins image:
     ```bash
     docker pull jenkins/jenkins
     ```
   - Run Jenkins in a container:
     ```bash
     docker run -d -p 8090:8080 -p 50000:50000 --name jenkins-master \
     -v /tmp/jenkins_home:/var/jenkins_home \
     -v /var/run/docker.sock:/var/run/docker.sock \
     jenkins/jenkins
     ```

3. **Set Up Jenkins**
   - Enter the Jenkins container:
     ```bash
     docker exec -it <container_id> /bin/bash
     ```
   - Retrieve the admin password:
     ```bash
     cat /var/jenkins_home/secrets/initialAdminPassword
     ```

4. **Install Required Plugins in Jenkins**
   - Docker
   - Pipeline
   - GitHub Branch Source
   - AWS Credentials
   - SSH Agent
   - Email Extension
   - SonarQube
   - Trivy
