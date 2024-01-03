# First steps:

## Step 1: Our application

* [Download zip file](http://localhost/assets/app.zip)
* create a Dockerfile in /app, put:  
`FROM node:18-alpine`  
`WORKDIR /app`  
`COPY . .`  
`RUN yarn install --production`  
`CMD ["node", "src/index.js"]`  
* Open cmd in /app, build container image: `docker build -t getting-started .`
* Run image: `docker run -dp 3000:3000 getting-started`
* Access the created app: `http://localhost:3000/`

## Step 2: Updating our app

* Make any change in the app source files (e.g src/static/js/app.js)
* Rebuild image: `docker build -t getting-started .`
* Run a new container: `docker run -dp 3000:3000 getting-started` -> will trigger an error because the previous container is using port 3000 already
* Get docker id of the current running containers: `docker ps`
* To stop a container: `docker stop <container-id>`, to remove a container: `docker rm <container-id>`
* To do both: `docker rm -f <container-id>`
* Try to run again the updated image with `docker run -dp 3000:3000 getting-started` and refresh `http://localhost:3000`, now it should work

## Step 3: Sharing our app

* Create a repo on [Docker Hub](https://hub.docker.com/)
* Name it getting-started, make it public
* Login to Docker Hub from terminal: `docker login -u YOUR-USER-NAME`
* To see local docker image from your machine: `docker image ls`
* You need to tag your image with the repo name to be able to push it: `docker tag getting-started YOUR-USER-NAME/getting-started`
* Push: `docker push YOUR-USER-NAME/getting-started`
* You can try the repository on docker playground [here](https://labs.play-with-docker.com/), add new instance and type: `docker run -dp 3000:3000 YOUR-USER-NAME/getting-started`, open port 3000

## Step 4: Persisting our DB 

Demo of unpersisting memory of containers images:
* Create a ubuntu container containing a file with a rondom number between 1 to 10K: `docker run -d ubuntu bash -c "shuf -i 1-10000 -n 1 -o /data.txt && tail -f /dev/null"`
* Cat the file: `docker exec <container-id> cat /data.txt`
* Create a new container from the same image, adding the ls command to observe if the file is there: `docker run -it ubuntu ls /`
* Remove persistante container: `docker rm -f <container-id>`

To create a persistance data with a container:
* `docker volume create YOUR-VOLUME`
* Run an image and mount the volume: `docker run -dp 3000:3000 -v YOUR-VOLUME:/etc/todos getting-started`
* Make a change using the app interface at `http://localhost:3000`
* Remove the container and rerun a new one, the change are still visible

 To inspect volume: `docker volume inspect YOUR-VOLUME`

## Step 5: Using bind mounts 

* After closing previous containers linked to port 3000, from /app type:
`docker run -dp 3000:3000 -w /app -v "$(pwd):/app" node:18-alpine sh -c "yarn install && yarn run dev"`
* %cd% instead of $(pwd) for windows (means current directory from your machine)
* Observe logs: `docker logs -f <container-id>`, of ends by `Listening on port 3000`, it is ready for change
* The app will update automatically from every code change
* Once all modifications are done, you can rebuild: `docker build -t getting-started .`
* To repush you need to tag again and push

## Step 6: Multi-container apps

* To create a containers network: `docker network create todo-app`
* To run a MySQL container in the todo-app network:
`docker run -d --network todo-app --network-alias mysql -v todo-mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=secret -e MYSQL_DATABASE=todos mysql:8.0`
* To access the shell of the container: `docker exec -it <mysql-container-id> mysql -p`
* To osbserve the databases: `show databases;`
* Exit: `exit`
* To access ip and containers network infos use nicolaka/netshoot container in your network: `docker run -it --network todo-app nicolaka/netshoot`
* To get MySQL container ip: `dig mysql`
* Exit: `exit`
* To run our app container connected to MySQL container:
`docker run -dp 3000:3000 -w /app -v "$(pwd):/app" --network todo-app -e MYSQL_HOST=mysql -e MYSQL_USER=root -e MYSQL_PASSWORD=secret -e MYSQL_DB=todos node:18-alpine sh -c "yarn install && yarn run dev"`
* Check state with: `docker logs <container-id>`, you should get:
`[nodemon] starting `node src/index.js`
Connected to mysql db at host mysql
Listening on port 3000`
* Add a task through the app interface at `http://localhost:3000`
* Check MySQL database: `docker exec -it <mysql-container-id> mysql -p todos`, you should see your tasks saved.
