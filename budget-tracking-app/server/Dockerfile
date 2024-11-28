# Step 1: Use the official Node.js image from Docker Hub
FROM node:20-alpine

# Step 2: Set the working directory in the container
WORKDIR /app

# Step 3: Copy the package.json and package-lock.json to install dependencies
COPY server/package*.json ./

# Step 4: Install the server dependencies
RUN npm install --production

# Step 5: Copy the rest of the server files 
COPY server ./

# Step 6: Build the TypeScript code
RUN npm run build

# Step 7: Expose the port your server runs on (change if necessary)
EXPOSE 5000

# Step 8: Run the server
CMD ["npm", "run", "start"]

