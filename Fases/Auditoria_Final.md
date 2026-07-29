## 🛡️ Auditoría Final y Hardening (Cierre de Seguridad)

> **[RA.06]** Diseña e implementa soluciones de seguridad perimetral y auditoría de sistemas.

### 📚 Fundamento Teórico: El Principio de "Zero Trust"

Para terminar el proyecto, debemos aplicar la filosofía **Zero Trust** (Confianza Cero). Hasta ahora, hemos dejado algunos puertos abiertos a todo Internet para facilitar la configuración inicial. Un administrador profesional, una vez terminado el trabajo, debe "cerrar el castillo" y solo permitir el paso a quien esté dentro de la muralla (la VPN).

> [!info] Dos capas de defensa, no una
> A diferencia de un laboratorio local (donde el único firewall posible vive dentro del propio sistema operativo del servidor), aquí tienes **dos murallas independientes** que cerrar: el **Security Group de AWS** (el firewall perimetral gestionado por el proveedor cloud, que filtra el tráfico *antes* de que llegue siquiera a la tarjeta de red de la instancia EC2) y el **Firewall de Windows Defender con Seguridad Avanzada** (el firewall nativo del propio Windows Server, que filtra *dentro* de la máquina). Cerrar solo una de las dos murallas deja la otra como puerta trasera: un administrador riguroso bastiona ambas.

### 📖 Diccionario de Conceptos Clave

- **Hardening:** El proceso de "endurecer" un servidor eliminando servicios innecesarios y cerrando puertos.
- **Whitelist (Lista Blanca):** Configuración que bloquea todo por defecto y solo permite el paso a IPs específicas.
- **Zero Trust:** Estrategia de seguridad que asume que la red ya está comprometida y exige verificación constante.
- **Firewall de Windows Defender con Seguridad Avanzada:** Componente nativo de Windows Server que filtra tráfico entrante y saliente mediante reglas de entrada/salida, gestionable por GUI (`wf.msc`) o por PowerShell (`New-NetFirewallRule`, `Get-NetFirewallRule`).
- **Perfil de Firewall (Domain/Private/Public):** Windows aplica un conjunto de reglas distinto según el tipo de red detectado en cada adaptador. En este proyecto, el perfil **Dominio** es el que gobierna el servidor una vez está unido a su propio dominio.

---

> [!important] 📹 Obligaciones de grabación (LÉEME — es igual en TODAS las fases)
> Esta práctica se **graba entera con OBS**, de principio a fin. No es un repaso al final: quiero ver **cómo lo haces tú**.
> 1. **Prepárate primero (sin grabar):** comprueba lo necesario, **léete el procedimiento entero** y **crea la entrada de apuntes de esta fase** en Obsidian: fichero `v3-1-auditoria-final-hardening-y-cierre-de-seguridad.md` dentro de `00_Apuntes/Trimestre_N/B5_Windows_Nube/`, con la estructura de la Fase 0.1 y **vacía**. Rellenarla es cosa tuya, después.
> 2. **Arranca OBS y PRESÉNTATE:** *"Hola, me llamo [Nombre], 2.º SMR, y en este vídeo voy a explicar la Auditoría Final de Boochan V3.1 — Hardening y cierre de seguridad."* Y **muestra algo que demuestre que eres tú** (tu perfil de GitHub, tu Teams o tu correo `@alu.edu.gva.es`). Di qué vas a hacer.
> 3. **Graba TODO el procedimiento**, explicando cada paso en voz alta mientras lo haces.
> 4. **Timestamps SIEMPRE** en la descripción: `00:00 Presentación` + uno por cada paso.
> 5. **Al terminar:** nombra el vídeo `V3.1 · Auditoría Final — Hardening y cierre de seguridad`, súbelo a tu playlist de YouTube **`B5_Windows_Nube`** (No listado) y **copia su enlace**.
> 6. **~8-10 min.** Esta fase es más larga que las de prerrequisitos: ve al grano, pero no te saltes pasos. Si se te va mucho, **pártela en dos vídeos** y ponlos los dos en la entrada.
> 7. **El enlace del vídeo va DENTRO de tu entrada de apuntes**, en el apartado `Enlace al vídeo explicativo`. Ahí, no en un papel.
> 8. **La entrega va por la TAREA de Teams.** Abriré una tarea que cubrirá **esta fase y otras**; te llegará notificación con fecha límite.

