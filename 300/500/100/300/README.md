# 300 - Docker for Production Environment

Let's add our mongodb service to the separate docker-compose file in production called ***sample.docker-compose.prod.yml***.

```
version: "3.7"

# See https://stackoverflow.com/questions/29261811/use-docker-compose-env-variable-in-dockerbuild-file
services:

  ... 

  mongodb:
    build:
      context: ./mongodb
      dockerfile: Dockerfile.prod
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
        MONGO_PORT: 28017        
    env_file:
      - .env      
    container_name: mongodb-prod  
    ports:
      - "28017:27017"
    volumes:
      - ./mongodb:/app
      - ./mongodb/scripts/init/:/docker-entrypoint-initdb.d
      - ./mongodb/scripts/init:/home/mongodb # chown -R 999:999 ./mongodb/scripts/init
      - ./mongodb/scripts/seed/:/home/mongodb/seed      
      - /app/node_modules
      - mongodb-prod-data:/data/db

volumes:
  mongodb-prod-data:   
```
containers/app/sample.docker-compose.prod.yml

Change the permission of the folder ```containers/app/mongodb/scripts/init``` to match the one used in the Dockerfile (here: 999).

```
$ chown -R 999:999 containers/app/mongodb/scripts/init
```

Copy the sample.docker-compose.prod.yml:

```
$ cp sample.docker-compose.prod.yml docker-compose.prod.yml
```

Let's also create a separate Dockerfile for use in production called ***Dockerfile.prod***:

```
ARG IMAGE_REPOSITORY
# development environment: pull official base image for node development
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
WORKDIR /app

# add `/app/node_modules/.bin` to $PATH
ENV PATH /app/node_modules/.bin:$PATH

# install app dependencies
COPY package.json ./
COPY package-lock.json ./
RUN npm ci --silent

# add app
COPY . ./

EXPOSE 28017

# start app
CMD ["npm", "start"]
```
containers/app/mongodb/Dockerfile.prod

Using the production docker-compose file, build and tag the Docker image and run the Docker container.

Before running docker-compose, make sure when on a Linux RHEL server, set enforcing to permissive, like so:

```
$ getenforce
$ Enforcing
$ sudo setenforce 0
$ getenforce
$ Permissive
```

Build and tag the Docker image, and run the container specifying its name as "pwc-databases-prod" to distinguish it from possible other stacks that are called "app" (the default name, based on the root directory):

```
$ cd containers/app
$ docker-compose --file docker-compose.prod.yml --project-name mongodb-prod up --build -d
```

**Note**:   
```
-p, --project-name NAME     Specify an alternate project name
                              (default: directory name)
``` 

If successful, browse to http://localhost:8000 to see the production version of the app.

![React App Screen Shot](react_app_screen_shot.png)

http://localhost:8000
