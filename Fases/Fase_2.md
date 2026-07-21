## 🧹 Fase 2: Preparación Inicial del Servidor

### Infraestructura de Servidores Cloud (AWS + Windows Server 2025)

> **[Módulo: SOR — Sistemas Operativos en Red]**
> **[U.T. 5: Administración en Windows Server - Instalación y Configuración]**
> **[RA.02]** Gestiona usuarios y grupos, interpretando especificaciones y aplicando herramientas del sistema.
>
> **Profesor:** Pedro Navarro Miralles  
> **Correo:** p.navarromiralles2@edu.gva.es  
> **Centro:** IES Jorge Juan (ALICANTE)
>
> **⏱️ Tiempo estimado:** ~1,25 horas (teoría + práctica + retos + troubleshooting)  
> **Requisitos:** Instancia `t3.large` (AWS) | Conectividad internet | Escritorio Remoto (RDP)

---

### 🎯 ¿Dónde Estamos?

> [!info] Vienes de Fase 1
> Creaste una instancia EC2 `t3.large` con **AMI Windows Server 2025 Base** en AWS. Está encendida, accesible por Escritorio Remoto, protegida por un Security Group que solo abre el puerto 3389. Pero viene "de fábrica": nombre genérico tipo `WIN-XXXXXXXXXXX`, sin actualizaciones aplicadas, y sin que nadie le haya confirmado cuál es su identidad definitiva dentro del proyecto.

> [!warning] El Problema
> A diferencia de Ubuntu Server (donde en BoochanV3 había que **purgar** Samba, CUPS y demonios heredados que ocupaban puertos), Windows Server no trae ningún servicio de directorio ni de archivos preinstalado que estorbe. El "problema" aquí es de otra naturaleza: el servidor tiene un nombre aleatorio que nadie reconocería en un log ni en una consulta DNS, y puede tener vulnerabilidades sin parchear desde el día en que se generó la imagen. Además, en AWS, la IP privada la asigna el DHCP de la VPC por defecto — no hace falta fijarla manualmente porque las instancias EC2 conservan su IP privada mientras no se termine la instancia, pero conviene comprobarla y anotarla antes de construir nada encima.

> [!success] Objetivo de esta Fase
> **Identidad:** Renombrar el servidor a `WindowsServer`, el nombre que usará todo el proyecto BoochanV3.1. **Red:** Verificar la IP privada que la VPC por defecto de AWS asignó al servidor (rango `172.31.x.x`) y comprobar que persiste tras reinicios. **Higiene:** Aplicar Windows Update para partir de un sistema parcheado antes de instalar ningún rol crítico como AD DS (Fase 4).

> [!tip] Hoja de Ruta
> 1. Conectarse al servidor por Escritorio Remoto (RDP) con la IP pública/Elástica de AWS
> 2. Renombrar el equipo con `Rename-Computer` a `WindowsServer`
> 3. Verificar la IP privada asignada por la VPC de AWS con `Get-NetIPConfiguration` (rango `172.31.x.x`)
> 4. Apuntar el DNS del propio adaptador a `127.0.0.1` (se explica por qué, aunque el DNS real no existirá hasta la Fase 4)
> 5. Instalar actualizaciones de Windows Update
> 6. Reiniciar y verificar identidad y red
>
> **Resultado Final:** Servidor con nombre `WindowsServer`, IP privada confirmada en el rango `172.31.x.x` de AWS, y parches de seguridad al día.
> **Siguiente:** Fase 3 (Conectividad VPN) — instalarás WireGuard para Windows para blindar el acceso remoto al servidor.

---

### 📚 Fundamento Teórico Avanzado

