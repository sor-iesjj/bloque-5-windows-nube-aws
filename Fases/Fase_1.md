## 🏗️ Fase 1: Infraestructura Cloud (AWS EC2 — Windows Server 2025)

### Infraestructura de Servidores Cloud

> **[Módulo: SOR — Sistemas Operativos en Red]**
> **[U.T. 1, 2 y 3: Instalación de Sistemas Operativos en Red]**
> **[RA.01]** Instala sistemas operativos en red describiendo sus características e interpretando la documentación técnica.
>
> **Profesor:** Pedro Navarro Miralles
> **Correo:** p.navarromiralles2@edu.gva.es
> **Centro:** IES Jorge Juan (ALICANTE)
>
> **⏱️ Tiempo estimado:** ~1,5 - 2 horas (teoría + práctica + retos + troubleshooting)
> **Requisitos:** 8 GB RAM (instancia) | AWS Console | Cliente de Escritorio Remoto (RDP) — ya viene instalado en Windows; en Mac/Linux hay que descargarlo

---

> [!important] 📹 Obligaciones de grabación (LÉEME — es igual en TODAS las fases)
> Esta práctica se **graba entera con OBS**, de principio a fin. No es un repaso al final: quiero ver **cómo lo haces tú**.
> 1. **Prepárate primero (sin grabar):** comprueba lo necesario, **léete el procedimiento entero** y **crea la entrada de apuntes de esta fase** en Obsidian: fichero `v3-1-fase-1-infraestructura-cloud-aws-ec2-windows-se.md` dentro de `00_Apuntes/Trimestre_N/B5_Windows_Nube/`, con la estructura de la Fase 0.1 y **vacía**. Rellenarla es cosa tuya, después.
> 2. **Arranca OBS y PRESÉNTATE:** *"Hola, me llamo [Nombre], 2.º SMR, y en este vídeo voy a explicar la Fase 1 de Boochan V3.1 — Infraestructura Cloud (AWS EC2 — Windows Server 2025)."* Y **muestra algo que demuestre que eres tú** (tu perfil de GitHub, tu Teams o tu correo `@alu.edu.gva.es`). Di qué vas a hacer.
> 3. **Graba TODO el procedimiento**, explicando cada paso en voz alta mientras lo haces.
> 4. **Timestamps SIEMPRE** en la descripción: `00:00 Presentación` + uno por cada paso.
> 5. **Al terminar:** nombra el vídeo `V3.1 · Fase 1 — Infraestructura Cloud (AWS EC2 — Windows Server 2025)`, súbelo a tu playlist de YouTube **`B5_Windows_Nube`** (No listado) y **copia su enlace**.
> 6. **~8-10 min.** Esta fase es más larga que las de prerrequisitos: ve al grano, pero no te saltes pasos. Si se te va mucho, **pártela en dos vídeos** y ponlos los dos en la entrada.
> 7. **El enlace del vídeo va DENTRO de tu entrada de apuntes**, en el apartado `Enlace al vídeo explicativo`. Ahí, no en un papel.
> 8. **La entrega va por la TAREA de Teams.** Abriré una tarea que cubrirá **esta fase y otras**; te llegará notificación con fecha límite.

---

### 🎯 Objetivos de la fase

Al terminar esta fase serás capaz de:

- [ ] Explicar qué es IaaS y por qué usamos AWS en lugar de un servidor físico en el aula.
- [ ] Crear una instancia **EC2** con **Windows Server 2025** dimensionada de forma realista para un futuro controlador de dominio con interfaz gráfica.
- [ ] Configurar un **Security Group** que abre solo el puerto RDP (3389), entendiendo que el resto de puertos se irán abriendo fase a fase.
- [ ] Generar un **Key Pair (.pem)** y usarlo para **descifrar la contraseña inicial** del Administrador — un mecanismo distinto al login SSH de BoochanV3 y a la contraseña elegida en Azure de BoochanV2.1.
- [ ] Asignar una **Elastic IP** para tener una dirección fija.
- [ ] Conectarte al servidor por **RDP (Escritorio Remoto)** desde tu propio ordenador.
- [ ] Conocer el nombre de dominio (`BOOCHAN.SPACE`) que usará todo el proyecto BoochanV3.1.

---

### 🎯 ¿Dónde Estamos?

> [!info] El Punto de Partida
> No vienes de una fase anterior — esta es la base. Necesitas un servidor que esté **disponible 24/7**, que no dependa de tu ordenador personal, que sea escalable, profesional y seguro. En BoochanV3 (Ubuntu + Samba) ese servidor vivía en AWS con Linux; en **BoochanV3.1** vamos a construir el mismo tipo de servidor cloud, en la misma nube de Amazon, pero con **Windows Server 2025** y su rol nativo de directorio (AD DS), sin Samba de por medio.

