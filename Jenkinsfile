pipeline {
  agent any

  environment {
    REGISTRY   = 'docker.io'
    IMAGE_NAME = 'sirhipotermia/examen-devops'   
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
    /* ------------------------------------------------------------------ */
    stage('Checkout SCM') {
      steps {
        echo "Clonando repositorio…"
        checkout scm
        sh 'git rev-parse --abbrev-ref HEAD && git log -1 --oneline'
      }
    }

    /* ------------------------------------------------------------------ */
    stage('Build WAR') {
      steps {
        script {
          echo "Compilando proyecto Maven"
        }
        // ‘set -e’ para salir ante primer fallo y ‘set -x’ para log detallado
        sh '''
          set -e -x
          mvn -B clean package -DskipTests
          ls -lh target | grep .war || true
        '''
      }
    }

    /* ------------------------------------------------------------------ */
    stage('Docker Build') {
      steps {
        echo "Construyendo imagen Docker…"
        sh '''
          set -e -x
          docker build -t $IMAGE_NAME:${BUILD_NUMBER} .
        '''
      }
    }

    /* ------------------------------------------------------------------ */
    stage('Docker Push') {
      steps {
        echo "Pusheando imagen a DockerHub…"
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

    /* ------------------------------------------------------------------ */
    stage('Deploy EC2') {
      steps {
        echo "Desplegando contenedor en la instancia…"
        sh '''
          set -e -x
          # Detener y eliminar contenedor previo (si existe)
          docker stop $CONTAINER     || true
          docker rm   $CONTAINER     || true

          # Levantar nueva versión
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
  } /* stages */

  post {
    success {
      echo "Build desplegado correctamente"
    }
    failure {
      echo "la build falló. "
    }
  }
}
