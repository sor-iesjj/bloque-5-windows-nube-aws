## Fase 2 · Apartado 7 — 🚩 Resolución de problemas

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Preparación Inicial del Servidor**
> 🧭 Índice de la fase: [[Fase_2]]
>
> **📍 Cuándo se lee:** **Cuando algo no salga.** Búscate por el síntoma.

---

> [!bug] Troubleshooting (¿Algo no va bien?)
> | Problema | Causa Probable | Solución Sugerida |
> | :--- | :--- | :--- |
> | `Rename-Computer` no pide reinicio y el nombre no cambia. | Se ejecutó sin permisos de administrador. | Cierra PowerShell y vuelve a abrirlo con `Ejecutar como administrador`. Repite el comando. |
> | Tras el `-Restart` no puedo reconectar por RDP. | La instancia todavía está reiniciando. | Espera 1-2 minutos y vuelve a intentar la conexión con `mstsc`. |
> | La IP privada que veo no empieza por `172.31.`. | Tu Learner Lab usa una VPC con un rango distinto. | No pasa nada, usa la única IP privada IPv4 que te muestre `Get-NetIPConfiguration` con puerta de enlace asignada. |
> | `Install-Module -Name PSWindowsUpdate` falla o se queda colgado. | Problema temporal de repositorio de PowerShell Gallery no confiado. | Responde `S` (Sí) si pregunta por confiar en el repositorio, o añade `-Force` al comando. |

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_2.6_Procedimiento]] | [[Fase_2]] | [[Fase_2.8_Punto_de_Control]] |
