# Build stage: Use Node.js 18 on Alpine Linux as the base image
FROM node:18-alpine

# Set the working directory inside the container
WORKDIR /app

# Copy package.json and package-lock.json first for better layer caching
COPY package.json .
COPY package-lock.json .

# Install dependencies using npm ci (clean install for reproducible builds)
RUN npm ci --omit=dev

# Copy the rest of the application source code
COPY . .

# Expose port 5000 for HTTP traffic
EXPOSE 5000

ENV PORT=5000

# Start Application
CMD ["npm", "start"]
