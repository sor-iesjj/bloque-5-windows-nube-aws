# 🚨 Guía de Errores y Resolución — Proyecto BoochanV3.1 (AWS / Windows Server 2025)

> [!info] Cómo usar esta guía
> Este documento recoge todos los errores conocidos de las 8 fases + Auditoría Final, organizados por fase. Cada error tiene un código único (`Fase.Número`). Si algo no funciona, localiza la fase en la que estás, busca el error que más se parece a lo que ves en pantalla y sigue el procedimiento paso a paso.
>
> A diferencia de BoochanV1.1 (donde el rescate de un servidor bloqueado pasaba por la ventana "Conectar..." de Hyper-V Manager, siempre disponible), en BoochanV3.1 tu red de seguridad si te quedas fuera por RDP es la **EC2 Serial Console** (o **SSM Session Manager**) — ver [[Comandos_AWS_CLI_Consola]] sección 3. Tenla siempre presente como último recurso.
>
> **Nota sobre las dos redes `10.0.0.x` vs `172.31.x.x`:** en este proyecto la IP privada de la VPC de AWS es `172.31.x.x`, mientras que el túnel WireGuard usa `10.0.0.0/24` (servidor `10.0.0.1`, cliente `10.0.0.2`) — son distintas, a diferencia de BoochanV2.1 (Azure) donde coincidían. Cuando un error mencione `10.0.0.1`, se refiere al servidor **a través del túnel VPN**; cuando mencione `172.31.x.x`, a la tarjeta de red real de la instancia dentro de AWS.

---

## 🧯 Sección 0 — Problemas Transversales de AWS (léela primero si algo va mal en general)

Estos problemas pueden aparecer en cualquier fase porque son de la capa cloud, no de una fase concreta.

### Error 0.1 — La instancia no responde y no sé si está encendida

> [!bug] Cuándo se produce
> En cualquier fase, si de repente no puedes conectar por RDP ni por VPN.

> [!example] Resolución
> 1. Primero comprueba que el **Learner Lab está en `Start Lab`** (círculo verde). Si está en `End Lab`, todo el entorno está congelado — vuelve a arrancarlo.
> 2. En la Consola de AWS → `EC2` → `Instancias` → comprueba el `Estado de la instancia` (`running` / `stopped`).
> 3. Si está `stopped`, pulsa `Estado de la instancia → Iniciar` y espera 2-3 minutos.
> 4. **Comprueba la IP pública tras el arranque**: si asociaste una **Elastic IP** en la Fase 1, se mantiene; si dependes de la IP pública automática, puede haber cambiado (ver [[Comandos_AWS_CLI_Consola]]).

### Error 0.2 — Perdí acceso por RDP y por VPN a la vez (bloqueo total)

> [!bug] Cuándo se produce
> Normalmente durante la Auditoría Final, si se restringe el Security Group o el Firewall de Windows Defender desde una IP que no está dentro del rango permitido.

> [!caution] ¿Hay que preocuparse?
> **Sí**, pero hay salida: la **EC2 Serial Console** (o SSM Session Manager) es tu vía de rescate — accede directamente a la instancia sin pasar por ninguna capa de red (ni Security Group ni Firewall de Windows). Consulta [[Comandos_AWS_CLI_Consola]] sección 3.

> [!example] Resolución
> 1. Consola de AWS → `EC2` → tu instancia → `Conectar` → `Consola de serie de EC2` (o `Administrador de sesiones`).
> 2. Inicia sesión con `Administrator` y su contraseña.
> 3. Corrige la regla de Security Group o de Firewall de Windows que te bloqueó, y reintenta el acceso normal.

### Error 0.3 — Se me está agotando el crédito del Learner Lab

> [!bug] Cuándo se produce
> Al revisar el crédito restante en el panel del Learner Lab.

> [!warning] ¿Hay que preocuparse?
> Sí, pero no es un error técnico — es un hábito de administración. Avisa a tu profesor. Recuerda que una instancia **Windows gasta crédito más rápido** que una Linux.

