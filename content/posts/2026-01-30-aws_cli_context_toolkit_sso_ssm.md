---
layout: post
title: "AWS CLI Context Toolkit (SSO + SSM)"
date: 2026-01-30
categories: [DevOps, AWS, CLI]
tags: [AWS, AWS CLI, SSO, SSM, IAM Identity Center, Bash, fzf]
description: "Toolkit para gestionar contexto (profile + region) y conectar por SSM sin consola web."
---

# AWS CLI Context Toolkit (SSO + SSM)

> Una forma **cómoda, rápida y sin consola web** de trabajar con múltiples cuentas y regiones de AWS, inspirada en `kubectx` / `kubens`.

Este documento explica **qué problema resuelve**, **cómo se configura** y **cómo se usa en el día a día**.

Repositorio: `https://github.com/sondosclick/aws-bash-toolbox.git`

---

## 🎯 Objetivo

Cuando trabajamos con AWS a diario solemos tener:

- muchas **cuentas**
- varios **permission sets / roles**
- múltiples **regiones**
- acceso vía **IAM Identity Center (SSO)**
- conexión a instancias vía **SSM Session Manager**

El resultado típico es:

- comandos largos con `--profile` y `--region`
- ir a la **AWS Console** solo para copiar un `instance-id`
- errores por estar en la **cuenta o región equivocada**

Este toolkit intenta resolver eso con:

- un **contexto activo** (profile + region)
- comandos cortos y coherentes
- selectores interactivos con `fzf`
- conexión por SSM **sin SSH, sin bastiones y sin consola web**

---

## 🧩 Qué incluye

### Gestión de contexto (tipo kubectx)

- `awsctx` → muestra el contexto actual
- `awsp <profile>` → cambia de profile
- `awsr <region>` → cambia de región
- `awsctxf` → selector interactivo profile@region

El contexto se **persiste** entre terminales.

---

### Utilidades EC2 / SSM

- `ec2ls` → lista instancias (Name, ID, IP, estado)
- `ssm <instance-id>` → conecta por SSM
- `ssm -n <NameTag>` → conecta por SSM usando el tag Name
- `ssmfzf` → selector interactivo de instancias + SSM

Todo usa automáticamente el **contexto actual**.

---

### Extras

- bypass automático de **proxy/VPN solo para SSM**
- autocompletado TAB para profiles y regiones
- prompt opcional con `[aws:profile@region]`

---

## 📦 Requisitos

- Ubuntu 24.04
- Bash
- AWS CLI v2
- Acceso a AWS vía **IAM Identity Center (SSO)**

Paquetes necesarios:

```bash
sudo apt update
sudo apt install -y fzf session-manager-plugin
```

---

## 🔧 Configuración

### 1️⃣ AWS CLI + SSO

Debe existir al menos:

- una sesión SSO (`[sso-session ...]`)
- uno o varios perfiles SSO (`[profile ...]`)

Ejemplo mínimo:

```ini
[sso-session corporation-sso]
sso_start_url = https://xxxx.awsapps.com/start
sso_region = eu-central-1
sso_registration_scopes = sso:account:access

[profile corporation-base]
sso_session = corporation-sso
sso_account_id = 123456789012
sso_role_name = InfraAdmin
region = eu-central-1
```

Login inicial:

```bash
aws sso login --sso-session corporation-sso
```

---

### 2️⃣ Toolkit de contexto

Copiar **todo el bloque** del toolkit en `~/.bashrc` y recargar:

```bash
source ~/.bashrc
```

(El bloque completo está pensado para ser autocontenido y no romper nada existente.)

---

## 🚀 Uso diario

### Ver contexto actual

```bash
awsctx
```

Salida típica:

```
AWS_PROFILE=corporation-pt-p-infraadmin  AWS_REGION=eu-west-3
```

---

### Cambiar de profile

```bash
awsp corporation-pt-p-infraadmin
```

Con TAB para autocompletar.

---

### Cambiar de región

```bash
awsr eu-west-3
```

---

### Cambiar profile + región de golpe (modo pro)

```bash
awsctxf
```

- selector interactivo
- muestra combinaciones `profile@region`
- recuerda el último contexto

Muy parecido a `kubectx`.

---

## 🖥️ Trabajar con EC2

### Listar instancias

```bash
ec2ls
```

Muestra:

- Name (tag)
- InstanceId
- IP privada
- Estado

Sin necesidad de consola web.

---

### Conectarse por SSM (directo)

```bash
ssm i-0123456789abcdef0
```

---

### Conectarse por SSM usando el Name tag

```bash
ssm -n api-01
```

---

### Selector interactivo de instancias (recomendado)

```bash
ssmfzf
```

- lista solo instancias en `running`
- flechas / búsqueda
- Enter → sesión SSM abierta

Este suele ser el flujo más cómodo.

---

## 🌐 Nota sobre proxy / VPN

En entornos corporativos es común que:

- AWS CLI funcione
- SSM falle con errores tipo `EOF`

Por eso:

- **solo los comandos SSM** ignoran variables de proxy
- el resto de AWS CLI sigue usando proxy si lo necesita

Esto evita romper otros flujos corporativos.

---

## 🧠 Filosofía del toolkit

- Menos contexto mental (siempre sabes dónde estás)
- Menos consola web
- Menos copiar/pegar IDs
- Flujos repetibles y rápidos

No pretende sustituir Terraform, CDK o la consola.
Es una **capa de ergonomía para el día a día**.

---

## 🧪 Siguientes ideas (si el equipo lo adopta)

- auto `aws sso login` al cambiar de profile
- contextos favoritos (shortcuts)
- integración con tmux
- soporte zsh
- wrapper para port-forwarding (DBs, apps internas)

---

## 🙌 Feedback

Si esto te ahorra tiempo:

- pruébalo una semana
- rompe cosas
- propón mejoras

La idea es que evolucione **desde el uso real del equipo**, no como algo rígido.
