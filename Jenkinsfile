pipeline {
  agent any   // Run on the existing Jenkins node (with Docker + Maven installed)

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

    stage('SonarQube analysis') {
      steps {
        withSonarQubeEnv('sonarqube') {
          withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
            sh 'mvn sonar:sonar -Dsonar.login=$SONAR_TOKEN'
          }
        }
      }
    }

    stage('Run tests') {
      steps {
        sh 'mvn test'
      }
    }

    stage('Docker Build & Push') {
      steps {
        script {
          def tag = "build-${env.BUILD_NUMBER}"

          // Build Docker image
          sh "docker build -t ${IMAGE_NAME}:${tag} ."

          // Login to Docker Hub
          sh "docker login -u $DOCKERHUB -p $DOCKERHUB_PWD"

          // Push Docker image
          sh "docker push ${IMAGE_NAME}:${tag}"

          // Tag and push latest
          sh "docker tag ${IMAGE_NAME}:${tag} ${IMAGE_NAME}:latest"
          sh "docker push ${IMAGE_NAME}:latest"

          env.IMAGE_TAG = tag
        }
      }
    }

    stage('Update Manifests Repo') {
      steps {
        withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
          sh """
            git clone https://github.com/priyankatanneru/k8s-manifests.git
            cd k8s-manifests
            sed -i 's|image:.*|image: ${IMAGE_NAME}:${env.IMAGE_TAG}|g' deployment.yaml
            git config user.email "tannerupriyanka712@gmail.com"
            git config user.name "Priyanka Tanneru"
            git add deployment.yaml
            git commit -m "Update image to ${IMAGE_NAME}:${IMAGE_TAG}"
            git push https://${GITHUB_TOKEN}@github.com/priyankatanneru/k8s-manifests.git
          """
        }
      }
    }

  }
}
