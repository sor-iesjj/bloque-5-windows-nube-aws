## 🕵️ Fase 7: Seguridad Avanzada (Permisos NTFS y Access-Based Enumeration)

### Infraestructura de Servidores Cloud (AWS)

> **[Módulo: SOR — Sistemas Operativos en Red]**
> **[U.T. 5 y 8: Gestión avanzada de permisos / Compartición SMB]**
> **[RA.04]** Gestiona los recursos compartidos del sistema interpretando especificaciones y determinando niveles de seguridad.
>
> **Profesor:** Pedro Navarro Miralles  
> **Correo:** p.navarromiralles2@edu.gva.es  
> **Centro:** IES Jorge Juan (ALICANTE)
>
> **⏱️ Tiempo estimado:** ~1,5 horas (teoría + práctica + retos + troubleshooting)  
> **Requisitos:** Carpetas con cuota FSRM operativas | Rol de Servidor de Archivos disponible | Instancia EC2 `WindowsServer` (`t3.large`)

---

> [!important] 📹 Obligaciones de grabación (LÉEME — es igual en TODAS las fases)
> Esta práctica se **graba entera con OBS**, de principio a fin. No es un repaso al final: quiero ver **cómo lo haces tú**.
> 1. **Prepárate primero (sin grabar):** comprueba lo necesario, **léete el procedimiento entero** y **crea la entrada de apuntes de esta fase** en Obsidian: fichero `v3-1-fase-7-seguridad-avanzada-permisos-ntfs-y-acces.md` dentro de `00_Apuntes/Trimestre_N/B5_Windows_Nube/`, con la estructura de la Fase 0.1 y **vacía**. Rellenarla es cosa tuya, después.
> 2. **Arranca OBS y PRESÉNTATE:** *"Hola, me llamo [Nombre], 2.º SMR, y en este vídeo voy a explicar la Fase 7 de Boochan V3.1 — Seguridad Avanzada (Permisos NTFS y Access-Based Enumeration)."* Y **muestra algo que demuestre que eres tú** (tu perfil de GitHub, tu Teams o tu correo `@alu.edu.gva.es`). Di qué vas a hacer.
> 3. **Graba TODO el procedimiento**, explicando cada paso en voz alta mientras lo haces.
> 4. **Timestamps SIEMPRE** en la descripción: `00:00 Presentación` + uno por cada paso.
> 5. **Al terminar:** nombra el vídeo `V3.1 · Fase 7 — Seguridad Avanzada (Permisos NTFS y Access-Based Enumeration)`, súbelo a tu playlist de YouTube **`B5_Windows_Nube`** (No listado) y **copia su enlace**.
> 6. **~8-10 min.** Esta fase es más larga que las de prerrequisitos: ve al grano, pero no te saltes pasos. Si se te va mucho, **pártela en dos vídeos** y ponlos los dos en la entrada.
> 7. **El enlace del vídeo va DENTRO de tu entrada de apuntes**, en el apartado `Enlace al vídeo explicativo`. Ahí, no en un papel.
> 8. **La entrega va por la TAREA de Teams.** Abriré una tarea que cubrirá **esta fase y otras**; te llegará notificación con fecha límite.

---

### 🎯 ¿Dónde Estamos?

> [!info] Vienes de Fase 6
> Tienes dos carpetas con cuotas físicas de 5 GB gestionadas por FSRM en `C:\ShareData\Prueba1` y `C:\ShareData\Prueba3`. Los usuarios pueden crear archivos dentro de ellas, pero cualquiera que llegue a la carpeta puede verla, aunque no tenga permiso real para entrar. No hay ningún control granular por grupo todavía.

> [!warning] El Problema
> Con solo los permisos por defecto de una carpeta nueva (heredados del disco), no puedes crear un modelo seguro para múltiples departamentos. Si necesitas que el grupo `Policia` tenga acceso total a `Prueba3` pero `Bomberos` no vea ni que existe, no basta con compartir la carpeta por SMB "a secas". Necesitas dos capas: (1) una física (permisos NTFS) que controle quién realmente accede, y (2) una visual (Access-Based Enumeration) que oculte la carpeta a quien no tiene permiso.

