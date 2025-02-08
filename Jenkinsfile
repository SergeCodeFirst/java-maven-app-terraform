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
        stage("deploy") {
            steps {
                script{
                    gv.deployApp()
                }
            }
        }
    }
}