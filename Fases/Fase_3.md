## 🔒 Fase 3: Conectividad VPN (WireGuard para Windows)

### Infraestructura de Servidores Cloud (AWS + Windows Server 2025)

> **[Módulo: SOR — Sistemas Operativos en Red]**
> **[U.T. 9: Gestión remota e Integración en Red]**
> **[RA.05]** Realiza tareas de monitorización y uso del sistema operativo en red.
>
> **Profesor:** Pedro Navarro Miralles  
> **Correo:** p.navarromiralles2@edu.gva.es  
> **Centro:** IES Jorge Juan (ALICANTE)
>
> **⏱️ Tiempo estimado:** ~2 horas (teoría + práctica + retos + troubleshooting)  
> **Requisitos:** Instancia `t3.large` (AWS) | WireGuard para Windows (PC del aula y servidor) | AWS Console | RDP

---

### 🎯 ¿Dónde Estamos?

> [!info] Vienes de Fase 2
> Completaste la preparación inicial del servidor y le diste identidad (`WindowsServer`), con la IP privada de la VPC de AWS confirmada (rango `172.31.x.x`). Ahora tienes un servidor limpio, identificado, actualizado. Pero hay un problema crítico: está expuesto a internet público. El puerto RDP (3389) está abierto a todo el mundo desde la Fase 1 — bots intentarán conectarse miles de veces al día para adivinar contraseñas de Administrador.

> [!warning] El Problema
> Sin una VPN privada, tu servidor es vulnerable a ataques de fuerza bruta contra el Escritorio Remoto. Cualquiera en internet puede intentar adivinar credenciales de Administrador. Además, en las próximas fases necesitarás que el aula acceda al servidor desde cualquier lugar, pero solo el aula — no todo el mundo. Necesitas un túnel privado cifrado que solo tú controles.

> [!success] Objetivo de esta Fase
> Instalar **WireGuard para Windows**: una VPN ligera y moderna que crea un túnel P2P cifrado entre tu PC del aula (`10.0.0.2`) y el servidor (`10.0.0.1`). Este túnel es tu "puerta trasera" secreta — solo quien tenga las llaves criptográficas puede entrar. Cerrarás el puerto RDP directo a internet y aceptarás conexiones solo desde dentro de la VPN, tal y como quedó anunciado como tarea pendiente al final de la Fase 1.

> [!tip] Hoja de Ruta
> 1. Añadir regla entrante UDP 51820 en el Security Group de AWS (WireGuard escucha aquí)
> 2. Instalar WireGuard en el servidor
> 3. Generar pares de llaves criptográficas (servidor + tu PC)
> 4. Crear archivo de configuración `wg0.conf` en el servidor
> 5. Crear perfil VPN para tu PC del aula
> 6. Activar el túnel y verificar con `ping 10.0.0.1` desde tu PC
> 7. Cerrar el acceso RDP público en el Security Group (Zero Trust)
>
> **Resultado Final:** Servidor accesible solo a través del túnel VPN. Totalmente blindado contra ataques de internet público.
> **Siguiente:** Fase 4 (Dominio) — provisionar Active Directory Domain Services. Ahora que hay conexión VPN segura, puedes instalar servicios críticos.

---

### 📚 Fundamento Teórico

> [!abstract] 1. El Dilema de la Nube
> La conectividad en la nube presenta un gran reto: queremos administrar nuestro servidor desde cualquier parte, pero no queremos exponerlo a ataques de todo el mundo. La solución es crear un **Túnel VPN P2P (Peer-to-Peer)**.

> [!info] 2. ¿Qué es WireGuard? (y qué cambia en Windows respecto a Linux)
> A diferencia de protocolos antiguos (como OpenVPN), WireGuard funciona a muy bajo nivel del sistema operativo. En Linux (BoochanV3) se integra directamente en el Kernel; en **Windows**, la aplicación oficial `WireGuard.exe` incluye su propio driver de red (Tunnel Service) que consigue un rendimiento equivalente, gestionado a través de una interfaz gráfica o de `wireguard.exe /installtunnelservice`. Utiliza **criptografía de curva elíptica**, asegurando que los datos viajen por un canal 100% blindado.

> [!important] 3. Intercambio de Llaves
> El servidor y el cliente se reconocen mediante un intercambio de llaves:
> *   **Llave Pública:** Se puede compartir (es como la dirección de tu casa).
> *   **Llave Privada:** Es el secreto absoluto. Solo quien posee la llave privada puede descifrar el tráfico que le llega.

