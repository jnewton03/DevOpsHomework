# Datical DevOps Homework Notes

## VirtualBox Setup

+ Virtualbox installed
+ Downloaded Lubuntu 16.04 VMI per instructions
+ Created new VM called 'Datical' with 4GB RAM and 10GB HDD.  Imported existing VDI for Lubuntu.
  + Increased video RAM and enabled 3d acceleration.
  + Installed VBOXADDITIONS from terminal and rebooted
    + `sudo apt-get install gcc make perl`
  + Enabled sshd on lubuntu system
    + `sudo apt-get install openssh-server`

## Docker Setup

    ```console
    sudo apt-get update
    sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
    sudo apt-get update
    sudo apt-get install docker-ce docker-ce-cli containerd.io
    sudo usermod -aG docker osboxes
        restart ssh session or logout and back in to take effect
    ```

Installed Jenkins in Docker
    docker run -d -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts
    docker update --restart=always <container_id>
Access Jenkins UI using VirtualBox bridged Network
    http://192.168.1.35:8080
Gather Unlock password
    docker exec <container_id> cat /var/jenkins_home/secrets/initialAdminPassword
Install suggested plugins
Create Admin User
    admin/admin
Created new job called 'Datical Homework' as multibranch pipeline
    Added SCM w/ GitHub account
Build Failure Log indicates Docker Client is not installed
    /var/jenkins_home/workspace/Datical_Homework_master@tmp/durable-fc32624e/script.sh: 1: /var/jenkins_home/workspace/Datical_Homework_master@tmp/durable-fc32624e/script.sh: docker: not found
Go to Manage Jenkins -> Global Tool Configuration
Add Docker
    Name: DaticalDocker
    Install Automatically: Yes
    Docker Version: Latest

Updated the Jenkins Docker instance to map the docker client socket and executable to be available within the container:
    docker run -d -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -v $(which docker):$(which docker) jenkins/jenkins:lts

Got permission denied error:
    + docker inspect -f . maven:3-alpine

    Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.39/containers/maven:3-alpine/json: dial unix /var/run/docker.sock: connect: permission denied

    This issue is caused by the jenkins user not being in the Docker group inside the container

 Updated the Jenkins Docker instance to add to the Docker group (gid 998)
    docker run -d -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -v $(which docker):$(which docker) --group-add 998 jenkins/jenkins:lts
# Recommend creating a new Dockerfile based off the jenkins Docker image.  Also may have been more straightforward to install Jenkins via apt instead of in Docker.

Failure on Deliver Stage due to permissions
   [Pipeline] stage
   [Pipeline] { (Deliver)
   [Pipeline] sh
    + ./jenkins/scripts/deliver.sh
    /var/jenkins_home/workspace/Datical_Homework_master@tmp/durable-2b0363fc/script.sh: line 1: ./jenkins/scripts/deliver.sh: Permission denied 
Fixed with 'git update-index --chmod=+x deliver.sh'

Builds now succeeding
