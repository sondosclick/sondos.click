---
layout: post
title: "Instalar la Root CA de Zscaler en openSUSE Tumbleweed (edición WSL)"
date: 2026-01-01
categories: [Linux, openSUSE, Redes]
tags: [Zscaler, Certificados, openSUSE, WSL, Solución de problemas]
description: "Cómo instalar el certificado CA de Zscaler en openSUSE Tumbleweed bajo WSL — para que por fin dejen de gritar tus errores SSL."
featureimage: "/images/posts/2026-01-01-zscaler-root-ca-opensuse-wsl/feature.svg"
---

# Instalar la Root CA de Zscaler en openSUSE Tumbleweed (edición WSL)

![Ilustración de confianza de certificados y rutas de red](/images/posts/2026-01-01-zscaler-root-ca-opensuse-wsl/feature.svg)

Ah, **Zscaler** — nuestro “amigo” corporativo favorito, al que le encanta jugar a intermediario entre nosotros e internet.  
Porque, claro, ¿a quién *no* le gusta la interceptación HTTPS a las 9 AM de un lunes? 😅

Si estás corriendo **openSUSE Tumbleweed** en **WSL** y de repente todo se rompe — `curl`, `wget`, `zypper`, incluso `git` — ¡felicidades! Has conocido la inspección SSL de Zscaler. Vamos a arreglarlo para que puedas volver al trabajo real (y quizá a quejarte un poco).

---

## ⚙️ Paso a paso: hacer que openSUSE confíe en Zscaler

openSUSE usa **p11-kit** y el sistema `update-ca-certificates` para gestionar raíces de confianza.  
Te muestro dos formas sencillas de instalar ese certificado CA corporativo (antes de que Zscaler vuelva a arruinarte el día).

---

### 🧙‍♂️ Opción 1: usa `trust anchor` (recomendada)

1. **Descarga el certificado Root CA de Zscaler**

   Búscalo en tu SharePoint corporativo o en el portal de administración de ZIA (suele ser un `.pem` o `.crt`).  
   Enlace de ejemplo (ficticio):  
   [Zscaler_Root_CA.pem](https://example.com/sites/Zscaler/Zscaler_Root_CA.pem)

2. **Añádelo al almacén de confianza del sistema**

    ```bash
   sudo trust anchor ~/Zscaler_Root_CA.pem
    ```

3. **Verifica que funcionó**

    ```bash
    trust list | grep Zscaler
    ```

    Si ves el certificado listado como *anchor* — estás listo.

### 🧰 Opción 2: copiar y actualizar manualmente

1. Mueve el certificado al directorio de *anchors*

    ```bash
    sudo cp Zscaler_Root_CA.pem /etc/pki/trust/anchors/
    ```

    (También puedes usar /usr/share/pki/trust/anchors/ para confianza a nivel de sistema.)

2. Regenera el almacén de CA

```bash
sudo update-ca-certificates
```

3. Celebra. Quizá con sarcasmo.
Tus herramientas de Linux deberían dejar de quejarse de errores de “unknown issuer”. 🎉

----

### 🕵️‍♀️ Por qué esto importa

Zscaler intercepta el tráfico HTTPS, lo re-firma con su propio certificado raíz y finge que te está haciendo un favor.  
Si no confías en ese CA, cada conexión segura falla miserablemente.  
Añadir el cert hace que tus herramientas — curl, wget, zypper, pip, etc. — vuelvan a comportarse.

### 🔍 Notas enterprise

- La documentación interna suele recomendar copiar el cert a /etc/ssl/certs/ca-certificates.crt o automatizarlo con scripts.

- Para contenedores, Git o Python, quizá tengas que añadir el cert a sus propios *CA bundles* manualmente.

> (Sí, ni Docker escapa al alcance de Zscaler. 🧟‍♂️)

### ✅ Comprobación rápida de confianza

Ejecuta:
```bash
openssl s_client -connect example.com:443 -showcerts
```

Si no ves “unknown issuer”, felicidades — tu sistema ya confía en Zscaler (a regañadientes).

### 🧠 Ideas clave

- Zscaler rompe SSL. Nosotros arreglamos SSL. Círculo de la vida.
- Usa `sudo trust anchor` para la instalación más limpia en openSUSE.
- No olvides verificar — *trust, but verify* (literalmente).
- Opcional: quejarte a IT por desplegar Zscaler en primer lugar. 😉

### 🔗 Enlaces útiles

- Documentación de gestión de certificados en openSUSE: https://en.opensuse.org/SDB:Administration_of_Trusted_CAs

- Portal de soporte de Zscaler: https://help.zscaler.com/ (para cuando necesites llorar en corporativo)