> [!warning] El Problema
> Instalar un servidor físico en la clase es caro, requiere mantenimiento constante y no es escalable. Además, Windows Server con interfaz gráfica y Active Directory consume bastantes más recursos que un Ubuntu Server headless: no vale con "la instancia más barata que encuentres", hay que dimensionarla con cabeza — y en AWS Academy eso también significa vigilar el consumo de crédito. La nube resuelve la disponibilidad; nosotros resolvemos el dimensionado.

> [!success] Objetivo de esta Fase
> Crear una **instancia EC2 en Amazon Web Services** llamada `WindowsServer`, con la AMI **Windows Server 2025 Base**, dimensionada en `t3.large` (2 vCPU, 8 GB RAM). Este servidor será, en las próximas fases, tu controlador de dominio Active Directory nativo. Lo protegerás con un **Security Group (firewall cloud)** que bloquea internet y abre solo el puerto imprescindible por ahora: RDP.

> [!tip] Hoja de Ruta
> 1. Crear el Key Pair (`boochan-key`) — aquí no sirve para SSH, sirve para descifrar la contraseña de Windows
> 2. Crear el Security Group con la regla RDP (3389)
> 3. Lanzar la instancia EC2 `WindowsServer` (Windows Server 2025 Base, `t3.large`)
> 4. Asignar una Elastic IP para tener una IP pública fija
> 5. Obtener la contraseña de Administrador con "Get Windows Password" / "Obtener contraseña de Windows"
> 6. Conectarte por **RDP** desde tu PC con el usuario `Administrator`
> 7. Verificar acceso a internet y medir la RAM base con el `Administrador de tareas`
> 8. Anotar el dominio de todo el proyecto: `BOOCHAN.SPACE`
>
> **Resultado Final:** Un servidor Windows Server 2025 en AWS, listo, accesible por RDP, y protegido perimetralmente.
> **Siguiente:** Fase 2 (Purga y Preparación del Entorno) — limpiaremos el servidor de roles y configuración innecesaria y prepararemos el terreno para el rol AD DS.

---

### 📚 Fundamento Teórico Avanzado

> [!info] ¿Por qué usamos la Nube (IaaS)?
> El concepto de **IaaS (Infraestructura como Servicio)** es el primer pilar de la administración moderna. Tradicionalmente instalaríamos Windows Server introduciendo un DVD o un USB en una máquina física en el aula. En este proyecto damos el salto profesional: en lugar de tocar un ordenador físico, alquilamos recursos en centros de datos masivos de Amazon Web Services (AWS), el mayor proveedor de nube del mundo.

> [!abstract] 1. La "Magia" del Hipervisor y la Virtualización
> El **Hipervisor** es una capa de software de bajo nivel que gestiona los recursos de un servidor físico real y te entrega una porción exacta de CPU, RAM y disco.
> - **Tu servidor:** Cree que es un ordenador físico completo.
> - **La Realidad:** Es un archivo ejecutándose dentro de otro ordenador gigante. Esto permite que un solo superordenador de AWS albergue cientos de servidores de alumnos de forma aislada.
> - **En AWS, esto se llama EC2 (Elastic Compute Cloud):** el servicio que alquila potencia de cómputo bajo demanda, pagando solo por lo que usas.

> [!warning] 2. El Modelo de Responsabilidad Compartida
> Trabajar en la nube no significa que "todo es mágico y seguro". AWS funciona bajo este modelo:
> *   **Responsabilidad de Amazon:** Seguridad física (evitar robos de discos), electricidad y conexión a Internet.
> *   **Tu Responsabilidad (El Administrador):** Eres el responsable absoluto de lo que ocurre **dentro** de tu instancia. Si configuras mal una contraseña o dejas un puerto abierto, los hackers entrarán. ¡Amazon no te protegerá de tus propios errores de configuración!

> [!important] 3. Windows Server con Interfaz Gráfica: por qué pesa más que Ubuntu Server
> En BoochanV3 (Ubuntu) usábamos un servidor **"Headless"** (sin entorno gráfico) para ahorrar RAM. Aquí hacemos una excepción deliberada: la AMI **Windows Server 2025 Base** ya incluye interfaz gráfica (Desktop Experience) por defecto, porque es tu primera vez administrando Active Directory de forma nativa y necesitas ver herramientas como `Administrador del servidor`, `Usuarios y equipos de Active Directory` o el `Administrador de DNS` mientras aprendes.
> * **Consecuencia práctica:** Windows Server + interfaz gráfica + AD DS necesita bastante más RAM que un Ubuntu Server headless. Por eso en BoochanV3.1 usamos `t3.large` (2 vCPU, **8 GB** de RAM) en lugar del `t3.small`/`t3.medium` (2-4 GB) que bastaba en la versión Ubuntu de BoochanV3.
> * **⚠️ Consumo de crédito en AWS Academy:** una instancia Windows `t3.large` consume el crédito del Learner Lab **más rápido** que una instancia Linux equivalente, porque AWS factura una licencia de Windows Server además del propio cómputo. Usa siempre el botón **"End Lab"** al terminar cada sesión — no dejes la instancia corriendo de un día para otro.

