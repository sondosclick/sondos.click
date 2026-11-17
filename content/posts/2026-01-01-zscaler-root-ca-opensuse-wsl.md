---
layout: post
title: "Instalar la CA raíz de Zscaler en openSUSE Tumbleweed (edición WSL)"
date: 2026-01-01
categories: [Linux, openSUSE, Redes]
tags: [Zscaler, Certificados, openSUSE, WSL, Resolución de problemas]
description: "Cómo instalar el certificado CA de Zscaler en openSUSE Tumbleweed bajo WSL para que los errores SSL dejen de gritarte de una vez."
featureimage: "/images/posts/2026-01-01-zscaler-root-ca-opensuse-wsl/feature.svg"
---

# Instalar la CA raíz de Zscaler en openSUSE Tumbleweed (edición WSL)

![Ilustración sobre confianza de certificados y rutas de red](/images/posts/2026-01-01-zscaler-root-ca-opensuse-wsl/feature.svg)

Ah, **Zscaler**, nuestro “amigo” corporativo favorito, al que le encanta hacer de intermediario entre nosotros e internet.  
Porque a quién no le gusta la interceptación HTTPS a las 9 de la mañana de un lunes… ¿verdad? 😅  

Si estás ejecutando **openSUSE Tumbleweed** en **WSL** y de repente todo deja de funcionar —`curl`, `wget`, `zypper`, incluso `git`—, ¡enhorabuena! Acabas de conocer la inspección SSL de Zscaler. Vamos a arreglarlo para que puedas volver al trabajo de verdad (y quizá a quejarte un poco).

---

## ⚙️ Paso a paso: hacer que openSUSE confíe en Zscaler

openSUSE usa **p11-kit** y el sistema `update-ca-certificates` para gestionar las raíces de confianza.  
Te mostramos dos formas sencillas de instalar ese certificado CA corporativo (antes de que Zscaler vuelva a arruinarte el día).

---

### 🧙‍♂️ Opción 1: usar `trust anchor` (recomendada)

1. **Descarga el certificado CA raíz de Zscaler**

   Consíguelo en el SharePoint corporativo o en el portal de administración de ZIA (normalmente será un archivo `.pem` o `.crt`).  
   Enlace de ejemplo (ficticio):  
   [Zscaler_Root_CA.pem](https://example.com/sites/Zscaler/Zscaler_Root_CA.pem)

2. **Añádelo al almacén de confianza del sistema**

    ```bash
   sudo trust anchor ~/Zscaler_Root_CA.pem
    ```

3. Comprueba que funciona

    ```bash
    trust list | grep Zscaler
    ```
    
    Si ves el certificado listado como un anclaje, ¡todo perfecto!


### 🧰 Opción 2: copiar y actualizar manualmente

1. Mueve el certificado al directorio de anclajes

    ```bash
    sudo cp Zscaler_Root_CA.pem /etc/pki/trust/anchors/
    ```

    (También puedes usar `/usr/share/pki/trust/anchors/` para la confianza a nivel de sistema.)

2. Regenera el almacén de CA

```bash
sudo update-ca-certificates
```

3. Celebra. Quizá con sarcasmo.
Tus herramientas de Linux deberían dejar de quejarse de los errores de “unknown issuer”. 🎉

----

### 🕵️‍♀️ Por qué importa

Zscaler intercepta el tráfico HTTPS, lo vuelve a firmar con su propio certificado raíz y finge que te está haciendo un favor.
Si no confías en esa CA, todas las conexiones seguras fallan estrepitosamente.
Añadir el certificado garantiza que tus herramientas —`curl`, `wget`, `zypper`, `pip`, etc.— vuelvan a comportarse como deben.

### 🔍 Notas para entornos empresariales

- La documentación interna suele recomendar copiar el certificado a `/etc/ssl/certs/ca-certificates.crt` o automatizarlo mediante scripts.

- En contenedores, Git o Python, quizá tengas que añadir manualmente el certificado a sus propios almacenes de CA.

> (Sí, ni siquiera Docker escapa al alcance de Zscaler. 🧟‍♂️)

### ✅ Comprobación rápida de confianza

Ejecuta:
```bash
openssl s_client -connect example.com:443 -showcerts
```

Si no ves “unknown issuer”, enhorabuena: tu sistema ya confía en Zscaler (a regañadientes).

### 🧠 Ideas clave

- Zscaler rompe SSL. Nosotros arreglamos SSL. El ciclo de la vida.
- Usa `sudo trust anchor` para una instalación limpia en openSUSE.
- No olvides verificar: confía, pero verifica (literalmente).
- Opcional: quéjate a IT por desplegar Zscaler en primer lugar. 😉

### 🔗 Enlaces útiles

- Documentación de gestión de certificados de openSUSE: https://en.opensuse.org/SDB:Administration_of_Trusted_CAs

- Portal de soporte de Zscaler: https://help.zscaler.com/ (para cuando necesites llorar en el entorno corporativo)