---

### 🛠️ Procedimiento Práctico de Hardening

> [!example] Paso 1: Cierre de Puertos en el Security Group de AWS
> Ve a **AWS Console → EC2 → Grupos de seguridad → tu grupo → Editar reglas de entrada** y restringe el **Origen** de las reglas sensibles para aplicar la máxima seguridad. Recuerda que este proyecto **no usa SSH (puerto 22)** — el acceso administrativo es por **RDP (puerto 3389)**:
>
> | Regla | Puerto | Protocolo | Origen actual | Nuevo origen |
> | :--- | :--- | :--- | :--- | :--- |
> | **RDP** | 3389 | TCP | `Anywhere-IPv4` (`0.0.0.0/0`) | `10.0.0.0/24` |
> | **SMB_Files** | 445 | TCP | `Anywhere-IPv4` | `10.0.0.0/24` |
> | Kerberos_TCP | 88 | TCP | `Anywhere-IPv4` | `10.0.0.0/24` |
> | Kerberos_UDP | 88 | UDP | `Anywhere-IPv4` | `10.0.0.0/24` |
> | DNS_TCP | 53 | TCP | `Anywhere-IPv4` | `10.0.0.0/24` |
> | DNS_UDP | 53 | UDP | `Anywhere-IPv4` | `10.0.0.0/24` |
> | RPC_Endpoint | 135 | TCP | `Anywhere-IPv4` | `10.0.0.0/24` |
> | LDAP_TCP | 389 | TCP | `Anywhere-IPv4` | `10.0.0.0/24` |
> | LDAP_UDP | 389 | UDP | `Anywhere-IPv4` | `10.0.0.0/24` |
> | LDAPS | 636 | TCP | `Anywhere-IPv4` | `10.0.0.0/24` |
> | RPC_Dinamico | 49152-65535 | TCP | `Anywhere-IPv4` | `10.0.0.0/24` |
> | Kerberos_Pass_TCP | 464 | TCP | `Anywhere-IPv4` | `10.0.0.0/24` |
> | Kerberos_Pass_UDP | 464 | UDP | `Anywhere-IPv4` | `10.0.0.0/24` |
> | NTP_Time | 123 | UDP | `Anywhere-IPv4` | `10.0.0.0/24` |
>
> Para cada regla de la tabla: entra en el Security Group de tu instancia EC2 → pestaña **"Reglas de entrada"** → **Editar reglas de entrada**, y en el campo **"Origen"** cambia `Anywhere-IPv4` por `Personalizado`, introduciendo `10.0.0.0/24` en el cuadro de texto. Pulsa **Guardar reglas**.
>
> > [!note] 💡 En AWS se restringe el "Origen", no se crea una regla "Denegar"
> > Recuerda que un Security Group solo tiene reglas que *permiten*. Para bloquear internet en estos puertos no se añade una regla de denegación (no existe): se **estrecha el Origen** de la regla que permite, dejando pasar solo el rango `10.0.0.0/24` de la VPN. Todo lo demás queda bloqueado por defecto.
>
> > [!important] ⚠️ El puerto 51820 (WireGuard) NO se toca
> > El puerto UDP `51820` debe seguir con origen `Anywhere-IPv4`: es la puerta por la que el túnel VPN establece la conexión inicial desde el PC del aula, que aún no está "dentro" de la red privada. Si lo restringes también a `10.0.0.0/24`, te quedarás fuera de tu propio servidor — es una paradoja: para entrar en la VPN necesitas que ese puerto acepte tráfico desde fuera de la VPN.
>
> *Resultado: A partir de ahora, nadie en Internet podrá siquiera intentar atacar estos puertos. Solo los alumnos conectados a la VPN podrán administrar el servidor o autenticarse contra el dominio.*

