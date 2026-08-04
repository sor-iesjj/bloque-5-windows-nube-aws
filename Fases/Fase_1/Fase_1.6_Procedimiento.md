## Fase 1 · Apartado 6 — 🛠️ Procedimiento práctico

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Infraestructura Cloud (AWS EC2 — Windows Server 2025)**
> 🧭 Índice de la fase: [[Fase_1]]
>
> **📍 Cuándo se lee:** **Con la VM delante.** Aquí está el trabajo.

---

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
> 2. **Léete los 8 pasos** del procedimiento enteros, para no atascarte a mitad del vídeo.
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

> [!example] 🔌 Paso 8 — EJERCICIO DE VERIFICACIÓN: comprueba tu red desde fuera
> Hasta aquí has configurado la red **y has confiado en que el panel dice la verdad**. Ahora vas a comprobarlo con fuentes **externas e independientes**, que es como se hace de verdad.
>
> > [!info] ¿Qué es una API y por qué la usa un administrador?
> > Una **API** es una web hecha para que la consulte un programa en vez de una persona: en vez de devolverte una página con colores, te devuelve **datos limpios** en formato JSON.
> >
> > ¿Y para qué la quiere un administrador de sistemas? Para **comprobar desde fuera lo que desde dentro no puede ver**. Tu servidor te dirá siempre lo que él cree de sí mismo; un servicio externo te dice **lo que se ve realmente**. Y esa diferencia, cuando aparece, es justo donde está el problema que llevas dos horas buscando.
> >
> > Se consultan con **`curl`**, que ya has usado y que viene instalado en todas partes. Sin programar y sin instalar nada.
>
> **a) Verifica tu cálculo de subred.** Tu red es **`10.0.0.0/24`** (la VPC de AWS).
> Primero, **a mano y sin ayuda**, escribe en tu entrada de apuntes: máscara decimal, dirección de red, broadcast, número de hosts asignables, primero y último.
> Ahora compruébalo:
> ```bash
> curl "https://networkcalc.com/api/ip/10.0.0.0/24"
> ```
> Si no coincide, **no borres tu respuesta**: déjala y explica en el vídeo dónde te equivocaste. Eso enseña más que acertar.
>
> **b) Tu servidor SÍ tiene IP pública. Averigua de quién es.** Desde dentro del servidor:
> ```bash
> curl "https://api.ipify.org?format=json"
> ```
> Compárala con la que te muestra el panel de AWS: **tienen que coincidir**.
>
> Ahora pregunta **quién es el dueño de esa IP**:
> ```bash
> curl "http://ip-api.com/json/TU_IP_PUBLICA?fields=query,country,isp,org,as"
> ```
>
> > [!success] 🤔 Mira bien la respuesta
> > No sale tu nombre: sale **Amazon**, con su número de **AS** y el país del centro de datos.
> > **Eso es "estar en la nube"**, dicho con datos: tu servidor vive dentro de la infraestructura de Amazon, y para el resto de Internet es una máquina más de las suyas.
> > **Explica en el vídeo:** ¿en qué país está físicamente tu servidor? ¿Coincide con el que elegiste al crearlo?
>
> > [!question] Lo que va a tu entrada de apuntes
> > 1. ¿Coincidió tu cálculo de subred con el de la API? Si no, ¿en qué fallaste?
> > 2. ¿Cuál es la IP privada de tu servidor y cuál la pública? ¿Por qué no son la misma?
> > 3. ¿Por qué una comprobación hecha **desde el propio servidor** vale menos que una hecha desde fuera?
>
> > [!note] 📌 Para saber más
> > La teoría completa de esto está en la práctica **B1.9b — Verificar tu red con APIs públicas** del Bloque 1. Aquí lo aplicas a tu servidor de verdad.
> > Y una consecuencia que conviene que asumas ya: **tu servidor es alcanzable desde cualquier punto del planeta.** En cuanto lo enciendes empieza a recibir intentos de conexión de desconocidos. Por eso las siguientes fases dedican tanto tiempo al cortafuegos y a la VPN.

---

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_1.5_Fundamento_Teorico]] | [[Fase_1]] | [[Fase_1.7_Resolucion_Problemas]] |
