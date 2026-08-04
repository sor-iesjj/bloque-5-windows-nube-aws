## Fase 5 · Apartado 5 — 📚 Fundamento teórico

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Gestión de Identidades (Usuarios y Grupos en Active Directory)**
> 🧭 Índice de la fase: [[Fase_5]]
>
> **📍 Cuándo se lee:** **Antes de teclear.** Los conceptos que necesitas.

---

> [!abstract] 1. Una Simplificación Real (no inventada)
> En **BoochanV3** (Ubuntu + Samba AD DC sobre AWS), esta misma fase requería instalar y configurar **winbind**, un servicio "traductor" que convertía el **SID** de Windows en un **UID/GID** de Linux, porque el sistema de archivos ext4 solo entiende números Unix. Aquí, sobre **Windows Server 2025 con AD DS nativo**, ese paso **no existe y no hace falta**. Los usuarios de Active Directory ya son, de fábrica, objetos nativos del mismo ecosistema que gestiona el sistema de archivos NTFS: no hay dos "idiomas" que traducir, porque el controlador de dominio, el servidor de archivos y el sistema operativo cliente son todos del mismo fabricante y hablan el mismo protocolo de identidad (SID) de principio a fin. Lo que en Samba era un puente necesario entre dos mundos, en Windows nativo es directamente el mismo mundo.

> [!info] 2. Unidades Organizativas (OU): la carpeta de las identidades
> Una **OU** es un contenedor dentro de Active Directory que permite organizar usuarios, grupos y equipos de forma jerárquica — y, sobre todo, permite aplicar políticas (GPOs) o delegar permisos administrativos a un subconjunto concreto del dominio. No confundas una OU con un grupo: la OU organiza *dónde vive* el objeto dentro del árbol de AD; el grupo organiza *a qué tiene acceso*. Un usuario pertenece a **una sola OU** (su ubicación), pero puede ser miembro de **varios grupos** (sus permisos).

### 📖 Diccionario de Conceptos Clave

> [!quote] Terminología de Identidades
> - **SID (Security Identifier):** El identificador único y permanente que Windows asigna a cada usuario, grupo o equipo del dominio. Es el equivalente funcional al UID de Linux, pero nativo de todo el ecosistema Windows — no necesita traducción.
> - **OU (Organizational Unit):** Contenedor jerárquico de AD para organizar objetos y aplicar políticas.
> - **Grupo de Seguridad:** Conjunto de usuarios al que se le asignan permisos de forma colectiva, en lugar de usuario por usuario.
> - **Ámbito Global (Global Scope):** El tipo de grupo más habitual para agrupar usuarios de un mismo dominio; es el equivalente que usaremos aquí.
> - **`New-ADUser` / `New-ADGroup` / `New-ADOrganizationalUnit`:** Los cmdlets de PowerShell (módulo `ActiveDirectory`) que sustituyen a la "navaja suiza" `samba-tool` de Samba.

---

### 🔓 Apertura de Puertos (Security Group de AWS)

> [!info] ℹ️ Sin cambios en el Security Group en esta fase
> Los puertos necesarios para Active Directory (LDAP 389, Kerberos 88, DNS 53, RPC 135...) ya fueron abiertos en el Security Group de la instancia EC2 `WindowsServer` en la **Fase 4**. No tienes que añadir ninguna regla nueva en AWS: la gestión de usuarios y grupos se hace en local, en la consola RDP del servidor o por PowerShell remoto, y no abre puertos nuevos.
>
> En este proyecto administramos el servidor por **RDP**, no por PowerShell remoto — así que el puerto **5985 (WinRM)** NO se abre en el Security Group en ninguna fase. Si quisieras usar PowerShell remoto desde tu equipo (no es necesario para el itinerario), tendrías que abrir tú mismo el 5985 en el Security Group y ejecutar `Enable-PSRemoting` en el servidor. Mientras administres por RDP, no hace falta.

---

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_5.4_Donde_Estamos]] | [[Fase_5]] | [[Fase_5.6_Procedimiento]] |
