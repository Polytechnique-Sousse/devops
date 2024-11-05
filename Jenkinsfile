// pipeline {
//     agent any
//     triggers {
//         pollSCM('H/5 * * * *')
//     }
//     environment {
//         DOCKERHUB_CREDENTIALS = credentials('dockerhub')
//         IMAGE_NAME_SERVER = 'naouresdoc/mern-server'
//         IMAGE_NAME_CLIENT = 'naouresdoc/mern-client'
//     }
//     stages {
//         // stage('Checkout') {
//         //     steps {
//         //         git branch: 'main',
//         //             url: 'git@github.com:Polytechnique-Sousse/devops.git',
//         //             credentialsId: 'github-ssh'
//         //     }
//         // }
//         stage('Build Server Image') {
//             steps {
//                 dir('server') {
//                     script {
//                         dockerImageServer = docker.build("${IMAGE_NAME_SERVER}")
//                     }
//                 }
//             }
//         }
//         stage('Build Client Image') {
//             steps {
//                 dir('client') {
//                     script {
//                         dockerImageClient = docker.build("${IMAGE_NAME_CLIENT}")
//                     }
//                 }
//             }
//         }
//         stage('Scan Server Image') {
//             steps {
//                 script {
//                     sh """
//                     docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \\
//                     aquasec/trivy:latest image --exit-code 0 \\
//                     --severity LOW,MEDIUM,HIGH,CRITICAL \\
//                     ${IMAGE_NAME_SERVER}
//                     """
//                 }
//             }
//         }
//         stage('Scan Client Image') {
//             steps {
//                 script {
//                     sh """
//                     docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \\
//                     aquasec/trivy:latest image --exit-code 0 \\
//                     --severity LOW,MEDIUM,HIGH,CRITICAL \\
//                     ${IMAGE_NAME_CLIENT}
//                     """
//                 }
//             }
//         }
//         stage('Push Images to Docker Hub') {
//             steps {
//                 script {
//                     docker.withRegistry('', "${DOCKERHUB_CREDENTIALS}") {
//                         dockerImageServer.push()
//                         dockerImageClient.push()
//                     }
//                 }
//             }
//         }
//     }
// }


pipeline {
    environment {
        registryCredential = 'dockerhub'
        IMAGE_NAME_SERVER = 'naouresdoc/mern-server'
        IMAGE_NAME_CLIENT = 'naouresdoc/mern-client'
        PUSH_SUCCESS = 'false' 
    }
    agent any
    stages {
        stage('cloning Git') {
            steps {
                checkout scm
            }
        }
        stage('Building image server') {
            steps {
                dir('server') {
                    script {
                        dockerImageServer = docker.build("${env.IMAGE_NAME_SERVER}:${env.BUILD_NUMBER}")
                    }
                }
            }
        }

        stage('Building image client') {
            steps {
                dir('client') {
                    script {
                        dockerImageClient = docker.build("${env.IMAGE_NAME_CLIENT}:${env.BUILD_NUMBER}")
                    }
                }
            }
        }

        stage('Vulnerability Scan server') {
            steps {
                script {
                    sh "trivy image --severity HIGH,CRITICAL ${env.IMAGE_NAME_SERVER}:${env.BUILD_NUMBER}"
                }
            }
        }

        stage('Vulnerability Scan client') {
            steps {
                script {
                    sh "trivy image --severity HIGH,CRITICAL ${env.IMAGE_NAME_CLIENT}:${env.BUILD_NUMBER}"
                }
            }
        }

        stage('Deploy Image') {
            steps {
                script {
                    try {
                        docker.withRegistry('', registryCredential) {
                            docker.image("${env.IMAGE_NAME_SERVER}:${env.BUILD_NUMBER}").push()
                           docker.image("${env.IMAGE_NAME_CLIENT}:${env.BUILD_NUMBER}").push()
                            // Set the environment variable
                            sh "echo 'true' > .push_success"
                        }
                    } catch (Exception e) {
                        echo "Image push failed: ${e.getMessage()}"
                        sh "echo 'false' > .push_success"
                        error("Failed to push image to registry")
                    }
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}