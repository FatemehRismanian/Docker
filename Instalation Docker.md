# Installing Docker on Ubuntu

This guide shows how to install the current stable version of **Docker Engine** on Ubuntu

### 1. Uninstall old / conflicting versions (if any)

```bash
sudo apt-get remove -y docker docker-engine docker.io containerd runc
```
### 2. Install required packages
```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg lsb-release
```
### 3. Add Docker's official GPG key
```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
  ```
### 4. Add Docker repository
``` bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  ```
### 5. Install Docker Engine + docker-compose-plugin
``` bash
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
### 6. Verify installation
``` bash
# Version check
docker --version
docker compose version
docker ps
```
### 7. (Recommended) Add your user to the docker group
``` bash
sudo usermod -aG docker $USER
newgrp docker
# or simply log out and log back in
```

### Optional – Enable Docker to start on boot
``` bash
sudo systemctl enable docker
sudo systemctl enable containerd
```
Good commands to diagnose problems:
``` bash
docker info
docker version --format '{{.Server.Version}}'
systemctl status docker
journalctl -u docker -n 200 --no-pager
systemctl restart docker
```
That's it! 🐳
You now have up-to-date Docker + Compose on Ubuntu.

Good luck with the repository! 🚀
