## DOCKER and JENKINS SETUP
- Install docker in ubuntu machine by passing the user_data.sh
- run jenkins as a container by docker pull jenkins/jenkins
- docker run -d -p 8090:8080 -p 50000:50000 --name jenkins-master \ -v /tmp/jenkins_home:/var/jenkins_home \ -v /var/run/docker.sock:/var/run/docker.sock \ jenkins/jenkins
- Set up jenkins by entering into the container "docker exec -it ${container_id} /bin/bash" 
- Get the admin password from /var/jenkins_home/secrets/initialAdminPassword 