> [!note] 4. Seguridad Perimetral y Protocolos (TCP vs UDP)
> Antes de encender el servidor, lo protegemos con un **Security Group**, que actúa como la muralla del castillo. Solo abriremos las "puertas" (puertos) necesarias usando estos protocolos:
> *   **TCP (Transmission Control Protocol):** Para administrar el servidor (RDP) y archivos (SMB). TCP exige confirmación de entrega. Si un dato se pierde, se vuelve a pedir. Es **fiable pero más lento**.
> *   **UDP (User Datagram Protocol):** Para la VPN (WireGuard) y la hora (NTP). UDP dispara los paquetes a máxima velocidad sin preguntar nada. Es **rapidísimo pero menos fiable**.

> [!important] 5. RDP en vez de SSH: la puerta de entrada cambia de naturaleza
> En BoochanV3 (Ubuntu), administrábamos el servidor con SSH (línea de comandos cifrada, puerto 22). En Windows Server, la herramienta equivalente para administración remota gráfica es **RDP (Remote Desktop Protocol)**, puerto **3389**. La diferencia es de fondo, no solo de puerto:
> * SSH te da una terminal de texto remota.
> * RDP te da **el escritorio completo del servidor**, como si estuvieras sentado delante de él — puedes abrir el `Administrador del servidor`, hacer clic con el ratón, ver ventanas.
> * Esto es coherente con que la AMI de Windows Server incluye interfaz gráfica: si vas a usar herramientas gráficas, necesitas un protocolo que te lleve el escritorio entero, no solo una consola.

> [!important] 6. Key Pair (.pem): el mismo fichero, un uso totalmente distinto
> En **BoochanV3 (Ubuntu)** el Key Pair `.pem` se usaba directamente para autenticar la sesión SSH: la clave privada abría la puerta. En **BoochanV2.1 (Azure, Windows)** ni siquiera existía un Key Pair — el usuario y la contraseña se elegían a mano al crear la máquina virtual. En **AWS con una instancia Windows**, el mecanismo es un tercer caso, distinto de los dos anteriores:
> * AWS **genera automáticamente** una contraseña aleatoria y compleja para la cuenta `Administrator` al lanzar la instancia.
> * Esa contraseña se guarda **cifrada** con la clave pública de tu Key Pair — nadie puede leerla, ni siquiera Amazon.
> * Para recuperarla, subes tu fichero `.pem` (la clave privada) al botón **"Get Windows Password"** de la consola EC2, y AWS la descifra **en tu navegador** y te la muestra en texto plano, **solo esa vez**.
> * A partir de ahí, esa contraseña funciona como una contraseña normal de Windows: la usas para conectarte por RDP y, si quieres, la cambias.
>
> Resumen de los tres modelos que vas a haber visto en el módulo:
>
> | Proyecto | Cloud | Mecanismo de acceso inicial |
> | :--- | :--- | :--- |
> | BoochanV3 (Ubuntu) | AWS | Key Pair `.pem` → login SSH directo |
> | BoochanV2.1 (Windows) | Azure | Usuario + contraseña elegidos al crear la VM |
> | **BoochanV3.1 (Windows)** | **AWS** | **Key Pair `.pem` → descifra la contraseña inicial → login RDP** |
>
> **⚠️ Si pierdes el fichero `.pem`, pierdes la posibilidad de recuperar esa contraseña inicial.** Guárdalo bien, igual que en BoochanV3.

> [!important] 7. ¿Por qué seguimos usando `BOOCHAN.SPACE` y no un dominio `.LOCAL`?
> A diferencia de un laboratorio local (sin salida a Internet, donde tendría sentido un dominio `.LOCAL`), aquí el servidor tiene una **IP pública real** en AWS. Mantenemos el mismo dominio real que ya usa BoochanV3: NetBIOS `BOOCHAN`, Realm **`BOOCHAN.SPACE`**. Es el mismo proyecto, la misma identidad de red; solo cambian el sistema operativo del controlador de dominio y el proveedor cloud respecto a BoochanV2.1.

