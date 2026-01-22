---
title: "Git: enviar el mismo código a GitHub y GitLab desde un único repo local (remoto llamado 'remote')"
date: 2026-01-22T00:00:00+01:00
draft: false
tags: ["git", "github", "gitlab", "devops", "workflow"]
categories: ["guías"]
description: "Procedimiento para configurar un repo local con un remoto llamado 'remote' que haga push a GitHub, GitLab o ambos, y para sincronizar ramas."
---

# Git: enviar el mismo código a GitHub y GitLab desde un único repo local (remoto llamado `remote`)

¿Quieres tener **un solo repo local** y empujar a **GitHub y GitLab** sin romper nada?  
Aquí va una guía rápida y sin dramas. 😎

---

## 🎯 Objetivo

Tener **un único repositorio local** y mantener **dos repositorios remotos con el mismo código**:

- **Repo 1:** GitHub  
- **Repo 2:** GitLab  
- El remoto “principal” en tu repo local puede que se llame **`origin`** (no `remote`).

Además:
- Saber **cuándo y cómo** hacer push **solo a GitHub**, **solo a GitLab**, o **a ambos**.
- Aprender a **sincronizar ramas** (y opcionalmente tags) para que ambos remotos queden alineados.

---

## 🗺️ Convenciones usadas en esta guía

Sustituye estas URLs por las tuyas:

- GitHub: `https://github.com/<org>/<repo>.git`
- GitLab: `https://gitlab.com/<org>/<repo>.git`

Rama principal: `main` (si usas `master`, cambia `main` por `master` en todos los comandos).

---

## ✅ 1) Estado inicial (o verificación)

En tu repo local, lista remotos:

```bash
git remote -v
```

Si ya existe un remoto llamado `remote`, verás algo como:

```bash
remote  https://github.com/<org>/<repo>.git (fetch)
remote  https://github.com/<org>/<repo>.git (push)
```

Si no existe, lo creas en el siguiente paso. Fácil.

---

## 🔗 2) Configurar remotos para GitHub + GitLab

### Caso A: Aún no tienes el remoto `remote`

Configúralo apuntando primero a GitHub (puede ser GitLab si prefieres):

```bash
git remote add remote https://github.com/<org>/<repo>.git
```

Ahora añade remotos explícitos para separar pushes cuando lo necesites:

```bash
git remote add github https://github.com/<org>/<repo>.git
git remote add gitlab https://gitlab.com/<org>/<repo>.git
```

Sí: vas a tener tres nombres (`remote`, `github`, `gitlab`) a propósito.  
`github` y `gitlab` te sirven para push selectivo, y `remote` será el alias para push a ambos.

Verifica:

```bash
git remote -v
```

### Caso B: Ya tienes `remote` y quieres añadir GitLab

Primero crea remotos explícitos:

```bash
git remote add github https://github.com/<org>/<repo>.git
git remote add gitlab https://gitlab.com/<org>/<repo>.git
```

Luego ajustamos `remote` para que pueda empujar a ambos (más abajo).

---

## 🚀 3) Separar el push: repo1, repo2 y ambos

Aquí va lo importante. Vamos con los tres modos:

### Modo 1 — Push SOLO a GitHub (repo1)

```bash
git push github main
```

Cuándo usarlo:
- Si GitHub es el repo “principal” (PRs, checks, releases).  
- Si quieres probar antes en un remoto y no tocar el otro todavía.

### Modo 2 — Push SOLO a GitLab (repo2)

```bash
git push gitlab main
```

Cuándo usarlo:
- Si GitLab se usa para pipelines internos.
- Si quieres actualizar el mirror sin afectar GitHub.

### Modo 3 — Push a AMBOS con un solo comando (usando `remote`)

Vamos a configurar `remote` para que empuje a dos URLs de push.

#### 3.1 Configura `remote` con múltiples push URLs

Primero asegúrate de tener `remote` creado:

```bash
git remote add remote https://github.com/<org>/<repo>.git
```

Ahora añade dos URLs de push a `remote`:

```bash
git remote set-url --add --push remote https://github.com/<org>/<repo>.git
git remote set-url --add --push remote https://gitlab.com/<org>/<repo>.git
```

