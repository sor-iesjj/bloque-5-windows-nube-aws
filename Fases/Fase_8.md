## 💻 Fase 8: Integración del Cliente (Windows 11)

### Infraestructura de Servidores Cloud

> **[Módulo: SOR — Sistemas Operativos en Red]**
> **[U.T. 10: Windows Server como servidor de dominio / Windows 11 como cliente de dominio]**
> **[RA.06]** Realiza tareas de integración de sistemas operativos libres y propietarios.
>
> **Profesor:** Pedro Navarro Miralles  
> **Correo:** p.navarromiralles2@edu.gva.es  
> **Centro:** IES Jorge Juan (ALICANTE)
>
> **⏱️ Tiempo estimado:** ~2 horas (teoría + práctica + retos + troubleshooting)  
> **Requisitos:** Windows 11 (PC físico del aula) | 8-12 GB RAM | VPN activa | `WindowsServer` (t3.large) con AD DS completo (Fases 1-7)

---

> [!important] 📹 Obligaciones de grabación (LÉEME — es igual en TODAS las fases)
> Esta práctica se **graba entera con OBS**, de principio a fin. No es un repaso al final: quiero ver **cómo lo haces tú**.
> 1. **Prepárate primero (sin grabar):** comprueba lo necesario, **léete el procedimiento entero** y **crea la entrada de apuntes de esta fase** en Obsidian: fichero `v3-1-fase-8-integracion-del-cliente-windows-11.md` dentro de `00_Apuntes/Trimestre_N/B5_Windows_Nube/`, con la estructura de la Fase 0.1 y **vacía**. Rellenarla es cosa tuya, después.
> 2. **Arranca OBS y PRESÉNTATE:** *"Hola, me llamo [Nombre], 2.º SMR, y en este vídeo voy a explicar la Fase 8 de Boochan V3.1 — Integración del Cliente (Windows 11)."* Y **muestra algo que demuestre que eres tú** (tu perfil de GitHub, tu Teams o tu correo `@alu.edu.gva.es`). Di qué vas a hacer.
> 3. **Graba TODO el procedimiento**, explicando cada paso en voz alta mientras lo haces.
> 4. **Timestamps SIEMPRE** en la descripción: `00:00 Presentación` + uno por cada paso.
> 5. **Al terminar:** nombra el vídeo `V3.1 · Fase 8 — Integración del Cliente (Windows 11)`, súbelo a tu playlist de YouTube **`B5_Windows_Nube`** (No listado) y **copia su enlace**.
> 6. **~8-10 min.** Esta fase es más larga que las de prerrequisitos: ve al grano, pero no te saltes pasos. Si se te va mucho, **pártela en dos vídeos** y ponlos los dos en la entrada.
> 7. **El enlace del vídeo va DENTRO de tu entrada de apuntes**, en el apartado `Enlace al vídeo explicativo`. Ahí, no en un papel.
> 8. **La entrega va por la TAREA de Teams.** Abriré una tarea que cubrirá **esta fase y otras**; te llegará notificación con fecha límite.

---

### 🎯 ¿Dónde Estamos?

> [!info] Vienes de Fase 7
> El servidor `WindowsServer` es ahora un "reino" completo: dominio `BOOCHAN.SPACE`, usuarios, grupos, discos protegidos, y permisos NTFS granulares. Todo está funcionando perfectamente desde PowerShell y desde el Administrador del servidor. Sin embargo, los usuarios del aula están esperando en sus PCs Windows 11 — ahora necesitas que esos equipos confíen en el servidor y usen sus identidades de dominio.

> [!warning] El Problema
> Aunque cliente y servidor "hablan el mismo idioma" (ambos son Windows), antes de que se confíen deben cumplirse varias condiciones: (1) el cliente debe encontrar el servidor por DNS, (2) sincronizar el reloj exactamente (Kerberos rechaza diferencias > 5 minutos), (3) establecer una "relación de confianza" registrándose en Active Directory, (4) permitir que los usuarios inicien sesión con sus credenciales de dominio. Si algo falla, el usuario ve "No se puede encontrar el dominio" o "Error de relación de confianza".

