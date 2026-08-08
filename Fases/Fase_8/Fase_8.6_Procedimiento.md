## Fase 8 · Apartado 6 — 🛠️ Procedimiento práctico

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Integración del Cliente (Windows 11)**
> 🧭 Índice de la fase: [[Fase_8]]
>
> **📍 Cuándo se lee:** **Con la VM delante.** Aquí está el trabajo.

---

> [!important] 🔌 Antes de empezar: Activa la VPN
> Para que Windows pueda encontrar el dominio, **el túnel WireGuard debe estar activo**. Abre la aplicación WireGuard en tu PC y haz clic en **"Activar"** antes de continuar con cualquier paso de esta fase.

> [!example] 🎬 Antes de empezar (todavía SIN grabar, y luego arranca)
> Ya conoces el método desde los prerrequisitos, así que va solo el recordatorio:
> 1. **Crea la entrada de apuntes** de esta fase (`b5-aws-8-integracion-del-cliente-windows-11.md`) con su estructura, vacía.
> 2. **Léete los 6 pasos** del procedimiento enteros, para no atascarte a mitad del vídeo.
> 3. Ten **OBS** listo y comprueba **pantalla y micrófono**.
>
> Cuando lo tengas: **arranca la grabación, preséntate y muestra tu identidad**. A partir de ahí, **todo queda grabado** — incluido cualquier paso previo de preparación que venga a continuación.

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
> Te pedirá la contraseña del `Administrator` del dominio (`P@ssw0rd`) en una ventana emergente. Al confirmarla, el PC se une al dominio y se reinicia automáticamente.
>
> **Opción B — Interfaz gráfica:**
> 1. Abre **Configuración** (tecla `Windows + I`).
> 2. Ve a **Sistema** → **Acerca de** → **"Cambiar nombre de este PC (avanzado)"** → pestaña **"Nombre de equipo"** → botón **"Cambiar..."**.
> 3. Selecciona **"Dominio"** e introduce: `BOOCHAN.SPACE`
> 4. Pulsa **Aceptar**. Te pedirá credenciales: introduce `Administrator` y `P@ssw0rd`.
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
> - **Contraseña:** `P@ssw0rd`
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

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_8.5_Fundamento_Teorico]] | [[Fase_8]] | [[Fase_8.7_Resolucion_Problemas]] |
