# 01 Getting Started

## docker installation

install docker desktop
hyperV - Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
container - Enable-WindowsOptionalFeature -Online -FeatureName containers –All
powershell as administrator: wsl.exe --update

# 02 Docker Images & Containers: Fundamentals

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

# 03 - Managing data & working volumes
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

# 04 Networking - Cross Container Communication
- connecting container to localhost machine
- reaching out to www
- container to container connection

### container to WWW
getting mongo to work in Dockerfile
- works out of the box to connect to web

### container to local mongoDb
replace localhost with 'host.docker.internal' translates to ip address of host machine as seen from inside container

```js
mongoose.connet(
  'mongodb://host.docker.internal:27017/swfavorites',
)

```

### container to container communication

#### MANUAL CONFIG
- make a mongodb container
- dockerhub -search mongodb image
- connecting to another container with mongodb running inside, we inspect that container and get the url
- using 'docker container inspect mongodb' can get 'IPAddress' under 'network settings':  "IPAddress": "172.17.0.3",
- so back on the app.js we change out the domain 

```
docker run mongo
docker run -d --name mongodb mongo

<!-- get info about container -->
docker container inspect mongodb
```

```js
// app.js
mongoose.connect(
  'mongodb://172.17.0.3:27017/swfavorites',
  
```

```shell
docker run --name favorites -d --rm -p 3000:3000 favorites-node
```

#### CONTAINER NETWORKS "NETWORKS" - MORE STREAMLINED METHOD CONTAINER TO CONTAINER COMMUNICATION
- add containers to same "network" 'docker run --network <name>'
- and docker automatically links up containers and ip lookup
- isolated containers created able to talk to each other
- unlike volumes, docker will not automatically create networks for you
- 'docker network create <name>'
- then in the code, instead of using a hardcoded domain like localhost, you put the name of the container you want to connect to

```
docker network create favorites-net
docker run -d --name mongodb --network favorites-net mongo
```

```js
// app.js - connecting to mongodb container
mongoose.connect('mongodb://mongodb:21017/swfavorites')
```

### rebuild nodejs side after code change

```
docker build -t favorites-node .

<!-- run node application again -->
docker run --name favorites --network favorites-net -d --rm -p 3000:3000 favorites-node
```

## 05 multi-container apps

- re-iterate what we have learnt in previous sections
- DATABASE / BACKEND / FRONTEND

#### GETTING mongodb up and running
```
docker run --name mongodb --rm -d -p 27017:27017 mongo
```

#### GETTING backend up and running
- create Dockerfile in backend/

```Dockerfile
<!-- create image -->
docker build -t goals-node .


<!-- run backend container -->
docker run --name goals-backend --rm goals-node
```

change backend to:
```js
//backend/app.js

mongoose.connect(
  'mongodb://host.docker.internal:27017/course-goals',
```

rebuild
```
docker build -t goals-node .
```

re-run
```
docker run --name goals-backend --rm goals-node
```

stop goals-backend
```
docker stop goals-backend
```

stopped so we can expose ports + run in detached mode: -d -p 80:80
```
docker run --name goals-backend --rm -d -p 80:80 goals-node
```

### dockerize frontend
- create frontend Dockerfile
```
<!-- tagged 'goals-react' -->
docker build -t goals-react .

<!-- run container. name it goals-frontend. detached mode, expose port 3000. run in interactive mode -->
docker run --name goals-frontend --rm -d -p 3000:3000 -it goals-react
```

### set up a network rather
#### step 1
- root folder (outside projects folders): 

```shell
docker network create goals-net
```

### MONGODB
#### step 2 (BETTER CODE LATER ON) - add mongo to network
 
<!-- put mongodb in network - remember removing the port is okay because the 3 containers are in a network -->
docker run --name mongodb --rm -d --network goals-net mongo
#---------------------------------------------------------------------------------