> [!success] Objetivo de esta Fase
> **Unir Windows 11 al dominio BOOCHAN.SPACE** de forma que los usuarios puedan iniciar sesión con sus credenciales de dominio (ej. `BOOCHAN\user1`) y acceder a las carpetas compartidas del servidor con los permisos que se les asignaron en fases anteriores. Es el momento de la verdad: la infraestructura híbrida en la nube (servidor Windows Server 2025 en AWS + cliente Windows 11 físico del aula) funcionando en sinergia.

> [!tip] Hoja de Ruta
> 1. **Validar VPN:** Activar el túnel WireGuard en el PC del aula para acceder a la red privada 10.0.0.0/24
> 2. **Configurar DNS de Windows:** Cambiar DNS primario a 10.0.0.1 (el servidor), DNS secundario a 8.8.8.8 (fallback a internet)
> 3. **Sincronizar reloj:** Ejecutar `w32tm /resync /force` para emparejar la hora exactamente con el servidor
> 4. **Unir al dominio:** Con `Add-Computer -DomainName "BOOCHAN.SPACE" -Restart` desde PowerShell, o vía Configuración → Cuentas → Acceso profesional o educativo → Conectar
> 5. **Reiniciar Windows:** Obligatorio para aplicar los cambios de dominio
> 6. **Primer login:** Iniciar sesión con `BOOCHAN\user1` y su contraseña desde la pantalla de inicio
> 7. **Instalar RSAT:** Herramientas administrativas para gestionar usuarios/grupos desde Windows gráficamente
> 8. **Mapear carpetas de red:** Conectar `\\WindowsServer.BOOCHAN.SPACE\prueba1` y `prueba3` como unidades de red (Z:, por ejemplo)
>
> **Resultado Final:** Windows 11 es ahora un cliente legítimo del dominio. Los usuarios pueden iniciar sesión, acceder a carpetas según sus permisos de grupo, y crear archivos que el servidor reconoce automáticamente.
> **Siguiente:** Fase completada — el proyecto es funcional de extremo a extremo. Servidor Windows Server como DC en la nube (AWS), usuarios en AD, almacenamiento seguro, y clientes Windows integrados a través del túnel VPN. Solo queda la Auditoría Final de seguridad.

---

### 📚 Fundamento Teórico

> [!abstract] 1. El Momento de la Verdad
> Unir un Windows 11 al dominio significa que el PC transfiere la autoridad de seguridad al servidor `WindowsServer`. A partir de ahora, el servidor decidirá quién entra y qué puede hacer.

> [!important] 2. Tu PC físico, no una VM anidada
> A diferencia de un laboratorio local con Hyper-V (donde el cliente Windows 11 sería otra máquina virtual dentro del mismo host que el servidor), aquí **el cliente Windows 11 es tu propio PC físico del aula**. El servidor `WindowsServer` vive en AWS, a cientos de kilómetros; tu PC vive en el aula. Para que ambos puedan hablarse como si estuvieran en la misma red local, necesitas el **túnel WireGuard** que configuraste en la Fase 3: es el "cable virtual" que hace que tu PC parezca estar conectado directamente a `10.0.0.0/24`, la red privada del servidor. Sin la VPN activa, tu PC está simplemente en Internet, sin ninguna vía para llegar al Controlador de Dominio.

> [!warning] 3. Sincronización Horaria (NTP)
> Kerberos (el sistema de tickets) utiliza marcas de tiempo para evitar ataques. Si el reloj del PC y el del Servidor varían más de **5 minutos (Clock Skew)**, la comunicación se cortará por seguridad y no podrás iniciar sesión.

> [!important] 4. DNS: El Guía de la Red
> Windows debe usar el DNS de nuestra VPN (10.0.0.1) para poder encontrar al "Rey" (el Controlador de Dominio, que en Windows Server además es el propio servidor DNS del dominio). Si usa el DNS del router de casa o del aula, jamás encontrará el servidor `BOOCHAN.SPACE`.

### 📖 Diccionario de Conceptos Clave

> [!quote] Integración de Clientes
> - **Unirse al Dominio:** Proceso de registrar un ordenador cliente en la base de datos central del Directorio Activo (creando su cuenta de equipo).
> - **Clock Skew:** El desfase de tiempo máximo permitido por seguridad (300 segundos = 5 minutos).
> - **RSAT:** Herramientas de administración remota para gestionar el dominio Windows Server desde la interfaz gráfica de Windows 11.
> - **Add-Computer:** Cmdlet de PowerShell que une un equipo a un dominio Active Directory.
> - **net use:** Comando de consola para conectar carpetas compartidas como si fueran discos locales.

