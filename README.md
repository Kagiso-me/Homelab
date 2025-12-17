<div align="center">

<img src="https://github.com/user-attachments/assets/cba21e9d-1275-4c92-ab9b-365f31f35add" align="center" width="175px" height="175px"/>

### <img src="https://fonts.gstatic.com/s/e/notoemoji/latest/1f680/512.gif" alt="🚀" width="16" height="16"> My Home Operations Repository <img src="https://fonts.gstatic.com/s/e/notoemoji/latest/1f6a7/512.gif" alt="🚧" width="16" height="16">

_... managed with ArgoCD, Renovate, and GitHub Actions_ <img src="https://fonts.gstatic.com/s/e/notoemoji/latest/1f916/512.gif" alt="🤖" width="16" height="16">

</div>

<div align="center">

[![Discord](https://img.shields.io/discord/673534664354430999?style=for-the-badge&label&logo=discord&logoColor=white&color=blue)](https://discord.gg/home-operations)&nbsp;&nbsp;
[![Kubernetes](https://img.shields.io/endpoint?url=https%3A%2F%2Fkromgo.turbo.ac%2Fkubernetes_version&style=for-the-badge&logo=kubernetes&logoColor=white&color=blue&label=%20)](https://kubernetes.io)&nbsp;&nbsp;
[![Argo CD](https://img.shields.io/badge/GitOps-Argo%20CD-orange?style=for-the-badge&logo=argo&logoColor=white)](https://argo-cd.readthedocs.io/)&nbsp;&nbsp;
[![Renovate](https://img.shields.io/github/actions/workflow/status/kagiso-me/helm-charts/renovate.yaml?branch=main&label=&logo=renovatebot&style=for-the-badge&color=blue)](https://github.com/kagiso-me/helm-charts/actions/workflows/renovate.yaml) 

</div>

<div align="center">

[![Home-Internet](https://img.shields.io/endpoint?url=https%3A%2F%2Fstatus.k13.dev%2Fapi%2Fv1%2Fendpoints%2Fbuddy_ping%2Fhealth%2Fbadge.shields&style=for-the-badge&logo=ubiquiti&logoColor=white&label=Home%20Internet)](https://status.turbo.ac)&nbsp;&nbsp;
[![Status-Page](https://img.shields.io/endpoint?url=https%3A%2F%2Fstatus.k13.dev%2Fapi%2Fv1%2Fendpoints%2Fbuddy_status-page%2Fhealth%2Fbadge.shields&style=for-the-badge&logo=statuspage&logoColor=white&label=Status%20Page)](https://status.turbo.ac)&nbsp;&nbsp;
[![Alertmanager](https://img.shields.io/endpoint?url=https%3A%2F%2Fstatus.k13.dev%2Fapi%2Fv1%2Fendpoints%2Fbuddy_heartbeat%2Fhealth%2Fbadge.shields&style=for-the-badge&logo=prometheus&logoColor=white&label=Alertmanager)](https://status.turbo.ac)

</div>

<div align="center">

[![Age-Days](https://img.shields.io/endpoint?url=https%3A%2F%2Fkromgo.turbo.ac%2Fcluster_age_days&style=flat-square&label=Age)](https://github.com/kashalls/kromgo)&nbsp;&nbsp;
[![Uptime-Days](https://img.shields.io/endpoint?url=https%3A%2F%2Fkromgo.turbo.ac%2Fcluster_uptime_days&style=flat-square&label=Uptime)](https://github.com/kashalls/kromgo)&nbsp;&nbsp;
[![Node-Count](https://img.shields.io/endpoint?url=https%3A%2F%2Fkromgo.turbo.ac%2Fcluster_node_count&style=flat-square&label=Nodes)](https://github.com/kashalls/kromgo)&nbsp;&nbsp;
[![Pod-Count](https://img.shields.io/endpoint?url=https%3A%2F%2Fkromgo.turbo.ac%2Fcluster_pod_count&style=flat-square&label=Pods)](https://github.com/kashalls/kromgo)&nbsp;&nbsp;
[![CPU-Usage](https://img.shields.io/endpoint?url=https%3A%2F%2Fkromgo.turbo.ac%2Fcluster_cpu_usage&style=flat-square&label=CPU)](https://github.com/kashalls/kromgo)&nbsp;&nbsp;
[![Memory-Usage](https://img.shields.io/endpoint?url=https%3A%2F%2Fkromgo.turbo.ac%2Fcluster_memory_usage&style=flat-square&label=Memory)](https://github.com/kashalls/kromgo)&nbsp;&nbsp;
[![Power-Usage](https://img.shields.io/endpoint?url=https%3A%2F%2Fkromgo.turbo.ac%2Fcluster_power_usage&style=flat-square&label=Power)](https://github.com/kashalls/kromgo)&nbsp;&nbsp;
[![Alerts](https://img.shields.io/endpoint?url=https%3A%2F%2Fkromgo.turbo.ac%2Fcluster_alert_count&style=flat-square&label=Alerts)](https://github.com/kashalls/kromgo)

</div>

---

# 🏡 Homelab of Dreams --- *Kagiso's Self‑Hosted Kingdom* 🚀

![Homelab Banner --- Upload Your Image
Here](assets/banner-placeholder.png)

Welcome to **my Homelab**, where DevOps obsession meets DIY chaos and
somehow results in beautiful, automated infrastructure.\
This repository documents **my entire self‑hosted stack** --- from
Kubernetes to ArgoCD, Ansible bootstrapping, TrueNAS-powered
persistence, monitoring, backups, observability... and lots of YAML.

------------------------------------------------------------------------

## 🌟 Why Self‑Host?

Because the cloud is great --- until it isn't.

I wanted: - Full control\
- Improved security\
- Zero surprise billing\
- To treat my home like a mini‑enterprise\
- A playground to sharpen DevOps, Kubernetes and automation skills\
- A chance to say: *"Yeah, my entire home runs on GitOps."*

And the cherry on top?

### 🌀 **GitOps with ArgoCD**

The entire cluster is deployed and managed using: - **ArgoCD** for
continuous deployment\
- **Infrastructure as Code (IaC)** principles\
- GitHub → ArgoCD → K3s pipeline\
- Declarative everything

Push a commit → ArgoCD syncs → Cluster updates → Done.\
No kubectl spam. No manual YAML drifting. No forgetting what you
deployed at 3AM.

------------------------------------------------------------------------

## 🖥️ Homelab Infrastructure & Hardware

### 🧱 **K3s Cluster**

  ------------------------------------------------------------------------
  Node           Hostname                Specs              Role
  -------------- ----------------------- ------------------ --------------
  Lenovo         **tywin**               Xeon E3‑1260L v3 • K3s Server
  ThinkCentre                            16GB RAM • 256GB
                                         SSD

  Lenovo         **jaime**               Xeon E3‑1260L v3 • Worker
  ThinkCentre                            16GB RAM • 256GB
                                         SSD

  Lenovo         **tyrion**              Xeon E3‑1260L v3 • Worker
  ThinkCentre                            16GB RAM • 256GB
                                         SSD
  ------------------------------------------------------------------------

### 🍓 **Raspberry Pi 4**

-   Hostname: `rpi`
-   **10.0.10.10**
-   Duties:
    -   kubectl node\
    -   Grafana\
    -   Prometheus\
    -   Monitoring stack
-   Low power, always on, perfect monitoring hub.

### 📦 **TrueNAS Core Server**

Running on **HP Microserver Gen 8**

  Component   Spec
  ----------- -----------------------------------------
  CPU         Xeon E3 1260L v2
  RAM         16GB ECC
  Boot        128GB SSD
  SSD Pool    2 × 512GB SSD (Mirror) --- *FAST tier*
  HDD Pool    2 × 8TB SAS 12K (Mirror) --- *BIG tier*

> **Image Placeholder --- Add your TrueNAS setup here:**\
> `assets/truenas-setup.png`

------------------------------------------------------------------------

## 📦 Persistent Storage --- Why NFS over Longhorn?

I originally explored Longhorn, but after performance tests and
real‑world use, I chose:

### **NFS on TrueNAS**

Because: - Lightweight\
- Reliable\
- Zero-maintenance\
- No heavy block storage engine\
- Perfect for home hardware\
- Native snapshots + replication\
- Can separate pools: - **SSD Pool (FAST)** → Databases, Jellyfin
Metadata, ArgoCD, Prometheus\
- **HDD Pool (BIG)** → Media, downloads, documents, backups

### **NFS PV/PVCs**

We deploy apps based on storage characteristics: - High I/O → SSD\
- Big storage → HDD\
- Apps with mixed requirements get their own PVC tuned accordingly

GitOps manages all PVC definitions, keeping everything reproducible and
declarative.

------------------------------------------------------------------------

## 🤖 Automation with Ansible

This repo includes an **Ansible bootstrap system** that:

1.  Provisions base OS settings\
2.  Installs K3s\
3.  Configures firewall\
4.  Sets up directories for persistence\
5.  Deploys monitoring agents\
6.  Performs cluster join operations\
7.  Pushes kubeconfig to the RPi node\
8.  Runs preflight checks\
9.  Triggers the initial ArgoCD sync

### 🛠️ IaC Philosophy:

> "If I cannot recreate the cluster from scratch in one command, it does
> not exist."

Ansible + ArgoCD =\
**Automated provisioning + Automated deployment + Automated
reconciliation**

------------------------------------------------------------------------

## 💾 Backup Strategy --- "If It Isn't Backed Up, It's Already Lost"

This README includes our entire comprehensive backup plan:

### 🔄 **1. K3s Backups**

-   Automated etcd snapshot scripts\
-   Encrypted upload to TrueNAS via SSH/rsync\
-   Verification script\
-   Grafana dashboard to visualize backup health\
-   Notification alerts on failure

### 📁 **2. PVC Backups**

-   Dataset-level snapshots on TrueNAS\
-   Daily + Weekly + Monthly retention\
-   Replication (optional)

### 📜 **3. GitOps as a Backup**

All apps, configs, ingress, PVCs, CRDs are stored as: - YAML\
- Helm values\
- Kustomize overlays

Rebuild cluster → ArgoCD applies everything → Back in minutes.

### 📊 Grafana Backup Dashboard

See backup age, snapshot status, and sync logs.\
(Add screenshot here!)

------------------------------------------------------------------------

## 📈 Monitoring & Observability (Grafana via RPi)

The RPi hosts: - Grafana\
- Prometheus\
- Node Exporters\
- Traefik Metrics\
- K3s Metrics\
- Backup dashboards\
- Home monitoring (CPU, RAM, temps, network)

It's low power and always always on --- perfect for observability.

### Dashboards include:

-   K3s Cluster Health\
-   Node Metrics\
-   Traefik Requests\
-   Plex/Jellyfin performance\
-   Backup verification\
-   Storage utilization\
-   Network throughput

> **Screenshot Placeholder:** `assets/grafana-overview.png`

------------------------------------------------------------------------

## 🧭 Repository Structure

    📁 ansible/
    📁 cluster/
       ├── argocd/
       ├── apps/
       ├── monitoring/
    📁 storage/
    📁 backups/
    📁 docs/
    README.md

Each folder is orchestrated by ArgoCD using declarative GitOps
principles.

------------------------------------------------------------------------

## 🧩 GitHub Badges You Can Add

![Visitors](https://komarev.com/ghpvc/?username=YOUR_USERNAME&color=blue)\
![Stars](https://img.shields.io/github/stars/YOUR_REPO?style=social)\
![CI](https://img.shields.io/github/actions/workflow/status/YOUR_REPO/ci.yml)\
![License](https://img.shields.io/github/license/YOUR_REPO)

------------------------------------------------------------------------

## 🎉 Final Thoughts
;l'
This homelab is more than a cluster ---\
It's a fully automated, GitOps-driven, self‑hosted dream.

Everything here is: - Declarative\
- Automated\
- Reproducible\
- Monitored\
- Backed up\
- Fun to maintain

If you love homelabs, DevOps, automation, Kubernetes, or just organized
chaos --- you'll feel right at home.

------------------------------------------------------------------------
