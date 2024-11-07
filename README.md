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

## Configure IAM Role for EC2 Instances

1. **Create an IAM role with the following permissions**
   - AmazonEC2ContainerRegistryFullAccess
   - SecretsManagerReadWrite

2. **Attach the role to both Master Jenkins and Slave Jenkins**

## Configure Jenkins Multi-Branch Pipeline Job with Automatic Trigger and Email Notification

1. **Setup GitHub WebHook to automatically trigger the job when events are pushed to GitHub Repo**
   - Go to GitHub Repository settings -> Webhooks
   - Add the Payload url like http://<Jenkins-Master-EC2-Public-IP>:<jenkins-service-port>/github-webhook/
   - Configure the webhook to trigger "ONLY FOR PUSH EVENT"
   - Verify recent deliveries and make sure connection is successful

2. **Configure SMTP Settings for Email Notification**
   - Go to Jenkins Dashboard > Manage Jenkins > Configure System
   - Set up SMTP Server (e.g., smtp.gmail.com for Gmail) and configure Email Extension Plugin.
   - Pass in your Username and Gmail App password and check connectivity
   - Save settings

3. **Create Multi-Branch Pipeline in Jenkins**
   - Create Multi-Branch Pipeline Job:
     - Go to Jenkins Dashboard > New Item and select Multi-branch Pipeline
     - Name the pipeline and click OK
