## Fase 6 · Apartado 6 — 🛠️ Procedimiento práctico

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Almacenamiento con Cuotas (FSRM)**
> 🧭 Índice de la fase: [[Fase_6]]
>
> **📍 Cuándo se lee:** **Con la VM delante.** Aquí está el trabajo.

---

> [!example] 🎬 Antes de empezar (todavía SIN grabar, y luego arranca)
> Ya conoces el método desde los prerrequisitos, así que va solo el recordatorio:
> 1. **Crea la entrada de apuntes** de esta fase (`b5-aws-6-almacenamiento-con-cuotas-fsrm.md`) con su estructura, vacía.
> 2. **Léete los 5 pasos** del procedimiento enteros, para no atascarte a mitad del vídeo.
> 3. Ten **OBS** listo y comprueba **pantalla y micrófono**.
>
> Cuando lo tengas: **arranca la grabación, preséntate y muestra tu identidad**. A partir de ahí, **todo queda grabado** — incluido cualquier paso previo de preparación que venga a continuación.

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

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_6.5_Fundamento_Teorico]] | [[Fase_6]] | [[Fase_6.7_Resolucion_Problemas]] |
