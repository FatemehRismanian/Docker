These are the most frequently used patterns for volumes in Docker.

Here are several practical samples of how to create and use Docker volumes, from basic to more realistic scenarios.

## 1. Basic named volume (most common & recommended)
```
# Create a named volume explicitly
docker volume create mydata

# Check it exists
docker volume ls
# → shows: local   mydata

# Use it in a container (read-write by default)
docker run -it --rm \
  -v mydata:/app/data \
  busybox sh
```

Inside the container:
```
echo "Hello from container" > /app/data/hello.txt
ls /app/data
cat /app/data/hello.txt
exit
```
Run another container to see persistence:
```
docker run -it --rm -v mydata:/app/data busybox ls -la /app/data
# → hello.txt is still there
```
## 2. Let Docker auto-create the volume on first use (very common)
No docker volume create needed:
```
docker run -d --name mysql-test \
  -v mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  mysql:8
```
Docker creates volume mysql-data automatically.
Data survives even if you docker rm mysql-test and start a new one with the same volume.

## 3. Read-only volume mount example
```
docker volume create static-files

docker run -d -p 8080:80 \
  -v static-files:/usr/share/nginx/html:ro \
  nginx:alpine
```

Nginx serves files read-only from the volume (good for shared config/static assets).

## 4. Multiple volumes in one container
```
docker volume create logs
docker volume create config

docker run -it --rm \
  -v logs:/var/log/app \
  -v config:/etc/app:ro \
  alpine sh
```

Inside:
```
touch /var/log/app/debug.log       # works
echo "setting=prod" > /etc/app/config.ini   # fails → Read-only file system
```
## 5. Simple docker-compose.yml sample with volumes
```
version: "3.9"

services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - html-content:/usr/share/nginx/html:ro     # read-only site files
      - ./custom.conf:/etc/nginx/conf.d/default.conf:ro   # bind mount for config

  logger:
    image: busybox
    command: sh -c "while true; do echo \"$(date)\" >> /logs/access.log; sleep 5; done"
    volumes:
      - logs:/logs

volumes:
  html-content:     # named volume
  logs:             # another named volume
  ```
Run → docker compose up -d
Two services share persistent storage via named volumes.

### 6. Create and use an NFS-backed Docker volume.(practical)
Docker supports mounting NFS shares as volumes using the built-in local driver.

Prerequisites:
An NFS server is already running and exporting a share
Your Docker host has NFS client support installed (nfs-common on Debian/Ubuntu, nfs-utils on CentOS/RHEL/Fedora)
The Docker host can reach the NFS server (firewall, network, exports allowed)
Know the NFS server IP and exported path (e.g. 192.168.50.10:/exports/data)

Step 1 – Create the NFS volume
```
#Replace with your real values
NFS_SERVER="192.168.50.10"
EXPORT_PATH="/exports/shared"  # path exported on the NFS server
VOLUME_NAME="nfs-shared"

docker volume create ${VOLUME_NAME} \
  --driver local \
  --opt type=nfs \
  --opt o=addr=${NFS_SERVER},rw,nfsvers=4,soft,timeo=100 \
  --opt device=:${EXPORT_PATH}
Common useful mount options (o=...):

rw          → read-write (use ro if read-only needed)
nfsvers=4   → prefer NFSv4 (more reliable; omit for NFSv3)
soft        → don't hang forever if server disappears
timeo=100   → timeout in 1/10 seconds
retrans=5   → retry count
nolock      → sometimes needed for older servers
noatime     → performance tweak
```
Check it worked:
```
docker volume ls | grep nfs-shared

docker volume inspect nfs-shared
```
You should see Mountpoint pointing to a /var/lib/docker/volumes/... location, but it's actually an NFS mount.
Step 2 – Use the NFS volume in a container
Simple test with busybox:
```
docker run -it --rm \
  --name nfs-test \
  -v nfs-shared:/data \
  busybox sh
```
Inside the container:
```
#Should succeed if NFS is writable
echo "Hello from container" > /data/test.txt
cat /data/test.txt
ls -la /data

# Create more files / directories
mkdir /data/backups
touch /data/backups/important.db
Exit the container (exit) — the files remain on the NFS server.
```
Step 3 – Real-world example with docker-compose (recommended)
```
YAMLversion: "3.8"

services:
  app:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - nfs-shared:/usr/share/nginx/html:ro     # read-only web content from NFS

  writer:
    image: busybox
    command: >
      sh -c "while true; do
               echo \"$(date)\" >> /data/log.txt;
               sleep 10;
             done"
    volumes:
      - nfs-shared:/data

volumes:
  nfs-shared:
    driver: local
    driver_opts:
      type: nfs
      o: addr=192.168.50.10,rw,nfsvers=4,soft,timeo=100
      device: ":/exports/shared"
```
Start it:
```
docker compose up -d
```
Now:
Nginx serves static files directly from the NFS share (read-only)
The writer container appends logs to the same NFS location