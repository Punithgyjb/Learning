# Mastering Docker and Docker Compose

## Docker Basics

Docker is a platform that uses containerization technology to package applications and their dependencies into standardized units called containers.

### Key Concepts

1.  **Containers**: Lightweight, standalone, executable packages that include everything needed to run an application
2.  **Images**: Read-only templates used to create containers
3.  **Dockerfile**: Text file containing instructions to build a Docker image
4.  **Registry**: Storage for Docker images (like Docker Hub)

### Essential Docker Commands

Let's start with the most fundamental commands:

```bash
# Check Docker version
docker --version  
Docker version 28.1.1, build 4eba377

# Download an image from Docker Hub
docker pull ubuntu

# List downloaded images
docker images

# Run a container
docker run ubuntu

# Run container interactively
docker run -it ubuntu bash

# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Stop a container
docker stop container_id

# Remove a container
docker rm container_id

# Remove an image
docker rmi image_name

```

### Creating Your First Dockerfile

Let's create a simple Dockerfile for a Node.js application:

```Dockerfile
FROM node:14
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "app.js"]

```

To build and run this:

```bash
# Build image
docker build -t myapp .

# Run container
docker run -p 3000:3000 myapp

```

## Docker Compose Basics

Docker Compose is a tool for defining and running multi-container Docker applications.

### Key Concepts

1.  **docker-compose.yml**: Configuration file that defines services, networks, and volumes
2.  **Services**: Containers that make up your application
3.  **Networks**: How containers communicate with each other
4.  **Volumes**: Persistent data storage

### Simple docker-compose.yml Example

```yaml
version: '3'
services:
  web:
    build: .
    ports:
      - "3000:3000"
  db:
    image: mongo
    volumes:
      - mongo-data:/data/db
volumes:
  mongo-data:

```

### Essential Docker Compose Commands

```bash
# Start services
docker-compose up

# Start in detached mode
docker-compose up -d

# Stop services
docker-compose down

# View logs
docker-compose logs

# Execute command in running service
docker-compose exec web bash

```
# Advanced Docker Concepts

Let's dive deeper into more advanced Docker and Docker Compose topics.

## Advanced Docker Networking

Docker provides several network drivers to accommodate different use cases:

```bash
# Create a custom bridge network
docker network create --driver bridge my_network

# List networks
docker network ls

# Connect container to network
docker network connect my_network container_name

# Run container with specific network
docker run --network=my_network nginx

# Inspect network
docker network inspect my_network

```

### Network Types

1.  **Bridge**: Default network driver that connects containers on the same Docker host
2.  **Host**: Container shares the host's networking namespace
3.  **Overlay**: Connects multiple Docker daemons for Swarm services
4.  **Macvlan**: Assigns a MAC address to containers, making them appear as physical devices
5.  **None**: Disables networking

## Docker Volumes and Data Management

```bash
# Create a named volume
docker volume create my_data

# List volumes
docker volume ls

# Mount volume to container
docker run -v my_data:/app/data nginx

# Use bind mounts (host directory)
docker run -v $(pwd):/app nginx

# Read-only mounts
docker run -v my_data:/app/data:ro nginx

# Use tmpfs mount (memory-only storage)
docker run --tmpfs /app/temp nginx

```

## Advanced Dockerfile Techniques

### Multi-stage Builds

Multi-stage builds allow you to use multiple FROM statements in your Dockerfile, each creating a new build stage.

```Dockerfile
# Build stage
FROM golang:1.16 AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp

# Final stage
FROM alpine:latest
WORKDIR /root/
COPY --from=builder /app/myapp .
CMD ["./myapp"]

```

### BuildKit and BuildX

BuildKit is Docker's next-generation build system offering improved performance, storage management, and security:

```bash
# Enable BuildKit
export DOCKER_BUILDKIT=1

# Use BuildX (cross-platform builds)
docker buildx create --name mybuilder
docker buildx use mybuilder
docker buildx build --platform linux/amd64,linux/arm64 -t myapp:latest .

```

