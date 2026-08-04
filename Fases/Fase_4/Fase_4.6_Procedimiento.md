## Fase 4 · Apartado 6 — 🛠️ Procedimiento práctico

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Aprovisionamiento del Dominio (AD DS nativo)**
> 🧭 Índice de la fase: [[Fase_4]]
>
> **📍 Cuándo se lee:** **Con la VM delante.** Aquí está el trabajo.

---

> [!example] Paso 1: Instalación del Rol AD DS
> A diferencia de BoochanV3, aquí no descargamos nada de un repositorio externo — el rol AD DS forma parte del propio Windows Server, solo hay que activarlo:
>
> > [!info] 📚 Diccionario de Comandos: Consulta el [[Diccionario_Comandos_Sistema]] para entender al detalle cómo funcionan los cmdlets administrativos que usaremos aquí.
>
> ```powershell
> # Instala el rol AD DS junto con las herramientas de gestión (RSAT: Get-ADUser, dsa.msc, etc.)
> Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
> ```
> > [!tip] 💡 ¿Qué hace este comando?
> > - **`Install-WindowsFeature`:** El equivalente Windows a `apt install`. Activa un componente del propio sistema operativo (no descarga un paquete externo de internet, salvo que falten archivos de origen).
> > - **`-IncludeManagementTools`:** Instala también las herramientas gráficas y de PowerShell para administrar el dominio después (`Get-ADUser`, `Get-ADGroup`, la consola "Usuarios y equipos de Active Directory", etc.). Sin este parámetro, el rol se instala pero no tendrás herramientas cómodas para gestionarlo.
> >
> > Este paso solo **instala los binarios** del rol. Todavía no existe ningún dominio — eso ocurre en el Paso 2.

> [!example] Paso 2: Promoción a Controlador de Dominio con `Install-ADDSForest`
> Este es el equivalente Windows al script `provision_boochan.sh` de BoochanV3, pero como un único cmdlet nativo del sistema, sin script externo que descargar ni ejecutar:
> ```powershell
> Install-ADDSForest `
>   -DomainName "BOOCHAN.SPACE" `
>   -DomainNetbiosName "BOOCHAN" `
>   -DomainMode "Win2025" `
>   -ForestMode "Win2025" `
>   -InstallDNS `
>   -SafeModeAdministratorPassword (ConvertTo-SecureString "P@ssw0rd" -AsPlainText -Force) `
>   -Force
> ```
>
> > [!tip] 💡 ¿Qué hace cada parámetro?
> > - **`-DomainName "BOOCHAN.SPACE"`:** El Realm completo del dominio, equivalente al `REALM_NAME` de `provision_boochan.sh` en BoochanV3.
> > - **`-DomainNetbiosName "BOOCHAN"`:** El nombre corto, equivalente al `DOMAIN_NAME` del script de BoochanV3.
> > - **`-DomainMode` / `-ForestMode "Win2025"`:** El **nivel funcional** del dominio y del bosque. Usamos el nivel más alto disponible porque este es un dominio nuevo, de un único Controlador de Dominio, sin necesidad de mantener compatibilidad con versiones antiguas de Windows Server.
> > - **`-InstallDNS`:** Instala y configura automáticamente el rol **DNS Server** integrado con Active Directory. Es el equivalente al `--dns-backend=SAMBA_INTERNAL` del script de Samba — pero aquí no hace falta indicarlo como opción de bajo nivel, es un simple interruptor.
> > - **`-SafeModeAdministratorPassword`:** La contraseña del modo de recuperación DSRM (ver Diccionario de Conceptos). Es una contraseña **distinta** de la del Administrador del dominio, pensada solo para emergencias de recuperación del directorio.
> > - **`-Force`:** Evita que el asistente pida confirmación interactiva en cada paso, útil para practicar el comando de forma reproducible.
>
> > [!caution] ⚠️ El servidor se reiniciará automáticamente
> > Al finalizar, `Install-ADDSForest` reinicia el servidor sin pedir confirmación adicional (salvo que uses `-NoRebootOnCompletion`). Es normal y esperado — la promoción a Controlador de Dominio requiere un reinicio para que todos los servicios (NTDS, DNS, Kerberos) arranquen con la nueva configuración. **Perderás la sesión RDP** durante el proceso, que suele tardar **entre 5 y 10 minutos** en total, incluyendo el reinicio.
> >
> > **Si el comando falla antes de llegar al reinicio:** revisa el mensaje de error en pantalla — a diferencia del script bash de BoochanV3 (donde había que interpretar mensajes de Samba en la terminal), aquí PowerShell describe en texto claro qué requisito previo no se cumplió, por ejemplo nombre de equipo incorrecto o falta de RAM. Revisa la tabla de troubleshooting al final de esta fase.
>
> > [!note] 📄 Comparación de enfoque: el script de V3 frente al cmdlet nativo de V3.1
> > A diferencia de BoochanV3, aquí no hay ningún fichero `provision_boochan.sh` que clonar desde un repositorio Git ni valores por defecto ocultos en variables de bash. Todo el "guion" de la instalación está contenido en los parámetros explícitos del propio cmdlet que acabas de ejecutar — no hay nada oculto ni que depender de una URL externa proporcionada por el profesor. Esto elimina de raíz un tipo de error muy habitual en BoochanV3 (clonar el repositorio equivocado, o con el Realm de otra versión del proyecto).

> [!example] Paso 3: Verificación de Servicios
> Tras el reinicio automático, vuelve a conectarte al servidor por RDP (ahora ya como miembro del dominio, usando la IP de la VPN: `mstsc /v:10.0.0.1`) y comprueba que el "corazón" del dominio está latiendo:
> ```powershell
> # Comprobar que los servicios de Active Directory están activos
> Get-Service -Name NTDS, DNS, Kdc | Select-Object Name, Status
> ```
> Los tres servicios (`NTDS` — base de datos del directorio, `DNS` — DNS integrado, `Kdc` — Centro de distribución de claves Kerberos) deben mostrar el estado `Running`.

> [!example] Paso 4: Verificación del DNS
> Es vital confirmar que el servidor se mira a sí mismo para resolver nombres de red:
> ```powershell
> # Debe devolver 127.0.0.1 o la propia IP privada de AWS (172.31.x.x); ambas son válidas tras la promoción
> Get-DnsClientServerAddress -InterfaceAlias "*Ethernet*" -AddressFamily IPv4
> ```
> > [!tip] 💡 ¿Por qué a veces aparece la IP `172.31.x.x` en lugar de `127.0.0.1`?
> > Durante `Install-ADDSForest -InstallDNS`, Windows a veces reconfigura automáticamente el DNS del adaptador para que apunte a su propia IP privada en lugar de al loopback (`127.0.0.1`). Ambos valores son funcionalmente equivalentes en este caso — los dos apuntan al propio servidor. Si prefieres mantener exactamente `127.0.0.1` (por coherencia con la configuración de la Fase 2), puedes reaplicarlo manualmente con `Set-DnsClientServerAddress`.

---

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_4.5_Fundamento_Teorico]] | [[Fase_4]] | [[Fase_4.7_Resolucion_Problemas]] |
