## Check Databases Interactively

Old ---> docker exec -it vidly-mydatabase-1 mongo --> container name & Image name

New ---> docker exec -it vidly-mydatabase-1 mongosh

show dbs

use vidly

show collections

db.movies.find().pretty()

## To see the Container Ports

docker inspect -f "{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}" vidly-mydatabase-1

## Finally Worked

mongodb://host.docker.internal:27017

## localhost didn't work, But host.docker.internal worked --->

The reason localhost didn't work while host.docker.internal did is due to how Docker handles networking on different platforms, particularly on Windows and macOS when using Docker Desktop.

#### Here's why:

### 1. **Docker Network Isolation**:

- When you run Docker containers, they are typically isolated from your host machine (the operating system). Each container has its own network environment.
- Docker containers use **network bridges** and may not directly communicate with your host machine using `localhost` in some cases, especially when you're trying to access services inside the container from your host.

### 2. **`localhost` Behavior**:

- When you try to use `localhost` on the host machine to access a service running inside the container, **Docker doesn't route `localhost` directly to the container**. Instead, it expects network communication between the container and the host to go through a specific interface.

### 3. **`host.docker.internal`**:

- Docker Desktop (for Windows and macOS) provides a special DNS name, `host.docker.internal`, which allows containers to communicate with services running on the host machine (i.e., your computer's OS).
- **`host.docker.internal`** is a virtual hostname that Docker automatically resolves to the IP address of your host machine. This enables containers to access services on the host, and vice versa.

### 4. **Linux**:

- On **Linux**, Docker doesn't provide `host.docker.internal` by default. Instead, you would use the IP of the container or the host's local network IP to access it. Hence, `localhost` might work on Linux, but `host.docker.internal` is not automatically available.

#### Summary:

- On **Windows** or **macOS** using **Docker Desktop**, `host.docker.internal` is a special DNS alias that allows Docker containers to access the host machine, which is why it worked for you.
- **`localhost`** doesn't work in this case because Docker containers don't directly bind to the host's `localhost` interface.

## Microservice(.NET Project Docker Compose) working vs this project(Vidly Docker Compose) Not working

#### 1. **Port Mapping and Accessibility**:

The reason you're able to access the MongoDB database using `localhost` in your other Docker Compose project (but not in the current one) is related to how the **ports** are mapped and exposed in the Docker Compose configuration.

##### Key Points:

- **Port Mapping**:
  - The line `- "27014:27021"` in your Docker Compose file means that the container's **port 27021** (inside the container) is being **mapped to port 27014** on your **host machine**.
  - This means that when you try to access `localhost:27014` on your host, it will be routed to the **MongoDB container's port 27021**.
  - In your case, `localhost:27014` is the accessible address on your host machine, which is why it works with `localhost` as expected.

#### 2. **What Happened in the Other Docker Compose File**:

- In this file, MongoDB is **explicitly mapped** to `localhost:27014`, and since `localhost` on the host machine is being mapped to the MongoDB container, you can connect directly using `localhost`.
- Specifically, the mapping is from **host port 27014** to **container port 27021**, which makes the database available on `localhost:27014`.

#### 3. **In Your Current Project**:

- In your current Docker setup (with MongoDB on port 27017 exposed as `0.0.0.0:27017->27017`), it should technically be accessible via `localhost:27017`.
- The fact that **`host.docker.internal`** worked but `localhost` didn’t suggests that **Docker's network configuration** might be slightly different. This could be a result of a network issue or misconfiguration, as `localhost` should typically work when the port is exposed on the host system.

#### 4. **Why `localhost` Worked in Your Other Project**:

- If you explicitly map the ports in Docker Compose (as you did with `27014:27021`), Docker Desktop on Windows/Mac will automatically handle routing correctly to the `localhost`, making it accessible via `localhost:27014` on the host.

#### 5. **In Summary**:

- **Explicit Port Mapping** (`localhost:27014 -> container port 27021`) in your Docker Compose project made `localhost` work.
- In your current setup, since you didn't specify a custom port mapping, Docker should expose the port correctly (as `0.0.0.0:27017->27017`), but `host.docker.internal` is necessary on Windows/Mac because the container uses a bridged network that doesn't map `localhost` directly.
