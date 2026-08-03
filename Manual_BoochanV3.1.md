# 🚀 BoochanV3.1 — Infraestructura de Servidores Cloud sobre AWS (Windows Server 2025 + AD DS nativo)

> **Módulo:** Sistemas Operativos en Red (SOR) · 2.º Curso SMR
> **Profesor:** Pedro Navarro Miralles · IES Jorge Juan (Alicante)
> **Correo:** p.navarromiralles2@edu.gva.es
> **Entorno:** Amazon Web Services (EC2, AWS Academy Learner Lab) — hermana de BoochanV3 (Ubuntu + Samba AD DC sobre AWS) y de BoochanV2.1 (AD DS nativo sobre Azure)
> **RA cubiertos:** RA.01, RA.02, RA.03, RA.04, RA.05, RA.06
> **⏱️ Tiempo estimado total:** ~13-14 horas repartidas en 9 sesiones (ver desglose por fase más abajo)

---

## ¿Qué es este proyecto?

BoochanV3.1 es un itinerario práctico de **8 fases + auditoría final** en el que el alumno construye, desde cero y **en la nube real de Amazon Web Services**, una infraestructura profesional completa: un servidor **Windows Server 2025** con **Controlador de Dominio (AD DS nativo)**, **VPN WireGuard**, **cuotas de disco (FSRM)**, **permisos avanzados (NTFS + Access-Based Enumeration)** y un **cliente Windows 11 físico del aula** integrado en el dominio a través de un túnel cifrado — todo ello desplegado como recursos reales de una cuenta cloud, protegidos por un **Security Group** perimetral.

Es la versión **Windows Server nativo, en la nube de AWS** del proyecto Boochan. La teoría, la estructura del itinerario y los conceptos son los mismos que en BoochanV1 (VirtualBox/Ubuntu), BoochanV1.1 (Hyper-V/Windows Server), BoochanV2 (Azure/Ubuntu), BoochanV2.1 (Azure/Windows Server) y BoochanV3 (AWS/Ubuntu), pero aquí se sustituye Samba AD DC por el rol **AD DS nativo** de Windows Server, manteniendo la misma infraestructura AWS que ya usaba BoochanV3.

---

## Relación con BoochanV3 (AWS + Ubuntu + Samba)

BoochanV3.1 comparte con BoochanV3 el **mismo escenario cloud completo**: mismo AWS Academy Learner Lab, misma VPC por defecto (`172.31.x.x`), mismo dominio Active Directory (`BOOCHAN` / `BOOCHAN.SPACE`), el mismo Security Group perimetral (ampliado fase a fase), y la misma VPN WireGuard para blindar el acceso remoto. Un alumno que complete cualquiera de las dos versiones ha aprendido exactamente los mismos conceptos de administración de sistemas en red en la nube — solo cambia el fabricante y la implementación técnica del Controlador de Dominio.

La diferencia de fondo es que BoochanV3 usa **Samba AD DC sobre Ubuntu Server 26.04**, una reimplementación de código abierto de Active Directory que exige ensamblar manualmente piezas independientes (`samba-tool`, `winbind` como traductor SID↔UID/GID, Loop Devices para las cuotas, `setfacl` + Access Based Enumeration emulada en `smb.conf`, y un script externo `provision_boochan.sh` clonado de un repositorio Git). BoochanV3.1 usa el **producto original de Microsoft**: el rol AD DS se promociona con un único cmdlet nativo (`Install-ADDSForest`), no hace falta ningún traductor de identidades porque usuarios, sistema de archivos y controlador de dominio hablan el mismo idioma (SID) de principio a fin, las cuotas se aplican directamente sobre NTFS con FSRM sin discos virtuales de por medio, y Access-Based Enumeration es una función nativa del recurso SMB en lugar de una imitación. Ninguna de las dos filosofías es "mejor" en abstracto: Samba es gratuito y corre sobre Linux; AD DS requiere licencia de Windows Server (incluida en el precio por hora de la instancia EC2 Windows) pero ofrece una experiencia mucho más asistida y con menos piezas artesanales que puedan fallar a medio camino.

También cambia el dimensionado y el mecanismo de acceso: BoochanV3 usa una instancia `t3.medium` (2 vCPU, 4 GB RAM) con acceso **SSH** mediante el Key Pair `.pem`; BoochanV3.1 necesita `t3.large` (2 vCPU, 8 GB RAM) porque Windows Server con Desktop Experience y el rol AD DS consumen bastantes más recursos que un Linux sin interfaz gráfica, y el acceso pasa a **RDP** (puerto 3389, escritorio gráfico completo). El Key Pair `.pem` sigue existiendo, pero su uso es distinto: en Linux autenticaba SSH directamente; en Windows sirve para **descifrar la contraseña inicial del Administrador** mediante el botón "Get Windows Password" de la consola EC2 (el `.pem` no se usa para conectar por RDP, solo para obtener la contraseña la primera vez).

