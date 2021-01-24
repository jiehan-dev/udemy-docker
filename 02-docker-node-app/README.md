_If we want to run this project locally, we need to run `npm install` to install all the dependencies required. Then execute `node server.js`._

1. First, to setup the docker image, we need to create `Dockerfile` in the directory.

> FROM node

`^ this line is telling docker to start with 'node' image`

> COPY . /app

`^ the first path (.) will tell docker where is the location of files that should go into the image, so for this is case, it is the current folder. The second path (/app) is telling docker where the file should be stored inside the image. (Basically host file sytem -> image internal file system)`

> RUN npm install

`^ This command will actually execute in the root folder by default. Since we copied the files to app subfolder, we added the following line to tell docker that it should execute subsequent commands in the app folder.`

> WORKDIR /app

`Since we define the new working directory, the 'COPY' command can also be updated as follow.`

Relative Path

> COPY . ./

OR

Absolute Path

> COPY . /app

_Whatever setup in `Dockerfile` is executed when the image is being built.
Hence we cant use the `RUN` command, instead we use `CMD` command to tell docker to run `node server.js` everytime the container is run._

> CMD ["node", "server.js"]

_NOTE: If we don't specify a `CMD`, the `CMD` of the base image will be executed. With no base image and no `CMD`, we will get an error. `CMD` should always be the last command._


> EXPOSE 80

_Before we end the `Dockerfile` with the `CMD` command, we should `EXPOSE` the port that container is listening to the local system. This is more for documentation, it will still works as long as we publish the port upon running even without `EXPOSE`._

_Run the following command to build the image using the Dockerfile in current directory._
> docker build .

It will produce result like `'Successfully built f344968fad6f'`

Then we can run a container with the image id
> docker run -p 3000:80 f344968fad6f

_Images are READ-ONLY. Any changes would need to be built again._

_If no changes has been made for a particular instructions (layer), docker will use the previous result from cache (a.k.a Layer Based Architecture). If one layer changed, any layer after it will be rerun. Hence to optimise the flow, we can copy the package.json first, run npm install, then copy the whole app folder._