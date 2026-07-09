pipeline {
  agent {
    docker {
      image 'maven:3.8.8-eclipse-temurin-8'
      args '-v /var/run/docker.sock:/var/run/docker.sock'
    }
  }

  options {
    timestamps()
    disableConcurrentBuilds()
  }

  environment {
    DOCKER_IMAGE = 'boxfuse-origin-6'
    CONTAINER_NAME = 'boxfuse-origin-6'
    PROD_HOST = '45.38.139.30'
    PROD_USER = 'serg'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build WAR') {
      steps {
        sh 'mvn clean package -DskipTests'
      }
    }

    stage('Build Docker image') {
      steps {
        sh 'docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .'
      }
    }

    stage('Deploy to prod') {
      steps {
        sh '''
          docker save ${DOCKER_IMAGE}:${BUILD_NUMBER} | ssh -o StrictHostKeyChecking=no ${PROD_USER}@${PROD_HOST} "docker load"

          ssh ${PROD_USER}@${PROD_HOST} "
            docker stop ${CONTAINER_NAME} || true
            docker rm ${CONTAINER_NAME} || true
            docker run -d --name ${CONTAINER_NAME} -p 8080:8080 ${DOCKER_IMAGE}:${BUILD_NUMBER}
          "
        '''
      }
    }
  }
}