## Relación con BoochanV2.1 (Azure + Windows Server)

Frente a la versión de Azure, BoochanV3.1 comparte con BoochanV2.1 el **mismo Controlador de Dominio AD DS nativo**: mismo cmdlet `Install-ADDSForest`, misma familia de comandos PowerShell/AD DS/FSRM, mismos conceptos de SID nativo sin traducción, misma Access-Based Enumeration real de Windows Server, mismas OUs/grupos/usuarios de ejemplo y mismas rutas de carpetas (`C:\ShareData\...`). Un alumno que haya hecho BoochanV2.1 reconocerá casi todos los comandos de esta versión — la diferencia no está en "qué se administra", sino en "qué proveedor cloud lo aloja".

Las diferencias entre ambas están en la capa de infraestructura del proveedor: **NSG de Azure → Security Group de AWS** (AWS solo permite reglas, sin prioridad ni denegar explícito); **usuario+contraseña elegidos al crear la VM → Key Pair `.pem` que descifra la contraseña del Administrator**; **IP privada `10.0.0.x` de Azure → `172.31.x.x` de la VPC por defecto de AWS**; **Consola Serie de Azure → SSM Session Manager / EC2 Serial Console** como mecanismo de rescate; y el modelo de coste: Azure con cuenta gestionada por el profesor frente a **AWS Academy Learner Lab** (crédito educativo, ciclo Start Lab / End Lab). Además, mientras BoochanV2.1 abría las 12 reglas del NSG de una sola vez en la Fase 1, BoochanV3.1 recupera el enfoque **incremental** de BoochanV3: el Security Group empieza abriendo solo RDP (3389) en la Fase 1 y va añadiendo los puertos de AD DS en las fases donde se necesitan.

---

## ⚠️ Antes de empezar: requisitos del proyecto (LÉEME)

- **AWS Academy Learner Lab activo, invitado por el profesor.** A diferencia de BoochanV1.1 (gratis, en el propio portátil), este proyecto consume el **crédito educativo** del Learner Lab (unos 50-100 USD por alumno, sin tarjeta de crédito ni datos bancarios). El profesor invita a cada alumno por correo y da acceso al laboratorio.
- **⚠️ Las instancias Windows consumen crédito MÁS RÁPIDO que las Linux.** El precio por hora de una `t3.large` con Windows Server incluye la licencia de Microsoft, por lo que es más caro que la equivalente Linux de BoochanV3. Usa **siempre el botón "End Lab"** al terminar cada sesión (no solo "Stop") para congelar el entorno sin seguir gastando crédito — una Elastic IP reservada con la instancia detenida también consume.
- **NO sirve el Free Tier normal de AWS**: solo ofrece `t2.micro`/`t3.micro` de 1 GB de RAM, insuficiente para AD DS (necesita 2-4 GB como mínimo, y aquí usamos 8 GB).
- **Windows 11 (PC físico del aula) como cliente**, necesario a partir de la Fase 8. Cualquier edición sirve (el cliente es el propio PC, no una VM anidada como en BoochanV1.1).
- **Cliente de Escritorio Remoto (RDP)**: ya viene instalado en Windows; en Mac "Microsoft Remote Desktop" desde la App Store; en Linux, `Remmina` o `rdesktop`.
- **WireGuard para Windows**, necesario desde la Fase 3, tanto en el servidor como en el PC del aula.
- **Conexión a Internet estable en el aula** — tanto el acceso RDP inicial como el túnel VPN dependen de que el aula tenga salida a Internet.

---

## 🗺️ Índice de fases

