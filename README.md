# Jenkins CI/CD Pipeline 
## Overview
This guide outlines the configuration and execution of a Jenkins CI/CD pipeline that integrates code testing, quality analysis, containerization, and deployment. The pipeline uses a shared Jenkins library and runs in a distributed setup with a master and slave architecture, each hosted on separate AWS EC2 instances.

![diagram-export-12-29-2024-4_19_48-PM](https://github.com/user-attachments/assets/5757b7da-59a3-4de5-bbd3-b6cf5184147c)

### 1. Jenkins Master Configuration
- **Instance Setup**: Jenkins master is deployed on an EC2 instance with necessary resources for orchestrating builds.
- **Plugins and Tools**:
  - Pipeline plugin
  - Git plugin
  - SonarQube Scanner plugin
  - Docker Pipeline plugin
- **Pipeline Credentials**

| **ID**                      | **Purpose**                                                      |
|-----------------------------|------------------------------------------------------------------|
| `github-credentials`        | Used for authenticating with GitHub repositories.                |
| `docker-hub-credentials`    | Used for Docker Hub login to push and pull Docker images.        |
| `openshift-credentials`     | Used for accessing OpenShift clusters during pipeline execution. |
| `jenkins-master-ssh-key`    | SSH private key for accessing Jenkins master node or servers.    |
| `sonarqube-token`           | Authentication token for integrating SonarQube.                  | 

### 2. Jenkins Slave Configuration
- **Instance Setup**: Jenkins slave is deployed on a separate EC2 instance and labeled `ivolve-build`.
- **Connection**: The slave node connects to the master using an SSH agent.

  ![2024-12-28 21_31_11-Greenshot](https://github.com/user-attachments/assets/3ed17aca-6c54-459d-bda2-547a20a76ffa)


### 3. Shared Library Setup
- **Purpose**: Centralized reusable scripts and functions for pipeline stages.
- **Usage**: The shared library is hosted in a [Git repository](https://github.com/Doaa-hemaid/Shared-Library.git) and referenced using `@Library('shared-library@main')` in the Jenkinsfile.

![chrome-capture (62)](https://github.com/user-attachments/assets/56e4dd4a-bb83-45cd-8920-d4276126bf1b)


### 4. Pipeline Stages

 #### Stage 1: JUnit Test Integration
- **Description**: Runs unit tests using JUnit to validate the code functionality.
- **Implementation**:
  ```groovy
  script {
      runUnitTests()
  }
  ```

 #### Stage 2: Gradle Build Process
- **Description**: Builds the application and packages artifacts using Gradle.
- **Implementation**:
  ```groovy
  script {
      cleanAndBuild()
  }
  ```

#### Stage 3: SonarQube Analysis
- **Description**: Analyzes code quality and generates reports using SonarQube.
- **Integration**:
  - Server: Configured with the environment variable `SONAR_ENV`.
  ```groovy
  script {
      runSonarQubeAnalysis(SONAR_ENV)
  }
  ```

 #### Stage 4: Docker Build and Push
- **Description**: Builds a Docker image of the application and pushes it to Docker Hub.
- **Steps**:
  - Build the image.
  - Authenticate and push the image.
  ```groovy
  script {
      dockerBuildAndPush(DOCKER_IMAGE, 'docker-hub-credentials')
  }
  ```

#### Stage 5: OpenShift Deployment
- **Description**: Deploys the application to an OpenShift cluster using rolling updates.
- **Implementation**:
  - Connects to the OpenShift cluster using the API server.
  - Deploys to the specified project using `oc` commands.
  ```groovy
  script {
      deployToOpenShift(OPENSHIFT_SERVER, OPENSHIFT_PROJECT, DOCKER_IMAGE, 'openshift-credentials')
  }
  ```
  ![2024-12-29 20_42_38-EC2 Instance Connect _ us-east-1](https://github.com/user-attachments/assets/50262ee7-6913-4f96-a5d2-85c98d23ec87)

### 5. Post Actions
The following actions are performed after the pipeline execution:

- **Docker Logout**: Logs out from Docker.
- **OpenShift Logout**: Logs out from OpenShift.
- **Docker Image Removal**: Removes the Docker image to free up space.


### 6. Pipeline Execution Summary
- ### **Successful Pipeline Execution**
  ![2024-12-28 21_30_16-Greenshot](https://github.com/user-attachments/assets/5395440d-c08f-4508-a5a1-fb8437a842bc)

- ### **SonarQube Analysis**

  ![2024-12-28 21_22_50-ivolve-project-quality - Overview - SonarQube Community Build](https://github.com/user-attachments/assets/1938441f-8735-46fb-b939-e7491594fb8f)

- ### **App Deployment Results**

  ![2024-12-28 21_56_10-iVolve Technologies](https://github.com/user-attachments/assets/6eb70660-1192-4297-a629-4a54a4a4a20a)



