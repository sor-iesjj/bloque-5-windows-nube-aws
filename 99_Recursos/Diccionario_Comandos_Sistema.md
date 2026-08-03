# 📚 Diccionario de Comandos del Sistema (PowerShell / AD DS / FSRM)

Esta enciclopedia de bolsillo te servirá para entender "qué estás escribiendo" a lo largo de todas las fases del proyecto BoochanV3.1. Un administrador copia y pega; **un ingeniero entiende la sintaxis**.

> [!info] Sobre este documento
> Los cmdlets de PowerShell, Active Directory y FSRM son los mismos tanto si administras un dominio en un laboratorio local como en la nube — no cambian según dónde viva el servidor. Este diccionario está centrado en los comandos **de Windows Server**, la pieza que diferencia a BoochanV3.1 de su hermano BoochanV3 (Ubuntu + Samba). Para los comandos propios de la capa cloud (lanzar/administrar la instancia EC2, el Security Group, la Elastic IP, el flujo Key Pair→contraseña), consulta [[Comandos_AWS_CLI_Consola]].

---

## ⚙️ 1. Identidad y Red del Sistema

### `Rename-Computer`
> **Descripción:** Cambia el nombre NetBIOS/DNS del equipo. A diferencia de Linux (donde basta editar `/etc/hostname` y aplicar sin reiniciar), Windows exige un **reinicio completo** para propagar el nuevo nombre a todos los servicios.
> **Sintaxis:** `Rename-Computer -NewName [nombre] -Restart`

> [!example] Ejemplo de uso (Fase 2):
> - `Rename-Computer -NewName "WindowsServer" -Restart`: Cambia el nombre genérico (`EC2AMAZ-XXXXXXX`) por `WindowsServer` y reinicia automáticamente. Debe hacerse **antes** de promocionar el servidor a Controlador de Dominio (Fase 4) — cambiar el nombre después es mucho más complicado.

### `Get-NetAdapter`
> **Descripción:** Lista todos los adaptadores de red del sistema con su índice (`ifIndex`), estado y descripción. Es el primer comando a ejecutar antes de tocar cualquier configuración de red.
> **Sintaxis:** `Get-NetAdapter`

> [!example] Ejemplo de uso (Fase 2):
> - En AWS, la instancia `WindowsServer` solo tiene **un** adaptador de red (a diferencia de un laboratorio local con dos adaptadores separados) — la tarjeta virtual (ENI) que AWS conecta a la subred de la VPC.

### `Get-NetIPConfiguration` y `Get-NetIPAddress`
> **Descripción:** Muestran la configuración de red del adaptador: IP, puerta de enlace y DNS (`Get-NetIPConfiguration`) o solo la IP asignada (`Get-NetIPAddress`).
> **Sintaxis:** `Get-NetIPConfiguration` · `Get-NetIPAddress -AddressFamily IPv4`

> [!example] Ejemplo de uso (Fase 2):
> - `Get-NetIPConfiguration`: Muestra la IP privada `172.31.x.x` que AWS asignó a la instancia por DHCP interno de la VPC. A diferencia de un laboratorio local, **no se fija esta IP con `New-NetIPAddress` dentro de Windows** — la IP privada de la VPC ya es estable mientras la instancia exista (AWS la mantiene reservada para esa ENI), así que no hay que "fijarla" como en Hyper-V. Cambiar la IP a mano dentro de Windows además rompería la conectividad, porque AWS espera esa IP concreta en su tabla de red.

### `Set-DnsClientServerAddress` y `Get-DnsClientServerAddress`
> **Descripción:** Fija (o consulta) qué servidor DNS debe consultar un adaptador de red concreto.
> **Sintaxis:** `Set-DnsClientServerAddress -InterfaceIndex [n] -ServerAddresses "[ip]"`

> [!example] Ejemplo de uso (Fase 2):
> - `Set-DnsClientServerAddress -InterfaceIndex 4 -ServerAddresses "127.0.0.1"`: Apunta el adaptador a sí mismo — preparación deliberada para cuando el rol AD DS instale el servicio DNS en la Fase 4. Es el mismo principio que `chattr +i /etc/resolv.conf` en BoochanV3: el Controlador de Dominio siempre se consulta a sí mismo.