| Fase | Título | Concepto AWS / Windows Server clave |
|------|--------|-------------------------------------|
| [1](Fases/Fase_1.md) | Infraestructura Cloud (AWS EC2) | Instancia `WindowsServer` (`t3.large`), Security Group (RDP), Key Pair→contraseña, Elastic IP |
| [2](Fases/Fase_2.md) | Preparación Inicial del Servidor | `Rename-Computer`, IP privada `172.31.x.x`, DNS local, Windows Update |
| [3](Fases/Fase_3.md) | Conectividad VPN (WireGuard para Windows) | Túnel cifrado servidor↔aula sobre Internet real, Tunnel Service, `PersistentKeepalive` |
| [4](Fases/Fase_4.md) | Aprovisionamiento del Dominio (AD DS nativo) | `Install-ADDSForest`, NTDS, Kerberos, DNS integrado, sin script externo |
| [5](Fases/Fase_5.md) | Gestión de Identidades (Usuarios y Grupos) | `New-ADUser`/`New-ADGroup`, SID nativo sin traducción, OUs |
| [6](Fases/Fase_6.md) | Almacenamiento con Cuotas (FSRM) | File Server Resource Manager, cuota directa sobre NTFS, sin discos virtuales |
| [7](Fases/Fase_7.md) | Seguridad Avanzada (NTFS + ABE) | `icacls`, herencia `(OI)(CI)`, Access-Based Enumeration nativa de SMB |
| [8](Fases/Fase_8.md) | Integración del Cliente (Windows 11) | PC físico del aula unido vía VPN, `Add-Computer`, RSAT, mapeo de unidades |
| [Final](Fases/Auditoria_Final.md) | Auditoría Final y Hardening | Zero Trust en dos capas: Security Group de AWS + Firewall de Windows Defender local + End Lab |

### Resumen de cada fase

**[Fase 1 — Infraestructura Cloud (AWS EC2)](Fases/Fase_1.md):** dentro del AWS Academy Learner Lab se lanza la instancia `WindowsServer` (AMI `Windows Server 2025 Base`, `t3.large` — 2 vCPU, 8 GB RAM, disco 50 GB gp3), se crea el Security Group `sg-boochan-[tunombre]` abriendo de momento **solo RDP (3389)**, se genera/asocia el Key Pair `boochan-key` y se obtiene la contraseña del `Administrator` descifrándola con el `.pem`, se asigna una **Elastic IP** y se realiza la primera conexión por **RDP**. Se fija el nombre del proyecto: `BOOCHAN` / `BOOCHAN.SPACE`.

**[Fase 2 — Preparación Inicial del Servidor](Fases/Fase_2.md):** a diferencia de Ubuntu (donde había que purgar Samba preinstalado), Windows Server no trae nada que estorbar. Se renombra el equipo a `WindowsServer` (`Rename-Computer`), se verifica la IP privada asignada por AWS en el rango `172.31.x.x`, se prepara el DNS local (anticipando la Fase 4) y se parchea el sistema con Windows Update antes de instalar ningún rol crítico.

**[Fase 3 — Conectividad VPN (WireGuard para Windows)](Fases/Fase_3.md):** se instala WireGuard para Windows y se construye un túnel cifrado punto a punto entre el servidor en AWS y el PC del aula, sobre Internet real. El puerto UDP 51820 se añade al Security Group en esta fase. El cierre definitivo del RDP público se pospone deliberadamente hasta la Auditoría Final.

**[Fase 4 — Aprovisionamiento del Dominio (AD DS nativo)](Fases/Fase_4.md):** un único cmdlet, `Install-ADDSForest`, sustituye por completo al script `provision_boochan.sh` de BoochanV3: instala la base de datos NTDS, configura Kerberos, activa el DNS integrado y el nivel funcional `Win2025`, sin script externo ni repositorio Git que clonar. Se provisiona el dominio `BOOCHAN.SPACE` (NetBIOS `BOOCHAN`) — el mismo Realm que usa BoochanV3. En esta fase se añaden al Security Group los puertos de AD DS (Kerberos 88, DNS 53, RPC 135, LDAP 389, SMB 445, LDAPS 636, RPC dinámico, Kerberos password 464, NTP 123).

**[Fase 5 — Gestión de Identidades (Usuarios y Grupos)](Fases/Fase_5.md):** se crean las OUs `Departamentos`/`Policia`/`Bomberos`, los grupos de seguridad `Policia` y `Bomberos`, y los usuarios `user1` y `user2` con `New-ADUser`/`New-ADGroup`. La simplificación clave: no existe ningún paso equivalente a `winbind` — los usuarios de AD ya son objetos nativos del mismo ecosistema que NTFS, sin traducción de identidades.

**[Fase 6 — Almacenamiento con Cuotas (FSRM)](Fases/Fase_6.md):** se instala el rol **FSRM** y se aplican cuotas físicas de 5 GB directamente sobre dos carpetas NTFS reales (`C:\ShareData\Prueba1` y `C:\ShareData\Prueba3`) del disco del sistema operativo de la propia instancia, con `New-FsrmQuotaTemplate`/`New-FsrmQuota`, sin crear ningún disco virtual (Loop Device) ni volumen EBS adicional como en BoochanV3.