> [!abstract] 1. "De fábrica" en Windows Server vs. Linux: no hay que purgar, hay que nombrar
> En BoochanV3 (Ubuntu), la instalación por defecto traía Samba básico, CUPS y otros demonios que había que **purgar agresivamente** porque ocupaban puertos que el futuro Controlador de Dominio necesitaría (el temido conflicto del puerto 445). **Windows Server no tiene ese problema:** la instalación no trae ningún rol activado por defecto — ni siquiera AD DS, DNS o el propio Escritorio Remoto están instalados hasta que tú los añades explícitamente con `Install-WindowsFeature`. El trabajo de esta fase no es "demoler", es **dar identidad**: nombre de equipo e IP privada consolidada, los dos datos que todo lo demás (Fase 3, Fase 4) dará por hecho que ya existen.

> [!warning] 2. Por qué el nombre del equipo importa tanto como el FQDN en Linux
> En BoochanV3 configurabas `/etc/hosts` para que el servidor supiera su FQDN completo (`UbuntuServer.BOOCHAN.SPACE`). En Windows Server el equivalente conceptual es el **nombre de equipo** (Computer Name). Cuando en la Fase 4 promociones este servidor a Controlador de Dominio con `Install-ADDSForest`, Windows construirá automáticamente el FQDN del propio servidor concatenando el nombre de equipo con el Realm del dominio: `WindowsServer.BOOCHAN.SPACE`. Si en ese momento el nombre de equipo sigue siendo el genérico `WIN-XXXXXXXXXXX`, el dominio se creará igualmente, pero el servidor tendrá un nombre absurdo para siempre — cambiarlo después de promocionar a Controlador de Dominio es mucho más complicado (requiere herramientas adicionales y reinicios en cadena). Por eso se hace **ahora**, antes de instalar ningún rol.

> [!tip] 3. IP privada de la VPC de AWS: por qué aquí no hace falta "fijarla"
> En BoochanV3 (Ubuntu), la IP privada la asigna la VPC por defecto de AWS por DHCP y el alumno simplemente la consulta con `hostname -I` para usarla. En Windows Server pasa exactamente lo mismo a nivel de sistema operativo: **no vamos a fijar la IP a mano dentro de Windows** con `New-NetIPAddress` (eso es lo que haríamos en un laboratorio local de Hyper-V, donde no existe ningún DHCP). A diferencia de Azure (donde había que marcar explícitamente la asignación como "Estática" desde el portal), en **AWS la IP privada de una instancia EC2 permanece fija automáticamente** mientras la instancia no se termine (aunque se pare y arranque, conserva la misma IP privada dentro de su VPC). Por eso en esta fase solo hace falta **verificarla y anotarla**, no fijarla desde ningún panel adicional.

> [!info] 4. ¿Dónde queda el "`/etc/hosts`" de Windows?
> Windows Server también tiene un archivo equivalente, `C:\Windows\System32\drivers\etc\hosts`, pero **no lo vamos a tocar en esta fase, y probablemente nunca en este proyecto**. La razón es que, en cuanto promociones el servidor a Controlador de Dominio en la Fase 4, el propio servicio **DNS Server** (que se instala junto con AD DS) se convertirá en la fuente de verdad para resolver `WindowsServer.BOOCHAN.SPACE` y cualquier otro nombre del dominio — de forma dinámica y automática, sin mantenimiento manual de ficheros.

> [!important] 5. Windows Update antes de instalar roles críticos
> Instalar AD DS (Fase 4) sobre un sistema sin parchear es una mala práctica que en producción podría dejar el Controlador de Dominio expuesto a vulnerabilidades conocidas desde el primer día. El hábito profesional correcto es siempre el mismo: **actualizar primero, instalar roles después.**

### 📖 Diccionario de Conceptos Clave

