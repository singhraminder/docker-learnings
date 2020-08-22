# Docker-Cheat-sheet
## Contents:

## Basics

Start & Stop
* `docker start [CONTAINER]` : start a container
* `docker stop [CONTAINER]` : stops a running container
* `docker pause [CONTAINER]` : suspend/pause-processes in a running container
* `docker unpause [CONTAINER]` : resume/unpause processes in a container
* `docker restart [CONTAINER]` : stop a running container and starts it up again
* `docker wait [CONTAINER]`  : block a container untill other contains stop and returns the exit code
* `docker kill [CONTAINER]` : kills a container by sending SIGKILL
* `docker commit -m "commit-msg" -a "author" [CONTAINER] [IMAGE]:[tag]` : commits a new docker image (snapshot of the container)
* `docker attach [CONTAINER]`: attach local stdout, stdin, error streams to a running container [To detach from a running container, use Ctrl + p, Ctrl + q]
* docker exec : Run commands in a container
```
docker exec [options] CONTAINER COMMAND
  -d, --detach        # run in background
  -i, --interactive   # stdin
  -t, --tty           # interactive
Example:
$ docker exec app_web_1 tail logs/development.log
$ docker exec -t -i app_web_1 rails c
```
* docker create  : create a container from an image
```
docker create [options] IMAGE
  -a, --attach               # attach stdout/err
  -i, --interactive          # attach stdin (interactive)
  -t, --tty                  # pseudo-tty
      --name NAME            # name your image
  -p, --publish 5000:5000    # port map (host:container)
      --expose 5432          # expose a port to linked containers
  -P, --publish-all          # publish all ports
      --link container:alias # linking
  -v, --volume `pwd`:/app    # mount (absolute paths needed)
  -e, --env NAME=hello       # env vars
Example:
$ docker create --name app_redis_1 \
  --expose 6379 \
  redis:3.0.2
```
Manage containers using ps/kill
* `docker version` 
* `docker ps`  :show a list of running containers
* `docker ps -a` : shows running and stopped containers.
* `docker kill $ID`
* `docker logs [CONTAINER]` : Lists logs from a runing container
```
$ docker logs CONTAINER_NAME
$ docker logs $ID
$ docker logs $ID 2>&1 | less
$ docker logs -f $ID # Follow log output
```
* `docker inspect [object_name/id]` : lists low level information of an object (json format}
* `docker event [CONTAINER]` : Lists real-time events from a container
* `docker port [CONTAINER]` : shows port mapping of a container
* `docker top [CONTAINER]` : show running processes in a container
* `docker stats [-all] [CONTAINER]` : show live resource usage statistics of a container
* `docker diff [CONTAINER]`: show changes to file/folders on filesystem

### Registry & Repository

A repository is a hosted collection of tagged images that together create the file system for a container.

