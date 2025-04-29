FROM node:14.16.0-alpine3.13

RUN addgroup myuser && adduser -S -G myuser myuser
USER myuser

WORKDIR /app

# Create a directory inside the container/image where the volume will be mounted
RUN mkdir -p /app/data

COPY package*.json .

RUN npm install

COPY . .

EXPOSE 3000  

CMD [ "npm", "start" ]



