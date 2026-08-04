## Fase 6 · Apartado 9 — ❓ Preguntas críticas

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Almacenamiento con Cuotas (FSRM)**
> 🧭 Índice de la fase: [[Fase_6]]
>
> **📍 Cuándo se lee:** **Después de la instantánea.** Trabajo de mesa, en tu entrada.

---

> [!help] Preguntas Críticas (Autoevaluación)
> 1. ¿Por qué FSRM no necesita un equivalente al `fstab` de Linux para que la cuota sobreviva a un reinicio del servidor?
> 2. ¿Qué diferencia hay entre una cuota **estricta (hard)** y una **flexible (soft)** en FSRM? ¿Cuál usamos en este proyecto y por qué?
> 3. En Samba, si olvidabas la palabra `loop` en el `fstab`, el servidor podía entrar en pánico al arrancar. ¿Por qué en FSRM no existe un riesgo equivalente?
> 4. 🔬 **Reto práctico:** Intenta llenar la carpeta `C:\ShareData\Prueba1` creando un archivo de 6 GB (mayor que la cuota de 5 GB): `fsutil file createnew C:\ShareData\Prueba1\lleno.dat 6442450944`. ¿Qué mensaje de error aparece? Borra el archivo con `Remove-Item C:\ShareData\Prueba1\lleno.dat` y comprueba con `Get-FsrmQuota -Path "C:\ShareData\Prueba1"` que el campo `Usage` vuelve a estar cerca de 0.
> 5. 🔬 **Reto práctico:** Ejecuta `Get-Volume -DriveLetter C` y anota el tamaño total del volumen `C:`. Compáralo con los 5 GB de cada cuota de carpeta. ¿Qué le ocurriría al volumen `C:` completo (y por tanto al sistema operativo) si `Prueba1` y `Prueba3` no tuvieran cuota y un usuario las llenara sin límite?

---

> [!danger] ⚠️ Las respuestas van en la ENTRADA, no en un documento aparte
> Estas preguntas demuestran que has **entendido** lo que has hecho, y no solo que has sabido copiar comandos. Se contestan **con tus palabras**. Una fase con el procedimiento perfecto y las preguntas en blanco está **incompleta**.

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_6.8_Punto_de_Control]] | [[Fase_6]] | [[Fase_6.10_Auditoria_y_Cierre]] |
