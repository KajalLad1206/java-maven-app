# Build Automation & CI/CD with Jenkins

### Demo Project:
Install Jenkins on AWS

### Technologies Used:
Jenkins, Docker, AWS, Linux

### Project Objectives:
- Create an Ubuntu server on AWS
- Set up and run Jenkins in server or as Docker container
- Initialize Jenkins
---
### Create an Ubuntu Server
AWS account

Running EC2 instance (Ubuntu 20.04 or 22.04)

Inbound security group open for:

Port 22 (SSH)
Port 8080 (Jenkins)
1. Connect to EC2 Instance
From your local machine:
ssh -i /path/to/your-key.pem ubuntu@your-ec2-public-ip

### Either you can use Docker image of Jenkins or Manually install Jenkins
1. Using Docker image of Jenkins
    Set Up and Run Jenkins as a Container
    1. after logging into the aws, install docker
        - `apt update` then `apt install docker.io`
    2. run the container
        - `docker run -p 8080:8080 -p 50000:50000 -d -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts`
            - port 50000 is for communication between jenkins master and worker nodes
            - jenkins/jenkins is the official jenkins image from dockerhub
2. Manually Install Jenkins on aws server (here I used this method)
    Jenkins requires Java (OpenJDK 11+). Run:
        `sudo apt update`
        `sudo apt install -y openjdk-17-jdk`
        Verify: `java -version`

   - Add Jenkins Repo 

        `curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null` 

        `echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null `

    Install Jenkins 
        `sudo apt-get update` 
        `sudo apt install jenkins -y` 

    Starting Jenkins 
        `sudo systemctl start jenkins.service` 
        `sudo systemctl status jenkins` 

    Setting Up Jenkins URL:

        http://your_server_ip_or_domain:8080

        example : http://99.79.36.91:8080/

### Initialize Jenkins
1. access jenkins from the browser
    - login with the provided admin credentials
    - you will be given the path to where the password is located, the path is a location inside of the container   
2. after login, install the suggested plugins
3. create first admin user
---
---
### Demo Project:
CI Pipeline with Jenkinsfile (Freestyle, Pipeline, Multibranch Pipeline)

#### Pipeline
1. in Jenkins on the Dashboard
    - 'New Item' > enter a name > select 'Pipeline'
2. in 'Pipeline' section
    - for 'Definition' select 'Pipeline script from scm'
    - in 'SCM' (source code manager)
        - select 'Git
        - add the git repo url and credentials
        - specify the branch (can be a regex (regular express) or text)
    - for 'Script Path', add the path to the Jenkinsfile in the project
3. complete pipeline setup
    - Groovy script: https://github.com/KajalLad1206/java-maven-app/blob/jenkins-job/script.groovy
    - Jenkinsfile with groovy functions: https://github.com/KajalLad1206/java-maven-app/blob/jenkins-job/Jenkinsfile
    - simple Jenkinsfile :https://github.com/KajalLad1206/java-maven-app/blob/jenkins-job/ex-Jenkinsfile
    
#### Multibranch Pipeline
1. in Jenkins on the Dashboard
    - 'New Item' > enter a name
2. similar to above, in the 'Branch Sources' section
    - select 'Git' and add git repository information
3. for 'Behaviors', select 'Discover branches' and 'Filter by name (with regular expression)'
    - enter a regex to match a branch(s) or match all branches
4. in 'Mode' section under 'Build Configuration'
    - select 'by Jenkinsfile' and give the path to the file
5. you can add branch based logic to the Jenkinsfile
    - this will execute stages based on the branch
    ```
        // execute this stage when the branch is the main branch
        // can `echo $BRANCH_NAME` to print out the branch
        when {
            expression { BRANCH_NAME == 'main' }
        }
        steps...
    ```
NOTE: all branches configured in the multibranch pipeline job should share the same Jenkinsfile

-screenshot : https://github.com/KajalLad1206/java-maven-app/tree/jenkins-job/screenshot


### Project Objectives:
- Create a Jenkins Shared Library to extract common build logic:
    - Create separate Git repository for Jenkins Shared Library project
    - Create functions in the JSL to use in the Jenkins pipeline
    - Integrate and use the JSL in Jenkins Pipeline (globally and for a specific project in Jenkinsfile)
---

### Demo Project:
Create a Jenkins Shared Library

https://github.com/KajalLad1206/jenkins-shared-library/tree/main#jenkins-shared-library

Example : Use of Library in Jenkinsfile : https://github.com/KajalLad1206/java-maven-app-multipipe/blob/app-deploy/Jenkinsfile

---

### Demo Project:
CD - Deploy Application from Jenkins Pipeline to EC2 Instance

https://github.com/KajalLad1206/java-maven-app-multipipe#build-automation--cicd-with-jenkins

---

### Demo Project:
Configure Webhook to trigger CI Pipeline automatically on git push

https://github.com/KajalLad1206/java-maven-app-multipipe#build-automation--cicd-with-jenkins




