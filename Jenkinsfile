pipeline {
    agent any

    /* ---------- variables de entorno ---------- */
    environment {
        REGISTRY   = 'docker.io'
        IMAGE_NAME = 'sirhipotermia/examen-devops'   // cambia “tuusuario”
        HOST_PORT  = '8082'
        APP_PORT   = '8080'
        CONTAINER  = 'examen_devops'
    }

    /* ---------- opciones globales ---------- */
    options {
        disableConcurrentBuilds()
        timeout(time: 30, unit: 'MINUTES')
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Clonando repositorio'
                checkout scm
            }
        }

        stage('Build WAR') {
            steps {
                echo 'Compilando proyecto Maven'
                sh '''
                    set -e -x
                    MAVEN_OPTS="-Xmx1024m" mvn -T1C -B clean package -DskipTests
                '''
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Construyendo imagen Docker'
                sh '''
                    set -e -x
                    docker build --memory 1g -t $IMAGE_NAME:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Docker Push') {
            steps {
                echo 'Subiendo imagen a Docker Hub'
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
                echo 'Desplegando contenedor'
                sh '''
                    set -e -x
                    docker stop $CONTAINER || true
                    docker rm   $CONTAINER || true

                    docker run -d --name $CONTAINER --network backend \
                    --memory="1g" \
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
        always {
            cleanWs()
            sh 'docker system prune -af --filter "until=24h" || true'
        }
        success {
            echo 'Despliegue completado correctamente'
        }
        failure {
            echo 'La build falló; revisa la consola para detalles'
        }
    }
}
