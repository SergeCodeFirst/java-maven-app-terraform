#!/user/bin/env groovy
// @Library('jenkins-shared-library') // global import from jenkins (we add it there in a global library)
library identifier: 'jenkins-shared-library@main', retriever: modernSCM(
    [$class: 'GitSCMSource',
    remote: 'https://github.com/SergeCodeFirst/jenkins-shared-library.git',
    credentialsId: 'github-credentials'
    ])
def gv
// @Library('jenkins-shared-library')_ // keep "_" if after the import we have the pipeline tag

pipeline {
    agent any
    environment {
        IMAGE_NAME = "sergevismok/demo-app:jma-3.0"
    }
    tools {
        maven 'maven-3.9'
    }
    stages {
        stage ("init") {
            steps {
                script {
                    gv = load "script.groovy"
                }
            }
        }
        stage("build jar") {
            steps {
                script{
                    buildJar()
                }
            }
        }
        stage ("build and push image stage") {
            steps {
                script {
                    dockerBuildImage(env.IMAGE_NAME)
                    dockerLogin()
                    dockerPush(env.IMAGE_NAME)
                }
            }
        }
        stage("provision server") {
            environment {
                // AWS_ACCESS_KEY_ID = credentials('jenkins_aws_access_key_id')
                // AWS_SECRET_ACCESS_KEY = credentials('jenkins_aws_secret_access_key')
                TF_VAR_env_prefix = 'test'
            }
            steps {
                script {
                    withCredentials([
                        string(credentialsId: 'jenkins_aws_access_key_id', variable: 'AWS_ACCESS_KEY_ID'),
                        string(credentialsId: 'jenkins_aws_secret_access_key', variable: 'AWS_SECRET_ACCESS_KEY')
                    ]) {
                        dir('terraform')
                        sh "terraform init"
                        sh "terraform apply --auto-approve"
                        EC2_PUBLIC_IP = sh(
                            script: "terraform output ec2_public_ip",
                            returnStdout: true
                            ).trim()
                    }
                }
            }
        }

        stage("deploy") {
            environment{
                DOCKER_CREDS = credentials('docker-hub-repo')
            }
            steps {
                script {
                    echo "waiting for EC2 server to initialize"
                    sleep(time: 90, unit: "SECONDS")

                    echo 'deploying docker image to EC2...'
                    echo "ec2 public IP: ${EC2_PUBLIC_IP}"

                    def shellCmd = "bash ./server-cmds.sh ${IMAGE_NAME} ${DOCKER_CREDS_USER} ${DOCKER_CREDS_PSW}"
                    def ec2Instance = "ec2-user@${EC2_PUBLIC_IP}"

                    sshagent(['server-ssh-key']) {
                    sh "scp -o StrictHostKeyChecking=no server-cmds.sh ${ec2Instance}:/home/ec2-user"
                    sh "scp -o StrictHostKeyChecking=no docker-compose.yaml ${ec2Instance}:/home/ec2-user"
                    sh "ssh -o StrictHostKeyChecking=no ${ec2Instance} ${shellCmd}"
                    }
                }
            }
        }
    }
}