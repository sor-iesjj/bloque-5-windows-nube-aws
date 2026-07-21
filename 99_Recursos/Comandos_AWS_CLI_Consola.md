# ☁️ Comandos de AWS CLI / Consola EC2

Esta guía no existe en BoochanV1.1 (Hyper-V local), porque allí no hay ningún proveedor cloud de por medio. En **BoochanV3.1 tú eres quien lanza, configura y administra recursos reales de AWS** — una instancia EC2, un Security Group, una Elastic IP, un Key Pair — dentro del **AWS Academy Learner Lab**, así que necesitas dominar tanto la Consola de AWS (interfaz web) como, opcionalmente, la AWS CLI (línea de comandos). Úsala como referencia rápida a lo largo de la Fase 1 (lanzar la instancia y el Security Group), la Fase 3/4 (ampliar el Security Group) y la Auditoría Final (cierre de puertos y End Lab).

> [!info] Consola vs. CLI: dos caminos, mismo resultado
> Todas las acciones de esta guía se explican primero por la **Consola de AWS** (la web `EC2`), la vía que sigue el manual paso a paso en las Fases 1-8, porque es más visual para quien administra AWS por primera vez. Cuando existe, se indica también el comando equivalente de **AWS CLI** (`aws`), útil si tu profesor te pide automatizar tareas repetitivas. En el Learner Lab, la AWS CLI usa credenciales temporales que caducan al terminar la sesión (`End Lab`) — se toman del apartado "AWS Details → CLI" del propio laboratorio.

> [!warning] ⚠️ El ciclo Start Lab / End Lab manda sobre todo lo demás
> A diferencia de una cuenta AWS normal, el Learner Lab se enciende con **Start Lab** (círculo que pasa de rojo a verde) y se congela con **End Lab**. Mientras está en `End Lab`, la instancia no consume crédito. **Usa siempre End Lab al terminar cada sesión** — es el equivalente al "Detener/Desasignar" de Azure, pero más contundente: congela todo el entorno de golpe.

---

## 🏗️ 1. Instancia EC2

### Lanzar la instancia `WindowsServer`
> **Ruta de la Consola:** Servicio `EC2` → `Instancias` → `Lanzar instancias`

> [!example] Valores usados en BoochanV3.1 (Fase 1)
> | Campo | Valor |
> | :--- | :--- |
> | Nombre (etiqueta Name) | `WindowsServer` |
> | AMI | `Windows Server 2025 Base` (Quick Start) |
> | Tipo de instancia | `t3.large` (2 vCPU, 8 GB RAM) |
> | Par de claves (Key Pair) | `boochan-key` (.pem, RSA) |
> | Almacenamiento | 50 GB gp3 |
> | Grupo de seguridad | `sg-boochan-[tunombre]` (solo RDP 3389 en Fase 1) |
>
> **AWS CLI equivalente (referencia, no obligatorio en el proyecto):**
> ```bash
> aws ec2 run-instances \
>   --image-id ami-xxxxxxxx \
>   --instance-type t3.large \
>   --key-name boochan-key \
>   --security-group-ids sg-xxxxxxxx \
>   --block-device-mappings '[{"DeviceName":"/dev/sda1","Ebs":{"VolumeSize":50,"VolumeType":"gp3"}}]' \
>   --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=WindowsServer}]'
> ```
>
> > [!important] 💡 ¿Por qué `t3.large` y no una instancia más barata?
> > La familia `t3` son instancias "ráfaga" (burstable), ideales para un laboratorio docente. Pero Windows Server con Desktop Experience y, más adelante, el rol AD DS, necesitan bastante más RAM que un Ubuntu Server headless (que se conformaba con `t3.medium`, 4 GB). `t3.large` (8 GB) da margen suficiente. **El Free Tier normal (`t2.micro`/`t3.micro`, 1 GB) NO sirve** para AD DS.

### El Key Pair y la contraseña del Administrador
> **Ruta de la Consola:** `EC2` → `Instancias` → selecciona la instancia → `Conectar` → pestaña `Cliente RDP` → `Obtener contraseña`

> [!important] 💡 El `.pem` NO conecta el RDP — descifra la contraseña
> Este es el concepto nuevo frente a BoochanV3 (Ubuntu): en Linux, el Key Pair `.pem` autenticaba directamente la sesión SSH. En Windows, AWS genera una contraseña aleatoria para el usuario `Administrator` al lanzar la instancia y la **cifra con tu clave pública**. Para obtenerla:
> 1. `Conectar → Cliente RDP → Obtener contraseña`.
> 2. Sube (o pega el contenido de) tu archivo `boochan-key.pem`.
> 3. AWS te devuelve la contraseña en texto plano — **cópiala y guárdala**, la usarás en cada conexión RDP.
>
> A partir de ahí, el RDP se conecta con usuario `Administrator` + esa contraseña, **no** con el `.pem`. La primera obtención de contraseña puede tardar unos minutos tras lanzar la instancia (AWS necesita completar el arranque de Windows).