> [!example] Resolución
> Revisa si la instancia se quedó **encendida** fuera del horario de clase (o si dejaste el Lab en `Start Lab`). Pulsa siempre **End Lab** al terminar la sesión — congela todo el entorno sin gasto. Ver [[Comandos_AWS_CLI_Consola]] sección 4.

---

## Fase 1 — Infraestructura Cloud (AWS EC2)

---

### Error 1.1 — No puedo conectar por RDP ("No se puede establecer la conexión")

> [!bug] Cuándo se produce
> Justo después de lanzar la instancia, al intentar la primera conexión RDP.

> [!example] Resolución
> La instancia no ha terminado de arrancar/aprovisionarse. Espera 3-5 minutos (Windows Server tarda más que Ubuntu) y vuelve a intentarlo. Confirma también en la Consola que el estado de la instancia es `running` y que ha pasado las comprobaciones de estado (`2/2 checks passed`).

---

### Error 1.2 — No consigo la contraseña del Administrador / "Obtener contraseña" está en gris

> [!bug] Cuándo se produce
> Al intentar descifrar la contraseña de Windows con el Key Pair justo tras lanzar la instancia.

> [!info] ¿Hay que preocuparse?
> No, solo es cuestión de tiempo o de subir bien el `.pem`.

> [!example] Resolución
> 1. AWS tarda unos minutos tras el arranque en generar y cifrar la contraseña — si el botón "Obtener contraseña" aún no la da, espera 4-5 minutos.
> 2. Asegúrate de subir el archivo correcto: `boochan-key.pem` (no la clave pública, ni otro Key Pair de un intento anterior).
> 3. Recuerda: la contraseña se descifra **con el `.pem`**, pero luego el RDP se conecta con `Administrator` + esa contraseña, **no** con el `.pem`. Ver [[Comandos_AWS_CLI_Consola]] sección 1.

---

### Error 1.3 — "Este equipo no puede conectarse al equipo remoto" o error de conexión rechazada

> [!bug] Cuándo se produce
> Al intentar la conexión RDP, si el error no es de credenciales sino de conexión rechazada.

> [!caution] ¿Hay que preocuparse?
> **Sí.**

> [!example] Resolución
> El puerto 3389 no está abierto en el Security Group, o la regla se borró por error. Revisa **EC2 → Grupos de seguridad → `sg-boochan-[tunombre]` → Reglas de entrada** y confirma que la regla RDP (3389) existe. Ver [[Comandos_AWS_CLI_Consola]] sección 2.

---

### Error 1.4 — El servidor no responde al `ping`

> [!bug] Cuándo se produce
> Al intentar verificar conectividad con `ping` contra la IP pública o privada del servidor desde fuera.

> [!info] ¿Hay que preocuparse?
> No. Es el comportamiento esperado.

> [!example] Resolución
> El protocolo ICMP no está permitido por defecto en el Security Group (que solo permite lo que declaras explícitamente). No hace falta abrir el ping de entrada; usa RDP o pruebas de puertos TCP concretos para verificar conectividad.

---

### Error 1.5 — La pantalla de RDP se ve muy lenta o pixelada

> [!bug] Cuándo se produce
> Durante una sesión RDP activa, especialmente en redes de aula con ancho de banda limitado.

> [!example] Resolución
> En el cliente RDP, antes de conectar, baja la calidad de color/experiencia en las opciones avanzadas de la conexión (`mstsc` → `Mostrar opciones` → pestaña `Experiencia`).

---

## Fase 2 — Preparación Inicial del Servidor

---

### Error 2.1 — `Rename-Computer` no pide reinicio y el nombre no cambia

> [!bug] Cuándo se produce
> Al ejecutar el renombrado del servidor.

> [!info] ¿Hay que preocuparse?
> No, es un error de permisos sencillo de corregir.

> [!example] Resolución
> Cierra PowerShell y vuelve a abrirlo con `Ejecutar como administrador`. Repite el comando `Rename-Computer -NewName "WindowsServer" -Restart`.

---

### Error 2.2 — Tras el `-Restart` no puedo reconectar por RDP

> [!bug] Cuándo se produce
> Justo después de renombrar el equipo.

> [!info] ¿Hay que preocuparse?
> No.

