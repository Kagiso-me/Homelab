# Kagiso Helm Charts

This repository hosts **Kagiso’s Helm Charts**, a curated collection of Helm charts for self-hosted services, homelabs, and Kubernetes workloads.

Charts are automatically packaged and published to **GitHub Pages**, making this repository a standard Helm chart repository and compatible with **Artifact Hub**.

---

## 📦 Helm Repository

**Repository URL**

```
https://kagiso-me.github.io/helm-charts
```

### Add the repository

```bash
helm repo add kagiso https://kagiso-me.github.io/helm-charts
helm repo update
```

### Install a chart

```bash
helm install my-release kagiso/<chart-name>
```

---

## 📁 Repository Structure

```
.
├── charts/
│   ├── argocd/
│   ├── authentik/
│   ├── crowdsec/
│   ├── erpnext/
│   ├── ghost/
│   ├── immich/
│   ├── jellyfin/
│   ├── jellyseerr/
│   ├── grafana/
│   ├── mariadb/
│   ├── nextcloud/
│   ├── prowlarr/
│   ├── sabnzbd/
│   └── wordpress/
│
├── artifacthub-repo.yml
├── .github/
│   └── workflows/
│       └── release.yml
└── README.md
```

---

## 🔁 Release & Publishing Workflow

On every push to `main`:

1. Helm charts are linted
2. Charts are packaged
3. `index.yaml` is generated
4. Packages and index are published to the **`gh-pages` branch**
5. GitHub Pages serves the Helm repository

This repository **does not require** a `gh-pages` folder in `main`.
The branch is created and managed automatically by GitHub Actions.

---

## 🌐 GitHub Pages Website (Human-Readable)

The Helm repository is consumed by tooling, but documentation is provided separately via:

```
kagiso-me.github.io
└── _charts/
    └── index.md
```

* `_charts/index.md` is **for humans**
* It explains how to use the Helm repo
* It may link to Artifact Hub and GitHub
* It is **not** used by Helm or Artifact Hub

---

## 📦 Artifact Hub

This repository is registered on **Artifact Hub**.

Repository metadata is defined in:

```
artifacthub-repo.yml
```

Each chart should include appropriate annotations in `Chart.yaml`, for example:

```yaml
annotations:
  artifacthub.io/category: Monitoring
```

> Artifact Hub **does not read** the GitHub Pages website repository.
> It reads the Helm repository directly from `index.yaml`.

---

## 🛡 Design Principles

* Clear separation between:

  * Helm distribution
  * Website documentation
* Stable, manual Artifact Hub metadata
* Fully automated chart releases
* Minimal CI complexity
* No system-level package installs in CI

---

## 📜 License

Apache License 2.0

---

## 👤 Maintainer

**Kagiso**
📧 [admin@kagiso.me](mailto:admin@kagiso.me)
🔗 [https://github.com/kagiso-me](https://github.com/kagiso-me)
