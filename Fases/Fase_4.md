## 👑 Fase 4: Aprovisionamiento del Dominio (AD DS nativo)

### Infraestructura de Servidores Cloud (AWS + Windows Server 2025)

> **[Módulo: SOR — Sistemas Operativos en Red]**
> **[U.T. 10: Integración de Sistemas Operativos - Servidor de Dominio]**
> **[RA.03]** Realiza tareas de gestión sobre dominios identificando necesidades y aplicando herramientas.
>
> **Profesor:** Pedro Navarro Miralles  
> **Correo:** p.navarromiralles2@edu.gva.es  
> **Centro:** IES Jorge Juan (ALICANTE)
>
> **⏱️ Tiempo estimado:** ~2,5 horas (teoría + práctica + retos + troubleshooting)  
> **Requisitos:** Instancia `t3.large` (AWS) | AWS Console | Túnel WireGuard operativo (Fase 3)

---

> [!important] 📹 Obligaciones de grabación (LÉEME — es igual en TODAS las fases)
> Esta práctica se **graba entera con OBS**, de principio a fin. No es un repaso al final: quiero ver **cómo lo haces tú**.
> 1. **Prepárate primero (sin grabar):** comprueba lo necesario, **léete el procedimiento entero** y **crea la entrada de apuntes de esta fase** en Obsidian: fichero `v3-1-fase-4-aprovisionamiento-del-dominio-ad-ds-nati.md` dentro de `00_Apuntes/Trimestre_N/B5_Windows_Nube/`, con la estructura de la Fase 0.1 y **vacía**. Rellenarla es cosa tuya, después.
> 2. **Arranca OBS y PRESÉNTATE:** *"Hola, me llamo [Nombre], 2.º SMR, y en este vídeo voy a explicar la Fase 4 de Boochan V3.1 — Aprovisionamiento del Dominio (AD DS nativo)."* Y **muestra algo que demuestre que eres tú** (tu perfil de GitHub, tu Teams o tu correo `@alu.edu.gva.es`). Di qué vas a hacer.
> 3. **Graba TODO el procedimiento**, explicando cada paso en voz alta mientras lo haces.
> 4. **Timestamps SIEMPRE** en la descripción: `00:00 Presentación` + uno por cada paso.
> 5. **Al terminar:** nombra el vídeo `V3.1 · Fase 4 — Aprovisionamiento del Dominio (AD DS nativo)`, súbelo a tu playlist de YouTube **`B5_Windows_Nube`** (No listado) y **copia su enlace**.
> 6. **~8-10 min.** Esta fase es más larga que las de prerrequisitos: ve al grano, pero no te saltes pasos. Si se te va mucho, **pártela en dos vídeos** y ponlos los dos en la entrada.
> 7. **El enlace del vídeo va DENTRO de tu entrada de apuntes**, en el apartado `Enlace al vídeo explicativo`. Ahí, no en un papel.
> 8. **La entrega va por la TAREA de Teams.** Abriré una tarea que cubrirá **esta fase y otras**; te llegará notificación con fecha límite.

---

### 🎯 ¿Dónde Estamos?

> [!info] Vienes de Fase 3
> Tienes un servidor con nombre correcto (`WindowsServer`), IP privada confirmada en la VPC de AWS, accesible de forma cifrada a través de un túnel WireGuard (`10.0.0.1`). Ahora necesitas darle la funcionalidad de un verdadero **Controlador de Dominio** — el "cerebro" que gestiona usuarios, grupos, autenticación y autorización.

> [!warning] El Problema
> Sin un dominio, Windows 11 en el aula es un equipo aislado. Los usuarios se loguean localmente (usuario/contraseña guardados en el PC). No hay forma centralizada de gestionar identidades, no hay Single Sign-On, no hay políticas de grupo. Si necesitas cambiar la contraseña de un usuario, debes hacerlo en cada PC manualmente. Además, Kerberos (el protocolo de seguridad profesional) requiere un dominio para funcionar.

> [!success] Objetivo de esta Fase
> Instalar el rol **AD DS (Active Directory Domain Services)** en el servidor y promocionarlo con `Install-ADDSForest`. Esto creará el dominio **`BOOCHAN.SPACE`** (Realm) / **`BOOCHAN`** (NetBIOS) como un "reino" Kerberos con servicios interdependientes: base de datos de directorio (NTDS), DNS integrado, Kerberos (autenticación) y replicación. Desde ahora, los usuarios se autenticarán contra el dominio, no contra máquinas individuales.

