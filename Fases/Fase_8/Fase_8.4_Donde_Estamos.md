## Fase 8 · Apartado 4 — 🎯 ¿Dónde estamos?

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Integración del Cliente (Windows 11)**
> 🧭 Índice de la fase: [[Fase_8]]
>
> **📍 Cuándo se lee:** **Antes de empezar.** De dónde vienes y a dónde llegas.

---

> [!info] Vienes de Fase 7
> El servidor `WindowsServer` es ahora un "reino" completo: dominio `BOOCHAN.SPACE`, usuarios, grupos, discos protegidos, y permisos NTFS granulares. Todo está funcionando perfectamente desde PowerShell y desde el Administrador del servidor. Sin embargo, los usuarios del aula están esperando en sus PCs Windows 11 — ahora necesitas que esos equipos confíen en el servidor y usen sus identidades de dominio.

> [!warning] El Problema
> Aunque cliente y servidor "hablan el mismo idioma" (ambos son Windows), antes de que se confíen deben cumplirse varias condiciones: (1) el cliente debe encontrar el servidor por DNS, (2) sincronizar el reloj exactamente (Kerberos rechaza diferencias > 5 minutos), (3) establecer una "relación de confianza" registrándose en Active Directory, (4) permitir que los usuarios inicien sesión con sus credenciales de dominio. Si algo falla, el usuario ve "No se puede encontrar el dominio" o "Error de relación de confianza".

> [!success] Objetivo de esta Fase
> **Unir Windows 11 al dominio BOOCHAN.SPACE** de forma que los usuarios puedan iniciar sesión con sus credenciales de dominio (ej. `BOOCHAN\user1`) y acceder a las carpetas compartidas del servidor con los permisos que se les asignaron en fases anteriores. Es el momento de la verdad: la infraestructura híbrida en la nube (servidor Windows Server 2025 en AWS + cliente Windows 11 físico del aula) funcionando en sinergia.

> [!tip] Hoja de Ruta
> 1. **Validar VPN:** Activar el túnel WireGuard en el PC del aula para acceder a la red privada 10.0.0.0/24
> 2. **Configurar DNS de Windows:** Cambiar DNS primario a 10.0.0.1 (el servidor), DNS secundario a 8.8.8.8 (fallback a internet)
> 3. **Sincronizar reloj:** Ejecutar `w32tm /resync /force` para emparejar la hora exactamente con el servidor
> 4. **Unir al dominio:** Con `Add-Computer -DomainName "BOOCHAN.SPACE" -Restart` desde PowerShell, o vía Configuración → Cuentas → Acceso profesional o educativo → Conectar
> 5. **Reiniciar Windows:** Obligatorio para aplicar los cambios de dominio
> 6. **Primer login:** Iniciar sesión con `BOOCHAN\user1` y su contraseña desde la pantalla de inicio
> 7. **Instalar RSAT:** Herramientas administrativas para gestionar usuarios/grupos desde Windows gráficamente
> 8. **Mapear carpetas de red:** Conectar `\\WindowsServer.BOOCHAN.SPACE\prueba1` y `prueba3` como unidades de red (Z:, por ejemplo)
>
> **Resultado Final:** Windows 11 es ahora un cliente legítimo del dominio. Los usuarios pueden iniciar sesión, acceder a carpetas según sus permisos de grupo, y crear archivos que el servidor reconoce automáticamente.
> **Siguiente:** Fase completada — el proyecto es funcional de extremo a extremo. Servidor Windows Server como DC en la nube (AWS), usuarios en AD, almacenamiento seguro, y clientes Windows integrados a través del túnel VPN. Solo queda la Auditoría Final de seguridad.

---

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_8.3_Obligaciones_Grabacion]] | [[Fase_8]] | [[Fase_8.5_Fundamento_Teorico]] |
