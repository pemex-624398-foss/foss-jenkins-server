# FOSS - Jenkins Server
Servidor Jenkins para Estancias FOSS.


## Requisitos
* Terminal de Línea de Comandos
* [Git](https://git-scm.com/)
* [Docker](https://www.docker.com/)
* Editor de Texto / IDE


## Instalar Docker
Seguir las instrucciones para cada sistema operativo.
* Windows y Mac: [Docker Desktop](https://www.docker.com/get-started/) 
* Linux: [Docker](https://docs.docker.com/engine/install/)


## Descargar Imágenes Docker
```shell
docker pull jenkins/jenkins:2.332.2-lts
docker pull docker:20.10.14-dind
```


## Ejecutar Jenkins

### Alternativa #1: Docker CLI
```shell
##
## Iniciar Jenkins
##
# 1. Crear Red
docker network create foss

# 2. Crear Volúmenes
docker volume create foss-jenkins-server_certs
docker volume create foss-jenkins-server_dind-docker
docker volume create foss-jenkins-server_dind-data
docker volume create foss-jenkins-server_jenkins-home

# 3. Iniciar Docker in Docker
docker run --name foss-dind --rm --detach --privileged --network foss --env DOCKER_TLS_CERTDIR=/certs --volume "foss-jenkins-server_certs:/certs/client" --volume "foss-jenkins-server_dind-docker:/var/lib/docker" --volume "foss-jenkins-server_dind-data:/var/jenkins_home" --publish 2376:2376 docker:20.10.14-dind --storage-driver overlay2

# 4. Iniciar Jenkins
docker run --name foss-jenkins --rm --detach --network foss --env DOCKER_HOST=tcp://foss-dind:2376 --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 --publish 8080:8080 --publish 50000:50000 --volume "foss-jenkins-server_jenkins-home:/var/jenkins_home" --volume "foss-jenkins-server_certs:/certs/client:ro" jenkins/jenkins:2.332.2-lts

# 5. Obtener password inicial de administrador
docker exec foss-jenkins cat /var/jenkins_home/secrets/initialAdminPassword

# 6. Abrir URL http://localhost:8080 y pegar el password inicial de administrador para iniciar configuración

############################################################

##
## Eliminar Jenkins
##
# 1. Detener Jenkins y Docker in Docker
docker stop foss-jenkins
docker stop foss-dind

# 2. Eliminar Jenkins y Docker in Docker
docker rm -f foss-jenkins
docker rm -f foss-dind

# 3. Eliminar Volúmenes
docker volume rm foss-jenkins-server_certs
docker volume rm foss-jenkins-server_dind-docker
docker volume rm foss-jenkins-server_dind-data
docker volume rm foss-jenkins-server_jenkins-home

# 4. Eliminar Red
docker network rm foss

# 5. Limpieza
docker system prune -f
```

### Alternativa #2: Docker Compose
```shell
##
## Iniciar
##
# 1. Crear Red
docker network create foss

# 2. Iniciar Servicios
docker-compose -f compose.yml up -d

# 3. Obtener password inicial de administrador
docker exec foss-jenkins cat /var/jenkins_home/secrets/initialAdminPassword

# 4. Abrir URL http://localhost:8080 y pegar el password inicial de administrador para iniciar configuración

############################################################

##
## Eliminar
##
# 1. Eliminar Servicios
docker-compose -f compose.yml down

# 2. Eliminar Volúmenes
docker volume rm foss-jenkins-server_certs
docker volume rm foss-jenkins-server_dind-docker
docker volume rm foss-jenkins-server_dind-data
docker volume rm foss-jenkins-server_jenkins-home

# 3. Eliminar Red
docker network rm foss

# 4. Limpieza
docker system prune -f
```
