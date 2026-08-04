## Fase 7 · Apartado 5 — 📚 Fundamento teórico

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Seguridad Avanzada (Permisos NTFS y Access-Based Enumeration)**
> 🧭 Índice de la fase: [[Fase_7]]
>
> **📍 Cuándo se lee:** **Antes de teclear.** Los conceptos que necesitas.

---

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

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_7.4_Donde_Estamos]] | [[Fase_7]] | [[Fase_7.6_Procedimiento]] |
