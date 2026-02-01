---
layout: post
title: "aws-bash-toolbox (AWS CLI + SSO + SSM)"
date: 2026-01-30
categories: [DevOps, AWS, CLI]
tags: [AWS, AWS CLI, SSO, SSM, IAM Identity Center, Bash, fzf]
description: "Toolkit en Bash para cambiar contexto de AWS, sesiones SSM y port-forwarding sin consola."
---

# aws-bash-toolbox (AWS CLI + SSO + SSM) 🧰

Estado: **alpha** (se permiten cambios rompientes).

Un toolkit en **Bash** que ofrece un flujo estilo `kubectx/kubens` para AWS:

- Contexto activo: `AWS_PROFILE` + `AWS_REGION` persistentes
- Cambio rápido: `abt change ...` / `abt select context` (con `fzf`)
- Listado EC2 sin consola: `abt list ec2`
- Conexión por SSM Session Manager: `abt connect ssm` / `abt select ssm`
- Port forwarding por SSM a servicios privados: `abt connect forward` / `abt select forward`
- Workaround para VPN/proxy corporativo: SSM ignora proxy solo durante sesiones
- Utilidades: `abt list profiles/regions`, `abt show identity`, `abt test doctor`, `abt sso login`

Repositorio: `https://github.com/sondosclick/aws-bash-toolbox.git`

---

## Requisitos

- Linux (probado en Ubuntu 24.04)
- Bash
- AWS CLI v2
- Acceso a IAM Identity Center (SSO)

Instalar dependencias:

```bash
sudo apt update
sudo apt install -y fzf session-manager-plugin
```

Verificar:

```bash
aws --version         # should be aws-cli/2.x
session-manager-plugin
```

---

## Convención de nombres de perfiles (recomendada)

Usamos:

**`corp-<env>-<role>`**

Ejemplos:

- `corp-dev-readonly`
- `corp-uat-admin`
- `corp-pro-support`

Dónde:

- `<env>`: entorno objetivo (p. ej., `dev`, `uat`, `pro`)
- `<role>`: permission set / rol en IAM Identity Center (p. ej., `admin`, `readonly`, `support`)

> Si tu equipo usa otros nombres (`sandbox`, `np`, `prod`, etc.), ajusta según sea necesario.
> Lo importante es la consistencia.

---

## 1) Configurar AWS SSO (perfiles)

Este toolkit asume perfiles SSO en `~/.aws/config`.

### 1.1 Crear/validar la sesión SSO

En `~/.aws/config`:

```ini
[sso-session corp-sso]
sso_start_url = https://<YOUR_START_URL>/start
sso_region = eu-central-1
sso_registration_scopes = sso:account:access
```

Importante: usa el Start URL **sin** `/#/` para evitar problemas de token/cache.

### 1.2 Crear perfiles por cuenta y rol (permission set)

Ejemplos completos:

```ini
[profile corp-dev-admin]
sso_session = corp-sso
sso_account_id = 111111111111
sso_role_name = admin
region = eu-west-3
output = json

[profile corp-uat-readonly]
sso_session = corp-sso
sso_account_id = 222222222222
sso_role_name = readonly
region = eu-west-3
output = json

[profile corp-pro-support]
sso_session = corp-sso
sso_account_id = 333333333333
sso_role_name = support
region = eu-west-3
output = json
```

Nota: para SSO el patrón correcto es `sso_session + sso_account_id + sso_role_name`.
Evita `source_profile + role_arn` salvo que tu organización requiera AssumeRole clásico.

### 1.3 Opcional: asumir un rol extra después del login SSO (SSM full access)

Si tu organización requiere un **segundo rol** después del SSO (típico en accesos elevados),
crea un **perfil encadenado** usando `source_profile` y `role_arn`.

Ejemplo: asumir un rol SSM full-access **después** de iniciar sesión con un
perfil SSO que termina en `assumerole`:

```ini
[profile corp-br-np-assumerole]
sso_session = corp-sso
sso_account_id = 418272778142
sso_role_name = AssumeRole
region = eu-west-3
output = json

[profile corp-br-np-ssmfullaccess]
role_arn = arn:aws:iam::418272778142:role/ssmfullcaccess
source_profile = corp-br-np-assumerole
role_session_name = your-user
region = eu-west-3
```

Flujo ASCII:

```
You
  │
  ├─(SSO login)─> corp-br-np-assumerole
  │                (Identity Center permission set)
  │
  └─(AssumeRole)→ corp-br-np-ssmfullaccess
                   (ssmfullcaccess)
```

Cuándo usar cada uno:

- Usa el perfil `assumerole` **solo para loguear** en SSO.
- Usa el perfil `ssmfullaccess` **para comandos que requieran el rol extra**.

Uso simple:

```bash
# 1) Loguear con la sesión SSO
abt sso login corp-sso

# 2) Cambiar al perfil del rol derivado
abt change profile corp-br-np-ssmfullaccess
aws sts get-caller-identity
```

Si tienes muchos perfiles `*-assumerole`, repite el segundo bloque para cada uno,
manteniendo `source_profile` apuntando a su base y el mismo patrón de `role_arn`
(`ssmfullcaccess`).

---

## 2) Login SSO

Si cambiaste perfiles o viste errores:

```bash
rm -rf ~/.aws/sso/cache/*
```

Login:

```bash
aws sso login --sso-session corp-sso
```

Verificar:

```bash
aws sts get-caller-identity --profile corp-dev-admin --region eu-west-3
```

---

## 3) Instalar el toolbox

### Método A) Instalación automática (recomendado)