### `Test-Connection` y `Resolve-DnsName`
> **Descripción:** `Test-Connection` es el equivalente PowerShell de `ping` (con más opciones de salida estructurada). `Resolve-DnsName` es el equivalente de `nslookup`, muy usado para verificar los registros SRV de Kerberos.
> **Sintaxis:** `Test-Connection [destino]` · `Resolve-DnsName [nombre]`

> [!example] Ejemplos de uso (Fases 1, 4):
> - `Test-Connection 8.8.8.8`: Verifica salida a internet desde la instancia.
> - `Resolve-DnsName _kerberos._tcp.BOOCHAN.SPACE`: El comando maestro de validación del dominio — si devuelve el registro SRV con la IP privada `172.31.x.x` del servidor, Kerberos está operativo.

---

## 📦 2. Gestión de Roles y Actualizaciones

### `Install-WindowsFeature`
> **Descripción:** El equivalente Windows a `apt install`, pero para **roles y características del propio sistema operativo** (no paquetes externos de internet, salvo que falten archivos de origen). Activa un componente que ya viene incluido en la instalación de Windows Server.
> **Sintaxis:** `Install-WindowsFeature -Name [rol] -IncludeManagementTools`

> [!example] Ejemplos de uso (Fases 4 y 6):
> - `Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools`: Instala los binarios del rol AD DS (Fase 4). Solo instala, todavía no crea ningún dominio — eso lo hace `Install-ADDSForest` a continuación.
> - `Install-WindowsFeature FS-Resource-Manager -IncludeManagementTools`: Instala FSRM (Fase 6), incluida la consola gráfica y el módulo de PowerShell `FileServerResourceManager`.
> - `-IncludeManagementTools`: Añade también las herramientas gráficas y de PowerShell de gestión (`Get-ADUser`, `dsa.msc`, consola de FSRM...). Sin este parámetro, el rol se instala pero sin herramientas cómodas de administración.

### `Get-WindowsFeature`
> **Descripción:** Comprueba si un rol o característica está instalado y en qué estado.
> **Sintaxis:** `Get-WindowsFeature [-Name rol]`

> [!example] Ejemplo de uso (Fase 6):
> - `Get-WindowsFeature FS-Resource-Manager`: Confirma que el rol FSRM está realmente instalado si un cmdlet `*-Fsrm*` no se reconoce.

### `Install-Module` y `Get-WindowsUpdate`
> **Descripción:** `Install-Module` descarga módulos desde PowerShell Gallery (el repositorio oficial de módulos de PowerShell), como el módulo `PSWindowsUpdate` que no viene instalado de fábrica. `Get-WindowsUpdate` (parte de ese módulo) busca e instala parches del sistema.
> **Sintaxis:** `Install-Module -Name [módulo] -Force` · `Get-WindowsUpdate -Install -AcceptAll -AutoReboot`

> [!example] Ejemplo de uso (Fase 2):
> - `Install-PackageProvider -Name NuGet -MinimumVersion 2.8.5.201 -Force` seguido de `Install-Module -Name PSWindowsUpdate -Force` y `Get-WindowsUpdate -Install -AcceptAll -AutoReboot`: Parchea el sistema antes de instalar cualquier rol crítico como AD DS — el hábito profesional de "actualizar primero, instalar roles después".

---

## 🛡️ 3. Active Directory Domain Services (AD DS)

### `Install-ADDSForest`
> **Descripción:** Es el cmdlet que promociona el servidor a Controlador de Dominio, creando un bosque de Active Directory completamente nuevo. Es el equivalente Windows al `samba-tool domain provision` de BoochanV3, pero sin script externo: un único cmdlet nativo que orquesta la base de datos NTDS, Kerberos, el DNS integrado y el nivel funcional, todo con validaciones automáticas (`Test-ADDSForestInstallation`) antes de aplicar ningún cambio.
> **Sintaxis:** `Install-ADDSForest -DomainName [realm] -DomainNetbiosName [netbios] -InstallDNS -SafeModeAdministratorPassword [pass] -Force`