> [!note] 4. Dos redes, dos propósitos: no confundas la IP privada de AWS con la del túnel
> En este proyecto conviven dos rangos de IP que no deben mezclarse:
> *   **`172.31.x.x` (real, de AWS):** La IP privada que el servidor tiene en la VPC por defecto de AWS (la verificaste en la Fase 2). Es la "tarjeta de red física" virtual de la instancia.
> *   **`10.0.0.1` / `10.0.0.2` (virtual, del túnel WireGuard):** La red que crea la propia VPN al levantar la interfaz `wg0`. Es una interfaz de red completamente distinta, gestionada por WireGuard, que viaja **encapsulada y cifrada** dentro del tráfico real de internet hasta la IP pública/Elástica del servidor. A partir de esta fase, cuando veas `10.0.0.1` en un comando, será casi siempre la IP del túnel, no la IP privada de la VPC.

### 📖 Diccionario de Conceptos Clave

> [!quote] Terminología VPN
> - **Cifrado Asimétrico:** Sistema que usa una llave para cerrar (pública) y otra distinta para abrir (privada).
> - **wg0.conf:** El "cerebro" o archivo maestro que define la red virtual y quién puede entrar en ella.
> - **Peer:** Cada uno de los extremos de la conexión (tu PC del aula y la instancia EC2 en AWS son "Peers").
> - **Endpoint:** La dirección IP pública real del servidor a la que se conecta el túnel.
> - **Tunnel Service:** El servicio de Windows que WireGuard instala para mantener el túnel activo incluso sin sesión de usuario iniciada, similar a `systemctl enable wg-quick@wg0` en Linux.

---

### 🔓 Apertura de Puertos (Security Group de AWS)

> [!example] Al empezar: abre el puerto de WireGuard
> Antes de tocar nada en el servidor, abre en el Security Group de AWS el puerto por el que viajará el tráfico VPN. Sin este paso, el túnel no puede establecerse aunque la configuración sea perfecta.
>
> 1. Entra en **console.aws.amazon.com** → busca **`EC2`** → en el menú izquierdo, dentro de **Red y seguridad**, haz clic en **`Grupos de seguridad`**.
> 2. Marca tu grupo `sg-boochan-[tunombre]` y abajo pulsa la pestaña **`Reglas de entrada`** → **`Editar reglas de entrada`**.
> 3. Pulsa **`Agregar regla`** y rellena los campos:
>    - **Tipo:** `UDP personalizado`
>    - **Intervalo de puertos:** el número de la columna "Puerto"
>    - **Origen:** `Anywhere-IPv4` (equivale a `0.0.0.0/0`)
>    - **Descripción:** el texto de la columna "Nombre"
> 4. Pulsa **`Guardar reglas`**:
>
> | Nombre | Puerto | Protocolo | Para qué sirve ahora |
> | :--- | :--- | :--- | :--- |
> | WireGuard | 51820 | **UDP** | Canal cifrado del túnel VPN entre el aula y el servidor. |
>
> > [!note] 💡 En AWS no hay "Prioridad" ni "Permitir/Denegar"
> > Los **Security Groups de AWS no tienen número de prioridad ni acción de denegar**. Solo se añaden reglas que *permiten* tráfico; todo lo que no esté explícitamente permitido queda bloqueado por defecto.
>
> > [!warning] ⚠️ Este puerto es UDP, no TCP
> > Es el error más habitual en esta fase. WireGuard usa UDP porque necesita velocidad, no garantía de orden — igual que una videollamada. Si lo abres como TCP, la VPN no conectará aunque todo lo demás esté perfectamente configurado.