> [!example] Resolución
> La instancia todavía está reiniciando. Espera 1-2 minutos y vuelve a intentar la conexión con `mstsc`.

---

### Error 2.3 — Dudo si tengo que fijar la IP privada como en Azure/Hyper-V

> [!bug] Cuándo se produce
> Al preparar la red del servidor, si vienes de haber hecho BoochanV1.1 (Hyper-V) o BoochanV2.1 (Azure).

> [!info] ¿Hay que preocuparse?
> No — en AWS este paso es más simple.

> [!example] Resolución
> En AWS **no hay que fijar la IP privada**: la `172.31.x.x` de la VPC ya es estable mientras la instancia exista. **No la cambies a mano** con `New-NetIPAddress` dentro de Windows — romperías la correspondencia que AWS mantiene en su tabla de red. Ver [[Comandos_AWS_CLI_Consola]] sección 2.

---

### Error 2.4 — `Install-Module -Name PSWindowsUpdate` falla o se queda colgado

> [!bug] Cuándo se produce
> Al instalar el módulo de gestión de Windows Update.

> [!warning] ¿Hay que preocuparse?
> Sí, bloquea el parcheo del sistema antes de instalar AD DS.

> [!example] Resolución
> Responde `S` (Sí) si PowerShell pregunta por confiar en el repositorio de PowerShell Gallery, o añade `-Force` al comando.

---

## Fase 3 — Conectividad VPN (WireGuard para Windows)

---

### Error 3.1 — `wireguard.exe /installtunnelservice` falla con "Address already in use"

> [!bug] Cuándo se produce
> Al instalar el túnel como servicio, si ya hay otra interfaz VPN activa con esa IP de un intento anterior.

> [!info] ¿Hay que preocuparse?
> No.

> [!example] Resolución
> ```powershell
> wireguard.exe /uninstalltunnelservice wg0
> wireguard.exe /installtunnelservice C:\WireGuard\wg0.conf
> ```

---

### Error 3.2 — No hay ping entre `10.0.0.1` y `10.0.0.2` (túnel)

> [!bug] Cuándo se produce
> El túnel arranca sin errores pero el `ping 10.0.0.1` desde el cliente del aula no responde.

> [!warning] ¿Hay que preocuparse?
> Sí. El túnel está mal configurado, la regla del Security Group es incorrecta, o las llaves están mal intercambiadas.

> [!example] Resolución — comprueba en este orden
> 1. **¿La regla `WireGuard` (51820) del Security Group existe y es protocolo UDP** (no TCP)? Es el error más habitual — revisa [[Comandos_AWS_CLI_Consola]]. Recuerda que este puerto se añade al Security Group **en la Fase 3** (no venía de la Fase 1).
> 2. **¿Las llaves públicas están cruzadas correctamente?** La llave pública del servidor debe estar en la configuración del cliente, y viceversa.
> 3. **¿Está el túnel activo en el cliente?** El botón de la app WireGuard debe mostrar "Desactivar".
> 4. **Verifica en el servidor:** `wg show`. Si aparece el peer con `latest handshake` reciente, funciona. Si no hay handshake, las llaves están mal intercambiadas.

---

### Error 3.3 — La llave pública pegada en `wg0.conf` tiene caracteres invisibles

> [!bug] Cuándo se produce
> Al copiar la llave pública desde la app WireGuard y pegarla con `notepad`.

> [!caution] ¿Hay que preocuparse?
> **Sí.** WireGuard no muestra ningún error claro; simplemente no habrá conexión.

> [!example] Resolución
> 1. Verifica el contenido real: `Select-String -Path C:\WireGuard\wg0.conf -Pattern "PublicKey"`.
> 2. Debe ser una sola línea limpia, sin espacios ni caracteres `<` o `>`.
> 3. Si hay algo raro, reescribe la línea desde cero y reinicia el túnel (`/uninstalltunnelservice` + `/installtunnelservice`).

---

### Error 3.4 — El cliente no encuentra el `Endpoint`

> [!bug] Cuándo se produce
> Al intentar levantar el túnel desde el PC del aula, si el `Endpoint` del fichero de configuración está mal escrito o la IP pública cambió.

