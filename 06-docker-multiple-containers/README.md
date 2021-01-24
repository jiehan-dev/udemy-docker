### Three Building Blocks

#### Database

- MongoDB
- Data must persist
- Access should be limited

#### Backend

- NodeJS REST API
- Data must persist
- Live Source Code Update

#### Frontend

- React SPA
- Live Source Code Update

Start with dockerizing the MongoDB Service. <br>
We can publish the port `27017` for host machine to access the db.

`docker run --name mongodb --rm -d -p 27017:27017 mongo`

Then, dockerizing the Node application. (EXECUTE IN BACKEND FOLDER)

`docker build -t goals-node .`

Publish the port for the React SPA running on host machine to connect.

`docker run --name goals-backend --rm -d -p 80:80 goals-node`

At this point, we have 2 separte container and Node App will fail becuase it can't connect to the mongodb with `mongodb://localhost:27017/course-goals`. Replace it with `mongodb://host.docker.interal:27017/course-goals` and it should works.

Follow by, dockerizing the React SPA. (EXECUTE IN FRONTEND FOLDER)

`docker build -t goals-react .`

`-it` flag is needed to keep the react app running.

`docker run --name goals-frontend --rm -d -p 3000:3000 -it goals-react`

Next, we can improve the structure by putting them under the same network.

`docker network create goals-net`

We don't need to publish the port since they can communicate within the network and we don't intend to connect it from host machine.

`docker run --name mongodb --rm -d --network goals-net mongo`

In Node app, replace ip address with the container name.

`mongodb://mongodb:27017/course-goals`

We still need to publish the port because the frontend app is accessing thru host machine browser.
`docker run --name goals-backend --rm -d -p 80:80 --network goals-net goals-node`

We dont have to replace the `localhost` address in React app, becuase the React code is running in the browser, the container is only for serving the files. Since docker can't help us to resolve the hostname in this scenario, we don't need to connect to the network.

`docker run --name goals-frontend --rm -p 3000:3000 -it goals-react`

At this point, if we stop the mmongodb container and it is removed due to the `--rm` flag, the data is gone, as the data is stored within the container. If we want to persist the data, we can use named volume. `data` is the volume name and `/data/db` is the path where the data is being stored within the container file system (refer to mongo image doc).

`docker run --name mongodb -v data:/data/db --rm -d --network goals-net mongo`

To add authentication for mongodb, we can pass it with env variable.

`docker run --name mongodb -v data:/data/db --rm -d --network goals-net -e MONGO_INITDB_ROOT_USERNAME=max -e MONGO_INITDB_ROOT_PASSWORD=secret mongo`

Add username, password and authSource to the mongodb connection string.

`mongodb://max:secret@mongodb:27017/course-goals?authSource=admin`

To persist data for backend logs, we can add use a name volume.
We can also bind mount our folder to the container app folder so that it will get updated when we modified the source code. To prevent the bind mount overwrite the node_modules folder in container, we link it with a anonymous volume. We also need to use nodemon.

`docker run --name goals-backend -v /Users/jiehan/Dev/udemy-docker/06-docker-multiple-containers/backend:/app -v logs:/app/logs -v /app/node_modules --rm -d -p 80:80 --network goals-net goals-node`

Instead of hardcoding the username and pw, we can set it with env variable.

```Dockerfile
ENV MONGODB_USERNAME=root
ENV MONGODB_PASSWORD=secret
```

Update the connection string:

`mongodb://${process.env.MONGODB_USERNAME}:${process.env.MONGODB_PASSWORD}@mongodb:27017/course-goals?authSource=admin`

`docker run --name goals-backend -v /Users/jiehan/Dev/udemy-docker/06-docker-multiple-containers/backend:/app -v logs:/app/logs -v /app/node_modules -e MONGODB_USERNAME=max --rm -d -p 80:80 --network goals-net goals-node`

Live source code update for React app using bind mount:

`docker run --name goals-frontend -v /Users/jiehan/Dev/udemy-docker/06-docker-multiple-containers/frontend/src:/app/src --rm -p 3000:3000 -it goals-react`

### Automating Multi-Container Setup With Docker Compose

Start with creating a `docker-compose.yml` file. The entension could be `yml` or `yaml`.

Specifying docker compose version at the start of the file.

Under services declare the containers with the correct indentation.

By default, when we bring the services down, the containers will be remove, so we dont have to specify it.
Detach mode can be specify when we run it.

There is multiple ways of adding environment variables.

```yml
environment:
  MONGO_INITDB_ROOT_USERNAME: max
  MONGO_INITDB_ROOT_PASSWORD: secret
```

```yml
environment:
  - MONGO_INITDB_ROOT_USERNAME=max
  - MONGO_INITDB_ROOT_PASSWORD=secret
```

Or we can create `env/mongo.env` and reference to it.

```yml
env_file:
  - ./env/mongo.env
```

We can manually create a network and assign it to the containers.

```yml
networks:
  - goals-net
```

But docker compose by default will automatically create new network and include all the services(containers).

For named volumes, we have to specify it under the same indentation with services. Anonymous volumes and bind mounts does not need to specify.

```yml
volumes:
  data:
```

To build the images and run the containers:

`docker-compose up`

add `-d` flag to run in detach mode

`docker-compose up -d`

To stop the services and remove the containers:
`docker-compose down`

add `-v` flag to also remove volumes

`docker-compose down -v`

### Setup Backend Config

To build the backend image, specify where the Dockerfile is located:

`build: ./backend`

Note: To add bind mount using `docker run`, we need to specify the absolute path, but we can use relative path to `docker-compose.yml` when configuring docker compose.

We can specify that our backend services should be setup after mongodb using the service name:

```yml
depends_on:
  - mongodb
```

Note: the name specified for each service will be map by docker that we can use them in our code to setup network connection.

### Setup Frontend Config

To start the container in interactive mode:

```yml
stdin_open: true
tty: true
```

We can specify the container name in `docker-compose.yml` file.