### 🔓 Apertura de Puertos (Security Group de AWS)

> [!info] ℹ️ Sin cambios en el Security Group en esta fase
> El cliente Windows se conecta al servidor a través del **túnel VPN (WireGuard)**, no directamente por internet. Todo el tráfico de dominio (Kerberos, LDAP, SMB…) viaja cifrado por dentro del túnel, cuyo puerto (51820 UDP) ya está abierto desde la Fase 3. No tienes que tocar nada en el Security Group de AWS.
>
> **Lo único que debes hacer antes de empezar:** activar el túnel WireGuard en tu PC del aula.

---

### 🛠️ Procedimiento Práctico

> [!important] 🔌 Antes de empezar: Activa la VPN
> Para que Windows pueda encontrar el dominio, **el túnel WireGuard debe estar activo**. Abre la aplicación WireGuard en tu PC y haz clic en **"Activar"** antes de continuar con cualquier paso de esta fase.

> [!example] Paso 1: Configuración del DNS en Windows
> Windows debe preguntar a nuestro servidor (10.0.0.1) para encontrar el dominio. Sigue estos pasos para cambiar el DNS manualmente:
>
> 1. Haz clic en el icono de **Red** de la barra de tareas → **"Configuración de red e Internet"**.
> 2. Haz clic en **"Ethernet"** (o "Wi-Fi" si usas inalámbrico) → **"Editar"** junto a "Asignación de servidor DNS".
> 3. En el desplegable, cambia "Automático (DHCP)" a **"Manual"**.
> 4. Activa el interruptor de **IPv4** e introduce:
>    - **DNS preferido:** `10.0.0.1`
>    - **DNS alternativo:** `8.8.8.8`
> 5. Pulsa **"Guardar"**.
>
> > [!tip] 💡 ¿Por qué ponemos también `8.8.8.8`?
> > Con solo `10.0.0.1`, Windows pierde acceso a internet si el servidor no reenvía consultas al exterior. El `8.8.8.8` (DNS público de Google) actúa de red de seguridad para que Windows pueda seguir navegando y descargando software como RSAT en el Paso 5.
>
> > [!tip] 💡 Verifica que el DNS funciona
> > Abre PowerShell y ejecuta:
> > ```powershell
> > nslookup BOOCHAN.SPACE
> > ```
> > Si devuelve la IP `10.0.0.1`, el DNS está funcionando. Si dice "no se encuentra el servidor", la VPN no está activa o el DNS es incorrecto.

> [!example] Paso 2: Sincronización de Tiempo
> Ejecuta este comando en **PowerShell como Administrador**:
>
> *(Para abrirlo como Administrador: pulsa `Windows + X` → "Terminal de Windows (Administrador)" o busca "PowerShell" en el menú inicio, clic derecho → "Ejecutar como administrador")*
>
> > [!info] 📚 Diccionario de Comandos: Recuerda que también tienes explicados los comandos vitales de Windows (`w32tm`, `nslookup`, `Add-Computer`) en el [[Diccionario_Comandos_Sistema]].
>
> ```powershell
> w32tm /resync /force
> ```
>
> > [!tip] 💡 ¿Qué hace este comando?
> > - **`w32tm`:** Es la herramienta de gestión del tiempo de Windows. El parámetro `/resync /force` obliga al PC a emparejar su reloj con el del Controlador de Dominio inmediatamente, ignorando cualquier restricción.

