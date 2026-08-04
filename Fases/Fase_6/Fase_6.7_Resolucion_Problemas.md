## Fase 6 · Apartado 7 — 🚩 Resolución de problemas

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Almacenamiento con Cuotas (FSRM)**
> 🧭 Índice de la fase: [[Fase_6]]
>
> **📍 Cuándo se lee:** **Cuando algo no salga.** Búscate por el síntoma.

---

> [!bug] Troubleshooting (¿La cuota no aparece o no funciona?)
> | Problema | Causa Probable | Solución Sugerida |
> | :--- | :--- | :--- |
> | `Get-FsrmQuota` da error "The specified path does not exist". | Se aplicó la cuota antes de crear la carpeta, o hay un error tipográfico en la ruta. | Verifica con `Test-Path "C:\ShareData\Prueba1"` y repite el `New-FsrmQuota` con la ruta correcta. |
> | El cmdlet `New-FsrmQuotaTemplate` no se reconoce. | El rol FSRM no está instalado o el módulo no se ha importado. | Ejecuta `Get-WindowsFeature FS-Resource-Manager` para confirmar la instalación e `Import-Module FileServerResourceManager`. |
> | Un usuario sigue escribiendo por encima de 5 GB. | Se aplicó una plantilla con `-SoftLimit`, o la carpeta tiene otra cuota distinta aplicada encima. | Ejecuta `Get-FsrmQuota -Path "C:\ShareData\Prueba1" \| Select SoftLimit, Size` y confirma que `SoftLimit` es `False`. |
> | `New-FsrmQuota` da error "A quota already exists for this path". | La cuota ya se aplicó en un intento anterior. | Ejecuta `Set-FsrmQuota -Path "C:\ShareData\Prueba1" -Template "Limite5GB-Estricto"` para actualizarla, o `Remove-FsrmQuota` y vuelve a crearla. |
> | El disco `C:` de la instancia se queda sin espacio antes de llegar a probar las cuotas. | El tamaño de disco EBS por defecto de la instancia `t3.large` es limitado y ya tiene el sistema operativo instalado. | Comprueba el espacio libre con `Get-Volume -DriveLetter C` antes de crear archivos de prueba grandes; nunca crees archivos de más de unos pocos GB fuera de las carpetas con cuota. |

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_6.6_Procedimiento]] | [[Fase_6]] | [[Fase_6.8_Punto_de_Control]] |