### ARG and ENV

```Dockerfile
# Build arguments
ARG VERSION=latest

# Environment variables
ENV NODE_ENV=production

# Using build args
FROM node:${VERSION}

```

## Docker Security Best Practices

```Dockerfile
# Use specific versions instead of latest
FROM node:16.14.0-alpine

# Run as non-root user
RUN adduser -D appuser
USER appuser

# Scan images for vulnerabilities
# Command: docker scan myimage

```

# Advanced Docker Compose

## Environment Variables and Configuration

```yaml
version: '3'
services:
  web:
    build: .
    environment:
      - NODE_ENV=production
      - DB_HOST=db
    env_file:
      - .env.production

```

## Scaling Services

```bash
# Scale a service to multiple instances
docker-compose up -d --scale web=3

```

## Extending Compose Files

```yaml
# docker-compose.base.yml
version: '3'
services:
  web:
    image: nginx
    
# docker-compose.dev.yml
version: '3'
services:
  web:
    volumes:
      - ./src:/var/www/html

```

```bash
# Combine multiple compose files
docker-compose -f docker-compose.base.yml -f docker-compose.dev.yml up

```

## Health Checks

```yaml
version: '3'
services:
  web:
    image: nginx
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

```

## Docker Compose Profiles

```yaml
version: '3.9'
services:
  app:
    image: myapp
  db:
    image: postgres
    profiles: ["dev"]
  test:
    image: myapp-test
    profiles: ["test"]

```

```bash
# Start only services without profiles
docker-compose up

# Start specific profile
docker-compose --profile dev up

```

# Docker Swarm and Orchestration

Docker Swarm is Docker's native clustering and orchestration solution that turns a group of Docker hosts into a single virtual Docker host.

## Docker Swarm Basics

### Initializing a Swarm

```bash
# Initialize a new swarm (on the manager node)
docker swarm init --advertise-addr <MANAGER-IP>

# The command outputs a token for workers to join
# Example output: docker swarm join --token SWMTKN-1-49nj1cmql0... <MANAGER-IP>:2377

```

### Adding Nodes to the Swarm

```bash
# Join as a worker node (run on worker machines)
docker swarm join --token <TOKEN> <MANAGER-IP>:2377

# Join as a manager node
docker swarm join-token manager # Get manager token
docker swarm join --token <MANAGER-TOKEN> <MANAGER-IP>:2377

# List nodes in the swarm
docker node ls

```

### Node Management

```bash
# Promote a worker to manager
docker node promote <NODE-ID>

# Demote a manager to worker
docker node demote <NODE-ID>

# Update node availability
docker node update --availability drain <NODE-ID>  # Stop scheduling new tasks
docker node update --availability active <NODE-ID> # Resume scheduling tasks

# Remove a node from the swarm
docker node rm <NODE-ID>

```

## Services in Docker Swarm

Services are the primary way to run containers in a swarm.

```bash
# Create a service with 3 replicas
docker service create --name webapp --replicas 3 -p 80:80 nginx

# List services
docker service ls

# Service details and status
docker service ps webapp

# Scale a service
docker service scale webapp=5

# Update service (rolling update)
docker service update --image nginx:1.19 webapp

# Remove a service
docker service rm webapp

```

### Advanced Service Configuration

```bash
# Global service (one container per node)
docker service create --name monitoring --mode global grafana/grafana

# Service with resource constraints
docker service create --name webapp \
  --limit-cpu 0.5 \
  --limit-memory 512M \
  --reserve-cpu 0.25 \
  --reserve-memory 256M \
  nginx

# Update policy for rolling updates
docker service create --name webapp \
  --update-parallelism 2 \
  --update-delay 10s \
  --update-failure-action rollback \
  nginx

```

## Working with Stacks in Docker Swarm

Stacks are groups of interrelated services that share dependencies and can be orchestrated and scaled together. They're defined using Docker Compose files.

### Stack Deployment

