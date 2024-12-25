pipeline {
    agent any

    environment {
        // Set environment variables
        REGISTRY = 'your-docker-registry' // Replace with your Docker registry URL
        IMAGE_NAME = 'your-image-name'     // Replace with the name of your Docker image
        OPENSHIFT_PROJECT = 'your-openshift-project' // Replace with your OpenShift project
    }

    stages {
        stage('Git Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Unit Test and Build JAR') {
            steps {
                script {
                    // Give execute permissions to gradlew
                    sh 'chmod +x gradlew'
                    
                    // Clean and Build the project
                    sh './gradlew clean build'
                    
                    // Run unit tests
                    sh './gradlew test'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    // Run SonarQube analysis
                    withSonarQubeEnv() {
                        sh './gradlew sonar'
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    sh "docker build -t ${REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER} ."
                }
            }
        }

        stage('Push Docker Image to Registry') {
            steps {
                script {
                    // Login to Docker registry and push the image
                    withCredentials([usernamePassword(credentialsId: 'docker-credentials-id', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin ${REGISTRY}"
                        sh "docker push ${REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER}"
                    }
                }
            }
        }

        stage('Deploy to OpenShift') {
            steps {
                script {
                    // Set OpenShift environment and deploy the Docker image
                    withCredentials([usernamePassword(credentialsId: 'openshift-credentials-id', usernameVariable: 'OC_USERNAME', passwordVariable: 'OC_PASSWORD')]) {
                        sh 'oc login --username=$OC_USERNAME --password=$OC_PASSWORD --insecure-skip-tls-verify'
                        sh "oc project ${OPENSHIFT_PROJECT}"
                        sh "oc new-app ${REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER} --name ${IMAGE_NAME}"
                    }
                }
            }
        }
    }

    post {
        always {
            // Clean up, log out from Docker and OpenShift
            sh 'docker logout'
            sh 'oc logout'
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}

