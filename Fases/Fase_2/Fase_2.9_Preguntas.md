## Fase 2 · Apartado 9 — ❓ Preguntas críticas

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Preparación Inicial del Servidor**
> 🧭 Índice de la fase: [[Fase_2]]
>
> **📍 Cuándo se lee:** **Después de la instantánea.** Trabajo de mesa, en tu entrada.

---

> [!help] Preguntas Críticas (Autoevaluación)
> 1. ¿Por qué en Windows Server no hace falta "purgar" nada, a diferencia de Ubuntu Server en BoochanV3?
> 2. ¿Qué ocurre si promocionas el servidor a Controlador de Dominio (Fase 4) sin haberlo renombrado antes? ¿Por qué es tan importante hacerlo en este orden?
> 3. ¿Por qué en AWS no hace falta marcar la IP privada como "Estática" desde ningún panel, a diferencia de lo que ocurriría en otros proveedores cloud?
> 4. 🔬 **Reto práctico:** Ejecuta `Get-NetIPConfiguration` y compara la salida con `Get-NetIPAddress`. ¿Qué información adicional aporta el primero (puerta de enlace, DNS) que el segundo no muestra directamente?
> 5. 🔬 **Reto práctico:** Ejecuta `Get-Process | Sort-Object WS -Descending | Select-Object -First 5` para ver los 5 procesos que más RAM consumen ahora mismo. Anota el total de RAM libre con `Get-Counter '\Memory\Available MBytes'`. Guarda ese dato — lo compararás con la Fase 4, cuando AD DS esté instalado, para ver cuánta RAM consume el dominio.

---

> [!danger] ⚠️ Las respuestas van en la ENTRADA, no en un documento aparte
> Estas preguntas demuestran que has **entendido** lo que has hecho, y no solo que has sabido copiar comandos. Se contestan **con tus palabras**. Una fase con el procedimiento perfecto y las preguntas en blanco está **incompleta**.

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_2.8_Punto_de_Control]] | [[Fase_2]] | [[Fase_2.10_Auditoria_y_Cierre]] |
