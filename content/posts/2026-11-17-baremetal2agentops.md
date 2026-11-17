---
layout: post
title: "De bare metal a agentes de IA: sesenta años de abstracciones"
date: 2026-11-17
categories: [Infraestructura, IA, Tecnología]
tags: [Cloud, Kubernetes, GitOps, Observabilidad, AgentOps, IaC, Control loops]
description: "Cómo pasamos de cambiar discos a declarar intenciones… sin que los discos desaparecieran."
featureimage: "/images/posts/2026-11-17-baremetal2agentops/baremetal2AI.png"
---

Vamos a recorrer varias décadas de administración de sistemas en un solo post. Empezaremos con un servidor que podemos tocar y terminaremos con una IA proponiendo cambios sobre producción. Mi objetivo es convencerte de que ambas cosas están mucho más relacionadas de lo que parece.

## El patrón que lo explica todo

En cada etapa ocurre lo mismo:

> **Problema → Abstracción → Nuevo problema ↺**

Tenemos un problema, inventamos una abstracción y descubrimos un problema nuevo. La informática funciona sorprendentemente bien así. Recuerda esta estructura, porque la vamos a ver repetirse con la virtualización, el cloud, los contenedores, Kubernetes y la IA.

## 1. Spoiler: el servidor sigue existiendo

Hay una mentira útil que repetimos constantemente: *la nube*. La nube no hizo desaparecer los ordenadores. Hizo desaparecer los ordenadores de tu lista inmediata de problemas, que es muy diferente.

Debajo de un Pod hay un contenedor, debajo una VM, un hipervisor, un servidor, un rack, y debajo de todo eso, energía y refrigeración. **Abstracción ≠ desaparición.**

Y lo que llamamos «la nube» es, en esencia, cómputo, red, almacenamiento, energía y refrigeración a los que alguien ha puesto una API delante: alimentación A/B, red redundante, discos, sensores, ventiladores, pasillos fríos y calientes. *Serverless* no significa que no haya servidores, sino que el servidor es problema de otra persona. Alguien recibe una caja pesada, la mete en un rack y conecta fuentes, red y gestión. Cuando un SSD muere, alguien acaba cambiándolo.

Nosotros vamos a empezar siendo esa persona.

## 2. Bare metal: `web01` y el problema de la escala

Primer servidor: `web01`. Una máquina física con Linux, nginx, nuestra aplicación, ficheros de configuración, cuentas locales, SSH y un agente de monitorización. Lo instalamos todo a mano, y funciona. *«Está hecho.»* Famous last words.

Porque negocio acaba de pedir otro. Y luego otro. ¿Cómo hacemos que `web02` sea exactamente igual que `web01`? ¿Copiamos documentación? ¿Clonamos un disco? ¿Confiamos en la memoria? Aquí nace la necesidad de imágenes, scripts y configuración reproducible.

## 3. Virtualización: un ordenador que finge ser muchos

La primera gran abstracción fue virtualizar el hardware. Un hipervisor abstrae CPU, memoria, red y almacenamiento para ejecutar varios sistemas operativos aislados sobre la misma máquina. Ganamos consolidación, *snapshots* y aprovisionamiento más rápido.

| Problema | Abstracción | Nuevo problema |
|---|---|---|
| Hardware infrautilizado | Virtualización | Caos de VM, redes virtuales, almacenamiento centralizado |

Como siempre: resolvimos un problema y creamos otros. Además, virtualizar no evita comprar, recibir e instalar servidores.

## 4. Cloud / IaaS: ¿y si el servidor fuese una API?

El cambio importante no es mover los servidores lejos. Es convertir cómputo, red y almacenamiento en recursos que se piden mediante una API. Antes había un proceso logístico (pedido, caja, rack, cables, instalación) que tardaba **semanas**. Ahora hay una llamada que tarda **segundos o minutos**:

```yaml
POST /servers
cpu: 4
memory: 16 GB
disk: 100 GB
network: production
```

Tomemos EC2 como ejemplo, sin convertir esto en un tutorial de AWS. Sigue siendo un ordenador: servidor → *instance*, disco → *volume*, red → *VPC / subnet*, firewall → *security group*, imagen → *AMI*. **La interfaz cambia; los fundamentos, no.** EC2 no es la nube completa: es un servidor virtual gestionado mediante API.

Y una API nos permite crear 200 servidores demasiado rápido para configurarlos a mano. Cloud solucionó el problema de conseguir infraestructura y convirtió inmediatamente la configuración manual en un problema mucho mayor.

## 5. Automatización: del *cómo* al *cómo debería estar*

La automatización no empieza con una gran plataforma. Empieza cuando te cansas de repetir el mismo comando. Si lo haces dos veces, probablemente deberías automatizarlo. Si lo haces 200 veces a mano, tenemos que hablar.

### Imperativo: el primer superpoder

```bash
# setup.sh
apt install -y nginx
echo "listen 8080;" >> app.conf
systemctl enable nginx
systemctl restart nginx
```

Le decimos al ordenador exactamente qué pasos ejecutar. Es sencillo y poderoso, pero depende del estado inicial. ¿Qué ocurre si lo ejecutas dos veces?

