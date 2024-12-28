@Library('shared-library@main') _
pipeline {
     agent { label 'ivolve-build' }

    environment {
        // Set environment variables
        DOCKER_IMAGE_BASE = 'doaahemaid01/my-app'
        IMAGE_TAG = "${env.BUILD_ID}-${new Date().format('yyyyMMddHHmmss')}"
        DOCKER_IMAGE = "${DOCKER_IMAGE_BASE}:${IMAGE_TAG}"
        OPENSHIFT_SERVER = 'https://api.ocp-training.ivolve-test.com:6443' 
        OPENSHIFT_PROJECT = 'doaahemaid' 
        SONAR_ENV = 'sounarqube'
    }

    stages {
        stage('Git Checkout') {
            steps {
                checkout scm
            }
        }

         stages {
        stage('Initialize') {
            steps {
                script {
                    makeGradlewExecutable()
                }
            }
        }
        stage('Build') {
            steps {
                script {
                    cleanAndBuild()
                }
            }
        }
        stage('Test') {
            steps {
                script {
                    runUnitTests()
                }
            }
        }
        stage('SonarQube Analysis') {
            steps {
                script {
                    runSonarQubeAnalysis(SONAR_ENV)
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                  dockerBuildAndPush(DOCKER_IMAGE,'docker-hub-credentials')
                }
            }
        }

       
        stage('Deploy to OpenShift') {
            steps {
                script {
                    deployToOpenShift(OPENSHIFT_SERVER, OPENSHIFT_PROJECT, DOCKER_IMAGE, 'openshift-credentials')
                   
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

