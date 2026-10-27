# Docker Cheat Sheet & Practical Guide

A practical reference guide for learning Docker and managing images, containers, volumes, networks, Docker Compose, and container restart policies.

## Table of Contents

1. [Docker Basics](#1-docker-basics)
2. [Docker Images](#2-docker-images)
3. [Docker Containers](#3-docker-containers)
4. [Docker Ports](#4-docker-ports)
5. [Docker Volumes](#5-docker-volumes)
6. [Bind Mounts](#6-bind-mounts)
7. [Docker Networks](#7-docker-networks)
8. [Docker Restart Policies](#8-docker-restart-policies)
9. [Dockerfile](#9-dockerfile)
10. [Docker Compose](#10-docker-compose)
11. [Docker Logs and Inspection](#11-docker-logs-and-inspection)
12. [Docker Cleanup](#12-docker-cleanup)
13. [Docker System Commands](#13-docker-system-commands)
14. [Common Errors and Solutions](#14-common-errors-and-solutions)
15. [Important Notes](#15-important-notes)

---

## 1. Docker Basics

### Check Docker Version

```bash
docker --version
```

### Display Docker System Information

```bash
docker info
```

### Check Running Containers

```bash
docker ps
```

### Check All Containers

```bash
docker ps -a
```

### Check Docker Images

```bash
docker images
```

### Check Docker Disk Usage

```bash
docker system df
```

### Test Docker Installation

```bash
docker run --rm hello-world
```

This downloads the `hello-world` image if needed, runs it, and removes the container after it exits.

---

## 2. Docker Images

A Docker image is a template used to create containers.

### Build an Image

Build an image using the Dockerfile in the current directory.

```bash
docker build -t my-image .
```

### Build an Image with a Version Tag

```bash
docker build -t my-app:v1 .
```

### Build Without Using the Build Cache

```bash
docker build --no-cache -t my-image .
```

### List All Images

```bash
docker images
```

### Pull an Image

Download an image from a registry.

```bash
docker pull nginx
```

Pull a specific version:

```bash
docker pull node:20-alpine
```

### Tag an Image

```bash
docker tag my-image my-app:v1
```

Tag an image for Docker Hub:

```bash
docker tag my-image YOUR_DOCKERHUB_USERNAME/my-image:latest
```

### Login to Docker Hub

```bash
docker login
```

### Push an Image to Docker Hub

```bash
docker push YOUR_DOCKERHUB_USERNAME/my-image:latest
```

Replace `YOUR_DOCKERHUB_USERNAME` with your actual Docker Hub username. The repository must exist or be creatable under your account, and you must have permission to push.

### Inspect an Image

```bash
docker image inspect nginx
```

### View Image History

```bash
docker image history nginx
```

### Remove an Image

```bash
docker rmi my-image
```

Remove a specific image tag:

```bash
docker rmi my-image:v1
```

### Remove Dangling Images

```bash
docker image prune
```

### Remove All Images Not Used by Containers

```bash
docker image prune -a
```

**Warning:** The `-a` option removes all images not referenced by existing containers, including images you may want to reuse later.

---

## 3. Docker Containers

A container is an instance of a Docker image.

### Create a Container Without Starting It

```bash
docker create --name my-container nginx:latest
```

### Start an Existing Container

```bash
docker start my-container
```

### Create and Run a Container

```bash
docker run --name my-container nginx
```

### Run a Container in Detached Mode

```bash
docker run -d --name my-nginx nginx
```

The `-d` option runs the container in the background.

### Run an Interactive Container

```bash
docker run -it --name my-ubuntu ubuntu bash
```

### Run a Temporary Container

```bash
docker run --rm hello-world
```

The `--rm` option automatically removes the container when it exits.

### List Running Containers

```bash
docker ps
```

### List All Containers

```bash
docker ps -a
```

### Start a Container

```bash
docker start my-container
```

### Stop a Container

```bash
docker stop my-container
```

### Stop Multiple Containers

```bash
docker stop container1 container2
```

### Restart a Container

```bash
docker restart my-container
```

### Pause a Container

```bash
docker pause my-container
```

### Resume a Paused Container

```bash
docker unpause my-container
```

### Remove a Stopped Container

```bash
docker rm my-container
```

### Force-Remove a Container

```bash
docker rm -f my-container
```

This forcibly stops and removes the container.

### Execute a Command in a Running Container

```bash
docker exec -it my-container sh
```

If Bash is installed:

```bash
docker exec -it my-container bash
```

### Run a Command Without Opening a Shell

```bash
docker exec my-container ls /app
```

### Rename a Container

```bash
docker rename old-name new-name
```

### Copy a File into a Container

```bash
docker cp file.txt my-container:/app/file.txt
```

### Copy a File from a Container

```bash
docker cp my-container:/app/file.txt ./file.txt
```

---

## 4. Docker Ports

Port mapping makes an application running inside a container accessible from the host.

### Publish a Port

```bash
docker run -d --name my-nginx -p 8080:80 nginx
```

Open the following address in your browser:

```text
http://localhost:8080
```

Here, `8080` is the host port and `80` is the container port.

### Run a Node.js Application

```bash
docker run -d --name my-app -p 5000:5000 my-image
```

Open:

```text
http://localhost:5000
```

The application must listen on `0.0.0.0` inside the container for normal external port publishing to work.

### View Container Port Mappings

```bash
docker port my-nginx
```

### Check Published Ports

```bash
docker ps
```

### Avoid a Port Conflict

If host port `5000` is already allocated, map another host port:

```bash
docker run -d --name my-app -p 5001:5000 my-image
```

Access the application at `http://localhost:5001`.

The container still uses port `5000`.

---

## 5. Docker Volumes

Volumes persist data independently of a container's writable layer.

### Create a Named Volume

```bash
docker volume create my-volume
```

### List Volumes

```bash
docker volume ls
```

### Inspect a Volume

```bash
docker volume inspect my-volume
```

### Run a Container with a Named Volume

```bash
docker run -it --name volume-demo \
  -v my-volume:/data \
  ubuntu bash
```

Files written to `/data` are stored in the named volume.

### Run a Container with an Anonymous Volume

```bash
docker run -it --rm -v /data ubuntu bash
```

Docker creates an anonymous volume for `/data`.

### Use the `--mount` Syntax

```bash
docker run -it --rm \
  --mount source=my-volume,target=/data \
  ubuntu bash
```

The `--mount` syntax is often easier to read than `-v`.

### Remove a Volume

```bash
docker volume rm my-volume
```

The volume must not be in use by a container.

### Remove Unused Volumes

```bash
docker volume prune
```

Remove all unused local volumes, including named volumes:

```bash
docker volume prune -a
```

**Warning:** Removing a volume permanently deletes the data stored in it.

---

## 6. Bind Mounts

A bind mount connects a directory or file on the host machine to a location inside a container.

Bind mounts are useful for development because changes made to project files on the host can be reflected inside the container.

### Basic Bind Mount

```bash
docker run -it --rm \
  --mount type=bind,source=/absolute/host/path,target=/app \
  ubuntu bash
```

Replace `/absolute/host/path` with an existing directory on your computer.

### Windows CMD: Bind Mount the Current Directory

Run this command from your project directory in Windows Command Prompt.

```cmd
docker run -it --rm --name bind-demo ^
  -v "%cd%:/app" ^
  -w /app ^
  node:20-alpine sh
```

### Git Bash on Windows: Bind Mount a Project

Example project directory: `C:/Apollo/docker-first`

```bash
MSYS_NO_PATHCONV=1 docker run -it --rm --name bind-demo \
  -v "C:/Apollo/docker-first:/app" \
  -w /app \
  node:20-alpine sh
```

### Node.js Development with Nodemon

The following example mounts the project directory, keeps `node_modules` in a named volume, installs dependencies, and starts the application with Nodemon.

**Git Bash on Windows:**

```bash
MSYS_NO_PATHCONV=1 docker run -it --rm \
  --name bind-demo \
  -v "C:/Apollo/docker-first:/app" \
  -v node_modules:/app/node_modules \
  -w /app \
  -p 5000:5000 \
  node:20-alpine \
  sh -c "npm install -g nodemon && npm install && nodemon --watch /app --legacy-watch index.js"
```

**Windows CMD:**

```cmd
docker run -it --rm --name bind-demo ^
  -v "%cd%:/app" ^
  -v node_modules:/app/node_modules ^
  -w /app ^
  -p 5000:5000 ^
  node:20-alpine ^
  sh -c "npm install -g nodemon && npm install && nodemon --watch /app --legacy-watch index.js"
```

Run the CMD command from your project directory.

### Bind Mount Command Explanation

| Option                              | Purpose                                            |
| ----------------------------------- | -------------------------------------------------- |
| `-v "HOST_PATH:/app"`               | Mounts the host project directory                  |
| `-v node_modules:/app/node_modules` | Stores dependencies in a named volume              |
| `-w /app`                           | Sets the working directory                         |
| `-p 5000:5000`                      | Maps host port 5000 to container port 5000         |
| `nodemon`                           | Restarts the application when watched files change |
| `--legacy-watch`                    | Enables polling-based file watching                |
| `--rm`                              | Removes the container after it exits               |

**Important:** Replace `C:/Apollo/docker-first` with your actual project path. Ensure that `index.js` exists and that the application listens on `0.0.0.0:5000` inside the container.

The named `node_modules` volume can persist after the container is removed. If dependencies change, you may need to rebuild or refresh that volume.

---

## 7. Docker Networks

Docker networks allow containers to communicate with each other.

### List Networks

```bash
docker network ls
```

### Create a Network

```bash
docker network create my-network
```

### Inspect a Network

```bash
docker network inspect my-network
```

### Run a Container on a Network

```bash
docker run -d --name app1 --network my-network nginx
```

### Connect a Running Container to a Network

```bash
docker network connect my-network my-container
```

### Disconnect a Container from a Network

```bash
docker network disconnect my-network my-container
```

### Remove a Network

```bash
docker network rm my-network
```

The network must not have connected containers that prevent removal.

### Remove Unused Networks

```bash
docker network prune
```

### Example: Two Containers on the Same Network

```bash
docker network create app-network

docker run -d --name web --network app-network nginx

docker run --rm --network app-network \
  busybox wget -qO- http://web
```

The second container accesses the Nginx container using its container name, `web`.

---

## 8. Docker Restart Policies

Restart policies control when Docker automatically restarts a container.

Available policies:

- `no` — Do not automatically restart.
- `always` — Always attempt to restart.
- `unless-stopped` — Restart unless explicitly stopped.
- `on-failure` — Restart after a non-zero exit.

### No Automatic Restart

```bash
docker run -d --name no-restart \
  --restart no \
  alpine sleep 1d
```

### Always Restart

```bash
docker run -d --name always-test \
  --restart always \
  alpine sleep 1d
```

### Restart Unless Stopped

```bash
docker run -d --name unless-test \
  --restart unless-stopped \
  alpine sleep 1d
```

### Restart on Failure

```bash
docker run -d --name on-failure \
  --restart on-failure \
  alpine sleep 1d
```

### Restart on Failure with a Retry Limit

```bash
docker run -d --name retry-test \
  --restart on-failure:5 \
  alpine sleep 1d
```

### Test a Failed Container

```bash
docker run --name check-failure \
  --restart on-failure:3 \
  alpine sh -c "exit 1"
```

The container exits with status `1`, so Docker can attempt to restart it according to the policy.

### Inspect a Restart Policy

```bash
docker inspect always-test \
  --format '{{.HostConfig.RestartPolicy}}'
```

### Stop Multiple Containers

```bash
docker stop always-test unless-test
```

### Remove Restart Policy Test Containers

```bash
docker rm -f always-test unless-test on-failure check-failure retry-test no-restart
```

### Shut Down WSL

Run in Windows CMD or PowerShell:

```powershell
wsl --shutdown
```

This shuts down running WSL distributions and their processes. It is not a Docker restart policy and can interrupt Docker Desktop when it relies on WSL 2.

---

## 9. Dockerfile

A Dockerfile contains instructions for building an image.

### Example Node.js Dockerfile

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install --omit=dev

COPY . .

EXPOSE 5000

CMD ["node", "index.js"]
```

This example assumes that the project has an `index.js` entry point and a `package.json` file.

For a reproducible production build, commit `package-lock.json` and use `npm ci --omit=dev` instead of `npm install --omit=dev`.

### Build the Image

```bash
docker build -t my-node-app .
```

### Run the Application

```bash
docker run -d --name my-node-app -p 5000:5000 my-node-app
```

### Example `.dockerignore`

Create a `.dockerignore` file in the project root.

```text
node_modules
npm-debug.log
.git
.gitignore
.env
coverage
dist
```

This prevents unnecessary files from entering the build context. Do not exclude files required by your build.

---

## 10. Docker Compose

Docker Compose manages applications using a YAML configuration file.

### Example `compose.yaml`

```yaml
services:
    app:
        build: .
        container_name: my-node-app
        ports:
            - '5000:5000'
        restart: unless-stopped
```

### Start the Application

```bash
docker compose up
```

Run in detached mode:

```bash
docker compose up -d
```

### Build and Start

```bash
docker compose up --build -d
```

### List Compose Services

```bash
docker compose ps
```

### View Compose Logs

```bash
docker compose logs
```

Follow logs:

```bash
docker compose logs -f
```

### Stop Compose Services

```bash
docker compose stop
```

### Stop and Remove Compose Containers

```bash
docker compose down
```

Remove containers and named volumes declared in the Compose file:

```bash
docker compose down -v
```

**Warning:** The `-v` option can permanently delete data stored in the removed volumes.

---

## 11. Docker Logs and Inspection

### View Container Logs

```bash
docker logs my-container
```

### Follow Logs in Real Time

```bash
docker logs -f my-container
```

### Show the Last 100 Lines

```bash
docker logs --tail 100 my-container
```

### Show Timestamps in Logs

```bash
docker logs -t my-container
```

### Inspect a Container

```bash
docker inspect my-container
```

### Display Container IP Address

```bash
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' my-container
```

### Display Container State

```bash
docker inspect -f '{{.State.Status}}' my-container
```

### Display Exit Code

```bash
docker inspect -f '{{.State.ExitCode}}' my-container
```

### Display Restart Count

```bash
docker inspect -f '{{.RestartCount}}' my-container
```

### Display Container Resource Usage

```bash
docker stats
```

### View Processes Inside a Container

```bash
docker top my-container
```

---

## 12. Docker Cleanup

Cleanup commands remove unused Docker resources.

### Remove Stopped Containers

```bash
docker container prune
```

### Remove Dangling Images

```bash
docker image prune
```

### Remove All Unused Images

```bash
docker image prune -a
```

### Remove Unused Networks

```bash
docker network prune
```

### Remove Unused Volumes

```bash
docker volume prune
```

### Remove Unused Containers, Networks, and Images

```bash
docker system prune
```

### Remove All Unused Images and Other Eligible Resources

```bash
docker system prune -a
```

### Include Unused Volumes in System Cleanup

```bash
docker system prune --volumes
```

**Warning:** Cleanup commands can permanently remove data and resources. Review the prompt before confirming. Check volumes carefully before removing them.

---

## 13. Docker System Commands

### View Docker Disk Usage

```bash
docker system df
```

### Show Detailed Disk Usage

```bash
docker system df -v
```

### Check Docker Version and Components

```bash
docker version
```

### Check Docker Daemon Information

```bash
docker info
```

### View Docker Events

```bash
docker events
```

### View Help

```bash
docker --help
```

Help for a specific command:

```bash
docker run --help
```

---

## 14. Common Errors and Solutions

### Error 1: Docker Engine Is Not Running

**Possible error:**

```text
failed to connect to the docker API
```

**Solutions:**

1. Open Docker Desktop.
2. Wait for the Docker Engine to finish starting.
3. Check the connection:

```bash
docker info
```

On Windows with the WSL 2 backend, if Docker Desktop is stuck, you can try:

```powershell
wsl --shutdown
```

Then restart Docker Desktop. This shuts down WSL distributions, so save any work running inside them first.

### Error 2: Port Is Already Allocated

**Possible error:**

```text
Bind for 0.0.0.0:5000 failed: port is already allocated
```

Find containers using published ports:

```bash
docker ps
```

Stop the container using port 5000:

```bash
docker stop CONTAINER_NAME
```

Alternatively, publish a different host port:

```bash
docker run -p 5001:5000 my-image
```

### Error 3: Container Name Already in Use

**Possible error:**

```text
The container name is already in use
```

List all containers:

```bash
docker ps -a
```

Remove the old container if you no longer need it:

```bash
docker rm my-container
```

Or start the existing container:

```bash
docker start my-container
```

### Error 4: `npm ci` Requires a Lock File

**Possible error:**

```text
npm ci can only install with an existing package-lock.json
```

Ensure that `package-lock.json` exists in your project and is copied into the image before running `npm ci`.

Example:

```dockerfile
COPY package*.json ./
RUN npm ci --omit=dev
```

If the project does not have a lock file, generate one in the project directory:

```bash
npm install
```

Commit the resulting `package-lock.json` and rebuild the image.

### Error 5: Bind Mount Path Is Incorrect in Git Bash

Use a Windows-style absolute path and disable Git Bash path conversion for the Docker command:

```bash
MSYS_NO_PATHCONV=1 docker run --rm \
  -v "C:/Apollo/docker-first:/app" \
  -w /app \
  node:20-alpine pwd
```

Replace the path with your actual project directory.

### Error 6: Container Exits Immediately

Inspect the container:

```bash
docker ps -a
```

Read its logs:

```bash
docker logs my-container
```

Inspect its exit code:

```bash
docker inspect -f '{{.State.ExitCode}}' my-container
```

Check the startup command, application entry point, environment variables, and application errors.

---

## 15. Important Notes

### Image vs Container

| Image                                                        | Container                                    |
| ------------------------------------------------------------ | -------------------------------------------- |
| A template for creating containers                           | An instance of an image                      |
| Created with `docker build` or downloaded with `docker pull` | Created with `docker create` or `docker run` |
| Managed with `docker images`                                 | Managed with `docker ps`                     |
| Removed with `docker rmi`                                    | Removed with `docker rm`                     |

### `docker create` vs `docker start` vs `docker run`

| Command         | Purpose                                 |
| --------------- | --------------------------------------- |
| `docker create` | Creates a container without starting it |
| `docker start`  | Starts an existing container            |
| `docker run`    | Creates and starts a container          |

### Named Volume vs Bind Mount

| Named Volume                             | Bind Mount                   |
| ---------------------------------------- | ---------------------------- |
| Managed by Docker                        | Uses a host filesystem path  |
| Suitable for persistent application data | Useful for local development |
| Example: `my-volume:/data`               | Example: `./src:/app/src`    |

### Common Command Options

| Option      | Meaning                            |
| ----------- | ---------------------------------- |
| `-d`        | Run in detached mode               |
| `-it`       | Interactive terminal               |
| `--rm`      | Remove the container when it exits |
| `--name`    | Assign a container name            |
| `-p`        | Publish a container port           |
| `-v`        | Mount a volume or bind mount       |
| `-w`        | Set the working directory          |
| `-e`        | Set an environment variable        |
| `--restart` | Configure a restart policy         |

### Important Safety Tips

- Never store passwords, API keys, or production secrets in a Dockerfile or public repository.
- Use `.dockerignore` to exclude unnecessary files and sensitive local configuration.
- Do not use `docker system prune -a` without checking which resources you need.
- Do not remove volumes containing important application data.
- Use a lock file and `npm ci` for reproducible Node.js production builds.
- Remember that `EXPOSE` documents a container port; `-p` publishes it to the host.
- Remember that `docker stop` stops a container but does not remove it.

---

**Happy Docker Learning!**

## N.B

Final 3 Things to Verify [English]

1. Make sure C:/Apollo/docker-first matches your actual project directory.
2. Verify that your Node.js application's entry point is index.js and that the application listens on 0.0.0.0:5000 inside the container.
3. When pushing an image to Docker Hub, replace YOUR_DOCKERHUB_USERNAME with your actual Docker Hub username.

শেষবারের মতো ৩টি বিষয় যাচাই করবে [Bangla]

1. C:/Apollo/docker-first path-টি তোমার নিজের project path কি না।
2. Node.js application-এর entry point সত্যিই index.js কি না এবং application 0.0.0.0:5000-এ listen করছে কি না।
3. Docker Hub-এ push করার সময় YOUR_DOCKERHUB_USERNAME-এর জায়গায় নিজের username ব্যবহার করবে।
