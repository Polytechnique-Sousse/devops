pipeline {
    agent any
    triggers {
        pollSCM('H/5 * * * *')
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
        IMAGE_NAME_SERVER = 'naouresdoc/mern-server'
        IMAGE_NAME_CLIENT = 'naouresdoc/mern-client'
    }
    stages {
        // Uncomment if you want to use the checkout stage
        // stage('Checkout') {
        //     steps {
        //         git branch: 'main',
        //             url: 'git@github.com:Polytechnique-Sousse/devops.git',
        //             credentialsId: 'github-ssh'
        //     }
        // }

        stage('Build Server Image') {
            steps {
                dir('server') {
                    script {
                        dockerImageServer = docker.build("${IMAGE_NAME_SERVER}")
                    }
                }
            }
        }

        stage('Build Client Image') {
            steps {
                dir('client') {
                    script {
                        dockerImageClient = docker.build("${IMAGE_NAME_CLIENT}")
                    }
                }
            }
        }

        stage('Vulnerability Scan - Server Image') {
            steps {
                script {
                    sh "trivy image --severity HIGH,CRITICAL ${IMAGE_NAME_SERVER}:${env.BUILD_NUMBER}"
                }
            }
        }

        stage('Vulnerability Scan - Client Image') {
            steps {
                script {
                    sh """
                    docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \\
                    aquasec/trivy:latest image --exit-code 0 \\
                    --severity LOW,MEDIUM,HIGH,CRITICAL \\
                    ${IMAGE_NAME_CLIENT}:${env.BUILD_NUMBER}
                    """
                }
            }
        }

        stage('Push Images to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('', "${DOCKERHUB_CREDENTIALS}") {
                        dockerImageServer.push("${env.BUILD_NUMBER}")
                        dockerImageClient.push("${env.BUILD_NUMBER}")
                    }
                }
            }
        }
    }
}