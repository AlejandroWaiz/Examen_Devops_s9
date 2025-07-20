pipeline {
  agent any

  environment {
    REGISTRY    = 'docker.io'
    IMAGE_NAME  = 'sirhipotermia/examen-devops'
    HOST_PORT   = '8082'
    APP_PORT    = '8080'
    CONTAINER   = 'examen_devops'
  }

  options {
    disableConcurrentBuilds()
    timeout(time: 30, unit: 'MINUTES')
  }

  stages {
    stage('Checkout') {
      steps {
        echo '🔄 Clonando repo'
        checkout scm
      }
    }

    stage('Build WAR') {
      steps {
        echo '🚧 Compilando con Maven'
        sh 'mvn clean package -DskipTests'
      }
    }

    stage('Docker Build & Push') {
      steps {
        echo '🐳 Construyendo y subiendo imagen'
        withCredentials([usernamePassword(credentialsId: 'docker-hub',
                                          usernameVariable: 'DOCKER_USER',
                                          passwordVariable: 'DOCKER_PASS')]) {
          sh '''
            docker login -u "$DOCKER_USER" -p "$DOCKER_PASS"
            docker build -t $IMAGE_NAME:$BUILD_NUMBER .
            docker push $IMAGE_NAME:$BUILD_NUMBER
            docker logout
          '''
        }
      }
    }

    stage('Deploy') {
      steps {
        echo '🚀 Desplegando contenedor'
        sh '''
          docker rm -f $CONTAINER || true
          docker run -d \
            --name $CONTAINER \
            -p $HOST_PORT:$APP_PORT \
            -e SPRING_DATASOURCE_URL=jdbc:mysql://localhost:3306/municipalidad_la_florida \
            -e SPRING_DATASOURCE_USERNAME=root \
            -e SPRING_DATASOURCE_PASSWORD=SuperS3cret \
            $IMAGE_NAME:$BUILD_NUMBER
          docker ps --filter "name=$CONTAINER"
        '''
      }
    }
  }

  post {
    always {
      cleanWs()
      sh 'docker system prune -af --filter "until=24h" || true'
    }
    success {
      echo '✅ Deploy OK'
    }
    failure {
      echo '❌ Falló el pipeline'
    }
  }
}