```yaml
# docker-compose.yml for swarm deployment
version: '3.8'
services:
  web:
    image: nginx
    deploy:
      replicas: 3
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure
    ports:
      - "80:80"
  db:
    image: postgres
    deploy:
      placement:
        constraints:
          - node.role == worker
    volumes:
      - db-data:/var/lib/postgresql/data
volumes:
  db-data:

```

```bash
# Deploy a stack
docker stack deploy -c docker-compose.yml myapp

# List stacks
docker stack ls

# List services in a stack
docker stack services myapp

# List tasks in a stack
docker stack ps myapp

# Remove a stack
docker stack rm myapp

```

## Secrets Management in Swarm

Docker Swarm provides built-in secrets management for sensitive data.

```bash
# Create a secret from a file
echo "mysecretpassword" > password.txt
docker secret create db_password password.txt
rm password.txt  # Remove the file for security

# Create a secret directly
echo "mysecretpassword" | docker secret create db_password -

# List secrets
docker secret ls

# Use secrets in services
docker service create \
  --name db \
  --secret db_password \
  -e POSTGRES_PASSWORD_FILE=/run/secrets/db_password \
  postgres

```

In a compose file:

```yaml
version: '3.8'
services:
  db:
    image: postgres
    secrets:
      - db_password
    environment:
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password
secrets:
  db_password:
    external: true

```

## Swarm Networking

```bash
# Create an overlay network
docker network create --driver overlay my-network

# Create service with specific network
docker service create --name webapp --network my-network nginx

# Service discovery is automatic by service name

```

## Health Checks and Self-Healing

Docker Swarm constantly monitors containers and automatically recreates them if they fail health checks.

```yaml
version: '3.8'
services:
  web:
    image: nginx
    deploy:
      replicas: 3
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

```
# Advanced Docker Orchestration, Monitoring, and CI/CD Integration

## Advanced Docker Swarm Features

### Swarm Routing Mesh

Docker Swarm implements a routing mesh that allows any node in the swarm to accept connections on published ports for any service, even if there's no task for that service running on the node.

```bash
# Create a service with published ports
docker service create --name web --publish published=8080,target=80 --replicas 2 nginx

```

The service is accessible on port 8080 from any node in the swarm, regardless of which nodes are running the containers.

### Configuration and Tuning

```bash
# Change swarm default settings
docker swarm update --task-history-limit 5 --dispatcher-heartbeat 10s

# Set custom labels for nodes
docker node update --label-add zone=east node1

```

### Node Constraints and Placement Preferences

```yaml
version: '3.8'
services:
  db:
    image: postgres
    deploy:
      placement:
        constraints:
          - node.labels.zone == east
          - node.role == worker
        preferences:
          - spread: node.labels.datacenter

```

## Monitoring and Logging Solutions

### Docker Stats and Basic Monitoring

```bash
# View real-time container stats
docker stats

# View container logs
docker logs container_id

# Follow logs in real-time
docker logs -f container_id

```

### Setting Up Prometheus and Grafana for Monitoring

```yaml
version: '3.8'
services:
  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
    deploy:
      placement:
        constraints:
          - node.role == manager

  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana-storage:/var/lib/grafana
    depends_on:
      - prometheus

  node-exporter:
    image: prom/node-exporter
    deploy:
      mode: global
    ports:
      - "9100:9100"

  cadvisor:
    image: gcr.io/cadvisor/cadvisor
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
    ports:
      - "8080:8080"
    deploy:
      mode: global

volumes:
  grafana-storage:

```

Example prometheus.yml configuration:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node-exporter'
    dns_sd_configs:
      - names: ['tasks.node-exporter']
        type: 'A'
        port: 9100

  - job_name: 'cadvisor'
    dns_sd_configs:
      - names: ['tasks.cadvisor']
        type: 'A'
        port: 8080

