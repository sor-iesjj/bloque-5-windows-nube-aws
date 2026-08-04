## Fase 3 · Apartado 6 — 🛠️ Procedimiento práctico

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Conectividad VPN (WireGuard para Windows)**
> 🧭 Índice de la fase: [[Fase_3]]
>
> **📍 Cuándo se lee:** **Con la VM delante.** Aquí está el trabajo.

---

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

> [!example] 🔌 Paso 5 — EJERCICIO DE VERIFICACIÓN: qué hace de verdad tu VPN
> Tienes el túnel levantado y `wg show` dice que hay tráfico. Bien. Pero **¿sabes qué hace exactamente esa VPN, y sobre todo qué NO hace?** Vamos a comprobarlo con fuentes externas.
>
> > [!info] Recordatorio: por qué usamos APIs
> > Una **API** es una web hecha para que la consulte un programa: devuelve **datos limpios** en JSON en vez de una página. Un administrador las usa para **comprobar desde fuera lo que desde dentro no puede ver**. La teoría completa está en la práctica **B1.9b** del Bloque 1.
>
> **a) La red del túnel.** Tu túnel es **`10.0.0.0/24`**. Antes de mirar nada, escribe en tu entrada de apuntes cuántos clientes VPN caben en él. Ahora compruébalo:
> ```bash
> curl "https://networkcalc.com/api/ip/10.0.0.0/24"
> ```
>
> **b) Y ahora la pregunta buena: ¿por qué el cliente lleva `/32`?**
> Fíjate en tu configuración: el servidor tiene `Address = 10.0.0.1/24` pero el cliente tiene `Address = 10.0.0.2/32`. **No es un error.** Míralo:
> ```bash
> curl "https://networkcalc.com/api/ip/10.0.0.2/32"
> ```
> ```json
> "subnet_mask": "255.255.255.255",   "network_address": "10.0.0.2",
> "broadcast_address": "10.0.0.2",       "assignable_hosts": 0
> ```
>
> > [!success] 🤔 Léelo y explícalo en el vídeo
> > Una máscara `/32` significa **una sola dirección**: red, broadcast y host son la misma. **Cero hosts asignables.**
> > Traducido: *"yo soy exactamente esta IP y ninguna más"*. Por eso WireGuard usa `/32` en los clientes — cada uno declara **su** dirección exacta, y el servidor sabe sin ambigüedad a quién enviar cada paquete. Si pusieras `/24` en el cliente, estarías diciendo *"yo soy toda la red"*, y el enrutado se rompería.
>
> **c) El experimento que desmonta un mito.** Tu servidor está en la nube y **sí tiene IP pública propia**. Aun así, el resultado de abajo es el mismo: el túnel no cambia por dónde sales tú a Internet.
>
> 1. Con la VPN **desconectada**, en el cliente:
>    ```bash
>    curl "https://api.ipify.org?format=json"
>    ```
>    Anota la IP.
> 2. **Conecta el túnel** y comprueba que funciona: `ping 10.0.0.1`
> 3. Con la VPN **conectada**, repite exactamente el mismo comando.
>
> > [!danger] 🤯 Sale la MISMA IP. Y está bien.
> > Casi todo el mundo cree que "conectarse a una VPN" cambia tu IP pública — es lo que venden los anuncios de NordVPN y compañía. **Tu VPN no hace eso, y es a propósito.**
> >
> > Mira tu configuración: `AllowedIPs = 10.0.0.0/24`. Le has dicho al cliente: *"manda por el túnel **solo** lo que vaya a esa red"*. Todo lo demás —YouTube, Google, ipify— **sigue saliendo por tu conexión normal**. Eso se llama **split tunnel** (túnel partido).
> >
> > | | Qué manda por el túnel | Tu IP pública |
> > | :--- | :--- | :--- |
> > | **Split tunnel** (`AllowedIPs = 10.0.0.0/24`) ← el tuyo | Solo el tráfico hacia el servidor | **No cambia** |
> > | **Full tunnel** (`AllowedIPs = 0.0.0.0/0`) | **Todo** tu tráfico de Internet | Sí: sale la del servidor |
> >
> > **¿Y por qué split y no full?** Porque tu VPN existe para **llegar a tu servidor de forma segura**, no para ocultarte. Si mandaras todo el tráfico por el túnel, cargarías tu servidor con el YouTube de todos los clientes, y si el túnel cae te quedas sin Internet. Un administrador elige *split* salvo que tenga una razón concreta para lo contrario.
>
> > [!question] Lo que va a tu entrada de apuntes
> > 1. ¿Cuántos clientes VPN caben en tu túnel? ¿Coincidió con tu cálculo?
> > 2. ¿Por qué el cliente lleva `/32` y el servidor `/24`? Explícalo con lo que devolvió la API.
> > 3. Tu IP pública **no cambió** al conectar la VPN. **¿Por qué?** ¿Qué habría que cambiar en la configuración para que sí cambiara?
> > 4. Un compañero dice: *"si uso VPN nadie sabe lo que hago en Internet"*. Con lo que acabas de comprobar, **¿tiene razón?**

---

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_3.5_Fundamento_Teorico]] | [[Fase_3]] | [[Fase_3.7_Resolucion_Problemas]] |