> [!example] Resolución
> Recuerda que el `Endpoint` es la **Elastic IP** de tu instancia (no la IP privada `172.31.x.x` ni la del túnel `10.0.0.x`):
> ```ini
> Endpoint = TU_ELASTIC_IP:51820
> ```
> Verifica la Elastic IP actual en la Consola de AWS. Como es una Elastic IP asociada, se mantiene entre `End Lab`/`Start Lab`, así que no debería cambiar — si lo hizo, comprueba que sigue asociada a la instancia (ver Error 0.1).

---

## Fase 4 — Aprovisionamiento del Dominio (AD DS nativo)

---

### Error 4.1 — Error "El nombre de equipo del equipo local no coincide con..." o similar

> [!bug] Cuándo se produce
> Al ejecutar `Install-ADDSForest`.

> [!caution] ¿Hay que preocuparse?
> **Sí.** El servidor no se renombró correctamente en la Fase 2, o el reinicio del renombrado no se completó antes de instalar AD DS.

> [!example] Resolución
> Ejecuta `$env:COMPUTERNAME` y confirma que devuelve `WindowsServer`. Si no, repite el Paso 1 de la Fase 2 (`Rename-Computer`) y su reinicio antes de reintentar.

---

### Error 4.2 — `Install-ADDSForest` avisa sobre configuración de red / IP dinámica

> [!bug] Cuándo se produce
> Al ejecutar `Install-ADDSForest`, un aviso sobre el adaptador con IP asignada por DHCP.

> [!info] ¿Hay que preocuparse?
> En AWS, normalmente **no** — es un aviso, no un error bloqueante.

> [!example] Resolución
> A diferencia de Azure (donde había que marcar la IP como estática en el portal), en AWS la IP privada `172.31.x.x` de la VPC es estable de facto aunque Windows la vea como "DHCP". El asistente puede mostrar un aviso amarillo; puedes continuar. Lo que **no** debes hacer es cambiar la IP a mano dentro de Windows (ver Error 2.3). Lo crítico es que el **DNS** del adaptador apunte a sí mismo (`127.0.0.1`) tras la promoción.

---

### Error 4.3 — `Install-ADDSForest` se detiene con un error de prerrequisitos (`Test-ADDSForestInstallation`)

> [!bug] Cuándo se produce
> Cuando el asistente de promoción falla antes de completarse.

> [!caution] ¿Hay que preocuparse?
> **Sí.** Si la promoción no terminó, el dominio no existe o está a medias.

> [!example] Resolución
> Lee el último mensaje de error visible en pantalla — PowerShell describe en texto claro qué requisito previo no se cumplió. Las causas más frecuentes: RAM insuficiente (revisa que la instancia sigue siendo `t3.large` y no se cambió por error a un tipo menor), o nombre NetBIOS ya existe de un intento anterior a medio provisionar (`Uninstall-ADDSDomainController -Force` si aplica).

---

### Error 4.4 — El servidor no responde tras el reinicio automático de `Install-ADDSForest`

> [!bug] Cuándo se produce
> Justo después de que el cmdlet termine y reinicie la instancia.

> [!info] ¿Hay que preocuparse?
> No. El primer arranque tras la promoción tarda más de lo habitual mientras se inicializan los servicios de AD DS.

> [!example] Resolución
> Espera 2-3 minutos adicionales antes de intentar reconectar. Es normal.

---

### Error 4.5 — `Resolve-DnsName` no encuentra el registro SRV de Kerberos

> [!bug] Cuándo se produce
> Al verificar el DNS tras la promoción del dominio.

> [!caution] ¿Hay que preocuparse?
> **Sí.** Es uno de los errores más críticos del proyecto. Sin el DNS correcto, el dominio no resolverá nombres y ningún cliente podrá unirse.

> [!example] Resolución
> 1. Ejecuta `Get-Service DNS` y confirma que está `Running`.
> 2. Revisa `Get-DnsClientServerAddress` — debe apuntar a `127.0.0.1` o a la propia IP privada `172.31.x.x` (ambos valores son válidos, ambos apuntan al propio servidor).
> 3. Si apunta a otra cosa, corrígelo con `Set-DnsClientServerAddress -InterfaceIndex [n] -ServerAddresses "127.0.0.1"`.

