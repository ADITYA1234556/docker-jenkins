# This project covers end-to-end automation, notification, and multi-environment Docker deployments on EC2 instances, with Jenkins in Docker and agent setup.

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
   - Ensure you give the folder on host machine right permissions so that Jenkins will be able to write to the location /tmp/jenkins_home
     ```bash
      chown -R 1000:1000 /tmp/jenkins_home
      #id of jenkins user is 1000 in official image
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
   - Docker pipeline
   - AWS Steps


## Jenkins Slave Setup

1. **Launch Another EC2 that will be Jenkins Slave**
    - Host another Ec2 and passing the user_data.sh to install docker

2. **Configure Master Slave by creating a ssh key-pair in master and using it with slave**
    - Create a key pair in Jenkins Master from inside the container "ssh-keygen -t rsa -b 4096 -C "jenkins-agent-key" -f /tmp/jenkins
    - -f flag is to avoid permission denied error. It will directly save it in /tmp directory where jenkins user will have permissions to write.
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
   - Configure GitHub Repository and Branches
     - In Branch Sources, add GitHub and specify the repository URL
     - Configure branch patterns to detect dev, staging, and main
   - Specify Jenkinsfile Location
     - Under Build Configuration, set it to by Jenkinsfile

4. **Define Jenkinsfile with Email Notification Step**
   - In the Jenkinsfile replace the Environment variables with yours.
   - Configure correct login details in Manage Jenkins -> Credentials -> add the following credentials
     - Sonarqube token ('sonartoken')
     - Github-Token
     - AWS CLI Access key and Secret access key ('aws-ecr')
     - Private SSH key pair for EC2 instance ('ec2-ssh-key')
     - Gmail Username and App password ('gmailcreds')
   - Add the following in Manage Jenkins -> System
     - Sonarqube server details -> pass the authentication details -> (Name: 'SonarQubeServer', Auth token: 'sonartoken')
     - E-mail Notification -> pass the authentication details -> ('gmailcreds')
   - Install trivy on worker machines
   ```bash
      wget https://github.com/aquasecurity/trivy/releases/download/v0.57.0/trivy_0.57.0_Linux-64bit.deb
      sudo dpkg -i trivy_0.57.0_Linux-64bit.deb
      trivy --version
      trivy --timeout 10m image imagename
   ```
   - The timeout will make sure trivy command gets enough time to download the dependencies
   - Create 3 EC2 instances for different environments and pass their IP addresses in stage('Deploy to Environment')
   - Install awscli on these instances because we will use awscli command to talk to AWS ECR to get a temporary token
   ```bash
      # aws ecr gets a temporary token from aws in eu-west-2 which will be used by docker without rquiring permanent creds
      # docker login --username AWS --password-stdin //temporary token ${ECRREPO}
      aws ecr get-login-password --region eu-west-2 | docker login --username AWS --password-stdin ${env.ECR_REPO}
      docker pull ${ECR_REPO}:${TAG}
      docker stop ${IMAGE_NAME} || true
      docker rm ${IMAGE_NAME} || true
      docker run -d --name ${IMAGE_NAME} -p 8080:8080 -p 8090:8090 ${ECR_REPO}:${TAG}
    ```
     