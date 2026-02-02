### Most Common Everyday Examples

# 1.Nginx (web server)
```
# 1. Pull the latest official nginx image from Docker Hub
docker pull nginx

# 2. Run it (maps port 8080 on your computer → port 80 inside container)
docker run -d -p 8080:80 --name my-nginx nginx

# Now open browser → http://localhost:8080
# You should see the "Welcome to nginx!" page
```

## 2.Hello-world (the official Docker test image)
```
docker pull hello-world
docker run hello-world
```
## 3. PostgreSQL database - very typical real-world example
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

## 4.Redis (fast in-memory store)

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
### Quick Start - Popular Docker Images

### Popular One-liner Docker Commands 🚀

| Command                                      | What you get          | Try it                          | Emoji |
|:---------------------------------------------|:----------------------|:--------------------------------|:------|
| `docker run -d -p 80:80 nginx`               | Web Server            | http://localhost                | 🌐    |
| `docker run -it python:3.12`                 | Python Shell          | type python code                | 🐍    |
| `docker run -d -e POSTGRES_PASSWORD=pass postgres:16` | Database       | pgAdmin / `psql`                | 🐘    |
| `docker run -d redis:7-alpine`               | Key-Value Store       | `redis-cli`                     | ⚡    |
| `docker run -d -p 8080:8080 jenkins/jenkins` | CI/CD Server          | http://localhost:8080           | 🤖    |
```