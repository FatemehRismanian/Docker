### Most Common Everyday Examples

## 1.Hello-world (the official Docker test image)
```
docker pull hello-world
docker run hello-world
```
## 2.Nginx (web server)
```
# 1. Pull the latest official nginx image from Docker Hub
docker pull nginx

# 2. Run it (maps port 8080 on your computer → port 80 inside container)
docker run -d -p 8080:80 --name my-nginx nginx

# Now open browser → http://localhost:8080
# You should see the "Welcome to nginx!" page
```
## 3. Traefik (Web / reverse proxy servers) 
```
Traefik v3 (modern reverse proxy & load balancer, auto Let's Encrypt)
docker pull traefik:v3.1
docker run -d -p 80:80 -p 8080:8080 \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  --name traefik traefik:v3.1 --api.insecure=true --providers.docker
```

## 4. Caddy (automatic HTTPS, very simple config)
```
docker pull caddy:2
docker run -d -p 80:80 -p 443:443 --name caddy caddy:2
```

## 5. PostgreSQL database - very typical real-world example
```
# Pull a specific version (recommended in production)
docker pull postgres:16

# Run PostgreSQL with password and persistent data
docker run -d \
  --name my-postgres \
  -e POSTGRES_PASSWORD=mysecretpassword \
  -p 5432:5432 \
  -v pgdata:/var/lib/postgresql/data \
  postgres:16
```

## 6.Redis (fast in-memory store)

```
docker pull redis:7-alpine

docker run -d \
  --name my-redis \
  -p 6379:6379 \
  redis:7-alpine
```

## One-liner style (very popular)
Many people combine pull + run in one command (docker automatically pulls if image doesn't exist):
```
# Nginx - most common one-liner
docker run -d -p 8080:80 nginx

# PostgreSQL one-liner
docker run -d -p 5432:5432 -e POSTGRES_PASSWORD=secret123 postgres

# MySQL
docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=my-secret-pw mysql:8

# MongoDB
docker run -d -p 27017:27017 mongo
```

### Quick reference – most popular images on Docker Hub

```
### Quick Start - Popular One-liner Docker Commands 🚀

| Command                                               | What you get          | Try it                          | Emoji |
|:------------------------------------------------------|:----------------------|:--------------------------------|:------|
| `docker run -d -p 80:80 nginx`                        | Web Server            | http://localhost                | 🌐    |
| `docker run -it python:3.12`                          | Python Shell          | type python code                | 🐍    |
| `docker run -d -e POSTGRES_PASSWORD=pass postgres:16` | Database              | pgAdmin / `psql`                | 🐘    |
| `docker run -d redis:7-alpine`                        | Key-Value Store       | `redis-cli`                     | ⚡    |
| `docker run -d -p 8080:8080 jenkins/jenkins`          | CI/CD Server          | http://localhost:8080           | 🤖    |

```

## 7.Lightweight base OS images (very popular for custom builds)
```
# Alpine – tiny (~5 MB), great for production microservices
docker pull alpine:latest
docker run -it --rm alpine:latest sh

# Ubuntu 24.04 LTS – very common dev/test base
docker pull ubuntu:24.04
docker run -it --rm ubuntu:24.04 bash

# Debian (stable & bookworm is still widely used)
docker pull debian:bookworm-slim
docker run -it --rm debian:bookworm-slim bash
```
## 8. Development / playground images
```
# Go (Golang) – official, great for quick experiments
docker pull golang:1.23-alpine
docker run -it --rm -v $(pwd):/go/src/app -w /go/src/app golang:1.23-alpine go version

# Rust – popular in 2025–2026 for performance-critical apps
docker pull rust:1.84-slim
docker run -it --rm -v $(pwd):/usr/src/myapp -w /usr/src/myapp rust:1.84-slim cargo --version

# Node 22 
docker pull node:22-alpine
docker run -it --rm node:22-alpine node -v
```
