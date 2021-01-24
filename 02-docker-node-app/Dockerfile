FROM node

WORKDIR /app

# we copy package.json first so that if no changes has been made
# we wont need to rerun npm install 
COPY package.json /app

RUN npm install

COPY . /app


EXPOSE 80

CMD ["node", "server.js"]