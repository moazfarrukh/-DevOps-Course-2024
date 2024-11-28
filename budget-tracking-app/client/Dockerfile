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

# Step 7: Use a simple static file server to serve the React app
FROM nginx:alpine

# Step 8: Copy the build folder to the nginx directory
COPY --from=build /app/build /usr/share/nginx/html

# Step 9: Expose port 80 for the React app
EXPOSE 80

# Step 10: Start nginx server
CMD ["nginx", "-g", "daemon off;"]

