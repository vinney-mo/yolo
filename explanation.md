## 1. Choice of Base Image
`node:16-alpine3.16` image is used as the base image to build the container. It is based on the Alpine Linux distribution. This was choosen as it is lightweight and has a smaller footprint compared to other distributions. It includes Node.js version 16, which is higher than the version that the application requires.

## 2. Dockerfile Directives
### _Client_
The client docker file implemeted using using multi-stage builds. This is neccessary as it help achieve:
- Smaller and efficient images by separating the build dependencies from the runtime dependencies.
- Faster builds as by separating the build stage from the production stage, only rebuild the parts that have changed.
- Improved security: it ensures that the production image does not include any development tools or libraries that may introduce security vulnerabilities.
- Simpler Dockerfile: separing the stages help create a Dockerfile which is simple and easier to read and maintain.

##### Build Stage
`WORKDIR` directive sets the working directory for the container to /client.

Then, the `COPY` directive copies the package.json and package-lock.json files to the /client directory which are required to install the necessary dependencies for the application.

The `RUN` directive is used to execute the following commands; `npm install --only=production` to installs only the production dependencies and also clears the npm cache and removes any temporary files using `npm cache clean --force` and `rm -rf /tmp/*` respectively.

The `COPY` directive is the used to copy the rest of the application code to the container's working directory.

The `RUN` directive builds the application by running `npm run build` and it also removes development dependencies using `npm prune --production`.

#####   Production Stage
The `WORKDIR` directive sets the working directory for the container to /client. Then `COP` directive copies only the necessary files from the build stage to the production stage which include the build directory, public directory, src directory, and package*.json files.

The `ENV` directive sets the NODE_ENV environment variable inside the Docker image to production.

The `EXPOSE` directive the port 3000 to be exposed by the container when it is running.

The `RUN` directive is used to execute the following commands; `npm prune --production` to remove development dependencies and also clears the npm cache and removes any temporary files using `npm cache clean --force` and `rm -rf /tmp/*` respectively similar to that in build stage.

`CMD` directive specifies the default command to run when a container is started and in this case.

### _Backend_
The backend Dockerfile does not implement multi build as the build scripts are not included in the application

`WORKDIR` directive sets the working directory for the container to /backend.

Then, the `COPY` directive copies the package.json and package-lock.json files to the /backend directory which are required to install the necessary dependencies for the application.

The `RUN` directive is used to execute the following commands; `npm install --only=production` to installs only the production dependencies and also clears the npm cache and removes any temporary files using `npm cache clean --force` and `rm -rf /tmp/*` respectively.

The `COPY` directive is the used to copy the rest of the application code to the container's working directory.

The `ENV` directive sets the NODE_ENV environment variable inside the Docker image to production.

The `EXPOSE` directive the port 5000 to be exposed by the container when it is running.

The `RUN` directive is used to execute the following commands; `npm prune --production` to remove development dependencies and also clears the npm cache and removes any temporary files using `npm cache clean --force` and `rm -rf /tmp/*` respectively similar to that in build stage.

`CMD` directive specifies the default command to run when a container is started and in this case.

## 3. Docker-compose Networking
There are two networks are defined: `backend-net` and `frontend-net`.
`ipam` which in full is IP Address Management option is used to configure the IP addressing for the containers in each network.
The `driver: default` is used to set the default IPAM driver provided by Docker.
The `frontend-net` network, uses the subnet `172.51.0.0/16` and `backend-net`  network uses the subnet `172.100.0.0/16`.
The client container is given an IP of `172.51.0.101` in this `frontend-net` network.
The backend container is given an IP of `172.100.0.100` on The `backend-net`, and `172.51.0.100` on the `frontend-net` network.
The database container is given the IP address `172.100.0.101` on the backend-net network.
This makes it possible for containers to communicate with one another over the network using their unique IP addresses and helps prevent IP address conflicts.
Ports `3000` of the client container, `5000` of the backend container and `27017` the database container, are mapped to the host machine's ports `3000`, `5000` and `27017`.

## 4. Docker-compose volume
A backend_voulme as been defined. The volume is then mounted to the container's /data/db directory using the syntax backend_volume:/data/db in the database service. This will help persist data written

## 5. Git workflow used to achieve the task
Dockerfile are version-controlled using Git alongside other project files, and changes are be committed and pushed to a Git repository.