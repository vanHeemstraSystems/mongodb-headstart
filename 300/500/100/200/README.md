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

Create (if now already existing) a new connection in Robo 3T, with the following settings:

![Robo 3T Connection String - Connection](robo_3t_connection_string_connection.png)

Robo 3T: Connection String - Connection

**Note**: Use the correct URL or IP address for the server your MongoDB Docker Container is hosted on, as well as the port.

Add the login credentials, as set in ```containers/app/.env``` file:

![Robo 3T Connection String - Authentication](robo_3t_connection_string_authentication.png)

Robo 3T: Connection String - Authentication

**Test** the connection by clicking the 'Test' button.

![Robo 3T Connection String - Test Successful](robo_3t_connection_string_test_successful.png)

Robo 3T: Connection String - Test Successful

Save the connection.

![Robo 3T MongoDB - Save Connection](robo_3t_mongodb_connections_save_connection.png)

Robo 3T: Connections

Once connected you will see the user (here: ```mongo_root```), which is the value of ```MONGO_INITDB_ROOT_USERNAME``` set in ```containers/app/.env``` file.

![Robo 3T MongoDB - Database Connected](robo_3t_mongodb_database_connected.png)

Robo 3T: Database connected

## Seeding the database

The next step is optional, but there are times when it is helpful to start the database with some boilerplate data to test the components more authentically, or to see how the application handles a more realistic volume of data. The ```docker exec``` command lets us enter the Docker container from the terminal, navigate the file system on the container, and execute bash commands.

With the container running, execute the following command from the server that hosts the container:

```
$ docker exec -it mongodb-dev bash
```

**Note**: ```mongodb-dev``` is the name of the running container.

This will open a bash terminal (in interactive mode, as specified by ```-it```) running inside of the container.

From inside the container, navigate to the directory where the seed scripts (e.g. ```mongo_seed.sh```) were mounted.

```
inside the container: $ cd /home/mongodb/seed
```

Run the seed script:

```
inside the container: $ bash mongo_seed.sh
```

Your terminal will prompt you with the progress of the seeding of the database.

Inspect the database (for example with Robo 3T) and you should see that the documents as configured in ```MOCK_DATA.json``` have been added to the collection.

To exit the container, type ```exit```.

Bring down the container before moving on:

```
$ docker-compose --file docker-compose.dev.yml stop
```