**[Fase 7 — Seguridad Avanzada (NTFS + ABE)](Fases/Fase_7.md):** se rompe la herencia de permisos de `Prueba3` con `icacls /inheritance:r` y se concede acceso exclusivo al grupo `Policia` con herencia `(OI)(CI)`, y se activa **Access-Based Enumeration nativa** (`New-SmbShare -FolderEnumerationMode AccessBased`) — no una imitación como en Samba, sino la función real de Windows Server, sin tocar ningún archivo de configuración ni reiniciar ningún servicio.

**[Fase 8 — Integración del Cliente (Windows 11)](Fases/Fase_8.md):** el **PC físico del aula** (no una VM anidada, a diferencia de BoochanV1.1) activa el túnel WireGuard, sincroniza su reloj (`w32tm /resync /force`), se une al dominio con `Add-Computer -DomainName "BOOCHAN.SPACE"` y mapea las carpetas compartidas — demostrando que el modelo de permisos NTFS + ABE se respeta desde un cliente real separado físicamente del servidor por cientos de kilómetros, conectado únicamente por el túnel cifrado.

**[Auditoría Final — Hardening](Fases/Auditoria_Final.md):** cierre de seguridad con el principio Zero Trust aplicado en **dos capas independientes**: el **Security Group de AWS** (restringiendo el origen de casi todas las reglas de `Anywhere-IPv4` al rango del túnel/VPN) y el **Firewall de Windows Defender con Seguridad Avanzada** dentro del propio servidor (`DefaultInboundAction Block` en el perfil de Dominio). El puerto WireGuard `51820/udp` es la única excepción que se mantiene abierta a `Anywhere-IPv4` — es la puerta de entrada al propio túnel. Se cierra la sesión con **End Lab** para no seguir gastando crédito.

---

## 📊 Datos clave del proyecto

| Concepto | Valor en BoochanV3.1 |
| :--- | :--- |
| **Nombre NetBIOS** | `BOOCHAN` |
| **Realm (dominio completo)** | `BOOCHAN.SPACE` |
| **FQDN del servidor** | `WindowsServer.BOOCHAN.SPACE` |
| **Instancia del servidor** | `WindowsServer` — `t3.large` (2 vCPU, 8 GB RAM), disco 50 GB gp3 |
| **AMI** | `Windows Server 2025 Base` (Quick Start, catálogo oficial de AWS) |
| **IP privada del servidor (VPC por defecto)** | `172.31.x.x` |
| **IP pública** | Elastic IP (asignada en la Fase 1, usada para RDP inicial y como `Endpoint` del túnel WireGuard) |
| **Red del túnel VPN (WireGuard)** | `10.0.0.0/24` — servidor `10.0.0.1`, cliente del aula `10.0.0.2` |
| **Acceso / credenciales** | Key Pair `boochan-key` (`.pem`) → descifra la contraseña del usuario `Administrator` en la consola EC2 |
| **Usuario administrador local del servidor** | `Administrator` (nombre fijo de AWS para instancias Windows) |
| **Usuario Administrador del dominio** | `BOOCHAN\Administrator` |
| **Usuarios de dominio de ejemplo** | `user1` (grupo `Policia`) · `user2` (grupo `Bomberos`) |
| **Sistema operativo servidor** | Windows Server 2025 (Desktop Experience) |
| **Sistema operativo cliente** | Windows 11 (PC físico del aula) |
| **Protocolo de administración remota** | RDP (puerto 3389) — sustituye al SSH de la versión Ubuntu |
| **Firewall perimetral** | Security Group de AWS (incremental: RDP en Fase 1, puertos de AD DS en Fase 4) |
| **Plataforma / coste** | AWS Academy Learner Lab (crédito educativo · ciclo Start Lab / End Lab) |

---

## ⚖️ Comparativa breve: Samba AD DC (V3) vs. AD DS nativo (V3.1)

