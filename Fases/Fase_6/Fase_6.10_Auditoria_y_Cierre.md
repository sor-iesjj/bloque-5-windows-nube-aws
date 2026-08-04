## Fase 6 · Apartado 10 — 🏁 Auditoría y cierre

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Almacenamiento con Cuotas (FSRM)**
> 🧭 Índice de la fase: [[Fase_6]]
>
> **📍 Cuándo se lee:** **Lo último.** No pases a la fase siguiente sin repasarlo.

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

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_6.9_Preguntas]] | [[Fase_6]] | **Fase 7** |
