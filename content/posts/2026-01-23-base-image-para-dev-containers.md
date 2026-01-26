---
layout: post
title: "Una imagen base para VS Code Dev Containers (y cómo no volverte loco manteniéndola)"
date: 2026-01-23
categories: [DevOps, Contenedores, VS Code]
tags: [Dev Containers, Docker, GitHub Actions, Proxy, Certificados, CI/CD]
description: "Cómo crear y mantener una imagen base corporativa para Dev Containers, con automatización mensual y publicación en registry."
featureimage: "/images/posts/2026-01-23-base-image-para-dev-containers/feature.svg"
---

# Una imagen base para VS Code Dev Containers (y cómo no volverte loco manteniéndola)

![Ilustracion de capas y contenedores](/images/posts/2026-01-23-base-image-para-dev-containers/feature.svg)

Si usas **VS Code Dev Containers** en un entorno corporativo (proxy, Zscaler, certificados internos, mirrors raros…), probablemente ya te has peleado con errores tipo:

> `x509: certificate signed by unknown authority`

Y casi siempre aparecen **en el peor momento**: cuando el contenedor todavía se está creando y VS Code intenta instalar features o descargar tooling.

Este artículo va de cómo evitar ese dolor **de una vez por todas** creando y manteniendo una **imagen base propia** para tus Dev Containers. Sin heroicidades, sin Dockerfiles monstruosos y sin convertirlo en deuda técnica.

---

## 🧠 La idea clave (en una frase)

👉 *Todo lo que sea transversal y corporativo debe existir **antes** de que VS Code empiece a instalar features.*

Eso incluye:
- Certificados corporativos (Zscaler, MITM, etc.)
- Proxy y conectividad
- Paquetes básicos

Si no, Dev Containers llega tarde… y falla.

---

## 😖 El problema típico

Visualízalo así:

```
VS Code
  └─ crea contenedor
      └─ instala features (Go, Python, Terraform…)
          └─ HTTPS
              └─ 💥 cert error
```

En ese punto:
- `postCreateCommand` no sirve
- `initializeCommand` tampoco
- y tú solo querías programar

---

## 💡 La solución: una imagen base propia

En lugar de empezar **cada repo desde cero**, creas una imagen base que ya venga preparada para tu entorno.

```
[ Imagen base corporativa ]
        │
        ▼
[ Dev Container del repo ]
        │
        ▼
[ Features de VS Code ]  ✅ funcionan
```

La imagen base es el sitio correcto para:
- certificados
- proxy
- paquetes comunes
- configuración transversal

---

## ⚖️ Cuándo merece la pena (y cuándo no)

### ✅ Sí merece la pena si:
- Usas Dev Containers en **varios repos o equipos**
- Estás detrás de **proxy / Zscaler / inspección TLS**
- Quieres arranques rápidos y predecibles

### ⚠️ No es buena idea si:
- Cada repo es completamente distinto
- Nadie se hace cargo del mantenimiento
- Quieres meter *todas* las herramientas “por si acaso”

---

## 🧱 Principios de diseño (esto te ahorra problemas)

Antes de escribir Dockerfiles:

1. 🧩 **La imagen base es un producto**, no un apaño
2. 🧼 **Incluye solo lo transversal**
3. 🔐 **Nunca metas secretos**
4. 🏷️ **Versiona siempre** (nada de solo `latest`)
5. 🤖 **Automatiza todo** (build, test, publish)

Si no cumples esto, la imagen se degrada rápido.

---

## 📦 Estructura recomendada del repositorio

Crea un repo dedicado, por ejemplo:

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

Este repo **no depende de ningún proyecto concreto**.

---

## 🐳 Dockerfile: simple y aburrido (como debe ser)

### Base oficial de Dev Containers

```dockerfile
FROM mcr.microsoft.com/devcontainers/base:noble
```

Esto garantiza compatibilidad total con VS Code.

---

### 🔐 Certificados corporativos (lo más importante)

```dockerfile
COPY certs/zscaler-ca.crt /usr/local/share/ca-certificates/zscaler-ca.crt
RUN update-ca-certificates
```

Con esto ya funcionan:
- `apt`
- `curl` / `wget`
- `git`
- `pip`, `npm`, etc.

Desde el **primer segundo**.

---

### 🧰 Paquetes comunes

```dockerfile
RUN apt-get update && apt-get install -y \
    ca-certificates \
    curl \
    git \
    build-essential \
    jq \
 && rm -rf /var/lib/apt/lists/*
```

Regla de oro:
> Si no lo usan la mayoría de proyectos, **no va aquí**.

---

## 📤 Dónde publicar la imagen

