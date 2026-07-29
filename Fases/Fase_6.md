## 💾 Fase 6: Almacenamiento con Cuotas (FSRM)

### Infraestructura de Servidores Cloud (AWS)

> **[Módulo: SOR — Sistemas Operativos en Red]**
> **[U.T. 5: Administración en Windows - Cuotas de Discos]**
> **[RA.04]** Gestiona los recursos compartidos del sistema determinando niveles de seguridad.
>
> **Profesor:** Pedro Navarro Miralles  
> **Correo:** p.navarromiralles2@edu.gva.es  
> **Centro:** IES Jorge Juan (ALICANTE)
>
> **⏱️ Tiempo estimado:** ~1,5 horas (teoría + práctica + retos + troubleshooting)  
> **Requisitos:** Instancia EC2 `WindowsServer` (`t3.large`) operativa | 2 GB disco libre en el volumen del sistema | RDP o PowerShell remoto

---

> [!important] 📹 Obligaciones de grabación (LÉEME — es igual en TODAS las fases)
> Esta práctica se **graba entera con OBS**, de principio a fin. No es un repaso al final: quiero ver **cómo lo haces tú**.
> 1. **Prepárate primero (sin grabar):** comprueba lo necesario, **léete el procedimiento entero** y **crea la entrada de apuntes de esta fase** en Obsidian: fichero `v3-1-fase-6-almacenamiento-con-cuotas-fsrm.md` dentro de `00_Apuntes/Trimestre_N/B5_Windows_Nube/`, con la estructura de la Fase 0.1 y **vacía**. Rellenarla es cosa tuya, después.
> 2. **Arranca OBS y PRESÉNTATE:** *"Hola, me llamo [Nombre], 2.º SMR, y en este vídeo voy a explicar la Fase 6 de Boochan V3.1 — Almacenamiento con Cuotas (FSRM)."* Y **muestra algo que demuestre que eres tú** (tu perfil de GitHub, tu Teams o tu correo `@alu.edu.gva.es`). Di qué vas a hacer.
> 3. **Graba TODO el procedimiento**, explicando cada paso en voz alta mientras lo haces.
> 4. **Timestamps SIEMPRE** en la descripción: `00:00 Presentación` + uno por cada paso.
> 5. **Al terminar:** nombra el vídeo `V3.1 · Fase 6 — Almacenamiento con Cuotas (FSRM)`, súbelo a tu playlist de YouTube **`B5_Windows_Nube`** (No listado) y **copia su enlace**.
> 6. **~8-10 min.** Esta fase es más larga que las de prerrequisitos: ve al grano, pero no te saltes pasos. Si se te va mucho, **pártela en dos vídeos** y ponlos los dos en la entrada.
> 7. **El enlace del vídeo va DENTRO de tu entrada de apuntes**, en el apartado `Enlace al vídeo explicativo`. Ahí, no en un papel.
> 8. **La entrega va por la TAREA de Teams.** Abriré una tarea que cubrirá **esta fase y otras**; te llegará notificación con fecha límite.

---

### 🎯 ¿Dónde Estamos?

> [!info] Vienes de Fase 5
> Tienes usuarios y grupos del dominio completamente estructurados: `user1` en el grupo `Policia`, `user2` en el grupo `Bomberos`, organizados en sus OUs correspondientes dentro de `BOOCHAN.SPACE`. El servidor reconoce a ambos como ciudadanos de pleno derecho de Active Directory. Sin embargo, no existe ningún límite sobre cuánto espacio pueden usar en el disco del servidor.

> [!warning] El Problema
> Sin cuotas de disco, un usuario malintencionado (o simplemente desprevenido) podría llenar el volumen del sistema completamente. Cuando un disco de Windows se llena, el sistema se vuelve inestable: los registros de eventos dejan de escribirse, los servicios pueden fallar al no poder escribir archivos temporales, y la administración remota se complica. En empresas, esto es una "Denegación de Servicio" (DoS) involuntaria pero devastadora — y en una instancia EC2 de AWS, además, un disco lleno puede impedir la propia conexión RDP de auxilio.

