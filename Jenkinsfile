#!/usr/bin/env groovy
def gv

pipeline{
    agent any

    tools{
        maven 'maven-3.9'
    }

    stages{
        stage('init'){
            steps{
                script{
                    gv =  load "script.groovy" 
                }
                            
            }
        }
        stage('build jar'){
            steps{
                script{
                    gv.buildJar()               
                }
            }
        }
        stage('build image'){
            steps{
                script{
                    gv.buildImage()
                }  
                
               
            }
        }
        stage('deploy'){
            steps{
                echo "Deploying the project on aws server..!" 
                 script{
                    //gv.deployApp()
                    sshagent(['aws-ec2-server-key']) {
                        def dockercommand = "docker run -p 3080:8080 -d kajallad126/java-maven-app:1.5"
                        sh "ssh -o StrictHostKeyChecking=no ubuntu@35.182.166.224 ${dockercommand}"
                        }
                } 
            }
        }
    }
}
