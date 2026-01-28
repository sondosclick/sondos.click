---
layout: post
title: "A base image for VS Code Dev Containers (and how not to go crazy maintaining it)"
date: 2026-01-23
categories: [DevOps, Containers, VS Code]
tags: [Dev Containers, Docker, GitHub Actions, Proxy, Certificates, CI/CD]
description: "How to create and maintain a corporate base image for Dev Containers, with monthly automation and registry publishing."
featureimage: "/images/posts/2026-01-23-base-image-para-dev-containers/feature.svg"
---

# A base image for VS Code Dev Containers (and how not to go crazy maintaining it)

![Illustration of layers and containers](/images/posts/2026-01-23-base-image-para-dev-containers/feature.svg)

If you use **VS Code Dev Containers** in a corporate environment (proxy, Zscaler, internal certs, weird mirrors…), you’ve probably already wrestled with errors like:

> `x509: certificate signed by unknown authority`

And they almost always show up **at the worst possible time**: when the container is still being created and VS Code is trying to install features or download tooling.

This article is about avoiding that pain **once and for all** by creating and maintaining **your own base image** for Dev Containers. No heroics, no monstrous Dockerfiles, no turning it into technical debt.

---

## 🧠 The core idea (in one sentence)

👉 *Anything that’s cross-cutting and corporate must exist **before** VS Code starts installing features.*

That includes:
- Corporate certificates (Zscaler, MITM, etc.)
- Proxy and connectivity
- Basic packages

If not, Dev Containers arrives late… and fails.

---

## 😖 The typical problem

Picture it like this:

```
VS Code
  └─ creates container
      └─ installs features (Go, Python, Terraform…)
          └─ HTTPS
              └─ 💥 cert error
```

At that point:
- `postCreateCommand` won’t help
- `initializeCommand` won’t either
- and you just wanted to code

---

## 💡 The solution: your own base image

Instead of starting **every repo from scratch**, create a base image that’s already prepared for your environment.

```
[ Corporate base image ]
        │
        ▼
[ Repo Dev Container ]
        │
        ▼
[ VS Code Features ]  ✅ works
```

The base image is the right place for:
- certificates
- proxy
- common packages
- cross-cutting configuration

---

## ⚖️ When it’s worth it (and when it’s not)

### ✅ It’s worth it if:
- You use Dev Containers in **multiple repos or teams**
- You’re behind **proxy / Zscaler / TLS inspection**
- You want fast, predictable startup

### ⚠️ It’s a bad idea if:
- Every repo is completely different
- No one owns maintenance
- You want to throw *all* tools in “just in case”

---

## 🧱 Design principles (this saves you headaches)

Before writing Dockerfiles:

1. 🧩 **The base image is a product**, not a hack
2. 🧼 **Include only the cross-cutting stuff**
3. 🔐 **Never put secrets in**
4. 🏷️ **Always version it** (no `latest`-only)
5. 🤖 **Automate everything** (build, test, publish)

If you don’t follow this, the image degrades fast.

---

## 📦 Recommended repository structure

Create a dedicated repo, for example:

```
devcontainers-base/
├── Dockerfile
├── certs/
│   └── zscaler-ca.crt
├── scripts/
│   ├── test-connectivity.sh
│   └── smoke-test.sh
├── .github/workflows/
│   └── build-and-publish.yml
└── README.md
```

This repo **doesn’t depend on any specific project**.

---

## 🐳 Dockerfile: simple and boring (as it should be)

### Official Dev Containers base

```dockerfile
FROM mcr.microsoft.com/devcontainers/base:noble
```

This guarantees full compatibility with VS Code.

---

### 🔐 Corporate certificates (the most important part)

```dockerfile
COPY certs/zscaler-ca.crt /usr/local/share/ca-certificates/zscaler-ca.crt
RUN update-ca-certificates
```

With that, these already work:
- `apt`
- `curl` / `wget`
- `git`
- `pip`, `npm`, etc.

From **second one**.

---

### 🧰 Common packages

```dockerfile
RUN apt-get update && apt-get install -y \
    ca-certificates \
    curl \
    git \
    build-essential \
    jq \
 && rm -rf /var/lib/apt/lists/*
```

Rule of thumb:
> If most projects don’t use it, **it doesn’t belong here**.

