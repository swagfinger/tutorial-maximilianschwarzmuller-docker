##docker installation

install docker desktop
hyperV - Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
container - Enable-WindowsOptionalFeature -Online -FeatureName containers –All
powershell as administrator: wsl.exe --update

## build a docker file

- root folder create a 'Dockerfile'
- build within current directory
- build image

```shell
docker build .
```

## running docker

- after build, it outputs: 'writing image sha256: <hash>'
- images are read-only and need to be rebuilt to add new content
- on port '3000:3000' which reaches application running on port 3000
- this allows us to goto localhost:3000
- containers are concrete instances based off images

```shell
docker run -p 3000:3000 <hash>
```

## stopping

- 'docker ps' lists all running processes
- 'docker stop <container-name>'

```
docker ps
docker stop <X>
```

## exposing an interactive session from built image

- show all docker containers
- '-it' tells us we want an interactive node session

```shell
docker ps -a
docker run -it node
```

## building your own docker image

- with a Dockerfile
- in root of project
- FROM node - tells docker we want to build up our image from another image
- COPY . . - first . where to copy files from, second . where to copy files to in container image

```Dockerfile
FROM node
WORKDIR /app
COPY . /app
RUN npm install
EXPOSE 80
CMD ["node", "server.js"]
```

## image layers

- each command in docker file is like a layer of code execution and dependin on how granular the code it you can streamline the build process but helping docker only runing a layer of code when a change occurs. eg.

- by separating `COPY package.json /app` and `COPY . /app`, any code changes only re-execute this line (`COPY . /app`) onwards
- by using caching, it optimises the build

```Dockerfile
FROM node
WORKDIR /app
COPY package.json /app
RUN npm install
COPY . /app
EXPOSE 80
CMD ["node", "server.js"]   
```

## execute an existing closed container
- as opposed to `docker run`
```Dockerfile
docker start <command-name>

```

docker run (default: attached) - listening to output
vs
docker start (default: detached)

- detached mode -d, can attach later 'docker attach CONTAINER'
- attached mode (no -d)

## interactive mode
-i interactive mode
-t terminal

```
docker run -it <hash>
```

## deleting containers
- you can pass multiple container names at once with space between string

```shell
# remove container
docker rm <name> 

# remove image
docker rmi <name>
```

## remove unused images
```
docker image prune
```
## stopping a docker container
docker stop <name>

## remove stopped containers
-rm remove after stoped

```
docker run -p 3000:80 -d --rm <hash>
```

## getting information about an image
docker images - lists all images

```
docker images 
docker inspect <imageid>
```

## copy from/out already running container
- copy into or out of already running container
- docker cp (souce eg. dummy) then container name : path you want to copy to

```
<!-- copy into container -->
docker cp dummy/. <name>:/test

<!-- copy off container we just copied to above-->
docker cp <name>:/test dummy 
```

## naming images or containers
- tagging images / containers
- giving own names
- containers assign a name via --name
- images get a tag - eg. FROM node:12

```
<!-- container -->
docker run -p 3000:80 -d --rm --name <assign-name> <hash>


<!-- image -->
docker build -t goals:latest .
```

## sharing images on dockerhub or Dockerfile
- login: 'docker login'
- sharing images dont need build step
- share with: docker push <imagename>
- use with: docker pull <imagename>

```
<!-- share -->
docker push <imagename>

<!-- use -->
docker pull <imagename>
```

# Section 3 - Managing data & working volumes
- specify link to volume: VOLUME ["/app/feedback"]

- volumes are folders on host machine harddrive mounted into containers
- different from COPY because its a "link" where changes in one folder will update the other (in container) too.
- volumes persist even if container is shut down. if container restarts - data inside volume is available in the container
- remove volumes: docker rmi feedback-node:volumes
- doocker build -t feedback-node:volumes .

- to get docker volumes, we can use docker volumes ls
- with named volumes - they survive container shutdown - restarting container revives the volume data

```
VOLUME ["/app/feedback"]

<!-- remove volumes -->
docker rmi feedback-node:volumes

<!-- rebuild -->
docker build -t feedback-node:volumes .

<!-- run -->
docker run -d -p 3000:80 --rm --name feedback-app feedback-node:volumes



```

### named volume
- this part -v feedback:/app/feedback (gives name feedback to folder path we want to associate with docker as a volume)
- named volumes are not attached to a container
- named volume data persists when container restarted

```
<!-- adding named volume -->
docker run -d -p 3000:80 --rm --name feedback-app -v feedback:/app/feedback feedback-node:volumes
```

## Bind mounts
changes in source code are not reflected in running container unless a rebuild is performed
- developer sets the path on host machine
- basically you are linking out from snapshop to source code on your machine so its always updated

```powershell
docker run -d -p 3000:80 --rm --name feedback-app -v feedback:/app/feedback -v "<full path on host machine>":/<map to path in container> feedback-node:volumes

<!-- FROM COMMENTS IN UDEMY CHAT -->
<!-- https://stackoverflow.com/questions/41485217/mount-current-directory-as-a-volume-in-docker-on-windows-10 -->
<!--  PowerShell, you use ${PWD}, which gives you the current directory (linux) -->
<!-- windows: -v "%cd%":/app -->
docker run -d -p 3000:80 --name feedback-app -v feedback:/app/feedback -v ${PWD}:/app feedback-node:volumes
```

## combining + merging volumes
ERROR: binding to local folder but it removes Dockerfile script as it is overwritten
SOLUTION: tell docker some internal parts should not be overwritten: achieved by an anonymous volume

below is equivalent to calling this in Dockerfile: 'VOLUME ["/app/node_modules"]'
- basically including the node_modules folder, so we have a link to local folder as well as node_modules (which wasnt there before)
- co-exist: the more specific path wins so with app/node_modules appearing after the local folder, both code is included.

```
docker run -d -p 3000:80 --name feedback-app -v feedback:/app/feedback -v ${PWD}:/app -v /app/node_modules feedback-node:volumes

```

## restarting just the server 
- stop the container
- re-run the container

## changing volumes to read only
only affects the container  
-ro
- a more specific subvolume / folder overwrites parent settings


```
:/<app>:ro

```

## copy vs bind mount
- question is whether COPY is necessary in Dockerfile if we bind mount,
no - but only no for development - we only use command line script for development

because with image created we will rely on Dockerfile in production

## docker ignore 
COPY . . copies everything
.dockerignore file ignores files to commit to docker COPY instruction - add node_modules to .dockerignore
- .git 

## args and env variables
-provide args via --build-arg set on image build eg. docker build
--env on 'docker run' - eg. process.env.PORT
add ENV to Dockerfile and use it for EXPOSE by adding $
on cmmand line can use --env-file ./.env  to point to .env file
```
ENV PORT 80
EXPOSE $PORT
```

## Build arguments
externalize the values to outside the Dockerfile using build time arguments

```
ARG DEFAULT_PORT=80
ENV PORT $DEFAULT_PORT
```

can only use in Dockerfile

but then on command line - you can overwrite the default value specified in dockerfile
eg. to change port number
- instructions add layers so push it lower down in Dockerfile so less subsequent layers are executed
```
docker build -t feedback-node:dev --build-arg DEFAULT_PORT=8000
```