---

### Error 4.6 — No puedo reconectar por RDP tras el reinicio de la Fase 4 (parece que la VPN bloquea el acceso)

> [!bug] Cuándo se produce
> Tras el reinicio automático de `Install-ADDSForest`, si ya venías administrando el servidor a través del túnel WireGuard desde la Fase 3.

> [!info] ¿Hay que preocuparse?
> No, es el comportamiento esperado si sigues el itinerario en orden.

> [!example] Resolución
> Reconecta usando la IP del túnel: `mstsc /v:10.0.0.1` (la IP del servidor dentro del túnel WireGuard). Si el túnel tampoco responde, verifica `wg show` y que el servicio `WireGuardTunnel$wg0` siga `Running` tras el reinicio.

---

## Fase 5 — Gestión de Identidades

---

### Error 5.1 — `Get-ADUser user1` devuelve error "Cannot find an object"

> [!bug] Cuándo se produce
> Al verificar el Punto de Control de la Fase 5.

> [!example] Resolución
> Ejecuta `Get-ADUser -Filter *` para listar todos los usuarios del dominio y localizar errores de nombre o de OU. Revisa que el `-Path` usado en `New-ADUser` no tenga un error tipográfico en la notación DN.

---

### Error 5.2 — "The password does not meet the length, complexity, or history requirement"

> [!bug] Cuándo se produce
> Al crear un usuario con `New-ADUser` con una contraseña que no cumple la política de complejidad de AD.

> [!info] ¿Hay que preocuparse?
> No. Usa una contraseña con mayúsculas, minúsculas, números y símbolos, como `P@ssword2026!`.

---

### Error 5.3 — "An attempt was made to add an object to the directory with a name that is already in use"

> [!bug] Cuándo se produce
> Al intentar crear un usuario o grupo que ya fue creado en un intento anterior de esta fase.

> [!info] ¿Hay que preocuparse?
> No. Solo hay que eliminar el objeto existente y volver a crearlo.

> [!example] Resolución
> ```powershell
> Remove-ADUser user1 -Confirm:$false
> Remove-ADUser user2 -Confirm:$false
> Remove-ADGroup Policia -Confirm:$false
> Remove-ADGroup Bomberos -Confirm:$false
> ```
> Después vuelve a ejecutar los pasos de creación desde el principio.

---

### Error 5.4 — "Insufficient access rights to perform the operation"

> [!bug] Cuándo se produce
> Al ejecutar cualquier cmdlet `New-AD*`.

> [!example] Resolución
> La sesión de PowerShell no se está ejecutando como Administrador, o el usuario no tiene privilegios de Admins. del dominio. Abre PowerShell con "Ejecutar como administrador" y confirma la sesión.

---

### Error 5.5 — `New-ADOrganizationalUnit` falla con "already exists"

> [!bug] Cuándo se produce
> Al crear las OUs, si ya existen de una ejecución anterior.

> [!example] Resolución
> Comprueba antes con `Get-ADOrganizationalUnit -Filter 'Name -eq "Departamentos"'`, o continúa sin volver a crearla.

---

### Error 5.6 — No puedo conectar por RDP/PowerShell remoto desde mi equipo para administrar el dominio

> [!bug] Cuándo se produce
> Al intentar administrar el dominio desde fuera de la sesión RDP habitual.

> [!example] Resolución
> El Security Group de AWS no tiene abierto el puerto RDP (3389), o la regla se restringió por error. Revisa las reglas de entrada de `sg-boochan-[tunombre]` en la Consola — RDP está abierto desde la Fase 1. (Nota: en este proyecto se administra por RDP, no por WinRM/PowerShell remoto, así que el 5985 no se abre en ninguna fase.)

---

## Fase 6 — Almacenamiento con Cuotas (FSRM)

---

### Error 6.1 — `Get-FsrmQuota` da error "The specified path does not exist"

> [!bug] Cuándo se produce
> Se aplicó la cuota antes de crear la carpeta, o hay un error tipográfico en la ruta.

> [!example] Resolución
> Verifica con `Test-Path "C:\ShareData\Prueba1"` y repite `New-FsrmQuota` con la ruta correcta.

