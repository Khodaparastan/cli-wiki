
## Table of Contents
1. [Docker Basics](#docker-basics)
2. [Image Management](#image-management)
3. [Container Management](#container-management)
4. [Network Management](#network-management)
5. [Volume Management](#volume-management)
6. [Docker Compose](#docker-compose)
7. [Best Practices](#best-practices)
8. [Troubleshooting](#troubleshooting)

## Docker Basics

### Installation & Configuration
```bash
# Install Docker (Ubuntu)
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Post-installation steps
sudo usermod -aG docker $USER
sudo systemctl enable docker
sudo systemctl start docker

# Configure Docker daemon
cat <<EOF | sudo tee /etc/docker/daemon.json
{
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "10m",
        "max-file": "3"
    },
    "default-address-pools": [
        {
            "base": "172.17.0.0/16",
            "size": 24
        }
    ]
}
EOF
```

### Basic Commands
```bash
# Version and info
docker version
docker info
docker system info

# Login to registry
docker login [registry-url]

# System cleanup
docker system prune -a --volumes
docker system df
```

## Image Management

### Building Images
```bash
# Basic build
docker build -t image-name:tag .

# Build with arguments
docker build \
    --build-arg ARG_NAME=value \
    --build-arg HTTP_PROXY=$HTTP_PROXY \
    -t image-name:tag .

# Multi-stage build
cat <<EOF > Dockerfile
# Build stage
FROM node:14 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
EOF

# Build with specific target
docker build --target builder -t build-image .
```

### Image Operations
```bash
# List images
docker images
docker image ls --format "table {{.ID}}\t{{.Repository}}\t{{.Size}}"

# Pull images
docker pull image:tag
docker pull --all-tags image

# Tag images
docker tag source-image:tag target-image:tag

# Save and load images
docker save image:tag > image.tar
docker load < image.tar

# Inspect image
docker image inspect image:tag
docker history image:tag

# Remove images
docker rmi image:tag
docker image prune -a --filter "until=24h"
```

## Container Management

### Running Containers
```bash
# Basic run
docker run -d \
    --name container-name \
    -p 8080:80 \
    image:tag

# Run with advanced options
docker run -d \
    --name container-name \
    --restart unless-stopped \
    --memory="512m" \
    --cpus="0.5" \
    -e ENV_VAR=value \
    -v /host/path:/container/path \
    --network network-name \
    --health-cmd="curl -f http://localhost/ || exit 1" \
    --health-interval=5m \
    image:tag

# Run with specific user/group
docker run -d \
    --user $(id -u):$(id -g) \
    image:tag

# Run interactive shell
docker run -it \
    --rm \
    image:tag /bin/bash
```

### Container Operations
```bash
# List containers
docker ps
docker ps -a --format "table {{.ID}}\t{{.Names}}\t{{.Status}}"

# Container lifecycle
docker start container-name
docker stop container-name
docker restart container-name
docker pause container-name
docker unpause container-name

# Execute commands
docker exec -it container-name /bin/bash
docker exec container-name env

# Copy files
docker cp container-name:/path/file.txt ./local/path/
docker cp ./local/file.txt container-name:/path/

# View logs
docker logs -f container-name
docker logs --since 5m container-name
docker logs --tail 100 container-name

# Monitor containers
docker stats
docker top container-name
```

## Network Management

### Network Operations
```bash
# Create network
docker network create \
    --driver bridge \
    --subnet 172.18.0.0/16 \
    --gateway 172.18.0.1 \
    network-name

# List networks
docker network ls
docker network inspect network-name

# Connect/disconnect containers
docker network connect network-name container-name
docker network disconnect network-name container-name

# Network troubleshooting
docker run -it --rm \
    --network container:target-container \
    nicolaka/netshoot
```

[Continue to Part 2...]

Would you like me to continue with Volume Management, Docker Compose, Best Practices, and the Quick Reference Cheatsheet?