### 📖 Diccionario de Conceptos Clave

> [!quote] Terminología Profesional AWS
> - **EC2 (Elastic Compute Cloud):** El servicio de AWS que alquila instancias (máquinas virtuales).
> - **Instancia:** Una máquina virtual activa y ejecutándose en AWS.
> - **AMI (Amazon Machine Image):** La "fotografía" del sistema operativo con la que arranca tu instancia. Equivale al DVD/ISO de instalación.
> - **Security Group:** Un firewall lógico que controla qué tráfico de red entra y sale de tu instancia.
> - **Elastic IP (EIP):** Una dirección IP pública fija reservada para ti en AWS. Sin ella, tu instancia cambia de IP cada vez que se reinicia.
> - **Key Pair (.pem):** Fichero de clave privada RSA. En una instancia Windows, sirve para descifrar la contraseña inicial del Administrador (no para hacer login directo, como en Linux).
> - **RDP (Remote Desktop Protocol):** Protocolo de Microsoft que transmite el escritorio completo de un servidor remoto a tu pantalla, cifrado, para administrarlo como si estuvieras delante.
> - **VPC (Virtual Private Cloud):** La red privada virtual donde viven tus instancias EC2.
> - **Get Windows Password:** Función de la consola EC2 que descifra, usando tu `.pem`, la contraseña inicial de la cuenta `Administrator` de una instancia Windows.

---

### 🛠️ Procedimiento Práctico (BoochanV3.1)

> [!danger] 🔑 LÉEME ANTES DE EMPEZAR: cómo accedemos sin pagar
> Este proyecto **NO usa una cuenta normal de AWS con tarjeta de crédito**. Para un aula, la vía correcta es **AWS Academy Learner Lab**:
> - El centro se da de alta en **AWS Academy** (gratis para instituciones educativas). El profesor invita a cada alumno por correo.
> - Cada alumno entra en un **"Learner Lab"** con un **crédito de unos 50-100 USD** ya cargado. **No se pide tarjeta de crédito ni datos bancarios** — ideal para alumnos menores de edad.
> - El laboratorio se **enciende y apaga** con un botón ("Start Lab" / "End Lab"). Mientras está apagado, **no consume crédito**.
>
> > [!warning] ⚠️ El "Free Tier" gratuito NO sirve para este proyecto
> > La capa gratuita de AWS (12 meses) solo da instancias de **1 GB de RAM** y, además, no cubre licencias de Windows Server de forma gratuita indefinida. Windows Server 2025 con Desktop Experience y, más adelante, AD DS necesita **8 GB de RAM** (Paso 3), así que con el Free Tier puro este proyecto no es viable. Por eso usamos AWS Academy, cuyo crédito sí cubre una `t3.large`.
>
> > [!warning] 💸 Una instancia Windows consume crédito más rápido que una Linux
> > A diferencia de BoochanV3 (Ubuntu), aquí AWS factura, además del cómputo, el **coste de la licencia de Windows Server**. Una `t3.large` con Windows gasta el crédito del Learner Lab notablemente más rápido que la `t3.small`/`t3.medium` con Ubuntu de BoochanV3. **Usa siempre el botón "End Lab" al terminar la sesión de clase** — nunca dejes la instancia encendida entre clase y clase.
>
> Una vez dentro de la consola, asegúrate de seleccionar la **región correcta** en el desplegable de arriba a la derecha (la que indique tu profesor; en AWS Academy suele ser fija, p. ej. `us-east-1`). **Todos tus recursos deben estar en la misma región.**

> [!example] 🎬 Antes de empezar (todavía SIN grabar, y luego arranca)
> Ya conoces el método desde los prerrequisitos, así que va solo el recordatorio:
> 1. **Crea la entrada de apuntes** de esta fase (`v3-1-fase-1-infraestructura-cloud-aws-ec2-windows-se.md`) con su estructura, vacía.
> 2. **Léete los 7 pasos** del procedimiento enteros, para no atascarte a mitad del vídeo.
> 3. Ten **OBS** listo y comprueba **pantalla y micrófono**.
>
> Cuando lo tengas: **arranca la grabación, preséntate y muestra tu identidad**. A partir de ahí, **todo queda grabado** — incluido cualquier paso previo de preparación que venga a continuación.