### Detener y arrancar la instancia (control de crédito)
> **Ruta de la Consola:** `EC2` → `Instancias` → selecciona → `Estado de la instancia` → `Detener` / `Iniciar`

> [!caution] ⚠️ "Detener" libera cómputo, pero el End Lab es tu red de seguridad real
> - **Detener (Stop):** libera el cómputo (deja de gastar crédito de cómputo), pero el volumen EBS sigue existiendo y una **Elastic IP asociada a una instancia detenida SÍ genera un pequeño coste por hora** en AWS. En el Learner Lab, ese coste consume crédito.
> - **End Lab:** congela **todo** el entorno del laboratorio de golpe (instancia + EIP + resto de recursos), sin penalización — es la forma recomendada de terminar cada sesión de clase.
>
> > [!tip] 💡 Hábito recomendado al terminar cada sesión
> > Pulsa **End Lab**. Cuando vuelvas a hacer `Start Lab`, la instancia conservará su Elastic IP (si la asociaste en la Fase 1), así que el `Endpoint` de WireGuard seguirá siendo válido — a diferencia de una IP pública dinámica normal de AWS, que cambia en cada arranque.

---

## 🌐 2. Red: Security Group, IP privada e IP pública

### Consultar y editar las reglas del Security Group
> **Ruta de la Consola:** `EC2` → `Grupos de seguridad` → selecciona `sg-boochan-[tunombre]` → `Reglas de entrada` → `Editar reglas de entrada`

> [!example] Ampliación incremental del Security Group (a diferencia de V2.1)
> BoochanV3.1 abre los puertos **de forma incremental**, no todos de golpe: en la Fase 1 solo se abre **RDP (3389)**; el resto (WireGuard 51820/UDP en la Fase 3, y los puertos de AD DS —Kerberos 88, DNS 53, RPC 135, LDAP 389, SMB 445, LDAPS 636, RPC dinámico, Kerberos password 464, NTP 123— en la Fase 4) se añaden cuando cada fase lo necesita. Este es el mismo enfoque que BoochanV3 (Ubuntu); BoochanV2.1 (Azure), en cambio, abría las 12 reglas de una sola vez.
>
> **AWS CLI equivalente (una regla, de ejemplo — WireGuard):**
> ```bash
> aws ec2 authorize-security-group-ingress \
>   --group-id sg-xxxxxxxx \
>   --protocol udp --port 51820 --cidr 0.0.0.0/0
> ```
>
> > [!tip] 💡 Diagnóstico rápido: "no me conecta pero el puerto debería estar abierto"
> > Revisa siempre tres cosas en este orden: (1) la regla existe en `Reglas de entrada`, (2) el **protocolo** es el correcto (TCP vs UDP es el error más común, especialmente con WireGuard), (3) el **origen** (`Source`) no se restringió por error en un paso de una fase anterior (por ejemplo, si ya aplicaste parte de la Auditoría Final antes de tiempo). Recuerda que en AWS un Security Group **solo permite** — no existen reglas de "denegar" con prioridad como en el NSG de Azure; lo que no está permitido, simplemente está bloqueado.

### La IP privada de la VPC (no hace falta fijarla)
> **Ruta de la Consola:** `EC2` → `Instancias` → selecciona → pestaña `Redes` → `IPv4 privada`

> [!important] 💡 Diferencia con un laboratorio local (Hyper-V) y con Azure
> En Hyper-V, la IP se fija **dentro** del propio Windows con `New-NetIPAddress`. En Azure, se marca como "Estática" en la NIC desde el portal. En **AWS es aún más simple**: la IP privada de la VPC (`172.31.x.x`) **ya es estable mientras la instancia exista** — AWS la mantiene reservada para esa ENI (tarjeta de red virtual), así que no hay que fijar nada. **No cambies la IP privada a mano dentro de Windows** con `New-NetIPAddress`: romperías la correspondencia que AWS mantiene en su tabla de red y perderías la conectividad.

### Elastic IP (IP pública fija)
> **Ruta de la Consola:** `EC2` → `IP elásticas` → `Asignar dirección IP elástica` → luego `Acciones → Asociar` a la instancia `WindowsServer`

> [!tip] 💡 ¿Por qué una Elastic IP y no la IP pública automática?
> Por defecto, la IP pública automática de una instancia EC2 **cambia cada vez que la detienes y la arrancas**. Para un proyecto de varias semanas como BoochanV3.1, donde el `Endpoint` de WireGuard depende de esa IP (Fase 3), se asigna una **Elastic IP** en la Fase 1 y se asocia a la instancia — así el `Endpoint` del cliente WireGuard sigue siendo válido entre sesiones, sin reconfigurar nada. Recuerda: una Elastic IP asociada a una instancia **detenida** consume crédito; por eso al terminar se usa `End Lab` (que congela todo) en vez de solo `Stop`.

---

