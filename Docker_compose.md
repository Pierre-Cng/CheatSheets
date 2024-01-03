# Docker compose:

## Purpose:

Docker compose allows you to have a single script that performs all operations to create your network and your multi-container environment in a single command.

## Docker compose file:

* Create a docker-compose.yml next to the Dockerfile
* Your file should look like this:
```yml
services:
  app:
    image: node:18-alpine
    command: sh -c "yarn install && yarn run dev"
    ports:
      - 3000:3000
    working_dir: /app
    volumes:
      - ./:/app
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos

  mysql:
    image: mysql:8.0
    volumes:
      - todo-mysql-data:/var/lib/mysql
    environment: 
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: todos

volumes:
  todo-mysql-data:
```
* services -> list of containers by their name
* &nbsp; app -> name of the container
* &emsp; image -> image used to run the container
* &emsp; command -> command to run in the container
* &emsp; ports -> port mapping
* &emsp; working_dir -> working directory of the container
* &emsp; volumes -> volume mapping
* &emsp; environment -> definition of environment variables
* volumes -> define volume

## Run docker compose:

* To run docker compose file: `docker compose up -d` (-d to run in background)
* To get logs: `docker compose logs -f`
* To get specific container logs: `docker compose logs -f <container-name>`

## Stop docker compose:

* To stop the containers: `docker compose down`, to stop the volumes add `--volumes` flag.
  
