## Fase 8 · Apartado 7 — 🚩 Resolución de problemas

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Integración del Cliente (Windows 11)**
> 🧭 Índice de la fase: [[Fase_8]]
>
> **📍 Cuándo se lee:** **Cuando algo no salga.** Búscate por el síntoma.

---

> [!bug] Troubleshooting (¿No puedes unirte?)
> | Problema | Causa Probable | Solución Sugerida |
> | :--- | :--- | :--- |
> | "No se encuentra el dominio". | El cliente está usando el DNS del router, no el nuestro, o la VPN no está activa. | Comprueba que el DNS primario es `10.0.0.1` y que la VPN está activa. |
> | "Error de relación de confianza". | Desfase horario (Clock Skew) superior a 5 minutos. | Ejecuta `w32tm /resync /force` (Paso 2) antes de reintentar. |
> | `Add-Computer` falla con "No se puede contactar con el dominio". | Igual que el primer caso: fallo de conectividad o de DNS hacia `10.0.0.1`. | Verifica primero con `nslookup BOOCHAN.SPACE` antes de reintentar la unión. |
> | La unidad `Z:` no aparece al reiniciar. | El mapeo no es persistente. | Añade `/persistent:yes` al final del comando `net use`. Recuerda que la VPN debe estar activa antes de que Windows intente reconectar la unidad. |
> | RSAT no se descarga / se queda "buscando actualizaciones". | El PC del aula no tiene salida a Internet en ese momento. | Comprueba la conexión a Internet del propio PC, independiente de la VPN. |

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_8.6_Procedimiento]] | [[Fase_8]] | [[Fase_8.8_Punto_de_Control]] |