## 🖥️ 3. Conexión y diagnóstico remoto

### Conectar por RDP
> **Ruta de la Consola:** `EC2` → `Instancias` → selecciona → `Conectar` → `Cliente RDP` → `Descargar archivo de escritorio remoto`

> [!tip] 💡 Alternativa más rápida: `mstsc` directo
> No hace falta descargar el archivo `.rdp` cada vez. Una vez que conoces la IP (la Elastic IP pública en la Fase 1-2, o la del túnel VPN `10.0.0.1` desde la Fase 3), basta con `Windows + R` → `mstsc` → introducir la IP, o desde PowerShell: `mstsc /v:<IP>`. Usuario `Administrator` + la contraseña que descifraste con el `.pem`.

### EC2 Serial Console / SSM Session Manager (rescate si el RDP falla)
> **Ruta de la Consola:** `EC2` → `Instancias` → selecciona → `Conectar` → pestaña `Consola de serie de EC2` (o `Administrador de sesiones`)

> [!caution] ⚠️ Tu red de seguridad si te bloqueas con el firewall
> Si en la Auditoría Final aplicas `DefaultInboundAction Block` en el Firewall de Windows Defender (o restringes el Security Group) desde una IP que no está dentro del rango permitido, te quedarás fuera del servidor por RDP. La **EC2 Serial Console** (o **SSM Session Manager**) es la vía de rescate: accede a la instancia a través de la infraestructura de AWS, sin pasar por ninguna capa de red (ni Security Group ni Firewall de Windows) — es el equivalente cloud a "Conectar..." en Hyper-V Manager, la ventana de consola directa de una VM local, y sustituye a la Consola Serie de Azure de BoochanV2.1.

### Comprobar puertos en escucha dentro de la instancia
> **Dentro de la instancia, PowerShell como Administrador:**
> ```powershell
> Get-NetTCPConnection -State Listen | Sort-Object LocalPort
> ```
> Útil para verificar que un servicio (RDP, SMB, LDAP...) realmente está escuchando en el puerto que esperas, antes de sospechar del Security Group.

---

## 💰 4. Control de crédito del proyecto (AWS Academy Learner Lab)

> [!warning] ⚠️ Este proyecto consume crédito educativo real
> A diferencia de BoochanV1.1 (gratis, en el propio portátil del alumno), cada hora que la instancia `t3.large` está **encendida** consume crédito del Learner Lab. Y una instancia **Windows gasta más rápido que una Linux**, porque el precio por hora incluye la licencia de Microsoft. Una Elastic IP asociada a una instancia detenida también consume un poco.

> [!example] Buenas prácticas de crédito durante el curso
> 1. **Pulsa `End Lab`** al terminar cada sesión de clase — congela todo el entorno sin gasto, la forma más segura de no dejarte nada encendido.
> 2. **No dupliques instancias** por error: si lanzaste una instancia de prueba con otro nombre mientras aprendías el formulario, termínala (`Terminate`) en cuanto lo confirmes con tu profesor.
> 3. **Vigila el crédito restante** en el panel del Learner Lab (arriba, junto al botón de sesión) — es una competencia profesional real: un administrador que no vigila el gasto cloud es un problema.
> 4. **`Terminate` es irreversible:** terminar una instancia borra su disco (salvo que lo marcaras para conservar). No termines la instancia `WindowsServer` a mitad de proyecto — usa `End Lab` para pausar entre sesiones, no `Terminate`.

---

## 📋 5. Tabla resumen: comandos AWS CLI más usados en el proyecto

| Acción | Comando `aws` |
| :--- | :--- |
| Listar instancias | `aws ec2 describe-instances --output table` |
| Ver estado de la instancia | `aws ec2 describe-instance-status --instance-ids i-xxxx` |
| Arrancar instancia | `aws ec2 start-instances --instance-ids i-xxxx` |
| Detener instancia (parar cómputo) | `aws ec2 stop-instances --instance-ids i-xxxx` |
| Ver reglas del Security Group | `aws ec2 describe-security-groups --group-ids sg-xxxx` |
| Añadir regla de entrada | `aws ec2 authorize-security-group-ingress --group-id sg-xxxx --protocol tcp --port [puerto] --cidr [origen]` |
| Ver IP pública/privada | `aws ec2 describe-instances --instance-ids i-xxxx --query "Reservations[].Instances[].[PublicIpAddress,PrivateIpAddress]"` |
| Obtener contraseña de Windows (descifrada) | `aws ec2 get-password-data --instance-id i-xxxx --priv-launch-key boochan-key.pem` |
| Asociar Elastic IP | `aws ec2 associate-address --instance-id i-xxxx --allocation-id eipalloc-xxxx` |

> [!note] En el Learner Lab, `Terminate` de recursos y borrados masivos suelen estar restringidos o gestionados por el propio laboratorio — ante la duda, usa `End Lab` para pausar y consulta con el profesor antes de terminar nada de forma permanente.
