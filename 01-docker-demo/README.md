DEMO

_We are trying to run this node project without having to install a speicifc node version on our local environment. We also no need to manually run `npm install` to download the dependencies._

1. Run the following command in the project folder to build docker image based on the docker file.

   > docker build .

   It will result in something like > `Successfully built a977baf15549`

2. We can then run the docker image using image id with the following command. As we are exposing port 3000 from the container, we need to publish it to the local port, for example port 5000. 

   > docker run -p 5000:3000 a977baf15549

   The terminal running the command would be stuck.

3. We can run the following command to see the current active containers.

   > docker ps

4. With the auto assigned name (e.g. 'condescending_darwin'), we can stop the container by the following command.
   > docker stop condescending_darwin

5. We can see the port mapping by using the following command.
    > docker port condescending_darwin
    
    It will result in `3000/tcp -> 0.0.0.0:5000`
