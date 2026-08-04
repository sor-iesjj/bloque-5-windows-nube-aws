## Fase 5 · Apartado 7 — 🚩 Resolución de problemas

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Gestión de Identidades (Usuarios y Grupos en Active Directory)**
> 🧭 Índice de la fase: [[Fase_5]]
>
> **📍 Cuándo se lee:** **Cuando algo no salga.** Búscate por el síntoma.

---

> [!bug] Troubleshooting (¿Los usuarios no funcionan?)
> | Problema | Causa Probable | Solución Sugerida |
> | :--- | :--- | :--- |
> | `Get-ADUser user1` devuelve error "Cannot find an object". | El usuario no se creó correctamente o hay un error tipográfico en el `-Path`. | Ejecuta `Get-ADUser -Filter *` para listar todos los usuarios del dominio y localizar errores de nombre o de OU. |
> | Error: "The password does not meet the length, complexity, or history requirement". | La política de contraseñas de dominio exige complejidad. | Usa una contraseña con mayúsculas, minúsculas, números y símbolos como `P@ssw0rd`. |
> | Error: "An attempt was made to add an object to the directory with a name that is already in use". | El usuario o grupo ya existía de un intento anterior. | Ejecuta `Remove-ADUser user1 -Confirm:$false` o `Remove-ADGroup Policia -Confirm:$false` y vuelve a crearlo. |
> | Error: "Insufficient access rights to perform the operation". | La PowerShell no se está ejecutando como Administrador de dominio. | Abre PowerShell con "Ejecutar como administrador" y confirma que tu sesión tiene privilegios de Admins. del dominio. |
> | `New-ADOrganizationalUnit` falla con "already exists". | La OU ya existe de una ejecución anterior. | Comprueba con `Get-ADOrganizationalUnit -Filter 'Name -eq "Departamentos"'` antes de recrearla, o continúa sin crearla de nuevo. |
> | No puedo conectar por RDP desde mi equipo. | El Security Group de AWS no tiene abierto el puerto RDP (3389), o la regla se restringió antes de tiempo. | Revisa las reglas de entrada del Security Group de `WindowsServer` en la Consola de AWS (EC2 → Grupos de seguridad) — RDP (3389) está abierto desde la Fase 1. (El 5985/WinRM NO se abre en este proyecto: administramos por RDP.) |

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_5.6_Procedimiento]] | [[Fase_5]] | [[Fase_5.8_Punto_de_Control]] |
