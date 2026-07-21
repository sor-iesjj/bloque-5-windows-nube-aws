## 👥 Fase 5: Gestión de Identidades (Usuarios y Grupos en Active Directory)

### Infraestructura de Servidores Cloud (AWS)

> **[Módulo: SOR — Sistemas Operativos en Red]**
> **[U.T. 5 y 6: Administración de usuarios y grupos en Windows]**
> **[RA.02]** Gestiona usuarios y grupos de sistemas operativos en red.
>
> **Profesor:** Pedro Navarro Miralles  
> **Correo:** p.navarromiralles2@edu.gva.es  
> **Centro:** IES Jorge Juan (ALICANTE)
>
> **[Estimación de Implementación]**
> - **Tiempo total:** ~0,5 horas (30 minutos)
> - **RAM del servidor:** sin coste adicional — la gestión de usuarios de AD DS es una función nativa del rol, no necesita ningún servicio extra instalado
> - **Desglose:** Crear estructura de Unidades Organizativas (5 min) + Crear grupos de seguridad (5 min) + Crear usuarios (5 min) + Verificaciones (10 min) + Troubleshooting (5 min)
> - **Dependencias externas:** rol AD DS operativo desde la Fase 4, instancia EC2 `WindowsServer` (`t3.large`) accesible por RDP o PowerShell remoto

---
### 🎯 ¿Dónde Estamos?

> [!info] Vienes de Fase 4
> Tienes un dominio Active Directory completamente provisionado sobre la instancia EC2 `WindowsServer` (AWS, `t3.large`) con el rol AD DS instalado. El dominio `BOOCHAN.SPACE` (NetBIOS `BOOCHAN`) existe, DNS es propio del controlador de dominio, y el Security Group de AWS permite el tráfico de Active Directory desde la Fase 4. Sin embargo, dentro de ese dominio todavía no existe ni un solo usuario ni grupo de "negocio" — solo las cuentas administrativas que Windows crea por defecto.

> [!warning] El Problema
> Un dominio sin usuarios ni grupos organizados es papel mojado. Si no estructuras a los empleados en grupos con un criterio claro (departamento, función, nivel de acceso), cada vez que necesites dar permisos a una carpeta tendrás que hacerlo usuario por usuario — un despropósito que no escala y que es la primera causa de "fugas de permisos" en auditorías de seguridad reales.

> [!success] Objetivo de esta Fase
> Crear una estructura organizativa dentro de Active Directory: **Unidades Organizativas (OU)** que reflejen los departamentos de la empresa ficticia, **grupos de seguridad** que agrupen a los empleados por función, y **usuarios** asignados a esos grupos. Todo con las herramientas nativas de AD DS — sin necesidad de traducir nada a otro sistema operativo.

> [!tip] Hoja de Ruta
> 1. Crear la Unidad Organizativa raíz `Departamentos` y dos sub-OUs: `Policia` y `Bomberos`
> 2. Crear dos grupos de seguridad de ámbito global: `Policia` y `Bomberos` — para demostrar después segregación de datos
> 3. Crear dos usuarios: `user1` (miembro de `Policia`) y `user2` (miembro de `Bomberos`)
> 4. Verificar con `Get-ADUser` y `Get-ADGroupMember` que la pertenencia a grupo es correcta
> 5. Comprobar el inicio de sesión de ambos usuarios (o al menos la validación de credenciales) contra el dominio
> 6. Reflexionar sobre la simplificación respecto a un entorno Samba: en AD DS nativo no existe ningún paso de "traducción" de identidades
>
> **Resultado Final:** El dominio tiene una estructura de OUs, grupos y usuarios lista para que la Fase 6 (cuotas con FSRM) y la Fase 7 (permisos NTFS + ABE) construyan sobre ella.
> **Siguiente:** Fase 6 (Almacenamiento con FSRM) — aplicarás cuotas de disco reales sobre carpetas NTFS para controlar que no llenen el servidor.

---

### 📚 Fundamento Teórico

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

### 🛠️ Procedimiento Práctico

> [!example] Paso 0: Confirmar que el módulo ActiveDirectory está disponible
> Al instalar el rol AD DS en la Fase 4, el módulo de PowerShell `ActiveDirectory` queda instalado automáticamente en el propio controlador de dominio. Comprueba que está cargado:
> ```powershell
> Get-Module -ListAvailable ActiveDirectory
> Import-Module ActiveDirectory
> ```
> Si vas a administrar el dominio desde otro equipo (no el propio DC), necesitarías instalar antes las **RSAT** (`Install-WindowsFeature RSAT-AD-PowerShell`), pero en este proyecto trabajamos siempre desde la consola de la propia instancia `WindowsServer`.

