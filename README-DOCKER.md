# 01 Getting Started

## docker installation

### install docker desktop

#### below install is if you have windows enterprise or home
```
hyperV - `Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All`
container - `Enable-WindowsOptionalFeature -Online -FeatureName containers –All`
powershell as administrator: `wsl.exe --update`
```

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

- each command in docker file is like a layer of code execution and depending on how granular the code is, you can streamline the build process by helping docker only running a layer of code when a change occurs. eg.

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

### BACKEND
#### step 3 - update connect to mongodb
<!-- put backend in network - was connecting using host.docker.internal...needs to update to name of mongo container: mongodb in app.js -->
mongoose.connect(
  'mongodb://mongodb:27017/course-goals',

### step 4 - rebuild because of change
#### INSIDE Backend/ folder
<!-- then rebuild -->
docker build -t goals-node .

### step 5 (BETTER CODE LATER ON) - create backend image
docker run --name goals-backend --rm --network goals-net goals-node
#---------------------------------------------------------------------------------

 <!-- WRONG step 6 - frontend - WRONG STEP... DONT DO THIS...  
 update 'localhost' in frontend/App.js to name of backend: 'goals-backend' then rebuild
 inside the frontend folder... run..
 docker build -t goals-react .
 WRONG step 7 - add frontend to network - but frontend runs in browser, so it doesnt know what http://goals-backend is
 docker run --name goals-frontend --network goals-net --rm -d -p 3000:3000 -it goals-react -->
#---------------------------------------------------------------------------------

### STEP6 - CORRECT WAY - is to leave frontend/App.js trying to access 'localhost'
#### inside Frontend/ folder
docker build -t goals-react .

### STEP7 (FRONTEND STEP!) - frontend does not need to be part of network (remove: --network goals-net )
#### since it doesnt interactive with backend or DB
#### part being executed is also not in docker environment
docker run --name goals-frontend --rm -d -p 3000:3000 -it goals-react

### STEP8  (BETTER CODE LATER ON) 
#### restart backend container but publish port 80 so frontend can talk to backend
docker run --name goals-backend --rm -d -p 80:80 --network goals-net goals-node

### STEP 9 (BETTER CODE LATER ON) DATABASE REVISIT - stopping mongo db looses the data that persisted there
#### save data using volume - define named volume eg. named 'data' - mongo stores data in container here: /data/db
docker run --name mongodb -v data:/data/db --rm -d --network goals-net mongo


### STEP 10 - SECURITY
```
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

---------------------------------------------------

# 6 Docker compose

- create a docker-compose.yaml file
- version is the version of docker compose you specify (which limits features you can use)
- https://docs.docker.com/compose/compose-file/
- indentaion and letter case matters in yaml
- services: lists containers and futher indentations list configuration for each container
- specify environment variables like: 
    MONGO_INITDB_ROOT_USERNAME:max OR - MONGO_INITDB_ROOT_USERNAME=max
- you can use a folder by specifying env_file: instead (if you dont want to commit keys to repo)
- network is created automatically by composer for all services in the group, you can specify a network:eg goals-net to have a name
and it will be an addition to the default network
- different containers can share the same volume (with same name)
- you can point docker to a folder to look for Dockerfile eg. with backend
- the longer form of build you specify: context: ./backend AND dockerfile: Dockerfile
- even if you let docker auto asign container names, you can still use the names you define in the docker file to reference eg. mongo
- for interactive mode - stdin_open: true, tty: true
- you can force images to be rebuilt with docker-compose up --build
- define your own container names with: container_name:
```yaml
version: '3.8'
services: 
  mongodb:
    image: 'mongo'
    volumes: 
      - data:/data/db
    # environment:
    #   MONGO_INITDB_ROOT_USERNAME:max
    #   MONGO_INITDB_ROOT_PASSWORD:secret 
    container_name: mongodb
    env_file:
      - ./env/mongo.env
  backend:
    build: ./backend
    # build: 
      # context: ./backend
      # dockerfile: Dockerfile
    ports: 
      - '80:80'
    volumes:
      - logs:/app/logs
      - ./backend:/app
      - /app/node_modules
    env_file:
      - ./env/backend.env
    depends_on:
      - mongodb
  frontend:
    build: ./frontend
    ports:
    - '3000:3000'
    volumes:
      - ./frontend/src:/app/src
    stdin_open: true
    tty: true
    depends_on:
      - backend
volumes:
  data:
  logs:

```

### starting a compose file
```
docker-compose up  //starting
docker-compose down // stopping

docker-compose up -d
docker-compose up --build
docker-compose build
```
---------------------------------------------------------------------

# Utility Containers
## Assist in running commands in already running containers

- starting in interactive AND detached mode
- docker exec allows running of commands in already running containers (in addition to start up Docker commands)
- you need the name of the running container...
- then you can execute commands in the container (needs -it interactive mode)

## restricting commands that you can run
with Dockerfile: 
ENTRYPOINT ['npm']

anything added to command after image name: eg node-util, is appended after ENTRYPOINT
- docker run -it -v </absolute path for bind mount>:</app=folder inside container> <name of container> init
-mirrors actions on container back to local folder, allows executing commands on container that mirror back to local
- docker run - container is not automatically removed after use.. add --rm

```shell
# start container with node
docker run -it -d node

# run a command inside container (which has node)
docker exec -it <name of container> npm init

# eg.
docker run -it -v /Users/mamillianschwarzmuller/development/teaching/udemy/docker-complete:/app mynpm ...
```

```yaml
# docker-compose.yaml

version:"3.8"
services:
  npm:
    build: ./
    stdin_open: true
    tty: true
    volumes:
      - ./:/app
```
- 'docker-compose run' allows us to run a single 'services' from yaml file (by name)

```
docker-compose run --rm npm init
```

## using docker to setup a laravel & php project
- i blanked out when he started talking about php - this chapter is a write off


# 09 Deploying Docker Containers
- web app deployment

### Development
- containers should encapsulate the runtime environment but not neccesarily the code
- use bind mounts - to provide your local host project files to the running container
- allows for instant updates without starting the container

### Production
- container is/should be standalone - image is single source of truth - should not have source code on host machine
- COPY - use COPY to copy code to make a snapshot into the image.

### AWS create EC2 instance
default username is: ec2-user
get instance details: Public IPv4 DNS

## convert your access key to ppk format 
- using puttyGen
- save as .ppk 

## connect to SSH via windows Putty
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/putty.html
- follow steps above..

- username@instance_public_IPV4_dns
- save session

- connects to EC2 instance
- so now we need to install docker 
- and get our image transfered from local to remote machine 
  1. USING DOCKERHUB - build locally - deploy image on remote - then just exec docker run on remote OR 
  2. deploy source code and build on remote (WHY!!?? no benefits)
- run a container 
- configure security groups

### install docker on virtual machine
- first update linux
```
sudo yum update -y
```

- install docker
```
sudo amazon-linux-extras install docker
```

<!-- start docker -->
```
sudo service docker start
```

## create image for Dockerhub
https://hub.docker.com/

1. create a repository on dockerhub (docker hub gives push to repo command: 'docker push swagfinger/node-example-1:tagname')
2. use command to push to repo: docker push swagfinger/node-example-1:tagname
3. create .dockerignore
4. build a local image (view built images: docker images)
5. renaming built image by re-tag: docker tag node-dep-example-1 swagfinger/node-example-1
6. login to docker via commandline: docker login
7. push to dockerhub: docker push swagfinger/node-example-1

```shell
docker build . -t tagname

# eg. create image
docker build . -t node-dep-example-1

# view built images
docker images

#rename built image USING dockerhub given repo
docker tag node-dep-example-1 swagfinger/node-example-1

# view built images
docker images

# login to docker via commandline
docker login

# docker push new tag image to dockerhub
docker push swagfinger/node-example-1
```

### running + publishing on EC2
1. download our published image from dockerhub and start a container
2. sudo docker ps (view image from remote)
3. to test: get IPv4 public IP from AWS

```shell
# publish container
sudo docker run -d --rm -p 80:80 swagfinger/node-example-1

# check container is on remote
sudo docker ps
```

### testing it on remote server (NO NEED FOR INSTALLING NODEJS ON SERVER - only DOCKER)
#### full control (self-managed method):
- ipv4 public ipaddress from AWS
- note the security group associated with EC2 (aws)
- by default public address doesnt allow any access, only SSH
- need a security group
- DEFAULT: outbound ALL traffic
- DEFAULT: inbound ssh 
- change inbound to allow HTTP 80 : Any
- retry IPV4 address: YAY this works!


## pushing updates to remote
- rebuild image - docker build -t node-dep-example-1 .
- push update to docker-hub - docker push swagfinger/node-example-1

```shell
# build
docker build -t node-dep-example-1 .
docker build --no-cache -t node-dep-example-1 .
# push
docker push swagfinger/node-example-1

```

- back on ec2instance cmd: stop the running container
```shell
sudo docker stop stoic_yalow
```

### docker pull updates
- DONT FORGET 'SUDO'
- rerun command on server
- first docker pull (to fetch new updates)

```shell
# from ec2 cmd
sudo docker pull swagfinger/node-example-1
sudo docker run -d --rm -p 80:80 swagfinger/node-example-1
```
#### managed AWS-ECS
- assists in managing containers
- deploying containers is not via docker anymore, its via ECS tools
  - container definition (CONTAINER CONFIG)
  - task definition (SERVER CONFIG eg. Fargate (on demand EC2) serverless launch mode) - gives public ip
  - service (eg ALB)
  - cluster (network, vpc)
- ECS costs money (e.g. load balancers, NAT gateways etc.)
 
#### updating ECS containers
- clusters -> default -> tasks -> task definition of running task -> create new revision -> create
- on confirmation page -> actions -> update service

## 142. Multi container app by adding Frontend
- wont use docker compose for deployment on remote (cloud)
- ecs cant use container name references as per Dockerfile
- HOWEVER, if they fall under same 'task', you can use 'localhost' on ECS instead of docker network name as they will be on same machine
- make use of env variables to allow different values for development vs production
- create repository on dockerhub (public)
- retag image and push again to dockerhub
- using environment variables under ECS settings instead of reading from .env file or Dockerfile
- create ELB - internet facing / port 80 / connect to VPC

## 145. load balancer 
- use load balancer dns name - we can use this url as stable domain
- the domain points at the deployed containers

## 146. EFS volumes with ECS
- rebuilding image and pushing image to docker-hub causes database is lost when task is shutdown
- ECS TALKING TO EFS: solution is to use a volume -add a volume *EFS (elastic file system) - create file system -> use same vpc -> network access ->  then ec2 security group ->create new security group, add it to vpc -> add inbound rule (NFS) and inbound source choose same security group used to manage containers (port 2049 used by EFS)-> create
- back at EFS select the newly created EFS Volume -> add
-  go to mongodb  - connect to container -> storage and mount points -> volume: data -> container path: /data/db (path mongodb writes to) -> update
- force redeployment (1.4+ supports EFS volumes)

## 149. moving away from mongodb to MongoDB atlas
- mongodb in the cloud
- create an account
- create a cluster (shared) and you can have multiple db's in a cluster
- connect an application
- get url connection string
- username+password can be substituted via env
- best to use mongodb atlas for both dev and prod
- or have a container with mongodb database during dev and atlas for production
- but versions may be different. same environment for both is best.
- removes need for port.
- set db for dev and another for prod

DOCKER COMPOSE
- remove mongodb service
- remove volume
- remove env_file
- remove depends_on

backend.env 
- no longer use mongodb container name, use url in connection string
- update MONGODB_USERNAME + MONGODB_PASSWORD with details from mongodb Atlas

mongodb Atlas
- network access - need to whitelist IP or from anywhere: 0.0.0.0/0
- db access: make user+password give read/write access

## MONGODB for production
- ECS remove mongodb container - run node backend container which sets mongodb connection string/credentials/db name in cloud
- remove volume / efs file system
- remove EC2 security group for volume
- update environment variables for ECS: MONGODB_URL, MONGODB_NAME, MONGODB_USERNAME, MONGODB_PASSWORD
- rebuild image and push to dockerhub
- login to docker, docker push <docker image>

## FRONTEND 

- development code is different from production
- frontend runs in browser / production transforms code (without running a server)
- prod doesnt expose port , only uses node for build
- prod: ["npm", "run", "build"]
- Multi-Stage docker builds
  - one docker file, multiple build /setup steps / stages can build from each other
  - use --target <FROM name> to target stage of Dockerfile
- switch to webserver nginx - serving from default folder: /user/share/nginx/html
- copy files from first stage to second stage
- DOCKER should start webserver

```Dockerfile
# Dockerfile.prod
FROM node:14-alpine as build
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
RUN npm run build

# further stages (multistage build)
# switching base image...

FROM nginx:stable-alpine
COPY --from=build /app/build /user/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

## 155. deploying multistage build file
- frontend App.js request urls become relative ie. remove http://localhost
-dockerhub - create a new frontend repository
- create production image locally:
- push image to dockerhub

```shell
docker build -f frontend/Dockerfile.prod -t <dockerhub repo name> ./frontend
```

- AWS ECS - point to frontend production image on dockerhub
- startup dependency ordering - specify which other containers should start up first (optional)
- only frontend or backend can map to port 80, cant have 2 containers on same ECS host - ie need to spin up aditional TASK
  - add the frontend container to the task, point to dockerhub image of frontend, map port 80 (new task/ new url)
  - new url means need to adjust frontend App.js relative paths to add URL
  - inject env variables (not Docker env variables) ie. use process.env.NODE_ENV === 'development'
- add load balancer - get its url

```js
const backendUrl = process.env.NODE_ENV === "development" ? "http://localhost" : <loadbalancer url from EC2>

// add backendUrl to relative path urls in code.
```
- rebuild frontend image 
- push again to Dockerhub
- ECS - create service based on Task


# 12 Kubernetes
[https://github.com/swagfinger/tutorial-maximilianschwarzmuller-docker/blob/main/README-KUBERNETES.md](https://github.com/swagfinger/tutorial-maximilianschwarzmuller-docker/blob/main/README-KUBERNETES.md)