> [!example] Paso 1: Crear el Key Pair (tu llave para obtener la contraseña)
> Antes de crear la instancia, necesitas generar el par de claves. Aquí no la usarás para hacer login directo (como en Ubuntu), sino para **descifrar la contraseña inicial** de Windows en el Paso 5.
>
> 1. En la barra de búsqueda superior, escribe `EC2` y haz clic en el servicio.
> 2. En el menú izquierdo, dentro de **Red y seguridad**, haz clic en **`Pares de claves`** (Key Pairs).
> 3. Haz clic en **`Crear par de claves`**.
> 4. Rellena el formulario:
>
> | Campo | Valor |
> | :--- | :--- |
> | **Nombre** | `boochan-key` |
> | **Tipo de par de claves** | `RSA` |
> | **Formato de archivo de clave privada** | `.pem` (para Linux/Mac/Windows con OpenSSH) |
>
> 5. Haz clic en **`Crear par de claves`**. Se descargará automáticamente un fichero `boochan-key.pem`.
>
> > [!warning] ⚠️ ¡Guarda el fichero .pem ahora!
> > Este fichero **solo se descarga una vez**. Sin él, en el Paso 5 no podrás descifrar la contraseña de tu servidor y tendrías que recrear la instancia con un Key Pair nuevo. Guárdalo en una carpeta segura de tu PC. **No lo compartas con nadie.**

> [!example] Paso 2: Crear el Security Group
> El Security Group actúa como el firewall de tu instancia. Lo creamos antes de lanzar la instancia para poder asignárselo en el momento de la creación.
>
> 1. En el menú izquierdo de EC2, dentro de **Red y seguridad**, haz clic en **`Grupos de seguridad`**.
> 2. Haz clic en **`Crear grupo de seguridad`**.
> 3. Rellena:
>
> | Campo | Valor |
> | :--- | :--- |
> | **Nombre del grupo de seguridad** | `sg-boochan-[tunombre]` |
> | **Descripción** | `Seguridad BoochanV3.1 SOR` |
> | **VPC** | La VPC por defecto (default) |
>
> 4. En **Reglas de entrada**, haz clic en **`Agregar regla`** y añade esta regla:
>
> | Tipo | Protocolo | Puerto | Origen | Descripción |
> | :--- | :--- | :--- | :--- | :--- |
> | RDP | TCP | 3389 | 0.0.0.0/0 | Acceso RDP inicial |
>
> > [!warning] ⚠️ El origen 0.0.0.0/0 significa "cualquier IP del mundo"
> > Es necesario de momento porque aún no tenemos VPN. En la **Fase 3**, cuando WireGuard esté funcionando, restringiremos RDP y lo cerraremos al exterior. Por ahora es el único acceso posible.
>
> 5. Deja las **Reglas de salida** por defecto (todo el tráfico saliente permitido).
> 6. Haz clic en **`Crear grupo de seguridad`**.
>
> > [!info] 💡 ¿Por qué solo un puerto de momento?
> > A diferencia de BoochanV2.1 (Azure), donde se abrían las 12 reglas del proyecto entero desde el principio, aquí seguimos el mismo criterio que BoochanV3 (Ubuntu): abrimos **solo lo estrictamente necesario para esta fase**. Los puertos de Kerberos, DNS, LDAP, SMB, RPC, NTP y WireGuard se añadirán en las fases donde realmente se necesiten (a partir de la Fase 4, con el rol AD DS). Así nunca abrimos una puerta sin saber para qué sirve.
>
> > [!note] 💡 AWS Security Group vs Azure NSG
> > Un Security Group de AWS **no tiene botón de "deshabilitar regla" ni acción de "denegar"** como el NSG de Azure. Solo existen reglas que permiten. Para "apagar" temporalmente un acceso hay que **borrar la regla** y, si hace falta, volver a crearla.

> [!example] Paso 3: Lanzar la Instancia EC2
> Ahora lanzamos el servidor con Windows Server 2025.
>
> 1. En el menú izquierdo, haz clic en **`Instancias`** → **`Lanzar instancias`**.
> 2. Rellena el formulario con exactamente estos valores:
>
> | Campo | Valor |
> | :--- | :--- |
> | **Nombre** | `WindowsServer` |
> | **AMI (imagen)** | Busca `Windows Server 2025 Base` (Quick Start) → selecciona la versión de 64 bits (x86) |
> | **Tipo de instancia** | `t3.large` (2 vCPU, 8 GB RAM) |
> | **Par de claves** | `boochan-key` (el que creaste en el Paso 1) |
> | **Grupo de seguridad** | Selecciona el existente: `sg-boochan-[tunombre]` |
> | **Almacenamiento** | 50 GB gp3 (disco SSD) |
>
> > [!warning] ⚠️ Por qué 50 GB y no los 20 GB de la versión Ubuntu
> > Windows Server 2025 con interfaz gráfica ocupa bastante más disco que Ubuntu Server. 20 GB se quedarían muy justos en cuanto instales el rol AD DS y apliques actualizaciones. 50 GB da margen para todo el itinerario.
>
> 3. Haz clic en **`Lanzar instancia`**. Espera **3-5 minutos** (Windows Server tarda más en aprovisionarse que Ubuntu Server) hasta que el estado sea `En ejecución` (verde) y las comprobaciones de estado estén en verde.
>
> > [!tip] 💡 ¿Qué es una AMI?
> > Una AMI (Amazon Machine Image) es como el DVD/ISO de un sistema operativo, pero ya preparada para AWS. En lugar de instalar Windows Server desde cero, AWS carga una "fotografía" del sistema en pocos minutos. `Windows Server 2025 Base` es una AMI oficial de Amazon, disponible en el catálogo "Quick Start" al lanzar la instancia.

