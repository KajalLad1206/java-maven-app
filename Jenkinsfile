pipeline{
    agent any

    tools{
        maven 'maven-3.9'
    }

    stages{
        stage('build jar'){
            steps{
                script{
                     echo "building jar.......!"
                     sh 'mvn clean package'               
                }
            }
        }
        stage('build image'){
            steps{
                script{
                    echo "building an image....!"
                    withCredentials([
                    usernamePassword(credentialsId: 'dockerhub-credential', usernameVariable: 'USER', passwordVariable: 'PWD')
                    ]) {
                        sh 'echo Logging into Docker...'
                        sh 'docker build -t kajallad126/java-maven-app:1.4 .'
                        sh "echo $PWD | docker login -u $USER  --password-stdin"
                        sh 'docker push kajallad126/java-maven-app:1.4'
                    }
                }  
                
               
            }
        }
        stage('deploy'){
            steps{
                 script{
                    echo "deploying an application...!"
                } 
            }
        }
    }
}