> [!example] Ejemplo de uso (Fase 4):
> ```powershell
> Install-ADDSForest -DomainName "BOOCHAN.SPACE" -DomainNetbiosName "BOOCHAN" `
>   -DomainMode "Win2025" -ForestMode "Win2025" -InstallDNS `
>   -SafeModeAdministratorPassword (ConvertTo-SecureString "P@ssw0rd" -AsPlainText -Force) -Force
> ```
> El servidor se reinicia automáticamente al finalizar (5-10 minutos en total). `-SafeModeAdministratorPassword` es la contraseña del modo de recuperación DSRM, **distinta** de la contraseña del Administrador del dominio.

### `Get-ADUser`, `New-ADUser`, `Get-ADGroup`, `New-ADGroup`, `Add-ADGroupMember`, `Get-ADGroupMember`
> **Descripción:** La familia de cmdlets del módulo `ActiveDirectory` que sustituye por completo a la "navaja suiza" `samba-tool` de Samba. Cada cmdlet gestiona un tipo de objeto del directorio: usuarios, grupos, su pertenencia mutua.
> **Sintaxis:** `New-ADUser -Name [nombre] -SamAccountName [login] -Path [OU en formato DN] -AccountPassword [SecureString] -Enabled $true`

> [!example] Ejemplos de uso (Fase 5):
> - `New-ADGroup -Name "Policia" -GroupScope Global -GroupCategory Security -Path "OU=Policia,OU=Departamentos,DC=boochan,DC=space"`: Crea un grupo de seguridad de ámbito Global — el equivalente natural al GID de Samba, pero sin necesidad de asignar ningún número manualmente.
> - `New-ADUser -Name "user1" -SamAccountName "user1" -UserPrincipalName "user1@boochan.space" -Path "OU=Policia,OU=Departamentos,DC=boochan,DC=space" -AccountPassword $securePass -Enabled $true`: Crea el usuario directamente en su OU, con SID único y permanente asignado automáticamente por el controlador de dominio.
> - `Add-ADGroupMember -Identity "Policia" -Members "user1"`: Añade el usuario al grupo.
> - `Get-ADUser user1 -Properties MemberOf`: Verifica a qué grupos pertenece un usuario.
> - `Get-ADGroupMember Policia`: Verifica qué usuarios pertenecen a un grupo (verificación en el sentido inverso).

### `New-ADOrganizationalUnit`
> **Descripción:** Crea una Unidad Organizativa (OU), el contenedor jerárquico de AD que organiza *dónde vive* cada objeto (a diferencia del grupo, que organiza *a qué tiene acceso*).
> **Sintaxis:** `New-ADOrganizationalUnit -Name [nombre] -Path [DN del contenedor padre]`

> [!example] Ejemplo de uso (Fase 5):
> - `New-ADOrganizationalUnit -Name "Departamentos" -Path "DC=boochan,DC=space"`: El parámetro `-Path` usa notación DN (Distinguished Name), que se lee de derecha a izquierda empezando por el dominio — la misma lógica de rutas de carpetas, pero en formato LDAP.

### `Get-Service` (aplicado a servicios de dominio)
> **Descripción:** Comprueba el estado de los servicios críticos de Windows, incluidos los que instala AD DS.
> **Sintaxis:** `Get-Service -Name [servicio1, servicio2...] | Select-Object Name, Status`

> [!example] Ejemplo de uso (Fase 4):
> - `Get-Service -Name NTDS, DNS, Kdc | Select-Object Name, Status`: Los tres pilares de un Controlador de Dominio deben mostrar `Running` — `NTDS` (base de datos del directorio), `DNS` (resolución de nombres integrada) y `Kdc` (Centro de distribución de claves, el corazón de Kerberos).

---

## 📁 4. Sistema de Archivos, Cuotas (FSRM) y Permisos NTFS

