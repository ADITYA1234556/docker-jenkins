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

## Jenkins Slave Setup

1. **Launch Another EC2 that will be Jenkins Slave**
    - Host another Ec2 and passing the user_data.sh to install docker

2. **Configure Master Slave by creating a ssh key-pair in master and using it with slave**
    - Create a key pair in Jenkins Master from inside the container "ssh-keygen -t rsa -b 4096 -C "jenkins-agent-key"
    - Copy the public key from within the Jenkins container to the Jenkins Slave machine "scp -i connect/privatekey filetocopy/jenkins-key.pub ubuntu@18.132.47.9:/home/ubuntu"
    - Add the contents of the public key from /home/ubuntu/jenkins-key.pub to ~/.ssh/authorized_keys, Now we can use the private key from master Jenkins to connect to slave

3. **In Jenkins Master configure the slave machine**
    - In Jenkins, go to Manage Jenkins > Manage Nodes and Clouds > New Node
    - Configure the agent node with:  
      - **Remote root directory:** `/home/ubuntu` ensure the directory has enough permissions
      - **Launch method:** SSH
      - **Host:** Agent server Public IP
      - **Credentials:** Add SSH private key credentials
      - **Pre-requisites** The Java version of Master Jenkins and Slave Jenkins should be the same


