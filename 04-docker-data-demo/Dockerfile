FROM node:14

# We can set argument which cant be used in code, only in Dockerfile
# Cannot be used by CMD
ARG DEFAULT_PORT=80

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

# We can set env variable here, 
# but will be override if we set again during run
#ENV PORT 80

# Default to argument if no env var pass in during run
ENV PORT $DEFAULT_PORT

#EXPOSE 80
EXPOSE $PORT

CMD [ "npm", "start" ]