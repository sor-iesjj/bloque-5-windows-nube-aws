## Fase 4 · Apartado 10 — 🏁 Auditoría y cierre

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Aprovisionamiento del Dominio (AD DS nativo)**
> 🧭 Índice de la fase: [[Fase_4]]
>
> **📍 Cuándo se lee:** **Lo último.** No pases a la fase siguiente sin repasarlo.

---

> [!caution] 🛑 Auditoría y Evaluación (RA.03)
> **Peligro Crítico:** Si el DNS vuelve a apuntar a otro sitio en lugar de al propio servidor, los ordenadores dirán "No se encuentra el dominio" y nadie podrá iniciar sesión.

> [!success] 🏁 Punto de Control (Antes de seguir)
> - [ ] ¿`Get-Service NTDS, DNS, Kdc` muestra los tres servicios como `Running`?
> - [ ] ¿`Get-ADDomain` devuelve la información del dominio `BOOCHAN.SPACE` sin errores?
> - [ ] ¿`Resolve-DnsName _kerberos._tcp.BOOCHAN.SPACE` devuelve el registro SRV correcto?

---

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_4.9_Preguntas]] | [[Fase_4]] | **Fase 5** |
