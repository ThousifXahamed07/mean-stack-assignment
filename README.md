# MEAN Stack CRUD Application

This is a production-ready MEAN (MongoDB, Express, Angular, Node.js) stack application, fully containerized with Docker and deployed via an automated CI/CD pipeline.

## 🚀 Infrastructure Overview

* **Frontend**: Angular 15+ served via Nginx (Multi-stage Docker build).
* **Backend**: Node.js & Express API with non-root security configurations.
* **Database**: MongoDB with root-user authentication and persistent volumes.
* **Reverse Proxy**: Nginx acting as a Gateway to route `/api` and frontend traffic.
* **Deployment**: Automated CI/CD via GitHub Actions to an AWS EC2 instance.

---

## 🛠️ Step-by-Step Setup & Deployment

### 1. Local Development

To run this project locally, ensure you have Docker and Docker Compose installed:

1. **Clone the repository**:
```bash
git clone https://github.com/ThousifXahamed07/mean-stack-assignment.git
cd mean-stack-assignment
```

2. **Start the stack**:
```bash
docker compose up --build
```

3. **Access the App**:
* Frontend: `http://localhost:4200`
* Backend API: `http://localhost:3000/api/tutorials`

### 2. AWS EC2 Configuration

The application is deployed on an Ubuntu 24.04 LTS instance:

1. **Security Group**: Inbound rules must allow **Port 80 (HTTP)** for public access and **Port 22 (SSH)** for management.
2. **Dependencies**: Install Docker and Docker Compose on the VM:
```bash
sudo apt update && sudo apt install docker.io docker-compose -y
sudo usermod -aG docker $USER && newgrp docker
```

### 3. CI/CD Pipeline (GitHub Actions)

The deployment is fully automated. Every push to the `main` branch triggers:

1. **Build & Push**: Docker images are built and pushed to **Docker Hub** (`thousif07/mean-backend` and `thousif07/mean-frontend`).
2. **Deploy**: The GitHub Action SSHs into the AWS VM, pulls the latest images, and restarts the services using `docker-compose`.

**Required GitHub Secrets**:

* `DOCKER_USER`, `DOCKER_PASS`: Docker Hub credentials.
* `VM_IP`: Public IP of the EC2 instance (`51.20.74.176`).
* `SSH_PRIVATE_KEY`: Content of the `.pem` file for VM access.

---

## 🔗 Live Application