```text
$ cat app.conf
listen 8080;
listen 8080;
$ systemctl restart nginx
ERROR: directiva "listen" duplicada
nginx no arranca
```

### Declarativo: dime cómo debería estar

```yaml
nginx:
  installed: true
  enabled: true
  port: 8080
  running: true
```

Pasamos de describir acciones a describir **intención**. No quiero que ejecutes tres comandos: quiero que el paquete esté instalado y el servicio activo. Guarda estas dos palabras: *desired state*.

### Herramientas distintas, estados distintos

- **Ansible**: configura y orquesta, normalmente sin agente.
- **Puppet**: mantiene la configuración de forma continua (agente + catálogo).
- **Terraform**: aprovisiona infraestructura (plan + estado).

Se solapan. La pregunta útil no es cuál gana, sino **qué estado estás intentando controlar** y qué ciclo de vida gestionas.

### Everything as Code (y por qué)

Una vez que empezamos a describir cosas, nos emocionamos: aplicación, infraestructura, configuración, políticas, seguridad, observabilidad, dashboards, alertas, pipelines, *runbooks*. Todo como artefactos versionables.

¿Qué aporta realmente el sufijo *as Code*?

- **Version**: cada cambio con historial.
- **Review**: otra persona lo revisa.
- **Test**: se prueba antes de aplicar.
- **Reproduce**: mismo resultado cada vez.
- **Automate**: sin pasos manuales.
- **Audit**: quién cambió qué y cuándo.

Que algo esté escrito en YAML no significa automáticamente que sea ingeniería. **El objetivo no es escribir más YAML, es hacerlo con sentido y propósito.**

## 6. Contenedores: «en mi máquina funciona»

Desarrollo tiene una librería, producción otra; el *runtime* cambia; la configuración también. ¿Y si pudiésemos empaquetar aplicación, dependencias y configuración de forma reproducible?

Eso no crea una VM pequeña. Un contenedor es **un proceso aislado que comparte kernel con el host**. Los *namespaces* aportan aislamiento; los *cgroups*, límites y contabilidad. Una imagen no es un contenedor, y un contenedor no es una frontera de seguridad absoluta.

## 7. Kubernetes: un control loop

Fantástico, ahora tenemos 200 contenedores. ¿Dónde se ejecutan? ¿Cuántos hay? ¿Qué pasa si mueren? ¿Cómo se encuentran entre sí? ¿Dónde están los secretos? ¿Y los datos? Necesitamos un sistema que esté mirando continuamente.

Kubernetes es una plataforma para ejecutar sistemas distribuidos de contenedores, pero lo más útil es verlo como un conjunto de *control loops* que mantienen el estado deseado: **declarar, observar, reconciliar**.

Del YAML al Pod, en seis pasos:

1. `kubectl apply` («quiero 3 réplicas»).
2. El **API server** recibe y valida.
3. **etcd** guarda lo deseado.
4. Los **controllers** comparan: ¿hay 3? Faltan.
5. El **scheduler** elige nodo.
6. El **kubelet** del nodo arranca y vigila los Pods.

Y si el estado real no coincide con el deseado, vuelta al paso 4. No memorices las cajas del diagrama: entiende el mecanismo.

Un ejemplo mínimo: declaramos `spec.replicas: 3`, el sistema observa 2, el controlador no recuerda una secuencia, compara nivel deseado y observado, y crea un reemplazo. Kubernetes no ejecuta un guion una vez: **persigue convergencia**.

### Demo: matemos un Pod

```text
$ kubectl get pods -w
pod-1   Running
pod-2   Running
pod-3   Running
$ kubectl delete pod pod-2
pod-4   Creating…
```

Kubernetes no quiere ese Pod. Quiere **3 Pods**.

## 8. Si todo es código… ¿dónde vive la verdad?

¿En producción? ¿En la documentación? ¿En Git? ¿En la CMDB? La palabra *verdad* es peligrosa; mejor hablemos de **intención declarada**.

Mira el *drift*:

| | Git | Producción |
|---|---|---|
| replicas | 3 | 5 |
| version | 2.1 | 2.0 |
| region | eu-west | ??? |

¿Cuál tiene razón? Antes, para saber cómo estaba producción, entrábamos en producción. Ahora queremos que tener que entrar para descubrir la configuración sea un fallo del proceso. Git no es mágicamente la realidad: es la fuente declarativa de intención que hemos decidido gobernar.

**GitOps** añade un controlador que observa Git y el entorno real y, si divergen, intenta reconciliarlos. No espera necesariamente a que un pipeline empuje comandos con credenciales de producción. Git expresa intención; el controlador persigue convergencia.

## 9. ¿Te suena este patrón?

**Declarar → Observar → Comparar → Reconciliar**, de forma continua. Kubernetes lo aplica a las réplicas; GitOps, al estado declarado en Git. ¿Qué otro sistema declara, observa y corrige? Guarda ese hueco, que volveremos a él.

Antes, bajemos por debajo de todo esto. ¿Dónde está realmente ese Pod? Pod → contenedor → proceso Linux → kernel → VM → hipervisor → servidor físico → rack → switch/fibra → energía → refrigeración. **Kubernetes no abolió Linux. Cloud no abolió los servidores.** Lo que cambia es quién necesita pensar directamente en cada capa.

