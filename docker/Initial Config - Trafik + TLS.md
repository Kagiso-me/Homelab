# Docker + Traefik + Portainer — Local-Only HTTPS Cheatsheet

This is a **quick, practical README** for setting up a Docker host with:
- Basic OS hardening
- Docker
- Traefik reverse proxy
- Portainer behind Traefik
- Local-only access (no public exposure)
- No Cloudflare / public ACME

---

## 1. Base OS Hardening

```bash
sudo apt update && sudo apt upgrade -y
sudo adduser kagiso
sudo usermod -aG sudo kagiso
```

### SSH keys
```bash
ssh-keygen -t ed25519
ssh-copy-id kagiso@SERVER_IP
```

Edit `/etc/ssh/sshd_config`:
```
PermitRootLogin no
PasswordAuthentication no
```

```bash
sudo systemctl restart ssh
```

### Fail2Ban
```bash
sudo apt install fail2ban -y
sudo systemctl enable --now fail2ban
```

---

## 2. Install Docker

```bash
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker kagiso
newgrp docker
docker version
```

---

## 3. Traefik

```bash
sudo mkdir -p /opt/traefik/{dynamic,letsencrypt}
sudo touch /opt/traefik/letsencrypt/acme.json
sudo chmod 600 /opt/traefik/letsencrypt/acme.json
docker network create traefik
```

```bash
docker run -d \
  --name traefik \
  --restart=unless-stopped \
  --network traefik \
  -p 80:80 \
  -p 443:443 \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  -v /opt/traefik/dynamic:/etc/traefik/dynamic \
  -v /opt/traefik/letsencrypt:/letsencrypt \
  traefik:v3.0 \
  --providers.docker=true \
  --providers.docker.exposedbydefault=false \
  --providers.file.directory=/etc/traefik/dynamic \
  --providers.file.watch=true \
  --entrypoints.web.address=:80 \
  --entrypoints.websecure.address=:443 \
  --api.dashboard=true
```

---

## 4. Traefik Dashboard (Local)

Create `/opt/traefik/dynamic/dashboard.yml`:
```yaml
http:
  routers:
    traefik-dashboard:
      rule: Host(`traefik.local`)
      entryPoints:
        - websecure
      service: api@internal
      tls: {}
```

```bash
docker restart traefik
```

---

## 5. Portainer (Behind Traefik)

```bash
docker volume create portainer_data
```

```bash
docker run -d \
  --name portainer \
  --restart=unless-stopped \
  --network traefik \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  -l "traefik.enable=true" \
  -l "traefik.http.routers.portainer.rule=Host(`portainer.local`)" \
  -l "traefik.http.routers.portainer.entrypoints=websecure" \
  -l "traefik.http.routers.portainer.tls=true" \
  -l "traefik.http.services.portainer.loadbalancer.server.port=9000" \
  portainer/portainer-ce:latest
```

---

## 6. Local DNS

On your local machine:
```
SERVER_IP traefik.local
SERVER_IP portainer.local
```

---

## Result

- https://traefik.local
- https://portainer.local
- No admin ports exposed
- Local-only access
