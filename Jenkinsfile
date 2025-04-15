pipeline {
  agent {
    docker {
        image 'docker:20.10.24'
        args '-v /var/run/docker.sock:/var/run/docker.sock'
    }
  }

  environment {
    TAG = "${new Date().format('yyyyMMddHHmmss')}"
    IMAGE_NAME = "yean1108/mochi"
    KUBECONFIG_PATH = "${WORKSPACE}/kubeconfig"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Install Docker CLI') {
      steps {
        sh '''
          apt-get update
          apt-get install -y docker.io
        '''
      }
    }

    stage('Docker Build & Push') {
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

    stage('Deploy to K8s') {
      steps {
        withCredentials([file(credentialsId: 'KUBECONFIG', variable: 'KUBECONFIG_FILE')]) {
          sh '''
            cp $KUBECONFIG_FILE $KUBECONFIG_PATH
            kubectl --kubeconfig=$KUBECONFIG_PATH set image deployment/mochi-deployment mochi=$IMAGE_NAME:$TAG
          '''
        }
      }
    }
  }

  post {
    always {
      echo "Cleaning up dangling images..."
      sh "docker image prune -f || true"
    }
  }
}
