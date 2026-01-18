# Build stage: Use Node.js 18 on Alpine Linux as the base image
FROM node:18-alpine AS build

# Set the working directory inside the container
WORKDIR /app

# Copy package.json and package-lock.json first for better layer caching
COPY package*.json ./

# Install dependencies using npm ci (clean install for reproducible builds)
RUN npm ci

# Copy the rest of the application source code
COPY . .

# Build application with legacy OpenSSL provider flag for compatibility
RUN NODE_OPTIONS=--openssl-legacy-provider npm run build

# Production stage: Use lightweight Nginx on Alpine Linux
FROM nginx:stable-alpine AS production

# Copy the built application from the build stage to Nginx's html directory
COPY --from=build /app/build /usr/share/nginx/html

EXPOSE 80

# Start Nginx in the foreground (daemon off prevents container from exiting)
CMD ["nginx", "-g", "daemon off;"]