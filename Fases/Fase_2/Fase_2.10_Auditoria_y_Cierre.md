## Fase 2 · Apartado 10 — 🏁 Auditoría y cierre

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Preparación Inicial del Servidor**
> 🧭 Índice de la fase: [[Fase_2]]
>
> **📍 Cuándo se lee:** **Lo último.** No pases a la fase siguiente sin repasarlo.

---

> [!caution] 🛑 Auditoría y Evaluación (RA.02)
> El alumno debe demostrar que el servidor tiene el nombre e IP correctos antes de avanzar. **Riesgo Crítico:** Si el nombre de equipo no es `WindowsServer` antes de la Fase 4, el FQDN del propio Controlador de Dominio quedará con un nombre erróneo de forma permanente.

> [!success] 🏁 Punto de Control (Antes de seguir)
> - [ ] ¿`$env:COMPUTERNAME` devuelve exactamente `WindowsServer`?
> - [ ] ¿`Get-NetIPConfiguration` muestra una IP en el rango `172.31.x.x` (o el rango de tu VPC)?
> - [ ] ¿El DNS del adaptador apunta a `127.0.0.1`?
> - [ ] ¿`Get-WindowsUpdate` no muestra actualizaciones pendientes?

---

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_2.9_Preguntas]] | [[Fase_2]] | **Fase 3** |