# BACKEND
# step 3 - update connect to mongodb
<!-- put backend in network - was connecting using host.docker.internal...needs to update to name of mongo container: mongodb in app.js -->
mongoose.connect(
  'mongodb://mongodb:27017/course-goals',

# step 4 - rebuild because of change
# INSIDE Backend/ folder
<!-- then rebuild -->
docker build -t goals-node .

# step 5 (BETTER CODE LATER ON) - create backend image
docker run --name goals-backend --rm --network goals-net goals-node
#---------------------------------------------------------------------------------

# WRONG step 6 - frontend - WRONG STEP... DONT DO THIS... 
# update 'localhost' in frontend/App.js to name of backend: 'goals-backend' then rebuild
# inside the frontend folder... run..
# docker build -t goals-react .
# WRONG step 7 - add frontend to network - but frontend runs in browser, so it doesnt know what http://goals-backend is
# docker run --name goals-frontend --network goals-net --rm -d -p 3000:3000 -it goals-react
#---------------------------------------------------------------------------------

# STEP6 - CORRECT WAY - is to leave frontend/App.js trying to access 'localhost'
# inside Frontend/ folder
docker build -t goals-react .

# STEP7 (FRONTEND STEP!) - frontend does not need to be part of network (remove: --network goals-net )
# since it doesnt interactive with backend or DB
# part being executed is also not in docker environment
docker run --name goals-frontend --rm -d -p 3000:3000 -it goals-react

# STEP8  (BETTER CODE LATER ON) 
# restart backend container but publish port 80 so frontend can talk to backend
docker run --name goals-backend --rm -d -p 80:80 --network goals-net goals-node

# STEP 9 (BETTER CODE LATER ON) DATABASE REVISIT - stopping mongo db looses the data that persisted there
# save data using volume - define named volume eg. named 'data' - mongo stores data in container here: /data/db
docker run --name mongodb -v data:/data/db --rm -d --network goals-net mongo
```

# STEP 10 - SECURITY
- preventing access to DB using env variables
- you specify in the shell docker string that it should use username and password, then this ensures that the code in backend can only access the db if the username and password is specified in connection string

- MONGO_INITDB_ROOT_USERNAME 
- MONGO_INITDB_ROOT_PASSWORD

```shell
# MONGO STEP! - rebuild node image with username + password 

docker run --name mongodb -v data:/data/db --rm -d --network goals-net -e MONGO_INITDB_ROOT_USERNAME=max -e MONGO_INITDB_ROOT_PASSWORD=secret mongo

``` 
- this causes the connection to fail because it connected without the username and password which was required and set in docker run (making it protected)...
- syntax: 'mongodb://[username:password@]host.docker.internal:27017/course-goals',
- add ?authSouce=admin' to end of connection string
```
mongoose.connect(
  'mongodb://max:secret@mongodb:27017/course-goals?authSource=admin',
```

- rebuild node image
```
docker build -t goals-node .
```

-run again from Backend/ folder

```shell
# BACKEND STEP! EXPOSING PORT (WITHOUT VOLUMES)
docker run --name goals-backend --rm -d -p 80:80 --network goals-net goals-node

```
## STEP 11 - authentication errors?
if you are getting error with Authentication failed message when setting username/password
https://www.udemy.com/course/docker-kubernetes-the-practical-guide/learn/lecture/22167028#questions/13014650

The problem could be the volume and the fact that you created another user with different credentials before you changed them. Because of the volume, your database is still there and hence your old root user is still set up - i.e. your old credentials apply.

### SOLUTION
docker volume prune
or 'docker rm' one or more

----------------------------------------------

## Volumes, Bind mounts & Polishing for nodejs Container,
- to make sure data persists on backend (OPTIONS ARE: named (dont know where its stored) or bind-mount (read from hosting machine))
- live source code updates
- to bindmount to everything in /app folder to localhosting directory, you need fill path to app.js (rightclick + copy path)
- -v /app/node_modules tells docker to keep existing node_modules folder and not overwrite

```shell
<!-- named method + bindmount everything in /app folder to localhosting directory -->

docker run --name goals-backend -v C:\Users\clark\Downloads\code\tutorial-maximilianschwarzmuller\tutorial-maximilianschwarzmuller-docker\05-multicontainer-app-01-starting-setup\backend:/app -v logs:/app/logs -v /app/node_modules --rm -d -p 80:80 --network goals-net goals-node

```

## swapping out hardcoded username+password

```js
  `mongodb://${process.env.MONGODB_USERNAME}:${process.env.MONGODB_PASSWORD}@mongodb:27017/course-goals?authSource=admin`,
```

```Dockerfile
ENV MONGODB_USERNAME=root
ENV MONGODB_PASSWORD=secret
```

<!-- rebuild docker file -->
docker build -t goals-node .
```shell
<!-- note that we are setting env variable to 'max': -e MONGODB_USERNAME=max  -->
docker run --name goals-backend -v C:\Users\clark\Downloads\code\tutorial-maximilianschwarzmuller\tutorial-maximilianschwarzmuller-docker\05-multicontainer-app-01-starting-setup\backend:/app -v logs:/app/logs -v /app/node_modules -e MONGODB_USERNAME=max --rm -d -p 80:80 --network goals-net goals-node
```
------------------------------------------------------------------

## frontend - live source updates
- copy path from frontend/app.js
- binding src to docker container src/

```shell
<!-- rebuild image -->
docker build -t goals-react .
```

```shell
docker run -v C:\Users\clark\Downloads\code\tutorial-maximilianschwarzmuller\tutorial-maximilianschwarzmuller-docker\05-multicontainer-app-01-starting-setup\frontend:/app/src --name goals-frontend --rm -d -p 3000:3000 -it goals-react
```