> [!example] Paso 1: Creación de la Estructura de Unidades Organizativas
> Creamos primero la OU raíz de departamentos y luego las dos sub-OUs donde vivirán los usuarios y grupos del proyecto:
>
> > [!info] 📚 Diccionario de Comandos: Para repasar la sintaxis de los cmdlets `*-AD*`, consulta el [[Diccionario_Comandos_Sistema]].
>
> ```powershell
> # OU raíz de departamentos
> New-ADOrganizationalUnit -Name "Departamentos" -Path "DC=boochan,DC=space"
>
> # Sub-OU para Policia
> New-ADOrganizationalUnit -Name "Policia" -Path "OU=Departamentos,DC=boochan,DC=space"
>
> # Sub-OU para Bomberos
> New-ADOrganizationalUnit -Name "Bomberos" -Path "OU=Departamentos,DC=boochan,DC=space"
> ```
>
> > [!tip] 💡 ¿Qué hace `-Path`?
> > El parámetro `-Path` indica el "domicilio" exacto del nuevo objeto dentro del árbol LDAP de Active Directory, escrito en notación **DN (Distinguished Name)**: se lee de derecha a izquierda, empezando por el dominio (`DC=boochan,DC=space`) y bajando por cada contenedor (`OU=Departamentos`). Es literalmente la misma lógica de rutas de carpetas, pero en formato LDAP.

> [!example] Paso 2: Creación de los Grupos de Seguridad
> Creamos los dos grupos del proyecto, ubicados dentro de su OU correspondiente. El grupo `Policia` tendrá acceso a las carpetas protegidas en la Fase 7, y `Bomberos` servirá para demostrar que los usuarios sin permisos no ven esas carpetas:
> ```powershell
> New-ADGroup -Name "Policia" `
>     -GroupScope Global -GroupCategory Security `
>     -Path "OU=Policia,OU=Departamentos,DC=boochan,DC=space"
>
> New-ADGroup -Name "Bomberos" `
>     -GroupScope Global -GroupCategory Security `
>     -Path "OU=Bomberos,OU=Departamentos,DC=boochan,DC=space"
> ```
>
> > [!tip] 💡 ¿Qué son `-GroupScope` y `-GroupCategory`?
> > - **`-GroupCategory Security`:** El grupo se usará para asignar permisos (frente a `Distribution`, que solo sirve para listas de correo).
> > - **`-GroupScope Global`:** Ámbito estándar para agrupar usuarios de un mismo dominio bajo un mismo criterio funcional — es el equivalente natural al GID que usábamos en Samba, pero sin necesidad de asignar ningún número manualmente: Active Directory gestiona el SID del grupo internamente.

> [!example] Paso 3: Creación de Usuarios y Asignación a Grupos
> Creamos dos usuarios: `user1` pertenecerá al grupo `Policia` y `user2` al grupo `Bomberos`. Esto nos permitirá demostrar en la Fase 7 que cada uno ve carpetas diferentes:
> ```powershell
> # Contraseña común para el proyecto (cumple la política de complejidad de AD)
> $securePass = ConvertTo-SecureString "P@ssword2026!" -AsPlainText -Force
>
> # Creamos user1 en la OU de Policia
> New-ADUser -Name "user1" -SamAccountName "user1" `
>     -UserPrincipalName "user1@boochan.space" `
>     -Path "OU=Policia,OU=Departamentos,DC=boochan,DC=space" `
>     -AccountPassword $securePass -Enabled $true -ChangePasswordAtLogon $false
>
> # Creamos user2 en la OU de Bomberos
> New-ADUser -Name "user2" -SamAccountName "user2" `
>     -UserPrincipalName "user2@boochan.space" `
>     -Path "OU=Bomberos,OU=Departamentos,DC=boochan,DC=space" `
>     -AccountPassword $securePass -Enabled $true -ChangePasswordAtLogon $false
>
> # Añadimos cada usuario a su grupo correspondiente
> Add-ADGroupMember -Identity "Policia" -Members "user1"
> Add-ADGroupMember -Identity "Bomberos" -Members "user2"
> ```
>
> > [!important] 💡 ¿Por qué no hace falta ningún `--uid-number` aquí?
> > **La simplificación clave de esta fase:** en Samba, olvidar `--uid-number` significaba que el sistema asignaba un identificador Unix impredecible, generando riesgos de reutilización y escalada de privilegios silenciosa. En AD DS nativo, cada usuario y grupo recibe automáticamente un **SID único y permanente** en el momento de su creación, gestionado internamente por el controlador de dominio — no hay ningún número que el administrador deba inventar ni vigilar manualmente. Es una responsabilidad completa del ecosistema Windows, no del administrador.