> [!example] Paso 2: Activar y configurar el Firewall de Windows Defender en el servidor
> El Security Group de AWS es la primera muralla, pero no la única. Conéctate al servidor (por RDP a través del túnel WireGuard) y ejecuta en **PowerShell como Administrador** para levantar la segunda muralla, esta vez dentro del propio Windows Server:
> ```powershell
> # Asegura que el firewall está activo en los tres perfiles
> Set-NetFirewallProfile -Profile Domain,Private,Public -Enabled True
>
> # Política por defecto del perfil de Dominio: bloquear entrante, permitir saliente
> Set-NetFirewallProfile -Profile Domain -DefaultInboundAction Block -DefaultOutboundAction Allow
>
> # RDP (administración) SOLO desde la red de la VPN
> New-NetFirewallRule -DisplayName "RDP - Solo VPN BOOCHAN" -Direction Inbound `
>     -RemoteAddress 10.0.0.0/24 -Action Allow -Protocol TCP -LocalPort 3389
>
> # Servicios de dominio (Kerberos, DNS, LDAP, SMB, LDAPS...) SOLO desde la red de la VPN
> New-NetFirewallRule -DisplayName "AD DS - Solo VPN BOOCHAN (TCP)" -Direction Inbound `
>     -RemoteAddress 10.0.0.0/24 -Action Allow `
>     -Protocol TCP -LocalPort 88,53,135,389,445,636,464
>
> New-NetFirewallRule -DisplayName "AD DS - Solo VPN BOOCHAN (UDP)" -Direction Inbound `
>     -RemoteAddress 10.0.0.0/24 -Action Allow `
>     -Protocol UDP -LocalPort 88,53,123,464
>
> # RPC dinámico (necesario para varias funciones de AD DS: replicación, gestión remota...)
> New-NetFirewallRule -DisplayName "AD DS - RPC dinamico VPN BOOCHAN" -Direction Inbound `
>     -RemoteAddress 10.0.0.0/24 -Action Allow -Protocol TCP -LocalPort RPC
> ```
>
> > [!tip] 💡 ¿Por qué configurar esto si el Security Group de AWS ya filtra el tráfico?
> > Es exactamente el principio de "defensa en profundidad" (*defense in depth*): si por error un administrador vuelve a abrir un puerto del Security Group a `Anywhere-IPv4` en el futuro (por ejemplo, para depurar un problema, y se olvida de cerrarlo después), el Firewall de Windows Defender dentro del propio servidor sigue bloqueando ese tráfico igualmente. Ninguna de las dos murallas debe depender de que la otra esté bien configurada — cada una debe poder detener el ataque por sí sola.
>
> > [!note] 💡 No hace falta abrir el 51820 en el Firewall de Windows
> > El túnel WireGuard termina en el propio adaptador de red virtual que crea WireGuard (`wg0`/`WireGuard Tunnel`), no en un puerto de escucha "de aplicación" del servidor gestionado por Windows Defender de la misma forma que RDP o SMB. El filtrado del puerto 51820 ya lo hace el Security Group de AWS (Paso 1); el Firewall de Windows Defender se ocupa de lo que ocurre **una vez el tráfico ya está dentro** del túnel.
>
> > [!tip] 💡 Alternativa gráfica: `wf.msc`
> > Todo lo anterior también se puede hacer desde el snap-in gráfico **"Firewall de Windows Defender con Seguridad Avanzada"** (ejecuta `wf.msc`): en el panel izquierdo, **Reglas de entrada** → **Nueva regla...**, y en el asistente eliges Puerto, el ámbito remoto ("Estas direcciones IP" → `10.0.0.0/24`), y la acción "Permitir la conexión". Es exactamente el mismo resultado que los cmdlets de PowerShell, solo que paso a paso con ventanas.

