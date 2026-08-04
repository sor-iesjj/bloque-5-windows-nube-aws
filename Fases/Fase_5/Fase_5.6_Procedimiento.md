## Fase 5 · Apartado 6 — 🛠️ Procedimiento práctico

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Gestión de Identidades (Usuarios y Grupos en Active Directory)**
> 🧭 Índice de la fase: [[Fase_5]]
>
> **📍 Cuándo se lee:** **Con la VM delante.** Aquí está el trabajo.

---

> [!example] 🎬 Antes de empezar (todavía SIN grabar, y luego arranca)
> Ya conoces el método desde los prerrequisitos, así que va solo el recordatorio:
> 1. **Crea la entrada de apuntes** de esta fase (`v3-1-fase-5-gestion-de-identidades-usuarios-y-grupos.md`) con su estructura, vacía.
> 2. **Léete los 3 pasos** del procedimiento enteros, para no atascarte a mitad del vídeo.
> 3. Ten **OBS** listo y comprueba **pantalla y micrófono**.
>
> Cuando lo tengas: **arranca la grabación, preséntate y muestra tu identidad**. A partir de ahí, **todo queda grabado** — incluido cualquier paso previo de preparación que venga a continuación.

> [!example] Paso 0: Confirmar que el módulo ActiveDirectory está disponible
> Al instalar el rol AD DS en la Fase 4, el módulo de PowerShell `ActiveDirectory` queda instalado automáticamente en el propio controlador de dominio. Comprueba que está cargado:
> ```powershell
> Get-Module -ListAvailable ActiveDirectory
> Import-Module ActiveDirectory
> ```
> Si vas a administrar el dominio desde otro equipo (no el propio DC), necesitarías instalar antes las **RSAT** (`Install-WindowsFeature RSAT-AD-PowerShell`), pero en este proyecto trabajamos siempre desde la consola de la propia instancia `WindowsServer`.

> [!example] Paso 1: Creación de la Estructura de Unidades Organizativas
> Creamos primero la OU raíz de departamentos y luego las dos sub-OUs donde vivirán los usuarios y grupos del proyecto:
>
> > [!info] 📚 Diccionario de Comandos: Para repasar la sintaxis de los cmdlets `*-AD*`, consulta el [[Diccionario_Comandos_Sistema]].
>
> ```powershell
> # OU raíz de departamentos
> New-ADOrganizationalUnit -Name "Departamentos" -Path "DC=boochan,DC=space"
>
> # Sub-OU para Policia
> New-ADOrganizationalUnit -Name "Policia" -Path "OU=Departamentos,DC=boochan,DC=space"
>
> # Sub-OU para Bomberos
> New-ADOrganizationalUnit -Name "Bomberos" -Path "OU=Departamentos,DC=boochan,DC=space"
> ```
>
> > [!tip] 💡 ¿Qué hace `-Path`?
> > El parámetro `-Path` indica el "domicilio" exacto del nuevo objeto dentro del árbol LDAP de Active Directory, escrito en notación **DN (Distinguished Name)**: se lee de derecha a izquierda, empezando por el dominio (`DC=boochan,DC=space`) y bajando por cada contenedor (`OU=Departamentos`). Es literalmente la misma lógica de rutas de carpetas, pero en formato LDAP.

> [!example] Paso 2: Creación de los Grupos de Seguridad
> Creamos los dos grupos del proyecto, ubicados dentro de su OU correspondiente. El grupo `Policia` tendrá acceso a las carpetas protegidas en la Fase 7, y `Bomberos` servirá para demostrar que los usuarios sin permisos no ven esas carpetas:
> ```powershell
> New-ADGroup -Name "Policia" `
>     -GroupScope Global -GroupCategory Security `
>     -Path "OU=Policia,OU=Departamentos,DC=boochan,DC=space"
>
> New-ADGroup -Name "Bomberos" `
>     -GroupScope Global -GroupCategory Security `
>     -Path "OU=Bomberos,OU=Departamentos,DC=boochan,DC=space"
> ```
>
> > [!tip] 💡 ¿Qué son `-GroupScope` y `-GroupCategory`?
> > - **`-GroupCategory Security`:** El grupo se usará para asignar permisos (frente a `Distribution`, que solo sirve para listas de correo).
> > - **`-GroupScope Global`:** Ámbito estándar para agrupar usuarios de un mismo dominio bajo un mismo criterio funcional — es el equivalente natural al GID que usábamos en Samba, pero sin necesidad de asignar ningún número manualmente: Active Directory gestiona el SID del grupo internamente.

> [!example] Paso 3: Creación de Usuarios y Asignación a Grupos
> Creamos dos usuarios: `user1` pertenecerá al grupo `Policia` y `user2` al grupo `Bomberos`. Esto nos permitirá demostrar en la Fase 7 que cada uno ve carpetas diferentes:
> ```powershell
> # Contraseña común para el proyecto (cumple la política de complejidad de AD)
> $securePass = ConvertTo-SecureString "P@ssw0rd" -AsPlainText -Force
>
> # Creamos user1 en la OU de Policia
> New-ADUser -Name "user1" -SamAccountName "user1" `
>     -UserPrincipalName "user1@boochan.space" `
>     -Path "OU=Policia,OU=Departamentos,DC=boochan,DC=space" `
>     -AccountPassword $securePass -Enabled $true -ChangePasswordAtLogon $false
>
> # Creamos user2 en la OU de Bomberos
> New-ADUser -Name "user2" -SamAccountName "user2" `
>     -UserPrincipalName "user2@boochan.space" `
>     -Path "OU=Bomberos,OU=Departamentos,DC=boochan,DC=space" `
>     -AccountPassword $securePass -Enabled $true -ChangePasswordAtLogon $false
>
> # Añadimos cada usuario a su grupo correspondiente
> Add-ADGroupMember -Identity "Policia" -Members "user1"
> Add-ADGroupMember -Identity "Bomberos" -Members "user2"
> ```
>
> > [!important] 💡 ¿Por qué no hace falta ningún `--uid-number` aquí?
> > **La simplificación clave de esta fase:** en Samba, olvidar `--uid-number` significaba que el sistema asignaba un identificador Unix impredecible, generando riesgos de reutilización y escalada de privilegios silenciosa. En AD DS nativo, cada usuario y grupo recibe automáticamente un **SID único y permanente** en el momento de su creación, gestionado internamente por el controlador de dominio — no hay ningún número que el administrador deba inventar ni vigilar manualmente. Es una responsabilidad completa del ecosistema Windows, no del administrador.

---

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_5.5_Fundamento_Teorico]] | [[Fase_5]] | [[Fase_5.7_Resolucion_Problemas]] |