---

### Error 6.2 — El cmdlet `New-FsrmQuotaTemplate` no se reconoce

> [!bug] Cuándo se produce
> Al intentar crear la plantilla de cuota.

> [!example] Resolución
> El rol FSRM no está instalado o el módulo no se ha importado. Ejecuta `Get-WindowsFeature FS-Resource-Manager` para confirmar la instalación e `Import-Module FileServerResourceManager`.

---

### Error 6.3 — Un usuario sigue escribiendo por encima de 5 GB en la carpeta

> [!bug] Cuándo se produce
> Tras aplicar la cuota, si el límite no bloquea la escritura.

> [!example] Resolución
> Se aplicó una plantilla con `-SoftLimit`, o la carpeta tiene otra cuota distinta aplicada encima. Ejecuta `Get-FsrmQuota -Path "C:\ShareData\Prueba1" | Select SoftLimit, Size` y confirma que `SoftLimit` es `False`.

---

### Error 6.4 — `New-FsrmQuota` da error "A quota already exists for this path"

> [!bug] Cuándo se produce
> Al aplicar la cuota, si ya se aplicó en un intento anterior.

> [!example] Resolución
> Ejecuta `Set-FsrmQuota -Path "C:\ShareData\Prueba1" -Template "Limite5GB-Estricto"` para actualizarla, o `Remove-FsrmQuota` y vuelve a crearla.

---

### Error 6.5 — El disco `C:` de la instancia se queda sin espacio antes de probar las cuotas

> [!bug] Cuándo se produce
> Al intentar generar archivos de prueba grandes para comprobar el bloqueo de la cuota.

> [!example] Resolución
> El volumen raíz de la instancia (50 GB gp3) ya tiene el sistema operativo instalado. Comprueba el espacio libre con `Get-Volume -DriveLetter C` antes de crear archivos de prueba grandes; nunca crees archivos de más de unos pocos GB fuera de las carpetas con cuota.

---

## Fase 7 — Seguridad Avanzada (NTFS y Access-Based Enumeration)

---

### Error 7.1 — `user2` sigue viendo `Prueba3` en el Explorador

> [!bug] Cuándo se produce
> Tras activar ABE, el usuario sin permisos sigue viendo la carpeta protegida.

> [!caution] ¿Hay que preocuparse?
> **Sí.** Es el fallo central de esta fase — invalida toda la protección.

> [!example] Resolución
> El permiso NTFS del grupo `Bomberos` (o de un grupo genérico como "Usuarios autenticados") no se eliminó antes de aplicar ABE. Ejecuta `icacls "C:\ShareData\Prueba3"` y revisa que solo aparezcan `Policia`, `Administradores` y `SYSTEM`. Si aparece otro grupo con acceso, quítalo con `icacls "C:\ShareData\Prueba3" /remove:g "Grupo"`.

---

### Error 7.2 — `user1` ve la carpeta pero no puede entrar ni crear archivos

> [!bug] Cuándo se produce
> Tras aplicar permisos NTFS y crear el recurso compartido.

> [!example] Resolución
> El permiso a nivel de recurso compartido (share) no incluye a `Policia`, o el permiso NTFS quedó en solo lectura. Revisa `Get-SmbShareAccess "Prueba3"` y confirma `FullAccess` para `BOOCHAN\Policia`; revisa también que el permiso NTFS sea `M` (Modify), no `R` (Read).

---

### Error 7.3 — `New-SmbShare` da error "The parameter is incorrect" con `-FolderEnumerationMode`

> [!bug] Cuándo se produce
> Al crear el recurso compartido con ABE.

> [!example] Resolución
> Error tipográfico en el valor del parámetro (debe ser exactamente `AccessBased` o `Unrestricted`). Revisa la sintaxis exacta con `Get-Help New-SmbShare -Parameter FolderEnumerationMode`.

---

### Error 7.4 — `icacls /inheritance:r` deja la carpeta sin ningún acceso, ni siquiera para el Administrador

> [!bug] Cuándo se produce
> Al romper la herencia sin encadenar inmediatamente el `/grant` para Administradores y SYSTEM.

