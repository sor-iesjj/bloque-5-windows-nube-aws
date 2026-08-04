## Fase 2 · Apartado 6 — 🛠️ Procedimiento práctico

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Preparación Inicial del Servidor**
> 🧭 Índice de la fase: [[Fase_2]]
>
> **📍 Cuándo se lee:** **Con la VM delante.** Aquí está el trabajo.

---

> [!important] 🔌 Antes de empezar: Conéctate al servidor
> Todos los comandos de esta fase se ejecutan **dentro de tu instancia EC2 en AWS**, no en tu PC del aula. Abre el cliente de Escritorio Remoto de Windows (`mstsc`) o la app "Conexión a Escritorio remoto" en Mac, y conéctate con la IP pública/Elástica que anotaste en la Fase 1:
> ```
> Equipo: TU_IP_ELASTICA
> Usuario: Administrator
> ```
> Recuerda que la contraseña es la que descifraste en la consola de EC2 con tu fichero `.pem` (explicado en la Fase 1). Cuando veas el escritorio de Windows Server, abre PowerShell **como Administrador** (clic derecho sobre el icono → `Ejecutar como administrador`) y ya estás listo para continuar.

> [!example] 🎬 Antes de empezar (todavía SIN grabar, y luego arranca)
> Ya conoces el método desde los prerrequisitos, así que va solo el recordatorio:
> 1. **Crea la entrada de apuntes** de esta fase (`v3-1-fase-2-preparacion-inicial-del-servidor.md`) con su estructura, vacía.
> 2. **Léete los 5 pasos** del procedimiento enteros, para no atascarte a mitad del vídeo.
> 3. Ten **OBS** listo y comprueba **pantalla y micrófono**.
>
> Cuando lo tengas: **arranca la grabación, preséntate y muestra tu identidad**. A partir de ahí, **todo queda grabado** — incluido cualquier paso previo de preparación que venga a continuación.

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

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_2.5_Fundamento_Teorico]] | [[Fase_2]] | [[Fase_2.7_Resolucion_Problemas]] |
