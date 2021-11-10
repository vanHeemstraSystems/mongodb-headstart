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

Inside the mongodb directory create a ***sample environment*** file.

```
$ cd containers/app/mongodb
$ touch sample.env
```

With the sample.env file created, add the following content to it:

```
PORT=8000
```
containers/app/mongodb/sample.env

Inside the mongodb directory create a ***.gitignore*** file.

```
$ cd containers/app/mongodb
$ touch .gitignore
```

With the .gitignore file created, add the following content to it:

```
.env
scripts/init/mongoInit.js
scripts/seed/MOCK_DATA.json
```
containers/app/mongodb/.gitignore

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
$ npm i --save express dotenv nodemon
```

After above command, verify if the packages have been mentioned inside ```package.json```:

```
...
  "dependencies": {
    "dotenv": "^10.0.0",
    "express": "^4.17.1",
    "nodemon": "^2.0.14"
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

If it is mentioned that there is already a process listening at port 8000, find the Process ID (PID) as follows:

In **general**, you can try netstat
```
$ netstat -vanp tcp | grep 8000
``` 

For **macOS El Capitan and newer** (or if your netstat doesn't support -p), use lsof

```
$ lsof -i tcp:8000 
``` 
 
For **Centos 7** use:

```
$ netstat -vanp --tcp | grep 8000
```

If it returns the Process ID (PID) at which an earlier process is listening, you can kill that process as follows:

```
$ kill -9 <PID>
```

After above execution open ```localhost:8000``` in a browser. If the "Welcome to MongoDB Service" message comes up on the page it means Express.js installation was successful.

![Screenshot 2021-11-01 at 11 14 46](https://user-images.githubusercontent.com/1499433/139656576-3039dbce-aa4d-4dc9-b4aa-c6d195be407e.png)

Let's continue.

Create a subdirectory under ```mongodb``` called ```scripts```:

```
$ cd containers/app/mongodb
$ mkdir scripts
```

Create a subdirectory under ```scripts``` called ```init```:

```
$ cd containers/app/mongodb/scripts
$ mkdir init
```

Inside the ```init``` directory, create a file called ```.dbshell``` with no content inside.

```
$ cd containers/app/mongodb/scripts/init
$ touch .dbshell
```

**IMPORTANT**: Set the read & write permission for .dbshell to everyone, as follows:

```
$ cd containers/app/mongodb/scripts/init
$ chmod o+rw .dbshell
```

The ```.dbshell``` file is just a blank placeholder file that is used to create a Linux user to run the scripts in the container.

Also, inside the ```init``` directory, create a file called ```sample.mongoInit.js```.

```
$ cd containers/app/mongodb/scripts/init
$ touch sample.mongoInit.js
```

Add the following content to ```sample.mongoInit.js```:

```
// use shell command to save env variable to a temporary file, then return the contents.
// source: https://stackoverflow.com/questions/39444467/how-to-pass-environment-variable-to-mongo-script/60192758#60192758
function getEnvVariable(envVar, defaultValue) {
    var command = run("sh", "-c", `printenv --null ${ envVar } >/tmp/${ envVar }.txt`);
    // note: 'printenv --null' prevents adding line break to value
    if (command != 0) return defaultValue;
    return cat(`/tmp/${ envVar }.txt`)
  }
  
  // create application user and collection
  var dbUser = getEnvVariable('APP_USER', 'app_user');
  var dbPwd = getEnvVariable('APP_PWD', 'app_user()');
  var dbName = getEnvVariable('DB_NAME', '<database_name>');
  var dbCollectionName = getEnvVariable('DB_COLLECTION_NAME', '<db_collection_name>');
  db = db.getSiblingDB(dbName);
  db.createUser({
    'user': dbUser,
    'pwd': dbPwd,
    'roles': [
      {
        'role': 'dbOwner',
        'db': getEnvVariable('DB_NAME', '<database_name>')
      }
    ]
  });
  
  db.createCollection(dbCollectionName);
```
containers/app/mongodb/scripts/init/sample.mongoInit.js

Copy sample.mongoInit.js to mongoInit.js:

```
$ cd containers/app/mongodb/scripts/init
$ cp sample.mongoInit.js mongoInit.js
```

Now inside ```mongoInit.js``` replace the following placeholders:

- <database_name> : replace with your database name (e.g. ```parceltracking```)
- <db_collection_name> : replace with your database name (e.g. ```tracking```)

```mongoInit.js``` is where we'll create our user and database.

The code in this script runs in the Mongo shell, so you’ll notice difference Mongo-specific objects are provided: e.g., ```db```, ```run```, ```cat```.

The first function uses the mongo shell ```run``` command to retrieve environment variables (and I have to credit [this Stack Overflow hack](https://stackoverflow.com/questions/39444467/how-to-pass-environment-variable-to-mongo-script/60192758#60192758) for help. Please let me know if there's a better way to do this).

Then, we create the database with ```db.getSiblingDB```, a user with ```db.createUser```, and a collection with ```db.createCollection```.

Note that this script will only run when the container is initially started and bound to a Docker volume on the host, but not on any subsequent container startups bound to the same volume. This prevents any duplication or data collision when the container is shut down and restarted.

The next step is optional, but I want to also add some seed data so that I can test out the application right away.

Create a subdirectory under ```scripts``` called ```seed```:

```
$ cd containers/app/mongodb/scripts
$ mkdir seed
```

Inside the ```seed``` directory, create a file called ```sample.MOCK_DATA.json```.

```
$ cd containers/app/mongodb/scripts/seed
$ touch sample.MOCK_DATA.json
```

Add the following content to ```sample.MOCK_DATA.json```:

```
sample.MOCK_DATA.json
```
containers/app/mongodb/scripts/seed/sample.MOCK_DATA.json

Copy sample.MOCK_DATA.json to MOCK_DATA.json:

```
$ cd containers/app/mongodb/scripts/seed
$ cp sample.MOCK_DATA.json MOCK_DATA.json
```

Now inside ```MOCK_DATA.json``` replace the content with the mock data that suits your database.

***For example***: We’ll download some data that I’ve already generated for the project using [Mockaroo](https://www.mockaroo.com/).
$ curl https://raw.githubusercontent.com/jsheridanwells/MeanUrls/main/scripts/mongo/seed/MOCK_DATA.json -o ./scripts/mongo/seed/MOCK_DATA.json

Also, inside the ```seed``` directory, create a file called ```mongo_seed.sh```.

```
$ cd containers/app/mongodb/scripts/seed
$ touch mongo_seed.sh
```

Add the following content to ```mongo_seed.sh```:

```
#!/bin/bash
if [ -f "/MOCK_DATA.json" ]; then
  FILE="/MOCK_DATA.json"
elif [ -f "./MOCK_DATA.json" ]; then
  FILE="./MOCK_DATA.json"
else
  echo "Mock data file not found. Make sure container has a MOCK_DATA.json file for this script to work"
  exit 1
fi

mongoimport --host $MONGO_HOSTNAME \
  --authenticationDatabase $DB_NAME \
  --username $APP_USER --password $APP_PWD \
  --db $DB_NAME \
  --collection $DB_COLLECTION_NAME \
  --file $FILE --jsonArray
```
containers/app/mongodb/scripts/seed/mongo_seed.sh

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