---

### 🚩 Resolución de Problemas y Evaluación

> [!bug] Troubleshooting (¿Los usuarios no funcionan?)
> | Problema | Causa Probable | Solución Sugerida |
> | :--- | :--- | :--- |
> | `Get-ADUser user1` devuelve error "Cannot find an object". | El usuario no se creó correctamente o hay un error tipográfico en el `-Path`. | Ejecuta `Get-ADUser -Filter *` para listar todos los usuarios del dominio y localizar errores de nombre o de OU. |
> | Error: "The password does not meet the length, complexity, or history requirement". | La política de contraseñas de dominio exige complejidad. | Usa una contraseña con mayúsculas, minúsculas, números y símbolos como `P@ssword2026!`. |
> | Error: "An attempt was made to add an object to the directory with a name that is already in use". | El usuario o grupo ya existía de un intento anterior. | Ejecuta `Remove-ADUser user1 -Confirm:$false` o `Remove-ADGroup Policia -Confirm:$false` y vuelve a crearlo. |
> | Error: "Insufficient access rights to perform the operation". | La PowerShell no se está ejecutando como Administrador de dominio. | Abre PowerShell con "Ejecutar como administrador" y confirma que tu sesión tiene privilegios de Admins. del dominio. |
> | `New-ADOrganizationalUnit` falla con "already exists". | La OU ya existe de una ejecución anterior. | Comprueba con `Get-ADOrganizationalUnit -Filter 'Name -eq "Departamentos"'` antes de recrearla, o continúa sin crearla de nuevo. |
> | No puedo conectar por RDP desde mi equipo. | El Security Group de AWS no tiene abierto el puerto RDP (3389), o la regla se restringió antes de tiempo. | Revisa las reglas de entrada del Security Group de `WindowsServer` en la Consola de AWS (EC2 → Grupos de seguridad) — RDP (3389) está abierto desde la Fase 1. (El 5985/WinRM NO se abre en este proyecto: administramos por RDP.) |

> [!help] Preguntas Críticas (Autoevaluación)
> 1. ¿Por qué es mejor y más profesional dar permisos a un grupo que a un usuario individual?
> 2. En Samba hacía falta el servicio **winbind** para traducir identidades. Explica, en tus propias palabras, por qué en Windows Server con AD DS nativo ese paso desaparece por completo.
> 3. 🔬 **Reto práctico:** Ejecuta `Get-ADUser user1 -Properties MemberOf` y `Get-ADUser user2 -Properties MemberOf`. Anota a qué grupo pertenece cada uno. Ahora ejecuta `Get-ADGroupMember Policia` y `Get-ADGroupMember Bomberos`. ¿Coincide la pertenencia en ambos sentidos (usuario→grupo y grupo→usuario)?
> 4. 🔬 **Reto práctico:** Intenta crear un usuario sin especificar `-Path` (déjalo caer en el contenedor `Users` por defecto): `New-ADUser -Name "user3" -SamAccountName "user3" -AccountPassword $securePass -Enabled $true`. Ejecuta después `Get-ADUser user3 | Select DistinguishedName`. ¿En qué contenedor cayó? ¿Por qué en una organización real es mala práctica dejar que los usuarios se acumulen en el contenedor por defecto en lugar de una OU planificada?
> 5. ¿Cómo verificarías desde PowerShell que un grupo de seguridad existe y qué tipo de ámbito (`GroupScope`) tiene?

---

> [!caution] 🛑 Auditoría y Evaluación (RA.02)
> El alumno debe demostrar que la estructura de OUs, grupos y usuarios existe y es coherente. **Validación:** El comando `Get-ADUser user1 -Properties MemberOf` debe mostrar `Policia` entre sus grupos, y `Get-ADUser user2 -Properties MemberOf` debe mostrar `Bomberos`.

> [!success] 🏁 Punto de Control (Antes de seguir)
> - [ ] ¿Existen las OUs `Departamentos`, `Policia` y `Bomberos` (`Get-ADOrganizationalUnit -Filter *`)?
> - [ ] ¿El comando `Get-ADGroupMember Policia` devuelve `user1`?
> - [ ] ¿El comando `Get-ADGroupMember Bomberos` devuelve `user2`?
> - [ ] ¿Puedes iniciar sesión (o validar credenciales con `Test-ComputerSecureChannel` / login RDP) como `user1` y `user2` contra el dominio `BOOCHAN.SPACE`?