### `New-Item`
> **Descripción:** Crea archivos o carpetas nuevas. Es el equivalente PowerShell de `mkdir`/`touch` en Linux.
> **Sintaxis:** `New-Item -Path [ruta] -ItemType Directory -Force`

> [!example] Ejemplo de uso (Fase 6):
> - `New-Item -Path "C:\ShareData\Prueba1" -ItemType Directory -Force`: El parámetro `-Force` crea también las carpetas intermedias que falten (equivalente al `-p` de `mkdir` en Linux) y no da error si ya existe. Usa siempre la unidad `C:` (el disco del sistema operativo de la instancia) — este proyecto no añade ningún volumen EBS de datos adicional en AWS.

### `New-FsrmQuotaTemplate` y `New-FsrmQuota`
> **Descripción:** `New-FsrmQuotaTemplate` crea una plantilla reutilizable de cuota (tamaño, tipo de límite, umbrales de aviso). `New-FsrmQuota` aplica esa plantilla a una carpeta concreta. A diferencia de un Loop Device en Linux, la cuota se aplica **directamente sobre la carpeta** del volumen NTFS real, sin crear ningún disco virtual adicional ni tocar el equivalente al `fstab`.
> **Sintaxis:** `New-FsrmQuotaTemplate -Name [nombre] -Size [tamaño] -Threshold [umbral]` · `New-FsrmQuota -Path [ruta] -Template [nombre]`

> [!example] Ejemplos de uso (Fase 6):
> - `New-FsrmQuotaTemplate -Name "Limite5GB-Estricto" -Size 5GB -Threshold $umbral85`: PowerShell entiende directamente el sufijo `GB` sin calcular bytes a mano. Al no indicar `-SoftLimit`, la plantilla es de tipo **estricto (hard)** por defecto — Windows bloquea físicamente cualquier escritura que supere el límite.
> - `New-FsrmQuota -Path "C:\ShareData\Prueba1" -Template "Limite5GB-Estricto"`: Aplica el límite. No requiere ningún reinicio ni comando de montaje adicional — la cuota se activa de inmediato.

### `Get-FsrmQuota`
> **Descripción:** Consulta el estado de una cuota aplicada: tamaño, uso actual, si es estricta o flexible.
> **Sintaxis:** `Get-FsrmQuota -Path [ruta]`

> [!example] Ejemplo de uso (Fase 6):
> - `Get-FsrmQuota -Path "C:\ShareData\Prueba1" | Select SoftLimit, Size`: Confirma que `SoftLimit` es `False` (cuota estricta) y el tamaño en bytes.

### `icacls`
> **Descripción:** La herramienta de línea de comandos nativa de Windows para gestionar permisos NTFS — el equivalente exacto a `setfacl`/`getfacl` en Linux. A diferencia de PowerShell puro, `icacls` es un binario clásico de `cmd.exe` que también funciona perfectamente desde PowerShell.
> **Sintaxis:** `icacls [ruta] /inheritance:r|/grant "[usuario/grupo]:([herencia])[permiso]"`

> [!example] Ejemplos de uso (Fase 7):
> - `icacls "C:\ShareData\Prueba3" /inheritance:r`: Rompe (`r` = remove) la herencia de permisos del padre — sin este paso, cualquier permiso genérico del disco `C:` seguiría dando acceso a todo el mundo.
> - `icacls "C:\ShareData\Prueba3" /grant "BOOCHAN\Policia:(OI)(CI)M"`: Concede el permiso **M** (Modify) al grupo `Policia`. `(OI)` = Object Inherit (los archivos nuevos heredan el permiso), `(CI)` = Container Inherit (las subcarpetas nuevas también) — el equivalente exacto al flag `-d` (default/herencia) de `setfacl`.

### `New-SmbShare` y `Get-SmbShare`
> **Descripción:** Publica una carpeta como recurso de red SMB. El parámetro `-FolderEnumerationMode` es el que activa (o no) **Access-Based Enumeration**, sin tocar ningún archivo de configuración de texto como `smb.conf` en Samba.
> **Sintaxis:** `New-SmbShare -Name [nombre] -Path [ruta] -FullAccess [grupo] -FolderEnumerationMode AccessBased|Unrestricted`

