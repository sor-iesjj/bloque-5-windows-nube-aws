## Fase 7 · Apartado 7 — 🚩 Resolución de problemas

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Seguridad Avanzada (Permisos NTFS y Access-Based Enumeration)**
> 🧭 Índice de la fase: [[Fase_7]]
>
> **📍 Cuándo se lee:** **Cuando algo no salga.** Búscate por el síntoma.

---

> [!bug] Troubleshooting (¿La seguridad falla?)
> | Problema | Causa Probable | Solución Sugerida |
> | :--- | :--- | :--- |
> | `user2` sigue viendo `Prueba3` en el Explorador. | El permiso NTFS del grupo `Bomberos` (o de un grupo genérico como "Usuarios autenticados") no se eliminó antes de aplicar ABE. | Ejecuta `icacls "C:\ShareData\Prueba3"` y revisa que solo aparezcan `Policia`, `Administradores` y `SYSTEM`. Si aparece otro grupo con acceso, quítalo con `icacls ... /remove:g "Grupo"`. |
> | `user1` ve la carpeta pero no puede entrar ni crear archivos. | El permiso a nivel de recurso compartido (share) no incluye a `Policia`, o el permiso NTFS quedó en solo lectura. | Revisa `Get-SmbShareAccess "Prueba3"` y confirma `FullAccess` para `BOOCHAN\Policia`; revisa también que el permiso NTFS sea `M` (Modify), no `R` (Read). |
> | `New-SmbShare` da error "The parameter is incorrect" con `-FolderEnumerationMode`. | Error tipográfico en el valor del parámetro (debe ser exactamente `AccessBased` o `Unrestricted`). | Revisa la sintaxis exacta con `Get-Help New-SmbShare -Parameter FolderEnumerationMode`. |
> | `icacls /inheritance:r` deja la carpeta sin ningún acceso, ni siquiera para el Administrador. | Se ejecutó `/inheritance:r` sin encadenar inmediatamente el `/grant` para Administradores y SYSTEM. | Vuelve a ejecutar el Paso 1 completo, en el orden indicado, empezando por conceder acceso a Administradores/SYSTEM antes de tocar nada más. |
> | No se puede acceder al recurso `\\WindowsServer.BOOCHAN.SPACE\Prueba3` desde un cliente externo a la VPC de AWS. | El Security Group bloquea el puerto 445 (SMB) para el origen concreto, o el cliente no está dentro de la misma red/VPN que el laboratorio. | Verifica en el Security Group de `WindowsServer` que la regla SMB (445) permite el origen del cliente; recuerda que en AWS el acceso SMB desde fuera de la VPC suele requerir VPN o un origen explícitamente permitido en las reglas de entrada. |

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_7.6_Procedimiento]] | [[Fase_7]] | [[Fase_7.8_Punto_de_Control]] |
