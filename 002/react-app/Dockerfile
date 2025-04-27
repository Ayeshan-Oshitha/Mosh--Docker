FROM node:14.16.0-alpine3.13

# By Default, Docker run the application with root user, that has the highest privilages. 
# So, To run this application we should create a regular user with limited privilages.
RUN addgroup myuser && adduser -S -G myuser myuser
# Switch to the 'myuser' so the application runs under a regular user, ensuring security
USER myuser

# Set the working directory inside the container to '/app'
# This is where your application files will be placed and where commands will be executed
WORKDIR /app


# Copy only 'package.json' and 'package-lock.json' from the host machine to the container's /app directory
# This allows npm install to run without copying the entire codebase yet, enabling better caching.
COPY package*.json .

RUN npm install

# Copy all the other application files (source code, config files, etc.) from the host machine to the container’s /app directory
COPY . .

# This container will listen on port 3000 
# (Not in Host machine, This is only releated to Container Port)
# to document the port the container is listening on
EXPOSE 3000  

CMD [ "npm", "start" ]