> [!example] Paso 4: Asignar una Elastic IP
> Sin una IP fija, tu instancia cambia de IP pública cada vez que se reinicia. La Elastic IP te da una IP permanente.
>
> 1. En el menú izquierdo, dentro de **Red y seguridad**, haz clic en **`Direcciones IP elásticas`**.
> 2. Haz clic en **`Asignar dirección IP elástica`** → **`Asignar`**.
> 3. Selecciona la IP recién creada → **`Acciones`** → **`Asociar dirección IP elástica`**.
> 4. En el formulario, selecciona tu instancia `WindowsServer` → **`Asociar`**.
>
> Anota la IP elástica asignada: la necesitarás en todas las fases del proyecto.
>
> > [!warning] 💸 Cuidado con el coste de la Elastic IP cuando apagas la instancia
> > AWS regala la Elastic IP **mientras está asociada a una instancia en ejecución**. Pero si **detienes** (stop) la instancia y la EIP queda reservada sin usarse, AWS cobra una pequeña tarifa por hora, que en AWS Academy consume crédito. Por eso, en este proyecto, en lugar de *detener* la instancia entre clases usamos el botón **"End Lab"** del laboratorio, que congela todo el entorno sin penalización.

> [!example] Paso 5: Obtener la contraseña de Administrador ("Get Windows Password")
> A diferencia de Azure (donde eliges usuario y contraseña al crear la VM) o de Ubuntu en AWS (donde la clave `.pem` sirve directamente para el login SSH), en una instancia **Windows** de AWS la contraseña la genera Amazon y hay que **descifrarla con tu Key Pair**.
>
> 1. En **`Instancias`**, selecciona tu instancia `WindowsServer`.
> 2. Haz clic en **`Conectar`** (arriba a la derecha) o en **`Acciones`** → **`Seguridad`** → **`Obtener contraseña de Windows`** ("Get Windows Password").
> 3. Haz clic en **`Cargar archivo de clave privada`** y selecciona tu `boochan-key.pem` descargado en el Paso 1.
> 4. Haz clic en **`Descifrar contraseña`**.
> 5. AWS te mostrará la contraseña de la cuenta `Administrator` **en texto plano, solo esta vez**. Cópiala y guárdala en un lugar seguro — no podrás volver a verla desde la consola.
>
> > [!warning] ⚠️ Espera a que la instancia esté lista
> > El botón "Obtener contraseña de Windows" puede tardar unos minutos en estar disponible tras lanzar la instancia (AWS necesita generar y cifrar la contraseña dentro de la máquina). Si aparece un error o el botón está desactivado, espera 3-4 minutos más y vuelve a intentarlo.
>
> > [!tip] 💡 ¿Por qué esto es más seguro que elegir tú la contraseña?
> > AWS genera una contraseña aleatoria larga y compleja, evitando el error humano de poner algo débil o repetido. Y como viaja cifrada con tu clave pública, solo quien tenga el `.pem` correspondiente puede descifrarla — ni siquiera el personal de Amazon puede leerla.