> [!success] Objetivo de esta Fase
> Implementar **dos capas de seguridad profesional**: los **permisos NTFS** (listas de control de acceso granulares, nativas del sistema de archivos) que otorgan permisos reales a grupos específicos, y **Access-Based Enumeration (ABE)** —nativa del recurso compartido SMB de Windows— que oculta visualmente las carpetas a las que el usuario no tiene acceso. El resultado: `user2` (Bomberos) simplemente no ve la carpeta `Prueba3` en el Explorador de Archivos.

> [!tip] Hoja de Ruta
> 1. Romper la herencia de permisos en `C:\ShareData\Prueba3` y dejar acceso exclusivo al grupo `Policia`
> 2. Conceder al grupo `Policia` el permiso NTFS `Modify` (Modificar) con herencia hacia subcarpetas y archivos futuros
> 3. Crear los recursos compartidos SMB `Prueba1` (sin ABE) y `Prueba3` (con ABE activado) con `New-SmbShare`
> 4. Verificar que `FolderEnumerationMode` está en `AccessBased` para `Prueba3`
> 5. Desde un cliente Windows: iniciar sesión como `user1` (Policia) y verificar que ve `Prueba3`
> 6. Desde un cliente Windows: iniciar sesión como `user2` (Bomberos) y verificar que NO ve `Prueba3`
>
> **Resultado Final:** Carpeta `Prueba3` completamente protegida — invisible para quienes no tienen permiso, accesible solo para el grupo `Policia`. Los archivos nuevos heredan automáticamente los permisos del grupo.
> **Siguiente:** Fase 8 (Integración del Cliente) — unirás un equipo Windows 11 al dominio y probarás el acceso desde el aula.

---

### 📚 Fundamento Teórico

> [!abstract] 1. Las Dos Capas de la Seguridad Profesional
> No basta con poner una contraseña. Para que un servidor de archivos sea profesional, usamos dos capas de protección:
> 1. **Capa Física (Permisos NTFS):** Son los permisos granulares del sistema de archivos de Windows. Dicen: *"Tú puedes entrar y leer esto"*. Es el candado real del archivo.
> 2. **Capa Visual (Access-Based Enumeration):** Es la "Capa de Invisibilidad" del recurso compartido SMB. Si un usuario no tiene permiso físico (NTFS), Windows simplemente **le oculta la carpeta** al listar el recurso compartido. Si no puedes entrar, no hace falta que sepas que existe.

> [!important] 2. La implementación original, no la imitación
> **BoochanV3** (Ubuntu + Samba sobre AWS) usaba `setfacl` para las ACLs y activaba `access based share enum = yes` en `smb.conf` para imitar el comportamiento de ABE de Windows — porque Samba, precisamente, se diseñó desde el principio para *parecerse* a un servidor Windows y ser compatible con sus clientes. Aquí, sobre Windows Server 2025, no hay ninguna imitación: **Access-Based Enumeration es una función nativa de Windows Server desde hace más de una década**, y la vas a activar con un simple parámetro del cmdlet que crea el propio recurso compartido (`New-SmbShare -FolderEnumerationMode AccessBased`), sin tocar ningún archivo de configuración de texto ni reiniciar ningún servicio.

> [!tip] 3. Herencia de Permisos NTFS
> Igual que la herencia de ACLs en Linux (`setfacl -d`), en NTFS los permisos se pueden marcar para que se apliquen automáticamente a "Esta carpeta, subcarpetas y archivos". Así el administrador no tiene que asignar permisos cada vez que alguien crea un archivo nuevo dentro de la carpeta protegida.

### 📖 Diccionario de Conceptos Clave

