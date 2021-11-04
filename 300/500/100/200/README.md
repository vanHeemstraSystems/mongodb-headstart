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
    env_file:
      - .env
    container_name: mongodb-dev      
    ports:
      - "28016:27017"
    volumes:
      - ./mongodb:/app
      - /app/node_modules
      - mongodb-dev-data:/data/db
...      
volumes:
  mongodb-dev-data:
...

```
containers/app/sample.docker-compose.dev.yml

Now it is time to build the development Docker Image, and run the container specifying its name as "mongodb-dev" to distinguish it from possible other stacks that are called "app" (the default name, based on the root directory), now including the ```mongodb``` service.

```
$ cd containers/app
$ docker-compose --file docker-compose.dev.yml up --project-name mongodb-dev --build -d
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

Bring down the container before moving on:

```
$ docker-compose --file docker-compose.dev.yml stop
```
