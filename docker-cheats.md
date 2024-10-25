# Docker Cheat Sheet

This file serves as a quick reference for useful Docker commands, helping you manage containers, volumes, and networks efficiently. It covers common tasks like starting and stopping containers, accessing running containers, managing Docker volumes and networks, and more. Whether you're troubleshooting or performing routine tasks, this cheat sheet provides the essential commands to streamline your Docker workflow.

## Table of Contents
- [Containers](#containers)
- [Volumes](#volumes)
- [Networks](#networks)

Please feel free to update this README.md if you notice anything that is missing. If you need a different version of Nginx or PHP, kindly fill out the form at [this link](https://docs.google.com/forms/d/1PA9piAEZfOT3rp3SQqsq8ASz2dIYLldU8lQQ-fPzvlQ/prefill).

## Containers

Start and stop the containers
```bash
docker-compose up -d --remove-orphans # to start containers
docker-compose down # to down containers
# use -f <file-name>.yml if file name is other than docker-compose.yml, e.g.
docker-compose -f dev-compose.yml up -d 
```

Note: The `--remove-orphans` option in the docker-compose up command is used to remove any containers that are not defined in the current docker-compose.yml file but were part of previous runs of Docker Compose.


Open a terminal inside container
```bash
docker-compose exec <service-name> bash
# e.g
docker-compose exec phpfpm bash
```

Execute commands directly into the container without entering it by using:
```bash
# to get container ID or image name
docker ps
# then
docker exec -it <container-id-or-image-name> [command]
# e.g.
docker exec -it <container-id-or-image-name> php -v
docker exec -it <container-id-or-image-name> bin/console cache:clean --env=prod
```

## Volumes
```bash
# to remove all volumes associated with the current docker-compose.yml
docker-compose down -v
# inspect a specific volume for details
docker volume inspect my-volume
# or to remove a specific volume
docker volume ls
# then copy the Volume Name, e.g., oro-docker_dbdata
docker volume rm oro-docker_dbdata
# get detailed usage information about volumes
docker system df -v
# remove all unused volumes
docker volume prune
```

## Networks
```bash
# list all docker networks
docker network ls

# create a new bridge network
docker network create oro_local

# inspect a specific network for details
docker network inspect oro

# connect a running container to a network (use service name for docker-compose or container ID from docker ps)
docker network connect oro phpfpm

# disconnect a container from a network (use service name for docker-compose or container ID from docker ps)
docker network disconnect oro phpfpm

# remove a network (only if no containers are connected)
docker network rm oro

# remove all unused networks
docker network prune

```

## 7. Checking Logs

To view logs for a specific service, use:
```bash
docker-compose logs <service_name>
```
If you want to see logs for all services, simply run:
```bash
docker-compose logs
```