The live application is accessible at: **[http://51.20.74.176](http://51.20.74.176)**

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                         Client Browser                       │
└──────────────────────────┬──────────────────────────────────┘
                           │ HTTP (Port 80)
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                    Nginx Reverse Proxy                       │
│  Routes:                                                     │
│    /          → Frontend (Angular SPA)                       │
│    /api/*     → Backend (Node.js API)                        │
└──────────────┬──────────────────────────┬───────────────────┘
               │                          │
               ▼                          ▼
┌──────────────────────────┐  ┌─────────────────────────────┐
│   Frontend Container     │  │    Backend Container        │
│   Angular 15 + Nginx     │  │    Node.js + Express        │
│   Port: 4200→80          │  │    Port: 3000               │
└──────────────────────────┘  └──────────────┬──────────────┘
                                             │
                                             ▼
                              ┌─────────────────────────────┐
                              │   MongoDB Container         │
                              │   Port: 27017               │
                              │   Volumes: Persistent Data  │
                              └─────────────────────────────┘
```

---

## 📁 Project Structure

```
crud-dd-task-mean-app/
├── backend/                    # Node.js backend service
│   ├── Dockerfile             # Backend container configuration
│   ├── .dockerignore          # Files to exclude from Docker build
│   ├── server.js              # Express server entry point
│   ├── package.json           # Node.js dependencies
│   └── app/
│       ├── config/            # Database configuration
│       ├── controllers/       # Business logic
│       ├── models/            # MongoDB schemas
│       └── routes/            # API route definitions
├── frontend/                   # Angular frontend service
│   ├── Dockerfile             # Multi-stage build for Angular
│   ├── .dockerignore          # Files to exclude from Docker build
│   ├── nginx.conf             # Nginx configuration for SPA
│   ├── angular.json           # Angular project configuration
│   ├── package.json           # npm dependencies
│   └── src/                   # Angular source code
├── nginx/                      # Reverse proxy configuration
│   ├── nginx.conf             # Main Nginx configuration
│   └── conf.d/
│       └── default.conf       # Proxy rules and routing
├── docker-compose.yml         # Multi-container orchestration
├── .env.example               # Environment variables template
└── README.md                  # This file
```

---

## 🔌 API Endpoints

### Tutorials

| Method | Endpoint                    | Description              |
|--------|----------------------------|--------------------------|
| GET    | `/api/tutorials`           | Get all tutorials        |
| GET    | `/api/tutorials/:id`       | Get tutorial by ID       |
| POST   | `/api/tutorials`           | Create new tutorial      |
| PUT    | `/api/tutorials/:id`       | Update tutorial          |
| DELETE | `/api/tutorials/:id`       | Delete tutorial          |
| DELETE | `/api/tutorials`           | Delete all tutorials     |
| GET    | `/api/tutorials/published` | Get published tutorials  |

### API Testing Examples

**Create a Tutorial**:
```bash
curl -X POST http://51.20.74.176/api/tutorials \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Docker Tutorial",
    "description": "Learn Docker containerization",
    "published": true
  }'
```

**Get All Tutorials**:
```bash
curl http://51.20.74.176/api/tutorials
```

**Update Tutorial**:
```bash
curl -X PUT http://51.20.74.176/api/tutorials/{id} \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Updated Docker Tutorial",
    "description": "Advanced Docker concepts",
    "published": true
  }'
```

**Delete Tutorial**:
```bash
curl -X DELETE http://51.20.74.176/api/tutorials/{id}
```

---

## 🐳 Docker Commands

### Local Development

**Start all services**:
```bash
docker compose up -d --build
```

**View logs**:
```bash
docker compose logs -f
```

**Stop services**:
```bash
docker compose down
```

**Remove everything (including volumes)**:
```bash
docker compose down -v
```

### Production (On AWS EC2)

**SSH into the VM**:
```bash
ssh -i your-key.pem ubuntu@51.20.74.176
```

**Pull latest images and restart**:
```bash
docker compose pull
docker compose up -d
```

**Check container status**:
```bash
docker compose ps
```

**View application logs**:
```bash
docker compose logs -f backend
docker compose logs -f frontend
docker compose logs -f mongodb
```

**Verify MongoDB data persistence**:
```bash
# Access MongoDB shell
docker compose exec mongodb mongosh -u root -p password123

# Inside MongoDB shell
use mean_db
db.tutorials.find()
```

**Check MongoDB volume**:
```bash
docker volume ls
docker volume inspect crud-dd-task-mean-app_mongodb_data
```

**Backup MongoDB data**:
```bash
docker compose exec mongodb mongodump --out=/data/backup -u root -p password123
docker cp $(docker compose ps -q mongodb):/data/backup ./mongodb-backup-$(date +%Y%m%d)
```

**Restore MongoDB data**:
```bash
docker cp ./mongodb-backup-20260223 $(docker compose ps -q mongodb):/data/restore
docker compose exec mongodb mongorestore /data/restore -u root -p password123
```

---

## 🔍 Verification & Testing

### 1. Check Service Health

```bash
# On AWS VM
docker compose ps

# Expected output:
# NAME            STATUS         PORTS
# mongodb         Up (healthy)   27017/tcp
# mean-backend    Up (healthy)   3000/tcp
# mean-frontend   Up (healthy)   80/tcp
# nginx-proxy     Up (healthy)   0.0.0.0:80->80/tcp
```

### 2. Test MongoDB Persistence

```bash
# Create a tutorial via API
curl -X POST http://51.20.74.176/api/tutorials \
  -H "Content-Type: application/json" \
  -d '{"title":"Test Persistence","description":"Testing MongoDB volume","published":true}'

# Restart all containers
docker compose restart

# Verify data still exists
curl http://51.20.74.176/api/tutorials
```

### 3. Verify Database Connection

```bash
# SSH into backend container
docker compose exec backend sh

# Test MongoDB connection
node -e "require('./app/config/db.config.js')"

# Check environment variables
echo $DB_HOST
echo $DB_NAME
```

---

## 🔐 Environment Variables

| Variable                      | Description                    | Default       |
|-------------------------------|--------------------------------|---------------|
| `MONGO_INITDB_ROOT_USERNAME`  | MongoDB root username          | root          |
| `MONGO_INITDB_ROOT_PASSWORD`  | MongoDB root password          | password123   |
| `MONGO_INITDB_DATABASE`       | Initial database name          | mean_db       |
| `DB_HOST`                     | MongoDB host                   | mongodb       |
| `DB_PORT`                     | MongoDB port                   | 27017         |
| `DB_NAME`                     | Database name                  | mean_db       |
| `NODE_ENV`                    | Node environment               | production    |

---

## 🎯 Key Features

### Multi-Stage Docker Builds
- **Frontend**: Builds Angular app in Node container, then serves with lightweight Nginx
- **Backend**: Optimized with layer caching and production dependencies only
- **Size Optimization**: Final images are minimal and production-ready

### Security Best Practices
- Non-root user in containers
- Environment-based secrets management
- Network isolation via Docker networks
- Health checks on all services

### Production-Ready Configuration
- Nginx reverse proxy with proper routing
- MongoDB persistent volumes for data safety
- Automated CI/CD with GitHub Actions
- AWS EC2 deployment with public accessibility

---

## 📄 License

This project is for educational purposes as part of a DevOps assignment.

## 👤 Author

**Thousif Ahamed**

---

**Happy Coding! 🚀**