> [!tip] Hoja de Ruta
> 1. Abrir 13 puertos en el Security Group de AWS (Kerberos, DNS, LDAP, SMB, RPC, NTP — todo lo que AD necesita)
> 2. Instalar el rol AD DS con `Install-WindowsFeature AD-Domain-Services -IncludeManagementTools`
> 3. Promocionar el servidor a Controlador de Dominio con `Install-ADDSForest` (tarda varios minutos y reinicia solo)
> 4. Verificar que los servicios `NTDS`, `DNS` y `Kdc` están activos tras el reinicio
> 5. Comprobar que el DNS del propio servidor apunta a sí mismo
> 6. Validar que Kerberos funciona: `Resolve-DnsName _kerberos._tcp.BOOCHAN.SPACE`
> 7. Listar usuarios creados automáticamente: `Get-ADUser -Filter *` (verás Administrator, krbtgt, etc.)
>
> **Resultado Final:** Dominio `BOOCHAN.SPACE` completamente provisionado y operativo. El servidor es ahora un verdadero Controlador de Dominio profesional.
> **Siguiente:** Fase 5 (Usuarios) — crearás usuarios del dominio (user1, user2) con `New-ADUser` y los organizarás en unidades organizativas.

---

### 📚 Fundamento Teórico

> [!abstract] 1. El "Cerebro" de la Red: Active Directory (AD)
> Estamos creando el **Active Directory**. Este es el "Cerebro" que gestiona la base de datos de todos los objetos de la red: usuarios, grupos y ordenadores. El rol AD DS instala tres servicios vitales para que esto funcione:
> *   **NTDS (base de datos del directorio):** El equivalente al LDAP de Samba — la base de datos jerárquica donde vive cada objeto del dominio.
> *   **Kerberos:** El sistema de "tickets" de seguridad (como un pase VIP de un festival).
> *   **DNS integrado:** El propio rol DNS Server, instalado junto con AD DS, gestiona los registros SRV que indican dónde están los servicios de red.

> [!important] 2. De lo artesanal a lo asistido: `provision_boochan.sh` vs. `Install-ADDSForest`
> En BoochanV3 (Samba), provisionar el dominio requería un **script externo** (`provision_boochan.sh`), clonado desde un repositorio Git, que ejecutaba `samba-tool domain provision`, configuraba a mano el `resolv.conf`, copiaba el `krb5.conf` generado y activaba los servicios uno por uno con `systemctl`. Era un proceso "artesanal": cada paso era responsabilidad del script y un solo error a mitad de camino podía dejar el dominio a medias — incluso había que recordar apagar `systemd-resolved` para liberar el DNS.
>
> En Windows Server, **no hace falta ningún script externo ni repositorio que clonar**. `Install-ADDSForest` es un único cmdlet nativo del sistema operativo que orquesta *todo* el proceso: instala la base de datos NTDS, configura Kerberos, activa el DNS integrado, establece el nivel funcional del dominio y reinicia el servidor cuando corresponde — todo con validaciones automáticas en cada paso (`Test-ADDSForestInstallation` se ejecuta de fondo antes de aplicar ningún cambio). Es un contraste interesante entre dos filosofías de administración:
> *   **Samba AD DC (artesanal):** una **reimplementación de código abierto** de Active Directory que requiere ensamblar manualmente piezas independientes (LDAP, Kerberos, DNS, Winbind) mediante un script.
> *   **AD DS (asistido):** el **producto original de Microsoft**, con un asistente guiado (gráfico o por PowerShell) que hace ese ensamblaje por ti en un solo cmdlet.
>
> Ninguno es "mejor" en abstracto — Samba es gratuito y funciona sobre Linux; AD DS requiere licencia de Windows Server pero ofrece una experiencia de instalación mucho más asistida y menos propensa a errores humanos.

### 📖 Diccionario de Conceptos Clave