> [!example] Paso 3: Verificar las reglas aplicadas
> ```powershell
> Get-NetFirewallProfile | Select-Object Name, Enabled, DefaultInboundAction, DefaultOutboundAction
> Get-NetFirewallRule -DisplayName "AD DS*" | Select-Object DisplayName, Enabled, Direction, Action
> Get-NetFirewallRule -DisplayName "RDP*" | Select-Object DisplayName, Enabled, Direction, Action
> ```
> Deberías ver el perfil `Domain` con `DefaultInboundAction: Block` y las reglas `AD DS - Solo VPN BOOCHAN (TCP/UDP)`, `AD DS - RPC dinamico VPN BOOCHAN` y `RDP - Solo VPN BOOCHAN` habilitadas (`Enabled: True`).
>
> > [!caution] ⚠️ No te quedes fuera de tu propio servidor
> > Si estás conectado por RDP desde una IP que **no** está dentro de `10.0.0.0/24` (por ejemplo, porque te conectaste directamente por la IP pública/Elastic IP de AWS sin pasar por la VPN), al aplicar `DefaultInboundAction Block` te quedarás fuera del servidor de inmediato. Verifica siempre desde qué IP estás conectado (`quser` o `Get-NetTCPConnection -State Established` en el propio servidor) antes de aplicar la política por defecto, y ten siempre a mano **AWS Systems Manager (SSM) Session Manager** como vía de rescate si te bloqueas accidentalmente — funciona sin necesidad de RDP ni SSH, directamente desde la consola de AWS.

> [!example] Paso 4: Auditoría Local de Servicios
> Ejecuta este comando en PowerShell del servidor para verificar que no hay "polizontes" o servicios desconocidos escuchando en red:
> ```powershell
> # Listar procesos que escuchan en red con su nombre
> Get-NetTCPConnection -State Listen | Select-Object LocalAddress, LocalPort, OwningProcess |
>     Sort-Object LocalPort | ForEach-Object {
>         $_ | Add-Member -NotePropertyName ProcessName -NotePropertyValue (Get-Process -Id $_.OwningProcess).ProcessName -PassThru
>     }
> ```
>
> > [!tip] 💡 ¿Qué hace este comando?
> > - **`Get-NetTCPConnection -State Listen`:** Equivalente PowerShell nativo de `ss -tunlp` / `netstat -ano`, lista todos los puertos TCP en escucha.
> > - **`OwningProcess`:** El PID del proceso dueño de ese puerto; se resuelve a nombre (`lsass`, `dns`, `svchost` con el módulo Netlogon/SMB, etc.) con `Get-Process`.
>
> Compara la salida con las reglas del Paso 3: cualquier puerto que aparezca en escucha y no esté cubierto por una regla de entrada permitida explícitamente (o que no reconozcas) merece investigación.

> [!example] Paso 5: Cierre correcto de la sesión de AWS Academy Learner Lab
> El hardening de red no termina el proyecto: falta cerrar correctamente el laboratorio para no desperdiciar crédito, tal y como se advirtió en la Fase 1.
>
> 1. Guarda cualquier trabajo pendiente y verifica que no necesitas la instancia el resto del día.
> 2. Vuelve a la página del **Learner Lab** de AWS Academy (no a la consola normal de AWS).
> 3. Pulsa el botón **"End Lab"** (no "Stop Lab" ni cerrar simplemente la pestaña).
> 4. Confirma la acción. El laboratorio completo (instancia EC2, Elastic IP, Security Group) se congela sin consumir más crédito.
>
> > [!warning] 💸 "End Lab" ≠ cerrar la pestaña del navegador
> > Cerrar la pestaña o apagar tu propio PC **no detiene nada en AWS** — la instancia `WindowsServer` (t3.large) sigue encendida y consumiendo el crédito del Learner Lab hasta que alguien pulse explícitamente **"End Lab"**. Es exactamente la misma advertencia de coste que se dio en la Fase 1: en AWS Academy el crédito es limitado y no hay tarjeta de crédito de respaldo — si se agota, el laboratorio deja de funcionar hasta la siguiente asignación. Acostúmbrate a terminar siempre la sesión con este paso, sea cual sea la fase en la que estés trabajando ese día.

---