> [!example] Paso 6: Primera Conexión al Servidor (RDP)
> Con la Elastic IP y la contraseña ya en tu poder:
>
> **En Windows (cliente ya instalado):**
> 1. Pulsa `Windows + R`, escribe `mstsc` y pulsa Enter (o busca "Conexión a Escritorio remoto" en el menú Inicio).
> 2. En el campo **Equipo**, escribe la IP elástica que anotaste.
> 3. Pulsa **`Conectar`**.
> 4. Cuando te pida credenciales, introduce:
>    - **Usuario:** `Administrator`
>    - **Contraseña:** la que descifraste en el Paso 5
> 5. Aparecerá una advertencia de certificado ("No se puede verificar la identidad del equipo remoto"). Es normal la primera vez porque el certificado es autofirmado — pulsa **`Sí`** para continuar.
>
> **En Mac / Linux:**
> Instala el cliente **Microsoft Remote Desktop** (Mac App Store) o `Remmina`/`rdesktop` (Linux), y repite los mismos pasos: IP elástica, usuario `Administrator`, contraseña.
>
> Si al final ves el escritorio de Windows Server con el `Administrador del servidor` abierto automáticamente, **ya estás dentro de tu servidor**. A partir de aquí, todo lo que hagas con el ratón y el teclado se ejecuta en AWS, a cientos de kilómetros.
>
> > [!tip] 💡 ¿Qué es RDP?
> > RDP (Remote Desktop Protocol) es como un **"mando a distancia" cifrado que te lleva la pantalla entera** del servidor a tu monitor. Desde tu ratón y teclado del aula estás controlando un ordenador de AWS, a cientos de kilómetros, como si tuvieras el monitor del servidor delante. Todo el tráfico viaja cifrado.
>
> > [!important] 💡 El usuario por defecto en Windows de AWS es `Administrator`, no `admin` ni `root`
> > A diferencia de Ubuntu en AWS (usuario `ubuntu`) o de BoochanV2.1 en Azure (usuario `azureadmin`, elegido por ti), la cuenta administradora local de una instancia Windows de AWS siempre se llama `Administrator`.

> [!example] Paso 7: Verificación de Internet y Medida de RAM Base
> Ya dentro del servidor por RDP:
>
> 1. Abre PowerShell y comprueba la salida a Internet:
>    ```powershell
>    ping google.com
>    ```
>    Deberías recibir respuestas sin pérdida de paquetes.
> 2. Comprueba también información general del sistema:
>    ```powershell
>    systeminfo | Select-String "OS Name","Total Physical Memory"
>    ```
> 3. Abre el **`Administrador de tareas`** (`Ctrl+Shift+Esc`) → pestaña **`Rendimiento`** → **`Memoria`**. Anota cuánta RAM está en uso ahora mismo, con el sistema recién instalado y sin roles adicionales. Guarda ese dato — lo compararás en la Fase 4, cuando el rol AD DS esté instalado y funcionando, para ver cuánta RAM consume el dominio.
> 4. Anota también el dominio de todo el proyecto, lo necesitarás desde la Fase 4 en adelante:
>
> | Concepto | Valor en BoochanV3.1 |
> | :--- | :--- |
> | **Nombre NetBIOS** | `BOOCHAN` |
> | **Realm (dominio completo)** | `BOOCHAN.SPACE` |
> | **Nombre del servidor** | `WindowsServer` |
> | **Usuario administrador local** | `Administrator` |
> | **Security Group** | `sg-boochan-[tunombre]` |
>
> > [!tip] 💡 ¿Cómo verifico si los puertos están "vivos" dentro del servidor?
> > Una vez que tengas servicios corriendo en las siguientes fases, puedes usar este comando de PowerShell (como administrador) para ver qué puertos están escuchando en tu servidor:
> > ```powershell
> > Get-NetTCPConnection -State Listen | Sort-Object LocalPort
> > ```

---

### 🚩 Resolución de Problemas y Evaluación

> [!bug] Tabla de Troubleshooting (¿Algo no funciona?)
> | Problema | Causa Probable | Solución Sugerida |
> | :--- | :--- | :--- |
> | El botón "Obtener contraseña de Windows" está desactivado o da error. | La instancia todavía no ha terminado de generar/cifrar la contraseña interna. | Espera 3-4 minutos tras ver el estado "En ejecución" y vuelve a intentarlo. |
> | No puedo conectar por RDP ("No se puede establecer la conexión"). | La instancia no ha terminado de arrancar/aprovisionarse. | Espera unos minutos (Windows Server tarda más que Ubuntu) y vuelve a intentarlo. |
> | RDP se conecta pero rechaza las credenciales. | Usuario o contraseña incorrectos, o Bloq Mayús activado. | Comprueba que escribes exactamente `Administrator` y la contraseña descifrada en el Paso 5, respetando mayúsculas/símbolos. |
> | Aparece "Este equipo no puede conectarse al equipo remoto". | El puerto 3389 no está abierto en el Security Group, o la regla se borró por error. | Revisa **Security Group `sg-boochan-[tunombre]`** y confirma que la regla RDP (3389) existe con origen `0.0.0.0/0`. |
> | El servidor **no responde al ping**. | El protocolo ICMP está bloqueado por defecto en el Security Group. | Es normal por seguridad. No abras el ping; usa RDP para verificar conectividad. |
> | "Host key verification failed" o la IP ha cambiado. | No se ha asignado o asociado correctamente la Elastic IP. | Revisa el Paso 4: la Elastic IP debe estar **asociada** a la instancia `WindowsServer`, no solo asignada. |
> | La pantalla de RDP se ve muy lenta o pixelada. | Configuración de calidad de la conexión demasiado alta para el ancho de banda del aula. | En el cliente RDP, antes de conectar, baja la calidad de color/experiencia en las opciones avanzadas de la conexión. |