> [!quote] Seguridad y Privacidad
> - **ACL NTFS (Access Control List):** Lista detallada de permisos para múltiples usuarios y grupos sobre un mismo archivo o carpeta.
> - **ABE (Access-Based Enumeration):** Función nativa del recurso compartido SMB de Windows que filtra la visibilidad de carpetas según los permisos NTFS del usuario que consulta.
> - **Herencia:** Característica de NTFS que hace que los archivos y subcarpetas nuevos copien automáticamente los permisos de la carpeta superior.
> - **`icacls` / `Set-Acl`:** Las herramientas nativas de Windows para gestionar permisos NTFS desde la línea de comandos, equivalentes a `setfacl` en Linux.
> - **`New-SmbShare`:** El cmdlet que publica una carpeta como recurso de red SMB, con el parámetro `-FolderEnumerationMode` para activar o no ABE.

### 🔓 Apertura de Puertos (Security Group de AWS)

> [!info] ℹ️ Sin cambios en el Security Group en esta fase
> Esta fase trabaja íntegramente dentro del servidor: permisos NTFS y publicación de recursos compartidos SMB. No requiere añadir ninguna regla nueva en el Security Group de AWS: el puerto SMB (445), Kerberos y LDAP ya están operativos desde la Fase 4. El Firewall de Windows Defender abre automáticamente las reglas de "Compartir archivos e impresoras" al crear el primer recurso compartido con `New-SmbShare`; si no fuera así, actívalas con `Set-NetFirewallRule -DisplayGroup "File and Printer Sharing" -Enabled True`.

---

### 🛠️ Procedimiento Práctico (Permisos y Visibilidad)

> [!example] Paso 1: Configuración de los Candados (Permisos NTFS)
> Primero rompemos la herencia de permisos de `Prueba3` respecto a su carpeta padre (para que no arrastre permisos genéricos), y a continuación concedemos acceso explícito solo al grupo `Policia`:
>
> > [!info] 📚 Diccionario de Comandos: Para repasar los operadores exactos de `icacls`, consulta el [[Diccionario_Comandos_Sistema]].
>
> ```powershell
> # 1. Rompemos la herencia y conservamos los permisos existentes como base (los limpiaremos después)
> icacls "C:\ShareData\Prueba3" /inheritance:r
>
> # 2. Concedemos control total al grupo Administradores y a SYSTEM (imprescindible para gestión del servidor)
> icacls "C:\ShareData\Prueba3" /grant "Administradores:(OI)(CI)F" "SYSTEM:(OI)(CI)F"
>
> # 3. Concedemos permiso de Modificar al grupo Policia, con herencia hacia subcarpetas y archivos futuros
> icacls "C:\ShareData\Prueba3" /grant "BOOCHAN\Policia:(OI)(CI)M"
> ```
>
> > [!tip] 💡 ¿Qué hace este comando?
> > - **`/inheritance:r`:** Elimina (r = remove) los permisos heredados de la carpeta padre. Sin este paso, cualquier permiso genérico del disco `C:` (como "Usuarios autenticados") seguiría dando acceso a todo el mundo, y la protección sería una ilusión.
> > - **`/grant "Grupo:(OI)(CI)M"`:** Concede el permiso **M (Modify / Modificar)** al grupo indicado. `(OI)` significa *Object Inherit* (los archivos nuevos heredan el permiso) y `(CI)` significa *Container Inherit* (las subcarpetas nuevas también lo heredan) — es el equivalente exacto al `-d` (default/herencia) de `setfacl` en Linux.
> > - **Carpeta `Prueba1`** no necesita este tratamiento: se deja con los permisos por defecto (accesible por `Usuarios del dominio`), igual que en BoochanV3.

