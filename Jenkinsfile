pipeline {
  agent any

  environment {
    TAG = "${new Date().format('yyyyMMddHHmmss')}"
    IMAGE_NAME = "yean1108/mochi"
    CONTAINER_NAME = "mochi-app"
  }

  stages {
    stage('Install Docker CLI') {
      steps {
        sh '''
          if ! command -v docker >/dev/null 2>&1; then
            echo "Installing Docker CLI..."
            apt-get update && apt-get install -y docker.io
          else
            echo "Docker already installed"
          fi
        '''
      }
    }

    stage('Build & Push Image') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'Dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh '''
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
            docker build -t $IMAGE_NAME:$TAG .
            docker tag $IMAGE_NAME:$TAG $IMAGE_NAME:latest
            docker push $IMAGE_NAME:$TAG
            docker push $IMAGE_NAME:latest
          '''
        }
      }
    }

    stage('Deploy: Run Locally') {
      steps {
        sh '''
          echo "Stopping old container if running..."
          docker stop $CONTAINER_NAME || true
          docker rm $CONTAINER_NAME || true

          echo "Starting new container from $IMAGE_NAME:$TAG..."
          docker run -d --name $CONTAINER_NAME -p 3000:80 $IMAGE_NAME:$TAG
        '''
      }
    }
  }

  post {
    success {
      echo "🚀 Mochi app deployed locally at http://localhost:3000"
    }
    always {
      sh "docker image prune -f || true"
    }
  }
}