> [!example] Paso 3: Unión al Dominio
> Puedes hacerlo de dos formas equivalentes. Se recomienda la de PowerShell, más rápida y menos propensa a errores de clic:
>
> **Opción A — PowerShell (recomendada):**
> ```powershell
> Add-Computer -DomainName "BOOCHAN.SPACE" -Credential BOOCHAN\Administrator -Restart
> ```
> Te pedirá la contraseña del `Administrator` del dominio (`P@ssword2026!`) en una ventana emergente. Al confirmarla, el PC se une al dominio y se reinicia automáticamente.
>
> **Opción B — Interfaz gráfica:**
> 1. Abre **Configuración** (tecla `Windows + I`).
> 2. Ve a **Sistema** → **Acerca de** → **"Cambiar nombre de este PC (avanzado)"** → pestaña **"Nombre de equipo"** → botón **"Cambiar..."**.
> 3. Selecciona **"Dominio"** e introduce: `BOOCHAN.SPACE`
> 4. Pulsa **Aceptar**. Te pedirá credenciales: introduce `Administrator` y `P@ssword2026!`.
> 5. Si aparece el mensaje **"Bienvenido al dominio BOOCHAN"**, el proceso ha sido correcto.
> 6. **Reinicia el equipo** cuando te lo pida. Este paso es obligatorio.
>
> > [!important] 💡 El reinicio es obligatorio
> > Sin reiniciar, Windows no aplica los cambios del dominio. Al volver a encender el PC, en la pantalla de inicio de sesión verás la opción de iniciar sesión con un usuario del dominio.

