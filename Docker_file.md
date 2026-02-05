#  Summary of Best Practices for Writing Dockerfiles
1- **Minimal Base Image.** Official images are vetted, updated, and often smaller. Avoid bloated bases like full OS images unless necessary.

2- **Use Multi-Stage Builds for Smaller Final Images.** Separate build-time dependencies from runtime, reducing image size by discarding build artifacts.

3- **Minimize Layers by Combining Commands.** Each Dockerfile instruction creates a layer; fewer layers mean faster builds and smaller images.

4- **Use .dockerignore to Exclude Unnecessary Files.** Prevents copying junk (e.g., Git files, logs) into the build context, speeding up builds and reducing image size.

5- **Copy Files as Late as Possible.** Docker caches layers; copying code early invalidates cache on changes. Copy dependencies first (e.g., package.json), then app code.

6- **Run as Non-Root User for Security.** Root privileges in containers can lead to host escapes; use a limited user.

7-**Define Environment Variables with ENV.** Makes configs injectable and portable. Use defaults where possible.

8-**Add Healthchecks for Reliability.** Allows orchestrators (e.g., Docker Compose, Kubernetes) to detect unhealthy containers and restart them.

9- **Add Labels for Metadata.** Provides info like version, maintainer; useful for automation and querying images

10- **Clean Up After Installations.** Removes caches and temp files to keep images small.


#### Here are several practical, real-world examples of well-written Dockerfiles using modern best practices.

### 1-  Multi-stage Build – Node.js + TypeScript (Production Optimized)
```
# Stage 1: Build
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Runtime (very small final image)
FROM node:20-alpine
WORKDIR /app

# Copy only what's needed for running
COPY --from=builder /app/package*.json ./
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules

# Non-root user
USER node
ENV NODE_ENV=production
EXPOSE 3000
CMD ["node", "dist/server.js"]
```
#### Build and Run & Test
```
cd /path/to/your/project
ls -la
# You should see (at minimum): Dockerfile, package.json (possibly package-lock.json), src/  (or other source folders)

docker build -t my-node-app:prod .
docker images

docker run -d \
  --name my-app \
  -p 3000:3000 \
  my-node-app:prod

#Test that it works Browser:
http://localhost:3000

# or via terminal:
curl http://localhost:3000

```
### 2- Python (FastAPI / Flask / Django)
```
# Use slim variant when possible
FROM python:3.11-slim

# Set environment variables
ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PIP_NO_CACHE_DIR=off \
    PIP_DISABLE_PIP_VERSION_CHECK=on

# Set working directory
WORKDIR /app

# Install system dependencies in one layer
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    gcc \
    && apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Copy and install Python dependencies first (great for caching)
COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Create non-root user (good security practice)
RUN useradd --create-home appuser
USER appuser

# Expose port
EXPOSE 8000

# Start the application (examples)
# FastAPI / Uvicorn
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]

# Flask / Gunicorn (uncomment if using Flask instead)
# CMD ["gunicorn", "--bind", "0.0.0.0:8000", "app:app"]
```
#### Minimal FastAPI example (main.py):
```
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"message": "Hello from Docker + FastAPI!"}

@app.get("/health")
def health_check():
    return {"status": "healthy"}
```
#### Build & Run & Test
```
cd /path/to/your/python-project
ls -la           # Should show at least: Dockerfile  requirements.txt  main.py  (or app.py)
docker build -t my-python-api:prod .
docker images

docker run -d \
  --name python-api \
  -p 8000:8000 \
  my-python-api:prod

# Basic endpoint
curl http://localhost:8000

# Health check (if you added it)
curl http://localhost:8000/health

# Basic endpoint
curl http://localhost:8000

# Health check (if you added it)
curl http://localhost:8000/health
```
### 3- Very Minimal Static Site (Nginx)
```
# Build stage (if you use something like Vite, Next.js static export, etc.)
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Final stage – just nginx
FROM nginx:alpine

# Copy static files
COPY --from=builder /app/dist /usr/share/nginx/html

# Copy custom nginx config (optional)
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Expose port
EXPOSE 80

# Start nginx
CMD ["nginx", "-g", "daemon off;"]

```
### Sample package.json
```
{
  "name": "my-static-site",
  "version": "1.0.0",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "devDependencies": {
    "vite": "^5.0.0"
  }
}
```
#### Build & Run & Test
```
docker build -t my-static-site .

# Basic run
docker run -d -p 8080:80 --name static-site my-static-site

# With volume for development (hot reload):
docker run -d -p 8080:80 \
  -v $(pwd)/dist:/usr/share/nginx/html \
  --name static-site my-static-site

# Production with restart policy:
docker run -d -p 80:80 \
  --restart unless-stopped \
  --name static-site-prod my-static-site

docker logs static-site
```
