# Dockerizing a MERN Stack Application

This guide will walk you through the process of dockerizing a MERN (MongoDB, Express, React, Node.js) application with separate client and server folders for this purpose we will be using my budget tracking app. We'll create Dockerfiles for both the client (React) and server (Node.js with Express), and a `docker-compose.yml` file to orchestrate the services.

---

## Prerequisites

Before you begin, ensure you have the following installed:

- [Docker](https://www.docker.com/get-started)
- [Docker Compose](https://docs.docker.com/compose/install/)
- A MERN application with `client` and `server` directories.

---

## **Docker Configuration**

### **1. Server Dockerfile**

Create a `Dockerfile` inside the `server` directory to build the Node.js application.

```dockerfile
# Step 1: Use the official Node.js image from Docker Hub
FROM node:20-alpine

# Step 2: Set the working directory in the container
WORKDIR /app

# Step 3: Copy the package.json and package-lock.json (or equivalent) to install dependencies
COPY server/package*.json ./

# Step 4: Install the server dependencies
RUN npm install --production

# Step 5: Copy the rest of the server files (source code)
COPY server ./

# Step 6: Build the TypeScript code (if using TypeScript)
RUN npm run build

# Step 7: Expose the port your server runs on (change if necessary)
EXPOSE 5000

# Step 8: Run the server
CMD ["npm", "run", "start"]
```
### **2. Client Dockerfile**

Create a `Dockerfile` inside the `server` directory to build the Node.js application.

```dockerfile
# Step 1: Use the official Node.js image from Docker Hub
FROM node:20-alpine as build

# Step 2: Set the working directory in the container
WORKDIR /app

# Step 3: Copy the package.json and package-lock.json to install dependencies
COPY client/package*.json ./

# Step 4: Install the client dependencies
RUN npm install

# Step 5: Copy the rest of the client files
COPY client ./

# Step 6: Build the React app
RUN npm run build

# Step 7: Use a simple static file server (e.g., `serve`) to serve the React app
FROM nginx:alpine

# Step 8: Copy the build folder to the nginx directory
COPY --from=build /app/build /usr/share/nginx/html

# Step 9: Expose port 80 for the React app
EXPOSE 80

# Step 10: Start nginx server
CMD ["nginx", "-g", "daemon off;"]
```

## Docker Compose

Create a `docker-compose.yml` file to manage the client, server, and mongo services. This will allow you to run all services with a single command.

```yaml
version: '3.8'

services:
  # MongoDB service
  mongo:
    image: mongo:6.0
    container_name: mongo
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: example
      MONGO_INITDB_DATABASE: budget-tracking-app
    volumes:
      - mongo-data:/data/db
    ports:
      - "27017:27017"

  # Server service (Node.js API)
  server:
    build:
      context: .
      dockerfile: server/Dockerfile
    container_name: server
    environment:
      JWT_SECRET: ${JWT_SECRET}
      REFRESH_TOKEN_SECRET: ${REFRESH_TOKEN_SECRET}
      MONGO_DB_CONNECTION_STRING: mongodb://root:example@mongo:27017/budget-tracking-app?authSource=admin
      COOKIE_SECRET: ${COOKIE_SECRET}
    ports:
      - "5000:5000"
    depends_on:
      - mongo

  # Client service (React app)
  client:
    build:
      context: .
      dockerfile: client/Dockerfile
    container_name: client
    ports:
      - "80:80"
    depends_on:
      - server

volumes:
  mongo-data:
```
### 4. Environment Variables 
To manage sensitive information and environment variables, create a `.env` file in your project root. This file will store your configuration settings securely, without hardcoding them into your source code.

#### **Example `.env` File:**
```env
JWT_SECRET=your_jwt_secret
REFRESH_TOKEN_SECRET=your_refresh_token_secret
COOKIE_SECRET=your_cookie_secret
```
Make sure to reference these environment variables in the `docker-compose.yml` and Dockerfile configurations, as shown earlier

## Running the Application 

Once you have created the Dockerfiles and the `docker-compose.yml` file, follow these steps to build, run, and manage the application.

### 1. Build the Docker Images

Navigate to your project root directory and run the following command to build the Docker images for the client, server, and MongoDB:

```bash
docker-compose build
```
### 2. Start the Services
Run the following command to start all services (client, server, and MongoDB):
```bash
docker-compose up
```
### 3. Access the Application
- The **React client** will be available at [http://localhost](http://localhost) (or port 80).
- The **Node.js server** will be available at [http://localhost:5000](http://localhost:5000).
- **MongoDB** will be running on port `27017` (the default MongoDB port).

### 4. Stop the Services
Run the following command to stop all services (client, server, and MongoDB):
```bash
docker-compose down
```

## Conclusion 
You have successfully **dockerized** your MERN stack application! With **Docker** and **Docker Compose**, you can easily spin up a consistent development environment across different machines. This setup simplifies deployment, ensures consistency across environments, and allows for easier scalability and management of your application.

