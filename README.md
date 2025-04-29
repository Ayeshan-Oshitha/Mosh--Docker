# Most Important Commands

`docker build -t hello-docker`

`docker run -d -p 3000:3000 --name <container_name> <image_name>`

`docker volume create <volume_name>`

`docker run -d -p 3000:3000 -v <created_volume_name>:/app/data <image_name>`

# 001 - Introduction to Docker

`docker build -t hello-docker .`

`-t` uses for tag. It means the docker image name

`.` path for Docker file

<hr>

`docker image ls`

list down all the images

<hr>

`docker run hello-docker`

runs the image

# 002 - Building Images

#### Difference between Images and Containers

- An image includes everything an application needs to run such as cut-down Operating System(OS), Third-party libraries, Application files, Environment variables and so on. So Image contains all the files and configuration settings needed to run an application.

- Once we have an image, We can start a container from it.

- A container is kind of like a virtual machine in the sense that It provides an isolated environment for executing an application. And similar to VM, we can stop and restart an container. Technically Container is just a process, But Special kind of process because It has it's own file system which is provided by the image.

- So, Each container is an isolated environment for executing an application.

After building the image, you can open a shell inside the container by running:

`docker run -it my-react-app sh`

`docker run` — Creates and starts a new container from the image.

`it` — Makes the container interactive (so you can type commands manually).

`my-react-app` — The name of your Docker image.

`sh` — Starts a shell inside the container.

**To Inspect the file**

`ls -la ` - Check the files

`printenv` - Check environmental variables

`whoami` - to see the current user

<hr>

## Docker Image Layers & Optimization

#### Docker Image Layers

- **Layers** are created for each instruction in the Dockerfile.
- Each layer represents a stage in the image (e.g., `FROM`, `RUN`, `COPY`).
- Layers are cached to avoid rebuilding unchanged parts, improving build speed.

#### Optimization Strategy

1. **Copy `package.json` First:**

   - Copy only `package.json` and `package-lock.json` before installing dependencies. This allows Docker to cache the dependencies layer.

2. **Install Dependencies Early:**

   - Run `npm install` after copying `package.json` to leverage caching. If dependencies don’t change, Docker skips the install step.

3. **Copy Application Files After Dependencies:**
   - Once dependencies are installed, copy the rest of your app files. This ensures changes in the app code don't force a reinstall of dependencies.

#### Example Optimized Dockerfile

```dockerfile
FROM node:14.16.0-alpine3.13

# Add user and group for security
RUN addgroup myuser && adduser -S -G myuser myuser
USER myuser

WORKDIR /app

# Copy package.json and install dependencies
COPY package*.json ./
RUN npm install

# Copy remaining files
COPY . .

EXPOSE 3000
CMD ["npm", "start"]
```

![alt text](image.png)

<hr>

## Removing Images

`docker container prune` - remove all stope containers

`docker image prune` - Removes dangling images (images not associated with any container)

`docker image prune -a` - Removes all unused images (not just dangling images)

## Tagging Images

When you build a Docker image, you can specify a tag (version or label) using the `-t` option in the `docker build` command. This helps identify and manage different versions of an image.

`docker build -t my-react-app:1.1.0 `

If you've already built an image without tagging it, you can tag it later using the `docker tag` command. This is useful when you want to add another version or label to an existing image.

`docker tag <image_id> my-react-app:1.1.0`

## Saving Docker Image

`docker save -o <output_file.tar> <image_name>:<tag>`

`<output_file.tar>` - The name of the tar file you want to create.

`<image_name>:<tag>` - The name and tag of the image you want to save.

## Loading Docker Image

`docker load -i <image_file.tar>`

`<image_file.tar>` - The tarball file you want to load into Docker.

# 003 - Working with Containers

## Starting the Container

`docker ps` - Lists all running containers.

`docker run -d <docker_image_name>` - Starts a container from the specified image in detached mode (runs in the background).

`docker run -d  --name <custom_container_name> <docker_image_name>` - Runs a container from the specified image in detached mode (background) and gives it a custom name.

## Viewing the logs

`docker logs <containerID>` - Show the logs (output) of a container.

`docker logs -f <containerID>` - Show the logs and keep following (live stream) the new log output.

## Publishing Ports

`docker run -d -p 3000:3000 --name <container_name> <image_name>`

`-d` — Runs the container in detached mode (background).

`-p 3000:3000` — Maps port 3000 on your machine to port 3000 inside the container (host:container).

`--name <container_name>` — Gives a custom name to the running container.

`<image_name>` — The name of the Docker image you want to run.

## Executing commands in Running Container

`docker exec <containerID> <command>` - Runs a command inside a running container.

**ex ->** `docker exec 123abc456def ls`- Listing the files in the WorkDIR

## Stoping and Starting Containers

`docker stop <containerID or container_name>` - Stops the container

`docker start <containerID or container_name>` - Starts the container again

`docker restart <containerID or container_name>` - This stops and then starts the container again.

## Persisting Data using **Volumes**

A volume is a storage outside of a container. It can be directory on the Host or somewhere in the cloud.

`docker volume create <volume_name>` - Creates a new volume

`docker volume inspect <volume_name>` - Provides detailed information about the specified volume, like its mount point and usage.

**Note:** Before assigning a volume to a container, it's a good practice to create a new directory inside the image. If you don't do this, the default directory used to store the data will have permissions that are too open. To ensure better security, the directory should have permissions set for only the created user inside the container.

`docker run -d -p 3000:3000 -v <created_volume_name>:/app/data <image_name>`

`-v <created_volume_name>:/app/data` — Mounts the Docker volume (<created*volume_name>) to the /app/data directory inside the container. If either the volume or the path doesn't exist, Docker will automatically create them. ( \_Note: While Docker can automatically create both the volume and the path, it's best to manually* )

**_Volumes are the right way to persist data in dockerized applications, because there are different lifecycles for each container. Even though if we deleted the container, the associated Volume will not be deleted. It will be still exist. Also, We can share same Volume among multiple containers._**

## Copying files between the Host and Container

`docker cp <container_id_or_name>:<complete_file_path> <host_path>` - Copy from container to host

ex -> `docker cp e1c:/app/log.txt .`

`docker cp <host_file_path> <container_id_or_name>:<container_path>` - Copy from host to container

ex -> `docker cp secret.txt e1c:/app`

## Sharing the Source Code with a container

To share your local source code with a container (for live development or testing), use bind mounts when running the container.
