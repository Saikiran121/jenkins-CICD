pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "my-spring-boot-app:${BUILD_NUMBER}"
    }
  stages {
    stage('Checkout') {
      steps {
        sh 'echo passed'
        git branch: 'main', url: 'https://github.com/Saikiran121/jenkins-CICD.git'
      }
    }

    stage('Build and Test') {
      steps {
        sh 'ls -ltr'
        // build the project and create a JAR file
        sh 'cd spring-boot-app && mvn clean package'
      }
    }

    stage('Static code analysis') {
      environment {
        SONAR_URL = "http://13.127.21.45:9000"
      }
      steps {
        withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
          sh "cd spring-boot-app && mvn sonar:sonar -Dsonar.login=${SONAR_AUTH_TOKEN} -Dsonar.host.url=${SONAR_URL}"
        }
      }
    }

      stage('Build and Push Docker Image') {
            steps {
                script {
                    // Build and tag Docker image
                    sh "docker build -t ${DOCKER_IMAGE} ."
                    // Push Docker image to registry
                    sh "docker push ${DOCKER_IMAGE}"
                }
            }
        }

    stage('Update Deployment File') {
      environment {
        GIT_REPO_NAME = "jenkins-CICD"
        GIT_USER_NAME = "Saikiran121"
      }
      steps {
        withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
          sh '''
            git config user.email "saikiranbiradar76642@gmail.com"
            git config user.name "Saikiran121"
            BUILD_NUMBER=${BUILD_NUMBER}
            sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" spring-boot-app-manifests/deployment.yml
            git add spring-boot-app-manifests/deployment.yml
            git commit -m "Update deployment image to version ${BUILD_NUMBER}"
            git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
          '''
        }
      }
    }
  }
}