### ❓ Preguntas Críticas de Cierre
1. ¿Por qué en este proyecto el hardening final se hace tanto en el Security Group de AWS como en el Firewall de Windows Defender, y no basta con uno solo de los dos?
2. ¿Qué diferencia de seguridad hay entre dejar el puerto `51820/udp` (WireGuard) abierto "a cualquiera" y dejar el puerto `445` (SMB) o `3389` (RDP) abiertos "a cualquiera"? ¿Por qué el primero es aceptable y los otros no?
3. Si después de cerrar los puertos ya no puedes conectar por RDP, ¿qué es lo primero que deberías comprobar en tu cliente VPN?
4. ¿Qué ventaja tiene cambiar el origen del tráfico en el Security Group de AWS en lugar de confiar solo en el firewall interno de Windows?
5. ¿Qué significa que un servidor esté "bastionado" (*Hardened*)?
6. Según `Get-NetTCPConnection -State Listen`, ¿qué proceso suele aparecer como dueño del puerto 445?
7. ¿Por qué es importante pulsar "End Lab" y no simplemente cerrar la pestaña del navegador al terminar la sesión de trabajo?

---

> [!success] 🏁 Proyecto Finalizado
> ¡Enhorabuena! Has construido una infraestructura híbrida profesional, segura y escalable en la nube: un Controlador de Dominio Windows Server 2025 en AWS, con cuotas de disco, permisos NTFS granulares y un cliente Windows 11 físico integrado a través de un túnel cifrado WireGuard, protegido por dos capas de defensa independientes — el Security Group de AWS en el perímetro y el Firewall de Windows Defender dentro del propio servidor — que solo confían en la red de la VPN (`10.0.0.0/24`). Recuerda cerrar siempre la sesión del Learner Lab con **"End Lab"** al terminar.

---

### ✅ Entregables y cierre

> [!abstract] Qué tienes que tener hecho al acabar esta fase
> | Entregable | Dónde vive | Qué debe contener |
> | :--- | :--- | :--- |
> | **Entrada de apuntes** | `00_Apuntes/Trimestre_N/B5_Windows_Nube/v3-1-auditoria-final-hardening-y-cierre-de-seguridad.md` | Estructura completa + **respuestas a las Preguntas Críticas y al 🔬 Reto** + **enlace del vídeo** |
> | **Vídeo** | Playlist `B5_Windows_Nube` (No listado) | Nombrado `V3.1 · Auditoría Final — Hardening y cierre de seguridad`, con presentación, identidad y timestamps |
> | **Repositorio** | Tu repo de apuntes en GitHub | La entrada, subida con `git add` → `commit` → `push` |
>
> > [!danger] ⚠️ Las respuestas van en la ENTRADA, no en un documento aparte
> > Las **Preguntas Críticas** y el **🔬 Reto** de más arriba no son decorativos: son la parte de la fase que demuestra que has entendido lo que has hecho, y no solo que has sabido copiar comandos. Se contestan **con tus palabras**, en el apartado `Respuesta a las preguntas` de tu entrada.
> > Una fase con el procedimiento perfecto y las preguntas en blanco está **incompleta**.
>
> > [!info] 🏷️ Por qué el nombre lleva `V3.1` delante
> > Porque el proyecto Boochan existe en **varias versiones** (VirtualBox, Hyper-V, Azure, AWS…) y algunas comparten bloque y playlist. Sin la etiqueta, la Fase 4 de Azure y la de AWS se llamarían **exactamente igual** y no habría forma de distinguirlas. Con ella, tu carpeta y tu playlist dicen siempre **qué versión hiciste**.
>
> > [!success] 🎯 Criterio de éxito
> > Abro tu repositorio, encuentro la entrada de esta fase, y dentro está: qué has hecho, qué has entendido, qué dudas te han quedado y el enlace al vídeo donde se te ve haciéndolo. Si falta el enlace o faltan las respuestas, la fase **no cuenta como entregada**.
>
> > [!tip] 💡 ¿Y si la fase te ha llevado tres clases?
> > **Una fase, una entrada.** No creas un fichero por día: abres el mismo y sigues escribiendo. Haz `commit` y `push` **al terminar cada sesión**, para no perder nunca más de un día de trabajo.
