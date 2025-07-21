Descripcion
En este proyecto se desarrollo una api rest para gestionar usuarios en una base de datos mysql

Ciclo de vida
Planificacion y analisis
Se definio usar una instancia t3.micro en ec2 dentro del free tier (no en la zona, tuve que hacerlo pq tenia problemas con la memoria y me di cuenta muy tarde que era por una configuracion del jenkinsfile y no queria empezar todo de nuevo a esta hora xD) y evaluar usar rds mysql free tier
Diseno de infraestructura
Se planeo un volumen de 30 giB gp2 para ebs, swapfile de 1 giB, seguridad en puertos 22,8081,8082 y 3306
Implementacion
Se instalo java 17, docker y jenkins en ec2, se creo una red docker llamada backend, se levanto mysql en un contenedor y se monto la api en tomcat
CI CD
Se configuro un jenkinsfile con etapas de checkout, build maven, build docker, push a docker hub y deploy en ec2
Despliegue
Jenkins levanta la imagen sirhipotermia/examen-devops y corre el contenedor exponiendo el puerto 8082
Pruebas
Se probaron los endpoints con curl y postman para listar, crear, actualizar y eliminar usuarios
Documentacion
Se agrego springdoc para exponer swagger ui en /swagger-ui.html y /v3/api-docs

Parametros de conexion
spring.datasource.url=jdbc:mysql://mysql:3306/municipalidad_la_florada
spring.datasource.username=root
spring.datasource.password=SuperS3cret
spring.datasource.port=3306
spring.datasource.schema=municipalidad_la_florada

Entorno ec2
AMI ubuntu 22.04, tipo t3.micro, ebs gp2 30 giB, swapfile 1 giB, seguridad puertos 22,8081,8082,3306

GitHub y Jenkins
Se uso el repositorio github.com/AlejandroWaiz/Examen_Devops_s9 y se configuro un job jenkins pipeline from scm con credenciales de docker hub

Pipeline
Etapa 1 checkout
Etapa 2 maven clean package
Etapa 3 docker build
Etapa 4 docker push
Etapa 5 deploy docker run

Despliegue docker
El contenedor examendevops corre en la red backend y se conecta al contenedor mysql para acceder a la base de datos

Pruebas crud
POST    /usuariosBuild/user
PUT     /usuariosBuild/user/{id}
GET     /usuariosBuild/user y /usuariosBuild/user/{id}
DELETE  /usuariosBuild/user/{id}