```

### ELK Stack for Log Management

```yaml
version: '3.8'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.13.0
    environment:
      - discovery.type=single-node
    ports:
      - "9200:9200"
    volumes:
      - esdata:/usr/share/elasticsearch/data

  logstash:
    image: docker.elastic.co/logstash/logstash:7.13.0
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    depends_on:
      - elasticsearch

  kibana:
    image: docker.elastic.co/kibana/kibana:7.13.0
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch

  filebeat:
    image: docker.elastic.co/beats/filebeat:7.13.0
    user: root
    volumes:
      - ./filebeat.yml:/usr/share/filebeat/filebeat.yml:ro
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
    deploy:
      mode: global

volumes:
  esdata:

```

## CI/CD Integration with Docker

### GitLab CI/CD with Docker

Example `.gitlab-ci.yml`:

```yaml
stages:
  - build
  - test
  - deploy

variables:
  DOCKER_DRIVER: overlay2
  DOCKER_TLS_CERTDIR: ""

build:
  stage: build
  image: docker:20.10
  services:
    - docker:20.10-dind
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA

test:
  stage: test
  image: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  script:
    - npm test

deploy:
  stage: deploy
  image: docker:20.10
  services:
    - docker:20.10-dind
  script:
    - docker stack deploy -c docker-compose.prod.yml myapp
  only:
    - main

```

### GitHub Actions with Docker

Example `.github/workflows/main.yml`:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v1
      
      - name: Login to DockerHub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      
      - name: Build and push
        uses: docker/build-push-action@v2
        with:
          context: .
          push: true
          tags: username/myapp:latest

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to Production
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          key: ${{ secrets.SSH_KEY }}
          script: |
            docker stack deploy -c docker-compose.prod.yml myapp

```

## Advanced Docker Optimization

### Image Optimization Techniques

```Dockerfile
# Use multi-stage builds
FROM node:16-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html

# Layer optimization
FROM python:3.9-slim

WORKDIR /app

# Copy and install requirements first (takes advantage of layer caching)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code after dependencies are installed
COPY . .

# Use .dockerignore file to prevent unnecessary files from being copied

```

### Docker Image Security Scanning

```bash
# Using Trivy scanner
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image myapp:latest

# Using Snyk
docker scan myapp:latest

```

## Production Readiness and Best Practices

### Docker in Production Checklist

1.  **Security**:
    
    -   Run containers with minimal privileges
    -   Use read-only file systems where possible
    -   Implement content trust with `DOCKER_CONTENT_TRUST=1`
    -   Regular security scanning of images
2.  **Reliability**:
    
    -   Implement health checks
    -   Use restart policies
    -   Set resource limits
    -   Properly handle signals for graceful shutdown
3.  **Performance**:
    
    -   Optimize image size
    -   Use appropriate storage drivers
    -   Configure logging properly to avoid disk fill-up
4.  **Monitoring**:
    
    -   Implement comprehensive monitoring (CPU, memory, disk I/O)
    -   Set up alerts for container health
    -   Collect and centralize logs

```yaml
version: '3.8'
services:
  app:
    image: myapp:latest
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 512M
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
        window: 120s
      update_config:
        parallelism: 2
        delay: 10s
        order: start-first
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    read_only: true
    tmpfs:
      - /tmp
      - /var/run
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

```

### Container Runtime Security

```bash
# Run with security options
docker run --security-opt no-new-privileges --cap-drop ALL --cap-add NET_BIND_SERVICE nginx

# Use user namespaces
docker run --userns-remap=default nginx

```

## Docker Networking Advanced Topics

### Macvlan and IPvlan Networks

```bash
# Create macvlan network
docker network create -d macvlan \
  --subnet=192.168.50.0/24 \
  --gateway=192.168.50.1 \
  -o parent=eth0 \
  macvlan-net

# Create ipvlan network
docker network create -d ipvlan \
  --subnet=192.168.60.0/24 \
  --gateway=192.168.60.1 \
  -o parent=eth0 \
  ipvlan-net

```

### Custom Network Plugins

```bash
# Install network plugin
docker plugin install store/weaveworks/net-plugin:latest-release

# Create network with custom plugin
docker network create --driver weave mynetwork

```