> [!quote] Terminología Profesional
> - **Rename-Computer:** Cmdlet de PowerShell que cambia el nombre NetBIOS/DNS del equipo. Requiere reinicio para aplicarse.
> - **Get-NetIPConfiguration:** Cmdlet que muestra de un vistazo la IP, la puerta de enlace y el DNS de cada adaptador de red.
> - **Set-DnsClientServerAddress:** Cmdlet que fija qué servidor DNS debe consultar un adaptador de red.
> - **IP privada de VPC (AWS):** Dirección interna asignada por la VPC por defecto a la tarjeta de red virtual de la instancia EC2. Persiste mientras la instancia exista, sin necesidad de marcarla como "estática" en ningún panel — a diferencia de Azure.

---

### 🛠️ Procedimiento Práctico (BoochanV3.1)

> [!important] 🔌 Antes de empezar: Conéctate al servidor
> Todos los comandos de esta fase se ejecutan **dentro de tu instancia EC2 en AWS**, no en tu PC del aula. Abre el cliente de Escritorio Remoto de Windows (`mstsc`) o la app "Conexión a Escritorio remoto" en Mac, y conéctate con la IP pública/Elástica que anotaste en la Fase 1:
> ```
> Equipo: TU_IP_ELASTICA
> Usuario: Administrator
> ```
> Recuerda que la contraseña es la que descifraste en la consola de EC2 con tu fichero `.pem` (explicado en la Fase 1). Cuando veas el escritorio de Windows Server, abre PowerShell **como Administrador** (clic derecho sobre el icono → `Ejecutar como administrador`) y ya estás listo para continuar.

> [!example] Paso 1: Renombrar el Equipo
> > [!info] 📚 Diccionario de Comandos: Para entender la sintaxis exacta y ver ejemplos de los cmdlets de red y sistema que usaremos en esta fase, consulta el [[Diccionario_Comandos_Sistema]].
>
> ```powershell
> # Comprueba el nombre actual (genérico, tipo WIN-XXXXXXXXXXX)
> $env:COMPUTERNAME
>
> # Renombra el equipo y reinicia para aplicar el cambio
> Rename-Computer -NewName "WindowsServer" -Restart
> ```
>
> > [!tip] 💡 ¿Qué hace este comando?
> > - **`Rename-Computer`:** Cambia el nombre del equipo tanto a nivel NetBIOS como en la configuración de red interna de Windows. A diferencia de Linux (donde bastaba editar `/etc/hostname`), Windows **exige un reinicio completo** para que el nuevo nombre se propague a todos los servicios del sistema.
> > - **`-Restart`:** Evita el paso manual de reiniciar tú mismo; la instancia se reiniciará automáticamente en cuanto el cmdlet aplique el cambio.
> >
> > La instancia tardará uno o dos minutos en reiniciarse. **Perderás la sesión de Escritorio Remoto** — es normal. Espera un minuto y vuelve a conectarte con `mstsc` a la misma IP.

> [!example] Paso 2: Verificación de la IP Privada de la VPC
> Tras el reinicio, vuelve a abrir PowerShell como Administrador. Comprueba qué IP privada te ha asignado la VPC por defecto de AWS:
> ```powershell
> # Muestra IP, puerta de enlace y DNS del adaptador de red
> Get-NetIPConfiguration
> ```
>
> > [!tip] 💡 ¿Qué IP anoto?
> > Busca la dirección que empieza por **`172.31.`** — es la IP privada de la VPC por defecto de AWS (por ejemplo, `172.31.20.45`). Es la que usarán todas las fases siguientes para identificar internamente al servidor. Anótala junto a la IP pública/Elástica que ya tenías de la Fase 1.
> >
> > **⚠️ AWS no usa el rango `10.x` por defecto** (a diferencia de Azure). La VPC por defecto reparte direcciones del rango **`172.31.0.0/16`**, por eso tu IP empieza por `172.31.` y no por `10.`.
> >
> > > [!warning] Si estás en un AWS Academy Learner Lab y tu IP NO empieza por `172.31.`
> > > Algunos laboratorios usan una VPC con un rango distinto (por ejemplo `10.0.x.x` o `172.30.x.x`). No pasa nada: la regla general es coger **la IP privada IPv4** que muestra `Get-NetIPConfiguration` en el adaptador con puerta de enlace activa (`Default Gateway` no vacío). Esa es la IP que usarás como referencia interna del servidor, empiece por `172.31.`, `172.30.` o lo que sea.
>
> Verifica también el nombre completo del equipo:
> ```powershell
> # Debe devolver: WindowsServer
> $env:COMPUTERNAME
> ```
>
> > [!info] 💡 ¿Por qué no hace falta fijar esta IP como "estática" desde ningún panel de AWS?
> > A diferencia de Azure (donde una VM podía perder su IP privada al reiniciar si no se marcaba explícitamente como "Estática"), en AWS **la IP privada de una instancia EC2 queda asociada a su interfaz de red mientras la instancia exista**: sobrevive a paradas y arranques (`Stop`/`Start`), y solo cambiaría si terminases (`Terminate`) la instancia y crearas otra nueva. Por eso en este proyecto solo la verificamos, sin ningún paso adicional en la consola de AWS.

