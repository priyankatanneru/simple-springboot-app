pipeline {
  agent any

  tools {
    jdk 'JDK17'
    maven 'Maven_3_9'
  }

  environment {
    DOCKERHUB = "chikky712"
    IMAGE_NAME = "${DOCKERHUB}/springboot-demo"
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', credentialsId: 'github-creds', url: 'https://github.com/priyankatanneru/simple-springboot-app.git'
      }
    }

    stage('Build') {
      steps {
        sh 'mvn -B -DskipTests=false clean package'
      }
    }

    stage('Docker Build & Push') {
      steps {
        script {
          def tag = "build-${env.BUILD_NUMBER}"
          sh "docker build -t ${IMAGE_NAME}:${tag} ."
          sh "docker login -u $DOCKERHUB -p $DOCKERHUB_PWD"
          sh "docker push ${IMAGE_NAME}:${tag}"
          sh "docker tag ${IMAGE_NAME}:${tag} ${IMAGE_NAME}:latest"
          sh "docker push ${IMAGE_NAME}:latest"
          env.IMAGE_TAG = tag
        }
      }
    }
  }
}
