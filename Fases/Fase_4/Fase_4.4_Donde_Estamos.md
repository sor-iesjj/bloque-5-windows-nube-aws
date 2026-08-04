## Fase 4 · Apartado 4 — 🎯 ¿Dónde estamos?

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Aprovisionamiento del Dominio (AD DS nativo)**
> 🧭 Índice de la fase: [[Fase_4]]
>
> **📍 Cuándo se lee:** **Antes de empezar.** De dónde vienes y a dónde llegas.

---

> [!info] Vienes de Fase 3
> Tienes un servidor con nombre correcto (`WindowsServer`), IP privada confirmada en la VPC de AWS, accesible de forma cifrada a través de un túnel WireGuard (`10.0.0.1`). Ahora necesitas darle la funcionalidad de un verdadero **Controlador de Dominio** — el "cerebro" que gestiona usuarios, grupos, autenticación y autorización.

> [!warning] El Problema
> Sin un dominio, Windows 11 en el aula es un equipo aislado. Los usuarios se loguean localmente (usuario/contraseña guardados en el PC). No hay forma centralizada de gestionar identidades, no hay Single Sign-On, no hay políticas de grupo. Si necesitas cambiar la contraseña de un usuario, debes hacerlo en cada PC manualmente. Además, Kerberos (el protocolo de seguridad profesional) requiere un dominio para funcionar.

> [!success] Objetivo de esta Fase
> Instalar el rol **AD DS (Active Directory Domain Services)** en el servidor y promocionarlo con `Install-ADDSForest`. Esto creará el dominio **`BOOCHAN.SPACE`** (Realm) / **`BOOCHAN`** (NetBIOS) como un "reino" Kerberos con servicios interdependientes: base de datos de directorio (NTDS), DNS integrado, Kerberos (autenticación) y replicación. Desde ahora, los usuarios se autenticarán contra el dominio, no contra máquinas individuales.

> [!tip] Hoja de Ruta
> 1. Abrir 13 puertos en el Security Group de AWS (Kerberos, DNS, LDAP, SMB, RPC, NTP — todo lo que AD necesita)
> 2. Instalar el rol AD DS con `Install-WindowsFeature AD-Domain-Services -IncludeManagementTools`
> 3. Promocionar el servidor a Controlador de Dominio con `Install-ADDSForest` (tarda varios minutos y reinicia solo)
> 4. Verificar que los servicios `NTDS`, `DNS` y `Kdc` están activos tras el reinicio
> 5. Comprobar que el DNS del propio servidor apunta a sí mismo
> 6. Validar que Kerberos funciona: `Resolve-DnsName _kerberos._tcp.BOOCHAN.SPACE`
> 7. Listar usuarios creados automáticamente: `Get-ADUser -Filter *` (verás Administrator, krbtgt, etc.)
>
> **Resultado Final:** Dominio `BOOCHAN.SPACE` completamente provisionado y operativo. El servidor es ahora un verdadero Controlador de Dominio profesional.
> **Siguiente:** Fase 5 (Usuarios) — crearás usuarios del dominio (user1, user2) con `New-ADUser` y los organizarás en unidades organizativas.

---

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_4.3_Obligaciones_Grabacion]] | [[Fase_4]] | [[Fase_4.5_Fundamento_Teorico]] |
