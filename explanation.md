# Docker Implementation Documentation
## YOLO E-Commerce Application

## 1. Base Image Selection

*MongoDB:*
Went with `mongo:7`- it's the official MongoDB image, works out of the box, and gets regular updates. Didn't see any reason to customize it or use something else.

*Backend:*
Used `node:18-alpine` because Alpine is tiny - cuts the image from ~900MB to 5MB. Less to download, faster deployments, and smaller attack surface. Still has everything Node needs to run.

*Frontend:*
Used a multi-stage build here with two different images:

Build stage - `node:18-alpine`
Need Node to compile the React app. Kept it consistent with the backend by using Alpine here too. This stage has access to npm and all the build tools needed.

Production stage - `nginx:stable-alpine`
React compiles down to just HTML, CSS, and JavaScript files. Don't need Node anymore at this point, just a web server. Nginx with Alpine gets the final image down to about 23MB and handles serving static files really well with built-in caching and compression.

The multi-stage approach basically lets me use the heavy build tools, then throw them away and keep only what's needed to actually run the app. Got the production image size down by 97% this way.

## 2. Dockerfile Architecture

### Backend Dockerfile

```dockerfile
FROM node:18-alpine
WORKDIR /app
```
Sets up the base image and working directory for subsequent operations.

```dockerfile
COPY package.json .
COPY package-lock.json .
RUN npm ci --omit=dev
```
Dependancies are copied prior to applications source code to leverage dockers layer caching mechanism and this ensures that unchanged dependancies don't trigger reinstallation during subsequent builds.

Used `npm ci` instead of `npm install` because it's more reliable installs exactly what's in package-lock.json. The `--omit=dev` flag skips dev dependencies to keep the image smaller.

```dockerfile
COPY . .
EXPOSE 5000
ENV PORT=5000
CMD ["npm", "start"]
```
Application source code is copied after dependency installation to maximize cache efficiency.

`EXPOSE` directive documents port usage, while `ENV` establishes runtime configuration. 

`CMD` directive employs exec form for proper signal handling.

### Frontend Dockerfile

**Build Stage:**
```dockerfile
FROM node:18-alpine AS build
WORKDIR /app
COPY package.json .
COPY package-lock.json .
RUN npm ci
COPY . .
RUN NODE_OPTIONS=--openssl-legacy-provider npm run build
```
Build stage compiles the React application with all necessary development dependencies. 

`NODE_OPTIONS` flag addresses OpenSSL compatibility requirements with legacy React build tools.

**Production Stage:**
```dockerfile
FROM nginx:stable-alpine AS production
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```
The production stage extracts only compiled artifacts from the build stage, eliminating build tools and dependencies. The `--from=build` flag enables selective copying between stages. Nginx operates in foreground mode (`daemon off`) to maintain container lifecycle.

---

## 3. Network Architecture and Port Configuration

### Port Allocation

*MongoDB Service:*
The standard MongoDB port
```yaml
ports:
  - "27017:27017"
```

*Backend Service:*
API endpoint
```yaml
ports:
  - "5000:5000"
```

*Frontend Service:*
Nginx serves on port 80 inside the container, mapped to port 3000 on my machine (keeps it consistent with the usual React dev setup)
```yaml
ports:
  - "3000:80"
```

### Network Implementation

```yaml
networks:
  yolo-network:
    driver: bridge
```

Set up a bridge network so all the containers can talk to each other. The cool thing about Docker networks is they have built-in DNS. So the backend can just connect to `mongodb://mongodb:27017/yolo` using the service name, and Docker figures out the IP address automatically. Way easier than managing IPs manually.

Other benefits:
- Containers are isolated from external networks and other Docker networks
- Service names automatically resolve to the right container
- Traffic between containers stays inside the Docker network

*How it all connects:*
```
Browser → Frontend (hostmachine:80) → Backend (backend:5000) → MongoDB (mongodb:27017)
```

## 4. Volume Management and Data Persistence

### Volume Configuration

```yaml
volumes:
  mongodb_data:
  mongodb_config:

services:
  mongodb:
    volumes:
      - mongodb_data:/data/db
      - mongodb_config:/data/configdb
```

MongoDB needs volumes or all the data disappears when you stop the container. Set up two volumes:

*mongodb_data → /data/db*
Where MongoDB stores all the actual database stuff - collections, documents, indexes, everything. Without this, you lose all your data when the container restarts.

*mongodb_config → /data/configdb* 
Stores MongoDB's config files and settings so everything stays consistent across restarts.

*How it works:*
```
Container Start → Volume Mount → Data Operations → Container Stop → Data Persists
```

## 5. Getting It Running

*Build Process:*
```bash
docker-compose build
```
All services compiled successfully without dependency resolution failures or build errors.

*Container Initialization:*
```bash
docker-compose up -d
```
Services started in detached mode `-d` with no immediate failures or restart loops.

*Verification:*
```bash
docker-compose ps
```
Expected output:
```
NAME            IMAGE           COMMAND                  SERVICE    CREATED         STATUS         PORTS
yolo-backend    yolo-backend    "docker-entrypoint.s…"   backend    50 minutes ago   Up 50 minutes   0.0.0.0:5000->5000/tcp, [::]:5000->5000/tcp
yolo-frontend   yolo-frontend   "/docker-entrypoint.…"   frontend   50 minutes ago   Up 50 minutes   0.0.0.0:3000->80/tcp, [::]:3000->80/tcp
yolo-mongodb    mongo:7         "docker-entrypoint.s…"   mongodb    50 minutes ago   Up 50 minutes  0.0.0.0:27017->27017/tcp, [::]:27017->27017/tcp
```

**Functional Testing:**
- Frontend accessible at http://localhost:80
- Backend API responsive at http://localhost:5000  
- Database connectivity verified
- CRUD operations functional


## 6. Naming Convention

Kept it simple with the format yolo-[service]:

- `yolo-mongodb` - database
- `yolo-backend` - API
- `yolo-frontend` - web interface

Makes it obvious what's what when looking at container lists and logs.


## 7. Publishing to Docker Hub

*Login:*
```bash
docker login
```

*Tag the images:*
```bash
docker tag yolo-backend sammaingi/yolo-backend:1.0.0
docker tag yolo-frontend sammaingi/yolo-frontend:1.0.0
```

*Push to Docker Hub:*
```bash
docker push sammaingi/yolo-backend:1.0.0
docker push sammaingi/yolo-frontend:1.0.0
```

*Pull them later:*
```bash
docker pull sammaingi/yolo-backend:1.0.0
docker pull sammaingi/yolo-frontend:1.0.0
```

Images are now available at:

Backend: https://hub.docker.com/r/sammaingi/yolo-backend

Frontend: https://hub.docker.com/r/sammaingi/yolo-frontend

### Screenshot:

*Frontend Repository:*
![DockerHub Frontend Repository](./frontend.png)
*Figure 1: DockerHub repository showing yolo-frontend image with version 1.0.0*

*Backend Repository:*
![DockerHub Backend Repository](./backend.png)
*Figure 2: DockerHub repository showing yolo-backend image with version 1.0.0*