## 10. Observabilidad: todo está verde, los usuarios dicen que no

Esta frase resume aproximadamente la mitad de los incidentes interesantes: todos los *health checks* locales están verdes, pero la experiencia extremo a extremo está rota. Necesitamos señales distintas que respondan preguntas distintas:

- **Métricas**: ¿qué está pasando?
- **Trazas**: ¿dónde?
- **Logs**: ¿qué ocurrió?
- **Cambios**: ¿qué acabamos de tocar?

Todas correlacionadas por un mismo contexto (por ejemplo, un *trace ID*). Es una simplificación pedagógica, pero útil.

Un ejemplo: una petición tarda 8 segundos. El *frontend* tarda 120 ms, la API 180 ms, la base de datos 90 ms… y *payments* 7,4 s. Con una métrica sabes el alcance y desde cuándo; con la traza localizas *payments*; con los logs ves el detalle; con los cambios compruebas si hubo un despliegue reciente. **Observabilidad no es hacer dashboards: es poder hacer preguntas que no habías previsto.**

## 11. Resiliencia: ¿y si desaparece una región?

Con el tráfico global conmutando de la región A a la B, aparecen dos preguntas:

- **RPO**: ¿cuántos datos puedo perder?
- **RTO**: ¿cuánto tardo en recuperar el servicio?

Multirregión implica tráfico, datos, identidad y dependencias, no copiar dos clusters. Y no significa invulnerable: ganamos disponibilidad y tolerancia a fallos; compramos coste, latencia, replicación, consistencia y complejidad. Dos regiones mal diseñadas pueden darte el doble de infraestructura y el mismo punto único de fallo. **La resiliencia también tiene factura.**

## 12. La IA entra en escena

Ahora tenemos demasiadas señales: *metrics, logs, traces,* Git, despliegues, alertas, tickets, cloud, Kubernetes. Es el *context overload*. La IA empieza a tener un caso de uso interesante no porque sepa operar por arte de magia, sino porque puede reunir todo ese contexto más rápido que una persona durante un incidente.

Pero correlacionar no es lo mismo que tener autoridad para actuar. Así que, ¿le darías `sudo` a una IA?

```text
$ sudo ai-agent █
```

La respuesta prudente es no. Un modelo probabilístico no debería convertirse mágicamente en nuestro sistema de autorización. La alternativa es encerrar sus propuestas dentro de controles deterministas: **AgentOps**.

> **Observar → Analizar → Proponer → Política → ¿Aprobar? → Actuar → Verificar**

Razonamiento probabilístico **dentro de** controles deterministas: mínimo privilegio, *blast radius* acotado, auditoría, *rollback* y *kill switch*. Si la verificación falla, rollback y a observar de nuevo. La autorremediación segura no es darle root al modelo; es encerrar una propuesta probabilística dentro de controles deterministas.

Y mira: la IA cumple el patrón del principio. El problema era el exceso de señales; la abstracción, los agentes que correlacionan contexto y proponen acciones; el nuevo problema, la autoridad, los errores plausibles y la auditoría. **La IA no rompe el ciclo: lo cierra.** Era el hueco de los interrogantes.

## El ciclo completo

| Problema | Abstracción | Nuevo problema |
|---|---|---|
| Hardware infrautilizado | Virtualización | VM sprawl, red, storage |
| Servidores que tardan semanas | Cloud / IaaS | 200 servidores sin configurar |
| Configuración manual a escala | IaC | Apps que no viajan entre entornos |
| «En mi máquina funciona» | Contenedores | 200 contenedores que orquestar |
| Orquestar y evitar el drift | Kubernetes · GitOps | Demasiadas señales |
| Context overload | IA · AgentOps | Autoridad, confianza, auditoría |

**Cada abstracción abre el siguiente ciclo.**

## Cierre

La historia no era *cloud*. Era **control loops**: expresar intención, observar la realidad y reducir la diferencia. IaC, Kubernetes, GitOps, *autoscaling*, *self-healing*, AgentOps… no son exactamente el mismo mecanismo, pero comparten una idea. La evolución no fue simplemente hacia el cloud: fue hacia sistemas que gestionan estado de manera continua.

Y aun así, las abstracciones no flotan; descansan unas sobre otras: agente de IA, GitOps, Kubernetes, contenedor, Linux, VM, hipervisor, servidor, rack. Y debajo de absolutamente todo… hay un ordenador. Tiene CPU, memoria, discos y red. Consume electricidad, genera calor y se rompe. Y hay una persona que, en algún lugar, acaba cambiando ese disco.

Las herramientas que usarás dentro de diez años probablemente no sean estas. Pero procesos, memoria, red, almacenamiento, concurrencia, consistencia, seguridad y *debugging* seguirán ahí.

> Cuantas más abstracciones conozcas, más arriba podrás trabajar. Cuantos más fundamentos entiendas, más abajo podrás llegar cuando todo falle.

*Roque Rivera García, Technical Manager en NTT DATA.*
