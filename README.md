# YOLO - E-commerce Application

A full-stack e-commerce application built with Node.js, Express, MongoDB, and React. 
It Demonstrates modern web development practices with containerized deployment using Docker.

## Features
- Product Management - List, Add, Edit and Delete Products
- Modern React Frontend
- Restful API Backend
- Persistent Data Storage with MongoDB

## Tech Stack

### Frontend
- React.js - UI library
- CSS3 - Styling
- HTML5 - Markup

### Backend
- Node.js - Runtime environment
- Express.js - Web framework
- MongoDB - NoSQL database
- Mongoose - MongoDB object modeling

### DevOps
- Docker - Containerization
- Docker Compose - Multi-container orchestration

## Prerequisites

- Node.js v18+
- npm 
- MongoDB v4.4+
- Docker and Docker Compose

## Installation
   
## Docker Deployment

### Quick Start with Docker

1. **Build and start all services**
   ```bash
   docker-compose up --build
   ```

2. **Access the application**
   - Frontend: `http://localhost:80`
   - Backend API: `http://localhost:5001`
   - MongoDB: `mongodb://localhost:27017`

The application consists of three services:

- mongodb: MongoDB database server (port 27017)
- backend: Express.js API server (port 5001)
- frontend: React development server (port 80)

### Docker Commands

```bash
# Start services in detached mode
docker-compose up -d

# Stop all services
docker-compose down

# View logs
docker-compose logs -f

# Rebuild specific service
docker-compose up --build backend

# Remove all containers and volumes
docker-compose down -v
```

## Testing

```bash
# Run backend tests
cd backend
npm test

# Run frontend tests
cd client
npm test
```

## License

This project is licensed under the MIT License - see the [LICENSE](./LICENSE) file for details

## Authors

- layersony - [GitHub Profile](https://github.com/layersony)

## Acknowledgments

- Original project forked from [kadimasum/yolo](https://github.com/kadimasum/yolo)
- Built with the MERN stack
- Inspired by modern e-commerce platforms