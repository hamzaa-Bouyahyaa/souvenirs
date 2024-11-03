# Stage 1: Build the React app
FROM node:14.21.2-alpine AS build

# Set working directory inside the container
WORKDIR /app

# Copy package.json and package-lock.json to the container
COPY package.json package-lock.json ./

# Install dependencies
RUN npm install

# Copy all files to the container
COPY . .

# Build the React app
RUN npm run build

# Stage 2: Serve the build files with Nginx
FROM nginx:alpine

# Copy the build files from the previous stage to the Nginx public directory
COPY --from=build /app/build /usr/share/nginx/html

# Copy a custom Nginx configuration file, if you have one (optional)
# COPY nginx.conf /etc/nginx/nginx.conf

# Expose port 80
EXPOSE 80

# Start Nginx
CMD ["nginx", "-g", "daemon off;"]
