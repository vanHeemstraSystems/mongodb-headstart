# 100 - Service Setup

## 100 - Install Create React App globally

Not Applicable

## 200 - Setup the directory structure

Create a subdirectory for the MongoDB service.

```
$ cd containers/app
$ mkdir mongodb
```

Inside the newly created subdirectory ```amqp``` run the following command to initialise npm:

```
$ cd containers/app/mongodb
$ npm init -y
```

As a result of above initialization, the following files will have been created in the ```mongodb``` subdirectory:

- package.json

MORE TO FOLOW ??

Create a file ```app.ts``` inside the ```mongodb``` directory.

```
$ cd containers/app/mongodb
$ touch app.ts
```

Add the following content to ```app.ts```:

```
import * as express from 'express';
import * as bodyParser from 'body-parser';
import { Express } from 'express';
import { routerIndex } from './routes/index';

export default function createApp(): Express {
    const app = express();
    app.use(bodyParser.json());
    app.use(bodyParser.urlencoded({ extended: false }));
    app.use('/api', routerIndex());
    app.use((req, res) => res.status(404).send('not found :('));
    return app;
}
```
containers/app/mongodb/app.ts

== WE ARE HERE == 

## 300 - Generate a new app

Inside the parent ```app``` directory update the previously created ***sample environment*** file.

```
$ cd containers/app/mongodb
$ vim sample.env
```

With the ```sample.env``` file in edit mode, add the following content to it:

```
...
MONGO_INITDB_ROOT_USERNAME=mongo_root
MONGO_INITDB_ROOT_PASSWORD=mongo_root()
APP_USER=app_user
APP_PWD=app_user()
DB_NAME=<database_name>
DB_COLLECTION_NAME=<db_collection_name>
MONGO_HOSTNAME=mongodb
MONGO_PORT=28017
...
```
containers/app/sample.env

These are the values that Docker will use to configure the server running in a Docker container. This example uses the following:

- The root login and password for the database server.
- An app user and password. These are the credentials the application itself will use, This helps to limit the application’s access to just what it needs.
- The name of the database and the name of a collection to start with.
- The value ```MONGO_HOSTNAME``` is the name to access the database server in the Docker container from our host machine. This host name must match the name we give the container service that we'll set up later.
- ```MONGO_PORT``` is the port that our application will use to access the database. MongoDB runs on port ```27017``` by convention; I like to change it to ```28017``` so I can tell it apart form any local instance of Mongo running on my machine, but that's just a personal preference.

Copy the ```sample.env``` file:

```
$ cd containers/app
$ cp sample.env .env
```

Replace ```<database_name>``` and ```<db_collection_name>``` in ```.env``` file by the values you intend for the application(s), hence here we assume the following values:

```
DB_NAME=parceltracking
DB_COLLECTION_NAME=tracking
```




== WE ARE HERE ==