> [!example] Al terminar: cierra el RDP público (tarea pendiente desde la Fase 1)
> Una vez que el túnel VPN funcione y hayas comprobado el `ping 10.0.0.1`, ejecuta la acción de seguridad que quedó anunciada como pendiente al final de la Fase 1: aplica **Zero Trust** y cierra la puerta pública del servidor, dejando solo la privada, accesible únicamente desde dentro de la VPN.
>
> **En el Security Group de AWS** (EC2 → Grupos de seguridad → tu grupo → Editar reglas de entrada): localiza la regla `RDP` (puerto 3389, origen `0.0.0.0/0`) que abriste en la Fase 1 y aplica una de estas dos opciones:
>
> | Opción | Acción | Cuándo usarla |
> | :--- | :--- | :--- |
> | **A — Eliminar la regla** | ❌ Borra por completo la regla RDP (3389) del Security Group | La recomendada: vas a acceder **siempre** a través del túnel WireGuard, así que no necesitas ninguna vía pública de entrada. |
> | **B — Restringir el origen** | ✏️ Cambia el `Origen` de `0.0.0.0/0` a únicamente el rango de la red privada de la VPN | Si quieres dejar constancia explícita en el Security Group de que solo el rango `10.0.0.0/24` debería usar RDP, como documentación adicional. |
>
> > [!info] 💡 ¿Por qué la Opción A es la más correcta técnicamente?
> > El Security Group de AWS filtra el tráfico que llega **directamente a la tarjeta de red pública** de la instancia. Una vez que el túnel WireGuard está activo, el tráfico RDP hacia `10.0.0.1` viaja **encapsulado dentro** del tráfico UDP del puerto 51820 (que ya está permitido) — nunca llega al Security Group como una conexión TCP 3389 independiente, porque WireGuard lo descifra y entrega internamente a Windows tras pasar por la interfaz virtual `wg0`. Por eso, **eliminar la regla del 3389 no rompe nada**: tu acceso por la VPN sigue funcionando exactamente igual, y cierras una puerta que ya no es necesaria. A partir de este momento, tu conexión RDP habitual usará la IP del túnel:
> > ```
> > mstsc /v:10.0.0.1
> > ```

---

### 🛠️ Procedimiento Práctico (BoochanV3.1)

> [!example] Paso 1: Instalación y Generación de Llaves Criptográficas del Servidor
> Descarga e instala WireGuard en el servidor desde dentro de la sesión RDP.
>
> > [!info] 📚 Diccionario de Comandos: Para entender la sintaxis exacta de `wg.exe` y repasar otros comandos, consulta el [[Diccionario_Comandos_Sistema]].
>
> 1. Desde dentro de la instancia, abre un navegador y ve a `https://www.wireguard.com/install/`.
> 2. Descarga el instalador oficial para Windows (`.msi`) y ejecútalo con permisos de administrador, aceptando las opciones por defecto.
> 3. Al terminar, se abrirá la aplicación **WireGuard** con una lista vacía de túneles.
>
> Genera las llaves del servidor por línea de comandos, para mantener el paralelismo con BoochanV3 y facilitar la reproducibilidad:
> ```powershell
> # Crea la carpeta de configuración si no existe
> New-Item -ItemType Directory -Path "C:\WireGuard" -Force
> cd C:\WireGuard
>
> # Genera la llave privada y, a partir de ella, la llave pública
> wg genkey | Out-File -Encoding ascii privatekey
> Get-Content privatekey | wg pubkey | Out-File -Encoding ascii publickey
> ```
> Ahora **lee y anota** la llave pública del servidor. La necesitarás cuando configures el cliente en el Paso 3:
> ```powershell
> Get-Content C:\WireGuard\publickey
> ```
>
> > [!tip] 💡 ¿Qué hace este comando?
> > - **El Pipe (`|`):** Igual que en Linux, encadena comandos: la salida de uno entra directamente al siguiente.
> > - **`wg genkey` / `wg pubkey`:** Son los mismos binarios `wg` que en Linux — WireGuard incluye herramientas de línea de comandos idénticas en ambas plataformas.
> > - **Permisos del archivo:** A diferencia de Linux (`umask 077`), Windows no gestiona permisos de archivo con ese mecanismo. Por buena práctica, asegúrate de que la carpeta `C:\WireGuard` no está compartida ni es accesible por otros usuarios del sistema.