> [!quote] Terminología de Dominio
> - **Reino (Realm):** El nombre de dominio completo (ej. `BOOCHAN.SPACE`). Siempre se escribe en **MAYÚSCULAS** para que Kerberos lo entienda.
> - **NetBIOS Domain:** El nombre corto del dominio (ej. `BOOCHAN`), usado por protocolos Windows heredados y como prefijo de inicio de sesión (`BOOCHAN\usuario`).
> - **Nivel Funcional (Functional Level):** El conjunto de características avanzadas de Active Directory disponibles en un dominio o bosque, determinado por la versión mínima de Windows Server que puede actuar como Controlador de Dominio. En BoochanV3.1 usamos el nivel más alto disponible, porque no hay Controladores de Dominio antiguos con los que mantener compatibilidad.
> - **DSRM (Directory Services Restore Mode):** Modo de arranque especial de un Controlador de Dominio para tareas de recuperación de emergencia. Requiere una contraseña propia, distinta de la del Administrador del dominio.
> - **SRV Record:** Un registro DNS especial que indica qué servidor ofrece un servicio específico (ej. "el servidor de tickets está en esta IP").
> - **Provisionamiento:** El acto de generar la base de datos del dominio desde cero.

---

### 🔓 Apertura de Puertos (Security Group de AWS)

> [!example] 🎬 Antes de empezar (todavía SIN grabar, y luego arranca)
> Ya conoces el método desde los prerrequisitos, así que va solo el recordatorio:
> 1. **Crea la entrada de apuntes** de esta fase (`v3-1-fase-4-aprovisionamiento-del-dominio-ad-ds-nati.md`) con su estructura, vacía.
> 2. **Léete los 4 pasos** del procedimiento enteros, para no atascarte a mitad del vídeo.
> 3. Ten **OBS** listo y comprueba **pantalla y micrófono**.
>
> Cuando lo tengas: **arranca la grabación, preséntate y muestra tu identidad**. A partir de ahí, **todo queda grabado** — incluido cualquier paso previo de preparación que venga a continuación.

> [!example] Al empezar: abre los puertos del dominio
> Active Directory es un ecosistema de servicios que se hablan entre sí. Antes de provisionar el dominio, todos sus puertos deben estar abiertos en AWS — si falta uno, los clientes Windows no podrán autenticarse ni resolver nombres.
>
> 1. Entra en **console.aws.amazon.com** → busca **`EC2`** → en el menú izquierdo, dentro de **Red y seguridad**, haz clic en **`Grupos de seguridad`**.
> 2. Marca tu grupo `sg-boochan-[tunombre]` y abajo pulsa la pestaña **`Reglas de entrada`** → **`Editar reglas de entrada`**.
> 3. Para **cada fila** de la tabla siguiente, pulsa **`Agregar regla`** y rellena los campos:
>    - **Tipo:** `TCP personalizado` o `UDP personalizado` según la columna "Protocolo"
>    - **Intervalo de puertos:** el número de la columna "Puerto"
>    - **Origen:** `Anywhere-IPv4` (equivale a `0.0.0.0/0`)
>    - **Descripción:** el texto de la columna "Nombre"
> 4. Cuando hayas añadido las 13 reglas, pulsa **`Guardar reglas`** (se guardan todas de golpe, no una a una):
>
> | Nombre | Puerto | Protocolo | Para qué sirve ahora |
> | :--- | :--- | :--- | :--- |
> | Kerberos_TCP | 88 | TCP | Emite los "tickets" de seguridad que identifican a cada usuario del dominio. |
> | Kerberos_UDP | 88 | UDP | Ídem por UDP — Windows usa ambos según el tipo de petición. |
> | DNS_TCP | 53 | TCP | Resuelve los nombres del dominio (ej. `BOOCHAN.SPACE`). |
> | DNS_UDP | 53 | UDP | Ídem por UDP — la mayoría de consultas DNS viajan por UDP. |
> | RPC_Endpoint | 135 | TCP | Punto de entrada para las llamadas a procedimiento remoto de Windows. |
> | LDAP_TCP | 389 | TCP | Permite consultar el directorio de usuarios y grupos del dominio. |
> | LDAP_UDP | 389 | UDP | Ídem por UDP. |
> | LDAPS | 636 | TCP | Versión cifrada de LDAP — protege las consultas de usuarios en tránsito. |
> | SMB_Files | 445 | TCP | Acceso a las carpetas compartidas del servidor (SYSVOL, NETLOGON). |
> | RPC_Dinamico | 49152-65535 | TCP | Rango de puertos que Active Directory negocia dinámicamente para comunicarse. |
> | Kerberos_Pass_TCP | 464 | TCP | Gestión de cambios de contraseña de los usuarios del dominio. |
> | Kerberos_Pass_UDP | 464 | UDP | Ídem por UDP. |
> | NTP_Time | 123 | UDP | Sincronización horaria del servidor — Kerberos falla si el reloj difiere más de 5 minutos. |
>
> > [!note] 💡 Recuerda: en AWS no hay columna "Prioridad"
> > Igual que en la Fase 3, los Security Groups de AWS solo permiten tráfico (no deniegan) y no tienen orden de prioridad. Por eso la tabla no lleva números de prioridad como tendría un NSG de Azure.
>
> > [!info] 💡 ¿Por qué tantos puertos de golpe?
> > En las fases anteriores abriste solo lo imprescindible para no exponer el servidor innecesariamente. Active Directory es diferente: es un ecosistema de servicios interdependientes. DNS encuentra el servidor, Kerberos autentica al usuario, LDAP consulta su perfil y RPC coordina todo el proceso. Si falta uno, la cadena se rompe. Esta es la única fase del proyecto donde abrirás tantos puertos a la vez. A partir de la Fase 5 no necesitarás añadir ninguno más.
>
> > [!tip] 💡 El Firewall de Windows también se reconfigura, pero automáticamente
> > A diferencia del Security Group de AWS (que ya abriste tú a mano en esta fase), el **Firewall de Windows Defender** local del propio servidor se reconfigura solo durante `Install-ADDSForest`: el asistente crea las reglas necesarias para todo el tráfico de AD DS sin que tengas que tocarlas.

