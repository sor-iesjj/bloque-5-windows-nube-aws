## Fase 7 · Apartado 6 — 🛠️ Procedimiento práctico

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Seguridad Avanzada (Permisos NTFS y Access-Based Enumeration)**
> 🧭 Índice de la fase: [[Fase_7]]
>
> **📍 Cuándo se lee:** **Con la VM delante.** Aquí está el trabajo.

---

> [!example] 🎬 Antes de empezar (todavía SIN grabar, y luego arranca)
> Ya conoces el método desde los prerrequisitos, así que va solo el recordatorio:
> 1. **Crea la entrada de apuntes** de esta fase (`v3-1-fase-7-seguridad-avanzada-permisos-ntfs-y-acces.md`) con su estructura, vacía.
> 2. **Léete los 3 pasos** del procedimiento enteros, para no atascarte a mitad del vídeo.
> 3. Ten **OBS** listo y comprueba **pantalla y micrófono**.
>
> Cuando lo tengas: **arranca la grabación, preséntate y muestra tu identidad**. A partir de ahí, **todo queda grabado** — incluido cualquier paso previo de preparación que venga a continuación.

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

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_7.5_Fundamento_Teorico]] | [[Fase_7]] | [[Fase_7.7_Resolucion_Problemas]] |