> [!success] Objetivo de esta Fase
> Instalar el rol **FSRM (File Server Resource Manager)** y aplicar **cuotas físicas** de 5 GB sobre dos carpetas del proyecto (`Prueba1` y `Prueba3`), directamente sobre el sistema de archivos NTFS real del disco del sistema operativo — sin crear ningún disco virtual adicional ni depender de un volumen EBS de datos separado en AWS. Si una carpeta se llena, solo ese departamento se ve afectado; el resto del servidor sigue funcionando con normalidad.

> [!tip] Hoja de Ruta
> 1. Instalar el rol `FS-Resource-Manager` (File Server Resource Manager)
> 2. Crear las carpetas del proyecto: `C:\ShareData\Prueba1` y `C:\ShareData\Prueba3`
> 3. Crear una **plantilla de cuota** de 5 GB con `New-FsrmQuotaTemplate`
> 4. Aplicar esa plantilla a ambas carpetas con `New-FsrmQuota`
> 5. Verificar con `Get-FsrmQuota` que el límite está activo
> 6. Intentar llenar una carpeta para comprobar que la barrera es infranqueable, y observar cómo se libera el espacio al borrar
>
> **Resultado Final:** Dos carpetas del proyecto, `C:\ShareData\Prueba1` y `C:\ShareData\Prueba3`, cada una con un límite físico de 5 GB gestionado por FSRM. El servidor está protegido contra el llenado descontrolado de disco.
> **Siguiente:** Fase 7 (Seguridad Avanzada) — aplicarás permisos NTFS y Access-Based Enumeration para controlar quién ve y accede a cada carpeta.

---

### 📚 Fundamento Teórico

> [!abstract] 1. Evitando el Colapso por Llenado
> La gestión de cuotas de disco es vital para evitar la "Denegación de Servicio" por llenado de disco. Si un usuario (o un atacante) llena todo el volumen del sistema, los registros de eventos no podrán escribirse y los servicios del servidor pueden empezar a fallar de forma impredecible.

> [!tip] 2. Cuota Directa sobre NTFS (sin discos virtuales)
> En **BoochanV3** (Ubuntu + Samba sobre AWS), las cuotas se implementaban con **Loop Devices**: archivos `.img` que actuaban como discos duros independientes formateados en ext4, montados como si fueran particiones reales. Era un rodeo necesario porque el ext4 clásico de Linux no siempre ofrece cuotas de carpeta sencillas de administrar.
> En **Windows Server con FSRM**, ese rodeo desaparece: FSRM aplica el límite **directamente sobre la carpeta** del volumen NTFS real (el mismo disco EBS raíz de la instancia), sin necesidad de crear ningún "disco falso dentro del disco" ni de añadir un volumen de datos adicional en AWS. Sigue siendo una barrera física e infranqueable —NTFS rechaza la escritura en cuanto se alcanza el límite—, pero la implementación es mucho más simple: no hay que crear archivos `.img`, no hay que formatearlos, y no hay ninguna entrada equivalente al `fstab` que pueda romper el arranque del servidor si te equivocas.

### 📖 Diccionario de Conceptos Clave

> [!quote] Almacenamiento Avanzado
> - **FSRM (File Server Resource Manager):** El rol de Windows Server que gestiona cuotas, filtrado de archivos e informes de almacenamiento sobre carpetas NTFS.
> - **Plantilla de Cuota (Quota Template):** Una configuración reutilizable (tamaño, tipo de límite, umbrales de aviso) que se aplica a una o varias carpetas.
> - **Cuota Estricta (Hard Quota):** El límite es infranqueable — al alcanzarlo, Windows rechaza nuevas escrituras. Es el equivalente funcional al Loop Device de Samba.
> - **Cuota Flexible (Soft Quota):** Solo genera avisos, no bloquea la escritura. No la usaremos en este proyecto porque necesitamos una barrera real.
> - **Umbral (Threshold):** Un porcentaje de la cuota (por ejemplo, 85%) en el que FSRM puede disparar una notificación o un evento antes de llegar al límite.