> [!example] Paso 2: Configuración del Túnel en el Servidor (`wg0.conf`)
> Crea el archivo de configuración del túnel:
>
> > [!info] 📚 Recurso: Para editar texto rápido en Windows Server usa el Bloc de notas (`notepad`) — no existe `nano` como en Linux; abre el fichero con `notepad C:\WireGuard\wg0.conf`.
>
> ```powershell
> notepad C:\WireGuard\wg0.conf
> ```
> Escribe este contenido. Sustituye `<CONTENIDO_DE_TU_PRIVATEKEY>` por el valor del archivo `privatekey`:
> ```ini
> [Interface]
> PrivateKey = <CONTENIDO_DE_TU_PRIVATEKEY>
> Address = 10.0.0.1/24
> ListenPort = 51820
>
> [Peer]
> PublicKey = <LLAVE_PÚBLICA_DEL_CLIENTE_AULA>
> AllowedIPs = 10.0.0.2/32
> ```
> Guarda y cierra Notepad. Deja el campo `<LLAVE_PÚBLICA_DEL_CLIENTE_AULA>` como está por ahora; lo completarás en el Paso 4 una vez que generes las llaves del cliente.
>
> Abre el puerto UDP en el Firewall de Windows para que el tráfico WireGuard pueda llegar al servidor (recuerda que el Security Group de AWS ya lo dejaste abierto al principio de esta fase, pero falta el firewall local):
> ```powershell
> New-NetFirewallRule -DisplayName "WireGuard VPN" -Direction Inbound -Protocol UDP -LocalPort 51820 -Action Allow
> ```
>
> > [!important] 💡 Dos firewalls, no uno
> > En AWS conviven dos capas de filtrado independientes: el **Security Group** (el firewall de la plataforma cloud, gestionado desde la consola web) y el **Firewall de Windows Defender** (el firewall local, dentro del propio sistema operativo). Un puerto abierto en el Security Group pero cerrado en el firewall de Windows sigue bloqueado, y viceversa. Ambas capas deben permitir el tráfico para que la conexión llegue.

> [!example] Paso 3: Instalación y Configuración del Cliente (PC del Aula)
> El túnel VPN necesita dos extremos configurados. Ahora le toca al **PC de tu aula**:
>
> **1. Instala la aplicación WireGuard en tu PC:**
> - **Windows:** Ve a `wireguard.com/install`, descarga el instalador `.exe` y ejecútalo.
> - **Mac:** Búscalo en la App Store buscando "WireGuard" o descárgalo desde `wireguard.com/install`.
>
> **2. Crea un nuevo túnel y obtén las llaves del cliente:**
> - Abre la aplicación WireGuard.
> - Haz clic en **"Agregar túnel"** → **"Crear nuevo túnel vacío"** (en Mac: icono `+`).
> - WireGuard genera automáticamente las llaves del cliente. Verás la **Clave Pública** del cliente en la parte superior del cuadro de configuración.
> - **Copia y anota esa Clave Pública**: la necesitarás en el servidor.
>
> **3. Completa el archivo de configuración del cliente** con este contenido:
> ```ini
> [Interface]
> PrivateKey = <SE_RELLENA_AUTOMÁTICAMENTE_por_WireGuard>
> Address = 10.0.0.2/32
> DNS = 10.0.0.1
>
> [Peer]
> PublicKey = <LLAVE_PÚBLICA_DEL_SERVIDOR_del_Paso_1>
> AllowedIPs = 10.0.0.0/24
> Endpoint = TU_IP_ELASTICA_AWS:51820
> PersistentKeepalive = 25
> ```
>
> > [!important] 💡 ¿Qué es `PersistentKeepalive`?
> > AWS cierra las conexiones que están inactivas. Este parámetro hace que el cliente envíe un pequeño "pulso" cada 25 segundos para mantener el túnel vivo aunque no haya tráfico real. Sin esta línea, la VPN se desconectaría sola a los pocos minutos.

> [!example] Paso 4: Intercambio de Llaves y Activación
> Vuelve a la sesión RDP del servidor y completa el archivo `wg0.conf` con la llave pública del cliente que anotaste en el Paso 3:
> ```powershell
> notepad C:\WireGuard\wg0.conf
> ```
> Sustituye `<LLAVE_PÚBLICA_DEL_CLIENTE_AULA>` por la llave pública real de tu PC. Guarda y cierra.
>
> > [!caution] ⚠️ Atención al Portapapeles (Copia-Pega)
> > Al borrar el texto de ejemplo `<LLAVE...>`, asegúrate de eliminar también los símbolos `<` y `>`. Un espacio extra, un salto de línea invisible o una letra comida arruinará la conexión VPN de forma silenciosa.
> >
> > **Antes de guardar**, verifica que la clave quedó bien pegada ejecutando:
> > ```powershell
> > Select-String -Path C:\WireGuard\wg0.conf -Pattern "PublicKey"
> > ```
> > La salida debe ser una sola línea limpia, sin espacios al principio ni al final, parecida a esto:
> > ```
> > PublicKey = aBcDeFgHiJkLmNoPqRsTuVwXyZ1234567890abcde=
> > ```
> > Si ves dos líneas, espacios raros o caracteres `<` o `>` sueltos, vuelve a editar el archivo antes de continuar.
>
> Ahora instala el túnel como servicio de Windows y actívalo:
> ```powershell
> # Instala el túnel como servicio (arranca automáticamente con el sistema, equivalente a systemctl enable)
> wireguard.exe /installtunnelservice C:\WireGuard\wg0.conf
> ```
>
> > [!tip] 💡 ¿Qué hace `/installtunnelservice`?
> > Es el equivalente Windows a `sudo wg-quick up wg0` seguido de `sudo systemctl enable wg-quick@wg0` en un solo comando: levanta el túnel inmediatamente **y** lo registra como servicio persistente que arrancará automáticamente en cada reinicio del servidor.
>
> **En el PC cliente (aula):** Activa el túnel haciendo clic en el botón **"Activar"** de la aplicación WireGuard.
>
> Verifica que el túnel está activo. En el servidor:
> ```powershell
> wg show
> ```
> Y desde el terminal de tu PC del aula:
> ```powershell
> # Si recibes respuestas, el túnel funciona correctamente
> ping 10.0.0.1
> ```
>
> > [!important] 🔒 VPN activa: momento de cerrar el servidor
> > El túnel funciona. Ahora es el momento de ejecutar la acción de seguridad que quedó pendiente desde la Fase 1: eliminar (o restringir) la regla RDP (3389) del Security Group de AWS. Encuentras el procedimiento exacto al principio de esta fase, en el apartado "Al terminar: cierra el RDP público".
> >
> > A partir de ese momento, **tu conexión RDP habitual usará la IP de la VPN**, no la IP pública/Elástica de AWS:
> > ```
> > mstsc /v:10.0.0.1
> > ```