A registry is a host -- a server that stores repositories and provides an HTTP API for [managing the uploading and downloading of repositories](https://docs.docker.com/engine/tutorials/dockerrepos/)

Docker.com hosts its own index to a central registry which contains a large number of repositories. Having said that, the central docker registry does not do a good job of verifying images and should be avoided if you're worried about security.

A local registry setup can be done using [docker distribution project](https://github.com/docker/distribution) and [local deploy](https://github.com/docker/docker.github.io/blob/master/registry/deploying.md) instructions.

* `docker login` to login to a registry 
* `docker logout` to logout from a registry
* `docker search` searches registry for image
* `docker pull myimage:1.0` Pull an image from a registry 
* `docker push myrepo/myimage:2.0` Push an image to a registry

### Image-Lifecycle
* `docker image ls` List all images that are locally stored with the Docker Engine
* `docker images` OR `docker images -a` -a shows intermediates
* `docker rmi b750fe78269d` Deletes images
* `docker image rm alpine:3.4` Delete an image from the local image store
* `docker history [IMAGE]` Show the history of an image (list of ancestors)
* `docker build .` Build an image from Dockerfile (in current directory)
* `docker build [options] . -t myimage:1.0 --build-arg APP_HOME=$APP_HOME` Build an image from Dockerfile, tag this image & Set build-time variables
* `docker run [options] IMAGE`  see `docker create` for options [$ docker run -it debian:buster /bin/bash] : create + start an container from an image
  - <code>docker run -d [image]</code> > starts a container in the background
  - <code>docker run -p HOSTNAME:PORT [image]</code> > starts a container from image and maps a hostname:port to it
  - <code>docker run --add-host HOSTNAME:IP [image]</code> > starts a container from image and assign a DNS entry
  - <code>docker run --rm [image]</code> > removes a container after it stops
  - <code>docker run -td [image]</code> > starts a container and keep it running
  - <code>docker run -it [image] [cmd]</code> > create, start and run a command inside this container
  - <code>docker run -it -rm [image] [cmd]</code> > create, start and run a command inside this container, and removes the container

* `docker tag myimage:1.0 myrepo/myimage:2.0` Retag a local image with a new image name and tag
* `docker import [url/file]` create an image from a tarball
* `docker save my_image:my_tag | gzip > my_image.tar.gz` saves image to tar archive with all parents, tags and versions
```
docker save ngnix > nginx.tar
```
* `docker load < my_image.tar.gz [tarball/stdin_file]` load an image from an tar-achive as stdin
```
docker load -i < nginx.tar
```
### Container-Lifecycle

* `docker create [IMAGE]` Create a container (without starting it)
* `docker container run --name web -p 5000:80 alpine:3.9` Run a container from the Alpine version 3.9 image, name the running container “web” and expose port 5000 externally, mapped to port 80 inside the container. 
* `docker container stop web` Stop a running container through SIGTERM 
* `docker container kill web` Stop a running container through SIGKILL
* `docker container ls` List the running containers (add --all to include stopped containers) 
* `docker rm [CONTAINER]` Delete a container
* `docker rm -f [CONTAINER]` Delete a running container (kill + rm)
* `docker rm -f $(docker ps -aq)` Delete all running and stopped containers 
* `docker rename [CONTAINER_NAME] [NEW_CONTAINER_NAME]` Rename an existing container
* `docker container logs --tail 100 web` Print the last 100  lines of a container’s logs
* `docker update [CONTAINER]` updates the configuration of a container
* `docker cp CONTAINER:SOURCE-PATH TARGET-PATH-On-Host` COPY a file from container to Host
* `docker cp TARGET-PATH-On-Host CONTAINER:SOURCE-PATH` COPY a file from host to container
### Network

Docker has a networks feature. Docker automatically creates 3 network interfaces when you install it (bridge, host none). A new container is launched into the bridge network by default. To enable communication between multiple containers, you can create a new network and launch containers in it. This enbales containers to communicate to each other while being isolated from containers that are not connected to the network. Furthermore, it allows to map container names to their IP addresses. See [working with networks](https://docs.docker.com/network/) for more details.

* `docker network ls` List the networks
* `docker network rm [NETWORK]` Remove one or more networks
* `docker network inspect [NETWORK]` Show information on one or more networks
* `docker network connect [NETWORK] [CONTAINER]` Connects a container to a network
* `docker network disconnect [NETWORK] [CONTAINER]` Disconnect a container from a network

```
# create a new bridge network with your subnet and gateway for your ip block
docker network create mynetworkname --subnet 101.0.111.0/24 --gateway 101.0.111.254 

# run a nginx container with a specific ip in same block
$ docker run --rm -it nginx --net mynetworkname --ip 101.0.111.2

# curl the ip (assuming this is a public ip block)
$ curl 101.0.111.2
```

### Cleanup

* `docker image prune [-a]` Delete all the images
* `docker stop $(docker ps -a -q)` Stop all running containers
* `docker container prune` Delete stopped containers
* `docker volume prune` Delete all the volumes
* `docker system prune` Cleans up dangling images, containers, volumes, and networks (ie, not associated with a container)
* `docker system prune -a` Additionally remove any stopped containers and all unused images (not just dangling images)
* `docker rm [CONTAINER]` Delete a container (if it is not running)

## Dockerfile syntax
* FROM image/scratch : base image for the build
* MAINTAINER email : metadata of the maintainer
* COPY host-path container-dest : copy file from host into container
* ADD src dest : same as COPY but untars archive and accepts http/urls
* RUN args : RUNs arbitrary commnd inside the container
* USER : sets the default username
* WORKDIR : sets the default working directory
* CMD : sets the default command
* ENV : sets environment variable for container 

```
LABEL version="1.0"
LABEL "com.example.vendor"="ACME Incorporated"
LABEL com.example.label-with-value="foo"
LABEL description="This text illustrates \
that label-values can span multiple lines."

FROM ruby:2.2.2
ENV APP_HOME /myapp
RUN mkdir $APP_HOME
WORKDIR /myapp
VOLUME ["/data"]
ADD file.xyz /file.xyz
COPY --chown=user:group host_file.xyz /path/container_file.xyz
EXPOSE 5900
CMD    ["bundle", "exec", "rails", "server"]
ENTRYPOINT ["executable", "param1", "param2"]
ENTRYPOINT command param1 param2
ENTRYPOINT exec top -b
```

## Docker-compose

```
# docker-compose.yml
version: '2'

services:
  web:
    build: .
    # build from Dockerfile
    context: ./Path
    dockerfile: Dockerfile
    ports:
     - "5000:5000"
    volumes:
     - .:/code
  redis:
    image: redis
```

```
version: "2"
services: 
  web:
    container_name: "web"
    image: java:8 # image name
    command: java -jar /app/app.jar # command to run
    ports: # map ports to the host
      - "4567:4567"
    volumes: # map filesystem to the host
      - ./myapp.jar:/app/app.jar
  mongo: # container name
    image: mongo # image name
```

`docker-compose ps`

`docker-compose start`

`docker-compose stop`

`docker-compose up`

`docker-compose down`


## External Links

[Cheat sheet at dockerlabs.collabnix.com](http://dockerlabs.collabnix.com/docker/cheatsheet/)

[wsargent/docker-cheat-sheet] (https://github.com/wsargent/docker-cheat-sheet)
