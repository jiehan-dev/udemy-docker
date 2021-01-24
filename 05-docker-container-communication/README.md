## Cross Container Communication

1. Build the image: `docker build -t favorites-node .`

2. Execute: `docker run --name favorites -d --rm -p 3000:3000 favorites-node`

_Communication between container and WWW simply works out of the box._

_Communication between container and host machine_
If we have mongoDb installed in our host machine and wanted the container to access it.
Replace `mongodb://localhost:27017/swfavorites` with `mongodb://host.docker.internal:27017/swfavorites`
Docker understand `host.docker.internal` and will map to local machine IP address.

_Communication between containers_

1. Create mongodb container: `docker run -d --name mongodb mongo`

2. We can get the ip address of mongodb: `docker container inspect mongodb`

3. Replace `mongodb://host.docker.internal:27017/swfavorites` with `mongodb://172.17.0.2:27017/swfavorites`

_Creating Container Network_

Alternatively, we can setup a container network to allow communication between containers.
Within a Docker network, all containers can communicate with each other and IPs are automatically resolved.

`docker network create favorites-net`

`docker run -d --name mongodb --network favorites-net mongo`

^ we dont need to specifically publish the port, since the communication is within docker network

Replace the hostname with the container name.

Replace `mongodb://172.17.0.2:27017/swfavorites` `with mongodb://mongodb:27017/swfavorites`

`docker run --name favorites --network favorites-net -d --rm -p 3000:3000 favorites-node`