> [!example] Ejemplos de uso (Fase 7):
> - `New-SmbShare -Name "Prueba3" -Path "C:\ShareData\Prueba3" -FullAccess "BOOCHAN\Policia" -FolderEnumerationMode AccessBased`: `AccessBased` hace que Windows compruebe, para cada usuario que consulta el recurso, si tiene permiso NTFS real — si no lo tiene, oculta la carpeta directamente del listado en vez de mostrarla y denegar el acceso al entrar.
> - `Get-SmbShare -Name "Prueba3" | Select-Object Name, Path, FolderEnumerationMode`: Verifica el modo activo. Los cambios de `New-SmbShare`/`Set-SmbShare` se aplican de inmediato, sin reiniciar ningún servicio (a diferencia de `sudo systemctl restart samba-ad-dc` en BoochanV3).

---

## 🔐 5. Redes y VPN (WireGuard para Windows)

### `wg genkey` / `wg pubkey`
> **Descripción:** Los mismos binarios `wg` que en Linux — WireGuard incluye herramientas de línea de comandos idénticas en ambas plataformas, instaladas junto con la aplicación gráfica de Windows.
> **Sintaxis:** `wg genkey | Out-File -Encoding ascii privatekey` · `Get-Content privatekey | wg pubkey | Out-File -Encoding ascii publickey`

> [!example] Ejemplo de uso (Fase 3):
> - El operador Pipe (`|`) funciona igual que en Linux: encadena la salida de un comando como entrada del siguiente. Genera la llave privada y, a partir de ella, calcula la pública.

### `wireguard.exe /installtunnelservice`
> **Descripción:** Instala un túnel como servicio persistente de Windows, arrancando inmediatamente y registrándose para iniciar automáticamente en cada reinicio. Es el equivalente Windows a `sudo wg-quick up wg0` + `sudo systemctl enable wg-quick@wg0` en un solo comando.
> **Sintaxis:** `wireguard.exe /installtunnelservice [ruta_al_.conf]`

> [!example] Ejemplo de uso (Fase 3):
> - `wireguard.exe /installtunnelservice C:\WireGuard\wg0.conf`: Levanta el túnel y lo deja persistente. Para desinstalarlo: `wireguard.exe /uninstalltunnelservice wg0`.

### `wg show`
> **Descripción:** El panel de radar de WireGuard — muestra el tráfico y el momento del último "apretón de manos" (`latest handshake`) con cada peer conectado.
> **Sintaxis:** `wg show`

> [!example] Ejemplo de uso (Fase 3):
> - Si `latest handshake` es reciente, el túnel funciona. Si no aparece ningún handshake, las llaves están mal intercambiadas o el `Endpoint` (la Elastic IP de AWS) es incorrecto.

### `New-NetFirewallRule`
> **Descripción:** Crea una regla en el Firewall de Windows Defender con Seguridad Avanzada — a diferencia de Ubuntu (donde `ufw` viene desactivado de serie), el firewall de Windows Server **está activo por defecto** en los tres perfiles de red.
> **Sintaxis:** `New-NetFirewallRule -DisplayName [nombre] -Direction Inbound -Protocol [TCP/UDP] -LocalPort [puerto] -RemoteAddress [rango] -Action Allow|Block`

> [!example] Ejemplos de uso (Fase 3 y Auditoría Final):
> - `New-NetFirewallRule -DisplayName "WireGuard VPN" -Direction Inbound -Protocol UDP -LocalPort 51820 -Action Allow`: Abre el puerto UDP de WireGuard en el firewall local, sin el cual el Firewall de Windows Defender bloquearía el tráfico entrante del túnel aunque el Security Group de AWS ya lo permita.
> - `New-NetFirewallRule -DisplayName "AD DS - Solo VPN BOOCHAN (TCP)" -Direction Inbound -RemoteAddress 10.0.0.0/24 -Action Allow -Protocol TCP -LocalPort 88,53,135,389,445,636,464`: Restringe los servicios de dominio para que solo respondan a la red de la VPN (`10.0.0.0/24`, el túnel WireGuard) — la segunda muralla, independiente del Security Group de AWS.

