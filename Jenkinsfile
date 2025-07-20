pipeline {
  agent any
  environment {
    REGISTRY    = 'docker.io'
    IMAGE_NAME  = 'tuusuario/sirhipotermia'   
    BUILD_DIR   = 'Actividad_Usuarios_REST/UsuariosREST'
    HOST_PORT   = '8082'   
    APP_PORT    = '8080'   
    CONTAINER   = 'examen_devops'
  }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Build WAR') {
  steps {
    dir("$BUILD_DIR") {
      sh 'mvn -B clean package -DskipTests'
    }
  }
}
    stage('Docker Build') {
      steps {
        dir("$BUILD_DIR") {
          sh "docker build -t $REGISTRY/$IMAGE_NAME:${env.BUILD_NUMBER} ."
        }
      }
    }
    stage('Docker Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'docker-hub',
                                          usernameVariable: 'DOCKER_USER',
                                          passwordVariable: 'DOCKER_PASS')]) {
          sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
          sh "docker push $REGISTRY/$IMAGE_NAME:${env.BUILD_NUMBER}"
        }
      }
    }
    stage('Deploy EC2') {
      steps {
        sh "docker stop $CONTAINER || true && docker rm $CONTAINER || true"
        sh """
           docker run -d --name $CONTAINER --network backend \
           -p $HOST_PORT:$APP_PORT \
           -e SPRING_DATASOURCE_URL=jdbc:mysql://mysql:3306/municipalidad_la_florida \
           -e SPRING_DATASOURCE_USERNAME=root \
           -e SPRING_DATASOURCE_PASSWORD=SuperS3cret \
           -e SPRING_PROFILES_ACTIVE=prod \
           $REGISTRY/$IMAGE_NAME:${env.BUILD_NUMBER}
        """
      }
    }
  }
  post { success { echo '🚀 Deploy listo en http://<IP_EC2>:' + HOST_PORT } }
}
