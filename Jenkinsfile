// pipeline {
//     agent any

//     options {
//         skipDefaultCheckout()
//     }

//     tools {
//         maven 'maven3'
//     }

    

//     environment {
//         SONAR_URL = "http://3.111.30.234:9000"

//         // JFrog Settings
//         ARTIFACTORY_HOST = '13.204.81.100:8081'
//         DOCKER_REPO = 'local'
//         IMAGE_NAME = 'my-app'
//         IMAGE_TAG = "${BUILD_NUMBER}"

//         // Credentials
//         ARTIFACTORY_CREDS = credentials('jfrog-test')
//     }

//     stages {

//         stage('Checkout') {
//             steps {
//                 echo 'Cloning GitHub Repo...'
//                 git branch: 'main', url: 'https://github.com/mallinagraj/mergecheck.git'
//             }
//         }

//         stage('SonarQube Analysis') {
//             environment {
//                 SONAR_TOKEN = credentials('sonarqube-token')
//             }
//             steps {
//                 script {
//                     withSonarQubeEnv('SonarQubeServer') {
//                         def scannerHome = tool name: 'SonarQubeScanner',
//                                                type: 'hudson.plugins.sonar.SonarRunnerInstallation'

//                         sh """
//                             ${scannerHome}/bin/sonar-scanner \
//                                 -Dsonar.projectKey=amrut \
//                                 -Dsonar.sources=./src \
//                                 -Dsonar.host.url=${SONAR_URL} \
//                                 -Dsonar.login=${SONAR_TOKEN}
//                         """
//                     }
//                 }
//             }
//         }

//         stage('Build Maven Artifact') {
//             steps {
//                 echo 'Building Maven package...'
//                 sh 'mvn clean package'
//             }
//         }

//         stage('Build Docker Image for JFrog') {
//             steps {
//                 script {
//                     echo "Building Docker image..."
//                    sh """
//     docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .

//     docker tag ${IMAGE_NAME}:${IMAGE_TAG} \
//     13.127.91.182:8081/artifactory/docker-local/${IMAGE_NAME}:${IMAGE_TAG}

//     docker tag ${IMAGE_NAME}:${IMAGE_TAG} \
//     13.127.91.182:8081/artifactory/docker-local/${IMAGE_NAME}:latest

// """

//                 }
//             }
//         }

//         stage('Login to JFrog Registry') {
//     steps {
//         script {
//             withCredentials([usernamePassword(credentialsId: 'jfrog-test',
//                                              usernameVariable: 'USR',
//                                              passwordVariable: 'PSW')]) {
//                 sh '''
//                     echo $PSW | docker login 13.127.91.182:8081 -u $USR --password-stdin
//                 '''
//             }
//         }
//     }
// }



//         stage('Push Images to JFrog') {
//             steps {
//                 script {
//                     sh """
//     docker push 13.127.91.182:8081/artifactory/docker-local/${IMAGE_NAME}:${IMAGE_TAG}
//     docker push 13.127.91.182:8081/artifactory/docker-local/${IMAGE_NAME}:latest

// """


//                     echo "Image pushed:"
//                     echo "${ARTIFACTORY_HOST}/${DOCKER_REPO}/${IMAGE_NAME}:${IMAGE_TAG}"
//                 }
//             }
//         }

//         stage('Build DockerHub Image') {
//             steps {
//                 echo "Building DockerHub Image..."
//                 sh "docker build -t jayesh7744/ultimate-cicd:${BUILD_NUMBER} ."
//             }
//         }

//         stage('Push to DockerHub') {
//             steps {
//                 script {
//                     withCredentials([string(credentialsId: 'dockerhub',
//                         variable: 'DOCKERHUB_TOKEN')]) {

//                         sh """
//                             echo "${DOCKERHUB_TOKEN}" | docker login -u "jayesh7744" --password-stdin
//                             docker push jayesh7744/ultimate-cicd:${BUILD_NUMBER}
//                         """
//                     }
//                 }
//             }
//         }
//     }

//     post {
//         always {
//             echo "Cleaning workspace..."
//             cleanWs()
//         }
//     }
// }


pipeline {
    agent any
    
    options {
        skipDefaultCheckout()
    }
    
    tools {
        maven 'maven3'
    }
    
    environment {
        SONAR_URL = "http://3.110.136.214:9000"
        ARTIFACTORY_URL = "http://3.109.200.182:8081/artifactory"
        DOCKERHUB_USER = 'jayesh7744'
        IMAGE_NAME = 'ultimate-cicd'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Cloning GitHub Repo...'
                git branch: 'main', url: 'https://github.com/mallinagraj/mergecheck.git'
            }
        }
        
        stage('SonarQube Analysis') {
            environment {
                SONAR_TOKEN = credentials('sonarqube-token')
            }
            steps {
                script {
                    withSonarQubeEnv('SonarQubeServer') {
                        def scannerHome = tool name: 'SonarQubeScanner', 
                                              type: 'hudson.plugins.sonar.SonarRunnerInstallation'
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                                -Dsonar.projectKey=amrut \
                                -Dsonar.sources=./src \
                                -Dsonar.host.url=${SONAR_URL} \
                                -Dsonar.login=${SONAR_TOKEN}
                        """
                    }
                }
            }
        }
        
        stage('Build Maven Artifact') {
            steps {
                echo 'Building Maven WAR package...'
                sh 'mvn clean package -DskipTests'
            }
        }
        
        stage('Deploy WAR to Artifactory') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'jfrog-test', 
                        usernameVariable: 'JFROG_USER', 
                        passwordVariable: 'JFROG_PASSWORD')]) {
                        
                        sh '''
                            # Find the WAR file
                            WAR_FILE=$(find target -name '*.war' -type f | head -n 1)
                            
                            if [ -z "$WAR_FILE" ]; then
                                echo "ERROR: No WAR file found in target directory"
                                exit 1
                            fi
                            
                            WAR_NAME=$(basename $WAR_FILE)
                            echo "Found WAR file: $WAR_NAME"
                            echo "Uploading to Artifactory..."
                            
                            # Upload WAR file directly to Artifactory using cURL
                            curl -v -u ${JFROG_USER}:${JFROG_PASSWORD} \
                                 -X PUT \
                                 "http://3.109.200.182:8081/artifactory/libs-release-local/${WAR_NAME}" \
                                 -T $WAR_FILE
                            
                            if [ $? -eq 0 ]; then
                                echo "✓ Successfully uploaded WAR to Artifactory"
                            else
                                echo "✗ Failed to upload WAR to Artifactory"
                                exit 1
                            fi
                        '''
                    }
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh "docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }
        
        stage('Push Docker Image to DockerHub') {
            steps {
                script {
                    withCredentials([string(
                        credentialsId: 'dockerhub', 
                        variable: 'DOCKERHUB_TOKEN')]) {
                        sh """
                            echo "\$DOCKERHUB_TOKEN" | docker login -u ${DOCKERHUB_USER} --password-stdin
                            docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                            echo "✓ Successfully pushed Docker image to DockerHub"
                        """
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo "✓ Pipeline completed successfully!"
            echo "WAR artifact: Available in Artifactory"
            echo "Docker image: ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
        }
        failure {
            echo "✗ Pipeline failed. Check the logs above."
        }
        always {
            echo "Cleaning workspace..."
            cleanWs()
        }
    }
}
