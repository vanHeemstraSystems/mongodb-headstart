# 300 - Docker for Production Environment

Let's create a separate docker-compose file in production called ***sample.docker-compose.prod.yml***.

```
version: "3.7"

# See https://stackoverflow.com/questions/29261811/use-docker-compose-env-variable-in-dockerbuild-file
services:
  webui:
    build:
      context: ./webui
      dockerfile: Dockerfile.prod
      args: # from env_file
        IMAGE_REPOSITORY: ${IMAGE_REPOSITORY}
        PROXY_USER: ${PROXY_USER}
        PROXY_PASSWORD: ${PROXY_PASSWORD}
        PROXY_FQDN: ${PROXY_FQDN}
        PROXY_PORT: ${PROXY_PORT}
    env_file:
      - .env      
    container_name: webui-prod  
    ports:
      - "80:80"
```
containers/app/sample.docker-compose.prod.yml

***Notice*** the difference in port number (dev: 8080, prod: 80) and overall size of the docker-compose files (dev: long, prod: short) between ***dev*** and ***prod***.

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
# WAS: WORKDIR /app

# add `/app/node_modules/.bin` to $PATH
# WAS: ENV PATH /app/node_modules/.bin:$PATH

# install app dependencies
# WAS: COPY package.json ./
# WAS: COPY package-lock.json ./
# WAS: RUN npm ci --silent

# add app
# WAS: COPY . ./

EXPOSE 28017

# start app
# WAS: CMD ["npm", "start"]
```
containers/app/webui/Dockerfile.prod

Here, we take advantage of the [multistage build](https://docs.docker.com/engine/userguide/eng-image/multistage-build/) pattern to create a temporary image used for building the artifact – the production-ready React static files – that is then copied over to the production image. The temporary build image is discarded along with the original files and folders associated with the image. This produces a lean, production-ready image.

NOTE: Check out the [Builder pattern vs. Multi-stage builds in Docker](https://blog.alexellis.io/mutli-stage-docker-builds/) blog post for more info on multistage builds.

Using the production docker-compose file, build and tag the Docker image, and run the container specifying its name as "pwc-databases-prod" to distinguish it from possible other stacks that are called "app" (the default name, based on the root directory):

```
$ cd containers/app
$ docker-compose --file docker-compose.prod.yml --project-name mongodb-prod up --build -d
```

If successful, browse to http://localhost (***note***: 80 is the default port for HTTP, and therefore not required to be added to the hostname) to see the production version of the app.

![React App Screen Shot](react_app_screen_shot.png)

http://localhost
