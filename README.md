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
        sudo apt update
        sudo apt install -y openjdk-17-jdk
        Verify: java -version
    Add Jenkins Repo 

        curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null 

        echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null 

    Install Jenkins 
        sudo apt-get update 
        sudo apt install jenkins -y 

    Starting Jenkins 
        sudo systemctl start jenkins.service 
        sudo systemctl status jenkins 

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

### Technologies Used:
AWS,Jenkins, Docker, Linux, Git, Java, Maven 

### Project Objectives:
CI Pipeline for a Java Maven application to build and push to the repository
- Install Build Tools Maven, NodeJS in Jenkins
-Install npm on jenkins server
- Make Docker available on Jenkins server
- Create Jenkins credentials for a git repository
- Create different Jenkins job types (Freestyle, Pipeline, Multibranch Pipeline) for the Java Maven project with Jenkinsfile to:
    - Connect to the application's git repository
    - Build Jar 
    - Build Docker Image
    - Push to private DockerHub repository
---
### Install Build Tools
FYI: Jenkins Plugins - https://plugins.jenkins.io/

1. configure maven plugin
    - in Jenkins go to Manage Jenkins > Gloabal Tool Configuration
    - scroll down to maven and select Add Maven
    - enter a name, select version, and hit Save
2. install npm and nodejs on the jenkins server
    - npm is not a plugin in jenkins, so we'll need to install 
 

### Make Docker Available on the Jenkins Server
1. create a Dockerfile on the server
```
FROM jenkins/jenkins:lts
RUN curl -sSL https://get.docker.com/ | sh
USER jenkins
```
2. tag and build a new jenkins image based on the Dockerfile
`docker build -t jenkins-with-docker .`
3. run jenkins container with the new image
4. shutdown the current running jenkins container
### Create Jenkins Credentials for a Git Repository
1. in Jenkins UI on the dashboard, click on Manage Jenkins
2. click on Manage Credentials
3. in the table, under domains, click (global)
4. click Add Credentials button
    - select 'Username with Password'
    - if setting up with username and password
        - username = github/gitlab username
        - password = github/gitlab password
    - if setting up with ssh
        - make sure your private key is added with the credentials in Jenkins
        - select 'SSH Username with private key'
        - enter your username, and copy your private key
        - if needed, add the key from github to the known_hosts file in the Jenkins container in the .ssh folder
            - can run `ssh-keyscan github.com >> ~/.ssh/known_hosts` to copy github's key to the known_hosts file
            - run this as jenkins user in the jenkins container
    - enter an id and description
5. click Create button

### Create Different Jenkins Job Types

#### Freestyle
1. in Jenkins UI on the dashboard, click 'New Item'
2. enter a name and select 'Freestyle Project'
    - under Source Code Management, select Git
        - enter the repository url (github or gitlab)
        - select credentials or add credentials and specify the branch name
    - add build steps
        - select 'Invoke top-level Maven targets'
        - select Maven version
        - add Goals : (clean package, Test, Package, etc.)
3. building a docker image
     - in the freestyle job dashboard, click configure
     - add on a build step, 'Execute shell'
     - enter docker commands
        ```
        docker build -t java-maven-app:1.0 .
        ```
    - save
4. pushing image to dockerhub (private docker repository)
    - add new Jenkins credentials for dockerhub
    - configure your credentials as secrets to be used in the docker login command
        - in configuration > build environment, select 'Use secret text(s) or file(s)
        - in bindings select 'Username and password (separated)'
        - enter variable names of your liking
        - choose 'Specific Credentials' and select the dockerhub credentials
        - save
    - update docker commands with docker push
        ```
        docker build -t <name of dockerhub repo>:<tag> .
        docker login -u $USERNAME -p $PASSWORD
        docker push <image name> 
        ```
NOTE: if there is a warning like this in the logs: "Using --password via the CLI is insecure. Use --password-stdin...", update the docker login command to `echo $PASSWORD | docker login -u $USERNAME --password-stdin`


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
    -simple Jenkinsfile :https://github.com/KajalLad1206/java-maven-app/blob/jenkins-job/ex-Jenkinsfile
    
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

Example screenshot : [https://github.com/KajalLad1206/java-maven-app/tree/main/screenshot](https://github.com/KajalLad1206/java-maven-app/tree/jenkins-job/screenshot)



