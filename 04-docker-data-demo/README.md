## Setup

1. `docker build -t docker-data-demo .`

2. `docker run -p 3000:80 -d --name feedback-app --rm docker-data-demo`

3. Access the app via localhost:3000, save some text with a simple title (e.g. awesome).
   We can access the text file via `localhost:3000/feedback/awesome.txt`.

## Problem

The file is created and stored within the container.
The file will be gone once the container is remove.
Another container based on the same image will not have the same file as they are isolated.

## Volumes

Volumes are folders on our host machine hard drive which are mounted into containers with read and write access.
Volumes persist if a container shuts down. If a container restarts and mounts a volume, any data inside of that volume is available in the container.

Adding `VOLUME ["/app/feedback"]` in Dockerfile will use anonymouse Volumes, path. Anonymous volume only exists as long as the container exists.

To use named Volumes which will survive removal of container, we build the image as usual and execute:
`docker run -d -p 3000:80 --rm --name feedback-app -v feedback:/app/feedback feedback-node:volumes`

where

`-v {VOLUME_NAME}:{FOLDER_PATH}`

We can see all volumes by executing `docker volume ls`.

## Bind

`docker run -d -p 3000:80 --rm --name feedback-app -v feedback:/app/feedback -v "/Users/jiehan/Dev/udemy-docker/04-docker-data-demo:/app" -v /app/node_modules feedback-node:volumes`

#### Bind Mounts - Shortcuts

macOS / Linux: `-v $(pwd):/app`

## Final Instruction

`docker run -d -p 3000:80 --rm --name feedback-app -v feedback:/app/feedback -v "/Users/jiehan/Dev/udemy-docker/04-docker-data-demo:/app" -v /app/node_modules -v /app/temp feedback-node:volumes`

We added `-v /app/temp` to avoid the folder being overwritten by the project repository.

We can manually create a named volume by `docker volume create {name}`

We can also inspect the volume by `docker volume inspect {name}`

We can remove volume by `docker volume rm {name}` if no container using it

Adding `.dockerignore` to avoid copying unwanted files.