> [!important] 💡 Dos murallas, no una
> En BoochanV3.1, cada regla de firewall convive con su equivalente en el **Security Group de AWS** (ver [[Comandos_AWS_CLI_Consola]]). El Security Group filtra el tráfico *antes* de llegar a la tarjeta de red (ENI) de la instancia; el Firewall de Windows Defender filtra *dentro* del propio sistema operativo. Ver la Auditoría Final para el detalle completo de ambas capas.

---

## 🪟 6. Comandos Críticos del Cliente Windows 11

### `w32tm`
> **Descripción:** Un protocolo criptográfico como Kerberos detesta las paradojas temporales. Si pasan más de 5 minutos de diferencia entre el reloj del cliente y el del servidor, los tickets de autenticación se rechazan. En BoochanV3.1 esto es especialmente relevante porque el PC del aula y la instancia en AWS son máquinas físicamente distintas, cada una con su propia fuente de hora, conectadas únicamente por el túnel VPN.
> **Sintaxis:** `w32tm /[acción]`

> [!example] Ejemplo de uso (Fase 8):
> - `w32tm /resync /force`: Obliga al PC del aula a sincronizar su reloj con el del Controlador de Dominio, ignorando cualquier restricción.

### `Add-Computer`
> **Descripción:** Une el equipo cliente a un dominio de Active Directory desde PowerShell — la alternativa por línea de comandos a `Configuración → Cuentas → Acceso profesional o educativo`.
> **Sintaxis:** `Add-Computer -DomainName [realm] -Credential [usuario] -Restart`

> [!example] Ejemplo de uso (Fase 8):
> - `Add-Computer -DomainName "BOOCHAN.SPACE" -Credential BOOCHAN\Administrator -Restart`: Une el cliente al dominio y reinicia automáticamente. El reinicio es **obligatorio** para que Windows aplique los cambios. Requiere que el túnel WireGuard esté activo — sin él, el PC del aula no puede alcanzar el Controlador de Dominio en AWS.

### `nslookup`
> **Descripción:** El detective de nombres de red: pregunta a qué servidor está resolviendo un nombre concreto.
> **Sintaxis:** `nslookup [nombre]`

> [!example] Ejemplo de uso (Fase 8):
> - `nslookup BOOCHAN.SPACE`: Debe devolver la IP del servidor por la que el cliente lo alcanza a través del túnel WireGuard. Si dice "no se encuentra el servidor", el DNS del cliente está mal configurado o la VPN no está activa.

### `net use`
> **Descripción:** Mapea una carpeta compartida de red como una unidad de disco local (letra de unidad).
> **Sintaxis:** `net use [letra]: \\[servidor]\[recurso] /user:[dominio\usuario] /persistent:yes`

> [!example] Ejemplo de uso (Fase 8):
> - `net use Z: \\WindowsServer.BOOCHAN.SPACE\prueba1 /user:BOOCHAN\user1 /persistent:yes`: Usa el nombre del servidor (no la IP) para que Windows emplee Kerberos en lugar de NTLM. `/persistent:yes` evita que la unidad desaparezca al reiniciar, aunque requiere que la VPN esté activa cuando Windows intente reconectarla.

### `Get-WindowsCapability` (RSAT)
> **Descripción:** Instala características bajo demanda de Windows, entre ellas las RSAT (Herramientas de administración remota del servidor), que permiten gestionar el dominio gráficamente desde el cliente.
> **Sintaxis:** `Get-WindowsCapability -Name [patrón] -Online | Add-WindowsCapability -Online`

> [!example] Ejemplo de uso (Fase 8):
> - `Get-WindowsCapability -Name RSAT.ActiveDirectory* -Online | Add-WindowsCapability -Online`: Requiere que el PC del aula tenga salida a Internet (independiente de la VPN), porque descarga el paquete desde los servidores de Microsoft.