> [!example] Paso 2: Publicación de las Carpetas como Recursos SMB
> A diferencia de Samba, aquí no editamos ningún archivo de texto (`smb.conf`): el propio cmdlet que crea el recurso compartido incluye el parámetro que activa ABE.
>
> Antes de crear los recursos, comprueba que no existan ya de un intento anterior de esta fase:
> ```powershell
> Get-SmbShare | Where-Object Name -in "Prueba1", "Prueba3"
> ```
> Si no devuelve nada, continúa. Si ya existen, edítalos con `Set-SmbShare` en lugar de crearlos de nuevo.
>
> Creamos ambos recursos compartidos:
> ```powershell
> # Prueba1: recurso general, sin ABE — visible para todo el dominio
> New-SmbShare -Name "Prueba1" -Path "C:\ShareData\Prueba1" `
>     -FullAccess "BOOCHAN\Domain Users" `
>     -FolderEnumerationMode Unrestricted
>
> # Prueba3: recurso protegido, con ABE activado
> New-SmbShare -Name "Prueba3" -Path "C:\ShareData\Prueba3" `
>     -FullAccess "BOOCHAN\Policia" `
>     -FolderEnumerationMode AccessBased
> ```
>
> > [!tip] 💡 ¿Qué diferencia hay entre `Prueba1` y `Prueba3`?
> > - **`Prueba1`:** Es una carpeta de acceso general para todos los usuarios del dominio. `-FolderEnumerationMode Unrestricted` es el valor por defecto: todo el mundo que tiene acceso al recurso lo ve listado.
> > - **`Prueba3`:** Es la carpeta protegida. `-FolderEnumerationMode AccessBased` activa ABE: Windows comprueba, para cada usuario que consulta el recurso, si tiene permiso NTFS real sobre la carpeta — y si no lo tiene, la oculta directamente del listado, en lugar de mostrarla y luego denegar el acceso al entrar.
>
> > [!important] 💡 Dos capas, dos permisos distintos
> > Observa que hemos dado `-FullAccess` a nivel de **recurso compartido (share)** solo al grupo que corresponde, y además tenemos el permiso **NTFS** aplicado en el Paso 1. En un entorno real es habitual dejar el permiso de share más abierto (p. ej. "Usuarios autenticados: Control total") y confiar toda la granularidad al permiso NTFS, que es más flexible. En este proyecto usamos ambas capas alineadas para que el ejemplo sea más claro pedagógicamente.

> [!example] Paso 3: Comprobar que No Hace Falta Reiniciar Nada
> A diferencia de Samba, donde cada cambio en `smb.conf` exige `sudo systemctl restart samba-ad-dc`, en Windows Server los cambios de `New-SmbShare` / `Set-SmbShare` **se aplican de inmediato**, sin reiniciar ningún servicio. Compruébalo:
> ```powershell
> Get-SmbShare -Name "Prueba3" | Select-Object Name, Path, FolderEnumerationMode
> ```
> Debe mostrar `FolderEnumerationMode : AccessBased`.

---

### 🚩 Resolución de Problemas y Evaluación

> [!bug] Troubleshooting (¿La seguridad falla?)
> | Problema | Causa Probable | Solución Sugerida |
> | :--- | :--- | :--- |
> | `user2` sigue viendo `Prueba3` en el Explorador. | El permiso NTFS del grupo `Bomberos` (o de un grupo genérico como "Usuarios autenticados") no se eliminó antes de aplicar ABE. | Ejecuta `icacls "C:\ShareData\Prueba3"` y revisa que solo aparezcan `Policia`, `Administradores` y `SYSTEM`. Si aparece otro grupo con acceso, quítalo con `icacls ... /remove:g "Grupo"`. |
> | `user1` ve la carpeta pero no puede entrar ni crear archivos. | El permiso a nivel de recurso compartido (share) no incluye a `Policia`, o el permiso NTFS quedó en solo lectura. | Revisa `Get-SmbShareAccess "Prueba3"` y confirma `FullAccess` para `BOOCHAN\Policia`; revisa también que el permiso NTFS sea `M` (Modify), no `R` (Read). |
> | `New-SmbShare` da error "The parameter is incorrect" con `-FolderEnumerationMode`. | Error tipográfico en el valor del parámetro (debe ser exactamente `AccessBased` o `Unrestricted`). | Revisa la sintaxis exacta con `Get-Help New-SmbShare -Parameter FolderEnumerationMode`. |
> | `icacls /inheritance:r` deja la carpeta sin ningún acceso, ni siquiera para el Administrador. | Se ejecutó `/inheritance:r` sin encadenar inmediatamente el `/grant` para Administradores y SYSTEM. | Vuelve a ejecutar el Paso 1 completo, en el orden indicado, empezando por conceder acceso a Administradores/SYSTEM antes de tocar nada más. |
> | No se puede acceder al recurso `\\WindowsServer.BOOCHAN.SPACE\Prueba3` desde un cliente externo a la VPC de AWS. | El Security Group bloquea el puerto 445 (SMB) para el origen concreto, o el cliente no está dentro de la misma red/VPN que el laboratorio. | Verifica en el Security Group de `WindowsServer` que la regla SMB (445) permite el origen del cliente; recuerda que en AWS el acceso SMB desde fuera de la VPC suele requerir VPN o un origen explícitamente permitido en las reglas de entrada. |