---

## 📤 Where to publish the image

Common options:
- Internal registry (Harbor, Artifactory, ACR, ECR…)
- GHCR if your org allows it

Example tags:

```
registry.company.com/devcontainers/base:noble-2026.01
registry.company.com/devcontainers/base:noble-latest
```

⚠️ Important: **don’t force `latest` only**.

---

## ✅ Enable GitHub Container Registry (GHCR)

To publish images to GHCR you need two things:
1. **Actions permissions** to create packages
2. **Package visibility** (public or private)

### Minimal steps

1) In the repo: `Settings → Actions → General`  
   - Enable **Read and write permissions** for the `GITHUB_TOKEN`
2) In an org: `Settings → Packages`  
   - Make sure Actions can **create and write** packages
3) After the first push, adjust package visibility in `Packages`

### Is it free?

- **Public packages:** free (no storage or transfer costs)
- **Private packages:** subject to your plan limits (storage and bandwidth quotas)

If you go private, check `Settings → Billing` in your org or account for exact limits.

---

## 🤖 Automation: build, test, and publish

Every change should automatically do this:

```
commit / tag
   │
   ▼
build image
   │
   ▼
basic tests
   │
   ▼
publish with tags
```

### Minimal but real tests

**TLS connectivity**:

```bash
curl -I https://github.com
git ls-remote https://github.com/git/git
```

**APT working**:

```bash
apt-get update
```

If this fails behind the proxy → **don’t publish**.

---

## 🧑‍💻 How to use it in a repo with Dev Containers

In `devcontainer.json`:

```jsonc
{
  "name": "My project",
  "image": "registry.company.com/devcontainers/base:noble-2026.01",
  "features": {
    "ghcr.io/devcontainers/features/go:1": {},
    "ghcr.io/devcontainers/features/python:1": {},
    "ghcr.io/devcontainers/features/terraform:1": {}
  }
}
```

Result:
- The container starts without errors
- Features install without drama
- The repo stays clean and simple

---

## ➕ Adding new tools

Always ask yourself:

### ❓ Is it cross-cutting?

#### ✅ Yes → base image
Examples:
- `kubectl`
- `awscli`
- `docker-cli`

Process:
1. PR to the base repo
2. Clear justification
3. Build + tests
4. New version

#### ❌ No → specific repo

Use:
- Dev Containers features
- A derived Dockerfile

Don’t bloat the base “just in case.”

---

## 📅 Monthly update (without pain)

### Recommended cadence
- Once a month
- Same week each month

### What gets updated
- Security patches
- Base packages
- Certificates (if they rotate)

### Typical flow

```
release/YYYY.MM
   │
   ▼
apt update / upgrade
   │
   ▼
tests
   │
   ▼
publish new image
```

Repos **migrate when they want** by changing the tag.

---

## 🕒 Optional: automate the monthly update with GitHub Actions

If you want the image to publish itself on the first day of each month, you can use a workflow with `cron` and a tag format like `base:noble-YYYY.MM`.

Simple example (build + publish) using GHCR:

```yaml
name: monthly-base-image

on:
  schedule:
    - cron: "0 6 1 * *" # first day of the month, 06:00 UTC
  workflow_dispatch: {}

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v4
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - name: Set tag
        run: echo "TAG=$(date +'%Y.%m')" >> "$GITHUB_ENV"
      - uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: |
            ghcr.io/your-org/base:noble-${{ env.TAG }}
            ghcr.io/your-org/base:noble-latest
```

If you prefer an internal registry, change `registry` and the `tags` prefix.

Extra recommendation: add a basic tests step (TLS + apt) before `build-push-action` so you don’t publish a broken image.

---

## 🧯 Common errors (learned the hard way)

- Using `latest` as the only reference
- Stuffing SDKs for every language
- Not testing behind the real proxy
- No clear ownership

---

## 🎯 Conclusion

A base image for **VS Code Dev Containers** is one of those invisible pieces that:
- nobody notices when it works,
- but everyone suffers when it doesn’t exist.

If your team wastes time with certificates, proxy, or slow provisioning, creating a base image **isn’t a luxury**: it’s efficiency.

Start small, keep it boring, and update it with discipline.

Your future self (and your team) will thank you.
