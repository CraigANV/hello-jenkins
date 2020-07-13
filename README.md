

Create a bridge network:

```bash
docker network create jenkins
```

Create the following volumes to share the Docker client TLS certificates needed to connect to the Docker daemon and persist the Jenkins data:

```bash
docker volume create jenkins-docker-certs
docker volume create jenkins-data

# To remove these volumes
docker volume rm jenkins-docker-certs
docker volume rm jenkins-data
```

In order to execute Docker commands inside Jenkins nodes, download and run the docker:dind Docker image:

```bash
docker container run --name jenkins-docker --rm --detach \
  --privileged --network jenkins --network-alias docker \
  --env DOCKER_TLS_CERTDIR=/certs \
  --volume jenkins-docker-certs:/certs/client \
  --volume jenkins-data:/var/jenkins_home \
  --publish 2376:2376 docker:dind
```

Download the jenkinsci/blueocean image and run it as a container
```bash
docker container run --name jenkins-blueocean --rm --detach \
  --network jenkins --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
  --volume jenkins-data:/var/jenkins_home \
  --volume jenkins-docker-certs:/certs/client:ro \
  --publish 8080:8080 --publish 50000:50000 jenkinsci/blueocean
```

Obtain the password from the container logs
```bash
docker container logs jenkins-blueocean
```

Browse to http://localhost:8080
