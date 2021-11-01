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

ENV HTTP_PROXY="http://${PROXY_USER}:${PROXY_PASSWORD}@${PROXY_FQDN}:${PROXY_PORT}"
ENV HTTPS_PROXY="http://${PROXY_USER}:${PROXY_PASSWORD}@${PROXY_FQDN}:${PROXY_PORT}"

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
EXPOSE 28017

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