> [!example] Paso 3: Preparar el DNS del Adaptador para la Fase 4
> Fija el DNS del adaptador de red al propio servidor, en preparación para cuando instales el rol DNS en la Fase 4:
> ```powershell
> # Averigua el índice del adaptador de red activo
> Get-NetAdapter
>
> # Sustituye 4 por el ifIndex real de tu adaptador (normalmente el único que aparece "Up")
> Set-DnsClientServerAddress -InterfaceIndex 4 -ServerAddresses "127.0.0.1"
> ```
>
> > [!important] 💡 ¿Por qué apuntar el DNS a `127.0.0.1` si todavía no hay servicio DNS instalado?
> > Es una preparación deliberada para la Fase 4. Cuando promociones el servidor a Controlador de Dominio con `Install-ADDSForest -InstallDNS`, el propio asistente instalará y activará el servicio **DNS Server** en esta misma máquina. Dejar el adaptador ya apuntando a `127.0.0.1` desde ahora evita un paso de reconfiguración posterior y refuerza la idea de que, en un dominio Active Directory, **el Controlador de Dominio se consulta siempre a sí mismo** para resolver nombres del dominio — el mismo principio que en BoochanV3 se lograba editando `/etc/hosts` y con `chattr +i` sobre `resolv.conf`. Hasta la Fase 4, este ajuste no tiene efecto práctico, pero deja el terreno preparado.
>
> Verifica la configuración aplicada:
> ```powershell
> Get-DnsClientServerAddress -InterfaceIndex 4
> ```

> [!example] Paso 4: Instalación de Actualizaciones de Windows Update
> Antes de instalar cualquier rol crítico como AD DS (Fase 4), parchea el sistema. El módulo `PSWindowsUpdate` no viene instalado por defecto; lo instalamos desde el repositorio oficial de PowerShell Gallery:
> ```powershell
> # Instala el proveedor NuGet si el sistema lo solicita
> Install-PackageProvider -Name NuGet -MinimumVersion 2.8.5.201 -Force
>
> # Instala el módulo de gestión de Windows Update
> Install-Module -Name PSWindowsUpdate -Force
>
> # Busca e instala todas las actualizaciones disponibles, reiniciando si hace falta
> Get-WindowsUpdate -Install -AcceptAll -AutoReboot
> ```
>
> > [!caution] ⚠️ Este proceso puede tardar bastante
> > Dependiendo de cuántas actualizaciones haya pendientes desde la imagen base de AWS, este paso puede tardar entre 10 y 30 minutos, e incluir uno o varios reinicios automáticos. Es normal. No pares la instancia manualmente durante este proceso — perderás la sesión RDP, espera unos minutos y vuelve a conectarte.
>
> Cuando termine, verifica que no quedan actualizaciones pendientes:
> ```powershell
> Get-WindowsUpdate
> ```
> Si la lista sale vacía, el sistema está al día.

