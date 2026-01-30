---
layout: post
title: "AWS CLI Context Toolkit (SSO + SSM)"
date: 2026-01-30
categories: [DevOps, AWS, CLI]
tags: [AWS, AWS CLI, SSO, SSM, IAM Identity Center, Bash, fzf]
description: "Toolkit to manage context (profile + region) and connect via SSM without the web console."
---

# AWS CLI Context Toolkit (SSO + SSM)

> A **convenient, fast, and web‑console‑free** way to work with multiple AWS accounts and regions, inspired by `kubectx` / `kubens`.

This document explains **what problem it solves**, **how to set it up**, and **how to use it day to day**.

Repository: `https://github.com/sondosclick/aws-bash-toolbox.git`

---

## 🎯 Goal

When we work with AWS daily we usually have:

- many **accounts**
- several **permission sets / roles**
- multiple **regions**
- access via **IAM Identity Center (SSO)**
- instance access via **SSM Session Manager**

The typical outcome is:

- long commands with `--profile` and `--region`
- jumping into the **AWS Console** just to copy an `instance-id`
- mistakes because we’re in the **wrong account or region**

This toolkit tries to solve that with:

- an **active context** (profile + region)
- short, consistent commands
- interactive selectors with `fzf`
- SSM access **without SSH, without bastions, and without the web console**

---

## 🧩 What’s included

### Context management (kubectx‑style)

- `awsctx` → shows the current context
- `awsp <profile>` → switches profile
- `awsr <region>` → switches region
- `awsctxf` → interactive profile@region selector

The context is **persisted** across terminals.

---

### EC2 / SSM utilities

- `ec2ls` → lists instances (Name, ID, IP, state)
- `ssm <instance-id>` → connects via SSM
- `ssm -n <NameTag>` → connects via SSM using the Name tag
- `ssmfzf` → interactive instance selector + SSM

Everything automatically uses the **current context**.

---

### Extras

- automatic **proxy/VPN bypass for SSM only**
- TAB autocompletion for profiles and regions
- optional prompt with `[aws:profile@region]`

---

## 📦 Requirements

- Ubuntu 24.04
- Bash
- AWS CLI v2
- AWS access via **IAM Identity Center (SSO)**

Required packages:

```bash
sudo apt update
sudo apt install -y fzf session-manager-plugin
```

---

## 🔧 Setup

### 1️⃣ AWS CLI + SSO

You need at least:

- one SSO session (`[sso-session ...]`)
- one or more SSO profiles (`[profile ...]`)

Minimal example:

```ini
[sso-session corporation-sso]
sso_start_url = https://xxxx.awsapps.com/start
sso_region = eu-central-1
sso_registration_scopes = sso:account:access

[profile corporation-base]
sso_session = corporation-sso
sso_account_id = 123456789012
sso_role_name = admin
region = eu-central-1
```

Initial login:

```bash
aws sso login --sso-session corporation-sso
```

---

### 2️⃣ Context toolkit

Copy the **entire toolkit block** into `~/.bashrc` and reload:

```bash
source ~/.bashrc
```

(The block is designed to be self‑contained and not break anything existing.)

---

## 🚀 Daily use

### Show current context

```bash
awsctx
```

Typical output:

```
AWS_PROFILE=corporation-env-admin  AWS_REGION=eu-west-3
```

---

### Switch profile

```bash
awsp corporation-env-admin
```

With TAB for autocompletion.

---

### Switch region

```bash
awsr eu-west-3
```

---

### Switch profile + region at once (pro mode)

```bash
awsctxf
```

- interactive selector
- shows `profile@region` combos
- remembers the last context

Very similar to `kubectx`.

---

## 🖥️ Working with EC2

### List instances

```bash
ec2ls
```

Shows:

- Name (tag)
- InstanceId
- Private IP
- State

No web console required.

---

### Connect via SSM (direct)

```bash
ssm i-0123456789abcdef0
```

---

### Connect via SSM using the Name tag

```bash
ssm -n api-01
```

---

### Interactive instance selector (recommended)

```bash
ssmfzf
```

- lists only `running` instances
- arrows / search
- Enter → SSM session opened

This is usually the most comfortable flow.

---

## 🌐 Note about proxy / VPN

In corporate environments it’s common that:

- AWS CLI works
- SSM fails with errors like `EOF`

So:

- **only SSM commands** ignore proxy variables
- the rest of AWS CLI still uses proxy if needed

This avoids breaking other corporate workflows.

---

## 🧠 Toolkit philosophy

- Less mental context switching (you always know where you are)
- Less web console usage
- Less copy/pasting IDs
- Repeatable, fast workflows

It doesn’t try to replace Terraform, CDK, or the console.
It’s an **ergonomics layer for day‑to‑day work**.

---

## 🧪 Next ideas (if the team adopts it)

- auto `aws sso login` on profile switch
- favorite contexts (shortcuts)
- tmux integration
- zsh support
- wrapper for port‑forwarding (DBs, internal apps)

---

## 🙌 Feedback

If this saves you time:

- try it for a week
- break things
- propose improvements

The idea is to evolve it **from real team usage**, not as something rigid.
