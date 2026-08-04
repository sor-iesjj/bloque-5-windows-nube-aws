## Fase 5 · Apartado 10 — 🏁 Auditoría y cierre

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Gestión de Identidades (Usuarios y Grupos en Active Directory)**
> 🧭 Índice de la fase: [[Fase_5]]
>
> **📍 Cuándo se lee:** **Lo último.** No pases a la fase siguiente sin repasarlo.

---

> [!caution] 🛑 Auditoría y Evaluación (RA.02)
> El alumno debe demostrar que la estructura de OUs, grupos y usuarios existe y es coherente. **Validación:** El comando `Get-ADUser user1 -Properties MemberOf` debe mostrar `Policia` entre sus grupos, y `Get-ADUser user2 -Properties MemberOf` debe mostrar `Bomberos`.

> [!success] 🏁 Punto de Control (Antes de seguir)
> - [ ] ¿Existen las OUs `Departamentos`, `Policia` y `Bomberos` (`Get-ADOrganizationalUnit -Filter *`)?
> - [ ] ¿El comando `Get-ADGroupMember Policia` devuelve `user1`?
> - [ ] ¿El comando `Get-ADGroupMember Bomberos` devuelve `user2`?
> - [ ] ¿Puedes iniciar sesión (o validar credenciales con `Test-ComputerSecureChannel` / login RDP) como `user1` y `user2` contra el dominio `BOOCHAN.SPACE`?

---

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_5.9_Preguntas]] | [[Fase_5]] | **Fase 6** |
