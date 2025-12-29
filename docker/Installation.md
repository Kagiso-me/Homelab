# 🐳 Docker Installation Guide (Ubuntu / Debian)

This repository provides a **simple, reliable guide** to installing Docker Engine and Docker Compose (v2)
using the **official Docker repository**.

Tested for:
- Ubuntu 20.04+
- Debian 11+

---

## 🚀 What This Installs

- Docker Engine (docker-ce)
- Docker CLI
- containerd
- Docker Buildx
- Docker Compose v2 (plugin)

---

## 1️⃣ Remove old Docker packages (recommended)

```bash
sudo apt remove -y docker docker-engine docker.io containerd runc
```

---

## 2️⃣ Update system and install prerequisites

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg lsb-release
```

---

## 3️⃣ Add Docker’s official GPG key

```bash
sudo install -m 0755 -d /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

> For Debian systems, replace `ubuntu` with `debian` in the URL above.

---

## 4️⃣ Add the Docker repository

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

---

## 5️⃣ Install Docker Engine and Compose plugin

```bash
sudo apt update
sudo apt install -y \
  docker-ce \
  docker-ce-cli \
  containerd.io \
  docker-buildx-plugin \
  docker-compose-plugin
```

---

## 6️⃣ Enable and start Docker

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

Verify:
```bash
systemctl status docker
```

---

## 7️⃣ Allow Docker without sudo (IMPORTANT)

```bash
sudo usermod -aG docker $USER
```

Then **log out and log back in**, or run:
```bash
newgrp docker
```

---

## 8️⃣ Verify installation

```bash
docker version
docker run hello-world
```

If you see **“Hello from Docker!”**, Docker is working correctly.

---

## 9️⃣ Verify Docker Compose (v2)

```bash
docker compose version
```

> Docker Compose v2 uses `docker compose`  
> (`docker-compose` is deprecated)

---

## 🔎 Optional Test: Run a real container

```bash
docker run -d -p 8080:80 --name nginx-test nginx
```

Open in browser:
```
http://<server-ip>:8080
```

Cleanup:
```bash
docker rm -f nginx-test
```

---

## 📝 Notes

- Re-login is required after adding your user to the `docker` group
- Docker data is stored in `/var/lib/docker`
- This install method is preferred over distro packages

---

## ✅ Done

Docker is now installed and ready to use.
