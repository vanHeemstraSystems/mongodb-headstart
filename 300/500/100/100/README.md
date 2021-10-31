# 100 - Service Setup

## 100 - Install Create React App globally

```
$ npm install -g create-react-app@3.4.1
```

## 200 - Setup the directory structure

Not Applicable

## 300 - Generate a new app

Inside the subdirectory /containers/app/ created a new app called '***mongodb***':

```
$ cd containers/app
$ npm init react-app mongodb --use-npm
$ cd mongodb
```

You will be prompted as follows:

```
found 27 vulnerabilities (8 moderate, 18 high, 1 critical)
  run `npm audit fix` to fix them, or `npm audit` for details

Success! Created webui at /Users/willemvanheemstra/git/mongodb-headstart/containers/app/mongodb
Inside that directory, you can run several commands:

  npm start
    Starts the development server.

  npm run build
    Bundles the app into static files for production.

  npm test
    Starts the test runner.

  npm run eject
    Removes this tool and copies build dependencies, configuration files
    and scripts into the app directory. If you do this, you can’t go back!

We suggest that you begin by typing:

  cd webui
  npm start

Happy hacking!
```

Take care of the reported vulnerabilities by running the following command from within the webui directory.

```
$ cd mongodb
$ npm audit fix
```

Add a .gitignore file to the ```mongodb``` folder.

```
$ cd containers/app/mongodb
$ touch .gitignore
```

Add the following to the .gitignore file.

```
# See https://help.github.com/articles/ignoring-files/ for more about ignoring files.

# dependencies
/node_modules
/.pnp
.pnp.js

# testing
/coverage

# production
/build

# misc
.DS_Store
.env       <<<<<< ========================== ADD THIS !!
.env.local
.env.development.local
.env.test.local
.env.production.local

npm-debug.log*
yarn-debug.log*
yarn-error.log*
```
containers/app/mongodb/.gitignore

Inside the mongodb directory create a ***sample environment*** file.

```
$ cd containers/app/mongodb
$ touch sample.env
```

With the ```sample.env``` file created, add the following content to it:

```
MONGO_INITDB_ROOT_USERNAME=mongo_root
MONGO_INITDB_ROOT_PASSWORD=mongo_root()
APP_USER=app_user
APP_PWD=app_user()
DB_NAME=<database_name>
DB_COLLECTION_NAME=<db_collection_name>
MONGO_HOSTNAME=mongodb
MONGO_PORT=28017
```
containers/app/mongodb/sample.env

These are the values that Docker will use to configure the server running in a Docker container. This example uses the following:

- The root login and password for the database server.
- An app user and password. These are the credentials the application itself will use, This helps to limit the application’s access to just what it needs.
- The name of the database and the name of a collection to start with.
- The value ```MONGO_HOSTNAME``` is the name to access the database server in the Docker container from our host machine. This host name must match the name we give the container service that we'll set up later.
- ```MONGO_PORT``` is the port that our application will use to access the database. MongoDB runs on port ```27017``` by convention; I like to change it to ```28017``` so I can tell it apart form any local instance of Mongo running on my machine, but that's just a personal preference.

Copy the ```sample.env``` file:

```
$ cd containers/app/mongodb
$ cp sample.env .env
```

Replace ```<database_name>``` and ```<db_collection_name>``` in ```.env``` file by the values you intend for the application(s), hence here we assume the following values:

```
DB_NAME=parceltracking
DB_COLLECTION_NAME=tracking
```




== WE ARE HERE ==