---

### 🛠️ Procedimiento Práctico (BoochanV3.1)

> [!example] Paso 1: Instalación del Rol AD DS
> A diferencia de BoochanV3, aquí no descargamos nada de un repositorio externo — el rol AD DS forma parte del propio Windows Server, solo hay que activarlo:
>
> > [!info] 📚 Diccionario de Comandos: Consulta el [[Diccionario_Comandos_Sistema]] para entender al detalle cómo funcionan los cmdlets administrativos que usaremos aquí.
>
> ```powershell
> # Instala el rol AD DS junto con las herramientas de gestión (RSAT: Get-ADUser, dsa.msc, etc.)
> Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
> ```
> > [!tip] 💡 ¿Qué hace este comando?
> > - **`Install-WindowsFeature`:** El equivalente Windows a `apt install`. Activa un componente del propio sistema operativo (no descarga un paquete externo de internet, salvo que falten archivos de origen).
> > - **`-IncludeManagementTools`:** Instala también las herramientas gráficas y de PowerShell para administrar el dominio después (`Get-ADUser`, `Get-ADGroup`, la consola "Usuarios y equipos de Active Directory", etc.). Sin este parámetro, el rol se instala pero no tendrás herramientas cómodas para gestionarlo.
> >
> > Este paso solo **instala los binarios** del rol. Todavía no existe ningún dominio — eso ocurre en el Paso 2.

