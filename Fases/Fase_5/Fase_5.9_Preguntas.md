## Fase 5 · Apartado 9 — ❓ Preguntas críticas

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Gestión de Identidades (Usuarios y Grupos en Active Directory)**
> 🧭 Índice de la fase: [[Fase_5]]
>
> **📍 Cuándo se lee:** **Después de la instantánea.** Trabajo de mesa, en tu entrada.

---

> [!help] Preguntas Críticas (Autoevaluación)
> 1. ¿Por qué es mejor y más profesional dar permisos a un grupo que a un usuario individual?
> 2. En Samba hacía falta el servicio **winbind** para traducir identidades. Explica, en tus propias palabras, por qué en Windows Server con AD DS nativo ese paso desaparece por completo.
> 3. 🔬 **Reto práctico:** Ejecuta `Get-ADUser user1 -Properties MemberOf` y `Get-ADUser user2 -Properties MemberOf`. Anota a qué grupo pertenece cada uno. Ahora ejecuta `Get-ADGroupMember Policia` y `Get-ADGroupMember Bomberos`. ¿Coincide la pertenencia en ambos sentidos (usuario→grupo y grupo→usuario)?
> 4. 🔬 **Reto práctico:** Intenta crear un usuario sin especificar `-Path` (déjalo caer en el contenedor `Users` por defecto): `New-ADUser -Name "user3" -SamAccountName "user3" -AccountPassword $securePass -Enabled $true`. Ejecuta después `Get-ADUser user3 | Select DistinguishedName`. ¿En qué contenedor cayó? ¿Por qué en una organización real es mala práctica dejar que los usuarios se acumulen en el contenedor por defecto en lugar de una OU planificada?
> 5. ¿Cómo verificarías desde PowerShell que un grupo de seguridad existe y qué tipo de ámbito (`GroupScope`) tiene?

---

> [!danger] ⚠️ Las respuestas van en la ENTRADA, no en un documento aparte
> Estas preguntas demuestran que has **entendido** lo que has hecho, y no solo que has sabido copiar comandos. Se contestan **con tus palabras**. Una fase con el procedimiento perfecto y las preguntas en blanco está **incompleta**.

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_5.8_Punto_de_Control]] | [[Fase_5]] | [[Fase_5.10_Auditoria_y_Cierre]] |