> [!help] Preguntas Críticas (Autoevaluación del alumno)
> 1. ¿Por qué Amazon AWS es responsable del hardware pero tú eres el responsable del Sistema Operativo?
> 2. ¿Qué ocurre exactamente si dejas el puerto de administración RDP (3389) abierto a **"Cualquiera" (0.0.0.0/0)** permanentemente?
> 3. Compara los tres mecanismos de acceso inicial que has visto en el módulo: SSH directo con `.pem` (BoochanV3), usuario+contraseña elegidos (BoochanV2.1) y `.pem` para descifrar una contraseña generada por AWS (BoochanV3.1). ¿Qué ventaja de seguridad tiene este último frente a los otros dos?
> 4. ¿Por qué RDP transmite el escritorio completo mientras que SSH (usado en BoochanV3) solo transmite texto? ¿Qué ventajas e inconvenientes tiene cada enfoque?
> 5. 🔬 **Reto práctico:** Entra en el Security Group de AWS y **elimina** temporalmente la regla del puerto 3389 (y vuelve a añadirla). Intenta conectarte por RDP mientras la regla no existe. ¿Qué ocurre? ¿Qué has comprobado con este experimento?
> 6. 🔬 **Reto práctico:** Anota en el `Administrador de tareas` cuánta RAM usa el sistema base recién instalado. Guarda ese dato — lo compararás en la Fase 4, cuando el rol AD DS esté corriendo.

---

> [!caution] 🛑 Auditoría de Seguridad — Tarea pendiente tras la Fase 3
> Una vez que la VPN esté funcionando, cerrarás el acceso RDP directo desde Internet. **No lo hagas ahora**: sin VPN activa te quedarías sin acceso al servidor.
>
> **Acción — Restringir el puerto 3389 al mundo exterior en el Security Group de AWS:**
> Vuelve a **Security Group `sg-boochan-[tunombre]`** → **Reglas de entrada**. Localiza la regla `RDP` (puerto 3389, origen `0.0.0.0/0`) y cámbiala para que el **Origen** sea únicamente el rango de la red privada de la VPN, o elimínala si vas a acceder siempre a través del túnel WireGuard.
>
> Esto es aplicar seguridad "Zero Trust": nadie en Internet puede llegar al servidor por RDP; solo quien esté dentro de la VPN.

---

### ✅ Checklist Final de la Fase 1

- [ ] Key Pair `boochan-key` creado y fichero `.pem` guardado en lugar seguro.
- [ ] Security Group `sg-boochan-[tunombre]` creado con la regla RDP (3389, origen `0.0.0.0/0`).
- [ ] Instancia EC2 `WindowsServer` lanzada con AMI `Windows Server 2025 Base`.
- [ ] Tipo de instancia: `t3.large` (2 vCPU, 8 GB RAM), disco 50 GB gp3.
- [ ] Elastic IP asignada y **asociada** a la instancia.
- [ ] Contraseña de `Administrator` obtenida con "Get Windows Password" y guardada en lugar seguro.
- [ ] Conexión por RDP realizada con éxito desde tu propio ordenador.
- [ ] `ping google.com` funciona desde dentro del servidor.
- [ ] RAM base anotada desde el `Administrador de tareas` (para comparar en la Fase 4).
- [ ] Dominio del proyecto anotado: NetBIOS `BOOCHAN`, Realm `BOOCHAN.SPACE`.

> **Siguiente paso:** Fase 2 — Purga y Preparación del Entorno, donde revisaremos la configuración inicial de Windows Server (nombre de host, actualizaciones, roles innecesarios) y prepararemos el terreno para el rol AD DS.

---

### ✅ Entregables y cierre

> [!abstract] Qué tienes que tener hecho al acabar esta fase
> | Entregable | Dónde vive | Qué debe contener |
> | :--- | :--- | :--- |
> | **Entrada de apuntes** | `00_Apuntes/Trimestre_N/B5_Windows_Nube/v3-1-fase-1-infraestructura-cloud-aws-ec2-windows-se.md` | Estructura completa + **respuestas a las Preguntas Críticas y al 🔬 Reto** + **enlace del vídeo** |
> | **Vídeo** | Playlist `B5_Windows_Nube` (No listado) | Nombrado `V3.1 · Fase 1 — Infraestructura Cloud (AWS EC2 — Windows Server 2025)`, con presentación, identidad y timestamps |
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
