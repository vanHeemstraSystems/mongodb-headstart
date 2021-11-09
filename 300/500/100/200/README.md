# 200 - Docker for Development Environment

Create a file called ```Dockerfile.dev``` in the ```mongodb``` directory.

```
$ cd containers/app/mongodb
$ touch Dockerfile.dev
```

Add the following content to this ```Dockerfile.dev``` file:

```
ARG IMAGE_REPOSITORY
# pull official base image
FROM ${IMAGE_REPOSITORY}/mongo:latest

# See https://stackoverflow.com/questions/29261811/use-docker-compose-env-variable-in-dockerbuild-file
ARG PROXY_USER
ARG PROXY_PASSWORD
ARG PROXY_FQDN
ARG PROXY_PORT
ARG MONGO_INITDB_ROOT_USERNAME
ARG MONGO_INITDB_ROOT_PASSWORD
ARG APP_USER
ARG APP_PWD
ARG DB_NAME
ARG DB_COLLECTION_NAME
ARG MONGO_HOSTNAME

ENV HTTP_PROXY="http://${PROXY_USER}:${PROXY_PASSWORD}@${PROXY_FQDN}:${PROXY_PORT}"
ENV HTTPS_PROXY="http://${PROXY_USER}:${PROXY_PASSWORD}@${PROXY_FQDN}:${PROXY_PORT}"
ENV MONGO_INITDB_ROOT_USERNAME=${MONGO_INITDB_ROOT_USERNAME}
ENV MONGO_INITDB_ROOT_PASSWORD=${MONGO_INITDB_ROOT_PASSWORD}
ENV APP_USER=${APP_USER}
ENV APP_PWD=${APP_PWD}
ENV DB_NAME=${DB_NAME}
ENV DB_COLLECTION_NAME=${DB_COLLECTION_NAME}
ENV MONGO_HOSTNAME=${MONGO_HOSTNAME}

# set working directory
# WAS: WORKDIR /app

# add `/app/node_modules/.bin` to $PATH
# WAS: ENV PATH /app/node_modules/.bin:$PATH

# install app dependencies
# WAS: COPY package.json ./
# WAS: COPY package-lock.json ./
# WAS: RUN npm install --silent

# add app
# WAS: COPY . ./

# expose port
EXPOSE 28016

# start app
# WAS: CMD ["npm", "start"]
```
containers/app/mongodb/Dockerfile.dev

***Note***: if you are ***not*** behind a proxy, comment out the following lines in Dockerfile.dev, like so:

```
# See https://stackoverflow.com/questions/29261811/use-docker-compose-env-variable-in-dockerbuild-file
# ARG PROXY_USER
# ARG PROXY_PASSWORD
# ARG PROXY_FQDN
# ARG PROXY_PORT

# ENV HTTP_PROXY="http://${PROXY_USER}:${PROXY_PASSWORD}@${PROXY_FQDN}:${PROXY_PORT}"
# ENV HTTPS_PROXY="http://${PROXY_USER}:${PROXY_PASSWORD}@${PROXY_FQDN}:${PROXY_PORT}"
```
containers/app/mongodb/Dockerfile.dev

Create a file called ```.dockerignore``` inside the ```mongodb``` directory.

```
$ cd containers/app/mongodb
$ touch .dockerignore 
```

Add the following content to ```.dockerignore```:

```
node_modules
.dockerignore
Dockerfile.dev
Dockerfile.prod
```
containers/app/mongodb/.dockerignore

Copy sample files:

```
$ cd containers/app/mongodb
$ cp sample.env .env
```

Now let us add the ```mongodb``` service and the respective ```mongodb-dev-data``` volume to ```sample.docker-compose.dev.yml``` by this entry:

