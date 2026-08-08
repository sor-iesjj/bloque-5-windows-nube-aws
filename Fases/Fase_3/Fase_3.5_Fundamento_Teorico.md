## Fase 3 · Apartado 5 — 📚 Fundamento teórico

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Conectividad VPN (WireGuard para Windows)**
> 🧭 Índice de la fase: [[Fase_3]]
>
> **📍 Cuándo se lee:** **Antes de teclear.** Los conceptos que necesitas.

---

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

> [!example] 🎬 Antes de empezar (todavía SIN grabar, y luego arranca)
> Ya conoces el método desde los prerrequisitos, así que va solo el recordatorio:
> 1. **Crea la entrada de apuntes** de esta fase (`b5-aws-3-conectividad-vpn-wireguard-para-windows.md`) con su estructura, vacía.
> 2. **Léete los 5 pasos** del procedimiento enteros, para no atascarte a mitad del vídeo.
> 3. Ten **OBS** listo y comprueba **pantalla y micrófono**.
>
> Cuando lo tengas: **arranca la grabación, preséntate y muestra tu identidad**. A partir de ahí, **todo queda grabado** — incluido cualquier paso previo de preparación que venga a continuación.

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

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_3.4_Donde_Estamos]] | [[Fase_3]] | [[Fase_3.6_Procedimiento]] |