Opcional (y recomendado): define la URL de fetch de `remote` en una sola (por ejemplo GitHub):

```bash
git remote set-url remote https://github.com/<org>/<repo>.git
```

Verifica:

```bash
git remote -v
```

Deberías ver `remote` con un fetch y dos push (a veces se lista repetido para push).

#### 3.2 Push a ambos

```bash
git push remote main
```

Cuándo usarlo:
- Cuando GitHub y GitLab deben estar siempre al día a la vez.
- Cuando estás haciendo un “mirror” operativo (sin querer clonar repos duplicados).

> Nota: Si uno de los push falla (permisos, protección de ramas, etc.), Git te lo dirá.  
> El otro puede haber subido bien, así que revisa la salida del comando. 👀

---

## 🔁 4) Sincronizar ramas (que GitHub y GitLab queden iguales)

La sincronización real se reduce a esto: empujar las mismas ramas a ambos remotos.

### 4.1 Sincronizar una rama concreta (ej. `main`)

A GitHub:

```bash
git push github main
```

A GitLab:

```bash
git push gitlab main
```

A ambos (si configuraste `remote` con dos push URLs):

```bash
git push remote main
```

### 4.2 Sincronizar TODAS las ramas locales al remoto

A GitHub:

```bash
git push github --all
```

A GitLab:

```bash
git push gitlab --all
```

---

## ✅ Mini checklist visual

- [ ] Tengo `remote`, `github` y `gitlab` creados.  
- [ ] Sé cuándo hacer push a uno u otro.  
- [ ] Puedo hacer push a ambos con `git push remote main`.  
- [ ] Mis ramas están sincronizadas en los dos remotos.

A ambos:

```bash
git push remote --all
```
Qué hace --all:

Empuja todas las ramas locales (no tags).

Muy útil cuando estás arrancando el espejo por primera vez.

4.3 Sincronizar tags (recomendado si haces releases)
Tags NO se incluyen con --all. Para tags:

A GitHub:

```bash
git push github --tags
```
A GitLab:

```bash
git push gitlab --tags
```
A ambos:

```bash
git push remote --tags
```
4.4 Sincronización completa típica (ramas + tags)
Para dejar ambos remotos iguales, normalmente haces:

```bash
git push remote --all
git push remote --tags
```

5) Buenas prácticas y “gotchas” típicos
Protección de ramas
Si main está protegida (muy común), puede que:

GitHub permita push solo vía PR, y GitLab también.

Entonces el push directo fallará en uno o ambos.

Solución: empuja una rama de feature y abre PR/MR en cada plataforma, o decide una sola como “fuente de verdad”.

Fuente de verdad

Decide si:

GitHub es fuente de verdad y GitLab es mirror, o al revés.

Evita tener cambios divergentes en ambos (por ejemplo merges independientes en cada uno).
Si lo haces, tendrás conflictos o historiales distintos.

Empujar “a ambos” es potente… y peligroso
Usa remote (push doble) cuando tengas claro que:

mismas reglas (protecciones) aplican,

y quieres sincronía permanente.

Si no, usa pushes separados (github / gitlab) para controlar el momento.

6) Checklist rápido (procedimiento operativo)
Setup (una sola vez)
Crear remotos github y gitlab

Configurar remote para push a ambos

Operación diaria
Solo GitHub:

```bash
git push github <rama>

```
Solo GitLab:

```bash
git push gitlab <rama>

```
Ambos:

```bash
git push remote <rama>

```
Sincronizar todo

```bash
git push remote --all
git push remote --tags
```

7) Verificación final: comprobar que todo está alineado
Lista las ramas remotas que ve tu local:

```bash
git fetch --all --prune
git branch -r
```

Deberías ver algo parecido a:

```bash
github/main
gitlab/main
...
```

Si ves diferencias (ramas que existen en un remoto y no en el otro), repite el push --all al que falte.

Conclusión
Con esta configuración:

github y gitlab te dan control fino (push selectivo).

remote te permite un “push único” a ambos cuando estás seguro de que deben ir sincronizados.

Para sincronía completa: --all + --tags.
