pipeline {
  agent any
  environment {
    AWS_REGION = 'us-east-1'                                   // Change if needed
    ACCOUNT_ID = '014498646137'
    REPO_NAME  = 'devrescomhch/repo'                                 // Replace with your ECR repo name
    ECR_URI    = "${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${REPO_NAME}"
    BRANCH     = env.BRANCH_NAME ?: 'master'
  }
  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }
    stage('Install & Build') {
      steps {
        script {
          sh '''
            npm install
            npm run build
          '''
        }
      }
    }
    stage('Build Docker Image') {
      steps {
        script {
          sh "docker build -t ${ECR_URI}:${BUILD_NUMBER} ."
        }
      }
    }
    stage('Login & Push to ECR') {
      steps {
        script {
          // Login to ECR
          sh """
            aws ecr get-login-password --region ${AWS_REGION} \
              | docker login --username AWS --password-stdin ${ECR_URI}
          """
          // Tag & Push
          sh """
            docker tag ${ECR_URI}:${BUILD_NUMBER} ${ECR_URI}:latest
            docker push ${ECR_URI}:${BUILD_NUMBER}
            docker push ${ECR_URI}:latest
          """
        }
      }
    }
  }
  post {
    success {
      echo "✅ Image pushed: ${ECR_URI}:${BUILD_NUMBER} and latest"
    }
    failure {
      echo "❌ Build failed"
    }
  }
}
