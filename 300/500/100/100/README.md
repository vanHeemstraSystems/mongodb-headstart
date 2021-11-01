# 100 - Service Setup

## 100 - Install Create React App globally

Not Applicable

## 200 - Setup the directory structure

Create a subdirectory for the MongoDB service.

```
$ cd containers/app
$ mkdir mongodb
```

Inside the newly created subdirectory ```mongodb``` run the following command to initialise npm:

```
$ cd containers/app/mongodb
$ npm init -y
```

As a result of above initialization, the following files will have been created in the ```mongodb``` subdirectory:

- package.json

Add a script entry inside ```package.json``` as follows:

```
...
  "scripts": {
    "start": "nodemon ./server.js --exec babel-node -e js"
  }
...
```
containers/app/mongodb/package.json

Create a file called ```server.js``` in the ```mongodb``` directory:

```
$ cd containers/app/mongodb
$ touch server.js
```

Add the following content to ```server.js```:

```
import express from "express"              
const app = express()              
app.use("/", (req, res) => {                
  res.send("Welcome to MongoDB Service")              
})      
app.listen(8000, () => console.log(`Server listening on 8000`))
```
containers/app/mongodb/server.js

Add npm packages by the following command:

```
$ cd containers/app/mongodb
$ # WAS npm i --save express dotenv tortoise mongoose socket.io nodemon
$ npm i --save express dotenv nodemon
```

After above command, verify if the packages have been mentioned inside ```package.json```:

```
...
  "dependencies": {
    "dotenv": "^10.0.0",
    "express": "^4.17.1",
    # WAS "mongoose": "^6.0.11",
    "nodemon": "^2.0.14",    
    # WAS "socket.io": "^4.3.1",
    # WAS "tortoise": "^1.0.1"
  }
...
```
containers/app/mongodb/package.json

Add npm packages used for development only by the following command:

```
$ cd containers/app/mongodb
$ npm i --save-dev @babel/core @babel/preset-env @babel/node @babel/cli babel-loader
```

After above command, verify if the packages have been mentioned inside ```package.json```:

```
...
  "devDependencies": {
    "@babel/cli": "^7.15.7",  
    "@babel/core": "^7.15.8",
    "@babel/node": "^7.15.8",    
    "@babel/preset-env": "^7.15.8",
    "babel-loader": "^8.2.3"
  }
...
```

Create a file called ```.babelrc``` in the ```mongodb``` directory.

```
$ cd containers/app/mongodb
$ touch .babelrc
```

Add the following content to this ```.babelrc``` file:

```
{
  "presets": ["@babel/preset-env"]
}
```
containers/app/mongodb/.babelrc

Test which version of babel you are running with commmand:

```
$ babel -V
```

If babel is not recognized or it is not verion 7 or higher

```
$ npm uninstall babel-cli -g
$ npm uninstall babel-core -g
```

And

```
$ npm install @babel/cli -g
$ npm install @babel/core -g
```

As an early trial, try to run the server.js file as to start a server with this command from within the mongodb directory:

```
$ cd containers/app/mongodb
$ nodemon ./server --exec babel-node -e js
```

**Note**: If you see ```command not found: nodemon``` you have to install nodeman first, globally, as follows:

```
$ sudo npm install nodemon -g
```

**Note**: If you see command not found: babel-node you have to install babel-node first, globally, as follows:

```
$ npm install @babel/node -g
```

After above execution open ```localhost:8000``` in a browser. If the "Welcome to MongoDB Service" message comes up on the page it means Express.js installation was successful.

SCREENSHOT GOES HERE

Let's continue.




====== DISREGARD BELOW FOR NOW ==========




MORE TO FOLLOW ??

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