---

### 🚩 Resolución de Problemas y Evaluación

> [!bug] Troubleshooting (¿No hay conexión?)
> | Problema | Causa Probable | Solución Sugerida |
> | :--- | :--- | :--- |
> | `wireguard.exe /installtunnelservice` falla con "Address already in use". | Ya hay otra interfaz VPN activa con esa IP. | Desinstala el servicio con `wireguard.exe /uninstalltunnelservice wg0` antes de volver a instalarlo. |
> | No hay ping entre `10.0.0.1` y `10.0.0.2`. | El puerto 51820 UDP está cerrado en el Security Group de AWS. | Abre el puerto 51820 **UDP** (no TCP) en el Security Group de AWS. |
> | WireGuard no conecta pero el puerto está abierto en AWS. | Las llaves públicas están intercambiadas incorrectamente, o el Firewall de Windows Defender bloquea el puerto 51820/UDP. | Verifica las llaves públicas cruzadas. Comprueba también `Get-NetFirewallRule -DisplayName "WireGuard VPN"` en el servidor. |
> | El cliente no encuentra el `Endpoint`. | Escribiste mal la IP Elástica de AWS, o la instancia no está encendida. | Comprueba la IP Elástica en la consola de EC2 y confirma que el servicio `WireGuardTunnel$wg0` está activo (`Get-Service`). |

> [!help] Preguntas Críticas (Autoevaluación)
> 1. ¿Por qué la llave privada **NUNCA** debe salir de tu servidor ni enviarse por correo?
> 2. ¿Qué diferencia hay entre instalar WireGuard "solo activado en la app" y hacerlo como Tunnel Service con `/installtunnelservice`?
> 3. ¿Para qué sirve el parámetro `AllowedIPs` en la configuración del Peer?
> 4. 🔬 **Reto práctico:** Con el túnel activo, ejecuta `wg show` en el servidor y localiza la línea `latest handshake`. ¿Hace cuántos segundos fue el último intercambio? Ahora desactiva el túnel desde tu PC del aula y vuelve a ejecutar el comando 30 segundos después. ¿Qué cambió en esa línea? ¿Qué te dice eso sobre el estado de la conexión?
> 5. 🔬 **Reto práctico:** Con el túnel WireGuard **desactivado** en tu PC, intenta conectarte al servidor por RDP usando la IP pública/Elástica de AWS (no la `10.0.0.1`). ¿Puedes entrar? ¿Por qué sí o por qué no? Razona tu respuesta mirando la regla RDP del Security Group tras el cambio de esta fase — ¿sigue existiendo la regla que lo permitía desde `0.0.0.0/0`?

---

> [!caution] 🛑 Auditoría y Seguridad (RA.05)
> Las llaves privadas son la **identidad** de tu servidor. Si un atacante las copia, podrá entrar en tu red privada como si fuera tú. **Validación:** El alumno debe demostrar el `ping 10.0.0.1` desde el cliente del aula y el `wg show` en el servidor mostrando el peer conectado.