```bash
git clone https://github.com/sondosclick/aws-bash-toolbox
cd aws-bash-toolbox
./install.sh
source ~/.bashrc
```

Qué hace `install.sh`:

- copia `abt.sh` a `~/.abt/abt.sh`
- agrega una línea en `~/.bashrc`:

```bash
source "$HOME/.abt/abt.sh"
```

No sobrescribe tu bashrc y es seguro ejecutarlo varias veces.

### Método B) Manual (copy/paste)

1. Clonar el repo:

```bash
git clone https://github.com/sondosclick/aws-bash-toolbox
```

2. Copiar `abt.sh` a `~/.abt`:

```bash
mkdir -p "$HOME/.abt"
cp abt.sh "$HOME/.abt/abt.sh"
```

3. Editar tu `~/.bashrc` y agregar al final:

```bash
source "$HOME/.abt/abt.sh"
```

4. Recargar:

```bash
source ~/.bashrc
```

---

## 4) Configuración opcional (abt)

El toolbox puede leer un archivo opcional: `~/.abt/abt.env` (formato shell).

Crear plantilla:

```bash
abt config init
```

Forzar overwrite:

```bash
abt config init --force
```

Variables disponibles (ejemplo):

```bash
ABT_DEFAULT_PROFILE="corp-base"
ABT_DEFAULT_REGION="eu-central-1"
ABT_REGIONS="eu-central-1 eu-west-1 eu-west-3 us-east-1 us-west-2"
ABT_COLOR=1
```

- `ABT_DEFAULT_*`: se usa cuando no existe contexto persistido.
- `ABT_REGIONS`: lista separada por espacios para los selectores.
- `ABT_COLOR=0`: desactiva colores.

Ver configuración actual:

```bash
abt show config
```

---

## 5) Uso rápido

Formato: `abt <verbo> <objeto>`

```bash
abt show context
abt show profile
abt show region
abt show identity
abt show version
abt export context
abt list profiles
abt list regions
abt change profile corp-uat-readonly
abt change region eu-west-3
abt change context corp-uat-readonly eu-west-3
abt select context
abt select profile
abt select region
abt list ec2
abt connect ssm i-0123456789abcdef0
abt connect ssm -n api-01
abt connect forward i-0123456789abcdef0 db.internal 5432
abt connect forward -n bastion-01 db.internal 5432 15432
abt select ssm
abt select forward
abt sso login
abt test sts
abt test sts eu-west-3 eu-central-1
abt test doctor
abt config init
abt show config
```

Tip: `abt export context` es muy útil para scripts:

```bash
source <(abt export context)
```

---

## 6) Port forward a servicios privados (SSM)

Usa una instancia gestionada por SSM (bastion) como túnel desde tu laptop a un
servicio privado (RDS, Redis, API interna, etc.). Esto evita SSH y funciona con
IAM Identity Center.

Iniciar una sesión de port-forward:

```bash
# Forward local 15432 -> db.internal:5432 via bastion
abt connect forward -n bastion-01 db.internal 5432 15432

# Si omites el puerto local, usa el remoto por defecto
abt connect forward i-0123456789abcdef0 db.internal 5432
```

Flujo interactivo (fzf) con selector de puertos comunes de DB:

```bash
abt select forward
```

Tip: agrega un tag en el bastion para prellenar el host remoto:

- `ForwardHost` (preferido)
- `DBHost`, `DbHost`, `DBEndpoint`, `RDSEndpoint`, `RDSHost`, `db_host`

Luego conecta tu cliente (p. ej., DBeaver) a:

```
Host: 127.0.0.1
Port: 15432   # o 5432 si no sobrescribiste el puerto local
```

Ejemplo: RDS (PostgreSQL)

```bash
abt connect forward -n bastion-01 mydb.abc123.eu-west-3.rds.amazonaws.com 5432 15432
```

Reglas de security group para que funcione:

- RDS SG inbound: permitir 5432 desde el SG de la instancia bastion.
- Bastion SG outbound: permitir 5432 hacia el SG de RDS (si el egress es restrictivo).
- Requisitos SSM: instancia con SSM agent + IAM role con `AmazonSSMManagedInstanceCore`.

Notas:

- Mantén el terminal abierto mientras la sesión esté activa (Ctrl+C para parar).
- La instancia debe estar gestionada por SSM y poder llegar al host/puerto remoto.

---

## FAQ

### "Error loading SSO Token: Token for <start_url> does not exist"

Solución:

```bash
rm -rf ~/.aws/sso/cache/*
aws sso login --sso-session corp-sso
```

Verifica que tus perfiles usen `sso_session = corp-sso`.

---

### `Cannot perform start session: EOF`

Probable proxy/VPN corporativo. Este toolbox ignora proxy solo para SSM, pero si sigue fallando:

- prueba sin VPN / otra red (si es posible)
- inspecciona variables de proxy:

```bash
env | grep -i proxy
```

---

### `session-manager-plugin: command not found`

Instalar:

```bash
sudo apt update
sudo apt install -y session-manager-plugin
```

---

### No aparecen instancias en `abt select ssm`

Revisar SSM Agent / permisos / conectividad:

```bash
aws ssm describe-instance-information \
  --profile <profile> --region <region> \
  --query 'InstanceInformationList[].{Id:InstanceId,Status:PingStatus,Agent:AgentVersion}' \
  --output table
```

---

## Archivos del repo

- `abt.sh` -> funciones Bash (toolbox)
- `install.sh` -> instalador (agrega `source ...` a `~/.bashrc`)
- `README.md` -> documentación

---

## Licencia

GPL-3.0-only. Ver `LICENSE`.
