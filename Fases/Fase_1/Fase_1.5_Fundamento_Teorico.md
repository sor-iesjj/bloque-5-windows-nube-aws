## Fase 1 · Apartado 5 — 📚 Fundamento teórico

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Infraestructura Cloud (AWS EC2 — Windows Server 2025)**
> 🧭 Índice de la fase: [[Fase_1]]
>
> **📍 Cuándo se lee:** **Antes de teclear.** Los conceptos que necesitas.

---

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

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_1.4_Donde_Estamos]] | [[Fase_1]] | [[Fase_1.6_Procedimiento]] |