> [!help] Preguntas Críticas (Autoevaluación)
> 1. ¿Qué diferencia fundamental hay entre un permiso de carpeta heredado por defecto y un permiso NTFS explícito con `icacls /grant`?
> 2. ¿Cómo mejora Access-Based Enumeration la privacidad de los datos en una empresa con muchos departamentos?
> 3. ¿Qué significa exactamente `(OI)(CI)` en el comando `icacls`?
> 4. 🔬 **Reto práctico:** Crea un archivo dentro de `Prueba3` desde el servidor: `New-Item -Path "C:\ShareData\Prueba3\heredado.txt" -ItemType File`. Luego ejecuta `icacls "C:\ShareData\Prueba3\heredado.txt"`. ¿Qué permisos tiene el archivo nuevo? ¿De dónde vienen esos permisos si no los has asignado explícitamente al archivo? ¿Qué habría pasado si en el Paso 1 hubieras usado `/grant` sin `(OI)(CI)`?
> 5. 🔬 **Reto práctico:** En un cliente Windows unido al dominio (o desde la propia consola del `WindowsServer` con una segunda cuenta), inicia sesión como `user2` (Bomberos) y navega a `\\WindowsServer.BOOCHAN.SPACE\` desde el Explorador de Archivos. ¿Ves la carpeta `Prueba3`? Cierra sesión e inicia como `user1` (Policia) y repite. ¿Qué diferencia hay? Haz una captura de pantalla de ambas vistas — eso es ABE trabajando en producción.

---

> [!caution] 🛑 Auditoría de Seguridad (RA.04)
> **Peligro:** Si no usas herencia `(OI)(CI)` al conceder el permiso NTFS, los archivos que cree un usuario del grupo `Policia` no podrán ser editados por sus compañeros del mismo grupo, generando tickets de soporte constantes — exactamente el mismo riesgo que en BoochanV3 al olvidar el flag `-d` de `setfacl`.

> [!success] 🏁 Punto de Control (Antes de seguir)
> - [ ] Comprueba con `icacls "C:\ShareData\Prueba3"` que el permiso del grupo `Policia` está aplicado con herencia `(OI)(CI)`.
> - [ ] ¿El recurso `Prueba3` muestra `FolderEnumerationMode : AccessBased` en `Get-SmbShare`?
> - [ ] En la Fase 8, cuando te conectes desde Windows, verifica que `user2` (Bomberos) no ve la carpeta `Prueba3` pero `user1` (Policia) sí.

---

### ✅ Entregables y cierre

> [!abstract] Qué tienes que tener hecho al acabar esta fase
> | Entregable | Dónde vive | Qué debe contener |
> | :--- | :--- | :--- |
> | **Entrada de apuntes** | `00_Apuntes/Trimestre_N/B5_Windows_Nube/v3-1-fase-7-seguridad-avanzada-permisos-ntfs-y-acces.md` | Estructura completa + **respuestas a las Preguntas Críticas y al 🔬 Reto** + **enlace del vídeo** |
> | **Vídeo** | Playlist `B5_Windows_Nube` (No listado) | Nombrado `V3.1 · Fase 7 — Seguridad Avanzada (Permisos NTFS y Access-Based Enumeration)`, con presentación, identidad y timestamps |
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
