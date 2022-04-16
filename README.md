# FOSS - Jenkins Server
Servidor Jenkins para Estancias FOSS.


## Requisitos
* Terminal de Línea de Comandos
* [Git](https://git-scm.com/)
* [Docker](https://www.docker.com/)
* Editor de Texto / IDE


## Instalar Docker Desktop
Visitar [Get Started with Docker](https://www.docker.com/get-started/) para descargar y ejecutar el instalador para el sistema operativo apropiado.


## Descargar Imágenes Docker
```shell
docker pull jenkins/jenkins:2.332.2-lts
docker pull docker:20.10.14-dind
```

## Alternativa #1. Ejecutar y Detener Servicios con Docker Compose
```shell
# Crear Red
docker network create foss

# Iniciar Servicios
docker-compose -f compose.yml up -d

# Ver Recursos Creados
docker ps -a
docker volume ls
docker network ls

# Detener Servicios
docker-compose -f compose.yml down

# Eliminar Volúmenes
docker volume rm foss-jenkins-server_certs
docker volume rm foss-jenkins-server_dind-docker
docker volume rm foss-jenkins-server_dind-data
docker volume rm foss-jenkins-server_jenkins-home

# Eliminar Red
docker network rm foss

# Limpieza de Recursos No Utilizados
docker system prune -f
```


## Alternativa #2. Ejecutar y Detener Servicios con Comandos Docker
```shell
# Crear Red
docker network create foss

# Crear Volúmenes
docker volume create foss-jenkins-server_certs
docker volume create foss-jenkins-server_dind-docker
docker volume create foss-jenkins-server_dind-data
docker volume create foss-jenkins-server_jenkins-home

# Iniciar Docker in Docker
docker run --name foss-dind --rm --detach --privileged --network foss --env DOCKER_TLS_CERTDIR=/certs --volume "foss-jenkins-server_certs:/certs/client" --volume "foss-jenkins-server_dind-docker:/var/lib/docker" --volume "foss-jenkins-server_dind-data:/var/jenkins_home" --publish 2376:2376 docker:20.10.14-dind --storage-driver overlay2

# Iniciar Jenkins
docker run --name foss-jenkins --rm --detach --network foss --env DOCKER_HOST=tcp://foss-dind:2376 --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 --publish 8080:8080 --publish 50000:50000 --volume "foss-jenkins-server_jenkins-home:/var/jenkins_home" --volume "foss-jenkins-server_certs:/certs/client:ro" jenkins/jenkins:2.332.2-lts

# Ver Recursos Creados
docker ps -a
docker volume ls
docker network ls

# Detener Servicios
docker stop foss-jenkins 
docker stop foss-dind

# Eliminar Servicios
docker rm -f foss-jenkins 
docker rm -f foss-dind

# Volúmenes
docker volume rm foss-jenkins-server_certs
docker volume rm foss-jenkins-server_dind-docker
docker volume rm foss-jenkins-server_dind-data
docker volume rm foss-jenkins-server_jenkins-home

# Eliminar Red
docker network rm jenkins

# Limpieza de Recursos No utilizados
docker system prune -f
```
