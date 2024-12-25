pipeline {
    agent any

    environment {
        // Set environment variables
        DOCKER_IMAGE_BASE = 'doaahemaid01/my-app'
        IMAGE_TAG = "${env.BUILD_ID}-${new Date().format('yyyyMMddHHmmss')}"
        DOCKER_IMAGE = "${DOCKER_IMAGE_BASE}:${IMAGE_TAG}"
        OPENSHIFT_SERVER = 'https://api.ocp-training.ivolve-test.com:6443' 
        OPENSHIFT_PROJECT = 'doaahemaid' 
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
                    withSonarQubeEnv('sounarqube') {
                        sh './gradlew sonar'
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    sh "docker build -t ${DOCKER_IMAGE} . "
                }
            }
        }

        stage('Push Docker Image to Registry') {
            steps {
                script {
                    // Login to Docker registry and push the image
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credential', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin "
                        sh "docker push ${DOCKER_IMAGE}"
                    }
                }
            }
        }

        stage('Deploy to OpenShift') {
            steps {
                script {
                    // Set OpenShift environment and deploy the Docker image
                   withCredentials([usernamePassword(credentialsId: 'openshift-credentials', usernameVariable: 'OPENSHIFT_USER', passwordVariable: 'OPENSHIFT_PASSWORD')]) {
                sh '''
                    # Log in to OpenShift with username and password
                    oc login ${OPENSHIFT_SERVER} --username=${OPENSHIFT_USER} --password=${OPENSHIFT_PASSWORD} --insecure-skip-tls-verify
                    
                    # Switch to the target project
                    oc project ${OPENSHIFT_PROJECT}

                    # Create a deployment
                    oc create deployment my-app --image=${DOCKER_IMAGE}
                '''
                    }}
                }
            }
        }
    

    post {
        always {
            // Clean up, log out from Docker and OpenShift
            sh 'docker logout'
            sh 'oc logout'
            sh 'docker rmi ${DOCKER_IMAGE} || true'
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}