> [!example] Resolución
> Vuelve a ejecutar el Paso 1 completo, en el orden indicado: primero conceder acceso a Administradores/SYSTEM, después tocar cualquier otro permiso.

---

### Error 7.5 — No se puede acceder al recurso `\\WindowsServer.BOOCHAN.SPACE\Prueba3` desde un cliente externo

> [!bug] Cuándo se produce
> Al probar el acceso al recurso compartido desde un equipo que no está dentro de la VPC de AWS ni conectado por VPN.

> [!example] Resolución
> El Security Group bloquea el puerto 445 (SMB) para el origen concreto, o el cliente no está dentro de la misma red/VPN que el laboratorio. Verifica en el Security Group que la regla SMB (445) permite el origen del cliente; recuerda que en AWS el acceso SMB desde fuera de la VPC requiere el túnel VPN activo (Fase 3).

---

## Fase 8 — Integración del Cliente (Windows 11)

---

### Error 8.1 — "No se encuentra el dominio" al unirse

> [!bug] Cuándo se produce
> Al intentar unir el PC del aula al dominio.

> [!caution] ¿Hay que preocuparse?
> **Sí.**

> [!example] Resolución
> El cliente está usando el DNS del router del aula, no el del servidor, o la VPN no está activa. Comprueba que el DNS primario del adaptador es `10.0.0.1` (la IP del servidor a través del túnel WireGuard) y que la aplicación WireGuard muestra el túnel como "Activado".

---

### Error 8.2 — "Error de relación de confianza" al autenticarse

> [!bug] Cuándo se produce
> Desfase horario (Clock Skew) superior a 5 minutos entre el PC del aula y el servidor — muy fácil que ocurra porque son dos máquinas físicamente distintas con relojes independientes.

> [!warning] ¿Hay que preocuparse?
> Sí. Ningún usuario podrá autenticarse hasta que los relojes estén sincronizados.

> [!example] Resolución
> 1. Abre PowerShell **como Administrador** en el PC del aula.
> 2. Fuerza la sincronización: `w32tm /resync /force`.
> 3. Si persiste, compara la hora del servidor (`Get-Date` por RDP/túnel) con la del cliente.

---

### Error 8.3 — `Add-Computer` falla con "No se puede contactar con el dominio"

> [!bug] Cuándo se produce
> Al ejecutar la unión al dominio desde PowerShell.

> [!example] Resolución
> Igual que el Error 8.1: fallo de conectividad o de DNS hacia el servidor por el túnel (`10.0.0.1`), casi siempre porque la VPN no está activa. Verifica primero con `nslookup BOOCHAN.SPACE` (debe resolver el servidor a través del túnel) y con `wg show` en el servidor antes de reintentar la unión.

---

### Error 8.4 — La unidad de red `Z:` desaparece al reiniciar Windows

> [!bug] Cuándo se produce
> El mapeo de la unidad no se marcó como persistente.

> [!info] ¿Hay que preocuparse?
> No.

> [!example] Resolución
> ```powershell
> net use Z: \\WindowsServer.BOOCHAN.SPACE\prueba1 /user:BOOCHAN\user1 /persistent:yes
> ```
> Recuerda que además la VPN debe estar activa **antes** de que Windows intente reconectar la unidad al iniciar sesión.

---

### Error 8.5 — RSAT no se descarga / se queda "buscando actualizaciones"

> [!bug] Cuándo se produce
> Al instalar las herramientas RSAT en el PC del aula.

> [!example] Resolución
> El PC del aula no tiene salida a Internet en ese momento — esto es independiente de la VPN, RSAT se descarga desde los servidores de Microsoft, no desde el servidor del proyecto. Comprueba la conexión a Internet general del equipo.

---

### Error 8.6 — Al iniciar sesión con `BOOCHAN\user1` dice que no puede contactar con el servidor

> [!bug] Cuándo se produce
> En el primer login con credenciales de dominio, tras la unión y el reinicio.

> [!caution] ¿Hay que preocuparse?
> **Sí.**

> [!example] Resolución
> El túnel WireGuard debe estar activo **antes** de intentar el login, no solo antes de la unión al dominio. Abre la aplicación WireGuard y activa el túnel antes de introducir usuario y contraseña en la pantalla de bloqueo.

