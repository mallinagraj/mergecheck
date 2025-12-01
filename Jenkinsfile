pipeline {
    agent any

    options {
        skipDefaultCheckout()
    }

    
    tools {
        maven 'maven3'
    }

    environment {
        SONAR_URL = "http://13.232.221.191:9000"
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
                // Get the full path of the SonarQubeScanner installation
                def scannerHome = tool name: 'SonarQubeScanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
                
                // Run sonar-scanner using full path
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

        stage('Build Artifact') {
            steps {
                echo 'Building Maven Artifact...'
                sh 'mvn clean package'
            }
        }

        stage('Docker Image Build') {
            steps {
                echo 'Building Docker Image...'
                sh "docker build -t jayesh7744/ultimate-cicd:${BUILD_NUMBER} ."
            }
        }

        stage('Push to DockerHub') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'dockerhub', variable: 'DOCKERHUB_TOKEN')]) {
                        sh """
                            echo "${DOCKERHUB_TOKEN}" | docker login -u "jayesh7744" --password-stdin
                            docker push jayesh7744/ultimate-cicd:${BUILD_NUMBER}
                        """
                    }
                }
            }
        }

    }

    post {
        always {
            echo "Cleaning up workspace..."
            cleanWs()
        }
    }
}