> [!example] Paso 2: Promoción a Controlador de Dominio con `Install-ADDSForest`
> Este es el equivalente Windows al script `provision_boochan.sh` de BoochanV3, pero como un único cmdlet nativo del sistema, sin script externo que descargar ni ejecutar:
> ```powershell
> Install-ADDSForest `
>   -DomainName "BOOCHAN.SPACE" `
>   -DomainNetbiosName "BOOCHAN" `
>   -DomainMode "Win2025" `
>   -ForestMode "Win2025" `
>   -InstallDNS `
>   -SafeModeAdministratorPassword (ConvertTo-SecureString "P@ssword2026!" -AsPlainText -Force) `
>   -Force
> ```
>
> > [!tip] 💡 ¿Qué hace cada parámetro?
> > - **`-DomainName "BOOCHAN.SPACE"`:** El Realm completo del dominio, equivalente al `REALM_NAME` de `provision_boochan.sh` en BoochanV3.
> > - **`-DomainNetbiosName "BOOCHAN"`:** El nombre corto, equivalente al `DOMAIN_NAME` del script de BoochanV3.
> > - **`-DomainMode` / `-ForestMode "Win2025"`:** El **nivel funcional** del dominio y del bosque. Usamos el nivel más alto disponible porque este es un dominio nuevo, de un único Controlador de Dominio, sin necesidad de mantener compatibilidad con versiones antiguas de Windows Server.
> > - **`-InstallDNS`:** Instala y configura automáticamente el rol **DNS Server** integrado con Active Directory. Es el equivalente al `--dns-backend=SAMBA_INTERNAL` del script de Samba — pero aquí no hace falta indicarlo como opción de bajo nivel, es un simple interruptor.
> > - **`-SafeModeAdministratorPassword`:** La contraseña del modo de recuperación DSRM (ver Diccionario de Conceptos). Es una contraseña **distinta** de la del Administrador del dominio, pensada solo para emergencias de recuperación del directorio.
> > - **`-Force`:** Evita que el asistente pida confirmación interactiva en cada paso, útil para practicar el comando de forma reproducible.
>
> > [!caution] ⚠️ El servidor se reiniciará automáticamente
> > Al finalizar, `Install-ADDSForest` reinicia el servidor sin pedir confirmación adicional (salvo que uses `-NoRebootOnCompletion`). Es normal y esperado — la promoción a Controlador de Dominio requiere un reinicio para que todos los servicios (NTDS, DNS, Kerberos) arranquen con la nueva configuración. **Perderás la sesión RDP** durante el proceso, que suele tardar **entre 5 y 10 minutos** en total, incluyendo el reinicio.
> >
> > **Si el comando falla antes de llegar al reinicio:** revisa el mensaje de error en pantalla — a diferencia del script bash de BoochanV3 (donde había que interpretar mensajes de Samba en la terminal), aquí PowerShell describe en texto claro qué requisito previo no se cumplió, por ejemplo nombre de equipo incorrecto o falta de RAM. Revisa la tabla de troubleshooting al final de esta fase.
>
> > [!note] 📄 Comparación de enfoque: el script de V3 frente al cmdlet nativo de V3.1
> > A diferencia de BoochanV3, aquí no hay ningún fichero `provision_boochan.sh` que clonar desde un repositorio Git ni valores por defecto ocultos en variables de bash. Todo el "guion" de la instalación está contenido en los parámetros explícitos del propio cmdlet que acabas de ejecutar — no hay nada oculto ni que depender de una URL externa proporcionada por el profesor. Esto elimina de raíz un tipo de error muy habitual en BoochanV3 (clonar el repositorio equivocado, o con el Realm de otra versión del proyecto).

> [!example] Paso 3: Verificación de Servicios
> Tras el reinicio automático, vuelve a conectarte al servidor por RDP (ahora ya como miembro del dominio, usando la IP de la VPN: `mstsc /v:10.0.0.1`) y comprueba que el "corazón" del dominio está latiendo:
> ```powershell
> # Comprobar que los servicios de Active Directory están activos
> Get-Service -Name NTDS, DNS, Kdc | Select-Object Name, Status
> ```
> Los tres servicios (`NTDS` — base de datos del directorio, `DNS` — DNS integrado, `Kdc` — Centro de distribución de claves Kerberos) deben mostrar el estado `Running`.

> [!example] Paso 4: Verificación del DNS
> Es vital confirmar que el servidor se mira a sí mismo para resolver nombres de red:
> ```powershell
> # Debe devolver 127.0.0.1 o la propia IP privada de AWS (172.31.x.x); ambas son válidas tras la promoción
> Get-DnsClientServerAddress -InterfaceAlias "*Ethernet*" -AddressFamily IPv4
> ```
> > [!tip] 💡 ¿Por qué a veces aparece la IP `172.31.x.x` en lugar de `127.0.0.1`?
> > Durante `Install-ADDSForest -InstallDNS`, Windows a veces reconfigura automáticamente el DNS del adaptador para que apunte a su propia IP privada en lugar de al loopback (`127.0.0.1`). Ambos valores son funcionalmente equivalentes en este caso — los dos apuntan al propio servidor. Si prefieres mantener exactamente `127.0.0.1` (por coherencia con la configuración de la Fase 2), puedes reaplicarlo manualmente con `Set-DnsClientServerAddress`.

---

### 🚩 Resolución de Problemas y Evaluación

> [!bug] Troubleshooting (¿El dominio no nace?)
> | Problema | Causa Probable | Solución Sugerida |
> | :--- | :--- | :--- |
> | Error "El nombre de equipo del equipo local no coincide con..." o similar. | El servidor no se renombró correctamente en la Fase 2, o el reinicio del renombrado no se completó antes de instalar AD DS. | Ejecuta `$env:COMPUTERNAME` y confirma que devuelve `WindowsServer`. Si no, repite el Paso 1 de la Fase 2. |
> | `Install-ADDSForest` se detiene con un error de prerrequisitos (`Test-ADDSForestInstallation`). | Falta de RAM, o el nombre NetBIOS ya existe en la red (poco probable en un entorno de alumno aislado). | Revisa el mensaje de error completo — PowerShell describe exactamente qué prerrequisito falló. |
> | El servidor no responde tras el reinicio automático. | El reinicio tarda más de lo habitual mientras se inicializan los servicios de AD DS por primera vez. | Espera 2-3 minutos adicionales antes de intentar reconectar por RDP. Es normal en el primer arranque tras la promoción. |
> | `Resolve-DnsName` no encuentra el registro SRV de Kerberos. | El servicio DNS no terminó de configurarse, o el adaptador de red no apunta al DNS correcto. | Ejecuta `Get-Service DNS` y confirma que está `Running`. Revisa `Get-DnsClientServerAddress` como en el Paso 4. |
> | No puedo reconectar por RDP tras el reinicio usando la IP pública/Elástica. | Es el comportamiento esperado de la Fase 3 — el RDP directo por internet quedó restringido a tu IP concreta. | Reconecta usando la IP del túnel: `mstsc /v:10.0.0.1`. Si el túnel tampoco responde, verifica `wg show` y que el servicio `WireGuardTunnel$wg0` siga `Running` tras el reinicio. |

> [!help] Preguntas Críticas (Autoevaluación)
> 1. ¿Por qué es fundamental que el servidor DNS del dominio sea el propio servidor?
> 2. ¿Qué es un "ticket" de Kerberos y por qué evita enviar contraseñas por la red constantemente?
> 3. Compara el proceso de aprovisionamiento de BoochanV3 (script `provision_boochan.sh`) con el de BoochanV3.1 (`Install-ADDSForest`). ¿Qué ventajas y qué desventajas tiene cada enfoque?
> 4. ¿Cuál es la diferencia entre el Realm (`BOOCHAN.SPACE`) y el nombre NetBIOS (`BOOCHAN`) del dominio? ¿Cuándo se usa cada uno?
> 5. 🔬 **Reto práctico:** Ejecuta `Resolve-DnsName _kerberos._tcp.BOOCHAN.SPACE` en el servidor. Si el dominio está bien provisionado, ¿qué debería devolver? Si falla, ¿qué componente del sistema está fallando?
> 6. 🔬 **Reto práctico:** Ejecuta `Get-ADUser -Filter *` en el servidor. ¿Qué usuarios ves, siendo que tú no has creado ninguno todavía? Localiza el usuario que empieza por `krbtgt` — busca en internet para qué sirve ese usuario en Kerberos y explícalo con tus palabras. Compara además la RAM libre actual (`Get-Counter '\Memory\Available MBytes'`) con la que anotaste al final de la Fase 2.

---

> [!caution] 🛑 Auditoría y Evaluación (RA.03)
> **Peligro Crítico:** Si el DNS vuelve a apuntar a otro sitio en lugar de al propio servidor, los ordenadores dirán "No se encuentra el dominio" y nadie podrá iniciar sesión.

> [!success] 🏁 Punto de Control (Antes de seguir)
> - [ ] ¿`Get-Service NTDS, DNS, Kdc` muestra los tres servicios como `Running`?
> - [ ] ¿`Get-ADDomain` devuelve la información del dominio `BOOCHAN.SPACE` sin errores?
> - [ ] ¿`Resolve-DnsName _kerberos._tcp.BOOCHAN.SPACE` devuelve el registro SRV correcto?

---

### ✅ Entregables y cierre

> [!abstract] Qué tienes que tener hecho al acabar esta fase
> | Entregable | Dónde vive | Qué debe contener |
> | :--- | :--- | :--- |
> | **Entrada de apuntes** | `00_Apuntes/Trimestre_N/B5_Windows_Nube/v3-1-fase-4-aprovisionamiento-del-dominio-ad-ds-nati.md` | Estructura completa + **respuestas a las Preguntas Críticas y al 🔬 Reto** + **enlace del vídeo** |
> | **Vídeo** | Playlist `B5_Windows_Nube` (No listado) | Nombrado `V3.1 · Fase 4 — Aprovisionamiento del Dominio (AD DS nativo)`, con presentación, identidad y timestamps |
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
