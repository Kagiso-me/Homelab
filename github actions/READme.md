# Kagiso Helm Charts Deployment Guide

This guide walks you through the full Helm charts setup, workflow, and integration with your Jekyll site.

---

## 1. Directory Structure

├─ .github/workflows/release.yml
├─ .github/templates/dashboard.html
├─ charts/
│ ├─ chart1/
│ └─ chart2/
├─ .deploy/ # Temporary build folder
└─ Kagiso-me.github.io/ # Optional Jekyll integration
└─ _charts/
├─ index.yaml
└─ index.html


---

## 2. Workflow Overview

The GitHub Actions workflow handles:

1. Checkout of repo
2. Helm setup
3. Tool installation (`yq`, `kubeval`, `kube-score`)
4. Detect changed charts
5. Lint & validate charts
6. Semantic version bump
7. Changelog automation
8. Packaging charts
9. Generate Helm repo index
10. Add Artifact Hub metadata
11. Deploy to `gh-pages` branch
12. Optional sync to Jekyll `_charts/` folder

---

## 3. Setting Up Your Charts

1. Add charts under `charts/`:
charts/
├─ mychart/
│ ├─ Chart.yaml
│ ├─ values.yaml
│ └─ templates/


2. Include a `CHANGELOG.md` if you want historical version notes.

---

## 4. Deployment

- Push changes to `main`.
- GitHub Actions will automatically:

  - Validate charts
  - Bump versions
  - Update changelogs
  - Package charts
  - Update `index.yaml`
  - Deploy to `gh-pages` for Artifact Hub
  - Optionally sync `_charts/` folder in Jekyll site

---

## 5. Jekyll Site Integration

1. Create a page: `helm-charts/index.html`.
2. Add JS (`assets/js/helm-charts.js`) to dynamically read `_charts/index.yaml`.
3. Style cards using your site theme.
4. Add a menu link to `/helm-charts/` from your main site navigation.

---

## 6. Artifact Hub Integration

- The workflow generates `artifacthub-repo.yml` in `.deploy/`.
- Artifact Hub will detect the charts and display:

  - Chart name, description, versions
  - Installation instructions
  - Maintainers
  - License

- Make sure your repo is registered in Artifact Hub under your account.

---

## 7. Notes

- Always push changes to `main`.
- Workflow is idempotent — no need to manually bump versions or changelogs.
- `_charts/index.yaml` is the source of truth for both Artifact Hub and Jekyll site.

---

## 8. Customization

- Dashboard look & feel is controlled via your Jekyll theme.
- You can add badges, install instructions, or additional metadata using `helm-charts.js`.
- Changelogs are automatically generated from commit messages prefixed with `feat:` or `fix:`.

---

## 9. Summary

With this setup:

- Helm repo is automated and Artifact Hub ready.
- Jekyll site shows charts with your theme.
- Versions, changelogs, and packages are fully automated.
- You have a **single source of truth** (`index.yaml`) for all charts.

---
