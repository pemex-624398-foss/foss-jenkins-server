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
```shell
##
## Iniciar Jenkins
##
# 1. Crear Red
docker network create foss

# 2. Crear Volúmenes
## Classic
docker volume create foss-jenkins-server_certs
docker volume create foss-jenkins-server_dind-docker
docker volume create foss-jenkins-server_jenkins-home

## Blue Ocean
docker volume create foss-jenkins-server_certs-blue
docker volume create foss-jenkins-server_dind-docker-blue
docker volume create foss-jenkins-server_jenkins-home-blue

# 3. Iniciar Docker in Docker
## Classic
docker run --name foss-dind --restart unless-stopped --detach --privileged --network foss --network-alias docker --env DOCKER_TLS_CERTDIR=/certs --volume "foss-jenkins-server_certs:/certs/client" --volume "foss-jenkins-server_dind-docker:/var/lib/docker" --volume "foss-jenkins-server_jenkins-home:/var/jenkins_home" --publish 2376:2376 docker:20.10.14-dind --storage-driver overlay2

## Blue Ocean
docker run --name foss-dind --restart unless-stopped --detach --privileged --network foss --network-alias docker --env DOCKER_TLS_CERTDIR=/certs --volume "foss-jenkins-server_certs-blue:/certs/client" --volume "foss-jenkins-server_dind-docker-blue:/var/lib/docker" --volume "foss-jenkins-server_jenkins-home-blue:/var/jenkins_home" --publish 2376:2376 docker:20.10.14-dind --storage-driver overlay2

# 4. Iniciar Jenkins
## Classic
docker build -t jenkins-docker:2.332.2-lts ./classic
docker run --name foss-jenkins --restart unless-stopped --detach --privileged --network foss --env DOCKER_HOST=tcp://docker:2376 --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 --publish 8080:8080 --publish 50000:50000 --volume "/var/run/docker.sock:/var/run/docker.sock" --volume "foss-jenkins-server_jenkins-home:/var/jenkins_home" --volume "foss-jenkins-server_certs:/certs/client:ro" jenkins-docker:2.332.2-lts

## Blue Ocean
docker build -t jenkins-blue:2.332.2-lts ./blue-ocean
docker run --name foss-jenkins-blue --restart unless-stopped --detach --privileged --network foss --env DOCKER_HOST=tcp://docker:2376 --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 --publish 8080:8080 --publish 50000:50000 --volume "/var/run/docker.sock:/var/run/docker.sock" --volume "foss-jenkins-server_jenkins-home-blue:/var/jenkins_home" --volume "foss-jenkins-server_certs-blue:/certs/client:ro" jenkins-blue:2.332.2-lts

# 5. Obtener Password Inicial de Administrador
docker exec foss-jenkins cat /var/jenkins_home/secrets/initialAdminPassword

# 6. Abrir URL http://localhost:8080 y Pegar el Password Obtenido

############################################################

##
## Eliminar Jenkins
##
# 1. Detener Jenkins y Docker in Docker
docker stop foss-jenkins
docker stop foss-dind

# 2. Eliminar Jenkins y Docker in Docker
docker rm -f foss-jenkins
docker rm -f foss-jenkins-blue
docker rm -f foss-dind

# 3. Eliminar Volúmenes
## Classic
docker volume rm foss-jenkins-server_certs
docker volume rm foss-jenkins-server_dind-docker
docker volume rm foss-jenkins-server_jenkins-home

## Blue Ocean
docker volume rm foss-jenkins-server_certs-blue
docker volume rm foss-jenkins-server_dind-docker-blue
docker volume rm foss-jenkins-server_jenkins-home-blue

# 4. Eliminar Red
docker network rm foss

# 5. Limpieza
docker system prune -f
```
