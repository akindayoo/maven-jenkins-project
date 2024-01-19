pipeline {
    agent any

    tools {
        maven "MAVEN3"
        jdk "oracleJDK11"
    }

    environment {
        awsEcrCreds = 'ecr:us-east-1:JenkinsAWSCLI'
        awsEcrRegistry =  "509085657504.dkr.ecr.us-east-1.amazonaws.com/devopsacad-d-image"
        devopsacadEcrImgReg = "https://509085657504.dkr.ecr.us-east-1.amazonaws.com"
        awsRegion = 'us-east-1'
        cluster = 'webapp'
        service = 'webapp-svc'
    }

    stages {
        stage ('fetch code') {
            steps {
                script {
                    echo "Pull Source code from Git"
                    git branch: 'docker', url: 'https://github.com/akindayoo/maven-jenkins-project.git'
                }
            }
        }

        stage ('Build App') {
            steps {
                script {
                    echo "Building WAR with Maven"
                    sh 'mvn install -DskipTests'
                }
            }
        }

        stage ('Build Docker Image') {
            steps{
                script {
                    dockerImage = docker.build(awsEcrRegistry + ":$BUILD_NUMBER", "./Docker-files/app/multistage/")
                }
            }
        }

        stage ('Push Image to ECR') {
            steps{
                script {
                    docker.withRegistry (devopsacadEcrImgReg, awsEcrCreds) {
                        dockerImage.push ("$BUILD_NUMBER")
                        dockerImage.push ('latest')
                    }
                }
            }
        }

        stage ('Deploy to ECS') {
            steps{
                script {
                    withAWS(credentials:'JenkinsAWSCLI', region: "${awsRegion}") {
                        sh 'aws ecs update-service --cluster ${cluster} --service ${service} --force-new-deployment'
                    }
                }
            } 
        }
    }
}                          
