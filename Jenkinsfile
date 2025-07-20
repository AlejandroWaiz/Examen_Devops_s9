pipeline {
  agent any

  environment {
    REGISTRY   = 'docker.io'
    IMAGE_NAME = 'tuusuario/examen-devops'   //  ← tu usuario Docker Hub
    HOST_PORT  = '8082'
    APP_PORT   = '8080'
    CONTAINER  = 'examen_devops'
  }

  options {
    buildDiscarder(logRotator(numToKeepStr: '10'))
    ansiColor('xterm')
    timestamps()
  }

  stages {

    stage('Checkout SCM') {
      steps {
        echo '🛎️  Clonando repositorio…'
        checkout scm
        sh 'git rev-parse --abbrev-ref HEAD && git log -1 --oneline'
      }
    }

    stage('Build WAR') {
      steps {
        echo '🏗️  Compilando proyecto Maven…'
        sh '''
          set -e -x
          mvn -B clean package -DskipTests
          echo "--- WAR generado ---"
          ls -lh target/*.war
        '''
      }
    }

    stage('Docker Build') {
      steps {
        echo '🐳  Construyendo imagen Docker…'
        sh '''
          set -e -x
          docker build -t $IMAGE_NAME:${BUILD_NUMBER} .
        '''
      }
    }

    stage('Docker Push') {
      steps {
        echo '📤  Subiendo imagen a Docker Hub…'
        withCredentials([usernamePassword(credentialsId: 'docker-hub',
                                          usernameVariable: 'DOCKER_USER',
                                          passwordVariable: 'DOCKER_PASS')]) {
          sh '''
            set -e -x
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
            docker push $IMAGE_NAME:${BUILD_NUMBER}
            docker logout
          '''
        }
      }
    }

    stage('Deploy EC2') {
      steps {
        echo '🚀  Desplegando contenedor…'
        sh '''
          set -e -x
          docker stop $CONTAINER || true
          docker rm   $CONTAINER || true

          docker run -d --name $CONTAINER --network backend \
            -p $HOST_PORT:$APP_PORT \
            -e SPRING_DATASOURCE_URL=jdbc:mysql://mysql:3306/municipalidad_la_florida \
            -e SPRING_DATASOURCE_USERNAME=root \
            -e SPRING_DATASOURCE_PASSWORD=SuperS3cret \
            -e SPRING_PROFILES_ACTIVE=prod \
            $IMAGE_NAME:${BUILD_NUMBER}

          docker ps --filter "name=$CONTAINER"
        '''
      }
    }
  }

  post {
    success {
      echo "✅  Build #${BUILD_NUMBER} desplegado: http://<IP_EC2>:$HOST_PORT/usuariosBuild/"
    }
    failure {
      echo '❌  La build falló. Revisa las etapas anteriores.'
    }
  }
}