---

### 🔓 Apertura de Puertos (Security Group de AWS)

> [!info] ℹ️ Sin cambios en el Security Group en esta fase
> Esta fase trabaja íntegramente en local sobre el servidor — instalación de un rol y gestión de carpetas NTFS en el disco de la propia instancia `WindowsServer`. No requiere añadir ninguna regla nueva en el Security Group de AWS: FSRM no expone ningún puerto de red nuevo salvo que actives las notificaciones por correo (SMTP), algo que no usaremos en este proyecto de laboratorio.

---

### 🛠️ Procedimiento Práctico (Creación del Almacenamiento con Cuotas)

> [!example] Paso 1: Instalación del Rol FSRM
> Instalamos el rol de gestión de recursos del servidor de archivos, que incluye las herramientas de administración y el módulo de PowerShell `FileServerResourceManager`:
> ```powershell
> Install-WindowsFeature FS-Resource-Manager -IncludeManagementTools
> ```
> Comprueba que el servicio quedó instalado y en marcha:
> ```powershell
> Get-Service SrmSvc
> ```
> Debe aparecer en estado `Running`. Si no lo está, arráncalo con:
> ```powershell
> Start-Service SrmSvc
> ```
>
> > [!tip] 💡 ¿Qué instala exactamente `-IncludeManagementTools`?
> > Además del servicio en sí (`SrmSvc`), instala la consola gráfica de FSRM (accesible desde el Administrador del Servidor) y el módulo de PowerShell `FileServerResourceManager`, que es el que usaremos con los cmdlets `*-Fsrm*`.

> [!example] Paso 2: Creación de las Carpetas del Proyecto
> Antes de aplicar las cuotas, necesitamos las carpetas donde vivirán los archivos de cada departamento. Usamos siempre la unidad `C:` (el disco EBS raíz de la instancia) — no se crea ni se monta ningún volumen de datos adicional de AWS:
> ```powershell
> New-Item -Path "C:\ShareData\Prueba1" -ItemType Directory -Force
> New-Item -Path "C:\ShareData\Prueba3" -ItemType Directory -Force
> ```
>
> > [!tip] 💡 ¿Qué hace `-Force`?
> > El parámetro `-Force` crea también las carpetas intermedias que no existan (en este caso, `C:\ShareData`) y no muestra error si la carpeta ya existe. Es el equivalente funcional del `-p` de `mkdir` en Linux.