---

## Auditoría Final — Hardening

---

### Error F.1 — Te quedas fuera del servidor al aplicar `DefaultInboundAction Block`

> [!bug] Cuándo se produce
> Si estás conectado por RDP o PowerShell remoto desde una IP que no está dentro de `10.0.0.0/24` (por ejemplo, si te conectaste directamente por la Elastic IP pública sin pasar por la VPN) y aplicas la política de bloqueo por defecto en el perfil de Dominio del Firewall de Windows.

> [!caution] ¿Hay que preocuparse?
> **Sí**, aunque en BoochanV3.1 el rescate siempre está disponible a través de la **EC2 Serial Console** (o SSM Session Manager) — ver [[Comandos_AWS_CLI_Consola]] sección 3.

> [!example] Resolución
> 1. Abre la EC2 Serial Console de tu instancia `WindowsServer` desde la Consola de AWS (`Conectar → Consola de serie de EC2`) e inicia sesión con `Administrator`.
> 2. Revisa las reglas activas: `Get-NetFirewallRule -DisplayName "AD DS*" | Select-Object DisplayName, Enabled, Direction, Action`.
> 3. Corrige o añade la regla que falte.
> 4. Antes de volver a aplicar la política de bloqueo, verifica siempre desde qué IP estás conectado con `quser` o `Get-NetTCPConnection -State Established` en el propio servidor — y confirma que es una IP dentro de `10.0.0.0/24` (la VPN).

---

### Error F.2 — Olvidé restringir alguna regla del Security Group y sigue en `Anywhere-IPv4`

> [!bug] Cuándo se produce
> Al revisar el Security Group tras completar el Paso 1 de la Auditoría Final, si alguna de las reglas de servicios de dominio quedó sin modificar.

> [!warning] ¿Hay que preocuparse?
> Sí, es la única "puerta trasera" real que puede quedar abierta en un laboratorio cloud — el equivalente a una regla mal cerrada en un firewall de producción.

> [!example] Resolución
> Repasa la tabla completa del Paso 1 de la [[../Fases/Auditoria_Final|Auditoría Final]] regla por regla en la Consola (`EC2 → Grupos de seguridad → Reglas de entrada`), y confirma que el **Origen** (`Source`) de cada regla de servicio de dominio (excepto WireGuard, 51820, que se mantiene en `Anywhere-IPv4` a propósito) está restringido al rango de la VPN `10.0.0.0/24`.

---

### Error F.3 — Tras cerrar el Security Group, la VPN también deja de funcionar

> [!bug] Cuándo se produce
> Al restringir por error la regla `WireGuard` (51820) del Security Group a `10.0.0.0/24` en lugar de dejarla en `Anywhere-IPv4`.

> [!caution] ¿Hay que preocuparse?
> **Sí.** Es una paradoja de configuración: para entrar en la VPN, el puerto que la establece debe aceptar tráfico desde fuera de la propia VPN.

> [!example] Resolución
> Desde la EC2 Serial Console (Error F.1), o editando el Security Group desde la Consola de AWS, devuelve el **Origen** de la regla `WireGuard` (51820/UDP) a `Anywhere-IPv4` (`0.0.0.0/0`). Es la única regla de la Auditoría Final que **no** se restringe a `10.0.0.0/24`.

---

### Error F.4 — Terminé la sesión sin `End Lab` y seguí gastando crédito

> [!bug] Cuándo se produce
> Al volver a una sesión posterior y ver que el crédito bajó más de lo esperado.

> [!warning] ¿Hay que preocuparse?
> Sí, es la causa más habitual de gasto innecesario en el Learner Lab, agravada porque una instancia Windows gasta más rápido que una Linux.

> [!example] Resolución
> Acostúmbrate a pulsar **End Lab** al terminar CADA sesión — congela todo el entorno sin gasto. Detener la instancia (`Stop`) no basta si dejas una Elastic IP asociada, que sigue consumiendo. Ver [[Comandos_AWS_CLI_Consola]] sección 4.

---

*Proyecto BoochanV3.1 — Curso 2025/2026*
