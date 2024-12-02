
## Volume Management

### Volume Operations
```bash
# Create volume
docker volume create \
    --driver local \
    --opt type=none \
    --opt device=/host/path \
    --opt o=bind \
    volume-name

# List and inspect volumes
docker volume ls
docker volume inspect volume-name

# Cleanup volumes
docker volume prune
docker volume rm volume-name

# Backup volume
docker run --rm \
    -v volume-name:/source \
    -v $(pwd):/backup \
    alpine tar czf /backup/volume-backup.tar.gz -C /source .

# Restore volume
docker run --rm \
    -v volume-name:/target \
    -v $(pwd):/backup \
    alpine tar xzf /backup/volume-backup.tar.gz -C /target
```

## Docker Compose

### Basic Compose Configuration
```yaml
# docker-compose.yml
version: '3.8'

services:
  web:
    build:
      context: ./web
      dockerfile: Dockerfile
      args:
        - BUILD_ENV=production
    image: webapp:latest
    container_name: webapp
    ports:
      - "80:80"
    environment:
      - NODE_ENV=production
      - API_URL=http://api:3000
    volumes:
      - ./web:/app
      - /app/node_modules
    depends_on:
      - api
      - db
    networks:
      - frontend
      - backend
    deploy:
      replicas: 2
      resources:
        limits:
          cpus: '0.5'
          memory: 512M

  api:
    image: api:latest
    container_name: api
    environment:
      - DB_HOST=db
      - DB_USER=${DB_USER}
      - DB_PASS=${DB_PASS}
    networks:
      - backend
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  db:
    image: postgres:13
    container_name: db
    volumes:
      - db-data:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=${DB_USER}
      - POSTGRES_PASSWORD=${DB_PASS}
    networks:
      - backend

volumes:
  db-data:

networks:
  frontend:
  backend:
    internal: true
```

### Compose Commands
```bash
# Start services
docker-compose up -d
docker-compose up -d --scale web=3

# Stop services
docker-compose down
docker-compose down -v --remove-orphans

# View logs
docker-compose logs -f
docker-compose logs -f service-name

# Execute commands
docker-compose exec service-name command
docker-compose run --rm service-name command

# View status
docker-compose ps
docker-compose top

# Build services
docker-compose build
docker-compose build --no-cache service-name
```

### Advanced Compose Features
```yaml
# docker-compose.override.yml
version: '3.8'

services:
  web:
    build:
      target: development
    volumes:
      - ./web:/app
    environment:
      - DEBUG=true

  db:
    ports:
      - "5432:5432"
    volumes:
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
```

## Best Practices

### Dockerfile Best Practices
```dockerfile
# Use multi-stage builds
FROM node:14 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
RUN chown -R nginx:nginx /usr/share/nginx/html
USER nginx
EXPOSE 80
HEALTHCHECK --interval=30s --timeout=3s \
    CMD wget -q --spider http://localhost/ || exit 1

# Layer optimization
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
RUN useradd -r appuser && \
    chown -R appuser /app
USER appuser
CMD ["python", "app.py"]
```

### Security Best Practices
```bash
# Scan images
docker scan image:tag

# Run with security options
docker run -d \
    --security-opt no-new-privileges \
    --cap-drop ALL \
    --cap-add NET_BIND_SERVICE \
    image:tag

# Use secrets
echo "secret_value" | docker secret create my_secret -

# Use read-only root filesystem
docker run -d \
    --read-only \
    --tmpfs /tmp \
    image:tag
```

## Quick Reference Cheatsheet

### Essential Docker Commands
```bash
# Container Management
docker run -d -p 80:80 --name web nginx
docker ps -a
docker start/stop/restart container
docker exec -it container bash
docker logs -f container

# Image Management
docker build -t image:tag .
docker pull/push image:tag
docker rmi image:tag
docker save/load image:tag

# Cleanup
docker system prune -af
docker volume prune
docker network prune
```

### Essential Docker Compose Commands
```bash
# Basic Operations
docker-compose up -d
docker-compose down
docker-compose ps
docker-compose logs -f

# Maintenance
docker-compose build
docker-compose pull
docker-compose restart
docker-compose exec service command
```

### Common Docker Run Options
```bash
# Port mapping
-p 8080:80

# Volume mounting
-v /host:/container

# Environment variables
-e KEY=VALUE

# Network
--network network-name

# Resources
--memory="512m"
--cpus="0.5"

# Restart policy
--restart unless-stopped
```

### Useful Docker Compose Snippets
```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "80:80"
    volumes:
      - .:/app
    environment:
      - NODE_ENV=development
    depends_on:
      - db
    networks:
      - app-network

networks:
  app-network:
    driver: bridge
```

Remember:
- Always use specific tags for images
- Implement proper logging
- Use multi-stage builds
- Implement health checks
- Regular security scanning
- Proper resource limits
- Documentation of configurations
- Regular cleanup of unused resources

Would you like me to expand on any particular aspect or add more examples?