| Concepto en BoochanV3 (Samba/Ubuntu) | Equivalente en BoochanV3.1 (AD DS/Windows Server) |
| :--- | :--- |
| Samba AD DC (`samba-tool domain provision`) | **AD DS nativo** (`Install-ADDSForest`) |
| Script externo `provision_boochan.sh` clonado de un repositorio Git | Cmdlet nativo único, sin script ni repositorio externo |
| `winbind` (traductor SID ↔ UID/GID) | No existe — el SID es nativo en todo el ecosistema Windows |
| `samba-tool user create --uid=... --gid-number=...` | `New-ADUser` sin ningún identificador manual — el SID lo asigna el propio controlador de dominio |
| `setfacl` / `getfacl` (ACLs de Linux) | `icacls` / `Set-Acl` (ACLs NTFS) |
| Access Based Enumeration **emulada** en `smb.conf` (`access based share enum = yes`) | Access-Based Enumeration **nativa** (`New-SmbShare -FolderEnumerationMode AccessBased`) |
| Loop Devices (`dd` + `mkfs.ext4` + `fstab` con `loop`) para las cuotas | **FSRM** (`New-FsrmQuota`) directamente sobre NTFS del disco de la instancia, sin discos virtuales ni EBS adicional |
| SSH (puerto 22) con Key Pair `.pem` como acceso remoto | **RDP** (puerto 3389); el `.pem` descifra la contraseña del `Administrator`, no autentica la conexión |
| Ubuntu Server headless (`t3.medium`, 2 vCPU / 4 GB RAM) | Windows Server con Desktop Experience (`t3.large`, 2 vCPU / 8 GB RAM) |
| Purga agresiva de Samba/CUPS preinstalados (`apt purge`) | No hace falta — Windows Server no trae ningún rol activado de fábrica |
| `chattr +i /etc/resolv.conf` para "blindar" el DNS | El propio DNS Server integrado de AD DS se convierte en la fuente de verdad, sin fichero que proteger |
| Rescate: SSM Session Manager / EC2 Serial Console | Rescate: SSM Session Manager / EC2 Serial Console (igual — es infraestructura AWS, no depende del SO) |
| `nano` / edición manual de `smb.conf` | PowerShell (cmdlets sobre objetos, no ficheros de configuración planos) |

---

## 📂 Estructura de la carpeta

```
BoochanV3.1/
├── Manual_BoochanV3.1.md         ← este documento (punto de entrada)
├── Fases/
│   ├── Fase_1.md … Fase_8.md     ← las 8 fases del itinerario
│   ├── Auditoria_Final.md        ← cierre de seguridad (hardening Security Group + Firewall de Windows Defender + End Lab)
│   └── Solucionario/             ← respuestas y retos resueltos (1 por fase)
└── 99_Recursos/
    ├── Diccionario_Comandos_Sistema.md    ← comandos PowerShell / AD DS / FSRM
    ├── Comandos_AWS_CLI_Consola.md        ← específico de BoochanV3.1 (gestión de la instancia/Security Group en AWS, flujo Key Pair→contraseña)
    └── Guía_Errores_y_Resolución.md       ← catálogo de errores por fase
```

---

## 🧭 Recomendación de uso

1. Lee este manual y la advertencia de requisitos (AWS Academy Learner Lab del profesor, crédito, Windows 11 del aula, WireGuard).
2. Sigue las fases **en orden** — son dependientes entre sí (las Fases 4, 5, 7 y 8 son secuenciales; la Fase 8 requiere las Fases 1-7 completas y un PC físico del aula disponible).
3. Si algo falla, antes de bloquearte consulta **[99_Recursos/Guía_Errores_y_Resolución.md](99_Recursos/Guía_Errores_y_Resolución.md)**, organizada por fase, e incluye los problemas específicos de AWS (Security Group mal configurado, contraseña de Windows no descifrada aún, crédito agotado, sesión del Learner Lab caducada).
4. Para repasar cmdlets de PowerShell/AD DS/FSRM consulta **[99_Recursos/Diccionario_Comandos_Sistema.md](99_Recursos/Diccionario_Comandos_Sistema.md)**; para la gestión de la instancia y el Security Group desde la consola o AWS CLI, consulta **[99_Recursos/Comandos_AWS_CLI_Consola.md](99_Recursos/Comandos_AWS_CLI_Consola.md)**.
5. Al terminar cada sesión pulsa **End Lab** — a diferencia de BoochanV1.1, aquí el "servidor" sigue existiendo (y gastando crédito) aunque cierres tu portátil, y una instancia Windows gasta más rápido que una Linux.

---

> **Nota sobre IPs:** a lo largo del proyecto conviven **tres** direcciones relevantes, no las confundas: la **IP privada de la VPC de AWS `172.31.x.x`** (la tarjeta de red real de la instancia dentro de la VPC por defecto), la **Elastic IP pública** (asignada en la Fase 1, usada para la primera conexión RDP y como `Endpoint` del túnel WireGuard) y la **red del túnel WireGuard `10.0.0.0/24`** (servidor `10.0.0.1`, cliente `10.0.0.2`) — una red lógica cifrada que viaja encapsulada dentro del tráfico real de Internet. A diferencia de BoochanV2.1 (donde el túnel reutilizaba el mismo rango `10.0.0.0/24` que la red de Azure), aquí el rango del túnel (`10.0.0.0/24`) y el de la VPC (`172.31.x.x`) son **distintos** — no los mezcles al configurar las reglas.
