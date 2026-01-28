---
layout: post
title: "Estenografía: el arte de esconder texto (y de encontrarlo) en internet"
date: 2026-01-08
categories: [Tecnología, Cultura, Internet]
tags: [Estenografía, Seguridad, CTF, Imágenes, Metadatos]
description: "Qué es la estenografía, por qué importa en la cultura digital y cómo usarla para ocultar y extraer texto en imágenes."
featureimage: "/images/posts/2026-01-08-estenografia-el-arte-de-esconder-texto/feature.svg"
draft: false
---

# Estenografía: el arte de esconder texto (y de encontrarlo) en internet

![Ilustracion sobre mensajes ocultos en imagenes](/static/images/posts/2026-01-08-estenografia-el-arte-de-esconder-texto/feature.svg)

Sí, estenografía. No, no es un curso secreto de escritura ultrarrápida. (Eso es *taquigrafía*).
Esto va de esconder mensajes dentro de cosas que parecen inofensivas: imágenes, audios, PDFs… lo que tengas a mano. 
Básicamente es el primo hacker del “te lo digo al oído”.

## ¿Qué es la estenografía?
Es la técnica de **ocultar información dentro de otros archivos**. A simple vista todo parece normal, pero hay algo extra escondido. Como cuando subes una foto de tu gato y en realidad lleva un manifiesto secreto en los píxeles. Cute y subversivo.

## Relevancia histórica (porque sí, esto ya existía antes de los memes)
La estenografía es vieja, viejísima. Se usaba en guerras para pasar mensajes sin levantar sospechas. Y ahora se usa en internet para cosas igual de serias… como meter rickrolls en PNGs. Evolución cultural, señores.

## Estenografía en internet: cultura, memes y conspiraciones
En el mundillo online, la estenografía es:
- **un juguete**: para esconder easter eggs en imágenes o música.  
- **un reto**: en CTFs y ARGs, donde buscar mensajes ocultos es literalmente el juego.  
- **un arma (a veces)**: porque también puede usarse para ocultar malware. Sí, la diversión se acaba aquí.  
- **un guiño cultural**: “si sabes, sabes”.  

Es como un club secreto, pero con menos batas y más Discord.

## Cómo configurarla y usarla (versión práctica y sin drama)

### 1) Elige tu herramienta

Para imágenes, lo más común es **steghide** o **zsteg** (para PNGs) y **stegsolve** (para visualizar canales de color). Tú decides tu nivel de sufrimiento.

### 2) Instalar (ejemplo rápido)

Si usas Linux:
```bash
sudo apt install steghide
```
### 3) Ocultar un texto dentro de una imagen

```bash
steghide embed -cf imagen.jpg -ef mensaje.txt
```
Te pedirá una contraseña. Pon una, o no seas tibio.

### 4) Extraer/descifrar el texto oculto

```bash
steghide extract -sf imagen.jpg
```
Si está protegido, necesitarás la contraseña. Y si no la tienes… buena suerte, Indiana Jones.

¿Y cómo “descifro” textos dentro de imágenes?
Depende del método usado. A veces el texto está escondido en:

* los bits menos significativos (LSB)
* un canal de color específico
* metadatos del archivo
* o simplemente un “truco visual” (contraste + zoom + magia)

Herramientas útiles:

* zsteg (detecta datos en PNG)
* stegsolve (para inspeccionar canales)
* exiftool (para metadatos)

Y si lo que buscas es un mensaje que “se ve” (pero no se ve), prueba esto:

* sube brillo
* baja contraste
* revisa canales RGB
* En otras palabras: ajusta la imagen como si fueras influencer, pero para detectives.

Final con moraleja

La estenografía es el arte de esconder cosas a plena vista.

En internet, eso la convierte en meme, reto, herramienta creativa y ocasionalmente, en sospecha.

¿Mi consejo? Úsala para cosas divertidas.
¿Mi advertencia? Si alguien dice “hay un mensaje oculto”, prepárate para perder tres horas de tu vida.

Bienvenido a l diversión.