Opciones habituales:
- Registry interno (Harbor, Artifactory, ACR, ECR…)
- GHCR si tu org lo permite

Ejemplo de tags:

```
registry.empresa.com/devcontainers/base:noble-2026.01
registry.empresa.com/devcontainers/base:noble-latest
```

⚠️ Importante: **no obligues a usar solo `latest`**.

---

## ✅ Habilitar GitHub Container Registry (GHCR)

Para publicar imágenes en GHCR necesitas dos cosas:
1. **Permisos de Actions** para crear paquetes
2. **Visibilidad del paquete** (público o privado)

### Pasos mínimos

1) En el repo: `Settings → Actions → General`  
   - Activa **Read and write permissions** para el `GITHUB_TOKEN`
2) En una org: `Settings → Packages`  
   - Asegura que Actions puede **crear y escribir** paquetes
3) Tras el primer push, ajusta la visibilidad del paquete en `Packages`

### ¿Es gratuito?

- **Paquetes públicos:** gratuitos (sin coste de almacenamiento ni transferencia)
- **Paquetes privados:** sujetos a los límites de tu plan (cuotas de almacenamiento y ancho de banda)

Si vas con privados, revisa `Settings → Billing` en tu org o cuenta para ver los límites exactos.

---

## 🤖 Automatización: build, test y publish

Cada cambio debería hacer esto automáticamente:

```
commit / tag
   │
   ▼
build imagen
   │
   ▼
tests básicos
   │
   ▼
publish con tags
```

### Tests mínimos pero reales

**Conectividad TLS**:

```bash
curl -I https://github.com
git ls-remote https://github.com/git/git
```

**APT funcionando**:

```bash
apt-get update
```

Si esto falla detrás del proxy → **no se publica**.

---

## 🧑‍💻 Cómo usarla en un repo con Dev Containers

En el `devcontainer.json`:

```jsonc
{
  "name": "Mi proyecto",
  "image": "registry.empresa.com/devcontainers/base:noble-2026.01",
  "features": {
    "ghcr.io/devcontainers/features/go:1": {},
    "ghcr.io/devcontainers/features/python:1": {},
    "ghcr.io/devcontainers/features/terraform:1": {}
  }
}
```

Resultado:
- El contenedor arranca sin errores
- Las features se instalan sin drama
- El repo queda limpio y simple

---

## ➕ Añadir nuevas herramientas

Hazte siempre esta pregunta:

### ❓ ¿Es transversal?

#### ✅ Sí → imagen base
Ejemplos:
- `kubectl`
- `awscli`
- `docker-cli`

Proceso:
1. PR al repo base
2. Justificación clara
3. Build + tests
4. Nueva versión

#### ❌ No → repo concreto

Usa:
- Features de Dev Containers
- Dockerfile derivado

No hinches la base “por si acaso”.

---

## 📅 Actualización mensual (sin dolor)

### Cadencia recomendada
- Una vez al mes
- Misma semana cada mes

### Qué se actualiza
- Parches de seguridad
- Paquetes base
- Certificados (si rotan)

### Flujo típico

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
publish nueva imagen
```

Los repos **migran cuando quieran** cambiando el tag.

---

## 🕒 Opcional: automatizar la actualización mensual con GitHub Actions

Si quieres que la imagen se publique sola el primer día de cada mes, puedes usar un workflow con `cron` y un tag con formato `base:noble-YYYY.MM`.

Ejemplo sencillo (build + publish) usando GHCR:

```yaml
name: monthly-base-image

on:
  schedule:
    - cron: "0 6 1 * *" # primer día del mes, 06:00 UTC
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
            ghcr.io/tu-org/base:noble-${{ env.TAG }}
            ghcr.io/tu-org/base:noble-latest
```

Si prefieres registry interno, cambia `registry` y el prefijo de `tags`.

Recomendación extra: añade un paso de tests básicos (TLS + apt) antes del `build-push-action` para no publicar una imagen rota.

---

## 🧯 Errores comunes (aprendidos a base de golpes)

- Usar `latest` como única referencia
- Meter SDKs de todos los lenguajes
- No testear detrás del proxy real
- No definir ownership claro

---

## 🎯 Conclusión

Una imagen base para **VS Code Dev Containers** es una de esas piezas invisibles que:
- nadie ve cuando funciona,
- pero todos sufren cuando no existe.

Si tu equipo pierde tiempo con certificados, proxy o provisión lenta, crear una imagen base **no es lujo**: es eficiencia.

Empieza pequeña, mantenla aburrida y actualízala con disciplina.

Tu yo del futuro (y tu equipo) te lo van a agradecer.