> [!example] Paso 4: Primer Inicio de Sesión con Usuario del Dominio
> > [!caution] ⚠️ La VPN debe estar activa antes de intentar el login
> > Al iniciar sesión con `BOOCHAN\user1`, Windows necesita contactar con el servidor en `10.0.0.1` para validar las credenciales. Si la VPN no está activa, el login fallará con "No se puede contactar con el dominio". Abre la aplicación WireGuard y activa el túnel **antes** de introducir el usuario y la contraseña.
>
> En la pantalla de inicio de sesión de Windows, introduce las credenciales del usuario del dominio. Fíjate en el formato correcto:
>
> - **Usuario:** `BOOCHAN\user1`  *(el nombre NetBIOS del dominio, una barra invertida `\`, y el nombre de usuario)*
> - **Contraseña:** `P@ssword2026!`
>
> > [!warning] ⚠️ La barra invertida `\`, no la barra normal `/`
> > La barra invertida se escribe con la tecla que tiene el símbolo `\` en tu teclado (normalmente junto al `Intro` o junto al `0`). Si usas la barra normal `/`, no funcionará.

> [!example] Paso 5: Instalación de RSAT (Herramientas de Administración)
> RSAT permite gestionar usuarios y grupos del dominio directamente desde Windows, con una interfaz gráfica. Requiere que la VPN esté activa (o simplemente salida a Internet, ya que el PC físico del aula ya tiene su propia conexión). Instálalo así:
>
> **Opción A — Características opcionales (gráfico):**
> 1. Ve a **Configuración** → **Aplicaciones** → **Características opcionales**.
> 2. Haz clic en **"Ver características"**.
> 3. Busca `RSAT` en el cuadro de búsqueda.
> 4. Instala **"RSAT: Herramientas de Servicios de dominio de Active Directory y Lightweight Directory"**.
> 5. Pulsa **"Instalar"** y espera a que termine.
>
> **Opción B — PowerShell (como Administrador):**
> ```powershell
> Get-WindowsCapability -Name RSAT.ActiveDirectory* -Online | Add-WindowsCapability -Online
> ```
>
> Una vez instalado, encontrarás las herramientas buscando **"Usuarios y equipos de Active Directory"** en el menú Inicio.

> [!example] Paso 6: Mapeo de Carpetas de Red
> Con el usuario del dominio iniciado, conecta las carpetas del servidor como si fueran discos locales. Abre **PowerShell** o el **Símbolo del sistema** y ejecuta:
> ```cmd
> net use Z: \\WindowsServer.BOOCHAN.SPACE\prueba1 /user:BOOCHAN\user1 /persistent:yes
> ```
> `/persistent:yes` hace que la unidad `Z:` se vuelva a conectar sola en cada inicio de sesión — eso sí, requiere que el túnel WireGuard esté activo cuando Windows intente reconectarla (ver Error 8.4 en la tabla de troubleshooting).
>
> > [!tip] 💡 ¿Qué hace este comando?
> > - **`Z:`**: Asigna una letra de unidad libre (como un disco duro más).
> > - **`\\WindowsServer.BOOCHAN.SPACE\prueba1`**: Es la ruta UNC (la dirección de la carpeta en la red). Usamos el nombre del servidor en lugar de la IP para que Windows use Kerberos (el sistema de tickets seguro) en lugar de un protocolo más antiguo y menos fiable (NTLM). Esa carpeta compartida apunta internamente a `C:\ShareData\Prueba1` en el servidor.
> > - **`/user:BOOCHAN\user1`**: Especifica con qué identidad del dominio queremos entrar.

---

### 🚩 Resolución de Problemas y Evaluación

> [!bug] Troubleshooting (¿No puedes unirte?)
> | Problema | Causa Probable | Solución Sugerida |
> | :--- | :--- | :--- |
> | "No se encuentra el dominio". | El cliente está usando el DNS del router, no el nuestro, o la VPN no está activa. | Comprueba que el DNS primario es `10.0.0.1` y que la VPN está activa. |
> | "Error de relación de confianza". | Desfase horario (Clock Skew) superior a 5 minutos. | Ejecuta `w32tm /resync /force` (Paso 2) antes de reintentar. |
> | `Add-Computer` falla con "No se puede contactar con el dominio". | Igual que el primer caso: fallo de conectividad o de DNS hacia `10.0.0.1`. | Verifica primero con `nslookup BOOCHAN.SPACE` antes de reintentar la unión. |
> | La unidad `Z:` no aparece al reiniciar. | El mapeo no es persistente. | Añade `/persistent:yes` al final del comando `net use`. Recuerda que la VPN debe estar activa antes de que Windows intente reconectar la unidad. |
> | RSAT no se descarga / se queda "buscando actualizaciones". | El PC del aula no tiene salida a Internet en ese momento. | Comprueba la conexión a Internet del propio PC, independiente de la VPN. |

> [!help] Preguntas Críticas (Autoevaluación)
> 1. ¿Por qué Windows necesita consultar específicamente el DNS del servidor para unirse al dominio?
> 2. ¿Qué sucede técnica y exactamente si hay más de 5 minutos de diferencia horaria?
> 3. ¿Para qué sirven las herramientas **RSAT** en esta infraestructura híbrida?
> 4. 🔬 **Reto práctico:** Con `user1` iniciado en Windows, crea un archivo de texto en la unidad `Z:` (por ejemplo `prueba_user1.txt`). Sin cerrar Windows, conéctate al servidor por RDP o PowerShell remoto a través del túnel WireGuard y comprueba el archivo en la ruta compartida `C:\ShareData\Prueba1`. ¿Ves el archivo? ¿Qué usuario y permisos NTFS aparecen como propietario? ¿Coincide con lo que configuraste en la Fase 5?
> 5. 🔬 **Reto práctico:** Con `user1` logueado, **desactiva el túnel WireGuard** desde la aplicación sin cerrar sesión de Windows. Intenta abrir un archivo de la unidad `Z:`. ¿Qué error aparece? ¿Qué le dirías a un usuario de empresa que llama al soporte diciendo que "la carpeta compartida ha desaparecido"?

---

> [!caution] 🛑 Auditoría de Integración (RA.06)
> **Validación:** El alumno debe loguearse con `user1` en el Windows 11 del aula y demostrar que puede crear un archivo en la unidad `Z:` que luego sea visible desde el propio servidor `WindowsServer` en la carpeta compartida `C:\ShareData\Prueba1`. Además, debe demostrar que `user2` (bomberos) no ve la carpeta `prueba3` en el explorador de archivos.

> [!success] 🏁 Punto de Control (Antes de seguir)
> - [ ] ¿Has podido unirte al dominio sin errores de DNS?
> - [ ] ¿La unidad de red `Z:` aparece en el explorador de archivos?
> - [ ] ¿`user1` puede crear archivos en `Z:` y se ven desde el servidor?
> - [ ] ¿`user2` no ve la carpeta `prueba3` al navegar por la red?

---

### ✅ Entregables y cierre

> [!abstract] Qué tienes que tener hecho al acabar esta fase
> | Entregable | Dónde vive | Qué debe contener |
> | :--- | :--- | :--- |
> | **Entrada de apuntes** | `00_Apuntes/Trimestre_N/B5_Windows_Nube/v3-1-fase-8-integracion-del-cliente-windows-11.md` | Estructura completa + **respuestas a las Preguntas Críticas y al 🔬 Reto** + **enlace del vídeo** |
> | **Vídeo** | Playlist `B5_Windows_Nube` (No listado) | Nombrado `V3.1 · Fase 8 — Integración del Cliente (Windows 11)`, con presentación, identidad y timestamps |
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