```
...
service:
...
  mongodb:
    build:
      context: ./mongodb
      dockerfile: Dockerfile.dev
      args: # from env_file
        IMAGE_REPOSITORY: ${IMAGE_REPOSITORY}
        PROXY_USER: ${PROXY_USER}
        PROXY_PASSWORD: ${PROXY_PASSWORD}
        PROXY_FQDN: ${PROXY_FQDN}
        PROXY_PORT: ${PROXY_PORT}
        MONGO_INITDB_ROOT_USERNAME: ${MONGO_INITDB_ROOT_USERNAME}
        MONGO_INITDB_ROOT_PASSWORD: ${MONGO_INITDB_ROOT_PASSWORD}
        APP_USER: ${APP_USER}
        APP_PWD: ${APP_PWD}
        DB_NAME: ${DB_NAME}
        DB_COLLECTION_NAME: ${DB_COLLECTION_NAME}
        MONGO_HOSTNAME: ${MONGO_HOSTNAME}
        MONGO_PORT: 28016        
    env_file:
      - .env
    container_name: mongodb-dev      
    ports:
      - "28016:27017"
    volumes:
      - ./mongodb:/app
      - ./mongodb/scripts/init/:/docker-entrypoint-initdb.d
      - ./mongodb/scripts/init:/home/mongodb  # chown -R $USER ./mongodb/scripts/init
      - ./mongodb/scripts/seed/:/home/mongodb/seed
      - /app/node_modules      
      - mongodb-dev-data:/data/db
...      
volumes:
  mongodb-dev-data:
...

```
containers/app/sample.docker-compose.dev.yml

Change the permission of the folder ```containers/app/mongodb/scripts/init``` to match the one used in the Dockerfile (here: $USER). See https://github.com/docker-library/mongo/issues/323

```
$ chown -R $USER containers/app/mongodb/scripts/init
```

Here’s a walkthrough of what the yaml file is doing:

- ```version: "3.7"``` is the version of Docker Compose we're using, the latest as I write this.
- We’ve named the service ```mongodb```. This will become the host name of the MongoDB server and must match ```MONGO_HOSTNAME``` in the ```.env``` file.
- We’ll build a container from the official MongoDB image: image: ```mongo:latest```. See ```Dockerfile.dev``` & ```Dockerfile.prod```.
- In the ```volumes``` list there are a few things going on: 
> - First, note that for each item, the syntax is such that the value to the left of the colon (```:```), pertains to the host machine, while to the right belongs to the container (```./host-directory:/container-directory```). 
> - We're mounting ```./scripts/init``` to the directory on the container called ```/docker-entrypoint-initdb.d```. As mentioned earlier, this is a special directory that runs any Javascript or Bash scripts inside of it when the container is first created. 
> - We'll also mount ```init``` to ```/home/mongodb``` on the container. This creates the Linux user that can run the scripts with the Mongo shell. 
> - The ```seed``` scripts will be copied over to ```/home/mongodb/seed```. That way they're available to seed the database when the container is running if we want. 
> - The last item - ```mongodb-dev-data:/data/db``` will bind the Mongo database files as a Docker volume. This allows data to persist when the container is stopped and restarted. It lets us easily wipe out all of the Mongo data and start over when necessary.
- Moving on, in ```ports```, we're connecting our local ```28016``` port (dev) or ```28017``` port (prod) to the conventional Mongo port ```27017``` on the container.
- The ```environment``` list will set all of the environment variables from ```.env``` in the container.
- Lastly, ```volumes: mongodb-dev-data``` or ```volumes: mongodb-prod-data```creates and names the Docker volume on the host machine and must match the last item of the volumes list set when defining the ```mongodb``` service.

Before running docker-compose, make sure when on a Linux RHEL server, set enforcing to permissive, like so:

```
$ getenforce
$ Enforcing
$ sudo setenforce 0
$ getenforce
$ Permissive
```

Now it is time to build the development Docker Image, and run the container specifying its name as "mongodb-dev" to distinguish it from possible other stacks that are called "app" (the default name, based on the root directory), now including the ```mongodb``` service.

```
$ cd containers/app
$ docker-compose --file docker-compose.dev.yml --project-name mongodb-dev up --build -d
```

**Note**:   
```
-p, --project-name NAME     Specify an alternate project name
                              (default: directory name)
```

Fingers crossed ... !

If successful, you can browse to the start page of the new React App, which will look like below:

SCREENSHOT HERE

http://localhost:8080

Now check if we can also see the ```mongodb``` server at http://localhost:8000

SCREENSHOT HERE

http://localhost:8000

## Testing with Robo 3T

If not already on your workstation, download and install the open-source MongoDB GUI called **Robo 3T** from https://robomongo.org/



== MORE HERE ==







Bring down the container before moving on:

```
$ docker-compose --file docker-compose.dev.yml stop
```