> [!example] Paso 3: Creación de la Plantilla de Cuota
> Creamos una plantilla reutilizable con un límite estricto de 5 GB, con un umbral de aviso al 85% para que quede registrado en el Visor de Eventos antes de llegar al límite:
>
> > [!info] 📚 Diccionario de Comandos: Para repasar la sintaxis de los cmdlets `*-Fsrm*`, consulta el [[Diccionario_Comandos_Sistema]].
>
> ```powershell
> # Umbral de aviso al 85% de la cuota (solo registra un evento, no bloquea nada)
> $umbral85 = New-FsrmQuotaThreshold -Percentage 85 -Action (
>     New-FsrmAction -Type Event -EventType Warning `
>         -Body "La carpeta ha superado el 85% de su cuota de 5 GB."
> )
>
> # Plantilla de cuota estricta de 5 GB
> New-FsrmQuotaTemplate -Name "Limite5GB-Estricto" `
>     -Description "Cuota física de 5 GB, infranqueable, para carpetas del proyecto Boochan" `
>     -Size 5GB -Threshold $umbral85
> ```
>
> > [!tip] 💡 ¿Qué hace este comando?
> > - **`-Size 5GB`:** El límite máximo de la plantilla. PowerShell entiende directamente el sufijo `GB` sin necesidad de calcular bytes a mano (a diferencia del `dd bs=1M count=5120` de Linux).
> > - **Ausencia de `-SoftLimit`:** Al no indicar `-SoftLimit`, la plantilla es de tipo **estricto (hard)** por defecto: Windows bloqueará físicamente cualquier escritura que supere los 5 GB.
> > - **`-Threshold`:** Los avisos intermedios son opcionales, pero muy recomendables en producción — permiten actuar antes de que el límite bloquee al usuario.

> [!example] Paso 4: Aplicar la Cuota a las Carpetas
> Aplicamos la plantilla a las dos carpetas del proyecto. A diferencia del `fstab` de Linux, esto **no requiere ningún reinicio ni comando de montaje** — la cuota se activa de inmediato:
> ```powershell
> New-FsrmQuota -Path "C:\ShareData\Prueba1" -Template "Limite5GB-Estricto"
> New-FsrmQuota -Path "C:\ShareData\Prueba3" -Template "Limite5GB-Estricto"
> ```
>
> > [!caution] ⚠️ La cuota se aplica a la carpeta, no a un volumen nuevo
> > A diferencia de un Loop Device, aquí **no se ha creado ningún disco nuevo** ni se ha añadido ningún volumen de datos adicional en AWS. `C:\ShareData\Prueba1` sigue siendo una carpeta normal dentro del mismo volumen `C:` que el resto del sistema operativo. La diferencia es que FSRM ahora vigila cuánto se escribe *dentro de esa carpeta concreta* y bloquea la escritura al llegar a 5 GB, sin afectar al resto de `C:`.

> [!example] Paso 5: Verificación de la Cuota Activa
> Comprobamos que ambas carpetas tienen la cuota aplicada correctamente:
> ```powershell
> Get-FsrmQuota -Path "C:\ShareData\Prueba1"
> Get-FsrmQuota -Path "C:\ShareData\Prueba3"
> ```
> Debes ver el campo `Size` en 5368709120 bytes (5 GB) y `Usage` cercano a 0 en ambas carpetas recién creadas.

---

### 🚩 Resolución de Problemas y Evaluación

> [!bug] Troubleshooting (¿La cuota no aparece o no funciona?)
> | Problema | Causa Probable | Solución Sugerida |
> | :--- | :--- | :--- |
> | `Get-FsrmQuota` da error "The specified path does not exist". | Se aplicó la cuota antes de crear la carpeta, o hay un error tipográfico en la ruta. | Verifica con `Test-Path "C:\ShareData\Prueba1"` y repite el `New-FsrmQuota` con la ruta correcta. |
> | El cmdlet `New-FsrmQuotaTemplate` no se reconoce. | El rol FSRM no está instalado o el módulo no se ha importado. | Ejecuta `Get-WindowsFeature FS-Resource-Manager` para confirmar la instalación e `Import-Module FileServerResourceManager`. |
> | Un usuario sigue escribiendo por encima de 5 GB. | Se aplicó una plantilla con `-SoftLimit`, o la carpeta tiene otra cuota distinta aplicada encima. | Ejecuta `Get-FsrmQuota -Path "C:\ShareData\Prueba1" \| Select SoftLimit, Size` y confirma que `SoftLimit` es `False`. |
> | `New-FsrmQuota` da error "A quota already exists for this path". | La cuota ya se aplicó en un intento anterior. | Ejecuta `Set-FsrmQuota -Path "C:\ShareData\Prueba1" -Template "Limite5GB-Estricto"` para actualizarla, o `Remove-FsrmQuota` y vuelve a crearla. |
> | El disco `C:` de la instancia se queda sin espacio antes de llegar a probar las cuotas. | El tamaño de disco EBS por defecto de la instancia `t3.large` es limitado y ya tiene el sistema operativo instalado. | Comprueba el espacio libre con `Get-Volume -DriveLetter C` antes de crear archivos de prueba grandes; nunca crees archivos de más de unos pocos GB fuera de las carpetas con cuota. |

> [!help] Preguntas Críticas (Autoevaluación)
> 1. ¿Por qué FSRM no necesita un equivalente al `fstab` de Linux para que la cuota sobreviva a un reinicio del servidor?
> 2. ¿Qué diferencia hay entre una cuota **estricta (hard)** y una **flexible (soft)** en FSRM? ¿Cuál usamos en este proyecto y por qué?
> 3. En Samba, si olvidabas la palabra `loop` en el `fstab`, el servidor podía entrar en pánico al arrancar. ¿Por qué en FSRM no existe un riesgo equivalente?
> 4. 🔬 **Reto práctico:** Intenta llenar la carpeta `C:\ShareData\Prueba1` creando un archivo de 6 GB (mayor que la cuota de 5 GB): `fsutil file createnew C:\ShareData\Prueba1\lleno.dat 6442450944`. ¿Qué mensaje de error aparece? Borra el archivo con `Remove-Item C:\ShareData\Prueba1\lleno.dat` y comprueba con `Get-FsrmQuota -Path "C:\ShareData\Prueba1"` que el campo `Usage` vuelve a estar cerca de 0.
> 5. 🔬 **Reto práctico:** Ejecuta `Get-Volume -DriveLetter C` y anota el tamaño total del volumen `C:`. Compáralo con los 5 GB de cada cuota de carpeta. ¿Qué le ocurriría al volumen `C:` completo (y por tanto al sistema operativo) si `Prueba1` y `Prueba3` no tuvieran cuota y un usuario las llenara sin límite?

---

> [!caution] 🛑 Auditoría de Persistencia (RA.04)
> **Diferencia crítica con Samba:** en BoochanV3, si el alumno olvidaba la opción `loop` en `/etc/fstab`, el servidor podía no arrancar tras un `reboot` de la instancia. En FSRM, la cuota se almacena en la base de datos interna del propio servicio `SrmSvc`, no en un archivo de configuración editable por el alumno — por lo que ese riesgo concreto de "romper el arranque" desaparece.

> [!success] 🏁 Punto de Control (Antes de seguir)
> - [ ] ¿Aparecen ambas cuotas al ejecutar `Get-FsrmQuota -Path "C:\ShareData\*"`?
> - [ ] Reinicia el servidor para comprobar que las cuotas siguen activas sin ninguna acción manual:
>   ```powershell
>   Restart-Computer
>   ```
>   > [!caution] ⚠️ La sesión RDP/remota se cortará al instante — es normal
>   > La instancia se está reiniciando en AWS. Espera 1-2 minutos y vuelve a conectarte (RDP o PowerShell remoto, según hayas usado). Cuando el servidor responda de nuevo, ejecuta `Get-FsrmQuota -Path "C:\ShareData\Prueba1"` y confirma que la cuota de 5 GB sigue aplicada sin que hayas tenido que volver a crearla.

---

### ✅ Entregables y cierre

> [!abstract] Qué tienes que tener hecho al acabar esta fase
> | Entregable | Dónde vive | Qué debe contener |
> | :--- | :--- | :--- |
> | **Entrada de apuntes** | `00_Apuntes/Trimestre_N/B5_Windows_Nube/v3-1-fase-6-almacenamiento-con-cuotas-fsrm.md` | Estructura completa + **respuestas a las Preguntas Críticas y al 🔬 Reto** + **enlace del vídeo** |
> | **Vídeo** | Playlist `B5_Windows_Nube` (No listado) | Nombrado `V3.1 · Fase 6 — Almacenamiento con Cuotas (FSRM)`, con presentación, identidad y timestamps |
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