> [!example] Paso 5: Verificación Final de Identidad y Red
> Confirma que todo quedó correctamente aplicado:
> ```powershell
> # Debe devolver: WindowsServer
> $env:COMPUTERNAME
>
> # Debe mostrar la IP 172.31.x.x del adaptador
> Get-NetIPAddress -AddressFamily IPv4 | Where-Object { $_.IPAddress -like "172.31.*" }
> ```
>
> > [!info] 📚 Recurso: Para editar texto rápido en Windows Server usa el Bloc de notas (`notepad`) — no existe `nano` como en Linux. Para los comandos de administración, consulta el [[Diccionario_Comandos_Sistema]].

---

### 🚩 Resolución de Problemas y Evaluación

> [!bug] Troubleshooting (¿Algo no va bien?)
> | Problema | Causa Probable | Solución Sugerida |
> | :--- | :--- | :--- |
> | `Rename-Computer` no pide reinicio y el nombre no cambia. | Se ejecutó sin permisos de administrador. | Cierra PowerShell y vuelve a abrirlo con `Ejecutar como administrador`. Repite el comando. |
> | Tras el `-Restart` no puedo reconectar por RDP. | La instancia todavía está reiniciando. | Espera 1-2 minutos y vuelve a intentar la conexión con `mstsc`. |
> | La IP privada que veo no empieza por `172.31.`. | Tu Learner Lab usa una VPC con un rango distinto. | No pasa nada, usa la única IP privada IPv4 que te muestre `Get-NetIPConfiguration` con puerta de enlace asignada. |
> | `Install-Module -Name PSWindowsUpdate` falla o se queda colgado. | Problema temporal de repositorio de PowerShell Gallery no confiado. | Responde `S` (Sí) si pregunta por confiar en el repositorio, o añade `-Force` al comando. |

> [!help] Preguntas Críticas (Autoevaluación)
> 1. ¿Por qué en Windows Server no hace falta "purgar" nada, a diferencia de Ubuntu Server en BoochanV3?
> 2. ¿Qué ocurre si promocionas el servidor a Controlador de Dominio (Fase 4) sin haberlo renombrado antes? ¿Por qué es tan importante hacerlo en este orden?
> 3. ¿Por qué en AWS no hace falta marcar la IP privada como "Estática" desde ningún panel, a diferencia de lo que ocurriría en otros proveedores cloud?
> 4. 🔬 **Reto práctico:** Ejecuta `Get-NetIPConfiguration` y compara la salida con `Get-NetIPAddress`. ¿Qué información adicional aporta el primero (puerta de enlace, DNS) que el segundo no muestra directamente?
> 5. 🔬 **Reto práctico:** Ejecuta `Get-Process | Sort-Object WS -Descending | Select-Object -First 5` para ver los 5 procesos que más RAM consumen ahora mismo. Anota el total de RAM libre con `Get-Counter '\Memory\Available MBytes'`. Guarda ese dato — lo compararás con la Fase 4, cuando AD DS esté instalado, para ver cuánta RAM consume el dominio.

---

> [!caution] 🛑 Auditoría y Evaluación (RA.02)
> El alumno debe demostrar que el servidor tiene el nombre e IP correctos antes de avanzar. **Riesgo Crítico:** Si el nombre de equipo no es `WindowsServer` antes de la Fase 4, el FQDN del propio Controlador de Dominio quedará con un nombre erróneo de forma permanente.

> [!success] 🏁 Punto de Control (Antes de seguir)
> - [ ] ¿`$env:COMPUTERNAME` devuelve exactamente `WindowsServer`?
> - [ ] ¿`Get-NetIPConfiguration` muestra una IP en el rango `172.31.x.x` (o el rango de tu VPC)?
> - [ ] ¿El DNS del adaptador apunta a `127.0.0.1`?
> - [ ] ¿`Get-WindowsUpdate` no muestra actualizaciones pendientes?
